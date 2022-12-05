## 修改docker默认数据目录

修改`/etc/docker/daemon.json`配置文件

```json
{
    "data-root":"/mnt/docker",
    "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"],
    "mtu": 1450
}
```

关闭容器和docker服务

```bash
docker stop xxx
systemctl stop docker
```

拷贝`/var/lib/docker`到指定数据目录下。注意一定要加`-a`选项，将文件权限也拷贝过来。否则可能因为权限问题，容器无法创建、读写文件导致无法正常启动。

```bash
cp -a /var/lib/docker /mnt/
```

启动docker服务和容器

```bash
systemctl daemon-reload
systemctl start docker
docker start xxx
```

