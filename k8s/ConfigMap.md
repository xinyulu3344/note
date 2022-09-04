# ConfigMap

## ConfigMap的用法

1. 生成为容器内的环境变量
2. 设置容器启动命令的启动参数
3. 以Volume的形式挂载为容器内部的文件或目录

## 创建ConfigMap

方法一：通过YAML配置文件方式创建

* YAML文件中定义变量

  ```yaml
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: cm-appvars
    namespace: default
  data:
    apploglevel: info
    appdatadir: /var/data
  ```

* YAML文件中定义文件内容

  ```yaml
  ```
  
  

  ```bash
  kubectl create -f cm-appvars.yaml 
  ```



方法二：通过kubectl命令行的方式创建

* `--from-file` 从文件中创建ConfigMap，key为指定的key，value为文件内容。不指定key，则key为文件名。

  ```bash
  kubectl create cm NAME --from-file=[key1=]source1 --from-file=[key2=]source2
  ```

* `--from-file`参数通过目录指定key和value。目录下每个文件名作为key，文件内容为value

  ```bash
  kubectl create cm NAME --from-file=DIR
  ```

* `--from-literal`参数直接指定key和value

  ```bash
  kubectl create cm NAME --from-literal=key1=value1 --from-literal=key2=value2
  ```

## 在Pod中使用ConfigMap

**1. 通过环境变量**

方式一：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: cm-test-pod
spec:
  containers:
  - name: cm-test
    image: busybox
    command: ["/bin/sh", "-c", "env | grep APP"]
    env:
    - name: APPLOGLEVEL     # 定义环境变量名称
      valueFrom:            # 定义环境变量值的来源
        configMapKeyRef:    # 环境变量的值来源于ConfigMap
          name: cm-appvars  # 环境变量的值来源于cm-appvars这个ConfigMap
          key: apploglevel  # 环境变量的值来源于key为apploglevel对应的value
    - name: APPDATADIR
      valueFrom:
        configMapKeyRef:
          name: cm-appvars
          key: appdatadir
  restartPolicy: Never
```

方式二：k8s 1.6版本之后，引入了envFrom字段，将ConfigMap中所有key-value自动生成环境变量

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: cm-test-pod
spec:
  containers:
  - name: cm-test
    image: busybox
    command: ["/bin/sh", "-c", "env | grep app"]
    envFrom:
    - configMapRef:
      # 根据cm-appvars这个ConfigMap中的key-value自动生成环境变量
        name: cm-appvars
  restartPolicy: Never
```

**2. 通过volumeMount**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: cm-test-app
spec:
  containers:
  - name: cm-test-app
    image: busybox
    command: ["sleep", "3600"]
    volumeMounts:
    - name: myconfig           # 引用Volume的名称
      mountPath: /configfiles  # 挂载到容器内的目录
  volumes:
  - name: myconfig
    configMap:
      name: cm-appconfigfiles  # ConfigMap名称
      items:
      - key: log-config        # ConfigMap中的key
        path: logconfig.yaml   # ConfigMap中的value作为内容，logconfig.yaml为文件名
```

登录容器，`/configfiles`目录下，有一个logconfig.yaml文件，内容就是`cm-appconfigfiles`这个ConfigMap中`log-config`这个key对应的value。



## 使用ConfigMap的限制条件

* ConfigMap必须在Pod之前创建。 
* ConfigMap受Namespace限制，只有处于相同Namespace中的 Pod才可以引用它。 
* ConfigMap中的配额管理还未能实现。 
* kubelet只支持可以被API Server管理的Pod使用ConfigMap。 
* kubelet在本Node上通过 --manifest-url或--config自动创建的静态Pod将无 法引用ConfigMap。
* 挂载目录下的其它文件会被ConfigMap覆盖。可以通过subPath解决

## 使用subPath解决挂载目录被覆盖

以subPath形式挂载的话，那么Pod无法感知到ConfigMap和Secret的更新。

如果Pod的变量来自于ConfigMap和Secret中定义的内容，那么ConfigMap和Secret更新后，Pod中的变量不会同步被更新

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: cm-appvars
  namespace: default
data:
  nginx.conf: hello

---

apiVersion: v1
kind: Pod
metadata:
  name: cm-test-app
spec:
  containers:
  - name: cm-test-app
    image: nginx:1.15.3
    imagePullPolicy: IfNotPresent
    volumeMounts:
    - name: myconfig           # 引用Volume的名称
      mountPath: /etc/nginx/nginx.conf # 只是覆盖/etc/nginx/nginx.conf文件，其它文件不会被覆盖
      subPath: nginx.conf
    command: ['sh', '-c', 'sleep 3600']
  volumes:
  - name: myconfig
    configMap:
      name: cm-appvars  # ConfigMap名称
```

