---
title: go笔记
date: 2023-10-13 18:55:49
tags: "go"
categories: "编程语言"
---

# 知识点

## 字符串

由一串固定长度的字符连接起来的字符序列，Go的字符串是由单个UTF-8表示的字节连接起来的，由于该编码的不定性，字符串可能根据需要占用1~4byte；即字符串是字节的定长数组

字符串为不可变类型，故不能直接修改字符串的内容，如需修改，则需要将字符串内容复制到一个可写的变量中（一般是`[]byte`或`[]rune`），然后进行修改

#### 字符串的转义

- 双引号创建可解析的字符串，支持转义，但不能用来引用多行
- 反引号创建原生的字符串字面量，可由多行组成，不支持转义，且可包含除反引号外的所有字符

```go
str1 := "\"Go web\",I love you \n"
str2 := `"Go web",
I love you \n`

"Go web",I love you

"Go web",
I love you \n
```

#### 字符串的修改

- 修改字节（用[]byte）

  ```go
  str := "Hi 世界!"
  by := []byte(str)
  by[2] = ','
  
  //Hi,世界
  ```

- 修改字符（用[]rune）

  ```go
  str := "Hi 世界"
  by := []rune(str)
  by[3] = '中'
  by[4] = '国'
  
  //Hi 中国
  ```

## 函数

Go 里面有三种类型的函数，函数参数、返回值以及它们的类型被统称为函数签名：  

- 普通的带有名字的函数
- 匿名函数或者lambda函数
- 方法（Methods）

这样是不正确的 Go 代码：

```go
func g()
{
}
```

它必须是这样的：

```go
func g() {
}
```

函数重载 (function overloading) 指的是可以编写多个同名函数，只要它们拥有不同的形参/或者不同的返回值，在 Go 里面函数重载是不被允许的

函数值之间可以比较：若引用的是相同函数或都是 `nil` ，则认为是相同的函数

### 函数的参数和返回值

任何一个有返回值（单个或多个）的函数都必须以 `return` 或 `panic`结尾

在函数调用时，切片 (slice)、字典 (map)、接口 (interface)、通道 (channel) 这样的引用类型都是默认使用引用传递（即使没有显式的指出指针）

#### 命名返回值

命名返回值作为结果形参被初始化为相应类型的零值，当需要返回的时候，我们只需要一条简单的不带参数的 `return` 语句，即使只有一个命名返回值也需用括号括起来；当有多个非命名返回值时需用括号括起来，如`(int,int)`，任何一个非命名返回值在`return`中都要指出返回值变量或是一个可计算的值

#### 传递变长参数

若变长参数的类型并不都相同时的传递方法：

1. 定义一个结构类型，假设它叫 `Options`，用以存储所有可能的参数：

   ```go
   type Options struct {
   	par1 type1,
   	par2 type2,
   	...
   }
   ```

   函数 `F1()` 可以使用正常的参数 `a` 和 `b`，以及一个没有任何初始化的 `Options` 结构： `F1(a, b, Options {})`。如果需要对选项进行初始化，则可以使用 `F1(a, b, Options {par1:val1, par2:val2})`

2. 使用空接口：

   若一个变长参数的类型没有被指定，则可以使用默认的空接口`interface{}`，该方案不仅可以用于长度未知的参数，还可以用于任何不确定类型的参数：

   ```go
   func typecheck(..,..,values … interface{}) {
   	for _, value := range values {
   		switch v := value.(type) {
   			case int: …
   			case float: …
   			case string: …
   			case bool: …
   			default: …
   		}
   	}
   }
   ```

### defer和追踪

关键字 `defer` 允许我们推迟到函数返回之前（或任意位置执行 `return` 语句之后）一刻才执行某个语句或函数

当有多个 `defer` 行为被注册时，它们会以逆序执行（类似栈，即后进先出）

关键字 `defer` 允许我们进行一些函数执行完成后的收尾工作

#### 使用`defer`实现代码追踪

```go
package main

import "fmt"

func trace(s string) string {
	fmt.Println("entering:", s)
	return s
}

func un(s string) {
	fmt.Println("leaving:", s)
}

func a() {
	defer un(trace("a"))
	fmt.Println("in a")
}

func b() {
	defer un(trace("b"))
	fmt.Println("in b")
	a()
}

func main() {
	b()
}
```

```
entering: b
in b
entering: a
in a
leaving: a
leaving: b

```

#### 使用 `defer` 语句记录函数的参数与返回值

```go
package main

import (
	"io"
	"log"
)

func func1(s string) (n int, err error) {
	defer func() {
		log.Printf("func1(%q) = %d, %v", s, n, err)
	}()
	return 7, io.EOF
}

func main() {
	func1("Go")
}

```

```
Output: 2011/10/04 10:46:11 func1("Go") = 7, EOF

```

### 内置函数

| 名称                             | 说明                                                         |
| -------------------------------- | ------------------------------------------------------------ |
| `close()`                        | 用于管道通信                                                 |
| `len()`、`cap()`                 | `len()` 用于返回某类型的长度或数量（字符串、数组、切片、`map` 和管道）；`cap()` 用于返回某个类型的最大容量（只能用于数组、切片和管道，不能用于 `map`） |
| `new()`、`make()`                | 分配内存，`new()` 值类型和自定义类型，`make` 内置引用类型（切片、`map` 和管道） |
| `copy()`、`append()`             | 用于复制和连接切片                                           |
| `panic()`、`recover()`           | 均用于错误处理机制                                           |
| `print()`、`println()`           | 底层打印函数，在部署环境中建议使用`fmt`包                    |
| `complex()`、`real ()`、`imag()` | 用于创建和操作复数                                           |

### 闭包

可以将匿名函数赋值给变量，即保存函数的地址到变量中，然后通过变量名对函数进行调用，也可以直接对匿名函数进行调用，匿名函数也被称为闭包：

`func(x, y int) int { return x + y } (3, 4)`

#### 应用闭包：将参数作为返回值

```go
package main

import "fmt"

func main() {
    var f = Adder()
    fmt.Print(f(1),"-")
    fmt.Print(f(20),"-")
    fmt.Print(f(300))
}

func Adder() func(int) int {
    var x int
    return func(delta int) int {
        x += delta
        return x
    }
}

//output
//1 - 21 - 321

```

闭包函数保存并积累其中的变量的值，不管外部函数退出与否，它都能够继续操作外部函数中的局部变量

##### 工厂函数

一个返回值为另一个函数的函数；在创建一系列相似函数时非常有用

```go
func MakeAddSuffix(suffix string) func(string) string {
    return func(name string) string {
        if !strings.HasSuffix(name, suffix) {
			return name + suffix
		}
		return name
    }
}

addBmp := MakeAddSuffix(".bmp")
addJpeg := MakeAddSuffix(".jpeg")
addBmp("file") // returns: file.bmp
addJpeg("file") // returns: file.jpeg

```

#### 使用闭包调试

在分析和调试复杂的程序时，无数个函数在不同的代码文件中相互调用，如果这时候能够准确地知道哪个文件中的具体哪个函数正在执行，对于调试是十分有帮助的；包 `runtime` 中的函数 `Caller()` 提供了相应的信息，因此可以在需要的时候实现一个 `where()` 闭包函数来打印函数执行的位置：

```go
where := func() {
    _,file,line,_ := runtime.Caller(1)
    log.Printf("%s:%d",file,line)
}
where()

```

#### 通过内存缓存提升性能

在大量计算时，提升性能最直接有效的方式即避免重复计算，在缓存重复利用相同计算的结果称为内存缓存

如斐波那契数列，要计算数列中第 n 个数字，需要先得到之前两个数的值，但很明显绝大多数情况下前两个数的值都是已经计算过的，此时将第 n 个数的值存在数组中索引为 n 的位置

## 数组和切片

#### 数组

数组是具有相同 **唯一类型** 的一组已编号且长度固定的数据项序列，声明格式为：

`var arr1 [5]int`

数组的长度是数组类型的一个组成部分，因此[3]int和[4]int是两种不同的数组类型。数组的长度必须是常量表达式，因为数组的长度需要在编译阶段确定

go数组是值类型，可通过`new()`创建：`var arr1 = new([5]int)`，故在函数中作为参数传入时不修改原始数组，若想修改，则需使用`&`操作符以引用方式传入

该种方式和 `var arr2 [5]int` 的区别是：`arr1` 的类型是 `*[5]int`，而 `arr2` 的类型是 `[5]int`；这样的结果就是当把一个数组赋值给另一个时，需要再做一次数组内存的拷贝操作：

```go
arr2 := *arr1
arr2[2] = 100
//这样两个数组就有了不同的值，在赋值后修改 arr2 不会对 arr1 生效

```

#### 切片

切片提供了计算容量的函数 `cap()` 可以测量切片最长可以达到多少：切片的长度 + 数组除切片之外的长度。若`s` 是一个切片，`cap(s)` 就是从 `s[0]` 到数组末尾的数组长度：`0 <= len(s) <= cap(s)`

slice由三个部分构成：指针、长度和容量

slice不能直接用`==`进行比较，对于字节型slice可以使用`bytes.Equal`函数判断是否相等，对于其它类型需要展开每个元素比较（slice允许与nil比较）

