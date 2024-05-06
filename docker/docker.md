## 安装

https://docs.docker.com/engine/install/centos/

## 常用命令

### 镜像管理

```bash
docker image tag 86f313609493 chenchen/busybox:v0.1-1
docker tag 86f313609493 chenchen/busybox:v0.1-1
docker image rm chenchen/busybox:latest
docker image rm -f 86f313609493
docker image save -o myimages.gz  busybox:latest nginx:latest
docker load -i myimages.gz
```

### 容器管理

```bash
docker ps
docker ps -a

docker run --name b1 -it busybox:latest
docker run --name web1 -d nginx:latest
docker run --name web1 -d -p 8080:80 nginx:latest
docker run --name t1 -it --network none --rm busybox:latest
docker run --name b2 --network container:b1 -it --rm busybox
docker run -it --name b1 -v /data busybox

docker start b1
docker stop b1
docker kill b1
docker rm b1
docker commit -p b1 REPOSITORY[:TAG]
```



### 网络管理

```bash
docker network create -d bridge --subnet 172.16.0.0/16 --gateway 172.16.0.1 mybr0
```



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
docker start xxrrrx
```

## docker-compose

### 安装

https://docs.docker.com/compose/install/

### 配置文件

docker-compose.yml

```yaml
version: "2.1"
name: nginx
services:
  nginx-demo:
    image: "nginx"
    container_name: "nginx_compose"
    restart: "always"
    networks:
    - nginx_net
    blkio_config:
      weight: 300
      weight_device:
      - path: /dev/vdb
        weight: 400
      device_read_bps:
      - path: /dev/vdb
        rate: "12mb"
      device_read_iops:
      - path: /dev/vdb
        rate: 1200
      device_write_bps:
      - path: /dev/vdb
        rate: "1024k"
      device_write_iops:
      - path: /dev/vdb
        rate: 1200
    volumes:
    - /opt/data:/usr/share/nginx/html:rw
    - volume_nginx:/etc/nginx
    environment:
      APP_ENV: dev
    dns:
    - 8.8.8.8
    - 114.114.114.114
    depends_on:
    - db
    - redis
    ports:
    - 80:80
networks:
  net_nginx:
    driver: "bridge"
    ipam:
      driver: default
      config:
      - subnet: 10.0.24.0/24
        gateway: 10.0.24.1
volumes:
  volume_nginx:
  
config:
secrets:
```

### 常用命令

```bash
]# docker compose 

Usage:  docker compose [OPTIONS] COMMAND

Define and run multi-container applications with Docker

Options:
      --all-resources              Include all resources, even those not used by services
      --ansi string                Control when to print ANSI control characters ("never"|"always"|"auto") (default "auto")
      --compatibility              Run compose in backward compatibility mode
      --dry-run                    Execute command in dry run mode
      --env-file stringArray       Specify an alternate environment file
  -f, --file stringArray           Compose configuration files
      --parallel int               Control max parallelism, -1 for unlimited (default -1)
      --profile stringArray        Specify a profile to enable
      --progress string            Set type of progress output (auto, tty, plain, quiet) (default "auto")
      --project-directory string   Specify an alternate working directory
                                   (default: the path of the, first specified, Compose file)
  -p, --project-name string        Project name

Commands:
  attach      Attach local standard input, output, and error streams to a service's running container
  build       Build or rebuild services
  config      Parse, resolve and render compose file in canonical format
  cp          Copy files/folders between a service container and the local filesystem
  create      Creates containers for a service
  down        Stop and remove containers, networks
  events      Receive real time events from containers
  exec        Execute a command in a running container
  images      List images used by the created containers
  kill        Force stop service containers
  logs        View output from containers
  ls          List running compose projects
  pause       Pause services
  port        Print the public port for a port binding
  ps          List containers
  pull        Pull service images
  push        Push service images
  restart     Restart service containers
  rm          Removes stopped service containers
  run         Run a one-off command on a service
  scale       Scale services 
  start       Start services
  stats       Display a live stream of container(s) resource usage statistics
  stop        Stop services
  top         Display the running processes
  unpause     Unpause services
  up          Create and start containers
  version     Show the Docker Compose version information
  wait        Block until the first service container stops
  watch       Watch build context for service and rebuild/refresh containers when files are updated

Run 'docker compose COMMAND --help' for more information on a command.
```



```bash
docker compose create [OPTIONS] [SERVICE...]
docker compose up [OPTIONS] [SERVICE...]
docker compose scale [SERVICE=REPLICAS...]
docker compose logs [OPTIONS] [SERVICE...]
```

