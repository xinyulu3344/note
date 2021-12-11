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

## switch

```go
func switchtest1() {
	switch 8 {
	case 1:
		fmt.Println(1)
	case 2:
		fmt.Println(2)
	case 8:
		fmt.Println(8)
	case 9:
		fmt.Println(1)
	default:
		fmt.Println(8)
	}
}
```

```go
func switchtest2() {
	t := time.Now()
	switch {
	case t.Hour() < 12:
		fmt.Println("Good morning!")
	case t.Hour() < 17:
		fmt.Println("Good morning!")
	default:
		fmt.Println("Good morning!")
	}
}
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

# time

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

## 链表

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

