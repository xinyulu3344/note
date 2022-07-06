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

### 生成证书所需配置文件准备

参考 https://github.com/dotbalo/k8s-ha-install 上的配置

**etcd-ca-csr.json**

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

**etcd-csr.json**

```js
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
  ]
}
```

**ca-config.json**

```json
{
  "signing": {
    "default": {
      "expiry": "876000h"
    },
    "profiles": {
      "kubernetes": {
        "usages": [
            "signing",
            "key encipherment",
            "server auth",
            "client auth"
        ],
        "expiry": "876000h"
      }
    }
  }
}
```

**ca-csr.json**

```json
{
  "CN": "kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "Beijing",
      "L": "Beijing",
      "O": "Kubernetes",
      "OU": "Kubernetes-manual"
    }
  ],
  "ca": {
    "expiry": "876000h"
  }
}
```



**apiserver-csr.json**

```json
{
  "CN": "kube-apiserver",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "Beijing",
      "L": "Beijing",
      "O": "Kubernetes",
      "OU": "Kubernetes-manual"
    }
  ]
}
```

**front-proxy-ca-csr.json**

```json
{
  "CN": "kubernetes",
  "key": {
     "algo": "rsa",
     "size": 2048
  }
}
```

**front-proxy-client-csr.json**

```json
{
  "CN": "front-proxy-client",
  "key": {
     "algo": "rsa",
     "size": 2048
  }
}
```

**manager-csr.json**

```json
{
  "CN": "system:kube-controller-manager",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "Beijing",
      "L": "Beijing",
      "O": "system:kube-controller-manager",
      "OU": "Kubernetes-manual"
    }
  ]
}
```

**scheduler-csr.json**

```json
{
  "CN": "system:kube-scheduler",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "Beijing",
      "L": "Beijing",
      "O": "system:kube-scheduler",
      "OU": "Kubernetes-manual"
    }
  ]
}
```

**admin-csr.json**

```json
{
  "CN": "admin",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "Beijing",
      "L": "Beijing",
      "O": "system:masters",
      "OU": "Kubernetes-manual"
    }
  ]
}
```



### 生成证书过程

下载生成证书的工具

```bash
wget https://pkg.cfssl.org/R1.2/cfssl_linux-amd64 -O cfssl
wget https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64 -O cfssljson
mv cfssl cfssljson /usr/local/bin
chmod u+x /usr/local/bin/cfssljson
```

创建证书目录

```bash
mkdir -p /etc/etcd/ssl
mkdir -p /etc/kubernetes/pki
```

#### 生成etcd证书

k8s-master01节点生成etcd CA证书和CA证书的key

```bash
cfssl gencert \
  -initca etcd-ca-csr.json | cfssljson -bare /etc/etcd/ssl/etcd-ca
# 生成etcd-ca.csr  etcd-ca-key.pem  etcd-ca.pem
```

k8s-master01节点生成etcd证书

```bash
cfssl gencert \
  -ca=/etc/etcd/ssl/etcd-ca.pem \
  -ca-key=/etc/etcd/ssl/etcd-ca-key.pem \
  -config=ca-config.json \
  -hostname=127.0.0.1,k8s-master01,192.168.25.3 \
  -profile=kubernetes \
  etcd-csr.json | cfssljson -bare /etc/etcd/ssl/etcd
# 生成etcd.csr  etcd-key.pem  etcd.pem
```

将k8s-master01节点生成如下4个文件，分发到其它master节点上

```bash
/etc/etcd/ssl/etcd-ca-key.pem 
/etc/etcd/ssl/etcd-ca.pem
/etc/etcd/ssl/etcd-key.pem
/etc/etcd/ssl/etcd.pem
```

#### 生成apiserver证书

k8s-master01节点生成kubernetes的ca证书和ca的key

```bash
cfssl gencert \
  -initca ca-csr.json \
  ca-csr.json | cfssljson -bare /etc/kubernetes/pki/ca
# 生成ca.csr  ca-key.pem  ca.pem三个文件
```

k8s-master01节点生成kubernetes的证书

