#1.基础
## 1.1 Go 中的 = 和 := 有什么区别？
= 是赋值
:= 是声明并赋值
一个变量只能声明一次，使用多次 := 是不允许的，而当你声明一次后，却可以赋值多次，没有限制。

## 1.2 Go中的指针的意义是什么
**什么是指针和指针变量**
普通的变量，存储的是数据，而指针变量，存储的是数据的内存地址。

学习指针，主要有两个运算符号，要记牢
&：地址运算符，从变量中取得值的内存地址
// 定义普通变量并打印
age := 18
fmt.Println(age) //output: 18
ptr := &age
fmt.Println(ptr) //output:
*：解引用运算符，从内存地址中取得存储的数据

指针的意义是什么？
意义一：省内存
当你往一个函数传递参数时，若该参数是一个值类型的变量，则在调用函数时，会将原来的变量的值拷贝一遍。
假想每次传参都用数组，那么每次数组都要被复制一遍。如果数组大小有 100万，在64位机器上就需要花费大约 800W 字节，即 8MB 内存。这样会消耗掉大量的内存。
意义二：易编码
写了一个函数来实现更新某对象里的一些数据，在值类型的变量中，若不使用指针，则函数需要重新返回一个更新过的全新对象。
而有了指针，则可以不用返回。

## 1.3 Go 多值返回有什么用？
利用这个特性，在 Go 中实现变量的交换，就不需要再使用中间变量（表象上看是这样，但实际还是会变量的拷贝）了，非常的方便。
在 Go 中没有异常机制，当一个函数运行出错的时候，除了返回该功能函数的结果外，还应该返回一个 error 类型的值。
若该值为 nil 则表示，函数正常运行结束，反之，则函数运行异常。

## 1.4 Go 有异常类型吗？
错误：指的是可能出现问题的地方出现了问题，比如打开一个文件时失败，这种情况在人们的意料之中 ；

异常：指的是不应该出现问题的地方出现了问题，比如引用了空指针，这种情况在人们的意料之外。

在 Go 没有异常类型，只有错误类型（Error）。
一个函数要是想返回错误，通常会使用返回值来表示异常状态，它很像 C语言中的错误码，可逐层返回，直到被处理。
Go 语言中虽然没有异常的概念，但是却有更为恐怖的 panic ，由于有了 recover，在一定程度上， panic 可以类比做异常。
Golang错误和异常（panic）是可以互相转换的：
错误转异常：比如程序逻辑上尝试请求某个URL，最多尝试三次，尝试三次的过程中请求失败是错误，尝试完第三次还不成功的话，失败就被提升为异常了。
异常转错误：比如panic触发的异常被recover恢复后，将返回值中error类型的变量进行赋值，以便上层函数继续走错误处理流程。

## 1.5 Go 中的 rune 和 byte 有什么区别？
一个字符串是由若干个字符组合而成的，比如 hello，就由 5 个字符组成。

在 Go 中字符类型有两种，分别是：

byte 类型：字节，是 uint8 的别名类型

rune 类型：字符，是 int32 的别名类型

byte 和 rune ，虽然都能表示一个字符，但 byte 只能表示 ASCII 码表中的一个字符（ASCII 码表总共有 256 个字符），数量远远不如 rune 多。

rune 表示的是 Unicode字符中的任一字符，而我们都知道，Unicode 是一个可以表示世界范围内的绝大部分字符的编码，这张表里几乎包含了全世界的所有的字符，当然中文也不在话下。
能表示的字符更多，意味着它占用的空间，也要更大，所占空间是 4个 byte 的大小。

## 1.6 Go 语言中的深拷贝和浅拷贝？
什么是拷贝？
当你把 a 变量赋值给 b 变量时，其实就是把 a 变量拷贝给 b 变量

a := "hello"
b := a
这只是拷贝最简单的一种形式，而有些形式却表现得非常的隐蔽。比如：

你往一个函数中传参

你向通道中传入对象

这些其实在 Go编译器中都会进行拷贝的动作。

什么是深浅拷贝？
知道了什么是拷贝，那我们再往深点开挖，聊聊深浅拷贝。

不过先别急，咱先了解下数据结构的两种类型：

值类型 ：String，Array，Int，Struct，Float，Bool

引用类型：Slice，Map

这两种不同的类型在拷贝的时候，在拷贝的时候效果是完全不一样的，这对于很多新手可能是一个坑。

对于值类型来说，你的每一次拷贝，Go 都会新申请一块内存空间，来存储它的值，改变其中一个变量，并不会影响另一个变量。

func main() {
aArr := [3]int{0,1,2}
fmt.Printf("打印 aArr: %v \n", aArr)
bArr := aArr
aArr[0] = 88
fmt.Println("将 aArr 拷贝给 bArr 后，并修改 aArr[0] = 88")
fmt.Printf("打印 aArr: %v \n", aArr)
fmt.Printf("打印 bArr: %v \n", bArr)
}
从输出结果来看，aArr 和 bArr 相互独立，互不干扰

打印 aArr: [0 1 2]
将 aArr 拷贝给 bArr 后，并修改 aArr[0] = 88
打印 aArr: [88 1 2]
打印 bArr: [0 1 2]
对于引用类型来说，你的每一次拷贝，Go 不会申请新的内存空间，而是使用它的指针，两个变量名其实都指向同一块内存空间，改变其中一个变量，会直接影响另一个变量。

func main() {
aslice := []int{0,1,2}
fmt.Printf("打印 aslice: %v \n", aslice)
bslice := aslice
aslice[0] = 88
fmt.Println("将 aslice 拷贝给 bslice 后，并修改 aslice[0] = 88")
fmt.Printf("打印 aslice: %v \n", aslice)
fmt.Printf("打印 bslice: %v \n", bslice)
}
从输出结果来看，aslice 的更新直接反映到了 bslice 的值。

打印 aslice: [0 1 2]
将 aslice 拷贝给 bslice 后，并修改 aslice[0] = 88
打印 aslice: [88 1 2]
打印 bslice: [88 1 2]

```
在Go语言中，复合类型（例如数组、结构体、切片等）的赋值操作和参数传递操作都是浅拷贝（Shallow Copy）。这意味着，当对复合类型进行赋值或传参时，只会复制一份指向底层数据的指针，而不会复制底层数据本身。

具体来说，浅拷贝的行为表现为：

1. 当对一个复合类型进行赋值时，会将指向底层数据的指针复制一份，两个指针指向的是同一个底层数据。

2. 当将一个复合类型作为参数传递给一个函数时，也会将指向底层数据的指针复制一份，但两个指针指向的是同一个底层数据。

这种浅拷贝的行为是由Go语言的内存模型所决定的。在Go语言中，每个变量都有一个地址和一个值，复合类型的变量的值是指向底层数据的指针。因此，复合类型的赋值和参数传递操作只会复制指针，而不会复制底层数据本身。

需要注意的是，由于浅拷贝只复制指针而不复制底层数据，因此对于复合类型中的引用类型（例如切片、映射、通道等），在进行赋值或传参时需要特别注意。这是因为，复制指针会导致多个变量共享同一个底层数据，如果其中一个变量修改了底层数据，会影响到其他变量。因此，在复合类型中包含引用类型时，需要进行深拷贝（Deep Copy）以避免这种问题。

深拷贝可以通过使用复合类型的拷贝函数或者自定义的深拷贝函数来实现。拷贝函数可以复制包含引用类型的复合类型，从而避免多个变量共享同一个底层数据。在Go语言中，所有的内置复合类型都提供了拷贝函数，例如切片的copy函数、映射的make函数等。此外，Go语言中也提供了一些第三方库，例如github.com/mitchellh/copystructure和github.com/jinzhu/copier等，可以用于进行深拷贝操作。自定义的深拷贝函数则可以根据实际需要，递归地遍历复合类型中的每个元素，并对其中的引用类型进行深拷贝。

需要注意的是，进行深拷贝操作可能会导致性能下降，因为需要复制底层数据本身。因此，在进行深拷贝操作时需要谨慎，避免过度使用深拷贝导致性能问题。
```

## 1.7 什么叫字面量和组合字面量？
1. 什么是字面量？
   在 Go 中内置的基本类型有：
布尔类型：bool

11个内置的整数数字类型：int8, uint8, int16, uint16, int32, uint32, int64, uint64, int, uint和uintptr

浮点数类型：float32和float64

复数类型：complex64和complex128

字符串类型：string

而这些基本类型值的文本，就是基本类型字面量。

比如下面这两个字符串，都是字符串字面量，没有用变量名或者常量名来指向这两个字面量，因此也称之为 未命名常量。

"hello, iswbm"

`hello,
iswbm`
2. 同值不同字面量
   值的字面量（literal）是代码中值的文字表示，一个值可能存在多种字面量表示。

举个例子，十进制的数值 15，可以由三种字面量表示

// 16进制
0xF

// 8进制
0o17

// 2进制
0b1111
通过比较，可以看出他们是相等的

import "fmt"

func main() {
fmt.Println(15 == 0xF)     // true
fmt.Println(15 == 017)     // true
fmt.Println(15 == 0b1111)  // true
}
3. 字面量和变量有啥区别？
   下面这是一段很正常的代码

func foo() string {
return "hello"
}

func main() {
bar := foo()
fmt.Println(&bar)
}
可要是换成下面这样

func foo() string {
return "hello"
}

func main() {
fmt.Println(&foo())
}
可实际上这段代码是有问题的，运行后会报错

./demo.go:11:14: cannot take the address of foo()
你一定觉得很奇怪吧？

为什么先用变量名承接一下再取地址就不会报错，而直接使用在函数返回后的值上取地址就不行呢？

这是因为，如果不使用一个变量名承接一下，函数返回的是一个字符串的文本值，也就是字符串字面量，而这种基本类型的字面量是不可寻址的。

要想使用 & 进行寻址，就必须得用变量名承接一下。

4. 什么是组合字面量？
   首先看下Go文档中对组合字面量（Composite Literal）的定义：
Composite literals construct values for structs, arrays, slices, and maps and create a new value each time they are evaluated. They consist of the type of the literal followed by a brace-bound list of elements. Each element may optionally be preceded by a corresponding key。

翻译成中文大致如下： 组合字面量是为结构体、数组、切片和map构造值，并且每次都会创建新值。它们由字面量的类型后紧跟大括号及元素列表。每个元素前面可以选择性的带一个相关key。

什么意思呢？所谓的组合字面量其实就是把对象的定义和初始化放在一起了。

接下来让我们看看结构体、数组、切片和map各自的常规方式和组合字面量方式。

结构体的定义和初始化
让我们看一个struct结构体的常规的定义和初始化是怎么样的。

常规方式
常规方式这样定义是逐一字段赋值，这样就比较繁琐。

type Profile struct {
Name string
Age int
Gender string
}

func main() {
// 声明对象
var xm Profile

    // 属性赋值
    xm.Name = "iswbm"
    xm.Age = 18
    xm.Gender = "male"
}
组合字面量方式

type Profile struct {
Name string
Age int
Gender string
}

