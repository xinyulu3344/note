## 交互式接口shell

显示当前使用的shell

```bash
echo ${SHELL}
```

查看当前系统使用的所有shell

```bash
]# cat /etc/shells 
/bin/sh
/bin/bash
/sbin/nologin
/usr/bin/sh
/usr/bin/bash
/usr/sbin/nologin
/bin/tcsh
/bin/csh
```

## 设置主机名

查看主机名

```bash
hostname 
```

临时设置主机名

```bash
hostname NAME
```

永久修改

CentOS7及之后版本

```bash
hostnamectl set-hostname NAME
```

CentOS6

```
vim /etc/hostname
```

范例

```bash
]# hostname bj-yz-k8s-node1-100-10.jdcloud.com
```

注意：主机名不要使用下划线

## 显示提示符

```bash
]# echo $PS1
[\u@\h \W]\$
```

提示符格式说明

| 提示符 | 解释             |      |
| ------ | ---------------- | ---- |
| \e     | 控制符\033       |      |
| \u     | 当前用户         |      |
| \h     | 主机名简称       |      |
| \H     | 主机名全称       |      |
| \w     | 当前工作目录     |      |
| \W     | 当前工作目录基名 |      |
| \t     | 24小时时间格式   |      |
| \T     | 12小时时间格式   |      |
| !      | 命令历史数       |      |
| #      | 开机后命令历史数 |      |

范例

```bash
PS1="\[\e[1;32m\][\t \[\e[1;33m\]\u\[\e[35m\]@\h\[\e[1;31m\] \W\[\e[1;32m\]]\\[\e[0m\]\\$"
```

## 内外部命令

判断是否是内部命令

```bash
type -a COMMAND
```

列出所有内部命令

```bash
enable
help
```

禁用内部命令

```bash
enable -n COMMAND
```

查看外部命令路径

```bash
which COMMAND
whereis COMMAND
```

## hash 缓存表

执行过的shell命令路径会缓存住，下次可以直接找到命令

查看缓存的命令路径列表

```bash
]# hash
hits    command
   2    /bin/hostname
   1    /sbin/runlevel
   2    /bin/man
   1    /bin/ls
```

清除缓存

```bash
]# hash -r
```

## 命令别名

列出所有别名

```bash
alias
```

临时定义别名

```bash
alias cdnet="cd /etc/sysconfig/network-scripts/"
```

取消指定别名

```bash
unalias cdnet
```

取消所有别名

```bash
unalias -a
```

持久保存

```bash
vim .bashrc
```

如果别名同原命令同名，如果要执行原命令，可以加个`\`

```bash
]# \ls
```

## 查看CPU信息

```bash
]# lscpu
Architecture:          x86_64
CPU op-mode(s):        32-bit, 64-bit
Byte Order:            Little Endian
CPU(s):                2
On-line CPU(s) list:   0,1
Thread(s) per core:    2
Core(s) per socket:    1
Socket(s):             1
NUMA node(s):          1
Vendor ID:             GenuineIntel
CPU family:            6
Model:                 106
Model name:            Intel(R) Xeon(R) Platinum 8338C CPU @ 2.60GHz
Stepping:              6
CPU MHz:               2594.138
BogoMIPS:              5188.27
Virtualization:        VT-x
Hypervisor vendor:     KVM
Virtualization type:   full
L1d cache:             32K
L1i cache:             32K
L2 cache:              4096K
L3 cache:              16384K
NUMA node0 CPU(s):     0,1
Flags:                 fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ss ht syscall nx pdpe1gb rdtscp lm constant_tsc arch_perfmon rep_good nopl xtopology eagerfpu pni pclmulqdq vmx ssse3 fma cx16 pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand hypervisor lahf_lm abm 3dnowprefetch tpr_shadow vnmi flexpriority ept vpid fsgsbase tsc_adjust bmi1 avx2 smep bmi2 erms invpcid avx512f avx512dq rdseed adx smap avx512ifma clflushopt clwb avx512cd sha_ni avx512bw avx512vl xsaveopt xsavec xgetbv1 arat avx512vbmi avx512_vpopcntdq
```

```bash
]# cat /proc/cpuinfo 
processor       : 0
vendor_id       : GenuineIntel
cpu family      : 6
model           : 106
model name      : Intel(R) Xeon(R) Platinum 8338C CPU @ 2.60GHz
stepping        : 6
microcode       : 0x1
cpu MHz         : 2594.138
cache size      : 16384 KB
physical id     : 0
siblings        : 2
core id         : 0
cpu cores       : 1
apicid          : 0
initial apicid  : 0
fpu             : yes
fpu_exception   : yes
cpuid level     : 27
wp              : yes
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ss ht syscall nx pdpe1gb rdtscp lm constant_tsc arch_perfmon rep_good nopl xtopology eagerfpu pni pclmulqdq vmx ssse3 fma cx16 pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand hypervisor lahf_lm abm 3dnowprefetch tpr_shadow vnmi flexpriority ept vpid fsgsbase tsc_adjust bmi1 avx2 smep bmi2 erms invpcid avx512f avx512dq rdseed adx smap avx512ifma clflushopt clwb avx512cd sha_ni avx512bw avx512vl xsaveopt xsavec xgetbv1 arat avx512vbmi avx512_vpopcntdq
bogomips        : 5188.27
clflush size    : 64
cache_alignment : 64
address sizes   : 46 bits physical, 57 bits virtual
power management:

processor       : 1
vendor_id       : GenuineIntel
cpu family      : 6
model           : 106
model name      : Intel(R) Xeon(R) Platinum 8338C CPU @ 2.60GHz
stepping        : 6
microcode       : 0x1
cpu MHz         : 2594.138
cache size      : 16384 KB
physical id     : 0
siblings        : 2
core id         : 0
cpu cores       : 1
apicid          : 1
initial apicid  : 1
fpu             : yes
fpu_exception   : yes
cpuid level     : 27
wp              : yes
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ss ht syscall nx pdpe1gb rdtscp lm constant_tsc arch_perfmon rep_good nopl xtopology eagerfpu pni pclmulqdq vmx ssse3 fma cx16 pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand hypervisor lahf_lm abm 3dnowprefetch tpr_shadow vnmi flexpriority ept vpid fsgsbase tsc_adjust bmi1 avx2 smep bmi2 erms invpcid avx512f avx512dq rdseed adx smap avx512ifma clflushopt clwb avx512cd sha_ni avx512bw avx512vl xsaveopt xsavec xgetbv1 arat avx512vbmi avx512_vpopcntdq
bogomips        : 5188.27
clflush size    : 64
cache_alignment : 64
address sizes   : 46 bits physical, 57 bits virtual
power management:
```

## 查看内存信息

```bash
]# cat /proc/meminfo 
MemTotal:        3881640 kB
MemFree:          245440 kB
MemAvailable:    3058008 kB
Buffers:               0 kB
Cached:          2882272 kB
SwapCached:            0 kB
Active:          2066672 kB
Inactive:        1084708 kB
Active(anon):     283668 kB
Inactive(anon):    23060 kB
Active(file):    1783004 kB
Inactive(file):  1061648 kB
Unevictable:           0 kB
Mlocked:               0 kB
SwapTotal:             0 kB
SwapFree:              0 kB
Dirty:                16 kB
Writeback:             0 kB
AnonPages:        269156 kB
Mapped:           103984 kB
Shmem:             37620 kB
Slab:             297960 kB
SReclaimable:     257160 kB
SUnreclaim:        40800 kB
KernelStack:        4032 kB
PageTables:         8220 kB
NFS_Unstable:          0 kB
Bounce:                0 kB
WritebackTmp:          0 kB
CommitLimit:     1940820 kB
Committed_AS:    1728840 kB
VmallocTotal:   34359738367 kB
VmallocUsed:       32300 kB
VmallocChunk:   34359685116 kB
HardwareCorrupted:     0 kB
AnonHugePages:     63488 kB
HugePages_Total:       0
HugePages_Free:        0
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:       2048 kB
DirectMap4k:      124752 kB
DirectMap2M:     4069376 kB
DirectMap1G:     2097152 kB
```

## 查看磁盘信息

```bash
]# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
vda    253:0    0   50G  0 disk 
├─vda1 253:1    0    1M  0 part 
├─vda2 253:2    0  100M  0 part 
└─vda3 253:3    0 49.9G  0 part /
vdb    253:16   0  100G  0 disk 
└─vdb1 253:17   0  100G  0 part /mnt
vdc    253:32   0  600G  0 disk 
└─vdc1 253:33   0  600G  0 part /vdc
```

```bash
]# cat /proc/partitions 
major minor  #blocks  name

 253        0   52428800 vda
 253        1       1024 vda1
 253        2     102400 vda2
 253        3   52324335 vda3
 253       16  104857600 vdb
 253       17  104856576 vdb1
 253       32  629145600 vdc
 253       33  629144576 vdc1
```

## 查看系统版本信息

查看内核版本

```bash
]# uname -r
3.10.0-693.el7.x86_64

]# uname -a
Linux CentOS74 3.10.0-693.el7.x86_64 #1 SMP Tue Aug 22 21:09:27 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
```

查看操作系统发行版本

```bash
]# cat /etc/redhat-release 
CentOS Linux release 7.4.1708 (Core) 

]# cat /etc/os-release 
NAME="CentOS Linux"
VERSION="7 (Core)"
ID="centos"
ID_LIKE="rhel fedora"
VERSION_ID="7"
PRETTY_NAME="CentOS Linux 7 (Core)"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:centos:centos:7"
HOME_URL="https://www.centos.org/"
BUG_REPORT_URL="https://bugs.centos.org/"