```bash
# 10.96.0.1是k8s service网段的第一个IP
cfssl gencert \
  -ca=/etc/kubernetes/pki/ca.pem \
  -ca-key=/etc/kubernetes/pki/ca-key.pem \
  -config=ca-config.json \
  -hostname=10.96.0.1,127.0.0.1,kubernetes,kubernetes.default,kubernetes.default.svc,kubernetes.default.svc.cluster,kubernetes.default.svc.cluster.local,192.168.25.3 \
  -profile=kubernetes \
  apiserver-csr.json | cfssljson -bare /etc/kubernetes/pki/apiserver

# 生成apiserver.csr  apiserver-key.pem  apiserver.pem三个文件
```

生成apiserver的聚合证书

```bash
cfssl gencert \
  -initca front-proxy-ca-csr.json | cfssljson -bare /etc/kubernetes/pki/front-proxy-ca
# 生成front-proxy-ca.csr front-proxy-ca-key.pem front-proxy-ca.pem
```

```bash
cfssl gencert \
  -ca=/etc/kubernetes/pki/front-proxy-ca.pem \
  -ca-key=/etc/kubernetes/pki/front-proxy-ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  front-proxy-client-csr.json | cfssljson -bare /etc/kubernetes/pki/front-proxy-client
# 生成front-proxy-client.csr front-proxy-client-key.pem front-proxy-client.pem
```

#### 生成controller-manager证书

```bash
cfssl gencert \
  -ca=/etc/kubernetes/pki/ca.pem \
  -ca-key=/etc/kubernetes/pki/ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  manager-csr.json | cfssljson -bare /etc/kubernetes/pki/controller-manager
# 生成controller-manager.csr controller-manager-key.pem controller-manager.pem
```

set-cluster: 设置一个集群项

```bash
kubectl config set-cluster kubernetes \
  --certificate-authority=/etc/kubernetes/pki/ca.pem \
  --embed-certs=true \
  --server=https://192.168.25.3:6443 \
  --kubeconfig=/etc/kubernetes/controller-manager.kubeconfig
```

设置一个用户项

```bash
kubectl config set-credentials system:kube-controller-manager \
  --client-certificate=/etc/kubernetes/pki/controller-manager.pem \
  --client-key=/etc/kubernetes/pki/controller-manager-key.pem \
  --embed-certs=true \
  --kubeconfig=/etc/kubernetes/controller-manager.kubeconfig
```

设置一个上下文环境

```bash
kubectl config set-context system:kube-controller-manager@kubernetes \
  --cluster=kubernetes \
  --user=system:kube-controller-manager \
  --kubeconfig=/etc/kubernetes/controller-manager.kubeconfig
```

使用某个环境当做默认环境

```bash
kubectl config use-context system:kube-controller-manager@kubernetes \
  --kubeconfig=/etc/kubernetes/controller-manager.kubeconfig
```

#### 生成scheduler证书

过程和controller-manager一样

```bash
cfssl gencert \
  -ca=/etc/kubernetes/pki/ca.pem \
  -ca-key=/etc/kubernetes/pki/ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  scheduler-csr.json | cfssljson -bare /etc/kubernetes/pki/scheduler
# 生成scheduler.csr scheduler-key.pem scheduler.pem
```

```bash
kubectl config set-cluster kubernetes \
  --certificate-authority=/etc/kubernetes/pki/ca.pem \
  --embed-certs=true \
  --server=https://192.168.25.3:6443 \
  --kubeconfig=/etc/kubernetes/scheduler.kubeconfig
```

```bash
kubectl config set-credentials system:kube-scheduler \
  --client-certificate=/etc/kubernetes/pki/scheduler.pem \
  --client-key=/etc/kubernetes/pki/scheduler-key.pem \
  --embed-certs=true \
  --kubeconfig=/etc/kubernetes/scheduler.kubeconfig
```

```bash
kubectl config set-context system:kube-scheduler@kubernetes \
  --cluster=kubernetes \
  --user=system:kube-scheduler \
  --kubeconfig=/etc/kubernetes/scheduler.kubeconfig
```

```bash
kubectl config use-context system:kube-scheduler@kubernetes \
  --kubeconfig=/etc/kubernetes/scheduler.kubeconfig
```

#### 生成admin证书

过程和上面一样，可以将该文件拷贝到~/.kube/config，后续可以用kubectl管理集群

```bash
cfssl gencert \
  -ca=/etc/kubernetes/pki/ca.pem \
  -ca-key=/etc/kubernetes/pki/ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  admin-csr.json | cfssljson -bare /etc/kubernetes/pki/admin
```