func main() {
// 声明 + 属性赋值
xm := Profile{
Name:   "iswbm",
Age:    18,
Gender: "male",
}
}
数组的定义和初始化
常规方式

在下面的代码中，我们在第1行定义了一个8个元素大小的字符串数组。然后一个一个的给元素赋值。即数组变量的定义和初始化是分开的。

var planets [8]string

planets[0] = "Mercury" //水星
planets[1] = "Venus" //金星
planets[2] = "Earth" //地球
组合字面量方式

该示例中，就是将变量balls的定义和初始化合并了在一起。

balls := [4]string{"basketball", "football", "Volleyball", "Tennis"}
slice的定义和初始化
常规方式

// 第一种
var s []string //定义切片变量s，s为默认零值nil
s = append(s, "hat", "shirt") //往s中增加元素，len(s):2,cap(s):2

// 第二种
s := make([]string, 0, 10) //定义s，s的默认值不为零值
组合字面量方式

由上面的常规方式可知，首先都是需要先定义切片，然后再往切片中添加元素。接下来我们看下组合字面量方式。

s := []string{"hat", "shirt"} //定义和初始化一步完成，自动计算切片的容量和长度
// or
var s = []string{"hat", "shirt"}
map的定义和初始化
常规方式

//通过make函数初始化
m := make(map[string]int, 10)
m["english"] = 99
m["math"] = 98
组合字面量方式

m := map[string]int {
"english": 99,
"math": 98,
}

//组合字面量初始化多维map
m2 := map[string]map[int]string {
"english": {
10: "english",
},
}
显然，使用组合字面量会比常规方式简单了不少。

5. 字面量的寻址问题
   字面量，说白了就是未命名的常量，跟常量一样，他是不可寻址的。

这边以数组字面量为例进行说明

func foo() [3]int {
return [3]int{1, 2, 3}
}

func main() {
fmt.Println(&foo())
// cannot take the address of foo()
}

## 1.8 对象选择器自动解引用怎么用？
从一个结构体实例对象中获取字段的值，通常都是使用 . 这个操作符，该操作符叫做 选择器。

选择器有一个妙用，可能大多数人都不清楚。

当你对象是结构体对象的指针时，你想要获取字段属性时，按照常规理解应该这么做

type Profile struct {
Name string
}

func main() {
p1 := &Profile{"iswbm"}
fmt.Println((*p1).Name)  // output: iswbm
}
但还有一个更简洁的做法，可以直接省去 * 取值的操作，选择器 . 会直接解引用，示例如下

type Profile struct {
Name string
}

func main() {
p1 := &Profile{"iswbm"}
fmt.Println(p1.Name)  // output: iswbm
}
也正是这个原因，因此在给你一个方法指定定一个接收者的时候，访问接收者的对象时，不需要像下面这样显示的解引用

type Person struct {
name string
}

func (p *Person) Say() {
fmt.Println((*p).name)
}
而可以直接这样写

type Person struct {
name string
}

func (p *Person) Say() {
fmt.Println(p.name)
}

## 1.9 map 的值不可寻址，那如何修改值的属性？
要回答本题，需要你知道什么是不可寻址。

不急，请先看一下如下这段代码，你知道它有什么问题吗？

package main

type Person struct {
Age int
}

func (p *Person) GrowUp() {
p.Age++
}

func main() {
m := map[string]Person{
"iswbm": Person{Age: 20},
}
m["iswbm"].Age = 23
m["iswbm"].GrowUp()
}
没错，这段代码是错误的，当你编译时，会直接报错呢？

原因在于这两行

m["iswbm"].Age = 23
m["iswbm"].GrowUp()
我们知道 map 的值是不可寻址的，当你使用 m["zhangsan"] 取得值时，其实返回的是其值的拷贝，虽然与原数据值相同，但是在内存中并不是同一个数据。

也正是这样，当 map 的值是一个普通对象（非指针），是无法直接对其修改的。

针对这种错误，解决方法有两种：

第一种：新建变量，修改后再覆盖
func main() {
m := map[string]Person{
"iswbm": Person{Age: 20},
}
p := m["iswbm"]
p.Age = 23
p.GrowUp()
m["iswbm"] = p
}
第二种：使用指针的方式
func main() {
m := map[string]*Person{
"iswbm": &Person{Age: 20},
}
m["iswbm"].Age = 23
m["iswbm"].GrowUp()
}

## 1.10 有类型常量和无类型常量的区别？
在 Go 语言中，常量分为有类型常量和无类型常量。

// 有类型常量
const VERSION string = "v1.0.0"

// 无类型常量
const RELEASE = 3
那么他们有什么区别呢？

当你把有无类型的常量，赋值给一个变量的时候，无类型的常量会被隐式的转化成对应的类型

package main

import "fmt"


func main() {
const RELEASE = 3

    var x int16 = RELEASE
    var y int32 = RELEASE
    fmt.Printf("type: %T \n", x) //type: int16
    fmt.Printf("type: %T \n", y) //type: int32
}
可要是有类型常量，不就会进行转换，在赋值的时候，类型检查就不会通过，从而直接报错

package main

import "fmt"


func main() {
const RELEASE int8 = 3

    var x int16 = RELEASE //cannot use RELEASE (type int8) as type int16 in assignment
    var y int32 = RELEASE //cannot use RELEASE (type int8) as type int32 in assignment
    fmt.Printf("type: %T \n", x)
    fmt.Printf("type: %T \n", y)
}
解决的方法是进行显式的转换

package main

import "fmt"


func main() {
const RELEASE int8 = 3

    var x int16 = int16(RELEASE)
    var y int32 = int32(RELEASE)
    fmt.Printf("type: %T \n", x)  // type: int16
    fmt.Printf("type: %T \n", y)  // type: int32
}

## 1.11 为什么传参使用切片而不使用数组？
Go里面的数组是值类型，切片是引用类型。
值类型的对象在做为实参传给函数时，形参是实参的另外拷贝的一份数据，对形参的修改不会影响函数外实参的值。
把第一个大数组传递给函数会消耗很多内存，采用切片的方式传参可以避免上述问题。切片是引用传递，所以它们不需要使用额外的内存并且比使用数组更有效率。

那么你肯定要问了，数组指针也是引用类型啊，也不一定要用切片吧？

确实，传递数组指针是可以避免对值进行拷贝的内存浪费。

## 1.12 Go 语言中 hot path 有什么用呢？
hot path ，热点路径，顾名思义，是你的程序中那些会频繁执行到的代码。

对于这些代码，由于执行次数非常多，意味着只要有一点设计或编码问题，影响就会被不断放大，而相反，你只要在这些代码中做一些优化，带来的效果也是非常明显的。

hot path 只是一个概念，到底你的程序中有哪些 hot path 还需要根据实际情况分析。

这边举一个比较常见的 hot path 优化的例子，在 sync.Once 有这么一段代码

在注释中首次提到了 hot path 概念

// src/sync/once.go

// Once is an object that will perform exactly one action.
//
// A Once must not be copied after first use.
type Once struct {
// done indicates whether the action has been performed.
// It is first in the struct because it is used in the hot path.
// The hot path is inlined at every call site.
// Placing done first allows more compact instructions on some architectures (amd64/386),
// and fewer instructions (to calculate offset) on other architectures.
done uint32
m    Mutex
}
这是什么意思呢？

当需要访问struct的第一个字段时，我们可以直接对指针解引用来访问第一个字段。

要访问其他字段时，除了结构指针之外， 还需要提供与第一个字段的偏移量

在机器码中，这个偏移量是传递指令的附加值，这会使指令变得更长。对性能的影响是，CPU必须对结构指针添加偏移量以获取想要访问的字段的地址。

因此访问struct的第一个字段的机器码更快，更加紧凑。

这里假设字段在内存中的布局与结构定义中的布局相同，因为编译器可以决定改变内存中结构的字段顺序来优化存储空间，目前go编译器未做这样的优化。

这是一个小优化，在一些对性能优化有极致的要求的人是值得得关注的点。

## 1.13 引用类型与指针，有什么不同？
切片是一个引用类型，将它作为参数传入函数后，你在函数里对数据作变更是会实时反映到实参切片的。
func foo(s []int)  {
s[0] = 666
}

func main() {
slice := []int{1,2}
fmt.Println(slice) // [1 2]
foo(slice)
fmt.Println(slice) // [666 2]
}
此时切片这一引用类型，是不是有点像指针的效果？是的。
但它又和指针不一样，这一点主要体现在：在形参中所作的操作并不一定都会反映在实参上。
还是以切片为例，我在形参上对切片进行扩容，发现形参扩容后，实参并没有发生改变。

func foo(s []int)  {
s = append(s, 666)
}

func main() {
slice := []int{1,2}
fmt.Println(slice) // [1 2]
foo(slice)
fmt.Println(slice) // [1 2]
}
这是为什么呢？

这是因为当你对一个切片 append 的时候，它会做这些事情：

新建一个新的切片 slice2，其实长度与 slice1 一样，但容量是 slice1 的两倍，此时 slice2 底层指向的匿名数组和 slice1 不是同一个。

将 slice1 底层的数组的元素，一个一个的拷贝给 slice2 底层的数组。

并把扩容的元素也拷贝到 slice2中

最后把新的 slice2 返回回来，这就是为什么指针不用返回，而 slice.append 也要返回的原因

从这个流程中，可以看到等号左边的 s （slice2）和 等号右边的 s （slice1）底层引用的数组已经不是同一个了

s = append(s, 666)
因此切片的形参做扩容，并不会影响到实参。

## 1.14 Go 是值传递，还是引用传递、指针传递？
Golang中函数的参数为切片时是传引用还是传值？

对于这个问题，可能会有很多认为是传引用，就比如下面这段代码

func foo(s []int)  {
s[0] = 666
}

func main() {
slice := []int{1,2}
fmt.Println(slice) // [1 2]
foo(slice)
fmt.Println(slice) // [666 2]
}
如果你不了解 Go 中切片的底层结构，你很可能会误信上面的观点。

但其实不是，Go语言中都是值传递，而不是引用传递，也不是指针传递。

Go 中切片的底层结构是这样的

type slice struct {
array unsafe.Pointer
len   int
cap   int
}
而当你将切片作为实参传给函数时，函数是会拷贝一份实参的结构和数据，生成另一个切片，实参切片和形参切片，不仅是长度、容量相等，连指向底层数组的指针都是一样的。

通过分别打印实参切片和形参切片的指针地址，就能验证这一观点

func foo(s []int)  {
fmt.Printf("%p \n", &s) // 0xc00000c080
s = append(s, 666)
}

func main() {
slice := []int{1,2}
fmt.Printf("%p \n", &slice)  // 0xc00000c060
foo(slice)
fmt.Printf("%p \n", &slice)  // 0xc00000c060
}

## 1.15 Go中哪些是可寻址，哪些是不可寻址的？
### 什么叫可寻址？
可直接使用 & 操作符取地址的对象，就是可寻址的（Addressable）。比如下面这个例子

