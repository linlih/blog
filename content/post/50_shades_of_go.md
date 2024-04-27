---
title: "50 Shades of Go"
description: "50_shades_of_go"
keywords: "50_shades_of_go"

date: 2024-04-27T22:08:27+08:00
lastmod: 2024-04-27T22:08:27+08:00

categories:
 -
tags:
  -
  -

# Post's origin author name
#author:
# Post's origin link URL
#link:
# Image source link that will use in open graph and twitter card
#imgs:
# Expand content on the home page
#expand: true
# It's means that will redirecting to external links
#extlink:
# Disabled comment plugins in this post
#comment:
# enable: false
# Disable table of content int this post
# Notice: By default will automatic build table of content 
# with h2-h4 title in post and without other settings
#toc: false
# Absolute link for visit
#url: "50_shades_of_go.html"
# Sticky post set-top in home page and the smaller nubmer will more forward.
#weight: 1
# Support Math Formulas render, options: mathjax, katex
#math: mathjax
# Enable chart render, such as: flow, sequence, classes etc
#mermaid: true
---

Go 是一门简单有趣的语言，但是像其他语言一样，它也有很多的陷阱。

作者在这个文章中总结 go 的一些注意事项，分为了入门、中级、进阶、高级，总的有 70 条内容，和标题的 50 shades 数量不太一致。

这些条目中有很多 go 的编译器以及 linter 都会帮忙给出错误提示，所以只需要关注一些重点的条目，其他的就过一遍就行。

原文地址：https://golang50shad.es/index.html
# beginner
1. 开放的大括号不能单独一行，go linter 已经完全 cover 这个场景，这样写也编译不过去
```go
func main()
{ // 这个括号不能单独一行

}
```
2. 未使用的变量，在 golang 当中不允许定义了变量但是没有使用
3. 未使用的 import，同上
4. 短变量定义方法只能用到在函数内部，package 的全局变量只能使用 var 进行定义
5. 不能使用短定义方法重定义变量
6. 不能使用短定义方法设置值，这个在 go 1.21 是可以的
```go
package main

import "fmt"

func a() (int, error) {
	return 1, nil
}

func main() {
	var aa int = 1
	aa, err := a()
	fmt.Println(aa, err) // go 1.21 可以正常编译执行
}
```
7. 变量意外被掩盖
```go
func main() {
	x := 1
	{
		fmt.Println(x)
		x := 2
		fmt.Println(x)
	}
	fmt.Println(x) // x = 1, 如果要得到 x = 2
}
```
golang linter vet 默认没有提供这个场景的检测，需要手动指定。go tool vet -shadow file.go

可以使用 go-nyet 工具来做更深入的检测

