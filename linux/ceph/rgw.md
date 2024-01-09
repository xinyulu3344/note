## rgw

https://docs.ceph.com/en/reef/cephadm/services/rgw/#high-availability-service-for-rgw

```bash
radosgw-admin realm create --rgw-realm=<realm-name>
radosgw-admin zonegroup create --rgw-zonegroup=<zonegroup-name> --master
radosgw-admin zone create --rgw-zonegroup=<zonegroup-name> --rgw-zone=<zone-name> --master
radosgw-admin period update --rgw-realm=<realm-name> --commit
```

```bash
ceph orch apply rgw <name> [--realm=<realm-name>] [--zone=<zone-name>] --placement="<num-daemons> [<host1> ...]"
ceph orch apply rgw east --realm=myorg --zonegroup=us-east-zg-1 --zone=us-east-1 --placement="2 myhost1 myhost2"
```

```bash
# 将打上rgw label的host作为部署rgw的机器，每台机器部署两个rgw，端口使用8000 8001
ceph orch host label add gwhost1 rgw  # the 'rgw' label can be anything
ceph orch host label add gwhost2 rgw
ceph orch apply rgw foo '--placement=label:rgw count-per-host:2' --port=8000
```

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
ceph orch ps --daemon_type rgw
```



```bash
radosgw-admin user create --uid admin --display-name "admin user"
```

