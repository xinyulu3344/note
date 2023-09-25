# systemd

ssystemd核心概念：unit

unit表示不同类型的system对象，通过配置文件进行标识和配置。

查看unit类型：

```bash
]# systemctl -t help
Available unit types:
service
socket
busname
target
snapshot
device
mount
automount
swap
timer
path
slice
scope
```

| unit      | 扩展名     | 解释                         |
| --------- | ---------- | ---------------------------- |
| service   | .service   | 定义系统服务                 |
| target    | .target    | 模拟实现运行级别             |
| device    | .device    | 定义内核识别的设备           |
| mount     | .mount     | 定义文件系统挂载点           |
| socket    | .socket    | 定义进程间通信用的socket文件 |
| snapshot  | .snapshot  | 管理系统快照                 |
| swap      | .swap      | 标识swap设备                 |
| automount | .automount | 文件系统的自动挂载点         |
| path      | .path      |                              |

## unit配置文件

```bash
/usr/lib/systemd/system
#Ubuntu对应的目录 
/lib/systemd/system
/run/systemd/system/
/etc/systemd/system
```

## 常用命令

```bash
systemctl start
systemctl stop
systemctl enable
systemctl disable
systemctl mask
systemctl unmask
systemctl is-active xxx.service
systemctl is-enabled xxx.service
# 查看所有已经激活的服务
systemctl list-units -t service
# 查看所有服务
systemctl list-units --type service --all
# 查看所有服务的开机自启状态
systemctl list-units-files --type service
```

