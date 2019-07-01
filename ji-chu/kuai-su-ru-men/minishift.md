# Minishift

## 下载地址

{% embed url="https://github.com/minishift/minishift/releases" %}

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



