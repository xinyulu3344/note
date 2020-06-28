# 硬件知识

## CPU

路数：CPU个数

核数：

## 内存

优点：

缺点：

buffer（缓冲区）：在数据准备存储到磁盘中之前，先放到内存中进行缓冲，将数据临时存储到内存的过程，叫做写（缓冲）内存。

cache（缓存区）：数据已经存储到磁盘中之后，再把数据转存到内存中；将数据从内存中进行读取的过程，叫做读（缓存）内存

## 磁盘

### 磁盘接口类型

机械硬盘（按存储效率排序）：IDE < SATA < SCSI < SAS

固态硬盘：SSD

### RAID卡

## 远程管理卡

### 远程管理卡的种类

1. 集成的远程管理卡，只能远程开关机
2. 外置的远程管理卡 远程安装系统 配置raid

### 厂商型号

dell  - idrac

HP - ilo

IBM - RSA

## 主板

# Linux安装

## 安装CentOS7时，网卡命名使用ethx而不是ensxx

1. 选中`Install CentOS 7`
2. 按下TAB键
3. 键入`net.ifnames=0 biosdevname=0`

# Linux基础知识

## 开机启动流程

### CentOS 6

1. 开机自检，检查服务器硬件是否正常
2. MBR引导，读取磁盘的MBR存储记录信息，引导系统启动
3. GRUB菜单，选择启动的内核 / 单用户模式重置密码
4. 加载内核
5. 运行INIT进程，系统中第一个进程
6. 加载系统运行级别文件/etc/inittab
7. 初始化脚本运行，初始化系统主机名称和网卡信息
8. 运行系统特殊的脚本，/etc/rc.d/rc，启动服务
9. 运行mingetty进程，显示登录界面

### CentOS 7

1. 开机自检
2. MBR引导
3. GRUB菜单
4. 加载内核
5. 运行Systemd进程
6. 读取系统启动级别文件，/etc/systemd/system/default.target
7. 读取系统初始化文件，/usr/lib/systemd/system/sysinit.target
8. 加载/etc/systemd/system/目录下的信息，使服务可以开机自启动
9. 运行mingetty进程，显示登录界面

## 系统运行级别

### CentOS 6

### CentOS 7

## 别名

### 查看别名

```bash
alias
alias 别名名称
```

### 设置别名

## 系统特殊符号

```bash
# ~
# ..
# >
标准输出重定向

# 2>
标准错误输出重定向

# >>
# 2>>
# <
# <<
# &&
# ;
# #
# $
# !
# ``
# |
# {}
```

## 用户

### 用户的分类

|          | 超级管理员   | 虚拟用户                          | 普通用户                                            |
| -------- | ------------ | --------------------------------- | --------------------------------------------------- |
| uid      | 0            | 1-999                             | 1000+                                               |
| 从权限上 | 想干嘛就干嘛 | 管理进程，无法登录系统/没有家目录 | 管理指定数据 可以管理指定进程 可以登录系统/有家目录 |
|          |              |                                   |                                                     |



* 超级管理员用户

  uid=0

* 虚拟用户

  

* 普通用户

### 用户和组的关系

## 权限管理

### 权限几个测试结论

**文件权限**

1. ****

**目录权限**

### 默认权限

## 正则表达式

### 系统符号

1. 基础符号

```bash
# $
1. 用于取出变量中的内容
2. 用于取出指定列的信息（awk）
3. 表示用户命令提示符号
4. 表示一行的结尾

# !
1. 用于表示取反或者排除
2. 命令行中表示取出最近命令
3. vim中表示强制操作
```

2. 引号

```bash
# ''：所见即所得，
[root@CentOS74 ~]# echo '$PATH'
$PATH

# ""：对特殊信息会做解析处理
[root@CentOS74 ~]# echo "$PATH"
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/opt/ibutils/bin:/root/bin

# 没有引号：功能基本与""类似，但是没有引号可以识别通配符信息
[root@CentOS74 ~]# echo {01..10}
01 02 03 04 05 06 07 08 09 10

# ``反引号：将命令执行结果留下，用于其它命令调用

# $()：与``一样，都是用于将命令执行结果留下，用于其它命令调用
```

3. 重定向

```bash
> / 1>
2>

>> / 1>>
2>>
# 将正确和错误信息同时保留到一个文件
方法一：
[root@CentOS74 ~]# echo "yes" >> info.log 2>> info.log 
[root@CentOS74 ~]# ech "yes" >> info.log 2>> info.log 
[root@CentOS74 ~]# cat info.log 
yes
-bash: ech: command not found

方法二：
[root@CentOS74 ~]# echo "yes" &>> info.log
[root@CentOS74 ~]# ech "yes" &>> info.log
[root@CentOS74 ~]# cat info.log 
yes
-bash: ech: command not found

方法三：
[root@CentOS74 ~]# echo "yes" >> info.log 2>&1
[root@CentOS74 ~]# echp "yes" >> info.log 2>&1
[root@CentOS74 ~]# cat info.log 
yes
-bash: echp: command not found

<
<<
```