CENTOS_MANTISBT_PROJECT="CentOS-7"
CENTOS_MANTISBT_PROJECT_VERSION="7"
REDHAT_SUPPORT_PRODUCT="centos"
REDHAT_SUPPORT_PRODUCT_VERSION="7"
```

## 时间

查看系统时间

```bash
]# date
Sun Apr  9 21:40:46 CST 2023
```

查看硬件时间

```bash
]# clock
Sun 09 Apr 2023 09:44:24 PM CST  -0.929108 seconds
```

使用硬件时间校准系统时间

```bash
]# clock -s
```

使用系统时间校准硬件时间

```bash
]# clock -w
```

查看当前时区

```bash
]# ll /etc/localtime 
lrwxrwxrwx 1 root root 35 Jan 22  2018 /etc/localtime -> ../usr/share/zoneinfo/Asia/Shanghai
```

```bash
# CentOS7及以上支持
]# timedatectl status
      Local time: Sun 2023-04-09 21:49:36 CST
  Universal time: Sun 2023-04-09 13:49:36 UTC
        RTC time: Sun 2023-04-09 13:49:35
       Time zone: Asia/Shanghai (CST, +0800)
     NTP enabled: no
NTP synchronized: yes
 RTC in local TZ: no
      DST active: n/a
```

查看所有时区

```bash
timedatectl list-timezones
```

修改时区

```bash
timedatectl set-timezone Asia/Shanghai
```

## 日历

查看当月日历

```bash
cal
```

查看当年日历

```bash
cal -y
```

查看某月日历

```bash
cal 10 2023
```

## 用户登录信息查看

- whoami: 显示当前登录的有效用户
- who：系统当前所有的登录会话
- w：系统当前所有的登录会话

```bash
]# whoami
root

]# who
root     tty1         2023-03-30 19:32
xinyulu  pts/0        2023-04-09 19:19 (192.168.21.9)

]# w
 22:04:33 up 17 days, 14:32,  2 users,  load average: 0.00, 0.02, 0.05
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
root     tty1                      30Mar23  4days  0.06s  0.06s -bash
xinyulu  pts/0    192.168.21.9     19:19    1.00s  0.34s  0.02s sshd: xinyulu
```

## screen

查看所有screen会话列表

```bash
]# screen -ls
There is a screen on:
        29761.vnc       (Attached)
1 Socket in /var/run/screen/S-root.
```

创建新会话

```bash
]# screen -S vnc
```

加入screen会话

```bash
]# screen -x vnc
```

退出并关闭screen会话

```bash
]# exit
```

剥离当前screen会话

```bash
]# Ctrl+a, d
```

## tmux

tmux是一个终端复用器，类似screen，但是更易用，也更强大

功能如下：

- 允许在单个窗口中，同时访问多个会话。
- 可以让新窗口接入已经存在的会话
- 允许每个会话有多个连接窗口，因此可以多人实时共享会话
- 支持窗口任意垂直和水平拆分

安装

```bash
yum install tmux
```

启动与退出

```bash
tmux
exit
```

窗格快捷键

```bash
ctrl+b, " 上下拆分窗格
ctrl+b, % 左右拆分窗格
ctrl+b, <arrow key> 光标切换到其它窗格
ctrl+b, ; 光标切换到上一个窗格
ctrl+b, o 光标切换到下一个窗格
ctrl+b, { 当前窗格左移
ctrl+b, } 当前窗格右移
ctrl+b, ctrl+o 当前窗格上移
ctrl+b, alt+o 当前窗格下移
ctrl+b, x 关闭当前窗格
ctrl+b, ! 将当前窗格拆分为一个独立窗口
ctrl+b, z 当前窗格全屏显示，再使用一次会变回原来大小
ctrl+b, ctrl+<arrow key> 按箭头方向调整窗格大小
ctrl+b, q 显示窗格编号
```

列出所有Tmux命令及其参数

```bash
tmux list-commands
```

列出所有快捷键，及其对应的Tmux命令

```bash
tmux list-keys
```

## echo

```bash
-n 不输出结尾的换行
-E (默认)不支持\解释功能
-e 启用\字符的解释功能
```

如果使用-e选项，则出现以下字符，则特别加以处理，而不会将它当成一般文字输出

```bash
\a 发出警告
\b 退格
\c 最后不加上换行符号
\e escape，相当于\0333
\n 换行且光标移至行首，但不换行
\t 插入tab
\0nnn插入nnn(八进制)所代表的ASCII字符
\xHH插入HH(十六进制)所代表的ASCII数字
```