func main() {
name := "iswbm"
fmt.Println(&name)
// output: 0xc000010200
}
程序运行不会报错，说明 name 这个变量是可寻址的。

但不能说 "iswbm" 这个字符串是可寻址的。

"iswbm" 是字符串，字符串都是不可变的，是不可寻址的，后面会介绍到。

在开始逐个介绍之前，先说一下结论

指针可以寻址：&Profile{}

变量可以寻址：name := Profile{}

字面量通通不能寻址：Profile{}

### 哪些是可以寻址的？
变量：&x
func main() {
name := "iswbm"
fmt.Println(&name)
// output: 0xc000010200
}
指针：&*x
type Profile struct {
Name string
}

func main() {
fmt.Println(unsafe.Pointer(&Profile{Name: "iswbm"}))
// output: 0xc000108040
}
数组元素索引: &a[0]
func main() {
s := [...]int{1,2,3}
fmt.Println(&s[0])
// output: xc0000b4010
}
切片
func main() {
fmt.Println([]int{1, 2, 3}[1:])
}
切片元素索引：&s[1]
func main() {
s := make([]int , 2, 2)
fmt.Println(&s[0])
// output: xc0000b4010
}
组合字面量: &struct{X type}{value}
所有的组合字面量都是不可寻址的，就像下面这样子

type Profile struct {
Name string
}

func new() Profile {
return Profile{Name: "iswbm"}
}

func main() {
fmt.Println(&new())
// cannot take the address of new()
}
注意上面写法与这个写法的区别，下面这个写法代表不同意思，其中的 & 并不是取地址的操作，而代表实例化一个结构体的指针。

type Profile struct {
Name string
}

func main() {
fmt.Println(&Profile{Name: "iswbm"}) // ok
}
虽然组合字面量是不可寻址的，但却可以对组合字面量的字段属性进行寻址（直接访问）

type Profile struct {
Name string
}

func new() Profile {
return Profile{Name: "iswbm"}
}

func main() {
fmt.Println(new().Name)
}
### 哪些是不可以寻址的？
常量
import "fmt"

const VERSION  = "1.0"

func main() {
fmt.Println(&VERSION)
}
字符串
func getStr() string {
return "iswbm"
}
func main() {
fmt.Println(&getStr())
// cannot take the address of getStr()
}
函数或方法
func getStr() string {
return "iswbm"
}
func main() {
fmt.Println(&getStr)
// cannot take the address of getStr
}
基本类型字面量
字面量分：基本类型字面量 和 复合型字面量。

基本类型字面量，是一个值的文本表示，都是不应该也是不可以被寻址的。

func getInt() int {
return 1024
}

func main() {
fmt.Println(&getInt())
// cannot take the address of getInt()
}
map 中的元素
字典比较特殊，可以从两个角度来反向推导，假设字典的元素是可寻址的，会出现 什么问题？

如果字典的元素不存在，则返回零值，而零值是不可变对象，如果能寻址问题就大了。

而如果字典的元素存在，考虑到 Go 中 map 实现中元素的地址是变化的，这意味着寻址的结果也是无意义的。

基于这两点，Map 中的元素不可寻址，符合常理。

func main() {
p := map[string]string {
"name": "iswbm",
}

    fmt.Println(&p["name"])
    // cannot take the address of p["name"]
}
搞懂了这点，你应该能够理解下面这段代码为什么会报错啦~

package main

import "fmt"

type Person struct {
Name  string
Email string
}

