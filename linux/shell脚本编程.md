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

| 变量名  | 解释                                   | 默认值                       |
| ------- | -------------------------------------- | ---------------------------- |
| SHLVL   | bash的嵌套层数                         |                              |
| PATH    |                                        |                              |
| USER    |                                        |                              |
|         |                                        |                              |
|         |                                        |                              |
|         |                                        |                              |
|         |                                        |                              |
|         |                                        |                              |
|         |                                        |                              |
|         |                                        |                              |
| _       | 前一个命令的最后一个参数               |                              |
| LANG    |                                        |                              |
| BASHPID |                                        |                              |
| PPID    |                                        |                              |
| IFS     | 内部字段分隔符。用于分隔字符串的界定符 | 空白字符（换行、制表、空格） |



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
[root@CentOS74 ~]# NUMS=`seq 3`
[root@CentOS74 ~]# 
[root@CentOS74 ~]# echo "$NUMS"
1
2
3
[root@CentOS74 ~]# 
[root@CentOS74 ~]# echo $NUMS
1 2 3
```

**利用变量实现动态命令**

```bash
[root@CentOS74 ~]# CMD=hostname
[root@CentOS74 ~]# $CMD
CentOS74
```



### 删除变量

```bash
unset 变量名1 变量名2
unset name1 name2
```

### 环境变量

环境变量：

- 可以使子进程（包括孙子进程）继承父进程的变量，但是无法让父进程使用子进程的变量。
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
[root@CentOS74 ~]# NAME=zhangsan
[root@CentOS74 ~]# AGE=10
[root@CentOS74 ~]# echo $NAME$AGE
zhangsan10
[root@CentOS74 ~]# 
[root@CentOS74 ~]# echo $NAME-$AGE
zhangsan-10
[root@CentOS74 ~]# 
[root@CentOS74 ~]# echo $NAME_$AGE
10
[root@CentOS74 ~]# 
[root@CentOS74 ~]# echo ${NAME}_$AGE
zhangsan_10
```

**显示所有环境变量**

```bash
env
printenv
export
declare -x
```

### 只读变量

只能声明定义，但后续不能修改和删除

**声明只读变量：**

```bash
readonly PI=3.1415926
declare -r PI=3.1415926
```

**查看只读变量**

```bash
readonly [-p]
declare -r
```

### 位置变量

在bash shell中内置的变量，在脚本代码中，通过命令行传递给脚本的参数

| 位置变量      | 解释                                         |      |
| ------------- | -------------------------------------------- | ---- |
| `$1, $2, ...` | 对应第1个，第2个等参数                       |      |
| `$0`          | 命令本身，包括路径                           |      |
| `$*`          | 传递给脚本的所有参数，全部参数合为一个字符串 |      |
| `$@`          | 传递给脚本的所有参数，每个参数为独立字符串   |      |
| `$#`          | 传递给脚本的参数的个数                       |      |

> 注意：`$@和$*`只在被双引号包起来的时候才会有差异

**清空所有位置变量**

```bash
set --
```

### 退出状态码变量

进程执行后，将使用`$?`来保存状态码的相关数字。取值范围0-255

```bash
0     代表成功
1-255 代表失败
```

通过exit命令自定义退出状态码

```bash
exit 0
```

> 注意：
>
> - 脚本中一旦遇到exit命令，脚本会立即终止。
> - 如果未使用exit命令指定退出码，最终退出码由脚本执行的最后一个命令的状态码决定

### 展开命令行

**展开命令执行顺序**

```bash
把命令行分成单个命令词
展开别名
展开花括号声明{}
展开~声明
命令替换$()和``
再次把命令行分成命令词
展开文件通配符*、?、[abc]等
准备I/O重定向<, >
运行命令
```

### 脚本安全和set

**$-变量**

```bash
]# echo $-
himBH
```

| 字母 | 解释                                                         |      |
| ---- | ------------------------------------------------------------ | ---- |
| h    | hashall，打开选项后，Shell会将命令所在的路径hash下来，避免每次都要查询。通过`set +h`将h选项关闭 |      |
| i    | interactive-comments。包含这个选项，说明当前的shell是一个交互式的shell。在脚本中，i选项是关闭的 |      |
| m    | monitor，打开监控模式，就可以通过job control来控制进程的停止、继续、后台或者前台执行 |      |
| B    | braceexpand，大括号扩展                                      |      |
| H    | history，H选项打开，可以展开历史列表中的命令，可以通过`!`感叹号来完成，例如`!!`返回最近的一个历史命令，`!n`返回第n个历史命令 |      |

**set命令实现脚本安全**

```bash
-u 在扩展一个没有设置的变量时，显示错误信息，等同于set -o nounset
-e 如果有一个命令退出状态码不是0，则立即中止脚本
-o
-x 在执行过程中打印出命令参数。
```

