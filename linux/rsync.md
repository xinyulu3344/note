# rsync

## 安装

```bash
yum install rsync -y
```



## 基本使用

```bash
1、本地备份
rsync [OPTION...] SRC... [DEST]

2、远程备份
Access via remote shell:
Pull: rsync [OPTION...] [USER@]HOST:SRC... [DEST]
Push: rsync [OPTION...] SRC... [USER@]HOST:DEST

3、守护进程方式备份
Access via rsync daemon:
Pull: rsync [OPTION...] [USER@]HOST::SRC... [DEST]
rsync [OPTION...] rsync://[USER@]HOST[:PORT]/SRC... [DEST]
Push: rsync [OPTION...] SRC... [USER@]HOST::DEST
rsync [OPTION...] SRC... rsync://[USER@]HOST[:PORT]/DEST

Usages with just one SRC arg and no DEST arg will list the source files instead of copying.
```

## 实现ls功能

```bash
[root@CentOS74 ~]# rsync /etc/hosts
-rw-r--r--            158 2013/06/07 22:31:32 hosts
```

## 实现rm功能

```bash
[root@CentOS74 rsynctest]# ls src/ # 空目录
[root@CentOS74 rsynctest]# 
[root@CentOS74 rsynctest]# ls dst/ # 待清空目录
test01.txt  test02.txt  test03.txt
[root@CentOS74 rsynctest]# 
[root@CentOS74 rsynctest]# rsync -rp --delete src/ dst/
[root@CentOS74 rsynctest]# 
[root@CentOS74 rsynctest]# ls dst/
[root@CentOS74 rsynctest]#
```

## 实现cp功能

```bash
# 只将src目录下所有文件同步到dst目录下
rsync -rp src/ dst/

# 将src整个目录同步到dst目录下
rsync -rp src dst/
```

## 实现scp功能

```bash
```

## rsync守护进程模式搭建

### 编写配置文件

**/etc/rsyncd.conf**

```
 # 指定管理备份目录的用户
uid = rsync
 # 指定管理备份目录的用户组
gid = rsync

# 指定rsync服务的端口号
port = 873

fake super = yes

use chroot = no

# 最大连接数
max connections = 200

# 超时时间（秒）
timeout = 300

# 定义进程号文件
pid file = /var/run/rsyncd.pid

# 锁文件
lock file = /var/run/rsync.lock

# rsync服务的日志文件
log file = /var/log/rsyncd.log

# 忽略简单错误
ignore errors

# 指定备份目录是可读可写的
read only = false 

# 服务端允许客户端查看模块信息
list = false

# 允许传输备份数据的主机
hosts allow = 192.168.0.0/16

# 禁止传输备份数据的主机
hosts deny = 0.0.0.0/32

# 指定认证用户
auth users = rsync_backup

# 指定认证文件
secrets file = /etc/rsync.password

# 模块信息
[backup]
comment = "backup dir"
# 指定备份目录
path = /backup
```

### 添加rsync虚拟用户

```bash
useradd rsync -M -s /usr/sbin/nologin
```

### 创建认证密码文件并修改权限

```bash
echo "rsync_backup:123456" > /etc/rsync.password
chmod 600 /etc/rsync.password
```

### 创建备份目录并修改属主属组信息

```bash
mkdir /backup
chown rsync:rsync /backup/
```

### 启动服务

```bash
systemctl start rsyncd
systemctl enable rsyncd
```

### 配置文件中黑白名单参数

白名单优先黑名单

1. 只有hosts allow：先allow，默认deny
2. 只有hosts deny：先deny，默认allow
3. 两个都有：先allow，后deny，默认是allow

### rsync命令的常用参数

| 参数                | 解释                           |
| ------------------- | ------------------------------ |
| -v                  | 显示详细信息                   |
| -a                  | 归档参数                       |
| -t                  | 保持文件属性修改时间不变       |
| -o                  | 保持文件的属主信息不变         |
| -g                  | 保持文件的属组信息不变         |
| -p                  | 保持文件权限信息不变           |
| -P                  | 显示传输进度信息               |
| -D                  | 保持设备文件信息不变           |
| -l                  | 保持链接文件                   |
| -L                  | 保持链接文件数据信息不变       |
| --exclude=PATTERN   | 指定数据不被传输               |
| --exclude-from=FILE | 从文件中获取不被传输的数据文件 |
| --bwlimit=KBPS      | 限制传输速率                   |
| --delete            | 无差异同步数据，两边一模一样   |

【注意】

想要-o、-g参数生效，需要将配置文件中，uid、gid改为root，同时，将fake super参数改为no

## 常见问题总结

**@ERROR: auth failed on module backup**

【报错现象】

```bash
[root@CentOS68 ~]# rsync -avz /etc/hosts rsync_backup@192.168.1.171::backup
Password:  
@ERROR: auth failed on module backup
rsync error: error starting client-server protocol (code 5) at main.c(1503) [sender=3.0.6]
```

【报错原因】

1. 用户名密码写错
2. 实际的认证文件和服务端配置文件中，secrets file指定的文件不一致
