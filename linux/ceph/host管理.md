## ceph的host管理

### 查看host信息

命令格式：

```bash
# --host-pattern、--label、--host-status用于过滤
# --host-status: offline | maintenance
ceph orch host ls [--format yaml] [--host-pattern <name>] [--label <label>] [--host-status <status>] [--detail]
```

示例: 

```bash
# ceph orch host ls
HOST   ADDR           LABELS  STATUS
node1  192.168.30.8   _admin
node2  192.168.30.9   _admin
node3  192.168.30.10  _admin
node4  192.168.30.11
4 hosts in cluster
```

```bash
# ceph orch host ls --detail
HOST   ADDR           LABELS  STATUS  VENDOR/MODEL                            CPU    RAM    HDD        SSD  NIC
node1  192.168.30.8   _admin          JD Virtual Machine (JCloud Iaas Jvirt)  1C/1T  4 GiB  1/42.9GB   -    1
node2  192.168.30.9   _admin          JD Virtual Machine (JCloud Iaas Jvirt)  1C/2T  4 GiB  4/204.0GB  -    1
node3  192.168.30.10  _admin          JD Virtual Machine (JCloud Iaas Jvirt)  1C/2T  4 GiB  4/204.0GB  -    1
node4  192.168.30.11                  JD Virtual Machine (JCloud Iaas Jvirt)  1C/2T  4 GiB  4/204.0GB  -    1
4 hosts in cluster
```

### 添加hosts

节点必须安装如下依赖：

- Python 3
- Systemd
- Podman or Docker for running containers
- Time synchronization (such as chrony or NTP)
- LVM2 for provisioning storage devices

添加节点：

1. 在新节点的root用户下，安装集群的公钥

   ```bash
   ssh-copy-id -f -i /etc/ceph/ceph.pub root@<new-host>
   ```

2. 添加节点

   ```bash
   ceph orch host add <newhost> [<ip>] [<label1> ...]
   ```

   例如：

   ```bash
   ceph orch host add node2 192.168.30.9 --labels _admin
   ceph orch host add node4 192.168.30.11
   ```

### 移除hosts

这台host上所有的守护进程都被移除后，host才可以被安全的从集群中移除

驱逐所有守护进程：

```bash
# 执行后_no_schedule和_no_conf_keyring标签会被应用到该host上
ceph orch host drain <host>

# 执行后只有_no_schedule标签会被应用到该host
ceph orch host drain <host> --keep-conf-keyring

# 驱逐并破坏osd的设备
ceph orch host drain <host> --zap-osd-devices
```

系统会定时移除host上的所有osd。使用命令查询OSD移除的执行进度：

```bash
ceph orch osd rm status
```

使用以下命令确定host上是否还有守护进程：

```bash
ceph orch ps <host>
```

将host上的所有守护进程移除后，再将host从集群中移除。

```bash
ceph orch host rm <host>
```

即使host离线且无法恢复，也可以通过以下命令将其从集群中移除（可能导致数据丢失！！）：

```bash
ceph orch host rm <host> --offline --force
```

### 标签

添加节点时指定标签：

```bash
ceph orch host add <newhost> --labels=my_label1
ceph orch host add <newhost> --labels=my_label1,my_label2
```

为已经存在的host添加标签：

```bash
ceph orch host label add <host> my_label
```

移除标签：

```bash
ceph orch host label rm <host> my_label
```

特殊含义的标签：

| 标签                | 含义                                                         |
| ------------------- | ------------------------------------------------------------ |
| _no_schedule        | 不在该host上调度或部署守护进程。这个标签防止cephadm在这个主机上部署守护进程。如果将它添加到已经包含Ceph守护进程的现有主机上，则会导致cephadm将这些守护进程移动到其他地方(osd除外，它们不会自动删除)。 |
| _no_conf_keyring    | 不在该host上部署配置文件和密钥。这个标签实际上与_no_schedule相同，但它不用于守护进程，而是用于由cephadm管理的客户端keyring和ceph配置文件 |
| _no_autotune_memory | 在此host上不自动调优内存。即使为该host上的一个或多个守护进程启用了osd_memory_target_autotune或类似选项，这个标签也会阻止对守护进程内存进行调优。 |
| _admin              | 分发client.admin和ceph.conf到这个节点。添加这个标签将导致cephadm将ceph.conf和client.admin到/etc/ceph目录下。从16.2.10和17.2.1开始，除了放到/etc/ceph目录，还会放到/var/lib/ceph/<fsid>/config目录下。 |

