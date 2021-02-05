# Go语言学习笔记

## 一、环境安装

下载地址： https://golang.google.cn/dl/

ide下载地址：https://www.jetbrains.com/go/download/#section=windows

破解包： [goland-2019.3.4.exe](D:\package\goland-2019.3.4.exe) 

## 二、基础语法

```go
package main

import "fmt"

func main() {
	var a string = "hello"
    fmt.Println(a)
    
    var b, c int = 1, 2
    fmt.Println(b, C)
    
}
```

变量默认初始值：

- 数值类型（包括completex64/128）为0
- 布尔类型为false
- 字符串为""（空字符串）
- 以下几种为nil

```go
var a *int
var a int[]
var a map[string] int
var a chan int
var a func(string) int 
var a error  //error是接口
```

**值类型和引用类型**

- 值类型：所有像int、float、bool和string这些基本类型都属于值类型，使用这些类型的变量直接指向存在内存中的值；当使用等号=将一个变量的赋值给另一个变量时，如 j = i，实际上是在内存中将i的值进行拷贝。
- 引用类型：可以通过&i来获取变量i的内存地址

**iota**

iota，特殊常量，可以认为是一个可以被编译器修改的常量

iota在const关键字出现是将被重置为0，const中每新增一行常量声明将使iota计数一次

```go
package main

import "fmt"

func main() {
    const (
        a = iota  //0
        b         //1
        c         //2
        d = "ha"  //独立值, iota += 1
        e         //"ha", iota += 1
        f = 100   //iota += 1
        g         //100, iota += 1
        h = iota  //7, 恢复计数
        i = 8     //8
    )
    fmt.Println(a,b,c,d,e,f,g,h,i)
}
```

go语言没有三目运算符，不支持?:形式的条件判断

**函数调用**

```go
package main

import "fmt"

func main() {
    var a int = 100
    var b int = 200
    var ret int
    ret = max(a,b)
    fmt.Println(ret)
}

func max(num1, num2 int) int {
    var result int
    if (num1 > num2) {
        result = num1
    } else {
        result = num2
    }
    return result
}
```

**函数闭包**

```go
package main

import "fmt"

func getSequence()  func() int {
    i:=0
    return func() int {
        i+=1
        return i
    }
}
```

**函数方法**

```go
package main

import "fmt"

/* 定义结构体 */
type Circle struct {
    radius float64
}

func main() {
    var c1 Circle
    c1.radius = 10.00
    fmt.Println("圆的面积：", c1.getArea())
}

func (c Circle) getArea() float64 {
    return 3.14 * c.radius * c.radius
}

//注意如果想要更改成功c的值，需要使用指针
func (c *Circle) changeRadius(radius float64) {
    c.radius = radius
}
```

**结构体**

```go
package main

import "fmt"

type Books struct {
    title string
    author string
    subject string
    book_id int
}

func main() {
    var Book1 Books
    var Book2 Books
}

func printBook(book *Books) {
    fmt.Printf(book.title)
}
```

结构体中属性的首字母大小写问题

- 首字母大写相当于public

- 首字母小写相当于private

注：当要将结构体对象转换为JSON时，对象中的属性首字母必须是大写，可以使用tag标记属性显示小写

```go
type Person struct {
    Name string `json:"name"`
    Age int `json:"age"`
    Time int64 `json:"-"`  //标记忽略该字段
}
```

**切片（Slice）**

```go
func main() {
    /* 创建切片 */
    numbers := []int{0,1,2,3,4,5,6,7,8}
    printSlice(numbers)
    
    fmt.Println(numbers)
	/* 打印子切片从索引1（包含）到索引4（不包含）*/
    fmt.Println(numbers[1:4])
    /* 默认下限为0 */
    fmt.Println(numbers[:3])
    /* 默认上限为len(s) */
    fmt.Println(numbers[4:])
    
    numbers1 := make([] int, 0, 5)
    printSlice(numbers1)
    
    /* 向切片添加一个元素 */
    numbers1 = append(numbers1, 1)
    /* 同时添加多个元素 */
    numbers1 = append(numbers1, 2,3,4)
    
    /* 拷贝numbers的内容到numbers1*/
    copy(numbers1, numbers)
}

func printSlice(x []int) {
    fmt.Printf("len=%d cap=%d slice=%v\n", len(x), cap(x), x)
}
```

**Map(集合)**

```go
func main() {
    var countryMap map[string]string
    countryMap = make(map[string]string)
    
    countryMap["France"] = "巴黎"
    countryMap["Italy"] = "罗马"
    countryMap["Japan"] = "东京"
    countryMap["India"] = "新德里"
    
    for country := range countryMap {
        fmt.Println(country, " ", countryMap[country])
    }
    
    delete(countryMap, "Japan")
}
```

**接口**

```go
type Phone interface {
    call()
}

type NokiaPhone struct {
}

func (nokiaPhone NokiaPhone) call() {
    fmt.Println("Nokia phone call you!")
}

func main() {
    var phone Phone = new(NokiaPhone)
    phone.call()
}
```