`-o`选项

```bash
[root@CentOS74 ~]# set -o
allexport       off
braceexpand     on
emacs           on
errexit         off
errtrace        off
functrace       off
hashall         on
histexpand      on
history         on
ignoreeof       off
interactive-comments    on
keyword         off
monitor         on
noclobber       off
noexec          off
noglob          off
nolog           off
notify          off
nounset         off
onecmd          off
physical        off
pipefail        off
posix           off
privileged      off
verbose         off
vi              off
xtrace          off
```

## 格式化输出printf

**常用格式替换符**

| 替换符 | 功能                                                         |
| ------ | ------------------------------------------------------------ |
| %s     | 字符串                                                       |
| %f     | 浮点格式                                                     |
| %b     | 相对应的参数中包含转义字符时，可以使用此替换符进行替换，对应的转义字符会被转义 |
| %c     | ASCII字符，即显示对应参数的第一个字符                        |
| %d,%i  | 十进制整数                                                   |
| %o     | 八进制值                                                     |
| %u     | 无符号十进制值                                               |
| %x     | 十六进制值(a-f)                                              |
| %X     | 十六进制值(A-F)                                              |
| %%     | 表示%本身                                                    |

**常用转义字符**

| 转义符 | 功能                           |
| ------ | ------------------------------ |
| \a     | 警告字符，通常为ASCII的BEL字符 |
| \b     | 后退                           |
| \f     | 换页                           |
| \n     | 换行                           |
| \r     | 回车                           |
| \t     | 水平制表符                     |
| \v     | 垂直制表符                     |
| \      | 表示\本身                      |

## 算数运算

shell支持算术运算，但只支持整数，不支持小数

```bash
+ - * / % **
```

实现算术运算：

```bash
let var=算术表达式
var=$[算术表达式]
var=$((算术表达式))
var=$(expr arg1 arg2 arg3 ...)
declare -i var=数值
echo '算术表达式' | bc
```

内建的随机数生成器变量：

```bash
$RANDOM 取值范围：0-32767
```

增强型赋值：

```bash
i+=10
i-=10
*=
/=
%=
i++
++i
i--
--i
```

## 逻辑运算

| 运算符       | 说明                               |
| ------------ | ---------------------------------- |
| <<、>>       | 左移右移                           |
| &、\|、^、~  | 按位与、按位或、按位异或、按位取反 |
| ==、!=       | 相等、不等                         |
| <、<=、>、>= | 小于、小于等于、大于、大于等于     |
| &&、\|\|、!  | 逻辑与、逻辑或、逻辑非(取反)       |

## 条件测试命令

条件测试：判断某需求是否满足，需要由测试机制来实现，专用的测试表达式需要由测试命令辅助完成测试过程。

若真，则状态码变量`$?`返回0

若假，则状态码变量`$?`返回1

**条件测试命令**

```bash
test EXPRESSION <==> [ EXPRESSION ]
[[ EXPRESSION ]]
```

> 注意：EXPRESSION前后必须有空白字符

### 变量测试

```bash
-v VAR 判断变量是否存在
```

### 数值测试

```bash
arg1 OP arg2   Arithmetic tests.  OP is one of -eq, -ne, -lt, -le, -gt, or -ge

-eq ==
-ne !=
-lt <
-le <=
-gt >
-ge >=
```

示例：

```bash
[root@CentOS74 ~]# i=10
[root@CentOS74 ~]# j=8
[root@CentOS74 ~]# [ $i -gt $j ]
[root@CentOS74 ~]# echo $?
0
[root@CentOS74 ~]# 
[root@CentOS74 ~]# [ $i -lt $j ]
[root@CentOS74 ~]# 
[root@CentOS74 ~]# echo $?
1
```



### 字符串测试

```bash
test和[ ] 用法
-n "STRING"          字符串是否不空，不空为真，没定义或空为假
-z "STRING"          字符串是否为空，没定义或空为真，不空为假
STRING1 = STRING2    字符串相等为真，不等为假
STRING1 != STRING2   字符串不等为真，相等为假
STRING1 < STRING2    ASCII码是否大于
STRING1 > STRING2    ASCII码是否小于

[[ ]] 用法
STRING1 == PATTERN   STRING1是否和PATTERN通配符匹配。注意：此表达式用于[[ ]]中
STRING1 =~ PATTERN   STRING1是否被PATTERN扩展正则匹配。注意：此表达式用于[[ ]]中
```

通配符示例：

```bash
[root@CentOS74 ~]# FILE=test.log
[root@CentOS74 ~]# [[ "$FILE" == *.log ]]
[root@CentOS74 ~]# echo $?
0
[root@CentOS74 ~]# 
[root@CentOS74 ~]# [[ "$FILE" == *.txt ]]
[root@CentOS74 ~]# echo $?
1
```

