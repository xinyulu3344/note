## rgw

### 官方文档

https://docs.ceph.com/en/reef/cephadm/services/rgw/#high-availability-service-for-rgw



### 命令行方式部署Ceph 对象网关

**方式一**

```bash
# 创建 realm
radosgw-admin realm create --rgw-realm=<realm-name>
radosgw-admin realm list

# 创建zone group
radosgw-admin zonegroup create --rgw-zonegroup=<zonegroup-name> --master
radosgw-admin zonegroup list

# 创建zone
radosgw-admin zone create --rgw-zonegroup=<zonegroup-name> --rgw-zone=<zone-name> --master
radosgw-admin zone list

# 提交更改
radosgw-admin period update --rgw-realm=<realm-name> --commit

# 部署rgw
ceph orch apply rgw <name> [--realm=<realm-name>] [--zone=<zone-name>] --placement="<num-daemons> [<host1> <host2>...]"
```

**方式二**

```bash
# 将打上label的host作为部署rgw的机器，每台机器部署两个rgw，端口使用8000 8001
ceph orch host label add <hostname> <label-name>
ceph orch host label add <hostname> <label-name>
ceph orch apply rgw foo '--placement=label:rgw count-per-host:2' --port=8000
```

**方式三**

```bash
# 为单个集群部署部署两个 Ceph 对象网关守护进程
ceph orch apply rgw <name>
```

### 查看

```bash
ceph orch ps --daemon_type rgw
```

```bash
ceph orch ls
```

### 使用服务规格部署 Ceph 对象网关

```bash
ceph orch apply -i myrgw.yaml
```

```yaml
service_type: rgw
service_id: foo
placement:
  label: rgw
  count_per_host: 2
networks:
- 192.168.142.0/24
spec:
  rgw_realm: myrealm
  rgw_zone: myzone
  rgw_zonegroup: myzg
  # beast/civetweb
  rgw_frontend_type: "beast"
  rgw_frontend_port: 5000
  rgw_frontend_extra_args:
  - "tcp_nodelay=1"
  - "max_header_size=65536"
  rgw_frontend_ssl_certificate: |
    -----BEGIN PRIVATE KEY-----
    V2VyIGRhcyBsaWVzdCBpc3QgZG9vZi4gTG9yZW0gaXBzdW0gZG9sb3Igc2l0IGFt
    ZXQsIGNvbnNldGV0dXIgc2FkaXBzY2luZyBlbGl0ciwgc2VkIGRpYW0gbm9udW15
    IGVpcm1vZCB0ZW1wb3IgaW52aWR1bnQgdXQgbGFib3JlIGV0IGRvbG9yZSBtYWdu
    YSBhbGlxdXlhbSBlcmF0LCBzZWQgZGlhbSB2b2x1cHR1YS4gQXQgdmVybyBlb3Mg
    ZXQgYWNjdXNhbSBldCBqdXN0byBkdW8=
    -----END PRIVATE KEY-----
    -----BEGIN CERTIFICATE-----
    V2VyIGRhcyBsaWVzdCBpc3QgZG9vZi4gTG9yZW0gaXBzdW0gZG9sb3Igc2l0IGFt
    ZXQsIGNvbnNldGV0dXIgc2FkaXBzY2luZyBlbGl0ciwgc2VkIGRpYW0gbm9udW15
    IGVpcm1vZCB0ZW1wb3IgaW52aWR1bnQgdXQgbGFib3JlIGV0IGRvbG9yZSBtYWdu
    YSBhbGlxdXlhbSBlcmF0LCBzZWQgZGlhbSB2b2x1cHR1YS4gQXQgdmVybyBlb3Mg
    ZXQgYWNjdXNhbSBldCBqdXN0byBkdW8=
    -----END CERTIFICATE-----
  ssl: true
```

```bash
service_type: rgw
service_id: REALM_NAME.ZONE_NAME
placement:
  hosts:
  - HOST_NAME_1
  - HOST_NAME_2
  count_per_host: NUMBER_OF_DAEMONS
spec:
  rgw_realm: REALM_NAME
  rgw_zone: ZONE_NAME
  rgw_zonegroup: ZONE_GROUP_NAME
  rgw_frontend_port: FRONT_END_PORT
networks:
  -  NETWORK_CIDR
```

### 创建rgw用户

```bash
radosgw-admin user create --uid admin --display-name "admin user"
```

### 删除rgw

```bash
radosgw-admin user rm --uid <>
ceph orch rm <rgw-name>
radosgw-admin zone delete --rgw-zonegroup=<zonegroup-name> --rgw-zone=<zone-name>
radosgw-admin zonegroup delete --rgw-zonegroup=<zonegroup-name>
radosgw-admin realm rm --rgw-realm=<realm-name>
```