```bash
kubectl config set-cluster kubernetes \
  --certificate-authority=/etc/kubernetes/pki/ca.pem \
  --embed-certs=true \
  --server=https://192.168.25.3:6443 \
  --kubeconfig=/etc/kubernetes/admin.kubeconfig
```

```bash
kubectl config set-credentials kube-admin \
  --client-certificate=/etc/kubernetes/pki/admin.pem \
  --client-key=/etc/kubernetes/pki/admin-key.pem \
  --embed-certs=true \
  --kubeconfig=/etc/kubernetes/admin.kubeconfig
```

```bash
kubectl config set-context kube-admin@kubernetes \
  --cluster=kubernetes \
  --user=kube-admin \
  --kubeconfig=/etc/kubernetes/admin.kubeconfig
```

```bash
kubectl config use-context kube-admin@kubernetes \
  --kubeconfig=/etc/kubernetes/admin.kubeconfig
```

#### ServiceAccount Key

用于生成ServiceAccount Secret中的token

```bash
openssl genrsa -out /etc/kubernetes/pki/sa.key 2048
openssl rsa -in /etc/kubernetes/pki/sa.key -pubout -out /etc/kubernetes/pki/sa.pub
```

将`/etc/kubernetes/pki/`下所有文件和`/etc/kubernetes/`下所有.kubeconfig文件拷贝到其它master节点

## 部署etcd

### 编辑配置文件

官方配置文件模板：https://github.com/etcd-io/etcd/blob/v3.4.13/etcd.conf.yml.sample

**/etc/etcd/etcd.config.yml**

```yaml
name: k8s-master01
data-dir: /var/lib/etcd
wal-dir: /var/lib/etcd/wal
snapshot-count: 5000
heartbeat-interval: 100
election-timeout: 1000
quota-backend-bytes: 0
listen-peer-urls: https://192.168.25.3:2380
listen-client-urls: https://192.168.25.3:2379,https://localhost:2379
max-snapshots: 3
max-wals: 5
cors:
initial-advertise-peer-urls: https://192.168.25.3:2380
advertise-client-urls: https://192.168.25.3:2379
discovery:
discovery-fallback: 'proxy'
discovery-proxy:
discovery-srv:
initial-cluster: 'k8s-master01=https://192.168.25.3:2380'
initial-cluster-token: 'etcd-k8s-cluster'
initial-cluster-state: 'new'
strict-reconfig-check: false
enable-v2: true
enable-pprof: true
proxy: 'off'
proxy-failure-wait: 5000
proxy-refresh-interval: 30000
proxy-dial-timeout: 1000
proxy-write-timeout: 5000
proxy-read-timeout: 0
client-transport-security:
  cert-file: /etc/kubernetes/pki/etcd/etcd.pem
  key-file: /etc/kubernetes/pki/etcd/etcd-key.pem
  client-cert-auth: true
  trusted-ca-file: /etc/kubernetes/pki/etcd/etcd-ca.pem
  auto-tls: true
peer-transport-security:
  cert-file: /etc/kubernetes/pki/etcd/etcd.pem
  key-file: /etc/kubernetes/pki/etcd/etcd-key.pem
  client-cert-auth: true
  trusted-ca-file: /etc/kubernetes/pki/etcd/etcd-ca.pem
  auto-tls: true
logger: zap
log-outputs: [stderr]
force-new-cluster: false
auto-compaction-mode: periodic
auto-compaction-retention: "1"
```

### 创建Service

**/usr/lib/systemd/system/etcd.service**

```
[Unit]
Description=Etcd Service
Documentation=https://coreos.com/etcd/docs/latest/
After=network.target

[Service]
Type=notify
ExecStart=/usr/local/bin/etcd --config-file=/etc/etcd/etcd.config.yml
Restart=on-failure
RestartSec=10
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
Alias=etcd3.service
```

```bash
mkdir /etc/kubernetes/pki/etcd
ln -s /etc/etcd/ssl/* /etc/kubernetes/pki/etcd/
systemctl daemon-reload
systemctl enable -now etcd
```

查看etcd状态

