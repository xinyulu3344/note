## 基本概念

DaemonSet：守护进程集，缩写ds。在所有节点或匹配的节点上都部署一个Pod

## 部署DaemonSet的应用

运行集群存储：ceph、glusterd

节点CNI网络插件：calico

节点日志收集：fluentd或者filebeat

节点的监控：node exporter

服务暴露：部署一个ingress nginx