# shell脚本编程

## bash

```bash
-n 检查语法，只读取不执行
-x 进入跟踪方式，显示所执行的每一条命令
-c 从字符串中读取命令
```

## 三种错误

- 语法错误，会导致后续的命令不继续执行，可以通过`bash -n`检查出来
- 命令错误，后续的命令还会继续执行，用`bash -n`无法检查出来，但是可以通过`bash -x`观察
- 逻辑错误

## 变量

### 变量类型

- 内置变量，如：PS1，PATH，HISTSIZE，UID，HOSTNAME，PPID，BASHPID 
- 用户自定义变量

### 常见内置变量

| 变量名  | 解释                     | 默认值 |
| ------- | ------------------------ | ------ |
| SHLVL   | bash的嵌套层数           |        |
| PATH    |                          |        |
| USER    |                          |        |
|         |                          |        |
|         |                          |        |
|         |                          |        |
|         |                          |        |
|         |                          |        |
|         |                          |        |
|         |                          |        |
| _       | 前一个命令的最后一个参数 |        |
| LANG    |                          |        |
| BASHPID |                          |        |
| PPID    |                          |        |



### Shell中变量命名法则

- 不能使用保留字：if、for、then、else
- 只能使用数字、字母和下划线，不能以数字开头
- 见名知义，用英文单词命名，并体现实际作用，不要用简写
- 统一命名规则：驼峰命名法
- 变量名大写
- 局部变量小写
- 函数名小写

### 变量定义和引用

变量的生效范围等标准划分变量类型

- 普通变量：生效范围为当前shell进程；对当前shell之外的其它shell进程，包括当前shell的子shell进程均无效
- 环境变量：生效范围为当前shell进程及其子shell进程
- 本地变量：生效范围为当前shell进程中某代码片段，通常指函数

**变量赋值：**

```bash
name='value'
name=123
name="$USER"
name=`COMMAND` 或 name=$(COMMAND)
```

变量赋值是临时生效的，当退出终端后，变量会自动删除。脚本中的变量会在脚本退出后删除。

**变量引用：**

```bash
$name
${name}
```

弱引用和强引用

- "$name" 弱引用，其中的变量引用会被替换为变量值
- '$name' 强引用，其中的变脸引用不会被替换为变量值，而是保持原样

> 注意，当变量值为多行的时候，引用变量是否加双引号有区别。加双引号，多行输出，不加单行输出。如：

```bash
]# NUMS=`seq 3`
]# 
]# echo "$NUMS"
1
2
3
]# 
]# echo $NUMS
1 2 3
```

### 删除变量

```bash
unset 变量名1 变量名2
unset name1 name2
```

### 环境变量

环境变量：

- 可以使子进程（包括孙子进程）继承父进程的变量，但是无法让父进程使用子进程的变脸。
- 一旦子进程修改从父进程继承的变量，将会把新的值传递给孙子进程
- 一般只在系统配置文件中使用，脚本中使用较少

**环境变量声明和赋值：**

```bash
# 声明并赋值
export name=value
declare -x name=value

# 赋值再声明
name=value
export name
```

**变量引用：**

```bash
$name
${name}
```

> 注意：花括号可以用在多个变量连起来写时，分割的作用

```bash
]# NAME=zhangsan
]# AGE=10
]# echo $NAME$AGE
zhangsan10
]# 
]# echo $NAME-$AGE
zhangsan-10
]# 
]# echo $NAME_$AGE
10
]# 
]# echo ${NAME}_$AGE
zhangsan_10
```

**显示所有环境变量**

**显示所有环境变量**

```bash
env
printenv
export
declare -x
```