```bash
export ETCDCTL_API=3
etcdctl --endpoints="192.168.25.3:2379" --cacert=/etc/kubernetes/pki/etcd/etcd-ca.pem --cert=/etc/kubernetes/pki/etcd/etcd.pem --key=/etc/kubernetes/pki/etcd/etcd-key.pem endpoint status --write-out=table
```

```
+-------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
|     ENDPOINT      |        ID        | VERSION | DB SIZE | IS LEADER | IS LEARNER | RAFT TERM | RAFT INDEX | RAFT APPLIED INDEX | ERRORS |
+-------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
| 192.168.25.3:2379 | 1687133f89ae3b17 |  3.4.13 |  3.8 MB |      true |      false |         2 |         54 |                 54 |        |
+-------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+

```

## master组件配置

### kube-apiserver

所有master节点创建**/usr/lib/systemd/system/kube-apiserver.service**

```bash
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes
After=network.target

[Service]
ExecStart=/usr/local/bin/kube-apiserver \
    --v=2 \
    --logtostderr=true \
    --allow-privileged=true \
    --bind-address=0.0.0.0 \
    --secure-port=6443 \
    --insecure-port=0 \
    --advertise-address=192.168.25.3 \
    --service-cluster-ip-range=10.96.0.0/12 \
    --service-node-port-range=30000-32767 \
    --etcd-servers=https://192.168.25.3:2379 \
    --etcd-cafile=/etc/etcd/ssl/etcd-ca.pem  \
    --etcd-certfile=/etc/etcd/ssl/etcd.pem  \
    --etcd-keyfile=/etc/etcd/ssl/etcd-key.pem  \
    --client-ca-file=/etc/kubernetes/pki/ca.pem  \
    --tls-cert-file=/etc/kubernetes/pki/apiserver.pem  \
    --tls-private-key-file=/etc/kubernetes/pki/apiserver-key.pem  \
    --kubelet-client-certificate=/etc/kubernetes/pki/apiserver.pem  \
    --kubelet-client-key=/etc/kubernetes/pki/apiserver-key.pem  \
    --service-account-key-file=/etc/kubernetes/pki/sa.pub  \
    --service-account-signing-key-file=/etc/kubernetes/pki/sa.key  \
    --service-account-issuer=https://kubernetes.default.svc.cluster.local \
    --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname  \
    --enable-admission-plugins=NamespaceLifecycle,LimitRanger,ServiceAccount,DefaultStorageClass,DefaultTolerationSeconds,NodeRestriction,ResourceQuota  \
    --authorization-mode=Node,RBAC  \
    --enable-bootstrap-token-auth=true  \
    --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.pem  \
    --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.pem  \
    --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client-key.pem  \
    --requestheader-allowed-names=aggregator  \
    --requestheader-group-headers=X-Remote-Group  \
    --requestheader-extra-headers-prefix=X-Remote-Extra-  \
    --requestheader-username-headers=X-Remote-User
    # --token-auth-file=/etc/kubernetes/token.csv

Restart=on-failure
RestartSec=10s
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```

```bash
systemctl daemon-reload && systemctl enable --now kube-apiserver
```

### ControllerManager

所有Master节点配置/usr/lib/systemd/system/kube-controller-manager.service

注意本文档使用的k8s Pod网段为`172.16.0.0/12`，该网段不能和宿主机的网段、k8s Service网段重复

```bash
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes
After=network.target

[Service]
ExecStart=/usr/local/bin/kube-controller-manager \
    --v=2 \
    --logtostderr=true \
    --address=127.0.0.1 \
    --root-ca-file=/etc/kubernetes/pki/ca.pem \
    --cluster-signing-cert-file=/etc/kubernetes/pki/ca.pem \
    --cluster-signing-key-file=/etc/kubernetes/pki/ca-key.pem \
    --service-account-private-key-file=/etc/kubernetes/pki/sa.key \
    --kubeconfig=/etc/kubernetes/controller-manager.kubeconfig \
    --leader-elect=true \
    --use-service-account-credentials=true \
    --node-monitor-grace-period=40s \
    --node-monitor-period=5s \
    --pod-eviction-timeout=2m0s \
    --controllers=*,bootstrapsigner,tokencleaner \
    --allocate-node-cidrs=true \
    --cluster-cidr=172.16.0.0/12 \
    --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.pem \
    --node-cidr-mask-size=24

Restart=always
RestartSec=10s

[Install]
WantedBy=multi-user.target
```

