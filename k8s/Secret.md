## 概念

用来保存敏感信息的，比如密码、令牌或者密钥

官方文档：https://kubernetes.io/zh-cn/docs/concepts/configuration/secret/

## 命令行创建

```bash
kubectl create secret generic db-user-pass \
--from-file=[username=]./username.txt \
--from-file=[password=]./password.txt
```

```bash
kubectl create secret generic db-user-pass \
  --from-literal=username=devuser \
  --from-literal=password='S!B\*d$zDsb='
```

