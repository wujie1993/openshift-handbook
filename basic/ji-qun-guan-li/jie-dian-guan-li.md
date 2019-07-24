# 节点管理

## 变更节点角色

对于节点的角色变更ansible采用的是只加不减的处理方式，打个比方：原先有个infra节点，后续需要调整为compute节点，这时就需要先手动把该节点的`region:infra`标签删除

```text
oc label node {节点名称} region-
```

再修改invenory文件，将对应节点上openshift\_node\_labels配置项中的`'region': 'infra'`标签删除，然后更新节点配置

```text
ansible-playbook playbooks/openshift-node/config.yml
```