一个nil值的slice的行为和其它任意0长度的slice一样；若需要判断一个slice是否为空，需要使用len(s) == 0 判断，不应该使用s == nil

可以将 `s2` 向后移动一位 `s2 = s2[1:]`，但是`s2 = s2[-1:]` 会导致编译错误，切片不能被重新分片获取数组的前一个元素

##### 创建切片

当相关数组未定义时，可以用 `make()` 创建切片，同时创建好相关数组：`var slice1 []type = make([]type, len)`；可以简写为`slice1 := make([]type,len)`，`len`是数组的长度且是`slice`的初始长度

故定义`s2 := make([]int,10)`，则`cap(s2)==len(s2)==10`

若想创建一个 `slice1`不占用整个数组，只占用以 `len` 为个数个项，那么只要：`slice1 := make([]type, len, cap)`

`make()` 的使用方式是：`func make([]T, len, cap)`，其中 `cap` 是可选参数

```go
make([]int, 50, 100)
new([100]int)[0:50]

```

##### 切片的扩容

###### 扩容函数

```go
func growslice(et *_type,old slice,cap int) slice {
    newcap := old.cap
    doublecap := newcap + newcap
    if cap > doublecap {
        newcap = cap
    }else{
        if old.cap < 1024 {
            newcap = doublecap
        }else{
            for 0 < newcap && newcap < cap {
                newcap += newcap/4
            }
            if newcap <= 0 {
                newcap = cap
            }
        }
    }
}

```

###### 扩容原理

- 当前所需容量（cap）大于原容量两倍（doublecap），则最终申请容量（newcap）为当前所需容量（cap）
- 若不大于原容量两倍时
  1. 原切片长度小于1024则申请原容量的两倍
  2. 否则，最终申请容量（newcap，初始值等于 old.cap）每次增加newcap/4，直到大于所需容量（cap）为止，然后判断最终容量是否溢出，若溢出，最终申请容量等于所需容量

###### go切片扩容为什么是2倍

1. 确定切片的大致容量

2. 根据元素所占字节大小，最终确定容量

   当元素所占字节大小为1、8或2的倍数时，会执行内存对齐操作

##### 多维切片

 Go 的多维切片可以任意切分，而且，内层的切片必须单独分配

##### bytes包

`bytes` 包和字符串包十分类似，而且还包含一个十分有用的类型 `Buffer`

长度可变的 `bytes` 的 buffer，提供 `Read()` 和 `Write()` 方法，读写长度未知的 `bytes` 最好使用 `buffer`

###### buffer定义

- `var buffer bytes.Buffer`
- `var r *bytes.Buffer = new(bytes.Buffer)`
- `func NewBuffer(buf []byte) *Buffer`；`NewBuffer` 最好用在从 `buf` 读取的时候

###### 通过buffer串联字符串

```go
var buffer bytes.Buffer
for {
    if s,ok := getNextString();ok{
        buffer.WriteString(s)
    }else{
        break
    }
}
fmt.Print(buffer.String(), "\n")

```

该方式比使用`+=`更节省内存和CPU，尤其是串联的字符串数目较多时

#### new()和make()的区别

两者都在堆上分配内存，但它们的行为不同，适用于不同类型

- `new(T)` 为每个新类型 `T` 分配一片内存，初始化为 `0` 并且返回类型为 `*T` 的内存地址：这种方法 **返回一个指向类型为 `T`，值为 `0` 的地址的指针**，它适用于值类型如数组和结构体，相当于 `&T{}`
- `make(T)` **返回一个类型为 T 的初始值**，它只适用于 3 种内建的引用类型：切片、`map` 和 `channel`

即`new()`分配内存，`make()`初始化，如下图所示：

<img src="https://s1.ax1x.com/2023/03/09/ppm4rY6.png" style="zoom:50%"/>

#### for-range结构

可以应用于数组和切片

```go
for ix, value := range slice1 {
	...
}

```

第一个返回值 `ix` 是数组或者切片的索引，第二个是在该索引位置的值

`value` 只是 `slice1` 某个索引位置的值的一个拷贝，不能用来修改 `slice1` 该索引位置的值

#### 切片重组

在切片达到容量上限后扩容改变切片长度的过程

#### 切片的复制与追加

如果想增加切片的容量，必须创建一个更大的切片并把原分片的内容都拷贝过来

可以使用`copy()`和`append()`函数，追加的元素必须和原切片元素是同类型

##### append

如果 `s` 的容量不足以存储新增元素，`append()` 会分配新的切片来保证已有切片元素和新增元素的存储。因此，返回的切片可能已经指向一个不同的相关数组，`append()` 方法总是返回成功，除非系统内存耗尽

```go
func AppendByte(slice []byte, data ...byte) []byte {
    m := len(slice)
    n := m + len(data)
    if n > cap(slice) {
        newSlice := make([]byte,(n+1)*2)
        copy(newSlice,slice)
        slice = newSlice
    }
    slice = slice[0:n]
    copy(slice[m:n],data)
    return slice
}

```

###### append常见操作

1. 将切片 `b` 的元素追加到切片 `a` 之后：`a = append(a, b...)`

2. 复制切片 `a` 的元素到新的切片 `b` 上：

   ```go
   b = make([]T, len(a))
   copy(b, a)
   
   ```

3. 删除位于索引 `i` 的元素：`a = append(a[:i], a[i+1:]...)`

4. 切除切片 `a` 中从索引 `i` 至 `j` 位置的元素：`a = append(a[:i], a[j:]...)`

5. 为切片 `a` 扩展 `j` 个元素长度：`a = append(a, make([]T, j)...)`

6. 在索引 `i` 的位置插入元素 `x`：`a = append(a[:i], append([]T{x}, a[i:]...)...)`

7. 在索引 `i` 的位置插入长度为 `j` 的新切片：`a = append(a[:i], append(make([]T, j), a[i:]...)...)`

8. 在索引 `i` 的位置插入切片 `b` 的所有元素：`a = append(a[:i], append(b, a[i:]...)...)`

9. 取出位于切片 `a` 最末尾的元素 `x`：`x, a = a[len(a)-1], a[:len(a)-1]`

10. 将元素 `x` 追加到切片 `a`：`a = append(a, x)`

##### copy

`func copy(dst, src []T) int` 方法将类型为 `T` 的切片从源地址 `src` 拷贝到目标地址 `dst`，覆盖 `dst` 的相关元素，并且返回拷贝的元素个数。源地址和目标地址可能会有重叠。拷贝个数是 `src` 和 `dst` 的长度最小值。若 `src` 是字符串则元素类型就是 `byte`。若还想继续使用 `src`，在拷贝结束后执行 `src = dst`

#### 字符串生成字节切片

可以通过`c := []byte(s)`获取一个字节的切片`c`，还可以通过 `copy()` 函数来达到相同的目的：`copy(dst []byte, src string)`

将一个字符串追加到某一个字节切片的尾部：

```go
var b []byte
var s string
b = append(b, s...)

```

#### 字符串和切片的内存结构

在内存中，一个字符串实际上是一个双字结构，即一个指向实际数据的指针和记录字符串长度的整数。因为指针对用户来说是完全不可见，因此我们可以依旧把字符串看做是一个值类型，也就是一个字符数组

字符串 `string s = "hello"` 和子字符串 `t = s[2:3]` 在内存中的结构可以用下图表示：

<img src="https://s1.ax1x.com/2023/03/09/ppmHohn.png" style="zoom:50%"/>

#### 修改字符串的某个字符

Go 中字符串不可变，即 `str[index]` 这样的表达式是不可以被放在等号左侧的

因此必须先将字符串转换成字节数组，然后再通过修改数组中的元素值来达到修改字符串的目的，最后将字节数组转换回字符串格式

```go
s := "hello"
c := []byte(s)
c[0] = 'c'
s2 := string(c)	//s2 == "cello"

```

#### 字节数组对比函数

```go
func Compare(a, b[]byte) int {
    for i:=0; i < len(a) && i < len(b); i++ {
        switch {
        case a[i] > b[i]:
            return 1
        case a[i] < b[i]:
            return -1
        }
    }
    // 数组的长度可能不同
    switch {
    case len(a) < len(b):
        return -1
    case len(a) > len(b):
        return 1
    }
    return 0 // 数组相等
}

```

#### 搜索及排序切片和数组

标准库提供了 `sort` 包实现常见的搜索和排序操作。可以使用 `sort` 包中的函数 `func Ints(a []int)` 实现对 `int` 类型的切片排序。例如 `sort.Ints(arri)`，其中变量 `arri` 就是需要被升序排序的数组或切片。为了检查某个数组是否已经被排序，可通过函数 `IntsAreSorted(a []int) bool` 检查，若返回 `true` 则表示已经被排序

类似的，可以使用函数 `func Float64s(a []float64)` 来排序 `float64` 的元素，或使用函数 `func Strings(a []string)` 排序字符串元素

想要在数组或切片中搜索一个元素，该数组或切片必须先被排序（因为标准库的搜索算法使用的是二分法）。使用函数 `func SearchInts(a []int, n int) int` 进行搜索，并返回对应结果的索引值

当然，还可以搜索 `float64` 和字符串：

```go
func SearchFloat64s(a []float64, x float64) int
func SearchStrings(a []string, x string) int

```