```bash
systemctl daemon-reload && systemctl enable --now kube-controller-manager.service
```

### Scheduler

所有Master节点配置/usr/lib/systemd/system/kube-scheduler.service

```bash
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes
After=network.target

[Service]
ExecStart=/usr/local/bin/kube-scheduler \
    --v=2 \
    --logtostderr=true \
    --address=127.0.0.1 \
    --leader-elect=true \
    --kubeconfig=/etc/kubernetes/scheduler.kubeconfig

Restart=always
RestartSec=10s

[Install]
WantedBy=multi-user.target
```

```bash
systemctl daemon-reload && systemctl enable --now kube-scheduler.service
```

## TLS Bootstrapping配置

用于自动给node节点上的kubelet颁发证书

### bootstrap的yaml文件

```bash
cat > bootstrap.secret.yaml << EOF
apiVersion: v1
kind: Secret
metadata:
  name: bootstrap-token-c8ad9c
  namespace: kube-system
type: bootstrap.kubernetes.io/token
stringData:
  description: "The default bootstrap token generated by 'kubelet '."
  token-id: c8ad9c
  token-secret: 2e4d610cf3e7426e
  usage-bootstrap-authentication: "true"
  usage-bootstrap-signing: "true"
  auth-extra-groups:  system:bootstrappers:default-node-token,system:bootstrappers:worker,system:bootstrappers:ingress
 
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: kubelet-bootstrap
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:node-bootstrapper
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: system:bootstrappers:default-node-token
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: node-autoapprove-bootstrap
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:certificates.k8s.io:certificatesigningrequests:nodeclient
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: system:bootstrappers:default-node-token
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: node-autoapprove-certificate-rotation
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:certificates.k8s.io:certificatesigningrequests:selfnodeclient
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: system:nodes
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: system:kube-apiserver-to-kubelet
rules:
  - apiGroups:
      - ""
    resources:
      - nodes/proxy
      - nodes/stats
      - nodes/log
      - nodes/spec
      - nodes/metrics
    verbs:
      - "*"
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: system:kube-apiserver
  namespace: ""
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:kube-apiserver-to-kubelet
subjects:
  - apiGroup: rbac.authorization.k8s.io
    kind: User
    name: kube-apiserver
EOF
```

```bash
kubectl apply -f bootstrap.secret.yaml
```

```bash
kubectl config set-cluster kubernetes \
  --certificate-authority=/etc/kubernetes/pki/ca.pem \
  --embed-certs=true \
  --server=https://192.168.25.3:6443 \
  --kubeconfig=/etc/kubernetes/bootstrap-kubelet.kubeconfig
```

```bash
# 此处的--token，要与bootstrap.secret.yaml文件中，bootstrap-token-c8ad9c这个Secret的token-id和token-secret保持一致
kubectl config set-credentials tls-bootstrap-token-user \
  --token=c8ad9c.2e4d610cf3e7426e \
  --kubeconfig=/etc/kubernetes/bootstrap-kubelet.kubeconfig
```

```bash
kubectl config set-context tls-bootstrap-token-user@kubernetes \
  --cluster=kubernetes \
  --user=tls-bootstrap-token-user \
  --kubeconfig=/etc/kubernetes/bootstrap-kubelet.kubeconfig
```

```bash
kubectl config use-context tls-bootstrap-token-user@kubernetes \
  --kubeconfig=/etc/kubernetes/bootstrap-kubelet.kubeconfig
```

## node节点配置

### 复制证书

```bash
scp /etc/etcd/ssl/{etcd-ca.pem,etcd.pem,etcd-key.pem} root@192.168.32.3:/etc/etcd/ssl
scp /etc/kubernetes/pki/{ca.pem,ca-key.pem,front-proxy-ca.pem} root@192.168.32.3:/etc/kubernetes/pki/
scp /etc/kubernetes/bootstrap-kubelet.kubeconfig root@192.168.32.3:/etc/kubernetes
```

### kubelet配置

node节点创建相关目录

```bash
mkdir -p /var/lib/kubelet /var/log/kubernetes /usr/lib/systemd/system/kubelet.service.d /etc/kubernetes/manifests
```

node节点配置kubelet.service