8. 不能使用 nil 初始化未明确指定类型的变量
9. 使用 nil map，map 必须要 make 后才能使用，否则是个空值；slice 是允许初始化后直接 append
10. map 在创建的时候可以预先指定容量大小，但不能使用 cap 函数
11. string 类型不能赋值为 nil
12. **array 作为函数参数的时候要注意，在 golang 中，array 的传递是值传递**，这一点要注意和 c/c++ 的区别，如果要在函数内修改 array 的内容，需要传递指针。**如果使用 Slice 就可以不需要传递指针就可以修改**，这个也是个坑，如果要传值的话，需要拷贝一份，否则函数内的修改会传递到函数外面
13. range 时要注意返回的是 index 和 value，单值返回的是index而不是value，要确定是需要 index 还是 value
14. slice 和 array 是一维的，如果要创建多维的数据，需要手动 make 出来
```go
x := 2
y := 4
table := make([][]int, x)
for i := range table {
	table[i] = make([]int, y)
}
```
15. 在 map 判空最好是使用双返回值的ok，而不是判断结果是否为空
```go
x := map[int]{1:"1", 2:"", 3:"3"}
if v := x[2]; v == "" {
	fmt.Println("no entry") // 错误, 应该使用 _, ok := x[2]; !ok 来判定
}
```
16. 字符串是不可更改的，只读类型；如果要修改，可以强制类型转换成 bytes 后修改，但是注意有的字符可能是多个字节的，可能需要转换成 rune byte 修改
17. string 和 byte slice 的转换，当你进行转换的时候得到的是底层数据的一份拷贝，并不是直接指向原数据的底层地址
18. string 的 index 指向的是 byte，而不是 rune
19. 字符串并不一定总是 UTF8 编码，它可以是任意字符
20. **字符串的长度返回的长度也是底层的 byte 长度，而不是 rune 的长度**；如果想要 rune 的长度，调用 utf8.RuneCountInString
21. 多行 slice 、array、map 的元素都需要逗号结尾
22. log 库的 Fatal 和 Panic 不仅仅是输出 log，还会终止程序
23. **golang 内建的数据结构都不是并发安全的**
24. **在 string 的 range 语法中要注意，默认情况下 range 会将 string 尝试解析为 UTF8 字符**，如果识别错误，会返回 0xfffd。如果要返回原始字符的话，则需要将其强制类型转换为 []byte 后再 range
25. map 每次 range 的结果都是不一致的。在 Go playgroud 中返回的结果是一致的，因为 Go playgroud 中如果代码没有变更的话，不会重新编译
26. 在 switch 当中的 case 会 fall through 到下一个 case 进行判断
```go
switch str {
case "a":
case "b":
    fmt.Println(str) // 当 str == "b" 的时候输出
}

switch str {
case "a", "b":
    fmt.Println(str) // 当 str == 'b' or str == 'a' 的时候输出
}
```
27. 数值的增加和减小，在 go 中是不允许前置++或者直接使用后置++的结果的
```go
data = []int{1, 2, 3}
i := 0
++i       // error
data[i++] // error
```
28. 在 golang 中位操作的 NOT 使用的是 `^` 而不是其他语言使用的 `~`
29. **操作符的优先级和其他语言是有一些不一致的**，这个我觉得是很容易搞混淆的，作为开发者最好是使用括号将优先级明确清楚，防止出现异常
30. 不可导出的字段在结构体中是不会被编码的，比如 json 编码的时候不会编码不可导出的字段
31. 如果 main 退出的时候，后台还有活跃的 goroutine，main 自身没有对这些 goroutine 的生命周期进行管控的话，就会直接退出，不管这些 goroutine 是否执行完成
32. 给一个没有 buffer 的 channel 发送消息，只要接收方一接收就会立即返回
33. 给一个关闭的 channel 发送消息会导致 panic
34. 在一个 nil 的 channel 中发送或者接收消息会永远阻塞
35. value 接收器并不能改变原始的值，如果要修改原有结构体元素的内容，需要使用指针接收器防止出错
# intermediate
36. 关闭 HTTP 响应体 body，就算返回的是空的 body 也需要关闭（这个待验证）
```go
resp, err := http.Get("https://api.ipify.org?format=json")
if resp != nil {  // 注意：为什么不使用 err != nil 作为判定条件，是因为在 302 重定向的时候 err 和 resp 都有可能不为空
	defer resp.Body.Close()
}
```
最开始的实现中，resp.Body.Close() 还是将数据读完，并把未读完的数据丢弃掉，确保了如果这个链接是长连接的情况下这还能被其他请求使用，但是最近的 resp.Body.Close 实现有变化，需要受手动将这些数据读完（这个待验证，感觉是错的）
```go
_, err := io.Copy(ioutil.Discard, resp.Body)
```
38. 关闭 HTTP 链接；默认情况下，go 标准库只会在 HTTP 服务器要求关闭连接才会关闭，这可能导致本地客户端的 socket 文件描述符被用完，所以可以手动将其关闭。有几个方法：一个是增加头部：Connection: Close，另外一个方法就是创建自己的 http.Transport，并将 DisableKeepAlives 置为 true
39. **JSON 编码的时候会加一个新行**；原因是 JSON Encoder 对象是为流设计的，换行作为一个 JSON 对象的分割符
40. **JSON 包会对特殊字符进行转义**，如果不想要转义的结果，需要创建 NewEncoder 并设置 SetEscapeHTML(false)
41. Unmarshal 方法将数值转换成 interface 的时候，会默认将其转换为 float64 类型，有几种处理方法
 -  直接类型转换为 float64
 -  转换为 json.Number
 -  不要使用 interface 转换，指定特定的类型进行转换
 -  使用 json.RawMessage，然后再做一个 Unmarshal 到特定类型
 42. JSON 字符串不能使用十六进制或者非 UTF-8 的字符，如果带了反斜杠，则需要转义下才行。如果实在要带上特殊字符，可以使用 []byte 转换后编码
 43. 比较 struct、array、slice、map。如果 struct 中有元素不可比较的话，直接对比两个 struct 则会编译错误。array 只有在其元素可比的情况下才可以比较。go也提供了一些比较函数，比如 reflect.DeepEqual，但是某些场景下 DeeqEqual 是不相等的，比如空的 []byte 和 nil 的 []byte，如果这个场景要判定为相等，可以使用 bytes.Equal 函数来判断。
     reflect.DeepEqual 、bytes.Equal、bytes.Compare 在含有密钥场景的判定中有可能遭受时间攻击，time attack，也就是说这几个函数会随着输入的不同，响应的时间不同，从而在攻击中构造不同的字符串输入得到相应的信息
 46. 从 panic 中 recover 要注意 recover 的覆盖范围。注意：recover 函数一定是在当前或者子函数会出现 panic 的函数中直接调用，而不是在函数内部调用
 ```go
import "fmt"

func doRecover() {  
    fmt.Println("recovered =>",recover()) //prints: recovered => <nil>
}

func main() {  
    defer func() {
        doRecover() //panic is not recovered
    }()

    panic("not good")
}
```
47. 在 range 语法中更新和引用 Slice Array 和 map 的元素是原始数据的拷贝，修改其内容以及引用都不会生效；但是如果 value 保存到是指针，那是可以的
48. Slice 如果执行 Re-Slice 会引用底层的内存信息
```go
package main

import "fmt"

func get() []byte {  
    raw := make([]byte,10000)
    fmt.Println(len(raw),cap(raw),&raw[0]) //prints: 10000 10000 <byte_addr_x>

	// 为了避免这个问题，可以 copy 一个 slice
	// res := make([]byte, 3)
	// copy(res, raw[:3])
    return raw[:3]
    // 这里的 raw 是取前面 3 个字节，如果是取后面的字节，则会向后偏移
}

func main() {  
    data := get()
    fmt.Println(len(data),cap(data),&data[0]) //prints: 3 10000 <byte_addr_x>
}
```
49. Slice 的数据污染
```go
package main

import (
	"bytes"
	"fmt"
)

func main() {
	path := []byte("AAAA/BBBBBBBBB")
	sepIndex := bytes.IndexByte(path, '/')
	dir1 := path[:sepIndex]
	dir2 := path[sepIndex+1:]
	fmt.Println("dir1 =>", string(dir1)) //prints: dir1 => AAAA
	fmt.Println("dir2 =>", string(dir2)) //prints: dir2 => BBBBBBBBB

	dir1 = append(dir1, "suffix"...)
	path = bytes.Join([][]byte{dir1, dir2}, []byte{'/'})

	fmt.Println("dir1 =>", string(dir1)) //prints: dir1 => AAAAsuffix
	// 如果 dir1 换成：dir1 := path[:sepIndex:sepIndex] 的话得到的结果就是 BBBBBBBBB
	// 这是因为全切片表达式 full slice expression 会将 cap 也修改了，否则 cap 会 path 底层的 cap 一致
	// 这样当 dir append 就会发生扩容生成一个新的切片，和底层的切片不一致了
	fmt.Println("dir2 =>", string(dir2)) //prints: dir2 => uffixBBBB (not ok)

	fmt.Println("new path =>", string(path))
}
```
50. 使用 re-slice 之后的数据是会保留原有 slice 数据的
51. 类型声明和方法，对以后的类型进行类型定义的时候，并不会继承已有类型的方法
```go
package main

import "sync"

type myMutex sync.Mutex

// 如果需要原始类型的方法，可以定义一个结构
// type myMutex struct {
//     sync.Mutex
// }
// var mtx myMutex
// mtx.Lock() // works

func main() {  
    var mtx myMutex
    mtx.Lock() //error
    mtx.Unlock() //error  
}
```
52. 要打破外层循环的话，可以使用 break + label 的方式或者 goto 语句，单纯使用 break 语句的话只会打破内层循环
```go
package main

import "fmt"

func main() {  
    loop:
        for {
            switch {
            case true:
                fmt.Println("breaking out...")
                break loop // or: goto loop
            }
        }

    fmt.Println("out!")
}
```
54. 在 for 语句中闭包的变量捕获得到的变量是 for 循环的临时变量，这个问题在 go 1.22 已经修正了
```go
data := []string{"one","two","three"}

for _,v := range data {
    // copy 一份ok
	// vCopy := v
	go func() {
		// fmt.Prinltn(vCopy)
		fmt.Println(v)
	}()
	// 这种方式 ok
	// go func(vParam) {
	//     fmt.Println(vParam)
	// }(v)
}

time.Sleep(3 * time.Second)
//goroutines print: three, three, three
```
55. 在 defer 中的变量捕获要注意，如果是闭包捕获的话，则会在 defer 之前就传值进去，而不是原有的变量，使用指针可以
56. 注意 defer 函数的执行范围是在函数退出的时候，如果在 for 循环中使用需要注意将其 wrap 成一个函数
```go
for file := range fileArray {
	f, err := os.Open(file)
	if err != nil {
		continue
	}
	defer f.Close() // will not work

    // works
    func() {
	    f, err := os.Open(file)
		if err != nil {
			continue
		}
		defer f.Close() 
    }()
}
```
57. 类型断言的陷阱，如果类型转换失败，返回的结果是 0 值
```go
package main

import "fmt"

func main() {  
    var data interface{} = "great"

    if data, ok := data.(int); ok {
        fmt.Println("[is an int] value =>",data)
    } else {
        fmt.Println("[not an int] value =>",data) 
        //prints: [not an int] value => 0 (not "great")
        // 这里的话会导致 data 被覆盖了，原有的 data 数据丢失了
    }
}
```
58. 阻塞的 goroutine 和泄露的资源，在 goroutine 创建的时候要保证它是会结束的，不会被阻塞导致 goroutine 泄露
```go
// 修改的方式有很多，比如将 c 改成 buffered 的 channel，或者 goroutine 使用 select 语句防止阻塞
func First(query string, replicas ...Search) Result {  
    c := make(chan Result)
    searchReplica := func(i int) { c <- replicas[i](query) }
    for i := range replicas {
        go searchReplica(i) // replicas 有多个
    }
    return <-c // 由于这里只返回一个结果，所起其他 goroutine 会因为 c 是阻塞的导致无法退出，造成泄露
}
```
59. 不同的变量但是值是零值的话则其地址相同
60. iota 并不一定完全从零开始
```go
const (
	zero = iota
	one = iota
)

const (
	non_zero = "123"
	one = iota
	two = iota
)
```
# advanced
61. 在值对象上使用指针方法会导致错误，编译器会直接报错，linter 也会提示
```go
package main

type data struct {
	name string
}

func (p *data) print() {
	fmt.Println("name:", p.name)
}

type printer interface {
	print()
}

func main() {
	d1 := data{"one"}
	d1.print()  // ok
	
	var in printer = data{"two"} // error

	m := map[string]data {"x":data{"three"}}
	m["x"].print() //error
}
```
62. 在 map 上更新值对象是不会成功的，因为值是拷贝的而不是源对象了。但是如果是 slice 却是可以的。这个 linter 和编译器都会报错，所以不用担心会出现这个错误
63. inteface 的值是 nil 的时候才是 nil，如果 inteface 指向了一个 nil 的变量，那其就不是 nil 了，所以这里会有一个陷阱
```go
package main

import "fmt"

func main() {  
    var data *byte
    var in interface{}

    fmt.Println(data,data == nil) //prints: <nil> true
    fmt.Println(in,in == nil)     //prints: <nil> true

    in = data
    fmt.Println(in,in == nil)     //prints: <nil> false
    //'data' is 'nil', but 'in' is not 'nil'

	// 陷阱
	doit := func(arg int) interface{} {
        var result *struct{} = nil

        if(arg > 0) {
            result = &struct{}{}
        } // 这里需要显示返回nil

        return result
    }

    if res := doit(-1); res != nil {
        fmt.Println("good result:",res) //prints: good result: <nil>
        //'res' is not 'nil', but its value is 'nil'
    }
}
```
64. 变量在 go 中是由编译器决定放在堆还是栈上的，不想 c/c++ new操作的会放在堆上。想查看变量会放在堆上还是栈上，可以使用命令：`go run -gcflags -m app.go` 来查看变量是分配在栈上还是堆上的
65. GOMAXPROCS 可以设置 go 最多运行在多少个 CPU 上
66. 在 go 中会对读写操作进行指令重排，如果需要严格的顺序，需要使用 channel 或者 sync 同步原语来实现
```go
package main

import (  
    "runtime"
    "time"
)

var _ = runtime.GOMAXPROCS(3)

var a, b int

func u1() {  
    a = 1
    b = 2
}

func u2() {  
    a = 3
    b = 4
}

func p() {  
    println(a)
    println(b)
}

func main() {  
	// 结果不唯一
    go u1()
    go u2()
    go p()
    time.Sleep(1 * time.Second)
}
```
66. 优先调度，这个应该 go 1.14 引入强制式调度就没有这个问题了
```go
package main

import "fmt"

func main() {  
    done := false

    go func(){
        done = true
    }()

    for !done { // 这里会永远卡住， go 1.22 测试不会
    }
    fmt.Println("done!")
}
```
67. import C 需要独立一个 import，其他的 import 另外起一个 block 
```go
package main

/*
#include <stdlib.h>
*/
import (
  "C"
)

import (
  "unsafe"
)

func main() {
  cs := C.CString("my go string")
  C.free(unsafe.Pointer(cs))
}
```
69. import C 和 C 代码之间有空行也是会报错的
70. 调用 C 函数不能直接使用变量参数
```go
package main

/*
#include <stdio.h>
#include <stdlib.h>

void out(char* in) {
  printf("%s\n", in);
}
*/
import "C"

import (
  "unsafe"
)

func main() {
  cstr := C.CString("go")
  C.printf("%s\n",cstr) //not ok
  C.free(unsafe.Pointer(cstr))

  cstr := C.CString("go")
  C.out(cstr) //ok
  C.free(unsafe.Pointer(cstr))
}
```