#### 切片和垃圾回收

只有在没有任何切片指向的时候，底层的数组内存才会被释放，这种特性有时会导致程序占用多余的内存

**示例** 函数 `FindDigits()` 将一个文件加载到内存，然后搜索其中所有的数字并返回一个切片。

```go
var digitRegexp = regexp.MustCompile("[0-9]+")

func FindDigits(filename string) []byte {
    b, _ := ioutil.ReadFile(filename)
    return digitRegexp.Find(b)
}

```

这段代码可以顺利运行，但返回的 `[]byte` 指向的底层是整个文件的数据。只要该返回的切片不被释放，垃圾回收器就不能释放整个文件所占用的内存。换句话说，一点点有用的数据却占用了整个文件的内存。

想要避免这个问题，可以通过拷贝我们需要的部分到一个新的切片中：

```go
func FindDigits(filename string) []byte {
   b, _ := ioutil.ReadFile(filename)
   b = digitRegexp.Find(b)
   c := make([]byte, len(b))
   copy(c, b)
   return c
}

```

事实上，上面这段代码只能找到第一个匹配正则表达式的数字串。要想找到所有的数字，可以尝试下面这段代码：

```go
func FindFileDigits(filename string) []byte {
   fileBytes, _ := ioutil.ReadFile(filename)
   b := digitRegexp.FindAll(fileBytes, len(fileBytes))
   c := make([]byte, 0)
   for _, bytes := range b {
      c = append(c, bytes...)
   }
   return c
}

```

## Map

### 键值对元素

`val1,isPresent = map1[key1]`

`isPresent`返回一个`bool`值：若 `key1` 存在于 `map1`，`val1` 就是 `key1` 对应的 `value` 值，并且 `isPresent` 为 `true`；若`key1` 不存在，`val1` 就是一个空值，且 `isPresent` 返回 `false`

判断某个`key`是否存在的常规方法：

```go
if _, ok := map1[key1]; ok {
	// ...
}

```

在`map1`中删除`key1`：`delete(map1,key1)`；若`key1`不存在，该操作不会产生错误

 `map` 不是按照 key 的顺序排列的，也不是按照 value 的序排列的

> map 的本质是散列表，而 map 的增长扩容会导致重新进行散列，这就可能使 map 的遍历结果在扩容前后变得不可靠，Go 设计者为了让大家不依赖遍历的顺序，每次遍历的起点--即起始 bucket 的位置不一样，即不让遍历都从某个固定的 bucket0 开始，所以即使未扩容时我们遍历出来的 map 也总是无序的

### map类型的切片

获取一个 `map` 类型的切片，必须使用两次 `make()` 函数，第一次分配切片，第二次分配切片中每个 `map` 元素

```go
package main
import "fmt"

func main() {
	// Version A:
    items := make([]map[int]int,5)
    for i := range items {
        items[i] = make(map[int]int, 1)
		items[i][1] = 2
    }
    fmt.Printf("Version A: Value of items: %v\n", items)
	
    // Version B: NOT GOOD!
	items2 := make([]map[int]int, 5)
	for _, item := range items2 {
		item = make(map[int]int, 1) // item is only a copy of the slice element.
		item[1] = 2 // This 'item' will be lost on the next iteration.
	}
	fmt.Printf("Version B: Value of items: %v\n", items2)
}

```

输出结果：

```
Version A: Value of items: [map[1:2] map[1:2] map[1:2] map[1:2] map[1:2]]
Version B: Value of items: [map[] map[] map[] map[] map[]]

```

 A 通过索引使用切片的 `map` 元素。 B 中获得的项只是 `map` 值的一个拷贝，所以真正的 `map` 元素没有得到初始化

### 将map的键值对调

若`map`的值类型可以作为key且所有value是唯一的，则可以通过下面的方法对调：

```go
package main
import (
	"fmt"
)

var (
	barVal = map[string]int{"alpha": 34, "bravo": 56, "charlie": 23,
							"delta": 87, "echo": 56, "foxtrot": 12,
							"golf": 34, "hotel": 16, "indio": 87,
							"juliet": 65, "kili": 43, "lima": 98}
)

func main() {
	invMap := make(map[int]string, len(barVal))
	for k, v := range barVal {
		invMap[v] = k
	}
	fmt.Println("inverted:")
	for k, v := range invMap {
		fmt.Printf("Key: %v, Value: %v / ", k, v)
	}
}

```

## 结构体和方法

### 工厂方法

将创建对象的具体过程屏蔽隔离；由工厂创建不同种类的对象

```go
type File struct {
    fd      int     // 文件描述符
    name    string  // 文件名
}

func NewFile(fd int, name string) *File {
    if fd < 0 {
        return nil
    }

    return &File{fd, name}
}

f := NewFile(10, "./test.txt")

```

若 `File` 是一个结构体类型，则 `new(File)` 和 `&File{}` 是等价的

#### 强制使用工厂方法

##### 可见性规则

当标识符（包括常量、变量、类型、函数名、结构字段等等）以一个大写字母开头，则这种形式的标识符的对象就可以被外部包的代码所使用（像面向对象语言中的 public）；标识符如果以小写字母开头，则对包外是不可见的，但是它们在整个包的内部是可见并且可用的（像面向对象语言中的 private ）

当使用 `.` 作为包的别名时，可以不通过包名来使用其中的项目：`import . "./pack1"`

`import _ "./pack1/pack1"`

`pack1` 包只导入其副作用，也就是说，只执行它的 `init()` 函数并初始化其中的全局变量

```go
type matrix struct {
    ...
}

func NewMatrix(params) *matrix {
    m := new(matrix) // 初始化 m
    return m
}

```

在其他包中使用工厂方法：

```go
package main
import "matrix"
...
wrong := new(matrix.matrix)     // 编译失败（matrix 是私有的）
right := matrix.NewMatrix(...)  // 实例化 matrix 的唯一方式

```

### 内嵌结构体和匿名字段

结构体可以包含一个或多个 匿名（或内嵌）字段，只有字段的类型是必须的，此时类型就是字段的名字，因此每种数据类型只能有一个匿名字段

```go
package main

import "fmt"

type innerS struct {
	in1 int
	in2 int
}

type outerS struct {
	b    int
	c    float32
	int  // anonymous field
	innerS //anonymous field
}

func main() {
	outer := new(outerS)
	outer.b = 6
	outer.c = 7.5
	outer.int = 60
	outer.in1 = 5
	outer.in2 = 10

	fmt.Printf("outer.b is: %d\n", outer.b)
	fmt.Printf("outer.c is: %f\n", outer.c)
	fmt.Printf("outer.int is: %d\n", outer.int)
	fmt.Printf("outer.in1 is: %d\n", outer.in1)
	fmt.Printf("outer.in2 is: %d\n", outer.in2)

	// 使用结构体字面量
	outer2 := outerS{6, 7.5, 60, innerS{5, 10}}
	fmt.Println("outer2 is:", outer2)
}

```

#### 命名冲突

当两个字段拥有相同的名字（可能是继承来的名字）时：

1. 外层名字会覆盖内层名字（但是两者的内存空间都保留），这提供了一种重载字段或方法的方式
2. 如果相同的名字在同一级别出现了两次，如果这个名字被程序使用了，将会引发一个错误（不使用没关系）。没有办法来解决这种问题引起的二义性

```go
type A struct {a int}
type B struct {a, b int}

type C struct {A; B}
var c C

type D struct {B; b float32}
var d D

```

此时使用`c.a`是错误的；使用`d.b`没有问题

## 接口

- 类型无需显示声明实现了某个接口，接口可以被隐式地实现，多个类型可以实现同一接口
- 实现某个接口的类型（除了实现接口方法外）可以有其他的方法
- 一个类型可以实现多个接口
- 接口类型可以包含一个实例的引用，该实例类型实现了此接口（接口是动态类型）

```go
package main

import "fmt"

type Shaper interface {
	Area() float32
}

type Square struct {
	side float32
}

func (sq *Square) Area() float32 {
	return sq.side * sq.side
}

func main() {
	sq1 := new(Square)
	sq1.side = 5

	var areaIntf Shaper
	areaIntf = sq1
	// shorter,without separate declaration:
	// areaIntf := Shaper(sq1)
	// or even:
	// areaIntf := sq1
	fmt.Printf("The square has area: %f\n", areaIntf.Area())
}

//输出
//The square has area: 25.000000

```

上述实例中创建了一个`Square`的实例，且在主程序外定义了一个接受者类型是`Square`的`Area()`：结构体`Square`实现了接口`Shaper` 

故可将一个 `Square` 类型的变量赋值给一个接口类型的变量：`areaIntf = sq1` 

现在接口变量包含一个指向 `Square` 变量的引用，通过它可以调用 `Square` 上的方法 `Area()`；接口变量里包含了接收者实例的值和指向对应方法表的指针；**这是多态的Go版本**

如果 `Shaper` 有另外一个方法 `Perimeter()`，但是 `Square` 没有实现它，即使没有在 `Square` 实例上调用这个方法，编译器会给出错误：

```
cannot use sq1 (type *Square) as type Shaper in assignment:
*Square does not implement Shaper (missing Area method)

```

### 接口嵌套接口

一个接口可以包含一个或多个其他的接口，相当于直接将这些内嵌接口的方法列举在外层接口中。