### 维护模式

设置主机进入或退出维护模式（停止主机上的所有Ceph守护进程）：

```bash
# --force: 标志允许用户绕过警告，但不允许告警
# --yes-i-really-mean-it: 绕过所有的安全检查并强制host进入维护模式，可能导致数据丢失！！
ceph orch host maintenance enter <hostname> [--force] [--yes-i-really-mean-it]
ceph orch host maintenance exit <hostnam
```

### 重新扫描主机devices

```bash
ceph orch host rescan <hostname> [--with-summary]
```

示例：

```bash
# 命令
ceph orch host rescan node4
# 输出
Ok

# 命令
ceph orch host rescan node4 --with-summary
# 输出
Ok. 2 adapters detected: 2 rescanned, 0 skipped, 0 failed (0.31s)
```

### 通过yaml文件添加节点

```bash
ceph orch apply -i <xxx.yaml>
```

```yaml
service_type: host
hostname: node2
addr: 192.168.30.9
labels:
- example1
- example2

---

service_type: host
hostname: node3
addr: 192.168.30.10
labels:
- grafana

---

service_type: host
hostname: node4
addr: 192.168.30.11
```

### 设置初始CRUSH location

### 将host从CRUSH MAP中移除

### OS Turning Profiles

编辑yaml文件：

```yaml
profile_name: 23-mon-host-profile
placement:
  hosts:
    - node1
    - node2
settings:
  fs.file-max: 1000000
  vm.swappiness: '13'
```

执行命令应用配置：

```bash
# --no-overwrite: 应用profile的时候是幂等的，且不会覆盖已有的同名配置
ceph orch tuned-profile apply -i <tuned-profile-file-name>
```

这个profile被写入到hosts指定的host的`/etc/sysctl.d/`目录下，文件名为`<profile_name>-cephadm-tuned-profile.conf`，并执行`sysctl --system`命令。

查看profiles：

```bash
ceph orch tuned-profile ls <--format yaml>
```

删除profiles：

```bash
ceph orch tuned-profile rm <profile-name>
```

当删除profiles之后，cephadm会清理`/etc/sysctl.d/`目录下的相关文件

修改profiles：

```bash
ceph orch tuned-profile add-setting <profile-name> <setting-name> <value>
ceph orch tuned-profile rm-setting <profile-name> <setting-name>
```

### ssh配置

#### 生成新的ssh密钥：

```bash
ceph cephadm generate-key
```

#### 获取公钥：

```bash
ceph cephadm get-pub-key
```

#### 删除当前存储的ssh key：

```bash
ceph cephadm clear-key
```

#### 导入自己的ssh key

```bash
ceph config-key set mgr/cephadm/ssh_identity_key -i <key>
ceph config-key set mgr/cephadm/ssh_identity_pub -i <pub>
```

重启mgr来加载配置

```bash
ceph mgr fail
```

#### 配置不同的SSH用户

Cephadm必须能够作为具有足够特权的用户登录到所有Ceph集群节点，以便下载容器映像、启动容器和执行命令，而不需要提示输入密码。如果您不想使用"root"用户(cephadm中的默认选项)，则必须向cephadm提供将用于执行所有cephadm操作的用户名。使用命令:

```bash
ceph cephadm set-user <user>
```

在运行此操作之前，需要将集群SSH密钥添加到该用户的authorized_keys文件中，并且非root用户必须具有无密码的sudo访问权限。

#### 自定义ssh配置

cephadm生成一个适当的ssh_config文件，用于连接到远程主机。有两种方法可以制此配置：

1. 导入ssh_config配置文件

   ```bash
   ceph cephadm set-ssh-config -i <ssh_config_file>
   
   # 移除自定义ssh_config，使用默认配置
   ceph cephadm clear-ssh-config
   ```

2. 配置ssh_config配置文件路径（不推荐）

   ```bash
   ceph config set mgr mgr/cephadm/ssh_config_file <path>
   ```