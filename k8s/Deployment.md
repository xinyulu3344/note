## Deployment概念

用于部署无状态服务，是最常用的控制器。一般用于管理维护企业内部无状态的微服务。可以管理多个Pod的副本，实现无缝迁移、自动扩容缩容、自动灾难恢复、一键回滚等功能。

## yaml文件定义

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  progressDeadlineSeconds: 600
  replicas: 3 # 副本数
  revisionHistoryLimit: 10 # 历史记录保留的个数
  selector:
    matchLabels:
      app: nginx
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

