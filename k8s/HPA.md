## 概念

HPA：Horizontal Pod Autoscale: Pod的水平自动伸缩器

观察Pod的CPU、内存使用率自动扩展或缩容Pod的数量。

不适用于无法缩容的对象，比如DaemonSet

CPU、内存、自定义指标的扩容

必须定义Requests参数，必须部署metrics-server

## 配置

```bash
kubectl autoscale deploy nginx-deploy --cpu-percent=80 --min=2 --max=5
```

