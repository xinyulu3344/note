## rbd

### 挂盘流程

1. 创建存储池

   ```bash
   ceph osd pool create pool-rbd 32 32
   ```

2. 将池应用到rbd

   ```bash
   rbd pool init pool-rbd
   ```

3. 创建image

   ```bash
   rbd create pool-rbd/test1 --size 2GB
   
   # list指定pool下的镜像
   rbd ls -p pool-rbd
   # 查看镜像信息
   rbd info pool-rbd/test1
   ```

4. 客户端安装ceph-common

   ```bash
   yum install ceph-common
   ```

5. 给客户端授权

   ```bash
   ceph config generate-minimal-conf
   ceph auth get-or-create client.rbd mon 'profile rbd' osd 'profile rbd pool=pool-rbd' -o /etc/ceph/ceph.client.rbd.keyring
   ```

6. 客户端映射镜像

   ```bash
   rbd map -p pool-rbd --image test1
   ```

7. 格式化挂载

   ```bash
   mkfs.xfs /dev/rbd0
   mount /dev/rbd0 /mnt
   ```

   

### 实现永久挂载

1. 编辑`/etc/ceph/rbdmap`

   ```bash
   # RbdDevice             Parameters
   #poolname/imagename     id=client,keyring=/etc/ceph/ceph.client.keyring
   pool-rbd/test1  id=admin,keyring=/etc/ceph/ceph.client.admin.keyring
   ```

2. 启动`rbdmap.service`

   ```bash
   systemctl enable --now rbdmap.service
   ```

3. 编辑`/etc/fstab`

   ```bash
   UUID=5408afce-b499-4534-baec-8dca45f39431 /mnt                    xfs     defaults,_netdev 0 0
   ```

### 调整镜像size

```bash
rbd resize --size 2048 <pool-name>/<image-name>
rbd resize --size 2048 <pool-name>/<image-name> --allow-shrink
```

### 删除image

```bash
rbd trash mv <pool-name/image-name>
rbd trash rm <pool-name/image-id>
```

```bash
rbd rm <pool-name/image-name>
```

### 恢复image

```bash
rbd trash restore <pool-name>/<image-id>
rbd trash restore <pool-name>/<image-id> --image <new-image-name>
```

### 快照snapshots

#### 功能启动

功能类型：

| 功能类型       | 作用                   |
| -------------- | ---------------------- |
| layering       | 用于启用克隆的分层支持 |
| striping       |                        |
| exclusive-lock |                        |
| object-map     |                        |
| fast-diff      |                        |
| deep-flatten   |                        |
| journaling     |                        |
| data-pool      |                        |

启用功能

```bash
rbd feature enable <pool-name>/<image-name> <features> <features>...
```

禁用功能

```bash
rbd feature disable <pool-name>/<image-name> <features> <features>...
```

#### 快照操作

https://docs.ceph.com/en/reef/rbd/rbd-snapshot/

创建快照

```bash
rbd snap create {pool-name}/{image-name}@{snap-name}
```

列出快照

```bash
rbd snap ls {pool-name}/{image-name}
```

```bash
# rbd snap ls pool-rbd/test1
SNAPID  NAME        SIZE   PROTECTED  TIMESTAMP               
     4  snap-test1  5 GiB             Sun Jan  7 23:41:03 2024
```

限制快照数量

移除限制

还原快照

保护快照

取消保护

删除快照
