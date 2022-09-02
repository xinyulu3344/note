## 概念

用来保存敏感信息的，比如密码、令牌或者密钥

官方文档：https://kubernetes.io/zh-cn/docs/concepts/configuration/secret/

## Secret管理

### 命令行创建

`--from-file`从文件创建。默认使用文件名作为key，文件内容的base64字符串作为value。key可以在命令行中指定

```bash
kubectl create secret generic db-user-pass \
--from-file=username=./username.txt \
--from-file=password=./password.txt
```

```bash
kubectl create secret generic db-user-pass \
--from-file=./username.txt \
--from-file=./password.txt
```

`--from-literal`从命令行指定key-value。value是base64编码前的内容。

```bash
kubectl create secret generic db-user-pass \
  --from-literal=username=devuser \
  --from-literal=password='S!B\*d$zDsb='
```

### 命令行删除

```bash
kubectl delete secret db-user-pass
```

### 命令行修改

```bash
kubectl edit secret db-user-pass
```

### 配置文件管理Secret

直接创建

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  # value需要填写base64编码之后的字符串
  username: YWRtaW4=
  password: MWYyZDFlMmU2N2Rm
```

使用stringData字段，可以使用base64编码之前的字符串填入，创建之后，会在data字段中，以config.yaml作为key，后面内容的base64字符串作为value

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
stringData:
  config.yaml: |
    apiUrl: "https://my.api.com/api/v1"
    username: admin
    password: 1f2d1e2e67df
```

如果在 `data` 和 `stringData` 中设置了同一个字段，则使用来自 `stringData` 中的值。

## 使用Secret

1. 以数据卷的方式挂载
2. 作为环境变量暴露给pod

### 以文件形式使用Secret

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  # value需要填写base64编码之后的字符串
  username: YWRtaW4=
  password: MWYyZDFlMmU2N2Rm

---

apiVersion: v1
kind: Pod
metadata:
  name: busybox
spec:
  containers:
  - name: busybox
    image: busybox
    args:
    - /bin/sh
    - -c
    - sleep 3600
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo" # 最终会在该目录下，生成password  username两个文件，文件内容是secret中base64解码后的value
      readOnly: true
  volumes:
  - name: foo
    secret:
      secretName: mysecret
      optional: false # 默认设置，意味着 "mysecret" 必须已经存在
```

如果想让Secret中指定的key映射到特定的目录，可以通过spec.volumes.secret.items来配置。示例如下

```yaml
```



