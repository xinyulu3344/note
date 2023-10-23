# sed

格式

```bash
sed [OPTION] {scripts} [FILE]
```

OPTION：

```bash
```

scripts

```bash
'地址命令'
```

地址格式

```bash
```



命令：

```bash
```

示例：

```bash
# 打印第2行
sed -n 2p /etc/passwd

# 打印最后一行
sed -n '$p' /etc/passwd

sed -n '/^[^#]/p' /etc/fstab
```