正则示例：

```bash
[root@CentOS74 ~]# FILE=test.log
[root@CentOS74 ~]# [[ "$FILE" =~ \.txt$ ]]
[root@CentOS74 ~]# 
[root@CentOS74 ~]# echo $?
1
[root@CentOS74 ~]# 
[root@CentOS74 ~]# [[ "$FILE" =~ \.log$ ]]
[root@CentOS74 ~]# echo $?
0
```

注意：`[[ == ]]` 这种写法，`==`右侧的`*`想做通配符，就不要加""，只想做一个普通`*`字符，需要加""或转义。示例如下：

```bash 
[root@CentOS74 ~]# NAME="linux1"
[root@CentOS74 ~]# [[ "$NAME" == linux* ]]
[root@CentOS74 ~]# echo $?
0
[root@CentOS74 ~]# [[ "$NAME" == "linux*" ]]
[root@CentOS74 ~]# echo $?
1
[root@CentOS74 ~]# NAME="linux*"
[root@CentOS74 ~]# [[ "$NAME" == "linux*" ]]
[root@CentOS74 ~]# echo $?
0
```

### 文件测试

**存在性测试**

```bash
-a FILE        判断文件是否存在，建议用-e
-e FILE        判断文件是否存在
-b FILE        判断是否存在且是一个块文件
-c FILE        判断是否存在且是一个字符文件
-d FILE        判断是否存在且是一个目录
-f FILE        判断是否存在且是普通文件
-h FILE        判断是否存在且文件是一个软链接
-L FILE        判断是否存在且文件是一个软链接
-p FILE        判断是否存在且是一个管道文件
-S FILE        判断是否存在且文件是一个套接字
```

注意：如果是判断软链接，会去判断它指向的文件

**文件权限测试**

```bash
-r FILE        判断是否存在且对执行测试用户可读
-w FILE        判断是否存在且对执行测试用户可写
-x FILE        判断是否存在且对执行测试用户可执行
-u FILE        判断是否存在且拥有suid权限
-g FILE        判断是否存在且拥有sgid权限
-k FILE        判断是否存在且设置了粘滞位(sticky bit)
```

> 注意：这里的是否可读可写，是指最终的权限。比如root用户的权限，acl、特殊权限等，都会影响最终是否可读可写

**文件属性测试**

```bash
-s FILE        判断是否存在且非空
-t FD          判断文件描述符是否在某终端已经打开
-N FILE        文件自从上一次被读取之后是否被修改过
-O FILE        当前有效用户是否为文件属主
-G FILE        当前有效用户是否为文件属组
FILE1 -ef FILE2 FILE1是否为FILE2的硬链接
FILE1 -nt FILE2 FILE1是否比FILE2新(mtime)
FILE1 -ot FILE2 FILE1是否比FILE2旧(mtime)
```

## 关于()和{}

**man bash的官方描述**

> (list)
> list  is  executed in a subshell environment (see COMMAND EXECUTION ENVIRONMENT below).  Variable assignments and builtin commands that affect the shell's environment  do  not  remain  in  effect after the command completes.  The return status is the exit status of list.
> { list; }
> list is simply executed in the current shell environment.  list must be terminated with a newline or semicolon.  This is known as a group command.  The return status is the exit status  of  list. Note  that  unlike  the metacharacters ( and ), { and } are reserved words and must occur where a reserved word is permitted to be recognized.  Since they do not cause a word break, they must be separated from list by whitespace or another shell metacharacter.

(list) 会开启子shell，并且list中的变量赋值以及内部命令执行后，将不再影响后续的环境

{ list; } 不会开启子shell，在当前shell中运行，会影响当前shell环境。

示例：

```bash
[root@CentOS74 ~]# name=zhangsan;(echo $name; name=lisi; echo $name); echo $name
zhangsan
lisi
zhangsan
[root@CentOS74 ~]# name=zhangsan;{ echo $name; name=lisi; echo $name; }; echo $name;
zhangsan
lisi
lisi
[root@CentOS74 ~]# 
```

