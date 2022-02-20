# golang的安装

> https://studygolang.com/dl
> 

```bash
export WORKSPACE="$HOME/workspace" # 设置工作目录
# Go envs
export GOROOT=/usr/local/go/go # GOROOT 设置
export GOPATH=$WORKSPACE/golang # GOPATH 设置
export GOBIN="$GOPATH/bin"
export GO111MODULE=on # 开启 Go moudles 特性
export GOPROXY=https://goproxy.cn,direct # 安装 Go 模块时，代理服务器设置
export GOPRIVATE=
```



# 循环

```go
func loop1(start int, end int) {
	sum := 0
	for i := start; i < end; i++ {
		sum += i
	}
	fmt.Printf("%d+...+%d = %d", start, end, sum)
}
```

```go
// 相当于其他语言里的while
func loop2() {
	sum := 1
	for sum < 1000 {
		sum += sum
	}
	fmt.Println(sum)
}
```

```go
// 死循环
func loop3() {
	for {
	}
}
```

# 判断

## if

## switch用法

```go
func TestSwitch1(t *testing.T) {
    grade := "A"
    switch grade {
    case "A":
        fmt.Println("优秀")
    case "B":
        fmt.Println("良好")
    default:
        fmt.Println("一般")
    }
}
```

```go
func TestSwitch2(t *testing.T) {
    week := 2
    switch week {
    case 1, 2, 3, 4, 5:  // case可以多条件
        fmt.Println("工作日")
    case 6, 7:
        fmt.Println("周末")
    }
}
```

```go
func TestSwitch3(t *testing.T) {
    score := 80
    switch {
    case score >= 90: // case可以是个表达式
        fmt.Println("优秀")
    case score >= 80:
        fmt.Println("良好")
    default:
        fmt.Println("好好学习吧")
    }
}
```

```go
func TestSwitch4(t *testing.T) {
    a := 100
    switch a {
    case 100:
        fmt.Println(100)
        fallthrough // 执行下一个case
    case 200:
        fmt.Println(200)
    default:
        fmt.Println(300)
    }
}
```

# 闭包

```go
func Add() func(y int) int {
    var x int
    return func(y int) int {
        x += y
        return x
    }
}

func TestAdd(t *testing.T) {
    f := Add()
    fmt.Println(f(10)) // 10
    fmt.Println(f(20)) // 30
    fmt.Println(f(30)) // 60
}
```

# 递归

函数内部调用函数自身的函数称为递归函数

1. 递归就是自己调用自己
2. 必须先定义函数的退出条件，没有退出条件，递归将成为死循环
3. golang的递归函数很可能会产生一大堆的goroutine，也很可能会出现栈空间内存溢出问题

```go
// 阶乘
func Factorial(n int) int {
    if n < 1 {
        return 0
    }
    if n == 1 { // 退出条件
        return 1
    }
    return n * Factorial(n - 1)
}
```

```go
// 斐波那契数列
func Fibonacci(n int) int {
    if n == 1 || n == 2 {
        return 1
    }
    return Fibonacci(n - 1) + Fibonacci(n - 2)
}
```

# init函数

init函数先于main函数执行，实现包级别的一些初始化操作

init函数的主要特点：

* 先于main自动执行，不能被其他函数调用
* init函数没有输入参数和返回值
* 每个包可以有多个init函数，包的每个源文件可以有多个init函数，执行顺序没有明确定义
* 不同包的init函数执行顺序按照包导入的依赖关系决定执行顺序

golang初始化顺序：变量初始化 -> init() -> main()

# 类型定义和类型别名

类型定义和类型别名的区别

1. 类型定义相当于定义了一个新的类型，和之前的类型不同；类型别名没有定义新类型，只是给已有的类型取了一个别名
2. 类型别名只会在代码中出现，在编译完成后并不会继续存在
3. 类型别名可以调用原来类型拥有的方法。类型定义的新类型无法调用原类型的方法

```go
type NewType Type // 类型定义
type NewType = Type // 类型别名
```



# strings

