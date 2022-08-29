Label：对K8s中各种资源进行分类、分组，添加一个具有特别属性的一个标签。

selector：通过一个过滤的语法进行查找到对应标签的资源

## 新增标签

```bash
kubectl label pods busybox app=busybox
```

## 查看标签

```bash
# 展示pods的标签
kubectl get pods --show-labels

# 过滤标签为app=busybox的pods
kubectl get pods -l app=busybox

kubectl get pods -A -l 'k8s-app in (metrics-server,kubernetes-dashboard)'
kubectl get pods -A -l 'version!=v1,app=nginx'

```

## 删除标签

```bash
kubectl label pods busybox app-
```

## 修改标签

```bash
kubectl label pods busybox app=busybox2 --overwrite
```