4. 通配符

```bash
# *

# {}：生成序列
[root@CentOS74 ~]# echo a{1..5..2}
a1 a3 a5
[root@CentOS74 ~]# echo a{1..5..2}{b,c}
a1b a1c a3b a3c a5b a5c
[root@CentOS74 ~]# echo info.log{.bak,.ori}
info.log.bak info.log.ori
[root@CentOS74 ~]# echo info.log{,.ori}
info.log info.log.ori
```

### 正则表达式分类

#### 基础正则

1. 基础正则符号

```bash
# ^：以什么开头
    ll /etc | grep "^d"
# $：以什么结尾
# ^$：空行
# .：匹配任意一个字符
# *：匹配前一个字符连续出现了0次或者多次
# .*：匹配所有信息
# \：转义
# []：匹配多个单个字符
    [abc]：表示匹配包含a或b或c
    ^[abc]：表示匹配以a或b或c开头
    [a-zA-Z0-9]：匹配所有字母数字
    [a-Z]：匹配大小写字母(grep/egrep能用)
    
# [^]：排除括号中信息
    [^abc]：表示排除包含a或b或c信息的字符
    ^[^abc]：表示排除以a或b或c开头的行（不包含空行）
```



#### 扩展正则

1. 扩展正则符号

```bash
# +：匹配前一个字符连续出现了1次或者多次

# |：用于匹配多个信息
    grep -E "aaa|bbb" file

# ()：后项引用前项，引用()中的内容
    [root@CentOS74 ~]# echo aabb | sed -r "s/(aa)(bb)/\1cc\2/g"
    aaccbb

# {}：
1. x{n}：表示前一个字符正好出现n次
2. x{n,m}：表示前一个字符至少出现n次，最多出现m次
3. x{n,}：表示前一个字符至少出现n次，最多出现多少次不限
4. x{,m}：表示前一个字符至少连续出现0次，最多出现m次

# ?：匹配前一个字符出现0或者1次
```

### 实用案例

#### 批量创建/删除用户

1. 批量创建用户

```bash
# 批量创建用户
echo test{01..10} | xargs -n1 | sed -r "s/(.*)/useradd \1/g" | bash

# 批量创建用户，并且给用户设置密码
seq -w 10 | sed -r "s/(.*)/useradd test\1;echo 12345 | passwd --stdin test\1/g | bash"
```

2. 批量删除用户

```bash
echo test{01..10} | xargs -n1 | sed -r "s/(.*)/userdel -r \1/g" | bash

seq -w 10 | sed -r "s/(.*)/userdel -r test\1/g" | bash
```



## 文件属性

```bash
[root@CentOS74 ~]# ls -li
total 0
1127607 drwxr-xr-x 2 root root 6 Feb 18 19:21 dir
1226852 lrwxrwxrwx 1 root root 8 Feb 18 19:21 file -> file.txt
 154754 -rw-r--r-- 1 root root 0 Feb 18 19:20 file.txt
```



### 时间属性

### 硬链接

```bash
# 作用：读取文件的多个入口
```

## 重要目录

### /etc/skel

该目录是样板目录，目录中的文件，会在新用户被创建的时候，自动添加到用户家目录中

```bash
[root@CentOS74 ~]# ll -a /etc/skel/
-rw-r--r--  1 root root   18 Apr 11  2018 .bash_logout
-rw-r--r--  1 root root  193 Apr 11  2018 .bash_profile
-rw-r--r--  1 root root  231 Apr 11  2018 .bashrc
```

**作用**

1. 在该目录下存储运维操作规范说明文件
2. 恢复家目录下被删掉的样板文件

## 磁盘分区



# Linux系统命令

## 用户和组管理

### useradd添加用户

**useradd命令相关参数**

```bash
useradd [options] 用户名
-b, --base-dir BASE_DIR # 指定家目录
-M # 不创建家目录
-s SHELL # 指定使用的shell，可以查看/etc/shells获取支持的shell
-u UID # 指定用户UID数值信息
-g GROUP # 指定用户所属的主要组信息，可以是组名，也可以是GID
-G GROUPS # 指定用户所属附属组
-c COMMENT # 注释信息
-k SKEL_DIR # 指定骨架目录，默认是/etc/skel/
-p PASSWORD # 指定新建用户的密码
```