```go
type ReadWrite interface {
    Read(b Buffer) bool
    Write(b Buffer) bool
}

type Lock interface {
    Lock()
    Unlock()
}

type File interface {
    ReadWrite
    Lock
    Close()
}

```

### 类型断言

在接口值上的操作，用于检查接口类型变量所持有的值是否实现了期望的接口或者具体的类型；语法格式如下：

```go
value,ok := x.(T)

```

其中，x 表示一个接口的类型，T 表示一个具体的类型（也可为接口类型）

该断言表达式会返回 x 的值（ value）和一个布尔值（ ok），可根据该布尔值判断 x 是否为 T 类型：

- 若T是具体某个类型，则检查x的动态类型是否等于具体类型T；若成功，则返回x的动态值，类型为T
- 若T 是接口类型，检查 x 的动态类型是否满足 T；若成功，x 的动态值不被提取，返回一个类型为 T 的接口值
- 无论 T 是什么类型，如果 x 是 nil 接口值，类型断言都会失败

通过类型断言可以做到：

- 检查`i`是否为`nil`
- 检查`i`存储的值是否为某个类型

```go
var i interface{} = 10
t1 := i.(int)
fmt.Println(t1)
fmt.Println("====分隔线====")
t2 := i.(string)
fmt.Println(t2)

```

```
10
=====分隔线=====
panic: interface conversion: interface {} is int, not string
goroutine 1 [running]:
main.main()
	E:/GoPlayer/src/main.go:12 +0x10e
exit status 2

```

#### Type Switch

```go
func classifier(items ...interface{}) {
	for i, x := range items {
		switch x.(type) {
		case bool:
			fmt.Printf("Param #%d is a bool\n", i)
		case float64:
			fmt.Printf("Param #%d is a float64\n", i)
		case int, int64:
			fmt.Printf("Param #%d is a int\n", i)
		case nil:
			fmt.Printf("Param #%d is a nil\n", i)
		case string:
			fmt.Printf("Param #%d is a string\n", i)
		default:
			fmt.Printf("Param #%d is unknown\n", i)
		}
	}
}

```

### 接口实例：使用Sorter接口排序

`sort` 包要对一组数字或字符串排序，需要实现三个方法：反映元素个数的 `Len()` 方法、比较第 `i` 和 `j` 个元素的 `Less(i, j)` 方法以及交换第 `i` 和 `j` 个元素的 `Swap(i, j)` 方法

排序函数的算法只会使用到这三个方法（可以使用任何排序算法来实现，此处我们使用冒泡排序）：

```go
func Sort(data Sorter) {
    for pass := 1; pass < data.Len(); pass++ {
        for i := 0;i < data.Len() - pass; i++ {
            if data.Less(i+1, i) {
                data.Swap(i, i + 1)
            }
        }
    }
}

```

`Sort` 函数接收一个接口类型的参数：`Sorter` ，它声明了这些方法：

```go
type Sorter interface {
    Len() int
    Less(i, j int) bool
    Swap(i, j int)
}

```

 `int` 是待排序序列长度的类型，而不是说要排序的对象一定要是一组 `int`，现在若想对一个 `int` 数组排序：

```go
type IntArray []int
func (p IntArray) Len() int           { return len(p) }
func (p IntArray) Less(i, j int) bool { return p[i] < p[j] }
func (p IntArray) Swap(i, j int)      { p[i], p[j] = p[j], p[i] }

```

```go
data := []int{74, 59, 238, -784, 9845, 959, 905, 0, 0, 42, 7586, -5467984, 7586}
a := sort.IntArray(data) //conversion to type IntArray from package sort
sort.Sort(a)

```

### 空接口

**空接口或者最小接口** 不包含任何方法，它对实现不做任何要求：

```go
type Any interface{}

```

可以给一个空接口类型的变量 `var val interface {}` 赋任何类型的值

每个 `interface {}` 变量在内存中占据两个字长：一个用来存储它包含的类型，另一个用来存储它包含的数据或者指向数据的指针

#### 构建通用类型

前面的排序实例中可以实现对`int`、`float`、`string`数组的排序，对于其他类型的排序可以使用空接口：`Element：type Element interface{}`

然后定义一个容器类型结构体`Vector`，包含一个`Element`类型元素的切片：

```go
type Vector struct {
    a []Element
}

```

`Vector` 里能放任何类型的变量，因为任何类型都实现了空接口，实际上 `Vector` 里放的每个元素可以是不同类型的。我们为它定义一个 `At()` 方法用于返回第 `i` 个元素，再定一个 `Set()` 方法用于设置第 `i` 个元素的值：

```go
func (p *Vector) At(i int) Element {
	return p.a[i]
}

func (p *Vector) Set(i int, e Element) {
	p.a[i] = e
}

```

`Vector` 中存储的所有元素都是 `Element` 类型，要得到它们的原始类型（unboxing：拆箱）需要用到类型断言

##### 通用类型节点数据结构

在列表和树等数据结构中定义时使用了节点的递归型结构体类型，现在可以使用空接口作为数据字段的类型：

```go
package main

import "fmt"

type Node struct {
	le   *Node
	data interface{}
	ri   *Node
}

func NewNode(left, right *Node) *Node {
	return &Node{left, nil, right}
}

func (n *Node) SetData(data interface{}) {
	n.data = data
}

func main() {
	root := NewNode(nil, nil)
	root.SetData("root node")
	// make child (leaf) nodes:
	a := NewNode(nil, nil)
	a.SetData("left node")
	b := NewNode(nil, nil)
	b.SetData("right node")
	root.le = a
	root.ri = b
	fmt.Printf("%v\n", root) // Output: &{0x125275f0 root node 0x125275e0}
}

```

#### 复制数据切片至空接口切片

若有一个 `myType` 类型的数据切片，将切片中的数据复制到一个空接口切片中：

```go
var dataSlice []myType = FuncReturnSlice()
var interfaceSlice []interface{} = dataSlice

```

**此时将会编译出错：**`cannot use dataSlice (type []myType) as type []interface { } in assignment`

因为它们两个在内存中的布局不一样，必须用`for-range`一个一个显示赋值：

```go
var dataSlice []myType = FuncReturnSlice()
var interfaceSlice []interface{} = make([]interface{}, len(dataSlice))
for i, d := range dataSlice {
    interfaceSlice[i] = d
}

```

#### 接口到接口

一个接口的值可以赋值给另一个接口变量，只要底层类型实现了必要的方法。这个转换是在运行时进行检查的，转换失败会导致一个运行时错误：这是 `Go` 语言动态的一面，可以拿它和 `Ruby` 和 `Python` 这些动态语言相比较

```go
var ai AbsInterface		// declares method Abs()
type SqrInterface interface {
    Sqr() float
}
var si SqrInterface
pp := new(Point)		// say *Point implements Abs, Sqr
var empty interface{}
empty = pp				// everything satisfies empty
ai = empty.(AbsInterface) // underlying value pp implements Abs()
// (runtime failure otherwise)
si = ai.(SqrInterface) 	// *Point has Sqr() even though AbsInterface doesn’t
empty = si				// *Point implements empty set
// Note: statically checkable so type assertion not necessary.

```

```go
type myPrintInterface interface {
	print()
}

func f3(x myInterface) {
	x.(myPrintInterface).print() // type assertion to myPrintInterface
}

```

`x` 转换为 `myPrintInterface` 类型是完全动态的：只要 `x` 的底层类型（动态类型）定义了 `print` 方法这个调用就可以正常运行（译注：若 `x` 的底层类型未定义 `print` 方法，此处类型断言会导致 `panic`，最佳实践应该为 `if mpi, ok := x.(myPrintInterface); ok { mpi.print() }`

### 反射包

#### 方法和类型的反射

反射包的 `Type` 表示一个 Go 类型，反射包的 `Value` 为 Go 值提供了反射接口

`reflect.TypeOf` 和 `reflect.ValueOf`返回被检查对象的类型和值

实际上，反射是通过检查一个接口的值，变量首先被转换一个空接口：

```go
func TypeOf(i interface{}) Type
func ValueOf(i interface{}) Value

```

反射可以从接口值反射到对象，也可以从对象反射回接口值

 `Type` 和 `Value` 都有 `Kind()` 方法返回一个常量来表示类型， `Kind()` 总是返回底层类型，同样 `Value` 有叫做 `Int()` 和 `Float()` 的方法可以获取存储在内部的值

变量 `v` 的 `Interface()` 可以得到还原（接口）值，可以这样打印 `v` 的值：`fmt.Println(v.Interface())`

```go
// blog: Laws of Reflection
package main

import (
	"fmt"
	"reflect"
)

func main() {
	var x float64 = 3.4
	fmt.Println("type:", reflect.TypeOf(x))
	v := reflect.ValueOf(x)
	fmt.Println("value:", v)
	fmt.Println("type:", v.Type())
	fmt.Println("kind:", v.Kind())
	fmt.Println("value:", v.Float())
	fmt.Println(v.Interface())
	fmt.Printf("value is %5.2e\n", v.Interface())
	y := v.Interface().(float64)
	fmt.Println(y)
}

```

```go
type: float64
value: 3.4
type: float64
kind: float64
value: 3.4
3.4
value is 3.40e+00
3.4

```

