## pid_max的设置

### 原理分析

关于pid_max，是在`include/linux/threads.h`中定义的。具体定义如下

```c
/*
 * This controls the default maximum pid allocated to a process
 */
#define PID_MAX_DEFAULT (CONFIG_BASE_SMALL ? 0x1000 : 0x8000)

/*
 * A maximum of 4 million PIDs should be enough for a while.
 * [NOTE: PID/TIDs are limited to 2^29 ~= 500+ million, see futex.h.]
 */
#define PID_MAX_LIMIT (CONFIG_BASE_SMALL ? PAGE_SIZE * 8 : \
    (sizeof(long) > 4 ? 4 * 1024 * 1024 : PID_MAX_DEFAULT))

```

如果当时编译内核的时候，配置了`CONFIG_BASE_SMALL`选项，那么默认的pid max为0x1000，也就是4096个，否则为0x8000，也就是32768个。

当然，这个pid max是可以修改的。如何修改呢？

```bash
# 查看pid_max
]# sysctl kernel.pid_max
kernel.pid_max = 32768

# 修改pid_max
]# sysctl -w kernel.pid_max=4194304
kernel.pid_max = 4194304
```

那么这个值能设置多大呢？从上面的源码，可以看出来，如果配置了`CONFIG_BASE_SMALL`选项，则最大限制为`PAGE_SIZE * 8`，否则如果是64位系统`(sizeof(long)=8)`，则最大限制为`4 * 1024 * 1024 = 4194304`，如果是32位系统`(sizeof(long)=4)`，则最大限制为PID_MAX_DEFAULT，也就是32768

如何判断编译的时候，是否配置了`CONFIG_BASE_SMALL`选项呢？

```bash
]# cat /boot/config-3.10.0-693.el7.x86_64 |grep BASE_SMALL
CONFIG_BASE_SMALL=0
```

为0表示没有配置，为1表示配置了。

如何查看系统`PAGE_SIZE `呢？

```bash
]# getconf PAGESIZE
4096
```

### 总结

**pid最大值**

|          | CONFIG_BASE_SMALL=0 | CONFIG_BASE_SMALL=1 |
| -------- | ------------------- | ------------------- |
| 32位系统 | 32768               | PAGE_SIZE * 8       |
| 64位系统 | 4194304             | PAGE_SIZE * 8       |

**查看修改pid_max**

```bash
# 查看pid_max
]# sysctl kernel.pid_max
kernel.pid_max = 32768

# 修改pid_max
]# sysctl -w kernel.pid_max=4194304
kernel.pid_max = 4194304
```

**编译时是否配置CONFIG_BASE_SMALL**

```bash
# 为0表示没有配置，为1表示配置了。
]# cat /boot/config-3.10.0-693.el7.x86_64 |grep BASE_SMALL
CONFIG_BASE_SMALL=0
```

**查看系统`PAGE_SIZE `**

```bash
]# getconf PAGESIZE
4096
```