**useradd使用**

1. 创建虚拟用户

```bash
useradd -M -s /sbin/nologin
```

### usermod修改用户信息

**usermod命令相关参数**

```bash
-s SHELL # 修改用户登录的shell
-g GROUP # 修改用户的主要组
-G GROUPS # 修改用户的附属组
-c COMMENT # 修改用户注释信息
-u UID # 修改用户的uid
```

**usermod使用**

### userdel删除用户

**userdel命令相关参数**

```bash
-r # remove home directory and mail spool
```

### passwd设置密码

```bash
# 免交互设置密码
echo PASSWD | passwd --stdin USER

# 交互式设置
passwd 用户名
```

### groupadd、groupmod、groupdel

### chown修改文件属主属组

```bash
-R 递归修改
chown 属主:属组 文件
```

## 用户登录信息

### w、who命令查看当前登录用户

```bash
[root@CentOS74 ~]# w
 00:33:18 up 3 days,  2:15,  2 users,  load average: 0.00, 0.01, 0.05
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
root     pts/0    111.199.188.82   Sat18    1:50   0.74s  0.36s -bash
root     pts/1    111.199.188.82   15:18    6.00s  0.36s  0.36s -bash
01       02       03               04       05          06      07

01: 当前登录的用户
02: 登录的方式，可以通过echo "xxx" > /dev/tty1、echo "xxx" > /dev/pts/x来给对方发消息
    pts/x: 远程登录的方式。
    tty1: 本地登录。
03: 连接服务器的源地址
04：登录时间
05：空闲时间
06：用户操作系统，消耗的CPU时间
07：用户在干什么



[root@CentOS74 ~]# who
root     pts/0        2020-02-29 18:22 (111.199.188.82)
root     pts/1        2020-03-01 15:18 (111.199.188.82)
```



## 权限管理

### chmod

```bash
chmod u+r FILE
chmod u-r FILE
chmod u=rw FILE
chmod 755 FILE
```

### sudo用户提权

```bash
visudo
## Allow root to run any commands anywhere 
root    ALL=(ALL)   ALL
用户名  ALL=(ALL)   /usr/sbin/useradd, /usr/sbin/userdel
01      02          03

01: 授权的用户
02：
03：授权的命令
    1) 授权单个命令或者多个命令
        /usr/sbin/useradd, /usr/sbin/userdel
    2) 授权目录，使用!排除命令
        /usr/sbin/*, !/usr/sbin/visudo, /usr/bin/*
```



## 软件安装和卸载

### yum

```bash
# 查看命令属于哪个软件包
yum provides locate

# 安装软件包，可以解决依赖关系
yum install 软件包名称

# 卸载软件包
yum erase 软件包名称  # 不建议使用
```

### rpm

```bash
# 安装软件包，无法解决依赖关系
rpm -ivh 软件包名称.rpm

# 卸载软件包
rpm -e 软件名称 --nodeps # 推荐使用
```

## 硬件信息查看

### CPU

```bash
[root@CentOS74 ~]# cat /proc/cpuinfo 
processor	: 0 # 
vendor_id	: GenuineIntel
cpu family	: 6
model		: 85
model name	: Intel(R) Xeon(R) Gold 6148 CPU @ 2.40GHz
stepping	: 4
microcode	: 0x1
cpu MHz		: 2394.370
cache size	: 16384 KB
physical id	: 0
siblings	: 1
core id		: 0
cpu cores	: 1
apicid		: 0
initial apicid	: 0
fpu		: yes
fpu_exception	: yes
cpuid level	: 13
wp		: yes
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ss syscall nx pdpe1gb rdtscp lm constant_tsc arch_perfmon rep_good nopl xtopology eagerfpu pni pclmulqdq ssse3 fma cx16 pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand hypervisor lahf_lm abm 3dnowprefetch fsgsbase tsc_adjust bmi1 hle avx2 smep bmi2 erms invpcid rtm mpx avx512f avx512dq rdseed adx smap clflushopt clwb avx512cd avx512bw avx512vl xsaveopt xsavec xgetbv1 arat
bogomips	: 4788.74
clflush size	: 64
cache_alignment	: 64
address sizes	: 46 bits physical, 48 bits virtual
power management:
```

```bash

```



### 内存

## 系统信息查看

### free

