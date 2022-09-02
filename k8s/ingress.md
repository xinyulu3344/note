## 安装部署

### 使用heml部署

添加仓库

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
```

查看仓库

```bash
helm repo list
NAME            URL                                       
ingress-nginx   https://kubernetes.github.io/ingress-nginx
```

```bash
helm search repo ingress-nginx --version 3.6.0
NAME                            CHART VERSION   APP VERSION     DESCRIPTION                                       
ingress-nginx/ingress-nginx     3.6.0           0.40.2          Ingress controller for Kubernetes using NGINX a...
```

拉取压缩包并解压

```bash
helm pull ingress-nginx/ingress-nginx --version 3.6.0
tar -zxf ingress-nginx-3.6.0.tgz
cd ingress-nginx/
```

修改`values.yaml`

```yaml
controller:
  image:
    repository: registry.cn-beijing.aliyuncs.com/dotbalo/controller
  hostNetwork: true
  dnsPolicy: ClusterFirstWithHostNet
  kind: DaemonSet
  nodeSelector:
    ingress: "true"
  service:
    type: ClusterIP
```

给k8s-node01打上ingress=true的标签

```bash
kubectl label node k8s-node-01 ingress=true
```

安装ingress

```bash
helm install ingress-nginx -n ingress-nginx .
```

查看

```bash
kubectl get ds ingress-nginx-controller -n ingress-nginx
NAME                       DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR                         AGE
ingress-nginx-controller   1         1         1       1            1           ingress=true,kubernetes.io/os=linux   17m
```

## ingress yaml格式

```yaml
# extensions/v1beta1已被废弃
# 1.19之前的版本使用：networking.k8s.io/v1beta1
# 1.19及以后的版本使用：networking.k8s.io/v1
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  # 完整说明：https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/
  name: example
  namespace: Default
spec:
  rules:
  - host: nginx.xinyulu3344.cn
    http:
      paths: # 相当于Nginx的location配置
      - path: /
        pathType: Prefix
        backend:
          service:
            name: svc-clusterip-nginx
            port: 
              number: 8080
  ingressClassName: nginx
```