func main() {
m := map[int]Person{
1:Person{"Andy", "1137291867@qq.com"},
2:Person{"Tiny", "qishuai231@gmail.com"},
3:Person{"Jack", "qs_edu2009@163.com"},
}

    //编译错误：cannot assign to struct field m[1].Name in map
    m[1].Name = "Scrapup"
数组字面量
数组字面量是不可寻址的，当你对数组字面量进行切片操作，其实就是寻找内部元素的地址，下面这段代码是会报错的

func main() {
fmt.Println([3]int{1, 2, 3}[1:])
// invalid operation [3]int literal[1:] (slice of unaddressable value)
}

## 1.16  Slice 和 array 的区别
array是固定长度的数组，使用前必须确定数组长度

slice是一个引用类型，是一个动态的指向数组切片的指针。
slice是一个不定长的，总是指向底层的数组array的数据结构。

数组长度不能改变，初始化后长度就是固定的；切片的长度是不固定的，可以追加元素，在追加时可能使切片的容量增大。
结构不同，数组是一串固定数据，切片描述的是截取数组的一部分数据，从概念上说是一个结构体。
初始化方式不同，如上。另外在声明时的时候：声明数组时，方括号内写明了数组的长度或使用...自动计算长度，而声明slice时，方括号内没有任何字符。
unsafe.sizeof的取值不同，unsafe.sizeof(slice)返回的大小是切片的描述符，不管slice里的元素有多少，返回的数据都是24字节。unsafe.sizeof(arr)的值是在随着arr的元素的个数的增加而增加，是数组所存储的数据内存的大小。
函数调用时的传递方式不同，数组按值传递，slice按引用传递。

## 1.17 向为 nil 的 channel 发送数据会怎么样
panic

向为 nil 的 channel 发送数据会导致程序 panic。
在 Go 语言中，向一个 channel 发送数据时，如果 channel 是 nil，则会在运行时引发 panic。这是因为 channel 是一种基于通信的并发原语，它需要先被创建才能使用，如果 channel 没有被创建或者已经关闭，向其发送数据都会导致 panic。
因此，在使用 channel 时，需要先对其进行初始化，可以使用 make 函数来创建 channel，例如 ch := make(chan int)。此外，在向 channel 发送数据之前，还需要确保 channel 的接收方已经准备好接收数据，否则发送方可能会被阻塞，直到接收方准备好接收数据或者 channel 被关闭。

## 1.18 go 协程调度原理
Go 语言的协程调度器使用了 M:N 的调度模型，即将 M 个用户级线程（也称为 goroutine）映射到 N 个操作系统线程上，通过协程调度器来实现协程的调度和管理。

Go 语言的协程调度器主要有以下几个组件：

M：代表用户级线程，也称为 goroutine，它们是 Go 语言并发模型的基本单位，每个 M 都有自己的协程栈和程序计数器，可以执行 Go 语言中的任意函数或方法。

P：代表处理器，每个 P 维护了一个本地的 goroutine 队列，用于存放等待执行的 goroutine。当一个 goroutine 执行时，会被分配到一个 P 上执行，当它执行完成或者阻塞时，会被放回到本地队列中等待下次调度。

G：代表 goroutine，它是一个轻量级的协程，每个 G 包含了一个协程栈、一个指向函数或方法的指针、以及一些额外的元数据。当一个 goroutine 被创建时，会被放入一个全局队列中等待分配 P 执行。当一个 goroutine 阻塞时，会将 P 释放出来，以便其他 goroutine 可以继续执行。

Scheduler：协程调度器，它负责将 goroutine 分配到 P 上执行，并监控每个 P 的运行状态，当 P 阻塞或者空闲时，会将其中的 goroutine 重新分配到其他 P 上执行，以实现协程的负载均衡和高效利用。

Go 语言的协程调度器采用了抢占式调度策略，即在某个 goroutine 执行过程中，调度器可以随时打断它并将 CPU 时间分配给其他 goroutine 执行。调度器会根据一定的策略来决定哪个 goroutine 可以获得 CPU 时间，例如根据优先级、时间片轮转等算法。此外，调度器还会通过协程的阻塞和唤醒来调整协程的状态，以避免协程的阻塞对整个程序的性能产生过大的影响。

# 2.进阶
## 2.1 slice 扩容过程
当一个 Go slice 的容量不足以容纳更多的元素时，它需要进行扩容。扩容的过程如下：
确定新的容量
当需要扩容时，Go 会创建一个新的底层数组，该数组的大小通常是原始容量的两倍。如果原始容量小于1024，则新容量将增加至原始容量的两倍；如果原始容量大于等于1024，则新容量将增加原始容量的 25%。
分配新的底层数组
Go 会为新的底层数组分配一段新的内存，并将原始数组中的元素复制到新的数组中。
更新 slice 的指针、长度和容量
一旦新的底层数组分配完成并且原始数组中的元素复制完成，Go 就会更新 slice 的指针、长度和容量。slice 的指针将指向新的底层数组，长度将保持不变，容量将更新为新的容量。
释放旧的底层数组
最后，Go 会释放原始数组所占用的内存，这样就完成了 slice 的扩容过程。

## 2.2 goroutine 存在的意义是什么？ （协程与线程区别）
goroutine 非常的轻量，初始分配只有 2KB，当栈空间不够用时，会自动扩容。同时，自身存储了执行 stack 信息，用于在调度时能恢复上下文信息。
而线程比较重，一般初始大小有几 MB(不同系统分配不同)，线程是由操作系统调度，是操作系统的调度基本单位。而 golang 实现了自己的调度机制，
goroutine 是它的调度基本单位。

线程其实分两种：

一种是传统意义的操作系统线程

一种是编程语言实现的用户态线程，也称为协程，在 Go 中就是 goroutine

因此，goroutine 的存在必然是为了换个方式解决操作系统线程的一些弊端 – 太重 。

太重表现在如下几个方面：

第一：创建和切换太重

操作系统线程的创建和切换都需要进入内核，而进入内核所消耗的性能代价比较高，开销较大；

第二：内存使用太重

一方面，为了尽量避免极端情况下操作系统线程栈的溢出，内核在创建操作系统线程时默认会为其分配一个较大的栈内存（虚拟地址空间，内核并不会一开始就分配这么多的物理内存），然而在绝大多数情况下，系统线程远远用不了这么多内存，这导致了浪费；

另一方面，栈内存空间一旦创建和初始化完成之后其大小就不能再有变化，这决定了在某些特殊场景下系统线程栈还是有溢出的风险。

相对的，用户态的goroutine则轻量得多：

goroutine是用户态线程，其创建和切换都在用户代码中完成而无需进入操作系统内核，所以其开销要远远小于系统线程的创建和切换；

goroutine启动时默认栈大小只有2k，这在多数情况下已经够用了，即使不够用，goroutine的栈也会自动扩大，同时，如果栈太大了过于浪费它还能自动收缩，这样既没有栈溢出的风险，也不会造成栈内存空间的大量浪费。

## 2.3 说说 Go 中闭包的底层原理？
一个函数内引用了外部的局部变量，这种现象，就称之为闭包。

package main

import "fmt"

func adder() func(int) int {
sum := 0
return func(x int) int {
sum += x
return sum
}
}



而这个闭包中引用的外部局部变量并不会随着外部函数的返回而被从栈上销毁。
我们尝试着调用这个函数，发现每一次调用，sum 的值都会保留在 闭包函数中以待使用。
func main() {
valueFunc:= adder()
fmt.Println(valueFunc(2))     // output: 2
fmt.Println(valueFunc(2))   // output: 4
}

闭包函数里引用的外部变量，是在堆还是栈内存申请的，取决于，你这个闭包函数在函数 Return 后是否还会在其他地方使用，若会， 就会在堆上申请，若不会，就在栈上申请。
闭包函数里，引用的外部变量，存储的并不是对值的拷贝，存的是值的指针。
函数的返回值里若写了变量名，则该变量是在上级的栈内存里申请的，return 的值，会直接赋值给该变量。

## 2.4 defer 的变量快照什么情况会失效？
其中有一个知识是 defer 的变量快照，举个简单的例子来说

在下面这段代码中，会先打印出来 18，即使后面 age 已经被改变了，可 defer 中的 age还是 修改之前的 0，这种现象称之为变量快照。

func func1() {
age := 0
defer fmt.Println(age) // output: 0

    age = 18
    fmt.Println(age)      // output: 18
}


func main() {
func1()
}
对于这个输出结果，相信还是挺容易理解的。

接下来，我请大家再看下面这个例子，可以猜猜看会输出什么？

func func1() {
age := 0
defer func() {
fmt.Println(age)
}()
age = 18
return
}

func main() {
func1()
}
正确的答案是：18， 而不是 0

你肯定会纳闷：不对啊，defer 不是会对变量的值做一个快照吗？答案应该是 0 啊，为什么会是 18？

实际上，仔细观察，可以发现上面的两个例子的区别就在于，一个 defer 后接的是单个表达式，另一个 defer 后接的是一个函数，并且不是普通函数，而是一个匿名的闭包函数。

根据闭包的特性，实际上在闭包函数存的是 age 这个变量的指针（原因可以查看上一篇文章：Go 语言面试题 100 讲之 014篇：说说 Go 中闭包的底层原理？），因而，在 defer 后所修改的值会直接影响到 defer 中的 age 的值。

总结一下：

若 defer 后接的是单行表达式，那defer 中的 age 只是拷贝了 func1 函数栈中 defer 之前的 age 的值；

若 defer 后接的是闭包函数，那defer 中的 age 只是存储的是 func1 函数栈中 age 的指针。

## 2.5 说说你对 Go 里的抢占式调度的理解
>在Go语言中，采用了抢占式调度（Preemptive Scheduling）的方式来实现协程（Goroutine）之间的并发执行。抢占式调度是一种操作系统调度方式，可以在执行任务的过程中中断当前任务，切换到另一个任务上执行，从而实现并发执行的效果。
具体来说，在Go语言中，抢占式调度的实现机制如下：
Go语言的运行时系统（Runtime）会将多个协程放置在一个线程中执行。
运行时系统会周期性地检查协程的运行状态，例如是否阻塞、是否需要等待I/O等。
当发现某个协程阻塞或者等待I/O时，运行时系统会将该协程从线程中移除，并将线程中的其他协程继续执行。
在某个协程执行时间过长或者出现死循环等情况时，运行时系统会中断该协程，并将线程中的其他协程继续执行。


Go 从 v1.1 发现展到目前的 v1.16，协程调度策略也在不断的完善优化。

下面我将从 v.1.1 开始讲讲 协程调度策略中抢占式调度的发展历程。

### v1.1 的非抢占式调用
在最初的 v1.1 版本中，只有当一个协程主动让出 CPU 资源（可以是运行结束，也可以是发生了系统调用或其他阻塞性操作），才能触发调度，进行下一个协程。

而如果一个协程运行了很久，也没有主动让出的动作发生，就会自私的一个人占用整个线程，该线程无法再去运行其他的 goroutine 了。

这种策略会让 Go 的并发性大打折扣，名不符实。

### v1.2 基于协作的抢占式调用
由于 v1.1 的非抢占式调用，以程序的并发效率影响实在太大。因为在下一个版本 v1.2 就紧急地对调度策略进行了临时的优化。经过优化后，go 从 v1.2 开始支持抢占式的调用：

如果 sysmon 监控线程发现有个协程 A 执行之间太长了（或者 gc 场景，或者 stw 场景），那么会友好的在这个 A 协程的某个字段设置一个抢占标记 ；

协程 A 在 call 一个函数的时候，会复用到扩容栈（morestack）的部分逻辑，检查到抢占标记之后，让出 cpu，切到调度主协程里；

之所以说 v1.2 的抢占式调用是临时的优化方案，是因为这种抢占式调度是基于协作的。在一些的边缘场景下，协程还是在会独自占用整个线程无法让出。

从上面的流程中，你应该可以注意到，A 调度权被抢占有个前提：A 必须主动 call 函数，这样才能有走到 morestack 的机会。

反面案例可以看下面这个程序，当运行到 time.Sleep 后，线程上的 goroutine 会从 main 切换到前面的匿名函数协程，而这个匿名函数协程并是在作for 死循环，并没有任何可以让出 cpu 运行权的操作，因为该程序在 go 1.14 之前的 go版本中，运行后会一直卡住，而不会打印 I got scheduled!

package main

import (
"fmt"
"runtime"
"time"
)

func main() {
runtime.GOMAXPROCS(1)

    fmt.Println("The program starts ...")

    go func() {
        for {
        }
    }()

    time.Sleep(time.Second)
    fmt.Println("I got scheduled!")
}
### v1.14 基于信号的抢占式调用
基于协作的抢占式调用，伴随着 Go 走过了12个版本，终于在 v1.14 迎来了真正的抢占式调用。

为什么说是真正的抢占式调用呢？

因为 v1.14 的这种抢占式调用是基于信号的，不管你的协程有没有意愿主动让出 cpu 运行权，只要你这个协程超过某个时间，就会发送信号强行夺取 cpu 运行权。

那么这个时间具体是多少呢？ 20ms

## 2.6 简述一下 Go 栈空间的扩容/缩容过程？
### 扩容流程
由于当前的 Go 的栈结构使用的是连续栈，并且初始值才 2k 比较小，因此随着函数的调用层级加深，Go 的初始栈空间就可能不够用，不够用的话，就会触发栈空间的扩容。
编译器会为函数调用插入运行时检查runtime.morestack，它会在几乎所有的函数调用之前检查当前goroutine 的栈内存是否充足，如果当前栈需要扩容，会调用runtime.newstack 创建新的栈。
而新的栈空间，是旧栈空间大小（通过保存在goroutine中的stack信息里记录的栈区内存边界计算出来的）的两倍，但最大栈空间大小不能超过 maxstacksize ，也就是 1G。

### 缩容流程
在函数返回后，对应的栈空间会回收，如果调用栈比较深，那么随着函数一个一个返回，回收的栈空间会越来越多。假设在调用栈最深的时候，整体的栈空间扩容到了 100M，那么随着函数的返回，到某一个函数的时候，100M 的栈空间只有 1M 是实际占用的，内存利用率只有区区的 1% ，实在太浪费了。
因此在垃圾回收的时候，有必要检查一下栈空间里内存利用率，当利用率低于 25% 时，就要开始进行缩容，缩容成原来的栈空间的 50%，但同时也不能小于栈空间的原始值即最小值，2KB。

相同点
不管是扩容还是缩容，都是使用 runtime.copystack 函数来开辟新的栈空间，然后将旧栈的数据全部拷贝至新的栈空间，并调整原来指针的指向。

## 2.7 说一下 GMP 模型的原理
### 1. 什么是 GMP ？
GMP 模型是 golang 自己的一个调度模型，它抽象出了下面三个结构：

G： 也就是协程 goroutine，由 Go runtime 管理。我们可以认为它是用户级别的线程。
P： processor 处理器。每当有 goroutine 要创建时，会被添加到 P 上的 goroutine 本地队列上，如果 P 的本地队列已满，则会维护到全局队列里。
M： 系统线程。在 M 上有调度函数，它是真正的调度执行者，M 需要跟 P 绑定，并且会让 P 按下面的原则挑出个 goroutine 来执行：
优先从 P 的本地队列获取 goroutine 来执行；如果本地队列没有，从全局队列获取，如果全局队列也没有，会从其他的 P 上偷取 goroutine。



### 2. GMP 核心
两个队列
在整个 Go 调度器的生命周期中，存在着两个非常重要的队列：
全局队列（Global Queue）：全局只有一个
本地队列（Local Queue）：每个 P 都会维护一个本地队列

当你执行 go func() 创建一个 goroutine 时，会优选将该协程放入到当前 P 的本地队列中等待被 P 选中执行。
但若当前 P 的本地队列任务太多了，已经存放不下了，那么这个 goroutine 就只能放入到全局队列中。

两种调度
一个协程得以运行，需要同时满足以下两个条件：
P 已经和某个线程进行绑定，这样才能参与操作系统的调度获得 CPU 时间
P 已经从队列中（可以是本地队列，也可以是全局队列，甚至是从其他 P 的队列）取到该协程
第一个条件就是 操作系统调度，而第二个其实就是 Go 里的调度器。

操作系统调度
假设一台机器上有两个 CPU 核心，意味着，同时在同一时间里，只能有两个线程运行着。
可如果该机器上实际开启了 4 个线程，要是先执行一个线程完再执行另一个线程，那么当
某一个线程因为一些阻塞性的系统调用而阻塞时，CPU 的时间就会因此而白白浪费掉了。
更合适的做法是，使用 操作系统调度策略，设定一个调度周期，假设是 10ms （毫秒），
那在一个周期里，每个线程都平均分，都只能得到 2.5ms 的CPU 运行时间。
可如果机器上有 1000 个线程呢？难道每个线程都分个 0.01 ms （也就是 10 微秒）吗？
要知道，CPU 从 A 线程切换到 B 线程，是有巨大的时间浪费在线程上下文的切换，如果切换
得太频繁，就会有大量的 CPU 时间白白浪费。


因此，通常会限制最小的时间片的长度，假设为 2ms，受此调整，现在调度周期就会变成 2*1000 = 2s 。

Go调度器
在 Go 中需要用到调度的，无非是如下几种：
1.将 P 绑定到一个合适的 M
P 本身不能直接运行 G，只将 P 跟 M 绑定后，才能执行 G。
假设 P1 当前正绑定在 M1 上运行 G1，此时 G1 内部发生了一次系统调度后，P1 就会与 M1 进行解绑，
然后再从空闲的线程队列中再寻找一个来绑定，假设绑定的是 M2，可如果没有空闲的线程呢？那没办法，只能创建一个新的线程再进行绑定。
绑定后，就会再从本地的队列中寻找 G 来执行（如果没找到，就会去其他队列找，上面已经讲过，不再赘述）。
过了一段时间后，之前 M1 上 G1 发生的系统调用结束后，M1 会去找原先自己的搭档 P1（它自己会记录），如果自己的老搭档也刚好空闲着，就可以再次合作进行绑定，接着运行 G1 未完成的工作。
可不幸的是，P1 已经找到了新的合作伙伴 M2，暂时没空搭理 M1 。
M1 联系不上 P1，只能去寻找有没有其他空闲的 P ，如果所有的 P 都被绑定着，说明现在任务非常繁重，想完成任务只能排队慢慢等。
于是，M1 上的 G1 就会被标记为 Runable ，放到全局队列中，而 M1 自身也会因为没有 P 可以绑定而进入休眠状态，如果长时间休眠等待 则会 GC 回收销毁

2.为 P 选中一个 G 来执行
P 就像是一个流水线工人，而 P 的本地队列就是流水线，G 是流水线上的零件。而 Go 调度器就是流水线组长，负责监督工人的是不是有在努力的工作。

完成一个 G 后，P 就得立马接着从队列中拿到下一个 G，继续干活。

遇到手脚麻利的 P ，干完了自己的活，本想着可以偷懒一会，没想到却被组长发现了，立马就从全局队列中拿了些新的 G 交到 P 的手里。

天真的 P 以为只要把 全局队列中的 G 的也干完了，就肯定 能休息了吧？

当 P 又快手快脚的把全局队列中的 G 也都干完的时候，P 非常得意，心想：终于可以休息会了。

没想到又被眼尖的组长察觉到了：不错啊，小 P，手脚挺麻利的。看看其他人，还那么多活没干完。真是拖后腿。可谁让咱是一个团队的呢，要有集体荣誉感，你能者多劳。

说完，就把其他人的 G 放到了我的工作上。。。

### 3. 调度器的设计策略
- 复用线程，避免频繁的创建、销毁线程，而是对线程的复用。
1）work stealing 机制
当本线程无可运行的 G 时，尝试从其他线程绑定的 P 偷取 G，而不是销毁线程。

2）hand off 机制
当本线程因为 G 进行系统调用阻塞时，线程释放绑定的 P，把 P 转移给其他空闲的线程执行。


- 利用并行
GOMAXPROCS 设置 P 的数量，最多有 GOMAXPROCS 个线程分布在多个 CPU 上同时运行。
GOMAXPROCS 也限制了并发的程度，比如 GOMAXPROCS = 核数/2，则最多利用了一半的 CPU 核进行并行。

- 抢占调度
在 Go 中，一个 goroutine 最多占用 CPU 10ms，防止其他 goroutine 被饿死，这是协作式抢占调度。
而在 go 1.14+ ，Go 开始支持基于信号的真抢占调度了。

- 全局 G 队列
在新的调度器中依然有全局 G 队列，但功能已经被弱化了，当 M 执行 work stealing 从其他 P 偷不到 G 时，
它可以从全局 G 队列获取 G。

## 2.8 GMP 模型为什么要有 P ？
GM 模型是怎样的？
在 Go v1.1 之前，实际上 GMP确实是没有 P 的，所有的 M 线程都要从 全局队列中获取 G 来执行任务，为了避免冲突，从全局队列中获取 G 的时候，要先获取一把大锁。
当一个程序的并发量比较小的时候，影响还不大，而当程序的并发量非常大的时候，这个全局队列会成为性能的瓶颈。
除此之外 ，若直接把 G 从全局队列分配给 M，那么当 G 中当生系统调用或者其他阻塞性的操作时，M 会有一段时间处于挂起的状态，此时又没有新创建线程的线程来代替该线程继续从队列中取出其他 G 来运行，从效率上其实会打折扣。

P 带来的改变
加了 P 之后会带来什么改变呢？

每个 P 有自己的本地队列，大幅度的减轻了对全局队列的直接依赖，所带来的效果就是锁竞争的减少。而 GM 模型的性能开销大头就是锁竞争。

当一个 M 中 运行的 G 发生阻塞性操作时，P 会重新选择一个 M，若没有 M 就新创建一个 M 来继续从 P 本地队列中取 G 来执行，提高运行效率。

每个 P 相对的平衡上，在 GMP 模型中也实现了 Work Stealing 算法，如果 P 的本地队列为空，则会从全局队列或其他 P 的本地队列中窃取可运行的 G 来运行，减少空转，提高了资源利用率。

## 2.9 不分配内存的指针类型能用吗？
不能！
原因是我们只声明了指针类型，但并没有为其分配内存，没有内存地址，你赋给它的值应该存在哪里呢？自然只能报错了。

## 2.10 Go 中的 GC 演变是怎样的？
### 在 Go v1.3 之前采用的是 标记-清除(mark and sweep)算法。
它的逻辑是，先将整个程序挂起（STW, stop the world），然后遍历程序中的对象，只要是可达的对象，都会被标记保留（红色），而那些不可达的对象（白色），则会被清理掉，清理完成后，会恢复程序。然后不断重复该过程。
这种标记-清除的算法，会有一段 STW 的时间，将整个程序暂停，这对于一些实时性要求比较高的系统是无法接受的。

另外，上面这个标记的过程扫描的是整个堆内存，耗时比较久，最重要的是，它在清除数据的时候，会产生堆内存的碎片。

因此从 Go v1.5 开始，就开始抛弃这种算法，而改用三色并发标记法。

### 三色并发标记法
新算法的出现，必然是要解决旧算法存在的最关键问题 ，即STW 的暂停挂起导致的程序卡顿。

它的逻辑就是，准备三种颜色，分别对三种对象进行标记：
黑色：检测到有被引用，并且已经遍历完它所有直接引用的对象或者属性
白色：还没检测到有引用的对象（检测开始前，所有对象都是白色，检测结束后，没有被引用的对象都是白色，会被清查掉）
灰色：检测到有被引用，但是他的属性还没有被遍历完，等遍历完后也会变成黑色

当垃圾回收开始时，Go 会把根对象标记为灰色，其他对象标记为白色，然后从根对象遍历搜索，按照上面的定义去不断的对灰色对象进行扫描标记。当没有灰色对象时，表示所有对象已扫描过，然后就可以开始清除白色对象了。

既然 STW 会挂起程序，那是不是可以考虑将其摘除呢？
摘除会带来一个问题就是在标记的时候，程序的运行会不断改变对象的引用路径，影响标记的准确性。
总结来说，就是当在标记的时候出现 ：一个白色对象被黑色对象引用，同时该白色对象又被某个灰色（或者上级有灰色对象）对象取消引用的情况，就会标记不准确。
因此如果想摘除 STW，那就得规避掉上面这个场景出现。
解决方法是：使用 插入屏障 和 删除屏障
- 插入屏障
在A对象引用B对象的时候，B对象被标记为灰色。(将B挂在A下游，B必须被标记为灰色)
- 删除屏障
被删除的对象，如果自身为灰色或者白色，那么被标记为灰色。

### 2.11 Go 中哪些动作会触发 runtime 调度？
goroutine 在遇到哪些情况会触发 runtime 的调度器去调度呢？

第一种：系统调用 SysCall

当你在 goroutine 进行一些 sleep 休眠、读取磁盘或者发送网络请求时，其实都会发生系统调用，进入操作系统内核。

而一旦发生系统调用，就会直接触发 runtime 的调度，当前的 P 就会去找其他的 M 进行绑定，并取出 G 开始运行。

第二种：等待锁、通道

此外，在你的代码中，若因为锁或者通道导致代码阻塞了，也会触发调度。

第三种：人工触发
在代码中直接调用 runtime.Gosched 方法，也可以手动触发。

# 3.进阶
## 3.1 局部变量分配在栈上还是堆上？
在Go语言中，局部变量的分配位置既可能是栈（Stack）上，也可能是堆（Heap）上，具体取决于变量的生命周期和使用方式。
### 什么是堆内存和栈内存？
根据内存管理（分配和回收）方式的不同，可以将内存分为 堆内存 和 栈内存。

那么他们有什么区别呢？

堆内存：由内存分配器和垃圾收集器负责回收

栈内存：由编译器自动进行分配和释放

一个程序运行过程中，也许会有多个栈内存，但肯定只会有一个堆内存。

每个栈内存都是由线程或者协程独立占有，因此从栈中分配内存不需要加锁，并且栈内存在函数结束后会自动回收，性能相对堆内存好要高。

而堆内存呢？由于多个线程或者协程都有可能同时从堆中申请内存，因此在堆中申请内存需要加锁，避免造成冲突，并且堆内存在函数结束后，需要 GC （垃圾回收）的介入参与，如果有大量的 GC 操作，将会吏程序性能下降得历害。

### 局部变量是从哪里分配的？
在函数里声明定义的变量，我们称之为局部变量。

一般来说，局部变量的作用域仅在该函数中，当函数返回后，所有局部变量所占用的内存空间都将被收回，对于这类变量，都是从栈上分配内存空间。

可有一种局部变量，比较特殊。

这种局部变量，虽然在函数里声明定义，但是在函数外还会持续的使用。

对于这类局部变量，显然我们是不希望函数退出后将其销毁的。

那怎么办呢？可以从堆区分配内存空间给这类局部变量。

不过这个事实其实不用程序员操心，Go 的编译器会自行判断做优化的。但我们仍然需要知道这个知识点（因为面试会问哈哈）

## 3.2 Go 的默认栈大小是多少？最大值多少？
Go 语言使用用户态线程 Goroutine 作为执行上下文，它的额外开销和默认栈大小都比线程小很多，然而 Goroutine 的栈内存空间和栈结构也在早期几个版本中发生过一些变化：

v1.0 ~ v1.1 — 最小栈内存空间为 4KB；

v1.2 — 将最小栈内存提升到了 8KB；

v1.3 — 使用连续栈替换之前版本的分段栈；

v1.4 — 将最小栈内存降低到了 2KB；

Goroutine 的初始栈内存在最初的几个版本中多次修改，从 4KB 提升到 8KB 是临时的解决方案，其目的是为了减轻分段栈的栈热分裂问题对程序造成的性能影响；

在 v1.3 版本引入连续栈之后，Goroutine 的初始栈大小降低到了 2KB，进一步减少了 Goroutine 占用的内存空间。

这个栈比 x86_64 构架下线程的默认栈 2M 要小很多，真的是轻量级的用户态线程。

关于这个初始值和最大值嘛，在 Go 的源码 runtime/stack.go 里其实都可以找到

// rumtime.stack.go
// The minimum size of stack used by Go code
_StackMin = 2048

var maxstacksize uintptr = 1 << 20 // enough until runtime.main sets it for real
那么这个 1<<20 代表多大呢？使用 Python 计算一下是 1G

## 3.3  go的内存分配与回收是怎么样的
Go 的内存分配借鉴了 Google 的 TCMalloc 分配算法，其核心思想是内存池 + 多级对象管理。内存池主要是预先分配内存，减少向系统申请的频率；多级对象有：mheap、mspan、arenas、mcentral、mcache。它们以 mspan 作为基本分配单位。

### 3.3.1 基础概念
go在程序启动时会分配一块虚拟内存地址是连续的内存，这一块内存分为了3个区域, 在X64上大小分别是512M, 16G和512G。
- arena
arena区域就是我们通常说的heap, go从heap分配的内存都在这个区域中.

- bitmap
bitmap区域用于表示arena区域中哪些地址保存了对象, 并且对象中哪些地址包含了指针.

>GC在标记时需要知道哪些地方包含了指针, bitmap区域涵盖了arena区域中的指针信息. 除此之外, GC还需要知道栈空间上哪些地方包含了指针, 因为栈空间不属于arena区域, 栈空间的指针信息将会在函数信息里面. 另外, GC在分配对象时也需要根据对象的类型设置bitmap区域, 来源的指针信息将会在类型信息里面.
总结起来go中有以下的GC Bitmap:
bitmap区域: 涵盖了arena区域, 使用2 bit表示一个指针大小的内存
函数信息: 涵盖了函数的栈空间, 使用1 bit表示一个指针大小的内存 (位于stackmap.bytedata)
类型信息: 在分配对象时会复制到bitmap区域, 使用1 bit表示一个指针大小的内存 (位于_type.gcdata)

  (bitmap区域中一个byte(8 bit)对应了arena区域中的四个指针大小的内存, 也就是2 bit对应一个指针大小的内存. 所以bitmap区域的大小是 512GB / 指针大小(8 byte) / 4 = 16GB.)

- spans
span是用于分配对象的区块,通常一个span包含了多个大小相同的元素, 一个元素会保存一个对象, 除非:
span用于保存大对象, 这种情况span只有一个元素
span用于保存极小对象且不包含指针的对象(tiny object), 这种情况span会用一个元素保存多个对象

span中有一个freeindex用来标记下一次分配对象时应该开始搜索的地址, 分配后freeindex会增加, 在freeindex之前的元素都是已分配的, 在freeindex之后的元素有可能已分配, 也有可能未分配.

span每次GC以后都可能会回收掉一些元素, allocBits用于标记哪些元素是已分配的, 哪些元素是未分配的. 使用freeindex + allocBits可以在分配时跳过已分配的元素, 把对象设置在未分配的元素中, 但因为每次都去访问allocBits效率会比较慢, span中有一个整数型的allocCache用于缓存freeindex开始的bitmap, 缓存的bit值与原值相反.

gcmarkBits用于在gc时标记哪些对象存活, 每次gc以后gcmarkBits会变为allocBits. 需要注意的是span结构本身的内存是从系统分配的, 上面提到的spans区域和bitmap区域都只是一个索引.


（spans区域用于表示arena区中的某一页(Page)属于哪个span, . spans区域中一个指针(8 byte)对应了arena区域中的一页(在go中一页=8KB). 所以spans的大小是 512GB / 页大小(8KB) * 指针大小(8 byte) = 512MB.)


