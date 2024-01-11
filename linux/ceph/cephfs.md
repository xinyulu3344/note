## cephfs

使用CephFS需要部署一个或多个MDS守护进程。`ceph fs volume`会自动创建mds

```bash
# placement指定mds位置
ceph fs volume create <fs_name> --placement="<placement spec>"
```

创建之后更新部署位置

```bash
ceph orch apply mds <fs_name> 'mds-[012]'
```

mds的yaml文件

```yaml
service_type: mds
service_id: fs_name
placement:
  count: 3
  label: mds
```

```bash
ceph orch apply -i <mds.yaml>
```

