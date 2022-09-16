## yaml文件定义

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
  namespace: default
data:
  prometheus.yml: |
    # my global config
    global:
      scrape_interval: 15s # 设置抓取间隔为15s, 默认是1分钟
      evaluation_interval: 15s # 监控数据规则评估频率, 指的是prometheus会多长时间进行一次监控规则的评估. 比如设置内存使用量 > 70%时进行报警, 那么prometheus会默认15s来执行一次这个规则, 检查内存的情况
      # scrape_timeout is set to the global default (10s).

    # Alertmanager configuration
    alerting:
      alertmanagers:
        - static_configs:
            - targets:
              # - alertmanager:9093

    # Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
    rule_files:
      # - "first_rules.yml"
      # - "second_rules.yml"

    # prometheus本身的配置
    scrape_configs:
      # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
      - job_name: "prometheus"

        # metrics_path defaults to '/metrics'
        # scheme defaults to 'http'.

        # 被监控节点
        static_configs:
          - targets: ["192.168.32.3:9100", "192.168.25.3:9100"]

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: prometheus
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: prometheus
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: prometheus
      name: prometheus
    spec:
      containers:
      - image: prom/prometheus:v2.38.0
        args:
        - --config.file=/etc/prometheus/prometheus.yml
        - --storage.tsdb.retention=3d
        imagePullPolicy: IfNotPresent
        name: prometheus
        ports:
        - containerPort: 9090
          protocol: TCP
        volumeMounts:
        - mountPath: /etc/prometheus
          name: config-volume
        - mountPath: /prometheus/
          name: data-volume
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
      volumes:
      - configMap:
          defaultMode: 420
          name: prometheus-config
        name: config-volume
      - emptyDir: {}
        name: data-volume 

---

apiVersion: v1
kind: Service
metadata:
  name: svc-prometheus
  namespace: default
  labels:
    app: prometheus
spec:
  selector:
    app: prometheus
  type: ClusterIP
  sessionAffinity: None
  ports:
  - name: prometheus
    protocol: TCP
    port: 49090
    targetPort: 9090

---

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  # 完整说明：https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/
  name: prometheus
  namespace: default
spec:
  rules:
  - host: prometheus.xinyulu3344.cn
    http:
      paths: # 相当于Nginx的location配置
      - path: /
        pathType: Prefix
        backend:
          service:
            name: svc-prometheus
            port: 
              number: 49090
  ingressClassName: nginx
```

## 创建相关资源

```bash
kubectl apply -f prometheus.yaml
```