### 3.3.2 什么时候从Heap分配对象
go会自动确定哪些对象应该放在栈上, 哪些对象应该放在堆上. 简单的来说, 当一个对象的内容可能在生成该对象的函数结束后被访问, 那么这个对象就会分配在堆上. 在堆上分配对象的情况包括:

- 返回对象的指针
- 传递了对象的指针到其他函数
- 在闭包中使用了对象并且需要修改对象
- 使用new
这个过程其实也叫逃逸分析。


当要分配大于 32K 的对象时，从 mheap 分配。
当要分配的对象小于等于 32K 大于 16B 时，从 P 上的 mcache 分配，如果 mcache 没有内存，则从 mcentral 获取，如果 mcentral 也没有，则向 mheap 申请，如果 mheap 也没有，则从操作系统申请内存。
当要分配的对象小于等于 16B 时，从 mcache 上的微型分配器上分配。

### 3.3.3 span 的分配
在goroutine 的调度模型中（也就是GMP模型），P是一个虚拟的资源, 同一时间只能有一个线程访问同一个P, 所以P中的数据不需要锁. 为了分配对象时有更好的性能, 各个P中都有span的缓存(也叫mcache)
在分配对象时将会从以下的位置获取适合的span用于分配:

首先从P的缓存(mcache)获取, 如果有缓存的span并且未满则使用, 这个步骤不需要锁
然后从全局缓存(mcentral)获取, 如果获取成功则设置到P, 这个步骤需要锁
最后从mheap获取, 获取后设置到全局缓存, 这个步骤需要锁
在P中缓存span的做法跟CoreCLR中线程缓存分配上下文(Allocation Context)的做法相似, 都可以让分配对象时大部分时候不需要线程锁, 改进分配的性能.

