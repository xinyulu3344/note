## Volume能干啥

1. 解决容器重启后文件丢失的问题
2. 解决多个容器共享数据的问题

## 常用类型

### emptyDir

如果删除Pod，emptyDir卷中的数据也会一并删除。一般用于Pod中不同Container共享数据。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-volume-emptydir
spec:
  containers:
  - name: c1
    image: nginx:1.15.3
    imagePullPolicy: IfNotPresent
    volumeMounts:
    - mountPath: /cache
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir:
      # "": 默认值，使用node节点的默认medium
      # Memory: 告诉 Kubernetes 为你挂载 tmpfs（基于 RAM 的文件系统）。 虽然 tmpfs 速度非常快，但是要注意它与磁盘不同。 tmpfs 在节点重启时会被清除，并且你所写入的所有文件都会计入容器的内存消耗，受容器内存限制约束。
      medium: ""
      sizeLimit: 
```

### hostPath

将主机节点文件系统上的文件或目录挂载到 Pod 中

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-volume-hostpath
spec:
  containers:
  - name: c1
    image: nginx:1.15.3
    imagePullPolicy: IfNotPresent
    volumeMounts:
    - mountPath: /cache
      name: cache-volume
  volumes:
  - name: cache-volume
    hostPath:
      # 宿主机上的目录
      path: /data
      # Directory: 在给定路径上必须存在的目录
      # DirectoryOrCreate: 如果在给定路径上什么都不存在，那么将根据需要创建空目录，权限设置为 0755，具有与 kubelet 相同的组和属主信息。
      # File: 在给定路径上必须存在的文件。
      # FileOrCreate: 如果在给定路径上什么都不存在，那么将在那里根据需要创建空文件，权限设置为 0644，具有与 kubelet 相同的组和所有权。
      # Socket: 在给定路径上必须存在的 UNIX 套接字
      # CharDevice: 在给定路径上必须存在的字符设备。
      # BlockDevice: 在给定路径上必须存在的块设备。
      type: DirectoryOrCreate
```

### NFS

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-volume-nfs
spec:
  containers:
  - name: c1
    image: nginx:1.15.3
    imagePullPolicy: IfNotPresent
    volumeMounts:
    - mountPath: /client-share
      name: nfs
  volumes:
  - name: nfs
    nfs:
      server: 192.168.32.3  # NFS Server
      path: /share
```