| 函数名                                                     | 作用                                                   |
| ---------------------------------------------------------- | ------------------------------------------------------ |
| strings.HasPrefix(s, prefix string) bool                   | 字符串s是否以prefix开头                                |
| strings.HasSuffix(s, suffix string) bool                   | HasSuffix tests whether the string s ends with suffix. |
| strings.Index(s, sep string) int                           | 判断str在s中首次出现的位置，如果没有出现，则返回-1     |
| strings.Replace(str string, old string, new string, n int) | 字符串替换，n表示替换次数                              |
| strings.Count(str string, substr string) int               | 字符串计数                                             |
| strings.Repeat(str string, count string) string            | 重复count次str                                         |
| strings.ToLower(str string) strings                        | 转为小写                                               |
| strings.ToUpper(str string) strings                        | 转为大写                                               |
| strings.TrimSpace(str string)                              | 去掉字符串首尾空白字符                                 |
| strings.TrimLeft(str string, cut string)                   | 去掉字符串首cut字符                                    |
| strings.TrimRight(str string, cut string)                  | 去掉字符串尾cut字符                                    |
| strings.Trim(str string, cut string)                       | 去掉字符串首尾cut字符                                  |
| strings.Fields(str string) slice                           | 返回以空格分隔的所有子串slice                          |
| strings.Split(str string, split string) slice              | 返回str split分隔的所有子串slice                       |
| strings.Join(s1 []string, sep string)                      | 用sep把s1中所有元素连接起来                            |
| strings.Itos(i int)                                        | 把一个整数i转换成字符串                                |
| strings.Atoi(str string) (int, error)                      | 把一个字符串转成整数                                   |



# 类型断言

```go
// 判断变量类型
package main

import "fmt"

type Student struct {
	Name string
}

func judgmentType(items ...interface{}) {
	for k, v := range items {
		switch v.(type) {
		case string:
			fmt.Printf("string, %d[%v]\n", k, v)
		case bool:
			fmt.Printf("bool, %d[%v]\n", k, v)
		case int, int32, int64:
			fmt.Printf("int, %d[%v]\n", k, v)
		case float32, float64:
			fmt.Printf("float, %d[%v]\n", k, v)
		case Student:
			fmt.Printf("Student, %d[%v]\n", k, v)
		case *Student:
			fmt.Printf("Student, %d[%p]\n", k, v)
		}
	}
}

func main() {
	stu1 := &Student{Name: "nick"}
	judgmentType(1, 2.2, "learing", stu1)
}

```



# map

## 声明和初始化

```go
dict := make(map[string]int)
dict["张三"] = 18

dict := map[string]int{"张三":18,"李四":19}
```



# 结构体

## 结构体定义

```go
type Student struct {
	name string
	age  int
}

stu1 := Student{name: "luxinyu", age: 18}

var stu2 Student
stu2.name = "chenchen"
stu2.age = 16

var stu3 *Student = &Student{name: "luxinyu", age:18}
var stu4 *Student = new(Student)
```

# 链表

## 遍历列表

```go
func trans(p *Student) {
	for p != nil {
		fmt.Println(*p)
		p = p.next
	}
}
```





## 尾部插入

```go
func insertTail(p *Student) {
	var tail = p
	for i := 0; i < 10; i++ {
		stu := Student{
			Name:  fmt.Sprintf("stu%d", i),
			Age:   rand.Intn(100),
			Score: rand.Float32() * 100,
		}
		tail.next = &stu
		tail = &stu
	}
}
```



## 头部插入

# 接口

## Demo

```go
package main

import "fmt"

type Test interface {
	Print()
}

type Student struct {
	name  string
	age   int
	score int
}

func (p *Student) Print() {
	fmt.Println("name:", p.name)
	fmt.Println("name:", p.age)
	fmt.Println("name:", p.score)
}

func main() {
	var t Test
	var stu Student = Student{
		name:  "stu1",
		age:   20,
		score: 100,
	}
	t = &stu
	t.Print()

}
```



## 接口的嵌套

