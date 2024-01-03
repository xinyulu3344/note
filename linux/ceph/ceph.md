# ceph

## ceph的组件

| 组件      | 说明                                                         |
| --------- | ------------------------------------------------------------ |
| Monitor   | 核心组件，守护进程（ceph-mon）。维护集群状态的映射，包括监视器映射、管理器映射、OSD映射、MDS映射和CRUSH映射。这些映射是ceph守护进程相互协调所需的关键集群状态。监视器还负责管理守护进程和客户端之间的身份验证。通常至少需要三个监视器才能实现冗余和高可用性。基于paxos协议实现节点间的信息同步。 |
| Managers  | Ceph管理器（守护进程ceph-mgr）负责跟踪运行时指标和ceph集群的当前状态。包括存储利用率、当前性能指标和系统负载。ceph管理器守护进程还托管基于Python的模块来管理和公开Ceph集群信息，包括基于Web的Ceph仪表板和REST API。高可用性通常至少需要两个管理器。基于raft协议实现节点间的信息同步 |
| Ceph OSDs | Ceph OSD（Object Storage Daemon，守护进程ceph-osd）存储数据，处理数据复制、恢复、重新平衡，并通过检查其它Ceph OSD守护进程的心跳来向Ceph监视器和管理器提供一些监控信息。通常至少需要3个Ceph OSD来实现冗余和高可用。本质上osd就是一个个host主机上的存储磁盘。 |
| MDSs      | Ceph元数据服务器（Metadata Server，守护进程ceph-mds）代表Ceph文件系统存储元数据。Ceph元数据服务器允许POSIX文件系统用户执行基本命令（如ls、find等），而不会给Ceph存储集群带来巨大负担。 |

## ceph部署完验证

### ceph集群状态检查

```bash
ceph -s
ceph status
ceph health
```

### osd查看

```bash
ceph osd status
ceph osd stat
ceph osd tree
```



### mon查看

```bash
ceph mon stat
ceph mon versions
ceph osd dump
ceph pg dump
ceph mon dump
ceph mds dump
```

### mds查看

```bash
ceph mds stat
```