#### 通过反射设置值

可以使用 `CanSet()` 方法测试是否可设置反射值，并不是所有的都有该属性

若要将上述`v`的值设置为`3.1415`可以小心使用`v.SetFloat(3.1415)`，但该方法不一定能够成功

可以使用 `Elem()` 函数，这间接地使用指针：`v = v.Elem()`

```go
package main

import (
	"fmt"
	"reflect"
)

func main() {
	var x float64 = 3.4
	v := reflect.ValueOf(x)
	// setting a value:
	// v.SetFloat(3.1415) // Error: will panic: reflect.Value.SetFloat using unaddressable value
	fmt.Println("settability of v:", v.CanSet())
	v = reflect.ValueOf(&x) // Note: take the address of x.
	fmt.Println("type of v:", v.Type())
	fmt.Println("settability of v:", v.CanSet())
	v = v.Elem()
	fmt.Println("The Elem of v is: ", v)
	fmt.Println("settability of v:", v.CanSet())
	v.SetFloat(3.1415) // this works!
	fmt.Println(v.Interface())
	fmt.Println(v)
}

```

```go
settability of v: false
type of v: *float64
settability of v: false
The Elem of v is:  <float64 Value>
settability of v: true
3.1415
<float64 Value>

```

反射中有些内容是需要用地址去改变它的状态的

#### 反射结构

有时需要反射一个结构类型。`NumField()` 方法返回结构内的字段数量；通过一个 `for` 循环用索引取得每个字段的值 `Field(i)`

可以调用签名在结构上的方法，使用索引 `n` 来调用：`Method(n).Call(nil)`

```go
package main

import (
	"fmt"
	"reflect"
)

type NotknownType struct {
	s1, s2, s3 string
}

func (n NotknownType) String() string {
	return n.s1 + " - " + n.s2 + " - " + n.s3
}

// variable to investigate:
var secret interface{} = NotknownType{"Ada", "Go", "Oberon"}

func main() {
	value := reflect.ValueOf(secret) // <main.NotknownType Value>
	typ := reflect.TypeOf(secret)    // main.NotknownType
	// alternative:
	// typ := value.Type()  // main.NotknownType
	fmt.Println(typ)
	knd := value.Kind() // struct
	fmt.Println(knd)

	// iterate through the fields of the struct:
	for i := 0; i < value.NumField(); i++ {
		fmt.Printf("Field %d: %v\n", i, value.Field(i))
		// error: panic: reflect.Value.SetString using value obtained using unexported field
		// value.Field(i).SetString("C#")
	}

	// call the first method, which is String():
	results := value.Method(0).Call(nil)
	fmt.Println(results) // [Ada - Go - Oberon]
}

```

```go
main.NotknownType
struct
Field 0: Ada
Field 1: Go
Field 2: Oberon
[Ada - Go - Oberon]

```

但是如果尝试更改一个值，会得到一个错误：

```
panic: reflect.Value.SetString using value obtained using unexported field

```

这是因为结构中只有被导出字段（首字母大写）才是可设置的

```go
package main

import (
	"fmt"
	"reflect"
)

type T struct {
	A int
	B string
}

func main() {
	t := T{23, "skidoo"}
	s := reflect.ValueOf(&t).Elem()
	typeOfT := s.Type()
	for i := 0; i < s.NumField(); i++ {
		f := s.Field(i)
		fmt.Printf("%d: %s %s = %v\n", i,
			typeOfT.Field(i).Name, f.Type(), f.Interface())
	}
	s.Field(0).SetInt(77)
	s.Field(1).SetString("Sunset Strip")
	fmt.Println("t is now", t)
}

```

```go
0: A int = 23
1: B string = skidoo
t is now {77 Sunset Strip}

```

#### Printf()和反射

```go
func Printf(format string, args ... interface{}) (n int, err error)

```

`Printf()` 中的 `...` 参数为空接口类型，它使用反射包来解析这个参数列表，故能够知道它每个参数的类型

### 解码任意数据

json 包使用 `map[string]interface{}` 和 `[]interface{}` 储存任意的 JSON 对象和数组；其可以被反序列化为任何的 JSON blob 存储到接口值中

```go
b := []byte(`{"Name": "Wednesday", "Age": 6, "Parents": ["Gomez", "Morticia"]}`)

```

```go
var f interface{}
err := json.Unmarshal(b, &f)

```

`f`指向的值是一个`map`，key为字符串，value是自身存储作为空接口类型的值

```go
map[string]interface{} {
	"Name": "Wednesday",
	"Age":  6,
	"Parents": []interface{} {
		"Gomez",
		"Morticia",
	},
}

```

要访问该数据可以使用类型断言

```go
m := f.(map[string]interface{})

```

```go
for k, v := range m {
	switch vv := v.(type) {
	case string:
		fmt.Println(k, "is string", vv)
	case int:
		fmt.Println(k, "is int", vv)

	case []interface{}:
		fmt.Println(k, "is an array:")
		for i, u := range vv {
			fmt.Println(i, u)
		}
	default:
		fmt.Println(k, "is of a type I don’t know how to handle")
	}
}

```

通过这种方式可以处理未知的 JSON 数据，同时确保类型安全

### 编码和解码流

`json` 包提供 `Decoder` 和 `Encoder` 类型来支持常用 JSON 数据流读写。`NewDecoder()` 和 `NewEncoder()` 分别封装了 `io.Reader` 和 `io.Writer` 接口

```go
func NewDecoder(r io.Reader) *Decoder
func NewEncoder(w io.Writer) *Encoder

```

要想把 JSON 直接写入文件，可以使用 `json.NewEncoder` 初始化文件（或者任何实现 `io.Writer` 的类型），并调用 `Encode()`；反过来与其对应的是使用 `json.NewDecoder` 和 `Decode()` 函数：

```go
func NewDecoder(r io.Reader) *Decoder
func (dec *Decoder) Decode(v interface{}) error

```

目标或源数据要能够被编码就必须实现 `io.Writer` 或 `io.Reader` 接口。由于 Go 语言中到处都实现了 Reader 和 Writer，因此 `Encoder` 和 `Decoder` 应用场景非常广泛，例如读取或写入 HTTP 连接、websockets 或文件

## 反射

计算机程序在运行时，可以访问、检测和修改它本身状态或行为的能力

在reflect包中定义了一个接口和一个结构体，即reflect.Type和reflect.Value结构体，它们提供了很多函数获取存储在接口中的类型信息

- reflect.Type接口主要提供关于类型相关的信息
- reflect.Value结构体主要提供关于值相关的信息，可以获取甚至改变类型的值

```go
func TypeOf(i interface{}) Type
func ValueOf(i interface{}) Value

```

TypeOf用于提取一个接口中值的类型信息，实参会先被转化为空接口类型，这样，实参的类型信息、方法集、值信息都存储到interface{}中了；ValueOf()返回一个结构体变量，包含类型信息及实际值

#### 反射的三大法则

- 反射可以将`接口类型变量`转换为`反射类型对象`

  ```go
  var x float64 = 6.8
  v := reflect.ValueOf(x)
  fmt.Println("type:",v.Type())
  fmt.Println("Kind is float64:",v.Kind() == reflect.Float64)
  fmt.Println("value",v.Float())
  
  ```

- 反射可以将`反射类型对象`转换为`接口类型变量`

  Go中的反射可以创造自己反面类型的对象，一个`reflect.Value`类型的变量，可以使用`Interface()`方法恢复其接口类型的值；该方法会将type和value信息打包并填充到一个接口变量中，然后返回

  ```go
  y := v.Interface().(float64)
  fmt.Println(y)
  
  ```

- 若要修改`反射类型对象`，则其值必须是`可写的`

  要让反射对象具备可写性需要注意：创建反射对象时传入变量的是指针；使用Elem()方法返回指针指向的数据

  - 不是接收变量指针创建的反射对象，是不具备可写性的

  - 是否具备可写性，可使用CanSet方法得知

  - 对不具备可写性的对象修改是无意义的，也是不合法的

    ```go
    var name string = "Go web"
    
    v1 := reflect.ValueOf(&name)
    v2 := v1.Elem()
    
    //v1不具有可写性，v2具有可写性
    
    ```

#### 修改反射对象

```go
func (v Value) SetBool(x bool)
func (v Value) SetBytes(x []byte)
func (v Value) SetFloat(x float64)
func (v Value) SetInt(x int64)
func (v Value) SetString(x string)

```

## 读写数据

### 文件读写

#### 读文件

标准输入 `os.Stdin` 和标准输出 `os.Stdout`的类型都是 `*os.File`；文件使用指向 `os.File` 类型的指针表示，也即文件句柄

```go
package main

import (
    "bufio"
    "fmt"
    "io"
    "os"
)

func main() {
    inputFile, _ := os.Open("input.dat")
    defer inputFile.Close()

    inputReader := bufio.NewReader(inputFile)
    for {
        inputString, readerError := inputReader.ReadString('\n')
        fmt.Printf("The input was: %s", inputString)
        if readerError == io.EOF {
            return
        }      
    }
}

//若文件不存在或无权打开文件，则Open函数会返回一个错误：inputFile, inputError = os.Open
//若文件打开正常，则使用defer inputFile.Close()确保程序退出前关闭该文件；然后使用bufio.NewReader()获得一个读取器变量，通过其可以方便地操作相对高层的string对象，避免操作较底层的字节

```

