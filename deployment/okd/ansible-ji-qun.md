# Ansible集群

## 脚本地址

{% embed url="https://github.com/openshift/openshift-ansible" %}

## 快速安装

规划安装3台节点，三个都作为主节点和计算节点

| 节点名称 | IP地址 | ansible | master | node |
| :--- | :--- | :--- | :--- | :--- |
| okd-0 | 192.168.149.129 | yes | yes | yes |
| okd-1 | 192.168.149.130 | no | yes | yes |
| okd-2 | 192.168.149.131 | no | yes | yes |

首先修改每个节点的主机名

{% code-tabs %}
{% code-tabs-item title="okd-0:~/ \#" %}
```text
hostnamectl set-hostname okd-0
hostname okd-0
```
{% endcode-tabs-item %}

{% code-tabs-item title="okd-1:~/ \#" %}
```
hostnamectl set-hostname okd-1
hostname okd-1
```
{% endcode-tabs-item %}

{% code-tabs-item title="okd-2:~/ \#" %}
```
hostnamectl set-hostname okd-2
hostname okd-2
```
{% endcode-tabs-item %}
{% endcode-tabs %}

配置hosts名称解析，在每个节点的/etc/hosts文件中追加以下内容

{% code-tabs %}
{% code-tabs-item title="okd-X:/etc/hosts" %}
```text
192.168.149.129 okd-0
192.168.149.130 okd-1
192.168.149.131 okd-2
```
{% endcode-tabs-item %}
{% endcode-tabs %}

以okd-0为集群引导节点，配置到所有节点的ssh免密码登录

```text
ssh-keygen
ssh-copy-id 192.168.149.129
ssh-copy-id 192.168.149.130
ssh-copy-id 192.168.149.131
```

下载安装脚本（以v3.9为例）

{% code-tabs %}
{% code-tabs-item title="okd-0:/root/Downloads/ \#" %}
```bash
wget https://github.com/openshift/openshift-ansible/archive/openshift-ansible-3.9.90-1.tar.gz
tar xvf openshift-ansible-3.9.90-1.tar.gz && cd openshift-ansible-openshift-ansible-3.9.90-1
```
{% endcode-tabs-item %}
{% endcode-tabs %}

为每个节点安装并更新必要的依赖软件和docker

{% code-tabs %}
{% code-tabs-item title="okd-X:~/ \#" %}
```text
yum install -y wget git net-tools bind-utils yum-utils iptables-services bridge-utils bash-completion kexec-tools sos psacct
yum update -y
reboot
yum install -y docker
systemctl start docker
systemctl enable docker
```
{% endcode-tabs-item %}
{% endcode-tabs %}

进入引导节点，配置epel源并安装ansible

{% code-tabs %}
{% code-tabs-item title="okd-0:/root/Downloads/openshift-ansible-openshift-ansible-3.9.90-1/ \#" %}
```text
yum -y install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
sed -i -e "s/^enabled=1/enabled=0/" /etc/yum.repos.d/epel.repo
yum -y --enablerepo=epel install ansible pyOpenSSL
```
{% endcode-tabs-item %}
{% endcode-tabs %}

配置inventory

{% code-tabs %}
{% code-tabs-item title="okd-0:/etc/ansible/hosts" %}
```text
[OSEv3:children]
masters
nodes
etcd

[OSEv3:vars]
openshift_deployment_type=origin
openshift_release=3.9

osm_cluster_network_cidr=10.128.0.0/14
openshift_portal_net=172.30.0.0/16
osm_host_subnet_length=9
openshift_disable_check=disk_availability,memory_availability

# 配置多租户网络隔离
os_sdn_network_plugin_name='redhat/openshift-ovs-multitenant'
# Router服务的默认域名后缀
openshift_master_default_subdomain=apps.oc.local
# 配置docker日志的滚动清理策略和非加密镜像仓库地址
openshift_docker_options=OPTIONS='--log-driver json-file --insecure-registry=172.30.0.0/16 --selinux-enabled --log-opt max-size=1M --log-opt max-file=3'

# 安装Hawkular,启用metrics
openshift_metrics_install_metrics=true
openshift_metrics_hawkular_hostname=hawkular-metrics.oc.local

# 配置认证方式
openshift_master_identity_providers=[{'name': 'htpasswd_auth', 'login': 'true', 'challenge': 'true', 'kind': 'HTPasswdPasswordIdentityProvider'}]

[masters]
okd-0

[etcd]
okd-0
okd-1
okd-2

[nodes]
okd-0 openshift_schedulable=true openshift_node_labels="{'region': 'infra', 'zone': 'default'}"
okd-1 openshift_schedulable=true openshift_node_labels="{'region': 'infra', 'zone': 'default'}"
okd-2 openshift_schedulable=true openshift_node_labels="{'region': 'infra', 'zone': 'default'}"
```
{% endcode-tabs-item %}
{% endcode-tabs %}

执行安装前检查

{% code-tabs %}
{% code-tabs-item title="okd-0:/root/Downloads/openshift-ansible-openshift-ansible-3.9.90-1/ \#" %}
```text
ansible-playbook playbooks/prerequisites.yml
```
{% endcode-tabs-item %}
{% endcode-tabs %}

