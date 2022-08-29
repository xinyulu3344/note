## 基本概念

DaemonSet：守护进程集，缩写ds。在所有节点或匹配的节点上都部署一个Pod

## 部署DaemonSet的应用

运行集群存储：ceph、glusterd

节点CNI网络插件：calico

节点日志收集：fluentd或者filebeat

节点的监控：node exporter

服务暴露：部署一个ingress nginx

## yaml定义

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  generation: 2
  labels:
    app: nginx
  name: nginx-ds
  namespace: default
spec:
  revisionHistoryLimit: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:1.15.2
        imagePullPolicy: IfNotPresent
        name: nginx
        ports:
        - containerPort: 80
          protocol: TCP
      dnsPolicy: ClusterFirst
      nodeSelector:
        ds: "true"
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
  # 更新策略: 建议使用: OnDelete
  updateStrategy:
    rollingUpdate:
      maxUnavailable: 1
    type: RollingUpdate
```