```bash
cat > /usr/lib/systemd/system/kubelet.service << EOF
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/kubernetes/kubernetes
After=docker.service
Requires=docker.service

[Service]
ExecStart=/usr/local/bin/kubelet

Restart=always
StartLimitInterval=0
RestartSec=10s

[Install]
WantedBy=multi-user.target
EOF
```

node节点配置kubelet service的配置文件

```bash
cat > /usr/lib/systemd/system/kubelet.service.d/10-kubelet.conf << EOF
[Service]
Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.kubeconfig --kubeconfig=/etc/kubernetes/kubelet.kubeconfig"

Environment="KUBELET_SYSTEM_ARGS=--network-plugin=cni --cni-conf-dir=/etc/cni/net.d --cni-bin-dir=/opt/cni/bin"

Environment="KUBELET_CONFIG_ARGS=--config=/etc/kubernetes/kubelet-conf.yml --pod-infra-container-image=registry.cn-hangzhou.aliyuncs.com/google_containers/pause-amd64:3.1"
Environment="KUBELET_EXTRA_ARGS=--node-labels=node.kubernetes.io/node='' "

ExecStart=
ExecStart=/usr/local/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_SYSTEM_ARGS $KUBELET_EXTRA_ARGS
EOF
```

创建kubelet的配置文件/etc/kubernetes/kubelet-conf.yml

```bash
cat > /etc/kubernetes/kubelet-conf.yml << EOF
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
address: 0.0.0.0
port: 10250
readOnlyPort: 10255
authentication:
  anonymous:
    enabled: false
  webhook:
    cacheTTL: 2m0s
    enabled: true
  x509:
    clientCAFile: /etc/kubernetes/pki/ca.pem
authorization:
  mode: Webhook
  webhook:
    cacheAuthorizedTTL: 5m0s
    cacheUnauthorizedTTL: 30s
cgroupDriver: systemd
cgroupsPerQOS: true
clusterDNS:
- 10.96.0.10
clusterDomain: cluster.local
containerLogMaxFiles: 5
containerLogMaxSize: 10Mi
contentType: application/vnd.kubernetes.protobuf
cpuCFSQuota: true
cpuManagerPolicy: none
cpuManagerReconcilePeriod: 10s
enableControllerAttachDetach: true
enableDebuggingHandlers: true
enforceNodeAllocatable:
- pods
eventBurst: 10
eventRecordQPS: 5
evictionHard:
  imagefs.available: 15%
  memory.available: 100Mi
  nodefs.available: 10%
  nodefs.inodesFree: 5%
evictionPressureTransitionPeriod: 5m0s
failSwapOn: true
fileCheckFrequency: 20s
hairpinMode: promiscuous-bridge
healthzBindAddress: 127.0.0.1
healthzPort: 10248
httpCheckFrequency: 20s
imageGCHighThresholdPercent: 85
imageGCLowThresholdPercent: 80
imageMinimumGCAge: 2m0s
iptablesDropBit: 15
iptablesMasqueradeBit: 14
kubeAPIBurst: 10
kubeAPIQPS: 5
makeIPTablesUtilChains: true
maxOpenFiles: 1000000
maxPods: 110
nodeStatusUpdateFrequency: 10s
oomScoreAdj: -999
podPidsLimit: -1
registryBurst: 10
registryPullQPS: 5
resolvConf: /etc/resolv.conf
rotateCertificates: true
runtimeRequestTimeout: 2m0s
serializeImagePulls: true
staticPodPath: /etc/kubernetes/manifests
streamingConnectionIdleTimeout: 4h0m0s
syncFrequency: 1m0s
volumeStatsAggPeriod: 1m0s
EOF
```

### kube-proxy配置

```bash
# 创建kube-proxy的ServiceAccount
kubectl -n kube-system create sa kube-proxy

# 创建ClusterRoleBinding，给kube-proxy这个sa绑定系统角色system:node-proxier
kubectl create clusterrolebinding system:kube-proxy --clusterrole system:node-proxier --serviceaccount kube-system:kube-proxy

# 获取sa对应的secret
SECRET=$(kubectl -n kube-system get sa kube-proxy --output=jsonpath='{.secrets[0].name}')

# 获取sa对应的secret token
JWT_TOKEN=$(kubectl -n kube-system get secret $SECRET --output=jsonpath='{.data.token}' | base64 -d)
```