**注意：** Unix 和 Linux 的行结束符是 `\n`，而 Windows 的行结束符是 `\r\n`；在使用 `ReadString` 和 `ReadBytes` 方法的时候不用关心操作系统类型，直接使用 `\n` 即可。另外也可以使用 `ReadLine()` 方法来实现相同的功能

一旦读取到文件末尾，`readerError`的值变为常量`io.EOF`

##### 将文件内容读入字符串

可以使用 `io/ioutil` 中的 `ioutil.ReadFile()` 方法，该方法第一个返回值的类型是 `[]byte`，里面存放读取到的内容，第二个返回值是错误，如果没有错误发生，第二个返回值为 `nil`

```go
buf, _ := ioutil.ReadFile(inputFile)
fmt.Printf("%s\n", string(buf))

```

##### 带缓冲的读取

有时文件内容不按行划分或为二进制，此时`ReadString()`无法使用；可以使用`bufio.Reader`中的`Read()`

```go
buf := make([]byte, 1024)
...
n, err := inputReader.Read(buf)		//n表示读取到的字节数
if (n == 0) { break}

```

##### 按列读取文件数据

若数据按列排列且用空格分隔，可使用`fmt`中以`FScan...`开头的系列函数读取

```go
package main
import (
    "fmt"
    "os"
)

func main() {
    file, err := os.Open("products2.txt")
    if err != nil {
        panic(err)
    }
    defer file.Close()

    var col1, col2, col3 []string
    for {
        var v1, v2, v3 string
        _, err := fmt.Fscanln(file, &v1, &v2, &v3)
        // scans until newline
        if err != nil {
            break
        }
        col1 = append(col1, v1)
        col2 = append(col2, v2)
        col3 = append(col3, v3)
    }

    fmt.Println(col1)
    fmt.Println(col2)
    fmt.Println(col3)
}

```

```
[ABC FUNC GO]
[40 56 45]
[150 280 356]

```

#### 写文件

```go
package main

import (
	"os"
	"bufio"
	"fmt"
)

func main () {
	// var outputWriter *bufio.Writer
	// var outputFile *os.File
	// var outputError os.Error
	// var outputString string
	outputFile, _ := os.OpenFile("output.dat", os.O_WRONLY|os.O_CREATE, 0666)
	defer outputFile.Close()

	outputWriter := bufio.NewWriter(outputFile)
	outputString := "hello world!\n"

	for i:=0; i<10; i++ {
		outputWriter.WriteString(outputString)
	}
	outputWriter.Flush()
}

```

`OpenFile` 函数有三个参数：文件名、一个或多个标志（使用逻辑运算符 `|` 连接），使用的文件权限

- `os.O_RDONLY`：只读  
- `os.O_WRONLY`：只写  
- `os.O_CREATE`：创建：如果指定文件不存在，就创建该文件 
- `os.O_TRUNC`：截断：如果指定文件已存在，就将该文件的长度截为 0 

在读文件的时候，文件的权限是被忽略的，所以在使用 `OpenFile()` 时传入的第三个参数可以用 0 。而在写文件时，不管是 Unix 还是 Windows，都需要使用 `

然后创建一个写入器（缓冲区）对象：

```go
outputWriter := bufio.NewWriter(outputFile)

```

接着用 `for` 循环写入缓冲区：`outputWriter.WriteString(outputString)`

缓冲区的内容紧接着被完全写入文件：`outputWriter.Flush()`

若写入的东西很简单，可以使用 `fmt.Fprintf(outputFile, "Some test data.\n")` 直接将内容写入文件。`fmt` 包里的 `F...` 开头的 `Print()` 函数可以直接写入任何 `io.Writer`

```go
//不使用fmt.FPrintf()函数写文件
func main() {
	os.Stdout.WriteString("hello, world\n")	//可以输出到屏幕
	f, _ := os.OpenFile("test", os.O_CREATE|os.O_WRONLY, 0666)
	defer f.Close()
	f.WriteString("hello, world in a file\n")
}
//不使用缓冲区，直接将内容写入文件：f.WriteString()

```

### XML数据格式

如同 `json` 包一样，也有 `xml.Marshal()` 和 `xml.Unmarshal()` 从 XML 中编码和解码数据

```go
// xml.go
package main

import (
	"encoding/xml"
	"fmt"
	"strings"
)

var t, token xml.Token
var err error

func main() {
	input := "<Person><FirstName>Laura</FirstName><LastName>Lynn</LastName></Person>"
	inputReader := strings.NewReader(input)
	p := xml.NewDecoder(inputReader)

	for t, err = p.Token(); err == nil; t, err = p.Token() {
		switch token := t.(type) {
		case xml.StartElement:
			name := token.Name.Local
			fmt.Printf("Token name: %s\n", name)
			for _, attr := range token.Attr {
				attrName := attr.Name.Local
				attrValue := attr.Value
				fmt.Printf("An attribute is: %s %s\n", attrName, attrValue)
				// ...
			}
		case xml.EndElement:
			fmt.Println("End of token")
		case xml.CharData:
			content := string([]byte(token))
			fmt.Printf("This is the content: %v\n", content)
			// ...
		default:
			// ...
		}
	}
}

```

```
Token name: Person
Token name: FirstName
This is the content: Laura
End of token
Token name: LastName
This is the content: Lynn
End of token
End of token

```

XML 文本被循环处理直到 `Token()` 返回一个错误，因为已经到达文件尾部，再没有内容可供处理了。通过一个 type-switch 可以根据一些 XML 标签进一步处理。Chardata 中的内容只是一个 `[]byte`，通过字符串转换让其变得可读性更强

## 错误处理

### recover

`recover()`内建函数用于从panic或错误场景中恢复：停止终止过程进而恢复正常执行

`recover` 只能在 `defer` 修饰的函数中使用：取得 `panic()` 调用中传递过来的错误值，如果是正常执行，调用 `recover()` 会返回 `nil`，且没有其它效果

即`panic()` 会导致栈被展开直到 `defer` 修饰的 `recover()` 被调用或程序中止

`defer`-`panic()`-`recover()` 某种意义上也是一种像 `if`，`for` 的控制流机制

### 闭包处理错误

每当函数返回时，我们应该检查是否有错误发生：但是这会导致重复乏味的代码。结合 defer/panic/recover 机制和闭包可以得到一个我们马上要讨论的更加优雅的模式。不过这个模式只有当所有的函数都是同一种签名时可用，这样就有相当大的限制。一个很好的使用它的例子是 web 应用，所有的处理函数都是下面这样：

```go
func handler1(w http.ResponseWriter, r *http.Request) { ... }

```

假设所有的函数都有这样的签名：

```go
func f(a type1, b type2)

```

参数的数量和类型是不相关的。给这个类型一个名字：

```go
fType1 = func f(a type1, b type2)

```

在该模式中使用了两个帮助函数：

1）`check()`：这是用来检查是否有错误和 panic 发生的函数：

```go
func check(err error) { if err != nil { panic(err) } }

```

2）`errorhandler()`：这是一个包装函数。接收一个 `fType1` 类型的函数 `fn` 并返回一个调用 `fn` 的函数。里面就包含有 defer/recover 机制

```go
func errorHandler(fn fType1) fType1 {
	return func(a type1, b type2) {
		defer func() {
			if err, ok := recover().(error); ok {
				log.Printf("run time panic: %v", err)
			}
		}()
		fn(a, b)
	}
}

```

当错误发生时会 recover 并打印在日志中；`check()` 函数会在所有的被调函数中调用，像这样：

```go
func f1(a type1, b type2) {
	...
	f, _, err := // call function/method
	check(err)
	t, err := // call function/method
	check(err)
	_, err2 := // call function/method
	check(err2)
	...
}

```

通过这种机制，所有的错误都会被 recover，并且调用函数后的错误检查代码也被简化为调用 `check(err)` 即可。在这种模式下，不同的错误处理必须对应不同的函数类型；它们（错误处理）可能被隐藏在错误处理包内部。可选的更加通用的方式是用一个空接口类型的切片作为参数和返回值

## 协程与通道

### 带缓冲通道实现一个信号量

信号量是实现互斥锁常见的同步机制，限制对资源的访问，解决读写问题；使用带缓冲的通道可以实现：

- 带缓冲通道的容量和同步资源容量相同
- 通道长度（当前存放元素个数）与当前资源被使用数量相同
- 容量减去通道长度即未处理资源个数（标准信号量的整数值）

创建一个长度可变但容量为0（字节）的通道：

```go
type Empty interface{}
type semaphore chan Empty

```

将可用资源数量`N`初始化信号量`semaphore`：`sem = make(semaphore,N)`

```go
// acquire n resources
func (s semaphore) P(n int) {
	e := new(Empty)
	for i := 0; i < n; i++ {
		s <- e
	}
}

// release n resources
func (s semaphore) V(n int) {
	for i:= 0; i < n; i++{
		<- s
	}
}

//实现互斥的例子
/* mutexes */
func (s semaphore) Lock() {
	s.P(1)
}

func (s semaphore) Unlock(){
	s.V(1)
}