```go
package main

import "fmt"

type Reader interface {
	Read()
}
type Write interface {
	Write()
}

type ReadWrite interface {
	Reader
	Write
}
type File struct{}

func (f *File) Read() {
	fmt.Println("read data")
}

func (f *File) Write() {
	fmt.Println("write data")
}

func Test(rw ReadWrite) {
	rw.Read()
	rw.Write()
}

func main() {
	var f File
	Test(&f)
}
```



## 类型断言

接口是一般类型，不知道具体类型，如果要转成具体类型可以采用以下方法进行转换

```go
package main

import "fmt"

func Test(a interface{}) {
	b, ok := a.(int)
	if ok == false {
		fmt.Println("convert failed")
		return
	}
	b += 3
	fmt.Println(b)
}

func main() {
	var b int
	Test(b)
}
  
```

# 标准库

## os



## time

Timer是定时器，可以实现一些定时操作，内部也是通过channel来实现的

```go
// 1s之后打印时间
func TestTimer(t *testing.T) {
    timer := time.NewTimer(time.Second)
    fmt.Println(time.Now().Format(time.RFC3339Nano))
    t1 :=<- timer.C
    fmt.Println(t1.Format(time.RFC3339Nano))
}

// 1s之后打印时间
func TestAfter(t *testing.T) {
    fmt.Println(time.Now().Format(time.RFC3339Nano))
    t1 :=<- time.After(time.Second)
    fmt.Println(t1.Format(time.RFC3339Nano))
}
```

Ticker支持周期性定时，Timer只能定时一次

```go
// 每隔1s打印时间
func TestTick(t *testing.T) {
    timer := time.Tick(time.Second)
    for t := range timer {
        fmt.Println(t.Format("2006-01-02 15:04:05.000"))
    }
}
```



# 反射

反射是在运行时获取变量的相关信息

```go
import "reflect"
```

# 终端读写

```go
os.stdout
os.stdin
os.stderr
```

```go
package main

import (
	"fmt"
)

type student struct {
	Name  string
	Age   int
	Score float32
}

func main() {
	var str = "stu01 18 89.92"
	var stu student
	// 从字符串格式化输入
	fmt.Sscanf(str, "%s %d %f", &stu.Name, &stu.Age, &stu.Score)
	fmt.Println(stu)
}

```

```go
package main

import (
	"bufio"
	"fmt"
	"os"
)

// 缓冲区读标准输入
func main() {    
	reader := bufio.NewReader(os.Stdin)
	str, err := reader.ReadString('\n')
	if err != nil {
		fmt.Println(err)
	}
	fmt.Printf(str)
}
```

```go
package main

import (
	"bufio"
	"fmt"
	"os"
)

//缓冲区读文件
func main() {
	file, err := os.Open("./test.log")
	if err != nil {
		fmt.Println("open file error: ", err)
	}
	defer file.Close()
	reader := bufio.NewReader(file)
	str, err := reader.ReadString('\n')
	if err != nil {
		fmt.Println("read error: ", err)
	}
	fmt.Println("read file: ", str)

}
```



# 文件读写

	os.O_WRONLY：只写
	os.O_CREATE：创建文件
	os.O_RDONLY：只读
	os.O_RDWR：读写
	os.O_APPEND：追加
	os.O_TRUNC：清空


```go
package main

import (
	"fmt"
	"os"
)

func main() {
	fmt.Fprint(os.Stdout, "hello world")

	// 写模式打开文件，如果没有就新建
	file, err := os.OpenFile("./test.log", os.O_CREATE|os.O_WRONLY, 0664)
	if err != nil {
		fmt.Println(err)
	}
	// 向file中写数据
	fmt.Fprint(file, "hello world 222")
	file.Close()
}
```

# net package

```go
// 返回host:port的字符串
func JoinHostPort(host, port string) string

// 返回给定主机的cname
func LookupCNAME(host string) (cname string, err error)

// 返回给定域名/IP的IP地址列表
host, err := net.LookupHost("www.jdcloud.com")
if err != nil {
    fmt.Println(err)
}
fmt.Println(host)
fmt.Println(reflect.TypeOf(host))


```

