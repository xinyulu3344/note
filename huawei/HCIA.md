# HCIA

查看接口简要信息

```
dis ip int bri
```

修改路由器名称

```
sysname R1
```



进接口配置IP

```
int g0/0/0

# 使用前缀来配置
ip add 12.1.1.2 24

# 使用掩码来配置
ip add 12.1.1.2 255.255.255.0
```

查看接口当前配置

```
dis this
```

删除IP配置

```
undo ip add
```

远程管理

```
user-interface vty 0 4
authentication-mode password
user 
```

