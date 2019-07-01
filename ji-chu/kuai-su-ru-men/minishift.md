# Minishift

## 下载地址

{% embed url="https://github.com/minishift/minishift/releases" %}

## 准备

如果是在虚拟机中安装，需要先确认宿主机是否开启了cpu嵌套虚拟化

```bash
egrep '(vmx|svm)' /proc/cpuinfo
```

输出内容非空则说明宿主机已经开启了cpu嵌套虚拟化

查看系统是否已经开启嵌套虚拟化

```bash
// intel处理器
cat /sys/module/kvm_intel/parameters/nested

// amd处理器
cat /sys/module/kvm_amd/parameters/nested
```

输出结果为`N`说明没有开启嵌套虚拟化，`Y`说明已经开启嵌套虚拟化

如果为`N`接着使用以下步骤开启嵌套虚拟化

{% code-tabs %}
{% code-tabs-item title="/etc/modprobe.d/kvm-nested.conf" %}
```text
options kvm-intel nested=1
options kvm-intel enable_shadow_vmcs=1
options kvm-intel enable_apicv=1
options kvm-intel ept=1
```
{% endcode-tabs-item %}
{% endcode-tabs %}

重新加载kvm模块

```bash
// intel处理器
modprobe -r kvm_intel
modprobe -a kvm_intel

// amd处理器
modprobe -r kvm_amd
modprobe -a kvm_amd
```

## 安装启动

安装虚拟化软件

```bash
yum install -y libvirt qemu-kvm
```

安装虚拟化驱动

```bash
usermod -a -G libvirt $(whoami)
newgrp libvirt
curl -L https://github.com/dhiltgen/docker-machine-kvm/releases/download/v0.10.0/docker-machine-driver-kvm-ubuntu14.04 -o /usr/local/bin/docker-machine-driver-kvm
chmod +x /usr/local/bin/docker-machine-driver-kvm
```

启动libvirt

```bash
systemctl start libvirtd
systemctl enable libvirtd
```

启动minishift

```bash
minishift start
```

{% hint style="info" %}
如果启动失败使用命令`minishift delete -f`清理后再次重试
{% endhint %}