```bash
[root@CentOS74 ~]# free -h
              total        used        free      shared  buff/cache   available
Mem:           3.7G        173M        3.1G        8.8M        449M        3.3G
Swap:            0B          0B          0B

# 参数说明
total：表示系统总内存
used：表示应用程序已经使用的内存
free：表示当前还没有被使用的内存
shared：表示共享链接库使用的内存
buff/cache：表示系统的page cache和buffer使用到的内存
available：表示应用程序还可以申请到的内存。

# 计算说明
total = used + free + buff/cache
available <= free + buff/cache # 小于是因为系统的一下page或者cache是不能回收的
```

## 时间相关

### date

```bash
date
# 设置系统时间
date -s
# 
```



## 文件信息查找

### file

### find

```bash
# 语法
find 查找路径 [-type 文件类型] [-maxdepth 最大层级] [-size 文件大小] [-name|-iname 文件名|通配符]
-type 文件类型：
    b：块文件
    c：字符文件
    d：目录
    p：管道文件
    f：一般文件
    l：链接文件
    s：socket文件
    
-size 文件大小：
    b：512字节块
    c：1字节
    w：for two-byte words
    k：1KB，1024字节
    M：1MB
    G：1GB

-maxdepth 最大层级
    0：所有层级

# 精确查找文件
find 查找路径 -type f -name "文件名"

# 模糊查找
find 查找路径 -type f -name "文件名通配符"

# 忽略文件名大小写
find 查找路径 -type f -iname "文件名"

# 根据文件大小查找
find 查找路径 -type f -size +100k  # 查找大于100KB的文件
find 查找路径 -type f -size -100k  # 查找小于100KB的文件

# 根据目录层级查找
find 查找路径 -type f -maxdepth 1 -name "文件名"

# 根据文件权限查找
find 查找路径 -type f -perm 644

# 根据文件时间查找
-mmin n
n < 0：修改时间在最近n分钟的文件
n > 0：修改时间在n分钟之前的文件
-mtime n
n < 0：修改时间在最近n天的文件
n > 0：修改时间在n天之前的文件
-amin n
-atime n
-cmin n
-ctime n

# 对查找出的文件做操作---删除
find ./ -name "*.txt" -delete
find ./ -name "test*" -exec rm -rf {} \;
find ./ -name "test*" | xargs rm
# 对查找出的文件做操作---移动
find ./ -name "test*" -exec mv {} 目标目录\;
find ./ -name "test*" | xargs -i cp {} 目标目录
# 对查找出的文件做操作---拷贝
find ./ -name "test*" -exec mv {} 目标目录\;
find ./ -name "test*" | xargs -i cp {} 目标目录
```



### which

```bash
# which查找文件只会去PATH环境变量下的路径中查找
# 只查找可执行文件
```



### whereis

```bash
# whereis查找文件是根据文件索引数据库来查找
# 查找二进制文件、源文件、man文件
```

### locate和updatedb

```bash

```

### tree

```bash
# 选项
-L level：
-d：只显示目录

# 查看目录树，指定层级
tree -L 2 /
```



## 文件打包压缩

### zip/unzip

### gzip/gunzip

### tar

```bash
# 只打包不压缩
# 打包压缩
tar -zcvf xxx.tar.gz 要压缩的数据文件

# 解压缩
tar -zxvf xxx.tar.gz

# 不解压list压缩包中文件
tar -tf xxx.tar.gz

# 在压缩过程中排除指定文件，注意前后使用的路径前缀必须一致
tar -zcvf xxx.tar.gz ./ --exclude=./xxx
tar -zcvf xxx.tar.gz /xxx --exclude=/xxx/xxx

# 在压缩过程中排出多个文件，可以将要排除的文件路径放入指定文件中
tar -zcvf xxx.tar.gz ./ --exclude-from=./xxx.txt
```



## 三剑客 ##

grep：过滤筛选信息

sed：修改替换文件内容 擅长对文件中的行进行操作

awk：擅长统计分析文件内容，还擅长对文件中的列进行操作

### grep ###

```
-B, --before-context=NUM 将过滤行以及过滤行的上NUM行信息都打印出来
-A, --after-context=NUM 将过滤行以及过滤行的下NUM行信息都打印出来
-C, --context=NUM 将过滤行的上下NUM行信息都打印出来
-c, --count 统计指定信息出现的行数
```

### sed ###

```
1. 过滤信息，p表示将过滤信息输出，-n表示取消默认输出（默认文件内容会输出到屏幕）
sed -n "/oldboy/p" test.txt

2. 替换
sed -i 's/xxx/xxx/g' 文件名
```

### awk ###

```
1. 过滤信息
awk "/oldboy/" test.txt
```

## tail

### 参数

```bash
-f 追踪文件信息：文件被删除或者移动走，需要进行重新追踪
-F 追踪文件信息：文件被删除或者移动走，不需要进行重新追踪，只要文件恢复回来会继续追踪
```



### 用法