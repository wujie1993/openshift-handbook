# 快速使用

## 初始化账号

在集群创建之初会初始化一个system:admin用户，该用户不需要密码即可登录，具有对于集群的完全控制权限，我们在日常使用时不建议使用system:admin用户，而是根据项目需要创建对应的项目用户。

比如创建一个开发者账号dev，首先在htpasswd文件中创建一个dev用户，密码为devpass。

在htpasswd文件中创建一个dev用户，密码为devpass

```text
htpasswd -b /etc/origin/master/htpasswd dev devpass
```

以dev账号登录openshift

```text
oc login -udev -pdevpass
```

创建一个demo项目，这时openshift会创建一个demo命名空间并将dev账号的登录上下文切换到demo项目中

```text
oc new-project demo
```

通过镜像名openshift/hello-openshift部署一个应用

```text
oc new-app openshift/hello-openshift
```