/* signal-wait */
func (s semaphore) Wait(n int) {
	s.P(n)
}

func (s semaphore) Signal() {
	s.V(1)
}

```

### 任务和worker

假设需要处理很多任务；一个 worker 处理一项任务。任务可以被定义为一个结构体：

#### 旧模式：使用共享内存进行同步

由各个任务组成的任务池共享内存；为了同步各个 worker 以及避免资源竞争，需要对任务池进行加锁保护

```go
type Pool struct {
    Mu		sync.Mutex
    Tasks	[]*Task
}

```

```go
func Worker(pool *Pool) {
    for {
        pool.Mu.Lock()
        // begin critical section:
        task := pool.Tasks[0]        // take the first task
        pool.Tasks = pool.Tasks[1:]  // update the pool of tasks
        // end critical section
        pool.Mu.Unlock()
        process(task)
    }
}

```

加锁保证了同一时间只有一个 go 协程可以进入到 `pool` 中：一项任务有且只有一个worker；但若任务量较多时，频繁地加锁/解锁会导致效率降低

#### 新模式：使用通道

使用通道进行同步：使用一个通道接受需要处理的任务，一个通道接受处理完成的任务（及其结果）。worker 在协程中启动，其数量 `N` 应该根据任务数量进行调整；主线程扮演着 Master 节点角色，可能写成如下形式：

```go
func main() {
	pending, done := make(chan *Task), make(chan *Task)
    go sendWork(pending)       // put tasks with work on the channel
    for i := 0; i < N; i++ {   // start N goroutines to do work
    	go Worker(pending, done)
    }
	consumeWork(done)          // continue with the processed tasks
}

func Worker(in,out chan *Task) {
    for {
        t := <-in
        process(t)
        out <- t
    }
}

```

## 网络及网页应用

### tcp服务器

```go
package main

import (
	"fmt"
	"net"
)

func main() {
	fmt.Println("Starting the server ...")
	// 创建 listener
	listener, err := net.Listen("tcp", "localhost:50000")
	if err != nil {
		fmt.Println("Error listening", err.Error())
		return //终止程序
	}
	// 监听并接受来自客户端的连接
	for {
		conn, err := listener.Accept()
		if err != nil {
			fmt.Println("Error accepting", err.Error())
			return // 终止程序
		}
		go doServerStuff(conn)
	}
}

func doServerStuff(conn net.Conn) {
	for {
		buf := make([]byte, 512)
		len, err := conn.Read(buf)
		if err != nil {
			fmt.Println("Error reading", err.Error())
			return //终止程序
		}
		fmt.Printf("Received data: %v", string(buf[:len]))
	}
}

```

用一个无限 `for` 循环的 `listener.Accept()` 来等待客户端的请求。客户端的请求将产生一个 `net.Conn` 类型的连接变量。然后一个独立的协程使用这个连接执行 `doServerStuff()`

```go
package main

import (
	"bufio"
	"fmt"
	"net"
	"os"
	"strings"
)

func main() {
	//打开连接:
	conn, err := net.Dial("tcp", "localhost:50000")
	if err != nil {
		//由于目标计算机积极拒绝而无法创建连接
		fmt.Println("Error dialing", err.Error())
		return // 终止程序
	}

	inputReader := bufio.NewReader(os.Stdin)
	fmt.Println("First, what is your name?")
	clientName, _ := inputReader.ReadString('\n')
	// fmt.Printf("CLIENTNAME %s", clientName)
	trimmedClient := strings.Trim(clientName, "\r\n") // Windows 平台下用 "\r\n"，Linux平台下使用 "\n"
	// 给服务器发送信息直到程序退出：
	for {
		fmt.Println("What to send to the server? Type Q to quit.")
		input, _ := inputReader.ReadString('\n')
		trimmedInput := strings.Trim(input, "\r\n")
		// fmt.Printf("input:--%s--", input)
		// fmt.Printf("trimmedInput:--%s--", trimmedInput)
		if trimmedInput == "Q" {
			return
		}
		_, err = conn.Write([]byte(trimmedClient + " says: " + trimmedInput))
	}
}

```

客户端通过 `net.Dial()` 创建了一个和服务器之间的连接，一旦连接到远程系统，函数就会返回一个`Conn`类型的接口，此时可以用它发送和接收数据

### web服务器

http描述了网页服务器如何与客户端浏览器进行通信

若`req`是来自html表单的POST请求，`"var1"是表单中一个输入域的名称`，则可以通过`req.FormValue("var1")`获取；或者先执行`request.ParseForm()`，然后再获取`request.Form["var1"]`的第一个返回参数

`var1,found := request.Form["var1"]`

若 `var1` 并未出现在表单中，`found` 就是 `false`

表单属性实际上是 `map[string][]string` 类型。网页服务器发送一个 `http.Response` 响应，它是通过 `http.ResponseWriter` 对象输出的，后者组装了 HTTP 服务器响应，通过对其写入内容，就将数据发送给了 HTTP 客户端

如何处理请求即是`http.HandleFunc()`函数完成的

#### 确保网页健壮性

当网页的处理函数发生 panic，服务器会简单地终止运行，这会造成严重的影响；我们需要网页能够承受突发问题

首先想到的是在每个处理函数中使用`defer/recover()`，但是这样会产生较多重复代码，使用闭包的错误处理模式是更优雅的方案

```go
func logPanics(function HandleFunc) HandleFunc {
	return func(writer http.ResponseWriter, request *http.Request) {
		defer func() {
			if x := recover(); x != nil {
				log.Printf("[%v] caught panic: %v", request.RemoteAddr, x)
			}
		}()
		function(writer, request)
	}
}

```

然后用`logPanics()`包装对处理函数的调用，处理函数现在可以恢复panic调用

```go
http.HandleFunc("/test1",logPanics(SimpleServer))
http.HandleFunc("/test2",logPanics(FormServer))

```

```go
package main

import (
	"io"
	"log"
	"net/http"
)

const form = `<html><body><form action="#" method="post" name="bar">
		<input type="text" name="in"/>
		<input type="submit" value="Submit"/>
	</form></html></body>`

type HandleFnc func(http.ResponseWriter, *http.Request)

/* handle a simple get request */
func SimpleServer(w http.ResponseWriter, request *http.Request) {
	io.WriteString(w, "<h1>hello, world</h1>")
}

/* handle a form, both the GET which displays the form
   and the POST which processes it.*/
func FormServer(w http.ResponseWriter, request *http.Request) {
	w.Header().Set("Content-Type", "text/html")
	switch request.Method {
	case "GET":
		/* display the form to the user */
		io.WriteString(w, form)
	case "POST":
		/* handle the form data, note that ParseForm must
		   be called before we can extract form data*/
		//request.ParseForm();
		//io.WriteString(w, request.Form["in"][0])
		io.WriteString(w, request.FormValue("in"))
	}
}

func main() {
	http.HandleFunc("/test1", logPanics(SimpleServer))
	http.HandleFunc("/test2", logPanics(FormServer))
	if err := http.ListenAndServe(":8088", nil); err != nil {
		panic(err)
	}
}

func logPanics(function HandleFnc) HandleFnc {
	return func(writer http.ResponseWriter, request *http.Request) {
		defer func() {
			if x := recover(); x != nil {
				log.Printf("[%v] caught panic: %v", request.RemoteAddr, x)
			}
		}()
		function(writer, request)
	}
}

```

# 常见问题

## 错误和陷阱

### 循环中defer

```go
for _, file := range files {
    if f, err = os.Open(file); err != nil {
        return
    }
    // 这是错误的方式，当循环结束时文件没有关闭
    defer f.Close()
    // 对文件进行操作
    f.Process(data)
}

```

在循环内结尾处的 `defer` 没有执行，所以文件一直没有关闭，不应该使用`defer`

**`defer` 仅在函数返回时才会执行，在循环内的结尾或其他一些有限范围的代码内不会执行**

### 函数参数传递切片

切片实际是一个指向潜在数组的指针，以切片为参数时只需如下：

```go
func findBiggest( listOfNumbers []int ) int {}

```

## 模式

### RPC

在RPC的应用中一般至少有三种角色：首先是服务端实现RPC方法的开发人员，其次是客户端调用PRC方法的人员，最后是指定服务端和客户端RPC接口规范

将RPC服务接口规范分为三个部分：首先是服务的名字，然后是实现的详细方法列表，最后是注册类型服务的函数

```go
const HelloServiceName = "path/to/pkg.HelloService"

type HelloServiceInterface interface {
	Hello(request string, reply *string) error
}

func RegisterHelloService(svc HelloServiceInterface) error {
	return rpc.RegisterName(HelloServiceName, svc)
}

```



## 代码片段

### 字符串

```go
//修改字符串中的一个字符
src := "hello"
c := []byte(str)
c[0] = 'c'
s2 := string(c)		//s2 == "cello"

//获取字符串的子串
substr := str[n:m]

//遍历字符串
// gives only the bytes:
for i:=0; i < len(str); i++ {
… = str[i]
}
// gives the Unicode characters:
for ix, ch := range str {
…
}

//获取一个字符串的字节数
len(str)

//获取一个字符串的字符数
utf8.RuneCountInString(str)	//len([]rune(str))

