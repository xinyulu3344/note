## 下载二进制包

下载etcd安装包

```bash
wget https://github.com/etcd-io/etcd/releases/download/v3.4.13/etcd-v3.4.13-linux-amd64.tar.gz
```

下载k8s安装包

```bash
wget https://dl.k8s.io/v1.20.15/kubernetes-server-linux-amd64.tar.gz
```

解压

```bash
tar -zxf kubernetes-server-linux-amd64.tar.gz
tar -zxf etcd-v3.4.13-linux-amd64.tar.gz 
cp kubernetes/server/bin/kube{let,ctl,-apiserver,-controller-manager,-scheduler,-proxy} /usr/local/bin/
cp etcd-v3.4.13-linux-amd64/etcd{,ctl} /usr/local/bin/
```



## 生成证书

下载生成证书的工具

```bash
wget https://pkg.cfssl.org/R1.2/cfssl_linux-amd64 -O cfssl
wget https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64 -O cfssljson
chmod u+x /usr/local/bin/cfssljson
mv cfssl cfssljson /usr/local/bin
```

创建证书目录

```bash
mkdir -p /etc/etcd/ssl
mkdir -p /etc/kubernetes/pki
```

生成etcd CA证书和CA证书的key

```bash
cfssl gencert -initca etcd-ca-csr.json | cfssljson -bare /etc/etcd/ssl/etcd-ca
```

etcd-ca-csr.json

```json
{
  "CN": "etcd",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "Beijing",
      "L": "Beijing",
      "O": "etcd",
      "OU": "Etcd Security"
    }
  ],
  "ca": {
    "expiry": "876000h"
  }
}
```

```bash
cfssl gencert \
  -ca=/etc/etcd/ssl/etcd-ca.pem \
  -ca-key=/etc/etcd/ssl/etcd-ca-key.pem \
  -config=ca-config.json \
  -hostname=127.0.0.1,k8s-master01,192.168.25.3 \
  -profile=kubernetes \
  etcd-csr.json | cfssljson - bare /etc/etcd/ssl/etcd
```



k8s-master01节点生成etcd证书

```bash
```

