## YAML定义

```yaml
apiVersion: v1
kind: Service
metadata:
  name: string
  namespace: string
  labels:
    - name: string
  annotations:
    - name: string
spec:
  selector: 
    key: value
  # ClusterIP: 虚拟出来的地址,用于Kubernetes集群内部的Pod访问
  #            在Node上kube-proxy通过设置iptable规则进行转发
  # NodePort: 使用宿主机端口, 通过Node的IP+端口来访问服务
  # LoadBalancer: 用于云环境负载均衡
  type: string
  # 当type=ClusterIP时,如果不指定,则系统自动分配
  clusterIP: string
  # ClientIP: 同一个客户端IP的请求都转发到同一个后端Pod
  # None: 轮询
  sessionAffinity: string
  # Service需要暴露的端口列表
  ports:
  - name: string
    # 端口协议: TCP(默认)、UDP
    protocol: string
    # 服务监听端口号
    port: int 
    # 需要转发到后端Pod的端口号
    targetPort: int 
    # 当type为NodePort时, 指定映射到物理机的端口号
    nodePort: int 
status:
  loadBalancer:
    ingress:
      ip: string
      hostname: string
```

## 外部服务Service

Service的后端服务，可以手动指定。比如集群外的某个IP+Port服务。实现方式：

1. 创建Service，不指定标签选择器
2. 手动创建和Service同名的Endpoints，指定ip和端口

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

```yaml
apiVersion: v1
kind: Endpoints
metadata:
  name: my-service
subsets:
- addresses:
  - ip: 220.181.38.150
  ports:
  - port: 80
```

## Headless Service

不为Service设置ClusterIP，仅通过Label Selector将后端的Pod列表返回给调用的客户端。

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
  clusterIP: None
  selector:
    app: nginx
```

## 从集群外访问Pod或者Service

### 将容器端口映射到宿主机

通过设置容器级别的hostPort，将容器的端口号映射到宿主机。此时可以通过pod所在宿主机的端口，访问容器。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp
  labels:
    app: webapp
spec:
  containers:
  - name: webapp
    image: tomcat
    ports:
    - containerPort: 8080
      hostPort: 8081
```

通过设置Pod级别的hostNetwork=true，该Pod中所有容器的端口号都将被直接映射到物理机上。在设置hostNetwork=true时需要注意，在容器的ports定义部分如果不指定hostPort，则默认hostPort等于containerPort，如果指定了hostPort，则hostPort必须等于containerPort的值：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp
  labels:
    app: webapp
spec:
  hostNetwork: true
  containers:
  - name: webapp
    image: tomcat
    ports:
    - containerPort: 8080
```

### 将Service端口映射到宿主机

nodePort的方式

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30000
  selector:
    app: nginx
```

loadBalancer的方式

## DNS服务

### Pod级别DNS

在pod的YAML文件中，指定spec.dnsPolicy

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox
  namespace: default
spec:
  containers:
  - name: busybox
    image: busybox
    imagePullPolicy: IfNotPresent
    args:
    - /bin/sh
    - -c
    - sleep 3600
  # Default
  # ClusterFirst
  # ClusterFirstWithHostNet
  # None
  dnsPolicy: ClusterFirst
  dnsConfig:
    nameservers:
    - 103.224.222.222
    - 103.224.222.223
    - 8.8.8.8
    options:
    - name: timeout
      value: "5"
```

dnsPolicy可配置字段如下：

* **Default**: 继承Pod所在宿主机的DNS设置
* **ClusterFirst**: 优先使用Kubernetes环境的DNS服务, 将无法解析的域名转发到从宿主机继 承的DNS服务器。
* **ClusterFirstWithHostNet**: 与ClusterFirst相同，对于以hostNetwork模式运行的Pod，应明确指定使用该策略
* **None**: 忽略Kubernetes环境的DNS配置，通过spec.dnsConfig自定义DNS配置。

自定义DNS配置可以通过`spec.dnsConfig`字段进行设置，可以设置下列信息。

* nameservers：一组DNS服务器的列表，最多可以设置3个
* searches：一组用于域名搜索的DNS域名后缀，最多可以设置6个
* options：配置其他可选DNS参数，例如ndots、timeout等，以name或name/value对的形式表示。

## Ingress

部署ingress controller

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.30.0/deploy/static/mandatory.yaml
```

controller会不断的报错

```
err services "ingress-nginx" not found
```

部署一个fix.yaml，让controller不再报错

```yaml
apiVersion: v1
kind: Service
metadata:
  name: ingress-nginx
  namespace: ingress-nginx
spec:
  type: ClusterIP
  ports:
  - name: http
    port: 80
    targetPort: 80
    protocol: TCP
  - name: https
    port: 443
    targetPort: 443
    protocol: TCP
  selector:
    app: ingress-nginx
```

定义一个svc-nginx-ingress-controller.yaml，将controller暴露到公网，公有云中可以通过loadbalance service暴露。

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
  name: nginx-ingress-controller
  namespace: ingress-nginx
spec:
  externalTrafficPolicy: Cluster
  ports:
  - port: 80
    protocol: TCP 
    targetPort: 80
    name: http
  - port: 443 
    protocol: TCP 
    targetPort: 443 
    name: https
  selector:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
  type: LoadBalancer
```

定义业务pod和Service

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  replicas: 4
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80

---

apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  type: ClusterIP
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx
```



定义ingress

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/backend-protocol: HTTP # 后端服务是HTTP还是HTTPS
  name: nginx
  namespace: default
spec:
  rules:
  - host: www.xinyulu3344.cn
    http:
      paths:
      - path: /
        backend:
          serviceName: nginx
          servicePort: 80
```



### TLS配置

创建tls类型的secret

```
kubectl create secret tls NAME --key tls.key --cert tls.crt
```

在ingress规则中引用secret

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/backend-protocol: HTTP
  name: nginx-tls
  namespace: default
spec:
  tls:
  - hosts:
    - www.xinyulu3344.cn
    secretName: www.xinyulu3344.cn
  rules:
  - host: www.xinyulu3344.cn
    http:
      paths:
      - path: /
        backend:
          serviceName: nginx
          servicePort: 80
```

