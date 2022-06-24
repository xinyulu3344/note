#  安装k8s集群

## 基本环境配置

关闭防火墙、selinux

关闭swap

```bash
swapoff -a && sysctl -w vm.swappiness=0
```

配置/etc/sysctl.d/k8s.conf

```
fs.inotify.max_queued_events = 163840
fs.inotify.max_user_instances = 12800
fs.inotify.max_user_watches = 81920
kernel.pid_max = 655300
fs.file-max = 60480000
net.ipv4.ip_forward=1
fs.may_detach_mounts=1
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.conf.default.rp_filter = 0
net.ipv4.conf.all.rp_filter = 0
net.ipv4.neigh.default.gc_thresh1 = 2048
net.ipv4.neigh.default.gc_thresh2 = 4096
net.ipv4.neigh.default.gc_thresh3 = 8192
net.ipv4.neigh.default.gc_interval = 120
net.ipv4.neigh.default.gc_stale_time = 240
vm.swappiness=0
net.ipv4.ip_local_reserved_ports = 50051,51678
```

安装ntpdate时间同步

修改时区

```bash
ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

配置/etc/security/limits.conf

```
# Default limit for number of user's processes to prevent
# accidental fork bombs.
# See rhbz #432903 for reasoning.

*          soft    nproc     4096
root       soft    nproc     unlimited
* soft nofile 102400
* hard nofile 204800
* soft nproc 204800
* hard nproc 307200
* soft stack 10240
* hard stack 20480
```

配置yum源

```bash
yum install -y yum-utils
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

```bash
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

升级系统（此处不升级内核）

```bash
yum install wget jg psmisc vim net-tools telnet yum-utils device-mapper-persistent-data lvm2 -y
yum update -y --exclude=kernel* && reboot
```

内核配置

升级内核

https://www.kernel.org/ 和 https://elrepo.org/linux/kernel/el7/x86_64/

http://193.49.22.109/elrepo/kernel/el7/x86_64/RPMS/

```bash
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm

yum --disablerepo="*" --enablerepo="elrepo-kernel" -y install kernel-ml kernel-ml-devel
或
dnf --disablerepo=\* --enablerepo=elrepo-kernel -y install kernel-ml kernel-ml-devel

grubby --default-kernel
```

修改内核启动顺序

```bash
awk -F \' '$1=="menuentry " {print i++ " : " $2}' /etc/grub2.cfg
0 : CentOS Linux (5.18.5-1.el7.elrepo.x86_64) 7 (Core)
1 : CentOS Linux (3.10.0-693.el7.x86_64) 7 (Core)
2 : CentOS Linux (0-rescue-e658d54604f59aa28275af74dce7c2cf) 7 (Core)
```

```bash
grub2-set-default "CentOS Linux (4.19.12-1.el7.elrepo.x86_64) 7 (Core)"
```



所有节点安装ipvs

```bash
yum install ipvsadm ipset sysstat conntrack libseccomp -y
```

所有节点配置ipvs模块，在内核4.19版本nf_conntrack_ipv4已经改为nf_conntrack

```bash
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack
```

```bash
cat /etc/modules-load.d/ip_vs.conf
ip_vs
ip_vs_rr
ip_vs_wrr
ip_vs_sh
nf_conntrack
ip_tables
ip_set
xt_set
ipt_set
ipt_rpfilter
ipt_REJECT
ipip
```

开启systemd-modules-load.service服务

```bash
systemctl enable --now systemd-modules-load.service
```

检查模块是否加载

```bash
lsmod | grep -e ip_vs -e nf_conntrack
```

## 基本组件安装

### docker

查看可安装版本

```bash
yum list docker-ce.x86_64 --showduplicates | sort -r
```

所有节点安装docker，注意版本

```bash
yum install -y docker-ce-19.03.9-3.el7
```

启动并开机自启docker

```bash
systemctl enable --now docker
```

### kubeadm、kubelet、kubectl

查看可安装版本

```bash
yum list kubeadm --showduplicates | sort -r
```

所有节点安装指定版本k8s组件

```bash
yum install kubeadm-1.18.5-0 kubelet-1.18.5-0 kubectl-1.18.5-0 kubernetes-cni-0.8.7.0
```

默认配置的pause镜像使用gcr.io仓库，国内无法访问，所以这里配置kubelet使用阿里云的pause镜像

```bash
DOCKER_CGROUPS=$(docker info | grep 'Cgroup' | cut -d' ' -f3)
cat > /etc/sysconfig/kubelet <<EOF
KUBELET_EXTRA_ARGS="--cgroup-driver=$DOCKER_CGROUPS --pod-infra-container-image=registry.cn-hangzhou.aliyuncs.com/google_containers/pause-amd64:3.1"
EOF
```

## 高可用组件安装

HAProxy + keepalived

## 初始化集群

生成kubeadm-config.yaml文件

```bash
kubeadm config print init-defaults > kubeadm-config.yaml
```

修改：

1. advertiseAddress的值为master的IP，如果是集群部署，就写vip
2. imageRepository的值为国内可用的仓库，比如阿里云的`registry.cn-hangzhou.aliyuncs.com/google_containers`
3. 添加`controlPlaneEndpoint: 192.168.25.3:6443`
4. 添加networking.podSubnet：`podSubnet: 172.16.0.0/16`

拉取master相关组件镜像

```bash
kubeadm config images pull --config kubeadm-config.yaml
```

```
[config/images] Pulled registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver:v1.18.5
[config/images] Pulled registry.cn-hangzhou.aliyuncs.com/google_containers/kube-controller-manager:v1.18.5
[config/images] Pulled registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler:v1.18.5
[config/images] Pulled registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy:v1.18.5
[config/images] Pulled registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.2
[config/images] Pulled registry.cn-hangzhou.aliyuncs.com/google_containers/etcd:3.4.3-0
[config/images] Pulled registry.cn-hangzhou.aliyuncs.com/google_containers/coredns:1.6.7
```

所有节点设置kubelet开机自启动

```bash
systemctl daemon-reload
systemctl enable --now kubelet
```

新版本k8s更新旧版本kubeadm-config.yaml文件

```bash
kubeadm config migrate --old-config kubeadm-config.yaml --new-config kubeadm-config-new.yaml
```

k8s-master01节点初始化，初始化后会在/etc/kubernetes目录下生成对应的证书和配置文件，之后其它master节点加入k8s-master01即可

```bash
kubeadm init --config kubeadm-config.yaml --upload-certs
```

如果初始化失败，重置后再次初始化：

```bash
kubeadm reset
```

初始化成功后，会产生Token值，用于其它节点加入时使用，因此要记录下初始化成功生成的token值。

加入master节点

```bash
kubeadm join 192.168.25.3:6443 \
  --token ysvnp2.z760j0z73i0x2koh \
  --discovery-token-ca-cert-hash sha256:986ad5d7cb487bdd2c69798a301c4a89eca97c56720462b15b808e04e37768a5 \
  --control-plane --certificate-key b0d8f6326f43db41d9368caac21d8a74240b5aa820f7f7cfb47c74033c3a6bdc