//如何连接字符串(bytes.Buffer/Strings.Join()/+=)
var buffer bytes.Buffer
for {
    if s,ok := getNextString(); ok {
        buffer.WriteString(s)
    } else {
        break
    }
}
fmt.Print(buffer.String(), "\n")

```

### 数组和切片

```go
//创建（分配内存，未初始化）
arr1 := new([len]type)
slice1 := make([]type,len)
//初始化（赋值）
arr1 := [...]type{i1,i2,i3,i4,i5}
arrKeyValue := [len]type{i1:val1,i2:val2}
var slice1 []type = arr1[start:end]
//appendInt实现
func appendInt(x []int,y int) []int {
    var z []int
    zlen := len(x) + 1
    if zlen <= cap(x) {
        z = x[:zlen]
    } else {
        zcap := zlen
        if zcap < 2*len(x) {
            zcap = 2 * len(x)
        }
        z = make([]int,zlen,zcap)
        copy(z,x)
    }
    z[len(x)] = y
    return z
}

```

### 映射

```go
//创建
map1 := make(map[keytype]valuetype)
//初始化
map1 := map[string]int{"one":1,"two":2}
//检测key1是否存在
val1,isPresent = map1[key1]
//在映射中删除一个键
delete(map1,key1)

```

### 接口

```go
//检测一个值v是否实现了接口Stringer
if v, ok := v.(Stringer); ok {
    fmt.Printf("implements String(): %s\n", v.String())
}

//使用接口实现一个类型分类函数
func classifier(items ...interface{}) {
    for i, x := range items {
        switch x.(type) {
        case bool:
            fmt.Printf("param #%d is a bool\n", i)
        case float64:
            fmt.Printf("param #%d is a float64\n", i)
        case int, int64:
            fmt.Printf("param #%d is an int\n", i)
        case nil:
            fmt.Printf("param #%d is nil\n", i)
        case string:
            fmt.Printf("param #%d is a string\n", i)
        default:
            fmt.Printf("param #%d’s type is unknown\n", i)
        }
    }
}

```

### 函数

```go
//内建函数recover()终止panic()过程
func protect(g func()) {
    defer func() {
    	log.Println("done")
        if x := recover(); x != nil {
        	log.Printf("run time panic: %v", x)
        }
    }()
    log.Println("start")
    g()
}

```

### 文件

```go
//打开一个文件并读取
file,err := os.Open("input.dat")
if err != nil {
	fmt.Printf("An error occurred on opening the inputfile\n" +
      "Does the file exist?\n" +
      "Have you got acces to it?\n")
    return
}
defer file.close()
iReader :+ bufio.NewReader(file)
for {
	str, err := iReader.ReadString('\n')
    if err != nil {
    	return // error or EOF
    }
    fmt.Printf("The input was: %s", str)
}

//通过切片读写文件
func cat(f *file.File) {
	const NBUF = 512
    var buf [NBUF]byte
    for {
        switch nr, er := f.Read(buf[:]); true {
            case nr < 0:
            	fmt.Fprintf(os.Stderr, "cat: error reading from %s: %s\n",f.String(), er.String())
      			os.Exit(1)
            case nr == 0:
            	return
            case nr > 0:
            	if nw, ew := file.Stdout.Write(buf[0:nr]); nw != nr {
        			fmt.Fprintf(os.Stderr, "cat: error writing from %s: %s\n",f.String(), ew.String())
      			}
        }
    }
}

```

### 协程与通道

在协程内部完成的工作量，必须远远高于协程的创建和相互来回通信的开销

- 出于性能考虑建议使用带缓存的通道：

  使用带缓存的通道可以轻易提高它的吞吐量，通过调整通道的容量，甚至可以尝试着更进一步的优化其性能

- 限制一个通道的数据数量并将它们封装成一个数组：

  如果使用通道传递大量单独的数据，那么通道将变成性能瓶颈。然而，将数据块打包封装成数组，在接收端解压数据时性能可以提高

```go
//创建一个带缓存的通道
ch := make(chan type,buf)

//遍历一个通道
for v := range ch {
    // do something with v
}

//检测一个通道是否关闭
for {
    if input,open := <-ch; !open {
        break
    }
    fmt.Printf("%s", input)
}		//或者遍历通道检测

//通过通道让主程序等待，直到协程完成（信号量模式）
ch := make(chan int)
go func(){
    ch <- 1
}()
doSomethingElseForAWhile()
<-ch	//若希望程序一直阻塞，在匿名函数中省略ch <- 1即可

//通道的工厂模版
func pump() chan int {
    ch := make(chan int)
    go func() {
        for i := 0; i++ {
            ch <- i
        }
    }()
    return ch
}

//终止一个协程
runtime.Goexit()

```

## URL映射应用

### 数据结构

变量 `URLStore` 是中心化的内存存储，当收到很多 `Redirect` 服务的请求时，这些请求只涉及读操作：以给定的短 URL 作为键，返回对应的长 URL 的值。然而，对 `Add` 服务的请求则大不相同，它们会更改 `URLStore`，添加新的键值对。当在瞬间收到大量更新请求时，可能会产生如下问题：添加操作可能被另一个同类请求打断，写入的长 URL 值可能会丢失；另外，读取和更改同时进行，导致可能读到脏数据。 `map` 并不保证当开始更新数据时，会彻底阻止另一个更新操作的启动，即`map` 不是线程安全的，要使线程安全最经典的方法是为其增加一个锁

将`URLStore`定义为一个结构体：

```go
import "sync"
type URLStore struct {
	urls map[string]string		// map from short to long URLs
	mu sync.RWMutex
}

```

```go
func (s *URLStore) Get(key string) string {
    s.mu.RLock()
    defer s.mu.RUnlock()
    
    return s.urls[key]
}

func (s *URLStore) Set(key,url string) bool {
    s.mu.Lock()
    defer s.mu.Unlock()
    _,present := s.urls[key]
    if present {
		return false
	}
	s.urls[key] = url
	return true
}

```

#### URLStore工厂函数

`URLStore()`结构体中包含`map`类型的字段，使用前需`make()`初始化；在Go中创建一个结构体实例，一般通过定义一个前缀`New`（其中锁无需指明初始化）

```go
func NewURLStore() *URLStore {
    return &URLStore{ urls: make(map[string]string) }
}

```

# 笔记

## 单例模式

### 常见错误

#### 不考虑并发安全

```go
package singleton

type singleton struct{}

var instance *singleton

func GetInstance() *singleton {
    if instance == nil {
        instance = &singleton{}
    }
    return instance
}

```

在上述情况中，多个goroutine可执行第一个检查，使得创建的`singleton`类型的实例相互覆盖，无法保证返回哪个实例

如果有代码保留了对该实例的引用，可能存在具有不同状态的该类型的多个实例

#### 激进的加锁

对上述问题加锁能够解决并发安全的问题，但是会使得并发调用变成串行

```go
var mu Sync.Mutex

func GetInstance() *singleton {
    mu.Lock()                    // 如果实例存在没有必要加锁
    defer mu.Unlock()

    if instance == nil {
        instance = &singleton{}
    }
    return instance
}

```

#### Check-Lock-Check模式

```c++
if check() {
    lock() {
        if check() {
            // 在这里执行加锁安全的代码
        }
    }
}

```

首先进行检查，因为if语句的开销比加锁小，其次，进行等待并获取互斥锁，此时在同一时刻只有一个执行，但是，在第一次检查和获取互斥锁之间可能有其他goroutine获取了锁，故在锁的内部需再次检查

因此改进后的`GetInstance()`方法如下：

```go
func GetInstance() *singleton {
    if instance == nil {	//不太完美，因为这里不是完全原子的
        mu.Lock()
        defer mu.Unlock()
        
        if instance == nil {
            instance = &singleton{}
        }
    }
    return instance
}

```

通过使用`sync/atomic`包，可以原子化加载并设置一个标志，该标志表明是否已初始化实例

```go
import "sync"
import "sync/atomic"

var initialized uint32

func GetInstance() *singleton {
    if atomic.LoadUint32(&initialized) == 1 {	// 原子操作
        return instance
    }
    mu.Lock()
    defer mu.Unlock()
    if initialized == 0 {
        instance = &singleton{}
        atomic.StoreUnit32(&initialized,1)
    }
    
    return instance
}

```

#### Go惯用的单例模式

在标准库`sync`中有`Once`类型，它能保证某个操作只执行一次

```go
type Once struct {
    done uint32
    m    Mutex
}

func (o *Once) Do(f func()) {
    if atomic.LoadUint32(&o.done) == 0 { // check
        o.doSlow(f)
    }
}

func (o *Once) doSlow(f func()) {
    o.m.Lock()                          // lock
    defer o.m.Unlock()

    if o.done == 0 {                    // check
        defer atomic.StoreUint32(&o.done, 1)
        f()
    }
}

```

因此可以利用`sync.Once`类型同步对`GetInstance()`的访问，并确保类型仅被初始化一次

```go
package singleton

import (
    "sync"
)

type singleton struct {}

var instance *singleton
var once sync.Once

func GetInstance() *singleton {
    once.Do(func(){
        instance = &singleton{}
    })
    return instance
}

```

使用`sync.Once`包是安全地实现此目标的首选方式，类似于Objective-C和Swift（Cocoa）实现`dispatch_once`方法来执行类似的初始化