> 注意：{ list; }，{右边有空格，命令最后有;

## 组合条件测试

**并且：**EXPRESSION1和EXPRESSION2都为真，结果才是真

```bash
[ EXPRESSION1 -a EXPRESSION2 ]
```

**或者：**EXPRESSION1和EXPRESSION2有一个为真，结果就是真

```bash
[ EXPRESSION1 -o EXPRESSION2 ]
```

**取反：**

```bash
[ ! EXPRESSION ]
! [ EXPRESSION ]
```

> 注意：-a和-o需要使用测试命令进行，[[ ]]不支持

**&&：**

```bash
[ EXPRESSION1 ] && [ EXPRESSION2]
```

**||：**

```bash
[ EXPRESSION1 ] || [ EXPRESSION2 ]
```

## read接受输入

使用read来把输入值分配给一个或多个shell变量，read从标准输入中读取值，给每个单词分配一个变量，所有剩余单词都被分配给最后一个变量

格式：

```bash
read [options] [name ...]
```

常用选项：

```bash
-p ""      指定要显示的提示
-s         静默输入，一般用于密码
-n N       指定输入的字符长度N
-d '字符'   输入结束符
-t N       TIMEOUT为N秒
```

示例：

```bash
[root@CentOS74 ~]# read 
zhangsan
[root@CentOS74 ~]# echo $REPLY
zhangsan
[root@CentOS74 ~]# read NAME
zhangsan
[root@CentOS74 ~]# echo $NAME
zhangsan
[root@CentOS74 ~]# read NAME AGE
zhangsan 18
[root@CentOS74 ~]# echo $NAME
zhangsan
[root@CentOS74 ~]# echo $AGE
18
```

**从文件重定向接受输入**

```bash
[root@CentOS74 ~]# cat f.txt 
a b
[root@CentOS74 ~]# read x y < f.txt
[root@CentOS74 ~]# echo $x $y
a b
```

**通过管道接受输入**

```bash
[root@CentOS74 ~]# echo 1 2 | (read x y; echo x=$x y=$y)
x=1 y=2
[root@CentOS74 ~]# echo 1 2 | read x y; echo x=$x y=$y
x= y=
```

> man bash：
>
> Each command in a pipeline is executed as a separate process (i.e., in a subshell).

## bash配置文件

### 按生效范围分两类

全局配置：

```bash
/etc/profile
/etc/profile.d/*.sh
/etc/bashrc
```

个人配置：

```bash
~/.bash_profile
~/.bashrc
```

### shell登录两种方式分类

#### 交互式登录

- 直接通过终端输入用户名密码登录
- 使用`su - username`切换用户

配置文件执行顺序：

```bash
/etc/profile
/etc/profile.d/*.sh
~/.bash_profile
~/.bashrc
/etc/bashrc
```

#### 非交互式登录

- `su username`
- 图形界面下打开终端
- 执行脚本
- 任何其它的bash实例

执行顺序：

```bash
/etc/profile.d/*.sh
/etc/bashrc
~/.bashrc
```

### 按功能划分分类

profile类和bashrc类

#### profile类

为交互式登录的shell提供配置

全局：`/etc/profile`，`/etc/profile.d/*.sh`

个人：`~/.bash_profile`

功能：

- 用于定义环境变量
- 运行命令或脚本

#### bashrc类

为非交互式和交互式登录的shell提供配置

全局：`/etc/bashrc`

个人：`~/.bashrc`

功能：

- 定义命令别名和函数
- 定义本地变量

### 编辑配置文件如何生效

1. 重新启动shell进程
2. `source | . 配置文件` 

```bash
. ~/.bashrc
```

## 流程控制

### 条件选择

#### if条件选择

```bash
格式一：
if [条件1]; then
    执行第一段程序
else
    执行第二段程序
fi

格式二： 
if [条件1]; then
    执行第一段程序
elif [条件2]；then
    执行第二段程序
else
    执行第三段程序
fi

```

#### case条件选择

格式：

```bash
case 变量引用 in
PAT1)
    分支1
    ;;
PAT2)
    分支2
    ;;
...
*)
    默认分支
    ;;
esac
```

case支持glob风格的通配符：

```bash
*: 任意长度任意字符
?: 任意单个字符
[]: 指定范围内的任意单个字符
|: 或，如a或b
```

### 循环

#### for循环

**格式1：**

```bash
# 语法1
for NAME [in WORDS]; do COMMANDS; done

for i in 1 2 3 4; do echo $i; done
for i in {0..20}; do echo $i; done

# 语法2
for 变量名 in 列表; do
    循环体
done

for i in 1 2 3 4; do
    echo $i
done

# 语法3
for 变量名 in 列表
do
    循环体
done

for i in 1 2 3 4
do
    echo $i
done
```

for循环列表的生成方式：

- 手写列表。1 2 3 4

- 整数列表。

  ```bash
  {start..end}
  $(seq [start [step]] end)
  ```

  

- 返回列表的命令

  ```bash
  $(COMMAND)
  ```

- 使用glob

  ```bash
  for i in /var/log/*.log; do ll $i; done
  ```

- 变量引用

  ```bash
  sum=0
  for i in $@;do
      let sum+=i
  done
  echo sum=$sume
  ```

**格式2：**

```bash
for((i=0;i<10;i++))
do
    循环体
done

# 死循环
for((;;))
do
    循环体
done
```

#### while循环

```bash
while COMMANDS; do COMMANDS; done

while CONDITION; do
    循环体
done
```

死循环：

```bash
while :; do
    循环体
done
```



#### until循环

