## ceph.conf配置管理

/etc/ceph/ceph.conf 

```ini
# minimal ceph.conf for b4f8caae-a7b4-11ee-b259-fa163efc51c7
[global]
fsid = b4f8caae-a7b4-11ee-b259-fa163efc51c7
cluster network = 192.168.30.0/24
public network = 192.168.30.0/24
mon_host = [v2:192.168.30.8:3300/0,v1:192.168.30.8:6789/0] [v2:192.168.30.9:3300/0,v1:192.168.30.9:6789/0] [v2:192.168.30.10:3300/0,v1:192.168.30.10:6789/0]
mon initial members = node1,node2,node3
mon_clock_drift_allowed = 0.5
mon_osd_allow_primary_affinity = 1
mon_pg_warn_max_object_skew = 0
mon_pg_warn_max_per_osd = 0
mon_pg_warn_min_per_osd = 0
mon_allow_pool_delete = true
# 最小副本数
osd_pool_default_min_size = 1
# 默认副本数
osd_pool_default_size = 2


[mon]

[osd]
osd journal size = 1024
osd mkfs options xfs = -f -i size=2048
osd mkfs type = xfs
osd mount options xfs = noatime,largeio,inode64,swalloc

[mgr]

[mds]

[client]
# 支持快照
rbd_default_features = 1
```

### 临时修改

```bash
ceph tell <type.id> config set <option> <value>
```

### 永久修改

```bash
ceph config set <type> <option> <value>
```

### 查看运行时配置

```bash
ceph config show <type.id> [<option>]
ceph config show-with-defaults <type.id> [<option>]
```

### 查看永久配置

```bash
ceph config get <type> [<option>]
```