### 3.3.4 分配对象的流程
首先会检查GC是否在工作中, 如果GC在工作中并且当前的G分配了一定大小的内存则需要协助GC做一定的工作, 这个机制叫GC Assist, 用于防止分配内存太快导致GC回收跟不上的情况发生.

之后会判断是小对象还是大对象, 如果是大对象则直接调用largeAlloc从堆中分配, 如果是小对象分3个阶段获取可用的span, 然后从span中分配对象:
- 首先从P的缓存(mcache)获取
- 然后从全局缓存(mcentral)获取, 全局缓存中有可用的span的列表
- 最后从mheap获取, mheap中也有span的自由列表, 如果都获取失败则从arena区域分配



### 3.3.5 回收对象的流程
在GC过程中会有两种后台任务(G), 一种是标记用的后台任务, 一种是清扫用的后台任务. 标记用的后台任务会在需要时启动, 可以同时工作的后台任务数量大约是P的数量的25%, 也就是go所讲的让25%的cpu用在GC上的根据. 清扫用的后台任务在程序启动时会启动一个, 进入清扫阶段时唤醒.

目前整个GC流程会进行两次STW(Stop The World), 第一次是Mark阶段的开始, 第二次是Mark Termination阶段. 第一次STW会准备根对象的扫描, 启动写屏障(Write Barrier)和辅助GC(mutator assist). 第二次STW会重新扫描部分根对象, 禁用写屏障(Write Barrier)和辅助GC(mutator assist). 需要注意的是, 不是所有根对象的扫描都需要STW, 例如扫描栈上的对象只需要停止拥有该栈的G. 从go 1.9开始, 写屏障的实现使用了Hybrid Write Barrier, 大幅减少了第二次STW的时间.

### 3.3.6 GC 的触发条件
GC在满足一定条件后会被触发, 触发条件有以下几种:

- gcTriggerAlways: 强制触发GC
- gcTriggerHeap: 当前分配的内存达到一定值就触发GC
- gcTriggerTime: 当一定时间没有执行过GC就触发GC
- gcTriggerCycle: 要求启动新一轮的GC, 已启动则跳过, 手动触发GC的runtime.GC()会使用这个条件

### 3.3.7 算法
- 在 Go v1.3 之前采用的是 标记-清除(mark and sweep)算法。
它的逻辑是，先将整个程序挂起（STW, stop the world），然后遍历程序中的对象，只要是可达的对象，都会被标记保留（红色），而那些不可达的对象（白色），则会被清理掉，清理完成后，会恢复程序。然后不断重复该过程。 这种标记-清除的算法，会有一段 STW 的时间，将整个程序暂停，这对于一些实时性要求比较高的系统是无法接受的。
另外，上面这个标记的过程扫描的是整个堆内存，耗时比较久，最重要的是，它在清除数据的时候，会产生堆内存的碎片。
因此从 Go v1.5 开始，就开始抛弃这种算法，而改用三色并发标记法。
- 三色并发标记法
它的逻辑就是，准备三种颜色，分别对三种对象进行标记： 黑色：检测到有被引用，并且已经遍历完它所有直接引用的对象或者属性 白色：还没检测到有引用的对象（检测开始前，所有对象都是白色，检测结束后，没有被引用的对象都是白色，会被清查掉） 灰色：检测到有被引用，但是他的属性还没有被遍历完，等遍历完后也会变成黑色

当垃圾回收开始时，Go 会把根对象标记为灰色，其他对象标记为白色，然后从根对象遍历搜索，按照上面的定义去不断的对灰色对象进行扫描标记。当没有灰色对象时，表示所有对象已扫描过，然后就可以开始清除白色对象了。


因为go支持并行GC, GC的扫描和go代码可以同时运行, 这样带来的问题是GC扫描的过程中go代码有可能改变了对象的依赖树, 例如开始扫描时发现根对象A和B, B拥有C的指针, GC先扫描A, 然后B把C的指针交给A, GC再扫描B, 这时C就不会被扫描到. 为了避免这个问题, go在GC的标记阶段会启用写屏障(Write Barrier).

启用了写屏障(Write Barrier)后, 当B把C的指针交给A时, GC会认为在这一轮的扫描中C的指针是存活的, 即使A可能会在稍后丢掉C, 那么C就在下一轮回收. 写屏障只针对指针启用, 而且只在GC的标记阶段启用, 平时会直接把值写入到目标地址.

go在1.9开始启用了混合写屏障(Hybrid Write Barrier)，混合写屏障会同时标记指针写入目标的”原指针”和“新指针”.
标记原指针的原因是, 其他运行中的线程有可能会同时把这个指针的值复制到寄存器或者栈上的本地变量, 因为复制指针到寄存器或者栈上的本地变量不会经过写屏障, 所以有可能会导致指针不被标记

混合写屏障可以让GC在并行标记结束后不需要重新扫描各个G的堆栈, 可以减少Mark Termination中的STW时间. 除了写屏障外, 在GC的过程中所有新分配的对象都会立刻变为黑色.