在master节点上，配置kube-proxy的kubeconfig，发送给node节点

```bash
kubectl config set-cluster kubernetes \
  --certificate-authority=/etc/kubernetes/pki/ca.pem \
  --embed-certs=true \
  --server=https://192.168.25.3:6443 \
  --kubeconfig=/etc/kubernetes/kube-proxy.kubeconfig
```

```bash
kubectl config set-credentials kubernetes \
  --token=${JWT_TOKEN} \
  --kubeconfig=/etc/kubernetes/kube-proxy.kubeconfig
```

```bash
kubectl config set-context kubernetes \
  --cluster=kubernetes \
  --user=kubernetes \
  --kubeconfig=/etc/kubernetes/kube-proxy.kubeconfig
```

```bash
kubectl config use-context kubernetes \
  --kubeconfig=/etc/kubernetes/kube-proxy.kubeconfig
```

创建配置文件

```yaml
cat > /etc/kubernetes/kube-proxy.conf << EOF
apiVersion: kubeproxy.config.k8s.io/v1alpha1
bindAddress: 0.0.0.0
clientConnection:
  acceptContentTypes: ""
  burst: 10
  contentType: application/vnd.kubernetes.protobuf
  kubeconfig: /etc/kubernetes/kube-proxy.kubeconfig
  qps: 5
# 填写pod的网段
clusterCIDR: 172.16.0.0/12
configSyncPeriod: 15m0s
conntrack:
  max: null
  maxPerCore: 32768
  min: 131072
  tcpCloseWaitTimeout: 1h0m0s
  tcpEstablishedTimeout: 24h0m0s
enableProfiling: false
healthzBindAddress: 0.0.0.0:10256
hostnameOverride: ""
iptables:
  masqueradeAll: false
  masqueradeBit: 14
  minSyncPeriod: 0s
  syncPeriod: 30s
ipvs:
  masqueradeAll: true
  minSyncPeriod: 5s
  scheduler: "rr"
  syncPeriod: 30s
kind: KubeProxyConfiguration
metricsBindAddress: 127.0.0.1:10249
mode: "ipvs"
nodePortAddresses: null
oomScoreAdj: -999
portRange: ""
udpIdleTimeout: 250ms
EOF
```

创建Service

```bash
cat > /usr/lib/systemd/system/kube-proxy.service << EOF
[Unit]
Description=Kubernetes Kube Proxy
Documentation=https://github.com/kubernetes/kubernetes
After=network.target

[Service]
ExecStart=/usr/local/bin/kube-proxy \
  --config=/etc/kubernetes/kube-proxy.conf \
  --v=2

Restart=always
RestartSec=10s

[Install]
WantedBy=multi-user.target
EOF
```

启动kube-proxy

```bash
systemctl daemon-reload
systemctl enable --now kube-proxy
```

## 部署Calico

```bash
curl https://docs.projectcalico.org/archive/v3.21/manifests/calico-etcd.yaml -o calico-etcd.yaml
```

修改calico-etcd.yaml配置

```bash
ETCD_CA=`cat /etc/kubernetes/pki/etcd/etcd-ca.pem | base64 -w 0`
ETCD_CERT=`cat /etc/kubernetes/pki/etcd/etcd.pem | base64 -w 0`
ETCD_KEY=`cat /etc/kubernetes/pki/etcd/etcd-key.pem | base64 -w 0`
sed -i "s@# etcd-key: null@etcd-key: ${ETCD_KEY}@g" calico-etcd.yaml
sed -i "s@# etcd-cert: null@etcd-cert: ${ETCD_CERT}@g" calico-etcd.yaml
sed -i "s@# etcd-ca: null@etcd-ca: ${ETCD_CA}@g" calico-etcd.yaml
```

```bash
sed -i 's#etcd_ca: ""#etcd_ca: "/calico-secrets/etcd-ca"#g' calico-etcd.yaml
sed -i 's#etcd_cert: ""#etcd_cert: "/calico-secrets/etcd-cert"#g' calico-etcd.yaml
sed -i 's#etcd_key: ""#etcd_key: "/calico-secrets/etcd-key"#g' calico-etcd.yaml
```

```yaml
# 修改name: CALICO_IPV4POOL_CIDR
- name: CALICO_IPV4POOL_CIDR
  value: "172.16.0.0/12"
```

