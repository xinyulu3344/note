## 创建ConfigMap

拿到Grafana的配置文件`defaults.ini`，然后使用kubectl命令创建ConfigMap

```bash
kubectl create cm grafana-config --from-file=defaults.ini
```



## 定义yaml文件

创建grafana.yaml文件，定义PVC、Deployment、Service、Ingress

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: grafana-pvc        #pvc声明卷名称
  namespace: default         #pvc与pod必须在相同的命名空间
spec:
  accessModes:
  - ReadWriteOnce      #读写模式，现在为单点读写
  storageClassName: nfs-slow
  resources:
    requests:
      storage: 5Gi      #卷大小

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana
  namespace: default
spec:
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      labels:
        app: grafana
    spec:
      volumes:
      - name: storage
        persistentVolumeClaim:
          claimName: grafana-pvc       #存放grafana数据盘
      - name: config
        configMap:
          name: grafana-config
      nodeSelector:
        app: prometheus
      securityContext:         #解决容器读取文件夹权限
        runAsUser: 0
      containers:
      - name: grafana
        image: grafana/grafana:9.1.4
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 3000
          name: grafana
        env:
        - name: GF_SECURITY_ADMIN_USER
          value: admin     #帐号
        - name: GF_SECURITY_ADMIN_PASSWORD
          value: XXX     #密码
        readinessProbe:
          failureThreshold: 10
          httpGet:
            path: /api/health
            port: 3000
            scheme: HTTP
          initialDelaySeconds: 60
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 30
        livenessProbe:
          failureThreshold: 3
          httpGet:
            path: /api/health
            port: 3000
            scheme: HTTP
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 1
        resources:
          limits:
            cpu: 150m
            memory: 512Mi
          requests:
            cpu: 150m
            memory: 512Mi
        volumeMounts:
        - mountPath: "/var/lib/grafana"
          name: storage
        - mountPath: "/usr/share/grafana/conf/defaults.ini"
          name: config
          subPath: defaults.ini

---

apiVersion: v1
kind: Service
metadata:
  name: svc-grafana
  namespace: default
spec:
  ports:
  - port: 3000
    protocol: TCP
    targetPort: 3000
  selector:
    app: grafana              #此处需与deployment的标签选择器一致
  type: ClusterIP

---

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  # 完整说明：https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/
  name: grafana
  namespace: default
spec:
  rules:
  - host: grafana.xinyulu3344.cn
    http:
      paths: # 相当于Nginx的location配置
      - path: /
        pathType: Prefix
        backend:
          service:
            name: svc-grafana
            port:
              number: 3000
  ingressClassName: nginx
```

执行创建命令

```bash
kubectl apply -f grafana.yaml
```