### 3.3.8 根对象有哪些
在GC的标记阶段首先需要标记的就是”根对象”, 从根对象开始可到达的所有对象都会被认为是存活的. 根对象包含了全局变量, 各个G的栈上的变量等, GC会先扫描根对象然后再扫描根对象可到达的所有对象. 扫描根对象包含了一系列的工作, 源码定义在[https://github.com/golang/go/blob/go1.9.2/src/runtime/mgcmark.go#L54]函数:

- Fixed Roots: 特殊的扫描工作
- fixedRootFinalizers: 扫描析构器队列
- fixedRootFreeGStacks: 释放已中止的G的栈
- Flush Cache Roots: 释放mcache中的所有span, 要求STW
- Data Roots: 扫描可读写的全局变量
- BSS Roots: 扫描只读的全局变量
- Span Roots: 扫描各个span中特殊对象(析构器列表)
- Stack Roots: 扫描各个G的栈
标记阶段(Mark)会做其中的”Fixed Roots”, “Data Roots”, “BSS Roots”, “Span Roots”, “Stack Roots”. 完成标记阶段(Mark Termination)会做其中的”Fixed Roots”, “Flush Cache Roots”.



## 3.4 对已经关闭的 channel 进行读写，会怎么样？
当 channel 被关闭后，如果继续往里面写数据，程序会直接 panic 退出。如果是读取关闭后的 channel，不会产生 pannic，还可以读到数据。但关闭后的 channel 没有数据可读取时，将得到零值，即对应类型的默认值。
为了能知道当前 channel 是否被关闭，可以使用下面的写法来判断。
`	if v, ok := <-ch; !ok {
		fmt.Println("channel 已关闭，读取不到数据")
	}`
还可以使用下面的写法不断的获取 channel 里的数据：
`	for data := range ch {
		// get data dosomething
	}`
这种用法会在读取完 channel 里的数据后就结束 for 循环，执行后面的代码。

## 3.5 map 为什么是不安全的？
map 在扩缩容时，需要进行数据迁移，迁移的过程并没有采用锁机制防止并发操作，
而是会对某个标识位标记为 1，表示此时正在迁移数据。如果有其他 goroutine 
对 map 也进行写操作，当它检测到标识位为 1 时，将会直接 panic。
如果我们想要并发安全的 map，则需要使用 sync.map。

## 3.6 map 的 key 为什么得是可比较类型的？
map 的 key、value 是存在 buckets 数组里的，每个 bucket 又可以容纳 8 个 key 和 8 个 value。当要插入一个新的 key - value 时，会对 key 进行 hash 运算得到一个 hash 值，然后根据 hash 值 的低几位(取几位取决于桶的数量，比如一开始桶的数量是 5，则取低 5 位)来决定命中哪个 bucket。

在命中某个 bucket 后，又会根据 hash 值的高 8 位来决定是 8 个 key 里的哪个位置。如果不巧，发生了 hash 冲突，即该位置上已经有其他 key 存在了，则会去其他空位置寻找插入。如果全都满了，则使用 overflow 指针指向一个新的 bucket，重复刚刚的寻找步骤。

从上面的流程可以看出，在判断 hash 冲突，即该位置是否已有其他 key 时，肯定是要进行比较的，所以 key 必须得是可比较类型的。像 slice、map、function 就不能作为 key。

## 3.7 mutex 的正常模式、饥饿模式、自旋？
正常模式
当 mutex 调用 Unlock() 方法释放锁资源时，如果发现有正在阻塞并等待唤起的 Goroutine 队列时，则会将队头的 Goroutine 唤起。队头的 goroutine 被唤起后，会采用 CAS 这种乐观锁的方式去修改占有标识位，如果修改成功，则表示占有锁资源成功了，当前占有成功的 goroutine 就可以继续往下执行了。

饥饿模式
由于上面的 Goroutine 唤起后并不是直接的占用资源，而是使用 CAS 方法去尝试性占有锁资源。如果此时有新来的 Goroutine，那么它也会调用 CAS 方法去尝试性的占有资源。对于 Go 的并发调度机制来讲，会比较偏向于 CPU 占有时间较短的 Goroutine 先运行，即新来的 Goroutine 比较容易占有资源，而队头的 Goroutine 一直占用不到，导致饿死。

针对这种情况，Go 采用了饥饿模式。即通过判断队头 Goroutine 在超过一定时间后还是得不到资源时，会在 Unlock 释放锁资源时，直接将锁资源交给队头 Goroutine，并且将当前状态改为饥饿模式。

后面如果有新来的 Goroutine 发现是饥饿模式时， 则会直接添加到等待队列的队尾。

自旋
如果 Goroutine 占用锁资源的时间比较短，那么每次释放资源后，都调用信号量来唤起正在阻塞等候的 goroutine，将会很浪费资源。

因此在符合一定条件后，mutex 会让等候的 Goroutine 去空转 CPU，在空转完后再次调用 CAS 方法去尝试性的占有锁资源，直到不满足自旋条件，则最终才加入到等待队列里。

## 3.8 Go 的逃逸行为是指？
在传统的编程语言里，会根据程序员指定的方式来决定变量内存分配是在栈还是堆上，比如声明的变量是值类型，则会分配到栈上，或者 new 一个对象则会分配到堆上。

在 Go 里变量的内存分配方式则是由编译器来决定的。如果变量在作用域（比如函数范围）之外，还会被引用的话，那么称之为发生了逃逸行为，此时将会把对象放到堆上，即使声明为值类型；如果没有发生逃逸行为的话，则会被分配到栈上，即使 new 了一个对象。

## 3.9 context 是如何一层一层通知子 context
当 ctx, cancel := context.WithCancel(父Context)时，会将当前的 ctx 挂到父 context 下，然后开个 goroutine 协程去监控父 context 的 channel 事件，
一旦有 channel 通知，则自身也会触发自己的 channel 去通知它的子 context， 关键代码如下
`go func() {
select {
case <-parent.Done():
child.cancel(false, parent.Err())
case <-child.Done():
}
}()`

## 3.10 waitgroup 原理
waitgroup 内部维护了一个计数器，当调用 wg.Add(1) 方法时，就会增加对应的数量；
当调用 wg.Done() 时，计数器就会减一。直到计数器的数量减到 0 时，就会调用
runtime_Semrelease 唤起之前因为 wg.Wait() 而阻塞住的 goroutine。

## 3.11 sync.Once 原理
内部维护了一个标识位，当它 == 0 时表示还没执行过函数，此时会加锁修改标识位，
然后执行对应函数。后续再执行时发现标识位 != 0，则不会再执行后续动作了。关键代码如下：
`type Once struct {
done uint32
m    Mutex
}

func (o *Once) Do(f func()) {
// 原子加载标识值，判断是否已被执行过
if atomic.LoadUint32(&o.done) == 0 {
o.doSlow(f)
}
}

func (o *Once) doSlow(f func()) { // 还没执行过函数
o.m.Lock()
defer o.m.Unlock()
if o.done == 0 { // 再次判断下是否已被执行过函数
defer atomic.StoreUint32(&o.done, 1) // 原子操作：修改标识值
f() // 执行函数
}
}`

## 3.12 定时器原理
一开始，timer 会被分配到一个全局的 timersBucket 时间桶。每当有 timer 被创建出来时，就会被分配到对应的时间桶里了。
为了不让所有的 timer 都集中到一个时间桶里，Go 会创建 64 个这样的时间桶，然后根据 当前 timer 所在的 Goroutine 的
P 的 id 去哈希到某个桶上：
`// assignBucket 将创建好的 timer 关联到某个桶上
func (t *timer) assignBucket() *timersBucket {
id := uint8(getg().m.p.ptr().id) % timersLen
t.tb = &timers[id].timersBucket
return t.tb
}`
接着 timersBucket 时间桶将会对这些 timer 进行一个最小堆的维护，
每次会挑选出时间最快要达到的 timer。如果挑选出来的 timer 时间还没到，
那就会进行 sleep 休眠；如果 timer 的时间到了，则执行 timer 上的函数，
并且往 timer 的 channel 字段发送数据，以此来通知 timer 所在的 goroutine。

## 3.13 gorouinte 泄漏有哪些场景
gorouinte 里有关于 channel 的操作，如果没有正确处理 channel 的读取，会导致 channel 一直阻塞住, goroutine 不能正常结束

## 3.14 defer、panic、recover 三者的用法
defer 函数调用的顺序是后进先出，当产生 panic 的时候，
会先执行 panic 前面的 defer 函数后才真的抛出异常。一般的，
recover 会在 defer 函数里执行并捕获异常，防止程序崩溃。

## 3.15 了解有栈协程，无栈协程吗
有栈协程和无栈协程是协程实现中的两种不同的方式。
有栈协程是指在创建协程时为其分配一定大小的栈空间，
协程执行时使用这段栈空间来存储局部变量、参数和返回
地址等信息。当协程执行完毕时，栈空间被释放。有栈协程
通常需要在创建时指定栈的大小，因此在协程数量较多或者
协程执行过程中栈空间的大小动态变化时，可能会出现一些问题，
例如占用过多的内存或者栈空间不足等。

无栈协程是指协程执行时不使用固定大小的栈空间，而是使用
一个固定大小的栈来管理协程的执行。无栈协程通常使用栈分
裂技术来动态分配和释放栈空间，避免了有栈协程可能出现的
栈空间大小限制和内存占用问题。无栈协程的实现通常需要对
程序代码进行特殊的编译器支持，例如 Go 语言中的 goroutine 
就是一种无栈协程实现。

## 3.16 


# 4. 源码、底层原理
## 4.1 Context
### 4.1.1 作用
上下文 context.Context Go 语言中用来设置截止日期、同步信号，传递请求相关值的结构体。上下文与 
Goroutine 有比较密切的关系，是 Go 语言中独特的设计， 它最初是被设计用来处理网络请求的。在Go语
言中，每个协程都有自己的goroutine，每个goroutine可以执行一个函数或者方法，而且goroutine之间
可以通过 channel进行通信。在进行网络编程时，通常需要创建一个或多个goroutine来处理客户端的请求。
为了方便管理和取消这些goroutine，Go语言提供了context包，用于在goroutine之间传递请求的上下文
信息，并在需要的时候取消请求。

### 4.1.2 接口方法
context.Context 是 Go 语言在 1.7 版本中引入标准库的接口1，该接口定义了四个需要实现的方法，其中包括：

- Deadline — 返回 context.Context 被取消的时间，也就是完成工作的截止日期；
- Done — 返回一个 Channel，这个 Channel 会在当前工作完成或者上下文被取消后关闭，多次调用 Done 
方法会返回同一个 Channel；
- Err — 返回 context.Context 结束的原因，它只会在 Done 方法对应的 Channel 关闭时返回非空的值；
如果 context.Context 被取消，会返回 Canceled 错误；
如果 context.Context 超时，会返回 DeadlineExceeded 错误；
- Value — 从 context.Context 中获取键对应的值，对于同一个上下文来说，多次调用 Value 并传入相同的 
Key 会返回相同的结果，该方法可以用来传递请求特定的数据；
```type Context interface {
Deadline() (deadline time.Time, ok bool)
Done() <-chan struct{}
Err() error
Value(key interface{}) interface{}
}
```
Go context 包中提供的 context.Background、context.TODO、context.WithDeadline 和 
context.WithValue 函数会返回实现该接口的私有结构体

### 4.1.3 原理
Go 服务的每一个请求都是通过单独的 Goroutine 处理的，HTTP/RPC 请求的处理器会启动新的 Goroutine 
访问数据库和其他服务。 我们可能会创建多个 Goroutine 来处理一次请求，而 context.Context 的作用是
在不同 Goroutine 之间同步请求特定数据、取消信号以及处理请求的截止日期。
（**在 Goroutine 构成的树形结构中对信号进行同步以减少计算资源的浪费是 context.Context 的最大作用。**）

每一个 context.Context 都会从最顶层的 Goroutine 一层一层传递到最下层。context.Context 可以在上层 
Goroutine 执行出现错误时，将信号及时同步给下层。

原理：多个 Goroutine 同时订阅 ctx.Done() 管道中的消息，一旦接收到取消信号就立刻停止当前正在执行的工作。

## 4.2 Channel
### 4.2.1 作用
主要作为Goroutine 之间的一种通信方式，是支撑 Go 语言高性能并发编程模型的重要结构
channel 内部维护了两个 goroutine 队列，一个是待发送数据的 goroutine 队列，
另一个是待读取数据的 goroutine 队列。 每当对 channel 的读写操作超过了可缓冲的 
goroutine 数量，那么当前的 goroutine 就会被挂到对应的队列上，直到有其他 goroutine 
执行了与之相反的读写操作，将它重新唤起。

### 4.2.2 设计原理
在很多主流的编程语言中，多个线程传递数据的方式一般都是共享内存，为了解决线程竞争，我们需要限制同一时间能够
读写这些变量的线程数量，然而这与 Go 语言鼓励的设计并不相同，它更鼓励：不要通过共享内存的方式进行通信，而是
应该通过通信的方式共享内存。

虽然我们在 Go 语言中也能使用共享内存加互斥锁进行通信，但是 Go 语言提供了一种不同的并发模型，即通信顺序进程
（Communicating sequential processes，CSP， 通过消息传递的方式进行通信和协调）。Goroutine 和 
Channel 分别对应 CSP 中的实体和传递信息的媒介，Goroutine 之间会通过 Channel 传递数据。

- 先入先出
目前的 Channel 收发操作均遵循了先进先出的设计，具体规则如下：
先从 Channel 读取数据的 Goroutine 会先接收到数据；
先向 Channel 发送数据的 Goroutine 会得到先发送数据的权利；

- 实现（有锁）
  Go语言中的channel分为有缓冲和无缓冲两种类型，它们的实现原理有所不同。

无缓冲的channel实现原理比较简单，它使用一个阻塞队列来实现并发访问的安全性和效率。
当一个goroutine向无缓冲的channel发送数据时，如果没有接收者在等待接收数据，发送
者会被阻塞，直到有接收者准备好接收数据。当一个goroutine从无缓冲的channel接收数
据时，如果没有发送者在等待发送数据，接收者会被阻塞，直到有发送者准备好发送数据。无
缓冲的channel保证了发送和接收操作的同步性，可以用于实现同步的通信和协程之间的同步
等待。

有缓冲的channel实现原理比较复杂，它使用一个固定大小的缓冲区来存储发送的数据，同时
使用一个阻塞队列来管理等待发送或接收数据的goroutine。当一个goroutine向有缓冲的
channel发送数据时，如果缓冲区没有满，数据会被复制到缓冲区中，并立即返回，发送者可
以继续执行。如果缓冲区已满，则发送者会被阻塞，直到有接收者从缓冲区中取走数据。当一
个goroutine从有缓冲的channel接收数据时，如果缓冲区不为空，数据会被从缓冲区中取出，
并立即返回，接收者可以继续执行。如果缓冲区为空，则接收者会被阻塞，直到有发送者向缓
冲区中发送数据。

有缓冲的channel实现原理使用了互斥锁和条件变量来保证并发访问的安全性和效率。
当一个goroutine访问有缓冲的channel时，会先获取一个互斥锁，然后进行操作，
最后释放互斥锁。同时，当一个goroutine被阻塞时，会将其加入一个等待队列中，
并使用条件变量等待信号通知。当有缓冲的channel中有数据可读或可写时，会通过
条件变量发送信号通知等待的goroutine，从而唤醒它们继续执行。

- 无锁提议
Channel 在运行时的内部表示是 runtime.hchan，该结构体中包含了用于保护成员变量的互斥锁，
从某种程度上说，Channel 是一个用于同步和通信的有锁队列，使用互斥锁解决程序中可能存在的线
程竞争问题是很常见的，我们能很容易地实现有锁队列。 然而锁导致的休眠和唤醒会带来额外的上下文
切换，如果临界区6过大，加锁解锁导致的额外开销就会成为性能瓶颈。1994 年的论文 Implementing
lock-free queues 就研究了如何使用无锁的数据结构实现先进先出队列，而 Go 语言社区也在 
2014 年提出了无锁 Channel 的实现方案，该方案将 Channel 分成了以下三种类型8：
同步 Channel — 不需要缓冲区，发送方会直接将数据交给（Handoff）接收方；
异步 Channel — 基于环形缓存的传统生产者消费者模型；
chan struct{} 类型的异步 Channel — struct{} 类型不占用内存空间，不需要实现缓冲区和直接发送（Handoff）的语义；
这个提案的目的也不是实现完全无锁的队列，只是在一些关键路径上通过无锁提升 Channel 的性能。社区中已经有无锁 Channel 
的实现，但是在实际的基准测试中，无锁队列在多核测试中的表现还需要进一步的改进。
因为目前通过 CAS 实现11的无锁 Channel 没有提供先进先出的特性，所以该提案暂时也被搁浅了。

### 4.2.3 底层数据结构
Go 语言的 Channel 在运行时使用 runtime.hchan 结构体表示。
实际上创建的都是如下所示的结构：

`type hchan struct {
qcount   uint
dataqsiz uint
buf      unsafe.Pointer
elemsize uint16
closed   uint32
elemtype *_type
sendx    uint
recvx    uint
recvq    waitq
sendq    waitq

lock mutex
}`

runtime.hchan 结构体中的五个字段 qcount、dataqsiz、buf、sendx、recv 构建底层的循环队列：
qcount — Channel 中的元素个数；
dataqsiz — Channel 中的循环队列的长度；
buf — Channel 的缓冲区数据指针；
sendx — Channel 的发送操作处理到的位置；
recvx — Channel 的接收操作处理到的位置；

除此之外，elemsize 和 elemtype 分别表示当前 Channel 能够收发的元素类型和大小；sendq 和 recvq 
存储了当前 Channel 由于缓冲区空间不足而阻塞的 Goroutine 列表，这些等待队列使用双向链表
runtime.waitq 表示，链表中所有的元素都是 runtime.sudog 结构：
`type waitq struct {
first *sudog
last  *sudog
}`

runtime.sudog 表示一个在等待列表中的 Goroutine，该结构中存储了两个分别指向前后 runtime.sudog 的指针以构成链表。

### 4.2.4 创建管道
Go 语言中所有 Channel 的创建都会使用 make 关键字。
代码编译时会进行代码检查，并进行中间代码的转换，
这一阶段会对传入 make 关键字的缓冲区大小进行检查，如果我们不向 make 传递表示缓冲区大小的参数，那么就会设置一个默认值 0，
也就是当前的 Channel 不存在缓冲区。
然后调用runtime.makechan 和 runtime.makechan64 会根据传入的参数类型和缓冲区大小创建一个新的 Channel 结构，其中后者用于处
理缓冲区大小大于 2 的 32 次方的情况，因为这在 Channel 中并不常见，所以我们重点关注 runtime.makechan：

makechan 根据 Channel 中收发元素的类型和缓冲区的大小初始化 runtime.hchan 和缓冲区：
如果当前 Channel 中不存在缓冲区，那么就只会为 runtime.hchan 分配一段内存空间；
如果当前 Channel 中存储的类型不是指针类型，会为当前的 Channel 和底层的数组分配一块连续的内存空间；
在默认情况下会单独为 runtime.hchan 和缓冲区分配内存；
在函数的最后会统一更新 runtime.hchan 的 elemsize、elemtype 和 dataqsiz 几个字段。

### 4.2.5 发送数据
当我们想要向 Channel 发送数据时，就需要使用 ch <- i 语句，也会涉及编译器的一个代码转换，最终会调用
runtime.chansend 并传入 Channel 和需要发送的数据。该函数包含了发送数据的全部逻辑，如果我们
在调用时将 block 参数设置成 true，那么表示当前发送操作是阻塞的：
在发送数据的逻辑执行之前会先为当前 Channel 加锁，防止多个线程并发修改数据。如果 Channel 已经关闭，那么向该 Channel 发送数据时会报 “send on closed channel” 错误并中止程序。

runtime.chansend 函数的实现比较复杂，该函数的执行过程主要分成三个部分：
当存在等待的接收者时，通过 runtime.send 直接将数据发送给阻塞的接收者；
当缓冲区存在空余空间时，将发送的数据写入 Channel 的缓冲区；
当不存在缓冲区或者缓冲区已满时，等待其他 Goroutine 从 Channel 接收数据；

### 4.2.6 接受数据
Go 语言中可以使用两种不同的方式去接收 Channel 中的数据：
i <- ch
i, ok <- ch
也会涉及编译器的一个代码转换，两个方式只是会被转换成两个不同的函数，但是最终都会调用
runtime.chanrecv 函数：
当我们从一个空 Channel 接收数据时会直接调用 runtime.gopark 让出处理器的使用权。
如果当前 Channel 已经被关闭并且缓冲区中不存在任何数据，那么会清除 ep 指针中的数据并立刻返回。

除了上述两种特殊情况，使用 runtime.chanrecv 从 Channel 接收数据时还包含以下三种不同情况：
当存在等待的发送者时，通过 runtime.recv 从阻塞的发送者或者缓冲区中获取数据；
当缓冲区存在数据时，从 Channel 的缓冲区中接收数据；
当缓冲区中不存在数据时，等待其他 Goroutine 向 Channel 发送数据；

### 4.2.7 关闭管道
编译器会将用于关闭管道的 close 关键字转换成 OCLOSE 节点以及 runtime.closechan 函数。
当 Channel 是一个空指针或者已经被关闭时，Go 语言运行时都会直接崩溃并抛出异常：
处理完了这些异常的情况之后就可以开始执行关闭 Channel 的逻辑了，主要就是将 recvq 和 
sendq 两个队列中的数据加入到 Goroutine 列表 gList 中，与此同时该函数会清除所有 
runtime.sudog 上未被处理的元素。在最后会为所有被阻塞的 Goroutine 调用 
runtime.goready 触发调度。


## 4.3 调度器
谈到 Go 语言调度器，我们绕不开的是操作系统、进程与线程这些概念，线程是操作系统调度时的最基本单元，而 Linux 在调度器并不区分进程和线程的调度，它们在不同操作系统上也有不同的实现，但是在大多数的实现中线程都属于进程

### 4.3.1 进程、线程、协程
进程是操作系统中的一个概念，它是一个正在执行中的程序的实例。每个进程都有自己的地址空间、内存、文件句柄和其他系统资源。进程之间是相互独立的，它们不能直接访问彼此的内存空间，必须通过进程间通信（IPC）机制来进行通信。
线程是进程中的一个执行单元，它与其他线程共享进程的地址空间和系统资源。线程之间可以直接访问彼此的内存空间，因此线程之间的通信比进程之间的通信更加高效。线程的创建和销毁比进程更加轻量级，因此在需要并发执行的场景中，使用线程比使用进程更加高效。
协程是一种用户态的轻量级线程，它是由程序员自己控制的。协程可以看作是一种特殊的函数，它可以执行过程中暂停、保存当前状态，然后在需要的时候恢复执行。协程之间的切换比线程之间的切换更加轻量级，因此在需要高并发、高吞吐量的场景中，使用协程比使用线程更加高效。


多个线程可以属于同一个进程并共享内存空间。因为多线程不需要创建新的虚拟内存空间，
所以它们也不需要内存管理单元处理上下文的切换，线程之间的通信也正是基于共享的内存
进行的，与重量级的进程相比，线程显得比较轻量。 虽然线程比较轻量，但是在调度时也
有比较大的额外开销。每个线程会都占用 1M 以上的内存空间，在切换线程时不止会消耗
较多的内存，恢复寄存器中的内容还需要向操作系统申请或者销毁资源，每一次线程上下文
的切换都需要消耗 ~1us 左右的时间1，但是 Go 调度器对 Goroutine 的上下文切换
约为 ~0.2us，减少了 80% 的额外开销。

Go 语言的调度器通过使用与 CPU 数量相等的线程减少线程频繁切换的内存开销，同时在
每一个线程上执行额外开销更低的 Goroutine 来降低操作系统和硬件的负载。

### 4.3.2 设计原理
TODO 
