拉取镜像

```bash
docker pull elasticsearch:8.3.2
docker pull kibana:8.3.2
```

启动容器

```bash
docker run -d --name kibana -p 5602:5601 kibana:8.3.2
docker run -d --name elasticsearch -p 9201:9200 -p 9301:9300 -e "discovery.type=single-node" elasticsearch:8.3.2
```

重置elasticsearch密码

```bash
bin/elasticsearch-reset-password -u elastic -i
```

配置es配置文件`/usr/share/elasticsearch/config/elasticsearch.yml`

```yaml
cluster.name: "docker-cluster"
network.host: 0.0.0.0

xpack.security.enabled: true

xpack.security.enrollment.enabled: true

xpack.security.http.ssl:
  enabled: true
  keystore.path: certs/http.p12

xpack.security.transport.ssl:
  enabled: true
  verification_mode: certificate
  keystore.path: certs/transport.p12
  truststore.path: certs/transport.p12
```

获取kibana连接es的token

```bash
bin/elasticsearch-create-enrollment-token --scope kibana
```

在kibana容器中执行。

```bash
kibana-setup --enrollment-token TOKEN
```

重启es和kibana容器

```bash
docker restart elasticsearch
docker restart kibana
```

