## rbd

```bash
ceph osd pool create pool-rbd 32 32
rbd pool init pool-rbd
rbd create pool-rbd/test1 --size 2GB
rbd ls -p pool-rbd
rbd info pool-rbd/test1
rbd map -p pool-rbd --image test1
```