```

加入node节点

```bash
kubeadm join 192.168.25.3:6443 \
  --token ysvnp2.z760j0z73i0x2koh \
  --discovery-token-ca-cert-hash sha256:986ad5d7cb487bdd2c69798a301c4a89eca97c56720462b15b808e04e37768a5
```

--token查看

```bash
# 列出所有token
kubeadm token list
kubectl get secret -n kube-system | grep bootstrap-token
```

--token生成

```bash
kubeadm token create --print-join-command
```

--discovery-token-ca-cert-hash查看

```bash
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'
```

--control-plane --certificate-key两个小时之后就过期了，后续如果有新的master要加入集群，需要重新生成，命令如下：

```bash
kubeadm init phase upload-certs --upload-certs

[upload-certs] Storing the certificates in Secret "kubeadm-certs" in the "kube-system" Namespace
[upload-certs] Using certificate key:
b0d8f6326f43db41d9368caac21d8a74240b5aa820f7f7cfb47c74033c3a6bdc
```

## Calico组件的安装

官网：https://projectcalico.docs.tigera.io/archive/v3.18/getting-started/kubernetes/self-managed-onprem/onpremises

注意版本！

```bash
curl https://docs.projectcalico.org/archive/v3.19/manifests/calico.yaml -O
```

修改

```yaml
- name: CALICO_IPV4POOL_CIDR
  value: "172.16.0.0/16"
```

创建

```bash
kubectl apply -f calico.yaml
```

## Metrics部署

下载yaml文件

```bash
wget https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.3.6/components.yaml
```

修改image为国内可用源

```bash
image: registry.cn-hangzhou.aliyuncs.com/google_containers/metrics-server-amd64:v0.3.6
```

## dashboard部署

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.3/aio/deploy/recommended.yaml
```

官方的dashboard不好用，可以用kuboard，国人开发

[kuboard](https://www.kuboard.cn)

## 配置kube-proxy使用ipvs

编辑kube-proxy的ConfigMap，将mode: ""改为mode: "ipvs"

```bash
kubectl edit cm kube-proxy -n kube-system
```

删除所有kube-proxy的pod，ds控制器会创建新的pod。

```bash
kubectl delete pods kube-proxy-xsm4h -n kube-system
```

