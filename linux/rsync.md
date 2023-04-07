# rsync

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

