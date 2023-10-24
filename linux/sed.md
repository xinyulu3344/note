# sed

格式

```bash
sed [OPTION] {scripts} [FILE]
```

OPTION：

```bash
-n 不输出模式空间内容到屏幕
-e 多点编辑
-f /PATH/SCRIPT_FILE 从指定文件中读取编辑脚本
-r, -E 使用扩展正则表达式
-i.bak 备份文件并原处编辑
```

scripts

```bash
'地址命令'
```

地址格式

```bash
不给地址，全文处理
#: 指定的行
$: 最后一行
/pattern/: 被此模式所能匹配到的每一行
地址范围:
  #,#
  #,+#
  /pat1/,/pat2/
  #,/pat/
步进: ~
  1~2 从第1行开始，每次步进2行
  2~2 从第2行开始，每次步进2行
```



命令：

```bash
p 打印当前模式空间内容，追加到默认输出之后
Ip 忽略大小写
d 删除模式空间匹配的行，并立即开始处理下一轮
a [\\]text  在指定行后面追加内容
i [\\]text  在指定行前面插入内容
c [\\]text  将指定行替换为指定内容
w /path/file 保存模式匹配的行至指定文件
r /path/file 读取指定文件的文本至模式空间中匹配到的行后
=  为模式空间中的行打印行号
!  模式空间中匹配行取反处理
s/pattern/string/修饰符 查找替换，支持使用其它分隔符s@@@,s ###
  修饰符:
    g 行内全局替换
    p 显示替换成功的行
    w /PATH/FILE  将替换成功的行保存至文件中
    I,i 忽略大小写
```

示例：

```bash
# 打印第2行
sed -n 2p /etc/passwd

# 打印最后一行
sed -n '$p' /etc/passwd

# 
sed -n '/^[^#]/p' /etc/fstab

# 打印3-6行
sed -n '3,6p' /etc/passwd

# 从第3行到第3+4行
nl /etc/passwd | sed -n '3,+4p'

# 匹配从time1到time2之间的内容
sed -n '/time1/,/time2/p' xxx.log

# 删除包含root的行
sed '/root/d' /etc/passwd

# 不打印包含root的行
sed -n '/root/!p' /etc/passwd

seq 10 | sed '5a\     abc \naaa'
sed -n '/root/w /tmp/sed.log' /etc/passwd
```

