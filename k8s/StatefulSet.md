## 基本概念

StatefulSet主要用于管理有状态应用程序。缩写为sts

StatefulSet管理的Pod，名称格式：xxx-0、xxx-1、xxx-2

StatefulSet的Service是headless的Service，一般格式为statefulSetName-{0...N-1}.serviceName.namespace.svc.cluster.local

## StatefulSet注意事项

一般statefulSet用于用于有以下一个或多个需求的应用程序：

* 需要稳定的独一无二的网络标识符
* 需要持久化数据
* 需要有序的、优雅的部署和扩展
* 需要有序的自动滚动更新

## 定义资源文件

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: nginx
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.15.2
        ports:
        - containerPort: 80
          name: web
```

## StatefulSet的更新策略

