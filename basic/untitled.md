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

## 部署openshift应用

创建一个demo项目，这时openshift会创建一个demo命名空间并将dev账号的登录上下文切换到demo项目中

```text
oc new-project demo
```

通过镜像名openshift/hello-openshift部署一个应用

```text
oc new-app openshift/hello-openshift
```

将应用暴露成外部服务

```text
oc expose svc/hello-openshift
```

部署完成后通过以下地址访问应用

```text
curl http://hello-openshift-demo.apps.oc.local
```

{% hint style="info" %}
由于通过router发布的服务以域名方式访问，因此需要配置域名解析，修改hosts文件并添加以下条目
{% endhint %}

{% code-tabs %}
{% code-tabs-item title="/etc/hosts" %}
```text
+ 192.168.149.129 hello-openshift-demo.apps.oc.local
```
{% endcode-tabs-item %}
{% endcode-tabs %}

通过以上步骤我们发布了一个简单的openshift定制镜像，以同样的方式还可以运行镜像仓库中的其他镜像。

而需要注意的一点是，许多镜像中都是以root用户运行的，存在安全上的问题，openshift默认限制是不允许容器镜像以root用户运行，因此我们会发现运行容器时出现了类似于permission denied的错误。如果需要运行此类镜像，需要为当前命名空间的sa账号赋予更高的权限，通过以下命令为sa账号授权:

```text
oc adm policy add-scc-to-user anyuid -z default -n demo
```

完成授权后通过命令对应用进行重新部署

```text
oc rollout latest {dc名称}
```

