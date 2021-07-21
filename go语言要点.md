# 零、入门

1. ++i是非法的，i++是语句，不能使用a=i++的表达
2. :=声明变量只能在函数内部，对于包的变量必须使用var的形式
3. printf的打印格式

```go
%d          十进制整数
%x, %o, %b  十六进制，八进制，二进制整数。
%f, %g, %e  浮点数： 3.141593 3.141592653589793 3.141593e+00
%t          布尔：true或false
%c          字符（rune） (Unicode码点)
%s          字符串
%p					打印地址
%q          带双引号的字符串"abc"或带单引号的字符'c'
%v          变量的自然形式（natural format），也就是go语言的形式输出
%+v 				先输出字段类型，再输出该字段的值
%#v 				先输出结构体名字值，再输出结构体（字段类型+字段的值）
%T          变量的类型
%%          字面上的百分号标志（无操作数）
```

![内存模型](./picture/go语言要点/内存模型.png)



# 一、程序结构

## 1. 命名

函数内部定义的变量只在函数内部有效

函数外部定义的变量在整个包中都有效，变量名的开头字母大小写决定了变量在包外的可见性：对于首字母大写的变量名，它将是**导出的**，即可以被其他包访问到（函数名同理），而对于首字母小写的变量，只能在包内被访问



## 2. 声明

四种类型的生命 var、const、type、func



## 3. 变量



左值与右值

等号左边的变量，代表变量所在的内存空间

等号右边的变量，代表变量内存空间存储的数据(值)

```go
var a,b int
a = 1   //a左值，代表的是a所在的内存空间
b = a   //a右值，代表的是a所在内存空间所存储的数据：1
```



**先计算右值，再赋值给变量：**

```go
func main() {
    x, y := 1, 2
    x, y = y+3, x+2     // 先计算出右值 y+3、x+2，然后再对 x、y 变量赋值。

    println(x, y)   //5 3
}
```



### 3.1 变量声明

- var 变量名 类型 = 表达式

在函数内部还可以使用简短的形式声明。   变量名:=表达式

零值初始化机制可以确保每个声明的变量总是有一个良好定义的值，因此在Go语言中不存在未初始化的变量

```go
var i, j, k int                 // int, int, int
var b, f, s = true, 2.3, "four" // bool, float64, string
```

- 包级别声明的变量在main函数执行前完成初始化（所以声明顺序无关紧要）
- 函数级别的变量（局部变量）在语句被执行到的时候完成初始化（必须先声明后使用）

### 3.2 指针变量

通过指针我们可以直接读取或者修改对应变量的值，而不需要知道变量的名字

如果用“var x int”声明语句声明一个x变量，那么&x表达式（取x变量的内存地址）将产生一个指向该整数变量的指针，指针对应的数据类型是`*int`，指针被称之为“指向int类型的指针”。如果指针名字为p，那么可以说“p指针指向变量x”，或者说“p指针保存了x变量的内存地址”。同时`*p`表达式对应p指针指向的变量的值。一般`*p`表达式读取指针指向的变量的值，这里为int类型的值，同时因为`*p`对应一个变量，所以该表达式也可以出现在赋值语句的左边，表示更新指针所指向的变量的值

```go
x := 1
p := &x         // p, of type *int, points to x
fmt.Println(*p) // "1"
*p = 2          // equivalent to x = 2
fmt.Println(x)  // "2"
```

任何类型的指针的零值都是`nil`

在Go语言中，返回函数中局部变量的地址也是安全的。例如下面的代码，调用f函数时创建局部变量v，在局部变量地址被返回之后依然有效，因为指针p依然引用这个变量

```go
var p = f()

func f() *int {
    v := 1
    return &v
}

//每次调用f函数都将返回不同的结果：
fmt.Println(f() == f()) // "false"
```



1. 指针只声明未初始化，则指向内存地址为0的位置。*0，0-255地址为系统占用，不允许用户进行读写操作

```go
var p *int
*p = 123   //panic: runtime error: invalid memory address or nil pointer dereference
```

2. 野指针：指向未知的空间

不允许访问空指针和野指针对应的内存空间。。。



### 3.3 new函数

`new(T)`将会创建一个T类型的匿名变量，同时初始化为T类型的0值，返回变量的地址，即*T类型

```go
p := new(int)   // p, *int 类型, 指向匿名的 int 变量
fmt.Println(*p) // "0"
*p = 2          // 设置 int 匿名变量的值为 2
fmt.Println(*p) // "2"
```

new只是一种语法糖，避免了声明一个临时变量的名字

p := new(int) 也就相当于 var t int,  p := &t



### 3.4 变量的生命周期和逃逸分析

- 包级别声明的变量的生命周期和整个程序的生命周期一致
- 局部变量有动态的生命周期：从创建该局部变量起，到该变量不再有引用为止。。因此局部变量可能超出其局部作用域，在函数返回之后依然存在。。**对于逃逸的局部变量，必须分配在堆上**，对于没有逃逸的局部变量，编译器可以选择分配在栈上。（也可以分配在堆上由gc进行回收）

发生逃逸的情况主要有两种：

1. 方法逃逸：当一个对象在方法中定义之后，作为参数传递或返回值到其他方法中
2. 线程逃逸：可能被其他线程访问到的变量

这里主要针对**方法逃逸**进行分析，通过逃逸分析判断一个变量到底是分配在栈上还是堆上

**策略**：编译器如果不能证明某个变量在函数返回后不再被引用，则分配在堆上。如果一个变量过大，也可能被分配在堆上

**目的：** 尽量在栈上分配，减少GC压力。栈上分配更高效。未逃逸对象可以进行锁消除



工具：`go run -gcflags "-m -l"`

说明：

moved to heap:xxx 和 xxx escapes to heap 都表示逃逸，前者表示值类型逃逸，后者表示指针类型逃逸





#### 1. 函数返回局部变量地址引起逃逸

函数**返回了局部变量的指针**，这种情况该变量一定发生了逃逸

```go
func f() (int, *int) {
	var a = 2
	var c = 2
	//返回函数局部变量地址
	return a, &c
}

func main() {
	f()
}

/*
$ go run -gcflags "-m -l" addr_escape.go
# command-line-arguments
.\addr_escape.go:5:6: moved to heap: c
*/
```

可以看到 c 逃逸了，而 a 没有逃逸



#### 2. 被已逃逸的指针引用的指针会逃逸

某个值**取地址传递给函数**并不会发生逃逸

```go
func f() (int, *int) {
	var a = 2
	var c = 2
	//返回函数局部变量地址
	return a, &c
}

func add(a *int, b *int) int {
	return *a + *b
}
func main() {
	x, y := f()
	add(&x, y)
}
/*
$ go run -gcflags "-m -l" addr_escape.go
# command-line-arguments
.\addr_escape.go:5:6: moved to heap: c
.\addr_escape.go:10:10: a does not escape
.\addr_escape.go:10:18: b does not escape
*/
```

可以看到 add 函数 传了 x 的地址，不会引起x逃逸



而**被已经逃逸的变量引用的指针一定会发生逃逸**

```go
func f() (int, *int) {
	var a = 2
	var c = 2
	//返回函数局部变量地址
	return a, &c
}

func main() {
	x, y := f()
	fmt.Printf("addr_x: %p\naddr_y: %p\n", &x, y)
}

/*
$ go run -gcflags "-m -l" addr_escape.go
# command-line-arguments
.\addr_escape.go:7:6: moved to heap: c
.\addr_escape.go:13:2: moved to heap: x
.\addr_escape.go:14:12: ... argument does not escape
addr_x: 0xc000012120
addr_y: 0xc000012128
*/
```

```go
type T struct {
	TA *int
	TB int
}

func f() (int, int) {
	var a = 2
	var c = 2
	//返回函数局部变量地址
	return a, c
}

func newT(ta *int, tb int) *T {
	t := T{TA: ta, TB: tb}
	return &t
}

func main() {
	x, y := f()
	t := newT(&x, y)
	_ = t
}

/*
$ go run -gcflags "-m -l" addr_escape.go
# command-line-arguments
.\addr_escape.go:15:11: leaking param: ta
.\addr_escape.go:16:2: moved to heap: t
.\addr_escape.go:21:2: moved to heap: x
*/
```

这里t逃逸了！！ t又引用了x的地址，所以引起 x 逃逸，这才是逃逸的根本原因。。

至于Printf：

fmt.Printf 源码，p这个指针是通过newPrinter返回的，p是逃逸的，而p引用了 arg，arg是指针的时候就会造成arg逃逸

```go
func Fprintf(w io.Writer, format string, a ...interface{}) (n int, err error) {
	p := newPrinter()		// p 逃逸了
	p.doPrintf(format, a)
...
}

func (p *pp) doPrintf(format string, a []interface{}) {
...
p.printArg(a[argNum], rune(c))
...
}

func (p *pp) printArg(arg interface{}, verb rune) {  
	p.arg = arg					// 逃逸的变量p，引用了arg，arg是指针的时候，也就是 x ，逃逸了，不是指针不逃逸
	p.value = reflect.Value{}
...
}
```



#### 3. map、slice、chan引用的指针逃逸

**被指针类型的slice、map和chan引用的指针一定发生逃逸**

> 备注：stack overflow上有人提问为什么使用指针的chan比使用值的chan慢30%，答案就在这里：使用指针的chan发生逃逸，gc拖慢了速度。问题链接https://stackoverflow.com/questions/41178729/why-passing-pointers-to-channel-is-slower

```go
func main() {
	a := make([]*int, 1)
	b := 12
	a[0] = &b

	c := make(map[string]*int)
	d := 14
	c["aaa"] = &d

	e := make(chan *int, 1)
	f := 15
	e <- &f
}

/*
$ go run -gcflags "-m -l" slice_map_chan_escape.go
# command-line-arguments
.\slice_map_chan_escape.go:5:2: moved to heap: b
.\slice_map_chan_escape.go:9:2: moved to heap: d
.\slice_map_chan_escape.go:13:2: moved to heap: f
.\slice_map_chan_escape.go:4:11: make([]*int, 1) does not escape
.\slice_map_chan_escape.go:8:11: make(map[string]*int) does not escape
*/
```



我们得出了指针**必然发生逃逸**的三种情况（go version go1.13.4 darwin/amd64)：

- 在某个函数中new或字面量创建出的变量，将其指针作为函数返回值，则该变量一定发生逃逸（构造函数返回的指针变量一定逃逸）；
- 被已经逃逸的变量引用的指针，一定发生逃逸；
- 被指针类型的slice、map和chan引用的指针，一定发生逃逸；



同时我们也得出一些**必然不会逃逸**的情况：

- 指针被未发生逃逸的变量引用；
- 仅仅在函数内对变量做取址操作，而未将指针传出；



有一些情况**可能发生逃逸，也可能不会发生逃逸**：

- 将指针作为入参传给别的函数；这里还是要看指针在被传入的函数中的处理过程，如果发生了上边的三种情况，则会逃逸；否则不会逃逸；



#### 4. 栈空间不足逃逸

创建较小的slice，没有逃逸：

```go
func main() {
	stack()
}

func stack() {
	s := make([]int, 10, 10)
	s[0] = 1
}

/*
$ go run -gcflags "-m -l" stack_escape.go
# command-line-arguments
.\stack_escape.go:8:11: make([]int, 10, 10) does not escape
*/
```



创建超大slice，逃逸：

```go
func main() {
	stack()
}

func stack() {
	s := make([]int, 10000, 10000)
	s[0] = 1
}

/*
$ go run -gcflags "-m -l" stack_escape.go
# command-line-arguments
.\stack_escape.go:8:11: make([]int, 10000, 10000) escapes to heap
*/
```



#### 5. 动态类型逃逸

```go
func main() {
	dynamic()
}

func dynamic() interface{} {
	i := 0
	return i
}

/*
$ go run -gcflags "-m -l" interface_escape.go
# command-line-arguments
.\interface_escape.go:9:2: i escapes to heap
*/
```



#### 6. 闭包引用逃逸

```go
func main() {
	f := fibonacci()
	for i := 0; i < 10; i++ {
		f()
	}
}
func fibonacci() func() int {
	a, b := 0, 1
	return func() int {
		a, b = b, a+b
		return a
	}
}

/*
$ go run -gcflags "-m" closure_escape.go
# command-line-arguments
.\closure_escape.go:11:9: can inline fibonacci.func1
.\closure_escape.go:10:2: moved to heap: a
.\closure_escape.go:10:5: moved to heap: b
.\closure_escape.go:11:9: func literal escapes to heap
*/
```





### 3.5 赋值

**元组赋值：**在赋值之前，赋值语句右边的所有表达式将会先进行求值，然后再统一更新左边对应变量的值

```go
x, y = y, x
```

**丢弃不需要的值**：

```go
_, err = io.Copy(dst, src) // 丢弃字节数
_, ok = x.(T)              // 只检测类型，忽略具体值
```

**可赋值性：**类型必须完全匹配才能进行赋值，nil可以赋值给任何指针或引用类型的变量

对于两个值是否可以用`==`或`!=`进行相等比较的能力也和可赋值能力有关系：对于任何类型的值的相等比较，第二个值必须是对第一个值类型对应的变量是可赋值的，反之亦然



**退化赋值：** 简短模式并不总是重新定义变量，也可能是部分退化的赋值操作。

```go
func main() {
    x := 100
    println(&x)

    x, y := 200, "abc"     // 注意: x 退化为赋值操作，仅有 y 是变量定义。

    println(&x, x)
    println(y)
}

/*
0xc00002e770
0xc00002e770 200         // 对比变量内存地址，可以确认 x 属于同一变量。
abc
*/
```



**调整作用域可以修正赋值：**

```go
func main() {
    x := 100
    println(&x, x)

    {                         // 使用大括号定义作用域。
        x, y := 200, 300      // 不同作用域，全部是新变量定义。
        println(&x, x, y)
    }
}

/*
0xc00002e770 100
0xc00002e768 200 300    // 两个x的地址不一样
*/
```



### 3.6 类型

使用type来声明一个新的类型：`type 类型名字 底层类型`

类型声明语句一般出现在包一级，因此如果新创建的类型名字的首字符大写，则在包外部也可以使用

```go
package tempconv

import "fmt"

type Celsius float64    // 摄氏温度
type Fahrenheit float64 // 华氏温度

const (
    AbsoluteZeroC Celsius = -273.15 // 绝对零度
    FreezingC     Celsius = 0       // 结冰点温度
    BoilingC      Celsius = 100     // 沸水温度
)

func CToF(c Celsius) Fahrenheit { return Fahrenheit(c*9/5 + 32) }

func FToC(f Fahrenheit) Celsius { return Celsius((f - 32) * 5 / 9) }
```

它们虽然有着相同的底层类型float64，但是**它们是不同的数据类型**，因此它们**不可以被相互比较**或**混在一个表达式运算**

**类型转换：** 只有当两个类型的底层基础类型相同时，才允许这种转型操作，或者是两者都是指向相同底层结构的指针类型，这些转换只改变类型而不会影响值本身

数值类型之间的转型也是允许的，并且在字符串和一些特定类型的slice之间也是可以转换的



比较运算符`==`和`<`也可以用来比较一个命名类型的变量和另一个有相同类型的变量，或有着相同底层类型的未命名类型的值之间做比较。但是如果两个值有着不同的类型，则不能直接进行比较：

```go
var c Celsius
var f Fahrenheit
fmt.Println(c == 0)          // "true"
fmt.Println(f >= 0)          // "true"
fmt.Println(c == f)          // compile error: type mismatch
fmt.Println(c == Celsius(f)) // "true"!
```

**Celsius(f)是类型转换操作，它并不会改变值**，仅仅是改变值的类型而已。测试为真的原因是因为c和g都是零值

### 3.7 包、文件、init函数

对于在包级别声明的变量，如果有初始化表达式则用表达式初始化，还有一些没有初始化表达式的，例如某些表格数据初始化并不是一个简单的赋值过程。在这种情况下，我们可以用一个特殊的init初始化函数来简化初始化工作。每个文件都可以包含多个init初始化函数

```go
func init() { /* ... */ }
```

这样的init初始化函数除了不能被调用或引用外，其他行为和普通函数类似。在每个文件中的init初始化函数，在程序开始执行时按照它们声明的顺序被自动调用



初始化工作是自下而上进行的，main包最后被初始化。以这种方式，可以确保在main函数执行之前，所有依赖的包都已经完成初始化工作了。同时p包若导入了q包，p包初始化的时候就认为q包已经初始化完成了



**main包：** 编译器发现**包名为main的包**，并且**包含main()函数**，才会**将该包编译为二进制可执行文件**。编译时会使用main包的代码所在目录的名字作为二进制可执行文件的文件名



**编译器查找import的包：** 首先查找Go的安装目录，然后按顺序查找GOPATH变量里的目录，未找到则会在执行run或者build的时候报错



**命名导入：**

```go
import(
	"fmt"
  myfmt "mylib/fmt"   //命名为新的名
)
```

**空白标识符：** 给导入的包赋予一个空名字`_` ，可以避免不使用该包的报错，但是init函数依然会执行，例如数据库驱动包的init函数将自己注册到sql包



**init函数：** 一个包可以包含多个init函数，编译器会保证所有的init函数都在main函数前执行

### 3.8 变量的作用域

一个声明语句将程序中的实体和一个名字关联，比如一个函数或一个变量。声明语句的作用域是指**源代码中可以有效使用这个名字的范围**

**生命周期和作用域：**声明语句的作用域对应的是一个源代码的文本区域；它是一个**编译时的属性**。一个变量的生命周期是指程序运行时变量存在的有效时间段，在此时间区域内它可以被程序的其他部分引用；是一个**运行时的概念**

```go
if x := f(); x == 0 {    //可以执行一个语句之后再进行判断
    fmt.Println(x)
} else if y := g(x); x == y {
    fmt.Println(x, y)
} else {
    fmt.Println(x, y)
}
fmt.Println(x, y) // compile error: x and y are not visible here
```



# 二、基本数据类型

## 1. 整形

整形：int8、int16、int32、int64 以及 uint8、uint16、uint32、uint64

int和uint则根据机器字长变化，32或64位

Unicode字符rune类型和int32类型等价

byte类型和int8类型等价

uintptr：没有指定具体的bit大小，但是足以容纳指针



`&^`位清空运算符：z = x &^ y，如果y为1，则z为0，如果y为0，则z为x

```go
o := 0666
fmt.Printf("%d %[1]o %#[1]o\n", o) // "438 666 0666"
x := int64(0xdeadbeef)
fmt.Printf("%d %[1]x %#[1]x %#[1]X\n", x)  //3735928559 deadbeef 0xdeadbeef 0XDEADBEEF
```

- %之后的`[1]`副词告诉Printf函数再次使用第一个操作数
- %后的`#`副词告诉Printf在用%o、%x或%X输出时生成0、0x或0X前缀。

```go
ascii := 'a'
fmt.Printf("%d %[1]c %[1]q\n", ascii)   // "97 a 'a'"
```

## 2. 浮点数

go语言提供了两种精度的浮点数：float32和float64

### NaN

- 正无穷大和负无穷大，分别用于表示太大溢出的数字和除零的结果；还有NaN非数，一般用于表示无效的除法操作结果0/0或Sqrt(-1)

```go
var z float64
fmt.Println(z, -z, 1/z, -1/z, z/z) // "0 -0 +Inf -Inf NaN"
```

函数`math.IsNaN`用于测试一个数是否是非数NaN，`math.NaN`则返回非数对应的值

另外NaN和正负无穷大都是不能直接比较的

```go
nan := math.NaN()
fmt.Println(nan == nan, nan < nan, nan > nan) // "false false false"
```

## 3. 复数

go语言提供了两种精度的复数：complex64和complex128，分别对应float32和float64两种精度的浮点数精度

```go
var x complex128 = complex(1, 2) // 1+2i
var y complex128 = complex(3, 4) // 3+4i
fmt.Println(x*y)                 // "(-5+10i)"
fmt.Println(real(x*y))           // "-5"
fmt.Println(imag(x*y))           // "10"
```

## 4. 布尔类型

true和false

&&的优先级比｜｜高

布尔类型不会隐式转换为0，1，如有需要应该手动进行转换

## 5. 字符串

**字符串类型自身存储了长度，所以不需要\0作为结束标志**

字符串底层结构在`reflect.StringHeader`中定义

```go
type StringHeader struct {
	Data uintptr
	Len  int
}
```

可以通过反射获取长度：

```go
s := "hello, world"
fmt.Println((*reflect.StringHeader)(unsafe.Pointer(&s)).Len)
```



字符串是utf8编码的，字符串通常被解释成UTF-8编码的Unicode码点（rune）序列(使用for range循环即可)

字符串是**不可改变的字节序列**，len函数返回字符串的**字节数目**！所以第i个字节并不一定是字符串的第i个字符，因为对于非ASCII字符的UTF8编码会要两个或多个字节

```go
	s := "left foot"
	t := s
	s += "left foot, right foot"[9:]     //支持 切片操作
	fmt.Println(s)  //left foot, right foot
	fmt.Println(t)  //left foot    不改变原来的字符串（因为字符串是不可变的）
```

尝试修改字符串的内部数据的操作是禁止的：`s[0]='L'. // compile error: cannot assign to s[0]`

```go
	m := "你好"
	fmt.Println(len(m))	//6
	fmt.Println(utf8.RuneCountInString(m))  //2
```

**原生字符串：** 使用一对反引号表示原生字符串，原生字符串里想要使用反引号可以通过拼接普通字符串的形式完成

Utf-8编码：

```go
0xxxxxxx                             runes 0-127    (ASCII)
110xxxxx 10xxxxxx                    128-2047       (values <128 unused)
1110xxxx 10xxxxxx 10xxxxxx           2048-65535     (values <2048 unused)
11110xxx 10xxxxxx 10xxxxxx 10xxxxxx  65536-0x10ffff (other values unused)
```

Utf-8编码是前缀编码，没有任何字符的编码是其它字符编码的子串，或是其它编码序列的字串，因此搜索一个字符时只要搜索它的字节编码序列即可，不用担心前后的上下文会对搜索结果产生干扰

**目前rune(unicode码点)只使用了21位**

**utf8解码器**可以帮助我们解码原生的utf8编码的字符串，另外range循环处理字符串的时候会隐式的解码utf8

`for range`不支持非utf8编码的字符串的遍历，非utf8编码的字符串可以看做是**只读的二进制数组**

字符串中实际存储的是utf8编码，使用for range可以获得rune码点



**如果不想解码utf8字符串**，想直接遍历原始字节码，可以将字符串强转为`[]byte`字节数组，或者直接按照下标顺序访问

```go
	s := "a中国"
	for i := 0; i < len(s); {   //for range 底层其实就是这种方法
		r, size := utf8.DecodeRuneInString(s[i:])
		fmt.Printf("%d\t%c\n", i, r)
		i += size
	}
	fmt.Println()
	for i, r := range s {
		fmt.Printf("%d\t%c\n", i, r)
	}
/*
0       a
1       中
4       国

0       a
1       中
4       国
*/

	str := "世"
	for _, c := range []byte(str) {   //转换为字节数组，，一般没有运行时开销
		fmt.Printf("%x ", c)
	}
	fmt.Println()
	for i := 0; i < len(str); i++ {  //逐字节访问
		fmt.Printf("%x ", str[i])
	}
	fmt.Println()
	for _, c := range str {  				//按rune访问
		fmt.Printf("%x  ", c)
	}
/*
e4 b8 96    // 11100100  10111000 10010110
4e16        //     0100    1110  0001 0110
*/

```

**utf8编码的字符串转换为[]rune的unicode码点序列([]int32)**：程序内部使用rune序列更方便，rune大小一致，支持数组索引和方便切割。

- 直接进行[]rune类型和string类型的转换即可
- **[]rune实际上就是[]int32类型**，只是个别名，不是重新定义的类型

```go
	//[]rune 和 string 的互相转换

	s = "a中🐶"
	fmt.Printf("% x\n", s) // "61 e4 b8 ad f0 9f 90 b6"
	r := []rune(s)
	fmt.Printf("%x\n", r) // "[61 4e2d 1f436]"
	fmt.Println(string(r)) // "a中🐶"

	fmt.Printf("%#v\n", []rune("世界"))    //[]int32{19990, 30028}
	fmt.Printf("%#v\n", string([]rune{'世', '界'}))   //"世界"
```



**Java中char是2字节，所以只能编码到unicode码点的FFFF，即三个字节utf-8编码，四个字节的utf-8编码一个char是装不下的，需要两个char**



**字符串和byte切片：bytes、strings、strconv、unicode包**

- strings包提供了许多如字符串的查询、替换、比较、截断、拆分和合并等功能。
- bytes包也提供了很多类似功能的函数，但是针对和字符串有着相同结构的[]byte类型。因为字符串是只读的，因此逐步构建字符串会导致很多分配和复制。在这种情况下，使用bytes.Buffer类型将会更有效
- strconv包提供了布尔型、整型数、浮点数和对应字符串的相互转换，还提供了双引号转义相关的转换
- unicode包提供了IsDigit、IsLetter、IsUpper和IsLower等类似功能，它们用于给字符分类。每个函数有一个单一的rune类型的参数，然后返回一个布尔值。而像ToUpper和ToLower之类的转换函数将用于rune字符的大小写转换。所有的这些函数都是遵循Unicode标准定义的字母、数字等分类规范。strings包也有类似的函数，它们是ToUpper和ToLower，将原始字符串的每个字符都做相应的转换，然后返回新的字符串

basename函数

```go
fmt.Println(basename("a/b/c.go")) // "c"
fmt.Println(basename("c.d.go"))   // "c.d"
fmt.Println(basename("abc"))      // "abc"

func basename(s string) string {
    slash := strings.LastIndex(s, "/") // -1 if "/" not found
    s = s[slash+1:]
    if dot := strings.LastIndex(s, "."); dot >= 0 {
        s = s[:dot]
    }
    return s
}
```

一个[]byte(s)转换是分配了一个新的字节数组用于保存字符串数据的拷贝，然后引用这个底层的字节数组.. 因为string是不可变的

**strings提供的函数**如下：

```go
func Contains(s, substr string) bool   //是否存在指定的子串
func Count(s, sep string) int   //统计sep的数量
func Fields(s string) []string  //去掉空格，并按原空格位置进行切割，切割成字符串数组
func HasPrefix(s, prefix string) bool
func Index(s, sep string) int   //返回指定子串的开始下标，找不到返回-1
func Join(a []string, sep string) string  //将字符串切片使用sep拼接成string
```

**bytes也对应六个函数**：它们之间唯一的区别是字符串类型参数被替换成了字节slice类型的参数。bytes包还提供了Buffer类型用于字节slice的缓存。一个Buffer开始是空的，但是随着string、byte或[]byte等类型数据的写入可以动态增长，一个bytes.Buffer变量并不需要初始化，因为零值也是有效的

```go
func Contains(b, subslice []byte) bool
func Count(s, sep []byte) int
func Fields(s []byte) [][]byte
func HasPrefix(s, prefix []byte) bool
func Index(s, sep []byte) int
func Join(s [][]byte, sep []byte) []byte
```

```go
func intsToString(values []int) string {
    var buf bytes.Buffer
    buf.WriteByte('[')
    for i, v := range values {
        if i > 0 {
            buf.WriteString(", ")
        }
        fmt.Fprintf(&buf, "%d", v)
    }
    buf.WriteByte(']')
    return buf.String()
}
```

**字符串和数字的相互转换**

将一个整数转为字符串，一种方法是用fmt.Sprintf返回一个格式化的字符串；另一个方法是用strconv.Itoa(“整数到ASCII”)：

```go
x := 123
y := fmt.Sprintf("%d", x)
fmt.Println(y, strconv.Itoa(x)) // "123 123"
```

FormatInt和FormatUint函数可以用不同的进制来格式化数字：

```go
fmt.Println(strconv.FormatInt(int64(x), 2)) // "1111011"
```

如果要将一个字符串解析为整数，可以使用strconv包的Atoi或ParseInt函数，还有用于解析无符号整数的ParseUint函数：

```go
x, err := strconv.Atoi("123")             // x is an int
y, err := strconv.ParseInt("123", 10, 64) // base 10, up to 64 bits
```

## 6.  常量

- 字面值常量

- const声明

- iota常量生成器

**常量赋值后不能进行修改**

常量存储在数据区

### 命名常量和字面值常量

命名常量既可以申明为`无类型`的，也可以申明为`有类型`的，声明一个字面量的时候，其实是声明了一个`匿名的无类型常量`

```go
const untypedInteger       = 12345
const untypedFloatingPoint = 3.141592

const typedInteger int           = 12345
const typedFloatingPoint float64 = 3.141592
```



**常量是完全精确的**，不管是多大的整数或者是浮点数，都是精确的，这点跟变量不一样

声明一个有类型常量时，右边常量的形式必须要与左边常量的类型兼容

```go
// 远远大于 int64 的整数
const myConst = 9223372036854775808543522345    //可以编译通过
const myConst int64 = 9223372036854775808543522345  // overflows int64 不会编译通过
```



**变量之间是不会进行隐式类型转换的，而常量与变量会经常发生隐式类型转换**

```go
var myInt int = 123.0     // 浮点数常量隐式转换为整型变量
var myInt2 = 123   //省略变量类型进行常量-变量转换，，，，默认使用 int
```



**常量的种类提升：** 除了移位运算，如果二元运算的操作数是不同种类的无类型常量，运算结果使用以下种类中最靠后的一个：整数、Unicode 字符、浮点数、复数

```go
var answer = 3 * 0.333     // answer会是浮点数类型 float64
```



**关于数字常量的比较：**

```go
	a := int64(1)
	b := 1
	fmt.Println(a == 1)     // true , 进行隐式类型转换比较
	fmt.Println(b == 1)     // true , 同上
	//fmt.Println(a == b)   // mismatched types int64 and int
```



### 无类型常量

许多常量并没有一个明确的基础类型。编译器为这些没有明确基础类型的数字常量提供比基础类型更高精度的算术运算；你可以认为至少有256bit的运算精度。这里有六种未明确类型的常量类型，分别是无类型的布尔型、无类型的整数、无类型的字符、无类型的浮点数、无类型的复数、无类型的字符串

除法运算符/会根据操作数的类型生成对应类型的结果，不同写法的常量除法可能产生不同的结果

```go
var f float64 = 212
fmt.Println((f - 32) * 5 / 9)     // "100"; (f - 32) * 5 is a float64
fmt.Println(5 / 9 * (f - 32))     // "0";   5/9 is an untyped integer, 0
fmt.Println(5.0 / 9.0 * (f - 32)) // "100"; 5.0/9.0 is an untyped float
```

只有常量可以是无类型的。当一个无类型的常量被赋值给一个变量的时候，就会进行隐式的类型转换

```go
var f float64 = 3 + 0i // untyped complex -> float64 相当于  var f float64 = float64(3 + 0i)
f = 2                  // untyped integer -> float64 相当于  f = float64(2)
```

对于一个没有显式类型的变量声明（包括简短变量声明），常量的形式将隐式决定变量的默认类型。无类型整数常量转换为int，它的内存大小是不确定的，但是无类型浮点数和复数常量则转换为内存大小明确的float64和complex128



# 三、复合数据类型

## 1. 数组

长度固定，所以一般很少用，一般用Slice，可以动态收缩和扩容

**go语言中数组是值语义，一个数组变量就表示整个数组，而不是隐式指向第一个元素的指针(C语言)**

数组变量被赋值或者传递的时候，会**复制整个数组**，为了减少开销，可以传递数组的指针，但是数组指针并不是数组

```go
const (
	apple int = iota
	banana
	orange
)

func main() {
	arr := []int{apple: 3, orange: 1, banana: 2, 10: 10}
	fmt.Println(arr)   //[3 2 1 0 0 0 0 0 0 0 10]
  
  a := [2]int{1, 2}
  b := [...]int{1:2, 0:1}
  c := [3]int{1,2}
  fmt.Println(a == b)   //true
  fmt.Println(a == c)   //Invalid operation: a == c (mismatched types [2]int and [3]int)
}
```

数组传的是**值**

```go
	a := [2]int{1, 2}	
	changeArr(a)
	fmt.Println(a)  //[1 2]

func changeArr(arr [2]int) {
	arr = [2]int{9, 9}
}
```

数组类型非常僵化，因为**数组的类型中封装了数组的长度**，只有相同长度相同类型的数组才可以进行函数的参数传递，所以很少使用，除非是对特定长度的数组进行操作。。一般使用Slice来代替数组

对数组来说，`Len()`和`Cap()`的值是永远相等的

还可以定义结构体数组，函数数组，接口数组，通道数组等



**长度为0的数组不占用内存空间，和无类型的空struct一样可以用来控制channel**

### 指针数组

指针数组赋值时浅拷贝，指针指向的是同一片内存



### 数组指针

```go
var arr [5]int{1,2,3,4,5}
var p *[5]int
p = &arr
fmt.Println((*p)[1])    //2
fmt.Println(p[1])		//2
fmt.Println((&arr)[1])	//2
fmt.Println(&arr[0])	//0xc00001a0e0
fmt.Println(&arr)		//&[1 2 3 4 5]
fmt.Printf("%p\n", &arr)//0xc00001a0e0
```

访问成员的方法：`(*p)[index]`，也可以简写为`p[index]`

## 2. Slice

**数组存在的问题：**

	1. 数组容量固定，不能自动拓展，且类型是带着长度的
	2. 数组是值传递，数组作为函数参数时会传递数组的拷贝

Slice（切片）代表变长的序列，序列中每个元素都有相同的类型。一个slice类型一般写作[]T，其中T代表slice中元素的类型。Slice没有固定的长度

一个slice由三个部分构成：**指针、长度和容量**。指针指向第一个slice元素对应的底层数组元素的地址，要注意的是slice的第一个元素并不一定就是数组的第一个元素。长度对应slice中元素的数目；长度不能超过容量，容量一般是从slice的开始位置到底层数据的结尾位置。内置的`len`和`cap`函数分别返回slice的长度和容量

如果切片操作超出cap(s)的上限将导致一个panic异常，但是超出len(s)则是意味着扩展了slice，因为新slice的长度会变大

和数组不同的是，**slice之间不能比较**，因此我们不能使用==操作符来判断两个slice是否含有全部相等元素。不过标准库提供了高度优化的`bytes.Equal`函数来判断两个字节型slice是否相等（[]byte），但是对于其他类型的slice，我们必须自己展开每个元素进行比较

- **make函数**创建制定len和cap的Slice，省略cap时cap将和len相等...底层就是**创建了一个匿名的数组**，然后返回Slice

```go
make([]T, len)
make([]T, len, cap) // same as make([]T, cap)[:len]
```

- **内置的append函数用于向slice追加元素**：我们不太好知道新的Slice和原Slice是否引用了同一个底层数组（即是否进行了内存的重新分配），所以不能确定在原来Slice上的操作是否会影响到新的Slice。因此通常直接将append返回的结果赋值给输入的Slice变量

```go
runes = append(runes, r)

//append函数也可以追加多个元素，或者追加一个slice
x = append(x, 4, 5, 6)
x = append(x, x...) // append the slice x
```



**cap并不是总容量，而是切片起点到数组右边界的容量**

Slice[i:j] 长度：j-i.  容量：k-i

```go
	a := [5]int{0,1,2,3,4}
	b := a[1:3]
	c := a[2:3]
	fmt.Println(len(b), cap(b))   //2 4
	fmt.Println(len(c), cap(c))   //1 3
```



### 空slice和nil slice

- 为nil的Slice没有底层数组（数组指针是nil），len和cap都是0，可以进行比较。但是非nil的Slice也可以len和cap为0，与普通Slice一样

```go
var s []int    // len(s) == 0, s == nil
s = nil        // len(s) == 0, s == nil
s = []int(nil) // len(s) == 0, s == nil
s = []int{}    // len(s) == 0, s != nil
s = make([]int, 0) // len(s) == 0, s != nil
```

除了和nil相等比较外，**一个nil值的slice的行为和其它任意0长度的slice一样**；例如reverse(nil)也是安全的。除了文档已经明确说明的地方，所有的Go语言函数应该以相同的方式对待nil值的slice和0长度的slice



**空slice用于实现filter**

```go
func Filter(s []byte, f func(x byte) bool) []byte {
  b := s[:0]
  for _,x := range s {
    if !f(x) {
      b = append(b, x)
    }
  }
  return b
}
```





### 切片指针

切片本身就是指针，所以切片的指针是指针的指针

由于是两级指针，所以访问切片成员只能`(*p)[index]`不能简写



当 slice 作为函数参数时，就是一个普通的结构体。若直接传 slice，在调用者看来，实参 slice 并不会被函数中的操作改变；若传的是 slice 的指针，在调用者看来，是会被改变原 slice 的。

值的注意的是，不管传的是 slice 还是 slice 指针，如果改变了 slice 底层数组的数据，会反应到实参 slice 的底层数据。底层数据在 slice 结构体里是一个指针，仅管 slice 结构体自身不会被改变，也就是说底层数据地址不会被改变。 但是通过指向底层数据的指针，可以改变切片的底层数据

```go
func main() {
	s := []int{1, 1, 1}
	f(s)
	fmt.Println(s)
}

func f(s []int) {
	// i只是一个副本，不能改变s中元素的值
	/*for _, i := range s {
		i++
	}
	*/

	for i := range s {
		s[i] += 1
	}
}
```

运行一下，程序输出：

```go
[2 2 2]
```

果真改变了原始 slice 的底层数据。这里传递的是一个 slice 的副本，在 `f` 函数中，`s` 只是 `main` 函数中 `s` 的一个拷贝。在`f` 函数内部，对 `s` 的作用并不会改变外层 `main` 函数的 `s`。

要想真的改变外层 `slice`，只有将返回的新的 slice 赋值到原始 slice，或者向函数传递一个指向 slice 的指针：

```go
func myAppend(s []int) []int {
	// 这里 s 虽然改变了，但并不会影响外层函数的 s
	s = append(s, 100)
	return s
}

func myAppendPtr(s *[]int) {
	// 会改变外层 s 本身
	*s = append(*s, 100)
	return
}

func main() {
	s := []int{1, 1, 1}
	newS := myAppend(s)

	fmt.Println(s)
	fmt.Println(newS)

	s = newS

	myAppendPtr(&s)
	fmt.Println(s)
}

//运行结果
//[1 1 1]
//[1 1 1 100]
//[1 1 1 100 100]
```



### 切片扩容

源代码位于 runtime/slice.go 中

```go
func growslice(et *_type, old slice, cap int) slice {  //cap是扩容后期望的最小容量，即oldCap+appendCap
 ...
 newcap := old.cap
 doublecap := newcap + newcap
 if cap > doublecap {
  newcap = cap
 } else {
  if old.cap < 1024 {
   newcap = doublecap
  } else {
   for 0 < newcap && newcap < cap {
    newcap += newcap / 4
   }
   if newcap <= 0 {
    newcap = cap
   }
  }
 }
 ...
}
```

如果**期望容量大于当前容量的两倍**就会使用期望容量

否则如果当前切片的长度小于 1024 就会将容量翻倍； 如果当前切片的长度大于 1024 就会每次增加 25% 的容量，直到新容量大于期望容量

```go
  // 例如
	s := []string{"a", "b"}
	s = append(s, "c", "d", "e")
	fmt.Printf("%d %d", len(s), cap(s))  //cap=5 > doublecap=4, 所以计算得到 5，5

	a := []int{1, 2}
	a = append(a, 4, 5, 6)
	fmt.Printf("%d %d", len(a), cap(a))  //运行结果：5,6 ???
```



上面只是确定切片的大致容量，接下来**还需要根据切片中元素的大小对它们进行内存对齐**，所以新Slice的容量会大于等于老Slice的两倍或者1.25倍的

```go
capmem := roundupsize(uintptr(newcap) * uintptr(et.size))
newcap = int(capmem / ptrSize)
```

newcap就是前面计算出的newcap，et.size代表slice中一个元素的大小，**capmem计算出来的是此次扩容需要申请的内存大小**。**roundupsize函数是处理内存对齐的函数**

上面的例子，roundupsize函数会传入 5*8 = 40

```go
func roundupsize(size uintptr) uintptr {
	if size < _MaxSmallSize {
		if size <= smallSizeMax-8 {
			return uintptr(class_to_size[size_to_class8[(size+smallSizeDiv-1)/smallSizeDiv]])
		} else {
			//……
		}
	}
    //……
}

const _MaxSmallSize = 32768
const smallSizeMax = 1024
const smallSizeDiv = 8
```

很明显，我们最终将返回这个式子的结果：

```go
class_to_size[size_to_class8[(size+smallSizeDiv-1)/smallSizeDiv]]
```

(size+smallSizeDiv-1)/smallSizeDiv，其实就是 size➗smallSizeDiv的结果向上取整

这是 `Go` 源码中有关内存分配的两个 `slice`。`class_to_size`通过 `spanClass`获取 `span`划分的 `object`大小。而 `size_to_class8` 表示通过 `size` 获取它的 `spanClass`。

```go
var size_to_class8 = [smallSizeMax/smallSizeDiv + 1]uint8{0, 1, 2, 3, 3, 4, 4, 5, 5, 6, 6, 7, 7, 8, 8, 9, 9, 10, 10, 11, 11, 12, 12, 13, 13, 14, 14, 15, 15, 16, 16, 17, 17, 18, 18, 18, 18, 19, 19, 19, 19, 20, 20, 20, 20, 21, 21, 21, 21, 22, 22, 22, 22, 23, 23, 23, 23, 24, 24, 24, 24, 25, 25, 25, 25, 26, 26, 26, 26, 26, 26, 26, 26, 27, 27, 27, 27, 27, 27, 27, 27, 28, 28, 28, 28, 28, 28, 28, 28, 29, 29, 29, 29, 29, 29, 29, 29, 30, 30, 30, 30, 30, 30, 30, 30, 30, 30, 30, 30, 30, 30, 30, 30, 31, 31, 31, 31, 31, 31, 31, 31, 31, 31, 31, 31, 31, 31, 31, 31}

var class_to_size = [_NumSizeClasses]uint16{0, 8, 16, 32, 48, 64, 80, 96, 112, 128, 144, 160, 176, 192, 208, 224, 240, 256, 288, 320, 352, 384, 416, 448, 480, 512, 576, 640, 704, 768, 896, 1024, 1152, 1280, 1408, 1536, 1792, 2048, 2304, 2688, 3072, 3200, 3456, 4096, 4864, 5376, 6144, 6528, 6784, 6912, 8192, 9472, 9728, 10240, 10880, 12288, 13568, 14336, 16384, 18432, 19072, 20480, 21760, 24576, 27264, 28672, 32768}
```

我们传进去的 `size` 等于 40。所以 `(size+smallSizeDiv-1)/smallSizeDiv = 5`；获取 `size_to_class8` 数组中索引为 `5` 的元素为 `4`；获取 `class_to_size` 中索引为 `4` 的元素为 `48`。

最终，新的 slice 的容量为 `6`：

```go
newcap = int(capmem / ptrSize) // 6
```



### 合并切片与删除元素

```go
	a := [5]int{0, 1, 2, 3, 4}
	b := a[1:]
	c := a[1:2]
	c = append(c, b...)   //解构，将一个切片的所有元素追加到另一个切片里
	fmt.Println(b)  //[1 2 3 4]
	fmt.Println(c)  //[1 1 2 3 4]
```

由于append返回了切片，所以还可以将多个append操作组合起来，实现在切片中间插入元素

```go
var a []int
a = append( a[:i], append([]int{1,2,3}, a[i:]...)... )   //在第i个位置插入切片
```

为了避免append创建中间的临时切片，可以使用`copy()和append()`组合：

```go
a = append(a, x...)   //保证切片 a 扩展到足够的空间,没有可以直接扩展切片的方法
copy(a[i+len(x):], a[i:])
copy(a[i:], x)
```



利用copy()函数实现删除开头元素

```go
a = a[ : copy(a, a[N:] )]   //删除a的前N个元素
```



利用append()删除中间元素

```go
a = append( a[:i], a[i+N:] )   //删除从i开始的N个元素
```



利用copy()删除中间元素

```go
a = a[ : i + copy(a[i:], a[i+N:])]    //删除从i开始的N个元素
```



### 使用第三个索引保护底层数组

未使用第三个索引：

```go
	a := [5]int{0, 1, 2, 3, 4}
	b := a[1:]
	c := a[1:2]
	c = append(c, 200)
	fmt.Println(a)  //[0 1 200 3 4]
	fmt.Println(b)  //[1 200 3 4]
	fmt.Println(c)  //[1 200]
```

使用第三个索引，限制切片的容量，保护底层数组。如果限制新切片容量和长度一样，就可以在第一个append操作创建新的底层数组，与原有数组分离，安全的进行后续的修改

指定切片 slice[i:j:k]，则**长度 j-i， 容量 k-i**

```go
	a := [5]int{0, 1, 2, 3, 4}
	b := a[1:]
	c := a[1:2:2]  //容量为1
	c = append(c, 200)
	fmt.Println(a)  //[0 1 2 3 4]
	fmt.Println(b)  //[1 2 3 4]
	fmt.Println(c)  //[1 200]
```



### range迭代slice的问题

```go
func main() {
	slice := []int{1, 2, 3}
	for i, v := range slice {
		fmt.Printf("v:%d, v_addr:%p, elem_addr:%p\n", v, &v, &slice[i])
	}
}

// v:1, v_addr:0xc000018080, elem_addr:0xc000014180
// v:2, v_addr:0xc000018080, elem_addr:0xc000014188
// v:3, v_addr:0xc000018080, elem_addr:0xc000014190
```

v每次都是slice**对应的值的一个拷贝**，对v修改是不会对原slice起作用的

**v的地址没有变化**



### 切片引起内存泄漏问题

```go
func FindPhoneNumber(filename string) []byte {
  b,_ := ioutil.ReadFile(filename)
  return regexp.MustComplile("[0-9]+").Find(b)
}
```

这段代码将加载整个文件到内存，然后搜索第一个出现的电话号码，最后以切片方式返回

但是由于切片引用了整个原始数组，导致垃圾回收期不能及时释放底层数组的空间，需要长时间保存整个文件数据，降低系统的整体性能

```go
func FindPhoneNumber(filename string) []byte {
  b,_ := ioutil.ReadFile(filename)
  b = regexp.MustComplile("[0-9]+").Find(b)
  return append( []byte{}, b...)   //返回一个新分配内存的切片
}
```



再例如，使用 `ioutil.ReadAll` 读取了一个10M的文件，并将其前512K切片为另外一个slice b使用。只要b还被引用，那么其指向的底层数组也不会被释放。

### 切片强制类型转换

将`[]float64` 类型的切片转换为`[]int`类型以进行高速排序

需要`unsafe.Pointer`来连接两个不同类型的指针传递

```go
func SortFloat64V1(a []float64) {
  //强转方法1，将切片起始地址转换为一个较大的数组指针，然后重新切片
  var b []int = ((*[1<<20]int)(unsafe.Pointer(&a[0])))[:len(a):cap(a)]
  sort.Ints(b)
}

func SortFloat64V2(a []float64) {
  var c []int
  //强转方法2，使用reflect
  aHdr := (*reflect.SliceHeader)(unsafe.Pointer(&a))
  cHdr := (*reflect.SliceHeader)(unsafe.Pointer(&c))
	*cHdr = *aHdr
  sort.Ints(c)
}
```



### 切片还是传值

只是切片中包含了 Len Cap 和一个数组指针，所以可以直接修改 切片内容



### append 不是线程安全的

```go
var s []int
    var wg sync.WaitGroup
    for i := 0; i < 10; i++ {
        wg.Add(1)
        go func(i int) {
            s = append(s, i)
            wg.Done()
        }(i)
    }
    wg.Wait()
    fmt.Println(s)
```

一种可能的输出：[0 6 7 9]

## 3. Map

map类型可以写为map[K]V，一个map中所有的key都是相同的类型，所有的value也都是相同的类型

**key**可以是任何值，但**必须支持==操作**，**切片、函数以及包含切片的结构体**类型具有引用语义，不能作为map的key

结构体支持`==`操作的前提是，结构体的每个成员都支持`==`操作

创建map：

```go
//1.
ages := make(map[string]int) // mapping from strings to ints

//2.
ages := map[string]int{
    "alice":   31,
    "charlie": 34,
}

//3.创建一个空的map
ages := map[string]int{}
```

删除map中的元素：这个操作是安全的，当删除某个map中不存在的key时，会返回value对应的零值

```go
delete(ages, "alice") // remove element ages["alice"]
ages["bob"] = ages["bob"] + 1 //不存在bob也是安全的，首次访问返回0值
ages["lili"]++   //和上面意思一样
```

- 对map的元素取址的操作是不允许的，例如`_ = &ages["bob"]`，这是因为随着map的增长，可能会导致内存重新分配，从而之前的地址失效
- 使用range迭代访问map的访问顺序是不确定的，每次遍历顺序都不相同
- 要排序key的方式输出value，可以先获取全部的key，排序之后再依次获取value

需要注意：**使用var声明的map是一个nil**，不可以进行添加元素操作，应该使用make创建一个不为nil的map

map之间不能进行比较，但可以进行nil的map的比较

```go
	//var ages map[string]int
	ages := make(map[string]int)

	ages["lilei"] = 1  //使用var声明的话，会报 panic: assignment to entry in nil map
	ages["xiaohong"] = 2
	for name, age := range ages {
		fmt.Println(name, age)
	}
```

**判断map中是否包含某个key**

```go
if age, ok := ages["bob"]; !ok { /* ... */ }
```



### 空map和nil map

```go
	var ages map[string]int    // nil
	ages1 := make(map[string]int, 0)  // !=nil
	ages2 := map[string]int{}  // ！=nil
	ages3 := map[string]int(nil)  // nil
```

向nil的map中添加元素会导致 `panic: runtime error: assignment to entry in nil map`



### map的初始化

**字面量：**

当哈希表中的元素数量少于或者等于 25 个时，编译器会将字面量初始化的结构体转换成以下的代码，将所有的键值对一次加入到哈希表中：

```go
hash := make(map[string]int, 3)
hash["1"] = 2
hash["3"] = 4
hash["5"] = 6
```

一旦哈希表中元素的数量超过了 25 个，编译器会创建两个数组分别存储键和值，这些键值对会通过如下所示的 for 循环加入哈希：

```go
hash := make(map[string]int, 26)
vstatk := []string{"1", "2", "3", ... ， "26"}
vstatv := []int{1, 2, 3, ... , 26}
for i := 0; i < len(vstak); i++ {
    hash[vstatk[i]] = vstatv[i]
}
```



**运行时：**

当创建的哈希被分配到栈上并且其容量小于 `BUCKETSIZE = 8` 时，Go 语言在编译阶段会使用如下方式快速初始化哈希，这也是编译器对小容量的哈希做的优化：

```go
var h *hmap
var hv hmap
var bv bmap
h := &hv
b := &bv
h.buckets = b
h.hash0 = fashtrand0()
```

除了上述特定的优化之外，无论 `make` 是从哪里来的，只要我们使用 `make` 创建哈希，Go 语言编译器都会在类型检查期间将它们转换成 `runtime.makemap`，使用字面量初始化哈希也只是语言提供的辅助工具，最后调用的都是 `runtime.makemap`





### map的结构

`runtime.hmap` 是最核心的结构体

```go
type hmap struct {
	count     int      //哈希表中元素数量
	flags     uint8
	B         uint8    //buckets 数量，对数表示，buckets = 2^B，桶的数量都是2的幂
	noverflow uint16
	hash0     uint32   //哈希种子，引入随机性

	buckets    unsafe.Pointer
	oldbuckets unsafe.Pointer   //扩容时，保存之前的buckets
	nevacuate  uintptr

	extra *mapextra
}
```



## 4. 结构体

```go
type Employee struct {
    ID        int
    Name,Address      string
    DoB       time.Time
    Position  string
    Salary    int
    ManagerID int
}

var dilbert Employee    // 创建一个结构体并初始化所有的成员为对应的零值！！！  结构体初始值并不是nil
```

所以**不存在没有初始化的结构体变量**

下面的EmployeeByID函数将根据给定的员工ID返回对应的员工信息结构体的指针。我们可以使用点操作符来访问它里面的成员：

```Go
func EmployeeByID(id int) *Employee { /* ... */ }

fmt.Println(EmployeeByID(dilbert.ManagerID).Position) // "Pointy-haired boss"

id := dilbert.ID
EmployeeByID(id).Salary = 0 // fired for... no real reason
```

如果将EmployeeByID函数的返回值从`*Employee`指针类型改为Employee值类型，那么更新语句将不能编译通过，因为在赋值语句的左边并不确定是一个变量（调用函数返回的是值，并不是一个可取地址的变量）

**一个命名为S的结构体类型将不能再包含S类型的成员：因为一个聚合的值不能包含它自身**。但是S类型的结构体**可以包含`*S`指针类型**的成员，这可以让我们创建递归的数据结构，比如链表和树结构等

- 可以使用空的struct和map来模拟set。struct{}表示空的struct，它的大小为0，也不包含任何信息

```go
seen := make(map[string]struct{}) // set of strings
```

### 结构体字面值

两种方式指定字面值

```go
type T struct{ a, b int }
var _ = p.T{a: 1, b: 2}
var _ = p.T{1, 2}  
```



因为结构体通常通过指针处理，可以用下面的写法来创建并初始化一个结构体变量，并返回结构体的地址：

```Go
pp := &Point{1, 2}
```

它和下面的语句是等价的

```Go
pp := new(Point)
*pp = Point{1, 2}
```

### 匿名成员

只声明一个成员对应的数据类型而不指名成员的名字，这类成员就叫匿名成员。得益于匿名嵌入的特性，我们**可以直接访问叶子属性而不需要给出完整的路径**

```go
type Circle struct {
    Point
    Radius int
}

type Wheel struct {
    Circle
    Spokes int
}

var w Wheel
w.X = 8            // equivalent to w.Circle.Point.X = 8
w.Y = 8            // equivalent to w.Circle.Point.Y = 8
w.Radius = 5       // equivalent to w.Circle.Radius = 5
w.Spokes = 20
```

但是，**结构体字面值并没有简短表示匿名成员的语法**， 因此下面的语句都不能编译通过：

```Go
w = Wheel{8, 8, 5, 20}                       // compile error: unknown fields
w = Wheel{X: 8, Y: 8, Radius: 5, Spokes: 20} // compile error: unknown fields
```

可以使用下面两种写法：

```go
w = Wheel{Circle{Point{8, 8}, 5}, 20}

w = Wheel{
    Circle: Circle{
        Point:  Point{X: 8, Y: 8},
        Radius: 5,
    },
    Spokes: 20, // NOTE: trailing comma necessary here (and at Radius)
}
```

匿名成员并不要求是结构体类型，命名类型也可以作为匿名成员。匿名类型的方法集。。.运算不仅可以获得匿名成员的嵌套成员，也可以获得它们的方法



**当匿名嵌入是非公开的时，无法直接通过结构体字面量的方式初始化匿名的内部类型，不过由于类型提升，依然可以设置内部类型的公开成员的值（内部类型的标识符提升到了外部类型）**

```go
package important

type a struct {
	aa int
	AA int
}

type B struct {
	a
	bb int
	BB int
}



package main

import (
	"fmt"
	"myLearnProject/important"
)

func main() {
	b := important.B{
		BB: 1,
		//bb: 1,  //Unexported field 'bb' usage in struct literal
		//AA: 1,  //Unknown field 'AA' in struct literal
	}
	b.AA = 2
	fmt.Printf("%v", b) //{{0 2} 0 1}
}
```



### new和make

new：申请了内存，但是不会将内存初始化，只会将内存置零，返回一个指针。

make：申请了内存，返回已初始化的结构体的零值。





## 5. JSON

```go
type Movie struct {
    Title  string
    Year   int  `json:"released"`      //tag可以使成员变为json后的字段名更改
    Color  bool `json:"color,omitempty"`  //omitempty表示当该成员为空或零值时JSON对象不生成该字段
    Actors []string
}

var movies = []Movie{
    {Title: "Casablanca", Year: 1942, Color: false,
        Actors: []string{"Humphrey Bogart", "Ingrid Bergman"}},
    {Title: "Cool Hand Luke", Year: 1967, Color: true,
        Actors: []string{"Paul Newman"}},
    {Title: "Bullitt", Year: 1968, Color: true,
        Actors: []string{"Steve McQueen", "Jacqueline Bisset"}},
    // ...
}
```

这样的数据非常适合json格式，将一个Go语言中类似movies的**结构体slice转为JSON**的过程叫**编组**（marshaling）。编组通过调用`json.Marshal`函数完成，要输出格式化的json可以使用`json.MarshallIndent`函数

```go
data, err := json.Marshal(movies)
if err != nil {
    log.Fatalf("JSON marshaling failed: %s", err)
}
fmt.Printf("%s\n", data)
/*
[{"Title":"Casablanca","released":1942,"Actors":["Humphrey Bogart","Ingr
id Bergman"]},{"Title":"Cool Hand Luke","released":1967,"color":true,"Ac
tors":["Paul Newman"]},{"Title":"Bullitt","released":1968,"color":true,"
Actors":["Steve McQueen","Jacqueline Bisset"]}]
*/
data, err := json.MarshalIndent(movies, "", "    ")  //格式化的json
```

使用`json.Unmarshal`进行解码，将json解码为go语言的数据类型，可以只解码我们关注的字段

```go
var titles []struct{ Title string }
if err := json.Unmarshal(data, &titles); err != nil {
    log.Fatalf("JSON unmarshaling failed: %s", err)
}
fmt.Println(titles) // "[{Casablanca} {Cool Hand Luke} {Bullitt}]"
```

## 6. 文本和HTML模版





## 7. 基本类型和引用类型

基本类型(内置类型)：数值，字符串，布尔

引用类型：切片、map、channel、接口、函数





# 四、函数



## 1. 多返回值

如果一个函数所有的返回值都有显式的变量名，那么该函数的return语句可以省略操作数。这称之为bare return

```Go
func CountWordsAndImages(url string) (words, images int, err error) {
    resp, err := http.Get(url)
    if err != nil {
        return
    }
    doc, err := html.Parse(resp.Body)
    resp.Body.Close()
    if err != nil {
        err = fmt.Errorf("parsing HTML: %s", err)
        return
    }
    words, images = countWordsAndImages(doc)
    return
}
```

每一个return语句等价于：`return words, images, err`

## 2. 错误

### 错误处理策略

1. **错误传播方式**：函数中某个子程序的失败，会变成该函数的失败，即把错误返回给调用者

```go
resp, err := http.Get(url)
if err != nil{
    return nil, err
}
```

或者不直接返回错误，而是构造一个错误信息返回。错误最终由main函数处理时，错误信息应提供清晰的从原因到后果的因果链

```Go
doc, err := html.Parse(resp.Body)
resp.Body.Close()
if err != nil {
    return nil, fmt.Errorf("parsing %s as HTML: %v", url,err)
}
```



2. **重新尝试**：如果错误的发生是偶然性的，或由不可预知的问题导致的。一个明智的选择是重新尝试失败的操作。在重试时，我们需要限制重试的时间间隔或重试的次数，防止无限制的重试

```go
func WaitForServer(url string) error {
    const timeout = 1 * time.Minute
    deadline := time.Now().Add(timeout)
    for tries := 0; time.Now().Before(deadline); tries++ {
        _, err := http.Head(url)
        if err == nil {
            return nil // success
        }
        log.Printf("server not responding (%s);retrying…", err)
        time.Sleep(time.Second << uint(tries)) // exponential back-off
    }
    return fmt.Errorf("server %s failed to respond after %s", url, timeout)
}
```



3. 如果错误发生后程序无法继续执行，就要采用第三种策略**输出错误信息，结束程序**，这种策略只应该在main函数中执行，库函数只应该向上传播，除非该错误意味着程序内部包含不一致性，即遇到了bug，才能在库函数中结束程序

```go
if err := WaitForServer(url); err != nil {
    fmt.Fprintf(os.Stderr, "Site is down: %v\n", err)
    os.Exit(1)
}

//或者采用log.Fatalf，也是一样的效果
if err := WaitForServer(url); err != nil {
    log.Fatalf("Site is down: %v\n", err)
}
```



4. 只输出错误信息，不中断程序

```go
if err := Ping(); err != nil {
    log.Printf("ping failed: %v; networking disabled",err)
}
```



5. 直接忽略掉错误





### 文件结尾错误EOF

io包保证任何由文件结束引起的读取失败都返回同一个错误——io.EOF

可以以此来判断文件是否读取完毕

```go
in := bufio.NewReader(os.Stdin)
for {
    r, _, err := in.ReadRune()
    if err == io.EOF {
        break // finished reading
    }
    if err != nil {
        return fmt.Errorf("read failed:%v", err)
    }
    // ...use r…
}
```

## 3. 匿名函数和闭包

当匿名函数需要被递归调用时，我们必须首先声明一个变量，再将匿名函数赋值给这个变量。如果不分成两部，函数字面量无法与visitAll绑定，我们也无法递归调用该匿名函数

```go
visitAll := func(items []string) {
    // ...
    visitAll(m[item]) // compile error: undefined: visitAll
    // ...
}
```

应该按下面的格式：

```go
var visitAll func(items []string)
visitAll = func(items []string) {
    // ...
    visitAll(m[item]) // compile error: undefined: visitAll
    // ...
}
```



### 闭包


匿名函数中捕获外部函数的局部变量，这种匿名函数一般称为`闭包`。**闭包对捕获的外部变量并不是以值传递的方式访问，而是以引用传递的方式访问**

```go
func Inc() (i int) {    //最后返回43
  defer func() {i++}
  return 42
}
```

闭包可能带来一些隐含问题：因为是闭包，所以是加上defer执行时都是引用的同一个变量i，结束时i是3

```go
func main() {
  for i:=0; i<3; i++ {
    defer func(){ fmt.Print(i) }()
  }
}
// 333
```

循环变量作用域问题：在 for 循环引进的一个块作用域内进行声明。在循环里创建的所有函数变量共享相同的变量，就是一个可访问的存储位置，而不是固定的值

```go
var slice []func()

func main() {
    sli := []int{1, 2, 3, 4, 5}
    for _, v := range sli {
        fmt.Println(&v)
        slice = append(slice, func(){
            fmt.Println(v) // 直接打印结果
        })
    }

    for _, val  := range slice {
        val()
    }
}
// 输出 25 25 25 25 25
```



解决办法：可以**作为参数传递进匿名函数**，而不是直接使用




## 4. 可变参数

```go
func sum(vals...int) int {
    total := 0
    for _, val := range vals {
        total += val
    }
    return total
}
```

vals被看作是[]int的切片，但是实际上并不是切片，和以切片作为参数还是不同的



如果切片作为参数传递的时候，需要**对切片进行结构**，即传入切片的...

**可变参数必须是可变参数函数的最后一个参数**



## 5. defer延迟调用机制

对于函数或方法前加了difer关键字的，当执行到该条语句时，**函数和参数表达式得到计算**，但**直到包含该defer语句的函数执行完毕时，defer后的函数才会被执行**，不论包含defer语句的函数是通过return正常结束，还是由于panic导致的异常结束。你可以在一个函数中执行多条defer语句，它们的**执行顺序与声明顺序相反**。注意**在defer之后执行完毕**才会执行difer语句

```go
//处理文件读写
func ReadFile(filename string) ([]byte, error) {
    f, err := os.Open(filename)
    if err != nil {
        return nil, err
    }
    defer f.Close()
    return ReadAll(f)
}
```

```Go
//处理互斥锁
var mu sync.Mutex
var m = make(map[string]int)
func lookup(key string) int {
    mu.Lock()
    defer mu.Unlock()
    return m[key]
}
```



### 多重defer

注意：

```go
//注意defer加载和传值的时间

func main() {
	a := 10
	defer func() {            
		fmt.Println(a)
	}()

	defer func(a int) {			// 此时，将a=10传递给了func的函数参数，只是函数被延迟执行
		fmt.Println(a)      
	}(a)

	a = 100
}

/*
10
100
*/
```



### 使用defer记录函数执行时间

**记录函数的执行时间：** bigSlowOperation被调时，**trace会返回一个匿名函数func()**，该**函数值会在bigSlowOperation退出时被调用**。通过这种方式， 我们可以只通过一条语句控制函数的入口和所有的出口，甚至可以记录函数的运行时间，如例子中的start。需要注意一点：**不要忘记defer语句后的圆括号**，否则本该在进入时执行的操作会在退出时执行，而本该在退出时执行的，永远不会被执行

```go
func bigSlowOperation() {
    defer trace("bigSlowOperation")() // don't forget the extra parentheses
    // ...lots of work…
    time.Sleep(10 * time.Second) // simulate slow operation by sleeping
}
func trace(msg string) func() {
    start := time.Now()
    log.Printf("enter %s", msg)
    return func() { 
        log.Printf("exit %s (%s)", msg,time.Since(start)) 
    }
}
```

defer在return之后执行，但是可以修改return返回给调用者的值



### defer的小坑-循环

循环体中的defer语句会被延迟到循环执行完毕之后才会执行，解决方法是把循环体中的defer放到另一个函数中，每次循环调用该函数

```go
for _, filename := range filenames {
    if err := doFile(filename); err != nil {
        return err
    }
}
func doFile(filename string) error {
    f, err := os.Open(filename)
    if err != nil {
        return err
    }
    defer f.Close()
    // ...process f…
}
```



### defer的小坑-返回值

```go
func hello1(i *int) (j int) {
	defer func() {
		j = 19
	}()
	j = *i
	return j
}

func hello2(i *int) (j int) {
	defer func() {
		j = 19
	}()
	return *i
}

func hello3(i *int) int {
	defer func() {
		*i = 19
	}()
	return *i
}

func main() {
	i := 10
	j := hello1(&i)
	fmt.Println(i, j)
	i = 10
	j = hello2(&i)
	fmt.Println(i, j)
	i = 10
	j = hello3(&i)
	fmt.Println(i, j)
}
```

输出：

```go
10 19
10 19
19 10
```

总结：**命名返回值，就好比函数参数一样，函数体内对命名返回值的任何修改，都会影响它。而非命名返回值，取决于 return 时候的值**，之后defer的修改不能影响到返回值了



## 6. error 接口

```go
type error interface {
    Error() string
}
```

创建error最简单的方法就是调用errors.New函数，返回一个新的error

```go
err = errors.New("错误信息")
```

**errors包：**

```go
package errors

func New(text string) error { return &errorString{text} }  //返回error对象的地址，避免两个error相等
type errorString struct { text string }		//errorString实现了error接口的全部方法
func (e *errorString) Error() string { return e.text }  //*errorString实现了error接口的Error()方法
```

使用errors.New函数的情况还是比较少

我们一般使用一个更方便的封装函数：**fmt.Errorf**，他还会处理字符串的格式化

```go
func Errorf(format string, args ...interface{}) error {
  return errors.New(Sprintf(format, args...))   //Errorf函数实际上还是调用的errors.New()
}
```



返回错误，调用者自行捕获错误信息



**error返回的是一般性错误，可以进行处理，继续运行。。。panic一般是让程序不能继续运行的错误**

## 7. Panic异常

- Go的类型系统会在编译时捕获很多错误，但有些错误只能在运行时检查，如`数组访问越界、空指针引用`等。这些运行时错误会引起painc异常

一般而言，当panic异常发生时，程序会**中断运行**，并立即执行在该goroutine中**被延迟的函数**（defer 机制）。随后，程序**崩溃并输出日志信息**。日志信息包括panic value和函数调用的堆栈跟踪信息

- **直接调用内置的panic函数也会引发panic异常**，某些不该发生的场景发生时，我们可以主动调用panic表示路径不可达

## 8. recover捕获异常

panic异常一旦发生就会导致程序崩溃。。

go语言为我们提供了专门拦截运行时panic的内建函数 `recover`，它可以使当前程序从运行时panic的状态中恢复，并重新获得流程控制权...

**recover只有在defer调用的函数中有效**，defer直接调用recover不行，**必须包装在另一个函数中**

**包含recover的defer必须在可能出现错误的语句之前声明**

`func recover() interface{}`



```go
func errGenFunc(i int){
    arr := [10]int{}
    
    defer func(){  //defer中通过匿名函数调用recover进行错误拦截
        recover()  //可以从panic异常中重新获取控制权
    }()
    
    fmt.Println(arr[i])  //可能越界，引发panic异常，使程序停止
}

func post(){fmt.Println("post....")}

func main(){
    errGenFunc(100)
    post()
}

/*
post...    //从panic中恢复，正常执行后续程序
*/
```



Go语言库的习惯：即使包内部使用了panic，在导出函数时也会被转化为明确的错误值

```go
//更好的处理方式应该是：
func Xxxx() {
  defer func(){
    if err := recover(); err! = nil{
      fmt.Errorf("Xxx: internal error: %v", err)   //处理错误信息
    }
	}()
  
  .....
} 

```







# 五、方法

在函数定义时，函数名前加上一个附加参数，就变成了方法，表示为这种类型定义了一种独占的方法

```go
type Point struct{ X, Y float64 }

// traditional function
func Distance(p, q Point) float64 {
    return math.Hypot(q.X-p.X, q.Y-p.Y)
}

// same thing, but as a method of the Point type
func (p Point) Distance(q Point) float64 {
    return math.Hypot(q.X-p.X, q.Y-p.Y)
}
```

**方法必须定义在与该类型的同一个包下，所以我们不能给int这种内置类型定义方法**

和函数一样，**不支持重载**

## 1. 方法的指针接收者和值接收者

1. **不管是指针接收者还是值接收者实现了一个接口的方法，都可以通过该类型的值和指针调用接口的方法**，go语言会进行自动的类型转换
2. **如果使用指针接收者来实现一个接口，那么只有该类型的指针才能实现对应接口（is interface a）。**
3. **如果使用值接收者来实现一个接口，那么该类型的值和指针都实现了对应接口。**

也就是说：值类型的方法集只包括使用值接收者实现的方法。而指针类型的方法集包含值和指针接收者实现的方法

因为并不是总能获取一个值的地址

| Values | Methods Receivers |
| :----: | :---------------: |
|   T    |       (t T)       |
|   *T   | (t T) and (t *T)  |



例如，使用值接收者实现接口：

```go
type buyer interface {
	buy()
}

type student struct {
	name string
}

func (s student) buy() {
	fmt.Println(s.name, "买买买")
}

func showBuyer(b buyer) {
	fmt.Println(b)
}

func main() {
	s := student{"小明"}
  
  //使用值接收者实现接口，该类型的值和指针都可以调用接口方法
	s.buy()       
	(&s).buy()
	
  //使用值接收者实现接口，该类型的值和指针都实现了该接口
	showBuyer(s)
	showBuyer(&s)
}
```

使用指针接收者实现接口：
```go
type buyer2 interface {
	buy()
}

type student2 struct {
	name string
}

func (s *student2) buy() {
	fmt.Println(s.name, "买买买")
}

func showBuyer2(b buyer2) {
	fmt.Println(b)
}

func main() {
	s := student2{"小明"}
  
  //使用指针接收者实现接口，该类型的值和指针都可以调用接口方法
	(&s).buy()
	s.buy()
	
  //使用指针接口者实现接口，只有该类型的指针才实现了接口
  //所以不能把该类型的值作为 buyer2 接口类型进行传递，它不是该接口类型
	showBuyer2(s) //Cannot use 's' (type student2) as type buyer2Type does not implement 'buyer2' as 'buy' method has a pointer receiver
	showBuyer2(&s)
}

```



## 2. 方法值和方法表达式

```go
func (p point) distance(q point) float64 {...}
func (cp coleredPoint) distance(cp2 coleredPoint) float64 {...}
func (cp coleredPoint) show() {...}
func (c color) distance(c2 color) float64 {...}

func main(){
  	cp := coleredPoint{point{1, 1},"red"}
	cp2 := coleredPoint{point{2, 2}, "green"}

	cpdistance := coleredPoint.distance
	pdistance := cp.point.distance
	cdistance := cp.color.distance

	fmt.Println(cpdistance(cp, cp2)) //func (cp coleredPoint) distance(cp2 coleredPoint) float64{...}

	fmt.Println(pdistance(cp2.point))
	fmt.Println(cdistance(cp2.color))
}
```



## 3. go语言可以方法重写，不能重载

相互嵌套的结构体可以定义相同的方法名，根据自己的类型调用对应的方法

但是同一个结构体不能定义同名的方法（go语言没有重载机制）



## 4. 方法操作的是副本

其实就是方法的第一个参数是结构体的对象或者对象的指针

所以就是值传递的

**要修改对象的属性，方法的接收者应该是指针** （只需要接收者是指针即可，不需要调用者是指针，就算调用者是对象，也会进行隐式转换，转换为指针）

```go
func main() {
	p := person{
		name: "xiaoming",
		age:  10,
	}
	p.addAge()
	fmt.Printf("%v\n", p)   //{xiaoming 11}
}

func (p *person) addAge() {
	p.age = p.age + 1
}
```





# 六、接口



## 1. 接口基本介绍

Printf和Sprintf底层都是调用了Fprintf：

Printf就是打印到os.Stdout标准输出上

Sprintf就是打印到一个bytes.Buffer，然后返回了buf的string

```go
package fmt

func Fprintf(w io.Writer, format string, args ...interface{}) (int, error)
func Printf(format string, args ...interface{}) (int, error) {
    return Fprintf(os.Stdout, format, args...)
}
func Sprintf(format string, args ...interface{}) string {
    var buf bytes.Buffer
    Fprintf(&buf, format, args...)
    return buf.String()
}
```

**空接口**：interface{}类型，它没有任何方法，空接口类型对实现它的类型没有要求，所以我们可以将任意一个值赋给空接口类型

```go
var any interface{}
any = true
any = 12.34
any = "hello"
any = map[string]int{"one": 1}
any = new(bytes.Buffer)
```



### 接口检查

```go
//1. 空值nil强转为 *GobCodec 类型，再转换(赋值)为Codec接口，IDE就可以帮我们验证GobCodec是否实现了Codec接口
var _ Codec = (*GobCodec)(nil) 

//2. 效果差不多
var _ Codec = new(GobCodec)    
```



## 2. sort接口

`sort.Interface` 接口定义了三个方法，自定义排序规则必须实现这三个接口

```go
package sort
type Interface interface{
  Len() int    //求待排序元素个数的方法
  Less(i, j int) bool     //比较 i j 位置元素大小的方法
  Swap(i, j int)  //交换 i j 位置元素的方法
}
```

对于 int 、 float64 和 string 数组或是分片的排序， go 分别提供了 sort.Ints() 、 sort.Float64s() 和 sort.Strings() 函数， 默认都是从小到大排
go 的 sort 包可以使用 sort.Reverse() 来调换 slice.Interface.Less，交换后调用sort.Sort函数进行排序

```go
intList := [] int {2, 4, 3, 5, 7, 6, 9, 8, 1, 0}
sort.Ints(intList)    //顺序排序
sort.Sort(sort.Reverse(sort.IntSlice(a)))   //逆序排序
```



### 便捷的排序结构

`sort.StringSlice`结构类型可以便捷的对[]string进行排序，`sort.IntSlice`可以便捷的对[]int进行排序（实际上还是[]string和[]int，只是帮我们定义好了自定义排序的三个方法），**类型转换为sort.Interface了，所以支持逆序排序**

也可以直接使用`sort.Strings(a []string)`和`sort.Ints(a []int)`来进行排序，但是类型没有变为sort.Interface，因此**不支持使用 sort.Reverse 进行逆序**

```go
	nums := []int{1, 8, 5, 5, 3, 8}
	s := sort.IntSlice(nums)   //转换为 sort.IntSlice 类型进行排序
	sort.Sort(sort.Reverse(s))
	fmt.Println(s)   // [8 8 5 5 3 1]

	strs := []string{"acd", "ab", "d", "b"}
	sort.Strings(strs)  //使用sort.Strings()函数进行排序
	fmt.Println(strs) // [ab acd b d]
```



### Slice的自定义排序

 sort包提供了三个slice的排序函数 `Slice`, `SliceStable`, `SliceIsSorted`

```go
package sort

func Slice(slice interface{}, less func(i, j int) bool) {
	rv := reflectValueOf(slice)
	swap := reflectSwapper(slice)
	length := rv.Len()
	quickSort_func(lessSwap{less, swap}, 0, length, maxDepth(length))
}

// 稳定排序
func SliceStable(slice interface{}, less func(i, j int) bool) {
	rv := reflectValueOf(slice)
	swap := reflectSwapper(slice)
	stable_func(lessSwap{less, swap}, rv.Len())
}

// 判断是否排序
func SliceIsSorted(slice interface{}, less func(i, j int) bool) bool {
	rv := reflectValueOf(slice)
	n := rv.Len()
	for i := n - 1; i > 0; i-- {
		if less(i, i-1) {
			return false
		}
	}
	return true
}
```

举例： leetcode179，求数组元素拼接组合后的最大数

```go
func largestNumber(nums []int) string {
	strs := make([]string, len(nums))
	for i:=0; i<len(nums); i++{
		strs[i] = strconv.Itoa(nums[i])
	}
	sort.Slice(strs, func(i, j int) bool {
		return strs[i]+strs[j] > strs[j]+strs[i]
	})
	if strs[0] == "0" {
		return "0"
	}
	return strings.Join(strs, "")
}
```





### 结构体自定义排序

结构体切片只要实现了 sort.Interface接口（定义Len，Swap，Less方法），就可以按照自定义的规则进行排序

```go
type Person struct {
    Name string
    Age  int
}

// 按照 Person.Age 从大到小排序
type PersonSlice [] Person

// PersonSlice实现了sort接口的三个方法，所以可以当作sort.Interface来使用
func (a PersonSlice) Len() int {return len(a)}
func (a PersonSlice) Swap(i, j int){a[i], a[j] = a[j], a[i]}
func (a PersonSlice) Less(i, j int) bool {return a[i].Age < a[j].Age}
 
func main() {
    people := [] Person{
        {"zhang san", 12},
        {"li si", 30},
        {"wang wu", 52},
        {"zhao liu", 26},
    }
 
  	//等价于：sort.Sort(sort.Interface(PersonSlice(people)))
    sort.Sort(PersonSlice(people))    // 按照 Age 升序排序
    fmt.Println(people)
 
    sort.Sort(sort.Reverse(PersonSlice(people)))    // 按照 Age 降序排序
    fmt.Println(people)
 
}

// 还可以把 Less 函数和 PersonSlice 封装到 PersonWrapper，使得调用者可以自己传入排序的 Less 方法
```



## 3. http.Handler接口

```go
package http

type Handler interface {
    ServeHTTP(w ResponseWriter, r *Request)
}

func ListenAndServe(address string, h Handler) error
```

**ListenAndServe函数**需要一个例如“localhost:8000”的服务器地址，和一个所有请求都可以分派的Handler接口实例。它会一直运行，直到这个服务因为一个错误而失败（或者启动失败），它的返回值一定是一个非空的错误

```go
func main() {
    db := database{"shoes": 50, "socks": 5}
    log.Fatal(http.ListenAndServe("localhost:8000", db))
}

type dollars float32
func (d dollars) String() string { return fmt.Sprintf("$%.2f", d) }

type database map[string]dollars
func (db database) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    for item, price := range db {
        fmt.Fprintf(w, "%s: %s\n", item, price)
    }
}
```

**ServeMux多路请求器**，位于`net/http`包，可以简化URL和handlers的URL

```go
func main() {
    db := database{"shoes": 50, "socks": 5}
    mux := http.NewServeMux()
    mux.Handle("/list", http.HandlerFunc(db.list))   //将database.list函数转化为http.HandlerFunc类型
    mux.Handle("/price", http.HandlerFunc(db.price))
    log.Fatal(http.ListenAndServe("localhost:8000", mux))
}

type database map[string]dollars

func (db database) list(w http.ResponseWriter, req *http.Request) {
    for item, price := range db {
        fmt.Fprintf(w, "%s: %s\n", item, price)
    }
}

func (db database) price(w http.ResponseWriter, req *http.Request) {
    item := req.URL.Query().Get("item")
    price, ok := db[item]
    if !ok {
        w.WriteHeader(http.StatusNotFound) // 404
        fmt.Fprintf(w, "no such item: %q\n", item)
        return
    }
    fmt.Fprintf(w, "%s\n", price)
}
```

`http.HandlerFunc`接口定义了ServlHttp方法，所有func(w ResponseWriter, r *Request)都可以转化为HandlerFunc

ServeHTTP方法的行为是**调用了它的函数本身**。因此HandlerFunc是一个**让函数值满足一个接口的适配器**，这里函数和这个接口仅有的方法有相同的函数签名。实际上，这个技巧让一个单一的类型例如database以多种方式满足http.Handler接口：一种通过它的list方法，一种通过它的price方法等等

```go
package http

type HandlerFunc func(w ResponseWriter, r *Request)

func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
    f(w, r)
}
```

为了方便，net/http包提供了一个**全局的ServeMux实例DefaultServerMux**和**包级别的http.Handle和http.HandleFunc函数**。现在，为了使用DefaultServeMux作为服务器的主handler，我们不需要将它传给ListenAndServe函数；nil值就可以工作。

然后服务器的主函数可以简化成：

```go
func main() {
    db := database{"shoes": 50, "socks": 5}
    http.HandleFunc("/list", db.list)
    http.HandleFunc("/price", db.price)
    log.Fatal(http.ListenAndServe("localhost:8000", nil))
}
```



## 4. 类型断言

**x.(T)被称为断言类型**，这里x表示一个接口的类型和T表示一个类型。一个类型断言检查它操作对象的动态类型是否和断言的类型匹配

1. **如果断言的T是一个具体类型，会检查x的动态类型是否和T相同，断言成功，返回x的动态值，断言失败则抛出panic异常**

```go
var w io.Writer
w = os.Stdout
f := w.(*os.File)      // success: f == os.Stdout
c := w.(*bytes.Buffer) // panic: interface holds *os.File, not *bytes.Buffer
```

2. **如果断言的T是一个接口类型，会检查x的动态类型是否满足T接口，断言成功则会把x的类型转化为T，断言失败则会panic异常，如果T是nil接口值，那么不管x都什么类型都会失败**

```go
var w io.Writer
w = os.Stdout
rw := w.(io.ReadWriter) // success: *os.File has both Read and Write
w = new(ByteCounter)
rw = w.(io.ReadWriter) // panic: *ByteCounter has no Read method
w = rw             // io.ReadWriter is assignable to io.Writer
w = rw.(io.Writer) // fails only if rw == nil
```



对于接口值的动态类型经常是不确定的，我们会去检验它是否是一些特定的类型，这时如果使用两个参数去接受断言的结果，便会得到成功与否的标志

```go
var w io.Writer = os.Stdout
if f, ok := w.(*os.File); ok {
    // ...use f...
}
```



## 5. -----基于类型断言区别错误类型

os包中文件操作返回的错误集合：

```go
package os

func IsExist(err error) bool      //创建文件，文件已存在
func IsNotExist(err error) bool   //读取操作，文件不存在
func IsPermission(err error) bool //权限拒绝
```

可以使用一个专门的类型来描述结构化的错误

```go
package os

// PathError records an error and the operation and file path that caused it.
type PathError struct {
    Op   string
    Path string
    Err  error
}

func (e *PathError) Error() string {
    return e.Op + " " + e.Path + ": " + e.Err.Error()
}

var ErrNotExist = errors.New("file does not exist")
func IsNotExist(err error) bool {
    if pe, ok := err.(*PathError); ok {
        err = pe.Err
    }
    return err == syscall.ENOENT || err == ErrNotExist
}
```



## 6. -----通过类型断言查询接口



## 7. 内部类型和外部类型的接口问题

由于**类型提升**，如果内部类型实现了某个接口，那么内部类型实现的接口会自动**提升到外部类型**，即**外部类型也同样实现了这个接口**

如果内部类型是指针接收者实现的接口，那么对应外部类型的指针才能实现接口



如果内部类型和外部类型实现了同一个接口，那么内部类型实现的接口不会被自动提升，通过外部类型访问外部类型实现的方法，内部类型访问内部类型实现的方法





## 8. 函数、方法、接口总结

函数分为`具名函数`和`匿名函数`

当匿名函数引用了外部作用域中的变量时就成了`闭包函数`，闭包是函数式变成语言的核心

`方法`是绑定到具体类型的特殊函数，必须在编译时`静态绑定`

`接口`定义了方法的集合，这些方法依托于运行时的接口对象，所以是在运行时`动态绑定`的





# 七、Goroutines和Channels

Go语言中的并发程序可以用两种手段来实现：

1. 本章的goroutine和channel，其支持“顺序通信进程”（communicating sequential processes）或被简称为CSP。CSP是一种现代的并发编程模型，在这种编程模型中值会在不同的运行实例（goroutine）中传递，尽管大多数情况下仍然是被限制在单一实例中。
2. 覆盖更为传统的并发模型：多线程共享内存



## 1. Goroutines-并发

Go语言中，每一个并发的执行单元叫做一个goroutine

当一个程序启动的时候，其主函数即在一个单独的goroutine中运行（main goroutine），使用go 语句创建一个新的goroutine.

**goroutine并不是线程，而是对线程的多路复用**

一个goroutine启动时的栈大小仅为2KB，而一个OS线程的栈大小一般是2MB，这个栈会用来存储当前正在被调用或挂起（指在调用其它函数时）的函数的内部变量

当一个goroutine被阻塞时，它**也会阻塞所复用的OS线程**，runtime会把位于被阻塞线程上的**其他goroutine移动到其他未被阻塞的线程上**继续运行



**修改逻辑处理器的数量**

```go
runtime.GOMAXPROCS(1)      //分配一个逻辑处理器给调度器使用
```



```go
runtime.GOMAXPROCS(runtime.NumCPU())   //分配逻辑处理器的数量=cpu核心数，每个cpu核心都分配一个逻辑处理器
```

### 动态栈

固定栈大小的缺陷：太小的栈空间虽然提高了空间利用率，但是递归调用容易栈溢出。太大的栈空间对那些只需要很小栈空间的线程来说是一个巨大的空间浪费

例如Linux中，栈大小默认为8MB，运行内存超过上限时程序就会崩溃，为了解决这个问题，可以调大内核参数中的 stack size，或者在创建线程时显示的传入所需要大小的内存块。前者会影响系统内所有线程，后者又需要精确计算thread大小，都有缺陷

我们可以在函数调用处插桩， 每次调用的时候检查当前栈的空间是否能够满足新函数的执行，满足的话直接执行，否则**创建新的栈空间并将老的栈拷贝到新的栈然后再执行**。但是Linux thread模型不能满足这个条件，实现的话只能在用户空间实现，go语言的runtime就实现了这个特性

goroutine的栈大小可以根据需要动态调整，一个goroutine会以一个很小的栈开始其生命周期，**一般只需要2KB**。一个goroutine的栈，和操作系统线程一样，会保存其活跃或挂起的函数调用的**本地变量**，但是和OS线程不太一样的是，一个goroutine的**栈大小并不是固定的**；栈的大小会根据需要动态地伸缩，**goroutine的栈的最大值有1GB**

由于栈会变化，所以会将之前的数据移动到新的内存空间，所以**指针不再是固定不变的**

此外，一些long running的goroutine可能由于某次函数调用中引发了栈的扩容， 被调用函数返回后很大部分空间都未被利用，为了解决这样的问题，需要能够对栈进行**收缩**，以节约内存提高利用率

### goroutines和线程

每一个**OS线程**都有一个**固定大小的内存**块（一般会是2MB）来做栈，这个栈会用来存储当前正在被调用或挂起（指在调用其它函数时）的函数的内部变量。



## 2. Channels-通信

使用make语句创建无buffer的channel：`ch := make(chan int)`或者`ch := make(chan int, 0)`

还可以指定buffer大小：`ch := make(chan int, 3)`，带buffer的channel

channel对应一个make创建的底层数据结构的引用，所以复制或者函数参数传递的时候，拷贝了channel的引用，因此**调用者和被调用者引用了同一个channel对象**，如果两个channel引用的是同一个对象，那么他们==比较的结果为真



**发送和接收：**

```go
ch <- x   //发送到channel，<-写在channel之后
x = <- ch //从channel接收，<-写在channel之前
```

**关闭channel：** 使用`close(ch)`可以关闭channel，随后基于该channel的任何发送操作都会导致panic异常，**但是接收操作依然可以进行**



### 无buffer的channel

无buffer的channel的**发送操作**将会导致**发送者goroutine阻塞**，直到另一个goroutine在相同的channel上执行接收操作

同理，接受操作先发生，也会阻塞，直到有另一个goroutine在相同的channel上执行发送操作

所以无buffer的channel也叫做**同步channel**，发送和接收操作都会导致两个goroutine做一次同步操作



- happens-before：并不是说x在时间上先于y时间发生，而是要保证在此之前的时间都已经完成了
- 如果x既不是在y之前，也不是在y事件之后，我们就说x和y是**并发**的，也不一定同时发生，只是不能确定发生的先后顺序



- **空结构体**`struct{}`类型的`channel`，它不能被写入任何数据，只有通过`close()`函数进行关闭操作，才能进行输出操作。`struct`{}类型的`channel`**不占用任何内存**，用来做同步用

```go
	ch := make(chan struct{})
	times := make([]struct{}, 10)
	for range times {
		go func() {
			handle()
			ch <- struct{}{}
		}()
	}
	// 如果注释掉这段，函数会立即返回，而这段会让函数等待所有的goroutine执行完成，进行一个同步，再ok 返回
	// 而加上之后，主函数慢慢的排空channel，子goroutine慢慢的进行写入
	//for range times {
	//	<-ch
	//}
	fmt.Println("ok")
```





### 串联的channels(Pipeline)

多个channel串联起来就形成了pipeline

如果发送者知道没有更多的值需要发送到channel了，就可以让接收着也知道没有更多的值可以接收了，这样接收者可以停止不必要的接收等待，这可以通过内置的close函数来关闭channel以实现

但是 **当一个被关闭的channle中已经发送的数据都被接收成功后**，后续的接收操作将不会再阻塞，它们**会立即返回0值**，会收到无休止的0序列，解决办法是接收者多接收一个ok参数，自己进行判断，例如：

```go
	for {
		x, ok := <-squares
		if !ok {
			break
		}
		fmt.Println(x)
	}
```

但是这种方法比较繁琐，更简洁的方法可以直接使用range循环在channel上迭代。**range循环在channel被关闭且没有值可接收时会自动跳出循环**

```go
func counter(out chan<- int) {
    for x := 0; x < 100; x++ {
        out <- x
    }
    close(out)
}

func squarer(out chan<- int, in <-chan int) {
    for v := range in {
        out <- v * v
    }
    close(out)
}

func printer(in <-chan int) {
    for v := range in {
        fmt.Println(v)
    }
}

func main() {
    naturals := make(chan int)
    squares := make(chan int)
    go counter(naturals)
    go squarer(squares, naturals)
    printer(squares)
}
```

**将双向channel赋值给单向channel会进行隐式转换，但是反过来是不可以转换的**



### 带buffer的channel

带buffer的channel内部持有一个元素队列，队列的最大容量在make的时候进行指定

`ch = make(chan string, 3)`

当队列满了的时候，发送操作的goroutine才阻塞，当队列空了的时候，接收操作的goroutine才阻塞

**使用len函数获取channel中有效元素个数，使用cap获得channel的缓存容量**

**带buffer的channel还可以用来管理一组可复用的资源(实现一个pool)**



### 无buffer的channel和buffer大小为1的channel

无buffer的channel必须等待到接收方和发送方同时准备好时才能进行数据传递，否则会阻塞

有buffer的channel当缓冲区满时，发送数据会阻塞，当缓冲区空时，接受数据会阻塞，不需要同时做好准备



**无缓冲通道，如果接收方和发送方在一个程序中，会导致死锁：**容量大小为0，发送方将数据发送到channel后会阻塞自己，直到接收方进行读取为止，反之亦然，一般用于**goroutine同步**

```go
func main() {
	c := make(chan int)
  fmt.Println(len(c))   //0
	c <- 1
	fmt.Println(<-c)    //fatal error: all goroutines are asleep - deadlock!
}
```

**修改为buffer大小为1的channel则不会有这个问题：**容量大小为1，发送方将数据发送到channel后不会阻塞自己(如果buffer容量足够)，一般用于**通信和资源共享**

```go
func main() {
	c := make(chan int, 1)
  fmt.Println(len(c))   //1
	c <- 1
	fmt.Println(<-c) //1
}
```

再看两个例子：

```go
func main() {
    var c = make(chan int) 
    var a string

    go func() {
        a = "hello world" 
        <-c 
    }()

    c <- 0  // 主线程走到这里会堵塞， 直到 c 中的 0 被取走， 所以协程里的 hello world 一定会被输出。
    fmt.Println(a)
}
```

```go
func main() {
    var c = make(chan int, 1) 
    var a string

    go func() {
        a = "hello world" 
        <-c 
    }()

    c <- 0  // 主线程走到这里不会堵塞，也就是 hello world 可能来不及输出，主线程就结束了。
    fmt.Println(a)
}
```



### 基于select的多路复用

select的基本结构：**如果有default则不会阻塞，如过没有default则是阻塞调用，阻塞直到一个通道有事件为止**

```go
select {
case <-ch1:
    // ...
case x := <-ch2:
    // ...use x...
case ch3 <- y:
    // ...
default:
    // ...
}
```



`time.Tick`函数返回一个channel，程序会周期性地像一个节拍器一样向这个channel发送事件。每一个事件的值是一个时间戳

但是如果不是整个程序都需要这个channel，可以使用`time.NewTicker`，通过 `time.NewTicker.C`转化为channel，用完之后使用`ticker.Stop()`将ticker的goroutine结束掉

```go
func main() {
	abort := make(chan struct{})
	go func() {
		os.Stdin.Read(make([]byte, 1)) // read a single byte
		abort <- struct{}{}
	}()

	fmt.Println("Commencing countdown.  Press return to abort.")
	ticker := time.NewTicker(1 * time.Second)
	for countdown := 10; countdown > 0; countdown-- {
		fmt.Println(countdown)
		select {
		case <-ticker.C:
			fmt.Println(countdown)
		case <-abort:
			fmt.Println("Launch aborted!")
			return
		}
	}
	fmt.Println("launch....")
}
```



**在for循环里，即使通道被关闭了，依然有机会被select**，通道读取的结果是通道的零值和false：

```go
const (
	fmtTime = "2006-01-02 15:04:05"
)

func main() {
	c1 := make(chan int, 2)
	c2 := make(chan int, 3)
	var wg sync.WaitGroup
	wg.Add(2)
  
	go func() {
		for i := 0; i < 2; i++ {
			c1 <- i
		}
		close(c1)
		wg.Done()
	}()
	go func() {
		for i := 0; i < 3; i++ {
			c2 <- i
		}
		close(c2)
		wg.Done()
	}()
	wg.Wait()
	
  for i := 0; i < 10; i++ {
		select {
		case x, ok := <-c1:
			fmt.Printf("%s : 从c1读取到 x=%v, ok=%v\n", time.Now().Format(fmtTime), x, ok)
			time.Sleep(500 * time.Millisecond)
		case x, ok := <-c2:
			fmt.Printf("%s : 从c2读取到 x=%v, ok=%v\n", time.Now().Format(fmtTime), x, ok)
			time.Sleep(500 * time.Millisecond)
		default:
			fmt.Printf("%s : 没有读取到信息。。。\n", time.Now().Format(fmtTime))
		}
	}
}

/*
2021-01-25 15:46:46 : 从c2读取到 x=0, ok=true
2021-01-25 15:46:46 : 从c2读取到 x=1, ok=true
2021-01-25 15:46:47 : 从c2读取到 x=2, ok=true
2021-01-25 15:46:47 : 从c1读取到 x=0, ok=true
2021-01-25 15:46:48 : 从c2读取到 x=0, ok=false
2021-01-25 15:46:48 : 从c1读取到 x=1, ok=true
2021-01-25 15:46:49 : 从c2读取到 x=0, ok=false
2021-01-25 15:46:49 : 从c1读取到 x=0, ok=false
2021-01-25 15:46:50 : 从c1读取到 x=0, ok=false
2021-01-25 15:46:50 : 从c1读取到 x=0, ok=false
*/
```



**nil值的channel：** 向nil的channel读或者写都会被永远阻塞，所以select一个nil的channel会永远都select不到，所以可以用来**禁用或者激活case**，例如想要修改上面的for循环，让通道关闭后就不再读取：

```go
	for i := 0; i < 10; i++ {
		select {
		case x, ok := <-c1:
			fmt.Printf("%s : 从c1读取到 x=%v, ok=%v\n", time.Now().Format(fmtTime), x, ok)
			if !ok {
				c1 = nil
			}
			time.Sleep(500 * time.Millisecond)
		case x, ok := <-c2:
			fmt.Printf("%s : 从c2读取到 x=%v, ok=%v\n", time.Now().Format(fmtTime), x, ok)
			if !ok {
				c2 = nil
			}
			time.Sleep(500 * time.Millisecond)
		default:
			fmt.Printf("%s : 没有读取到信息。。。\n", time.Now().Format(fmtTime))
		}
	}

/*
2021-01-25 15:52:06 : 从c2读取到 x=0, ok=true
2021-01-25 15:52:07 : 从c1读取到 x=0, ok=true
2021-01-25 15:52:07 : 从c2读取到 x=1, ok=true
2021-01-25 15:52:08 : 从c2读取到 x=2, ok=true
2021-01-25 15:52:08 : 从c2读取到 x=0, ok=false
2021-01-25 15:52:09 : 从c1读取到 x=1, ok=true
2021-01-25 15:52:09 : 从c1读取到 x=0, ok=false
2021-01-25 15:52:10 : 没有读取到信息。。。
2021-01-25 15:52:10 : 没有读取到信息。。。
2021-01-25 15:52:10 : 没有读取到信息。。。
*/

//如果没有default，则会出现 fatal error: all goroutines are asleep - deadlock!
```



为了避免死锁，我们的for循环条件可以修改为：`for ok1 || ok2 {...}`



### goroutine并发的退出

go语言并没有提供在一个goroutine中终止另一个goroutine的方法，因为这样会导致goroutine之间的共享变量落在为定义的状态上



## 3. goroutine调度

半抢占式：当前goroutine发生阻塞时才会导致调度

发生在用户态：切换代价小





# 八、 基于共享变量的并发

一个提供对一个指定的变量通过channel来请求的goroutine叫做这个变量的monitor（监控）goroutine



## 1. sync.Mutex互斥锁

首先我们可以用缓存大小为1的 channel 来实现互斥锁：

```go
var (
    sema    = make(chan struct{}, 1) // a binary semaphore guarding balance
    balance int
)

func Deposit(amount int) {  //存款
    sema <- struct{}{} // acquire token
    balance = balance + amount
    <-sema // release token
}

func Balance() int {  //查询
    sema <- struct{}{} // acquire token
    b := balance
    <-sema // release token
    return b
}
```



**sync包中的Mutex类型直接支持这个操作：** 通过 `Mutex.Lock()` 和 `Mutex.Unlock()` 函数进行加锁和解锁，获取锁失败的goroutine会被阻塞，直到锁可用

**go里没有重入锁**

```go
var (
    mu      sync.Mutex // guards balance
    balance int
)

func Deposit(amount int) {
    mu.Lock()
	  defer mu.Unlock()
    balance = balance + amount
}

func Balance() int {
    mu.Lock()
 		defer mu.Unlock()
    b := balance
    return b
}
```



## 2. sync.RWMutex读写锁

允许多个读操作并发执行，但是写操作完全互斥，`RLock()`, `RUnlock()`获取和释放共享锁，

```go
var mu sync.RWMutex
var balance int
func Balance() int {
    mu.RLock() // readers lock
    defer mu.RUnlock()
    return balance
}
```



## 3. sync.Once 惰性初始化

```go
var loadIconsOnce sync.Once
var icons map[string]image.Image
// Concurrency-safe.
func Icon(name string) image.Image {
    loadIconsOnce.Do(loadIcons)
    return icons[name]
}
```

保证只初始化一次



## 4. atomic原子操作

```go
atomic.AddInt64(&counter, 1)   //原子地对counter加1
```

强制同一时刻只能有一个goroutine运行并完成这个加法操作



`atomic.LoadInt64` 和 `atomic.StoreInt64` 两个函数提供了安全读写一个整型值的方式。这两者之间会互相同步，保证安全，不会进入竞争状态

```go
var (
   // shutdown is a flag to alert running goroutines to shutdown.
   shutdown int64

   // wg is used to wait for the program to finish.
   wg sync.WaitGroup
)

func main() {
   wg.Add(2)

   go doWork("A")
   go doWork("B")

   // Give the goroutines time to run.
   time.Sleep(1 * time.Second)

   // Safely flag it is time to shutdown.
   fmt.Println("Shutdown Now")
   atomic.StoreInt64(&shutdown, 1)

   // Wait for the goroutines to finish.
   wg.Wait()
}

// doWork simulates a goroutine performing work and
// checking the Shutdown flag to terminate early.
func doWork(name string) {
   defer wg.Done()

   for {
      fmt.Printf("Doing %s Work\n", name)
      time.Sleep(250 * time.Millisecond)

      // Do we need to shutdown.
      if atomic.LoadInt64(&shutdown) == 1 {
         fmt.Printf("Shutting %s Down\n", name)
         break
      }
   }
}
```



### 单例：sync.Once就基于atomic实现

```go
type Once struct {
	done uint32
	m    Mutex
}

func (o *Once) Do(f func()) {
	if atomic.LoadUint32(&o.done) == 0 {
		o.doSlow(f)
	}
}

func (o *Once) doSlow(f func()) {
	o.m.Lock()
	defer o.m.Unlock()
	if o.done == 0 {     // DLC
    //重要：必须要保证f执行完才能进行store，否则其他线程可能获取到没有初始化的对象
		defer atomic.StoreUint32(&o.done, 1)   
		f()
	}
}
```



## 5. 顺序一致性内存模型

```go
var a string
var done bool

func setup() {
  a = "hello"
  done = true
}

func main() {
  go setup()
  for !done {}
  fmt.Println(a)
}
```

并发的事件是无法确定发生顺序的。为了最大化并行，go编译器和处理器会在对语句/指令进行重排序，前提是保证 as-if-serial，保证 happens before 原则



## 6. happens-before

[Golang 并发编程核心—内存可见性](https://juejin.cn/post/6911126210340716558)

happens-before 是比 **指令重排、内存屏障**这些概念更上层的东西，是语言层面给我们的承诺。我们没有办法穷举在所有计算机架构下重排序会如何发生，也没办法为重排序定义该在什么时候插入屏障来阻止重排序、刷新 cache 的顺序，这个是无法做到的事情。

happens-before表示的是**前一个操作的结果对于后续操作是可见的**，它表达了**多个线程之间对于内存的可见性**，本质上是一种偏序关系，所以满足传递性

> 为什么要有happens-before原则：
>
> 如果Java、Go语言提供内存屏障操作，让开发者自己决定何时插入屏障，那么并发代码的开发将很难写，易出错(C就是)
>
> 对于编译器和CPU来说，这些内存屏障，优化屏障都是约束，让底层无法随心所欲进行优化，这些约束肯定越少越好

happens-before像是几条公理，只有我们开发的时候遵守这些规则，才能保证正确的语义，满足正确的可见性

Channel涉及到的happens-before规则有4跳，最多，Golang里面channel才是保证内存可见性、有序性、代码同步的第一选择：

1. `import package` 的时候，如果 package p 里面执行 `import q` ，那么逻辑顺序上 package q 的 init 函数执行先于 package p 后面执行任何其他代码。
2. `goroutine创建`：goroutine的创建函数本身 happens before 于 goroutine 内的第一行代码执行
3. `goroutine销毁`：goroutine 的 exit 事件并不保证先于所有的程序内的其他事件，或者说某个 goroutine exit 的结果并不保证对其他goroutine可见
4. `channel写入`：像channel写入 happens before 于对应元素的读取操作
5. `channel关闭`：channel的关闭 happens before 于channel的接收(<-chan)的返回，也就是说channel一定可以接收到关闭前写入的信息
6. `无缓冲channel`：无缓冲channel的接收操作 happens before 发送操作
7. `有缓冲channel`：缓冲长度为C的channel，第K个元素的接收先于第K+C个元素的发送
8. `锁`：针对任意 `sync.Mutex` 和 `sync.RWMutex` 的锁变量 L，第 n 次调用 `L.Unlock()` 逻辑先于（结果可见于）第 m 次调用 `L.Lock()` 操作 （n<m）。也就是下次加锁必须等待上次的锁释放
9. `读写锁`：针对 `sync.RWMutex` 类型的锁变量 L， `L.Unlock( )` 可见于 `L.Rlock( )` ，第 n 次的 `L.Rlock( )` 先于 第 n+1 次的 `L.Lock()`
10. `once`：`f( )` 函数执行先于 `once.Do(f)` 的返回。换句话说，`f( )` 必定先执行完，`once.Do(f)` 函数调用才能返回







# 九、测试

`go test`命令是一个按照一定的约定和组织来测试代码的程序。在包目录内，所有以`_test.go`为后缀名的源文件在执行go build时不会被构建成包的一部分，它们是go test测试的一部分

`*_test.go`文件中，有三种类型的函数：

- **测试函数**：以`Test`作为函数前缀名(之后必须紧跟**大写字母**)，函数必须接收`*testing.T`，并且不能有返回值，go test会调用这些测试函数，并报告结果为PASS或FAIL

```go
func TestDownload(t *testing.T) {
  ...
}
```



- **基准测试（benchmark）函数**：以Benchmark为函数前缀名，用于衡量一些函数的性能。go test会多次运行基准测试函数，计算平均时间
- **示例函数**

go test命令会遍历所有的`*_test.go`文件中符合上述命名规则的函数，生成一个**临时的main包**用于调用相应的测试函数，接着构建并运行、报告测试结果，最后清理测试中生成的临时文件



## 1. 单元测试

参数`-v`可用于打印**每个测试函数的名字和运行时间**

参数`-run`对应一个正则表达式，只有**测试函数名被它正确匹配的单元测试函数**才会被`go test`测试命令运行

参数`-cover` 展示测试用例对代码的覆盖率



表格驱动的测试案例：

```Go
func TestIsPalindrome(t *testing.T) {
    var tests = []struct {
        input string
        want  bool
    }{
        {"", true},
        {"a", true},
        {"A man, a plan, a canal: Panama", true},
        {"Evil I did dwell; lewd did I live.", true},
        {"Able was I ere I saw Elba", true},
        {"été", true},
        {"Et se resservir, ivresse reste.", true},
    }
    for _, test := range tests {
        if got := IsPalindrome(test.input); got != test.want {
            t.Errorf("IsPalindrome(%q) = %v", test.input, got)
        }
    }
}
```



`testing.T`结构拥有几个非常有用的函数：

- `Log` 将给定文本记录到错误日志，与fmt.Println 类似
- `Logf` 
- `Fail` 将测试函数标记为“已失败”，但允许测试函数继续执行
- `FailNow` 将测试函数标记为“已失败”，并且停止测试函数
- `Error = Log + Fail`，标记失败，继续执行。。。。Errorf = Logf + Fail
- `Fatal = Log + FailNow`，标记失败，立即结束。。。Fatalf = Logf + FailNow



**跳过测试用例**

`t.Skip()` 

```go
func TestEncode(t *testing.T) {
	t.Skip("Skipping encoding for now")
}
```



**并行方式运行测试**

`t.Parallel()`，，同时运行时命令`go test -v -parallel 3` 表示并行方式运行，最多3个任务并行

```go
func TestParallel_1(t *testing.T) {
  t.Parallel()    //三个测试任务并行
  time.Sleep(1 * time.Second)   //模拟需要耗时1秒的任务
}

func TestParallel_2(t *testing.T) {
  t.Parallel()
  time.Sleep(2 * time.Second)   //模拟需要耗时2秒的任务
}

func TestParallel_3(t *testing.T) {
  t.Parallel()
  time.Sleep(3 * time.Second)   //模拟需要耗时3秒的任务
}
```



## 2. 基准测试

基准测试也定义在 xxx_test.go 形式的文件中，基准测试的函数名为`BenchmarkXxxx(b *testing.B)`

测试程序要做的就是将被测试的代码执行 b.N 次，以便检测代码的响应时间，b.N的值是根据被执行代码而改变的，由Go自行决定

运行时的命令`go test -v -bench .` 其中 -bench 传入一个正则表达式用于匹配基准测试函数

```go
func BenchmarkDecode(b *testing.B) {
  for i := 0; i < b.N; i++ {
    decode("post.json") 
  }
}

func BenchmarkUnmarshal(b *testing.B) {
  for i := 0; i < b.N; i++ {
    unmarshal("post.json")
  }
}
```



同时该命令也会执行单元测试函数，如果想要忽略单元测试，只进行基准测试，可以指定 -run 的参数为一个不存在的正则模式，这样所有的功能测试函数都会被忽略

```shell
➜  unit_testing git:(master) ✗ go test -v -cover -run x -bench .     
goos: darwin
goarch: amd64
pkg: github.com/sausheong/gwp/Chapter_8_Testing_Web_Applications/unit_testing
BenchmarkDecode
BenchmarkDecode-8          55884             21660 ns/op
BenchmarkUnmarshal
BenchmarkUnmarshal-8       49724             25388 ns/op
PASS
coverage: 42.4% of statements
ok      github.com/sausheong/gwp/Chapter_8_Testing_Web_Applications/unit_testing        4.143s

```





# 十、反射

程序在编译时，变量被转换为内存地址，变量名不会被编译器写入到可执行部分。在运行程序时，程序无法获取自身的信息

反射是指**在程序运行期间对程序本身进行访问和修改的能力**

支持反射的语言可以**在程序编译期**将变量的**反射信息**，如**字段名称、类型信息、结构体信息等**整合到可执行文件中，并给程序提供接口访问反射信息，这样就可以在程序运行期获取类型的反射信息，并且有能力修改它们

Go语言使用reflect包访问程序的反射信息

reflect包提供了两个重要的类型：`Type`和`Value`，任意接口值在反射中都可以理解为由 `reflect.Type` 和 `reflect.Value` 两部分组成。。reflect 包提供了 `reflect.TypeOf` 和 `reflect.ValueOf` 两个函数来获取任意对象的 Value 和 Type

## 1. reflect.Type

函数 reflect.TypeOf 接受任意的 interface{} 类型，并以 `reflect.Type` 形式返回其 **动态类型**：

```Go
t := reflect.TypeOf(3)  // a reflect.Type
fmt.Println(t.String()) // "int"
fmt.Println(t)          // "int"
```

这里将3作为接口传递给TypeOf函数，即**将一个具体的值转为接口类型**，会有一个**隐式的接口转换**操作，它会创建一个包含两个信息的接口值：操作数的**动态类型**（这里是 int）和它的**动态的值**（这里是 3）

```go
    var a int
    typeOfA := reflect.TypeOf(a)
    fmt.Println(typeOfA.Name(), typeOfA.Kind())    //int  int
```



### 1.1 Kind和Name

reflect包定义的对象的**反射种类(Kind)**：系统的原生数据类型以及使用type关键字定义的类型

```go
const (
    Invalid Kind = iota  // 非法类型
    Bool                 // 布尔型
    Int                  // 有符号整型
    Int8                 // 有符号8位整型
    Int16                // 有符号16位整型
    Int32                // 有符号32位整型
    Int64                // 有符号64位整型
    Uint                 // 无符号整型
    Uint8                // 无符号8位整型
    Uint16               // 无符号16位整型
    Uint32               // 无符号32位整型
    Uint64               // 无符号64位整型
    Uintptr              // 指针
    Float32              // 单精度浮点数
    Float64              // 双精度浮点数
    Complex64            // 64位复数类型
    Complex128           // 128位复数类型
    Array                // 数组
    Chan                 // 通道
    Func                 // 函数
    Interface            // 接口
    Map                  // 映射
    Ptr                  // 指针
    Slice                // 切片
    String               // 字符串
    Struct               // 结构体
    UnsafePointer        // 底层指针
)
```

Map、Slice、Chan 属于引用类型，使用起来类似于指针，但是在种类常量定义中仍然属于独立的种类，不属于 Ptr。type A struct{} 定义的结构体属于 Struct 种类，*A 属于 Ptr。



获取对象名称和种类等信息：

```go
	m := mmmm{}
	typeOfm := reflect.TypeOf(m)
	fmt.Println(typeOfm.Name())       //mmmm 名称
	fmt.Println(typeOfm.Kind())				//struct 种类
	fmt.Println(typeOfm.String())     //main.mmmm

type Enum int
Zero Enum = 0
    
	typeOfA := reflect.TypeOf(Zero)
  fmt.Println(typeOfA.Name(), typeOfA.Kind())  // Enum int
```



### 1.2 指针和Elem

对于指针获取反射对象时，可以通过`reflect.Elem()`获取**指针指向的元素的反射对象`reflect.Value`**，相当于于`*`操作

**指针变量的`Name`是空**

```go
	m2 := &mmmm{}
	typeOfm2 := reflect.TypeOf(m2)
	fmt.Println(typeOfm2.Name())    //指针变量的Name都是空
	fmt.Println(typeOfm2.Kind(), typeOfm2.String(), typeOfm2.Elem())  //ptr *main.mmmm main.mmmm
	
	typeOfm2Elem := typeOfm2.Elem()  //获取到指针指向元素的类型
	fmt.Println(typeOfm2Elem.Name(), typeOfm2Elem.Kind())   //mmmm struct
```



### 1.3 反射获取结构体成员类型

任意值通过 reflect.TypeOf() 获得反射对象信息后，如果它的类型是**结构体**，可以通过反射值对象 reflect.Type 的 `NumField()` 和 `Field()` 方法获得结构体成员的详细信息

StructField 的结构如下：

```go
type StructField struct {
    Name string          // 字段名
    PkgPath string       // 字段路径
    Type      Type       // 字段反射类型对象
    Tag       StructTag  // 字段的结构体标签
    Offset    uintptr    // 字段在结构体中的相对偏移
    Index     []int      // Type.FieldByIndex中的返回的索引值
    Anonymous bool       // 是否为匿名字段
}
```



- `Field(i int) StructField`	根据索引 i 返回索引对应的结构体字段的信息，当值不是结构体或索引超界时发生宕机
- `NumField() int`	返回结构体成员字段数量，当类型不是结构体时发生宕机
- `FieldByName(name string) (StructField, bool)`	根据给定字符串返回字符串对应的结构体字段的信息，没有找到时 bool 返回 false，当类型不是结构体或索引超界时发生宕机
- `FieldByIndex(index []int) StructField`	多层成员访问时，根据 []int 提供的每个结构体的字段索引，返回字段的信息，没有找到时返回零值。当类型不是结构体或索引超界时发生宕
- `FieldByNameFunc(match func(string) bool) (StructField,bool)`	根据匹配函数匹配需要的字段，当值不是结构体或索引超界时发生宕机

```go
	type cat struct {
		Name string
		// 带有结构体tag的字段
		Type int `json:"type" id:"100"`
	}
	mycat := cat{Name: "mimi", Type: 1}
	typeOfCat := reflect.TypeOf(mycat)
	// 遍历结构体所有成员
	for i := 0; i < typeOfCat.NumField(); i++ {
		fieldType := typeOfCat.Field(i)
		// 输出成员名和tag
		fmt.Printf("name: %v  tag: '%v'\n", fieldType.Name, fieldType.Tag)
	}
	// 通过字段名, 找到字段类型信息
	if catType, ok := typeOfCat.FieldByName("Type"); ok {
		// 从tag中取出需要的tag
		fmt.Println(catType.Tag.Get("json"), catType.Tag.Get("id"))
	}

/* 结果如下：
  name: Name  tag: ''
  name: Type  tag: 'json:"type" id:"100"'
  type 100
*/
```



### 1.4 reflect.Type是一个接口类型

```go
type Type interface {
        Align() int
        FieldAlign() int
        Method(int) Method
        MethodByName(string) (Method, bool)   //获取指定名字的method的引用
        NumMethod() int
        ...
        Implements(u Type) bool   //判断是否实现了某个接口
        ...
}
```



## 2. reflect.Value

- reflect.Value是一个**结构体类型**，这个结构体没有对外暴露的字段，但是提供了获取或者写入数据的方法

```go
type Value struct {
        // 包含过滤的或者未导出的字段
}

func (v Value) Addr() Value
func (v Value) Bool() bool
func (v Value) Bytes() []byte
```



因为 reflect.TypeOf 返回的是一个动态类型的接口值，它总是**返回具体的类型**。因此，下面的代码将打印 "*os.File" 而不是 "io.Writer"

同样的reflect.Valueof也接收任意的interface{}类型，返回状态其动态值的reflect.Value，也是具体的类型

reflect.Type 和 reflect.Value 都实现了 fmt.Stringer 接口，但是除非 Value 持有的是字符串，否则 String 方法只返回其类型。而使用 fmt 包的 %v 标志参数会对 reflect.Values 特殊处理

```Go
var w io.Writer = os.Stdout
fmt.Println(reflect.TypeOf(w)) // "*os.File"

v := reflect.ValueOf(3) // a reflect.Value
x := v.Interface()      // an interface{}
i := x.(int)            // an int  类型断言
fmt.Printf("%d\n", i)   // "3"
```

 **fmt.Printf 提供的 %T 参数，内部就是使用 reflect.TypeOf 来输出**

Type 和 Value 都有一个名为 Kind 的方法，它会返回一个常量，表示底层数据的类型，常见值有：Uint、Float64、Slice 等。

### 2.1 获取值

```go
    // 声明整型变量a并赋初值
    var a int = 1024
    // 获取变量a的反射值对象
    valueOfA := reflect.ValueOf(a)
    // 获取interface{}类型的值, 通过类型断言转换
    var getA int = valueOfA.Interface().(int)
    // 获取64位的值, 强制类型转换为int类型
    var getA2 int = int(valueOfA.Int())
    fmt.Println(getA, getA2)               // 1024 1024
```



### 2.2 修改数据

Value 类型也有一些类似于 Int、Float 的方法，用来提取底层的数据：

- Int 方法用来提取 int64
- Float 方法用来提取 float64

```go
    var x float64 = 3.4
    v := reflect.ValueOf(x)
    fmt.Println("type:", v.Type())		//type: float64
    fmt.Println("kind is float64:", v.Kind() == reflect.Float64)  //kind is float64: true
    fmt.Println("value:", v.Float())  //value: 3.4
```

还有一些用来修改数据的方法，比如 SetInt、SetFloat。。。但是需要有`“可修改性”（settability）`,

首先要求**可被寻址**(反射第三大法则)，其次必须是**导出的**

 Value 的 getter 和 setter 方法：为了保证 API 的精简，这两个方法操作的是某一组类型**范围最大的那个**。比如，处理任何含符号整型数，都使用 `int64`也就是说 Value 类型的 Int 方法返回值为 int64 类型，SetInt 方法接收的参数类型也是 int64 类型。实际使用时，可能需要转化为实际的类型

```go
    var x uint8 = 'x'
    v := reflect.ValueOf(x)
    fmt.Println("type:", v.Type())                            // uint8.
    fmt.Println("kind is uint8: ", v.Kind() == reflect.Uint8) // true.
    x = uint8(v.Uint())                                       // v.Uint returns a uint64.
```



### 2.3 修改结构体字段值

根据反射的第三法则，我们需要有结构体的指针，才可以修改它的字段

```go
    type T struct {
        A int
        B string
    }
    t := T{23, "skidoo"}
    s := reflect.ValueOf(&t).Elem()  // 获取指针指向元素类型的反射对象 reflect.Value
    typeOfT := s.Type()     // 获取指针指向元素类型的反射对象 reflect.Type
    for i := 0; i < s.NumField(); i++ {
        f := s.Field(i)
        fmt.Printf("%d: %s %s = %v\n", i,
            typeOfT.Field(i).Name, f.Type(), f.Interface())
    }

// 0: A int = 23
// 1: B string = skidoo
```

修改:

```go
    type T struct {
        A int
        B string
    }
    t := T{23, "skidoo"}
    s := reflect.ValueOf(&t).Elem()
    s.Field(0).SetInt(77)
    s.Field(1).SetString("Sunset Strip")
    fmt.Println("t is now", t)    // t is now {77 Sunset Strip}
```



## 3. 反射三大法则

 Go 语言反射的三大法则：

1. 从 `interface{}` 变量可以反射出反射对象；
2. 从反射对象可以获取 `interface{}` 变量；
3. 要修改反射对象，其值必须是可设置的；



### 3.1 反射可以将接口类型变量转换为反射类型对象

`reflect.TypeOf()` 和 `reflect.ValueOf()` 可以**将go语言的 interface{} 对象转化为反射对象**，他们的参数类型都是 interface{}。。所以这可能会进行隐式的类型转换。 如果我们认为 **Go 语言的类型** 和 **反射类型**处于两个不同的世界，那么这两个函数就是连接这两个世界的桥梁

```go
func TypeOf(i interface{}) Type     //接收一切 interface{},返回reflect.Type
func ValueOf(i interface{}) Value 
```



![golang-interface-to-reflection](picture/go语言要点/golang-interface-to-reflection.png)

```go
func main() {
	author := "draven"
	fmt.Println("TypeOf author:", reflect.TypeOf(author))   //TypeOf author: string
	fmt.Println("ValueOf author:", reflect.ValueOf(author)) //ValueOf author: draven
}
```

有了变量的类型之后，我们可以通过 `Method` 方法获得类型实现的方法，通过 `Field` 获取类型包含的全部字段。对于不同的类型，我们也可以调用不同的方法获取相关信息：

- 结构体：获取字段的数量并通过下标和字段名获取字段 `StructField`；
- 哈希表：获取哈希表的 `Key` 类型；
- 函数或方法：获取入参和返回值的类型；
- …



### 3.2 反射可以将反射类型对象转换为接口类型变量

反射的第二法则是我们可以从反射对象可以获取 `interface{}` 变量。既然能够将接口类型的变量转换成反射对象，那么一定需要其他方法将反射对象还原成接口类型的变量， `reflect.Value.Interface`就能完成这项工作：

```go
func (v Value) Interface() interface{}    //返回interface{}
```



![golang-reflection-to-interface](picture/go语言要点/golang-reflection-to-interface.png)

```go
v := reflect.ValueOf(1)
v.Interface().(int)   //通过类型断言，将 interface{} 类型转化为了 int 类型
```



**无论是接口值到反射对象还是反射对象到接口值，都是需要两次类型转换的**：

![img](picture/go语言要点/golang-bidirectional-reflection.png)

当然如果变量本身就是 `interface{}` 类型的，那么它不需要类型转换，因为类型转换这一过程一般都是隐式的，所以我不太需要关心它，**只有在我们需要将反射对象转换回基本类型时才需要显式的转换操作**



标准库中的 fmt.Println 和 fmt.Printf 等函数都接收空接口变量作为参数，**fmt 包内部会对接口变量进行拆包**，因此 fmt 包的打印函数在打印 reflect.Value 类型变量的数据时，只需要把 Interface 方法的结果传给格式化打印程序：

```go
	fmt.Println(v.Interface())
```



### 3.3 要修改反射类型对象，其值必须是可写的

最后一条法则是与值是否可以被更改有关，如果我们想要更新一个 `reflect.Value`，那么它持有的值一定是可以被更新的。可以通过 `CanSet()` 方法检查一个 `reflect.Value` 类型变量的可写性

```go
func main() {
	i := 1
	v := reflect.ValueOf(i)
  fmt.Println(v.CanSet())  // false
	v.SetInt(10)     //panic: reflect: reflect.flag.mustBeAssignable using unaddressable value
	fmt.Println(i)
}
```

由于go语言的函数调用都是**值传递**，`reflect.ValueOf` 函数的变量 i 实际上只是 i 的一个拷贝，而非i本身。**我们得到的反射对象和最开始的变量没有任何关系**，所以直接修改反射对象无法改变原始变量，程序为了防止错误就会崩溃

**要修改原变量只能使用如下的方法：**

```go
	i := 1
	v := reflect.ValueOf(&i) //调用 reflect.ValueOf       获取变量指针的反射对象
	elem := v.Elem()         //调用 reflect.Value.Elem    获取指针指向的变量
	elem.SetInt(10)          //调用 reflect.Value.SetInt  修改变量的值   ok， i=10
  elem.Set(reflect.ValueOf(20))    //ok, i = 20
  elem.Set(reflect.ValueOf("abc"))   //panic: string is not assignable to int
  
  //或者通过指针修改
	x := 2
	d := reflect.ValueOf(&x).Elem()   // d refers to the variable x
	px := d.Addr().Interface().(*int) // px := &x
	*px = 3                           // x = 3
```

## 3. 类型和值

Go 语言的 `interface{}` 类型在语言内部是通过 `reflect.emptyInterface` 结体表示的，其中的 `rtype` 字段用于表示变量的类型，另一个 `word` 字段指向内部封装的数据：

```go
type emptyInterface struct {
	typ  *rtype
	word unsafe.Pointer
}
```

用于获取变量类型的 `reflect.TypeOf` 函数将传入的变量隐式转换成 `reflect.emptyInterface`类型并获取其中存储的类型信息 `reflect.rtype`：

```go
func TypeOf(i interface{}) Type {
	eface := *(*emptyInterface)(unsafe.Pointer(&i))
	return toType(eface.typ)
}

func toType(t *rtype) Type {
	if t == nil {
		return nil
	}
	return t
}
```



## 4. 通过类型信息创建实例

当已知 reflect.Type 时，可以动态地创建这个类型的实例，实例的类型为指针。例如 reflect.Type 的类型为 int 时，创建 int 的指针，即`*int`

```go
    var a int

    // 取变量a的反射类型对象
    typeOfA := reflect.TypeOf(a)

		// 根据反射类型对象创建类型实例
    aIns := reflect.New(typeOfA)
    
		// 输出Value的类型和种类
    fmt.Println(aIns.Type(), aIns.Kind())     //*int  ptr
```



## 5. 通过反射调用函数

需要构建大量的 reflect.Value 和中间变量，检查参数，还要将参数复制到调用函数的参数内存中，调用完毕再将返回值转换为 reflect.Value ，性能非常差

```go
// 普通函数
func add(a, b int) int {
    return a + b
}

func main() {
    // 将函数包装为反射值对象
    funcValue := reflect.ValueOf(add)

  	// 构造函数参数, 传入两个整型值
    paramList := []reflect.Value{reflect.ValueOf(10), reflect.ValueOf(20)}
    
  	// 反射调用函数
    retList := funcValue.Call(paramList)
    
  	// 获取第一个返回值, 取整数值
    fmt.Println(retList[0].Int())
}
```





# 十一、标准库



## 1. log包

log.Println()  打印到标准日志记录器

log.Fatalln()  调用后会紧接着调用os.Exit(1)

log.Panicln() 调用后会紧接着调用panic()



## 2. json相关



**两种编码和解码方式：**

1. 编码
   `json.NewEncoder(<Writer>).encode(v)`    
   `json.Marshal(&v)`  返回字节切片 []byte ，为了让编码后的json更加易读，还可以使用 `MatshalIndent(&v, "", "\t")`   以更易读的方式解码，第一个字符串是指定前缀，后面一个字符串指定缩进符
2. 解码
   `json.NewDecoder(<Reader>).decode(&v)`     
   `json.Unmarshal([]byte, &v)`     需要传入字节切片



```go
type Person struct {
    Name string `json:"name"`
    Age int `json:"age"`
}

func main()  {
    // 1. 使用 json.Marshal 编码
    person1 := Person{"张三", 24}
    bytes1, err := json.Marshal(&person1)
    if err == nil {
        // 返回的是字节切片 []byte
        fmt.Println("json.Marshal 编码结果: ", string(bytes1))
    }

    // 2. 使用 json.Unmarshal 解码
    str := `{"name":"李四","age":25}`
    // json.Unmarshal 需要字节数组参数, 需要把字符串转为 []byte 类型
    bytes2 := []byte(str) // 字符串转换为字节切片
    var person2 Person    // 用来接收解码后的结果
    if json.Unmarshal(bytes2, &person2) == nil {
        fmt.Println("json.Unmarshal 解码结果: ", person2.Name, person2.Age)
    }

    // 3. 使用 json.NewEncoder 编码
    person3 := Person{"王五", 30}
    // 编码结果暂存到 buffer(为了变成 io.Writer)
    bytes3 := new(bytes.Buffer)
    err = json.NewEncoder(bytes3).Encode(person3)
    if err == nil {
        fmt.Print("json.NewEncoder 编码结果: ", string(bytes3.Bytes()))
    }

    // 4. 使用 json.NewDecoder 解码
    str4 := `{"name":"赵六","age":28}`
    var person4 Person
    // 创建一个 string reader 作为参数
    err = json.NewDecoder(strings.NewReader(str4)).Decode(&person4)
    if err == nil {
        fmt.Println("json.NewDecoder 解码结果: ", person4.Name, person4.Age)
    }
}

//json.Marshal 编码结果:  {"name":"张三","age":24}
//json.Unmarshal 解码结果:  李四 25
//json.NewEncoder 编码结果: {"name":"王五","age":30}
//json.NewDecoder 解码结果:  赵六 28
```





- 对于**json原生字符串**，**反序列化到对象**，需要使用 `json.Unmarshal()` 函数，同时，需要传入的是原生字符串的字节数组，需要进行转换

```go
type Cat struct {
	Color string `json:"color"`
	Name  string `json:"name"`
}

type CatGroup struct {
	Cats  []Cat  `json:"cats"`
	Owner string `json:"owner"`
}

func main() {
	catsOfXiaoming := `{
		"cats" : [
			{"color":"黄色","name":"小黄猫"},
			{"color":"花色","name":"小花猫"}
		],
		"owner" : "小明"
	}`
	var cg CatGroup

	err := json.Unmarshal([]byte(catsOfXiaoming), &cg)
	if err != nil {
		log.Fatal("解码失败")
	}
	fmt.Printf("%+v", cg)
}
```



- 对于实现了 `io.Reader` 接口的对象，可以直接使用 `json.NewDecoder()` 进行反序列化

```go
	catsOfXiaoming := `{
		"cats" : [
			{"color":"黄色","name":"小黄猫"},
			{"color":"花色","name":"小花猫"}
		],
		"owner" : "小明"
	}`
	var cg CatGroup
	
	json.NewDecoder(strings.NewReader(catsOfXiaoming)).Decode(&cg)
	fmt.Printf("%+v", cg)
```



## 3. io.Reader和io.Writer接口

```go
type Writer interface {
  Write(p []byte) (n int, err error)
}
```

**Write方法**

- 将字节切片p里的 **len(p) 字节**的数据写入到**底层数据流**
- 方法返回从p里**成功写入到底层数据流的字节数n (0<=n<=len(p)) **和可能的错误error
- **当n<len(p)时，必须返回一个非nil的error**

- 不管什么情况都**不能修改byte切片里的数据**



```go
type Reader interface {
  Read(p []byte) (n int, err error)
}
```

**Read方法**

- **最多读入len(p)字节的数据到切片p里**
- 方法返回**成功读入的字节数n (0<=n<=len(p))** 和可能的错误error
- 如果 n<len(p) ，Read会**立即返回可用的数据**，而不是等待更多的数据
- n>0时，遇到错误，或者读取完毕的时候，Read方法均会返回成功读入的字节数，**读取完毕返回EOF错误**
- 调用者在返回的n>0的时候，应该**先处理读入的数据，再处理err**，EOF也要这样处理
- 不要返回n=0的时候返回nil的错误,也就是说，没有读到数据的时候应该返回一个错误



**应用的案例**

`bytes.Buffer` 和 `os.File` 都实现了 `Writer` 和 `Reader` 接口

```go
func main() {
	// bytes.Buffer实现了io.Writer接口
	var b bytes.Buffer
	b.Write([]byte("Hello "))

	// 使用Fprintf将一个字符串拼接到buffer里
	fmt.Fprintf(&b, "World!")

	// 将buffer输出到标准输出
	b.WriteTo(os.Stdout)
}
```

几个函数的说明

```go
func (b *Buffer) Write(p []byte) (n int, err error) {
	b.lastRead = opInvalid
	m, ok := b.tryGrowByReslice(len(p))
	if !ok {
		m = b.grow(len(p))     //缓冲区会自动增大
	}
	return copy(b.buf[m:], p), nil   //追加写
}

func Fprintf(w io.Writer, format string, a ...interface{}) (n int, err error) {...}

func (b *Buffer) WriteTo(w io.Writer) (n int64, err error) {...}
```

关于 `os.Stdout`

```go
var (
	Stdin  = NewFile(uintptr(syscall.Stdin), "/dev/stdin")
	Stdout = NewFile(uintptr(syscall.Stdout), "/dev/stdout")   //就是一个文件
	Stderr = NewFile(uintptr(syscall.Stderr), "/dev/stderr")
)

func NewFile(fd uintptr, name string) *File {...}

func (f *File) Write(b []byte) (n int, err error) {  //File也实现了io.Writer接口
	if err := f.checkValid("write"); err != nil {
		return 0, err
	}
	n, e := f.write(b)
	if n < 0 {
		n = 0
	}
	if n != len(b) {
		err = io.ErrShortWrite
	}

	epipecheck(f, e)

	if e != nil {
		err = f.wrapErr("write", e)
	}

	return n, err
}
```



## 4. json和gob编解码

```go
type MyInfo struct {
	S       string
	I, J, K int
}

type MyReadWriter struct {
	Buf *bytes.Buffer
}

func (m MyReadWriter) Write(p []byte) (n int, err error) {
	return m.Buf.Write(p)
}

func (m MyReadWriter) Read(p []byte) (n int, err error) {
	return m.Buf.Read(p)
}

func main() {
	ms := MyInfo{
		S: "aabbccdd",
		I: 1000,
		J: 200,
		K: 10,
	}
	s := MyReadWriter{
		Buf: new(bytes.Buffer),
	}
	var enc = gob.NewEncoder(s.Buf)    //需要一个writer，将编码结果写入
	var dec = gob.NewDecoder(s.Buf)    //需要一个reader，读取解码信息
	if err := enc.Encode(&ms); err != nil {
		fmt.Println("encode error:", err)
	}
	fmt.Println("编码后：", s.Buf)
	var decMsg MyInfo
	dec.Decode(&decMsg)
	fmt.Println("解码得：", decMsg)

	var enc2 = json.NewEncoder(s.Buf)
	var dec2 = json.NewDecoder(s.Buf)
	if err := enc2.Encode(&ms); err != nil {
		fmt.Println("encode error:", err)
	}
	fmt.Println("编码后：", s.Buf)
	var decMsg2 MyInfo
	dec2.Decode(&decMsg2)
	fmt.Println("解码得：", decMsg2)
}
```



# 十二、Go内存管理、GC

## 1. TCMalloc介绍

Golang的内存分配算法绝大部分都是来自 TCMalloc，Golang只针对其中改动了一小部分。可以说TCMalloc就是Golang内存分配的框架。

Thread-Caching Malloc。是谷歌开发的内存分配器。具有现代化内存分配的基本特征：**对抗内存碎片、在多核处理器能够scale**

主要特点：内存分配快，多线程减少了锁机制(小对象没有锁，大对象自旋锁)，对小对象的处理空间效率更好

![TCMalloc](picture/go语言要点/TCMalloc.png)

TCMalloc分为三层：Front-End, Middle-End, Back-End 层层递进，最后是OS

- 前端是一个高速缓存，为应用程序提供快速的内存分配和重新分配

- 中端负责重新填充前端缓存
- 后端处理从OS提取的内存



我们运行的程序首先向前端申请内存，前端内存缓存不足时，会向中端请求内存，中端内存不足时会想后端请求内存，后端也内存不足就会向OS申请内存

![TCMalloc](picture/go语言要点/TCMalloc2.png)

虚拟内存的概念引入后，让内存的并发访问问题的粒度从多进程级别，降低到多线程级别。TCMalloc**为每个线程预分配一块缓存，线程申请小内存时，可以从缓存分配内存**，这样有2个好处：

1. 为线程预分配缓存需要进行1次系统调用，后续线程申请小内存时，从缓存分配，都是在用户态执行，没有系统调用，**缩短了内存总体的分配和释放时间**
2. 多个线程同时申请小内存时，从各自的缓存分配，访问的是不同的地址空间，无需加锁，**把内存并发访问的粒度进一步降低了**。



### 1.1 一些基础概念

#### Page

操作系统对内存管理的单位，TCMalloc也会是以页为单位管理内存，但是TCMalloc中的Page大小是OS中Page的倍数。TCMalloc默认Page大小是8KB，而Linux是4KB。

TCMalloc并非只将堆内存看做是一个个的page，而是将整个虚拟内存空间都看做是page的集合。从内存地址0x0开始，每个page对应一个递增的PageID，如下图（以32位系统为例）：

![page](picture/go语言要点/page.png)

#### Span

Span是 PageHeap 中管理内存页的单位，由**连续的**一个或者多个Page组成，多个Span用链表进行管理

**TCMalloc 以 Span为单位向 OS 申请内存**

一个Span记录了起始page的 PageID，以及所包含的page数量

Span一般用来管理大对象(占多个Page)，或者已拆分为一系列小对象的一组Page (一个或多个Page被Size-Class拆分为固定大小的Object链表，如果Span管理的是小对象，会在Span中记录对象的Size-Class信息)

![span](picture/go语言要点/span.png)

一个span可以处于三种状态：

- `IN_USE`：要么被拆分成小对象分配给了ThreadCache或CentralCache，要么分配给了应用程序。因为span是由PageHeap来管理的，因此即使只是分配给了CentralCache，还没有被应用程序所申请，在PageHeap看来，也是IN_USE了

- `ON_NORMAL_FREELIST`：空闲，但未归还给OS
- `ON_RETURNED_FREELIST`：空闲，span对应内存释放给OS了，虽然归还了，但是下次该虚拟地址依然可以访问，只是会导致 page fault



#### Size-Class

由Span分裂出的对象，由**同一个Span分裂出的SizeClass大小相同**，SizeClass是对象内存实际的载体

小对象被分配到大小不同的Size-Class上，例如一个12字节的对象会被四舍五入分配到16字节的Size-Class。

Size-Class的设计是为了在舍入到下一个最大的size时尽量减少内存浪费

每个Size-Class对应的 FreeList 链表中的Object大小相同，是由Span分割出来的

![SizeClass](picture/go语言要点/SizeClass.png)

#### ThreadCache

每个线程各自的Cache，一个Cache包含多个空闲内存块链表，每个链表连接的都是内存块，同一个链表上内存块的大小是相同的，也可以说按内存块大小，给内存块分了个类，这样可以根据申请的内存大小，快速从合适的链表选择空闲内存块。由于每个线程有自己的ThreadCache，所以ThreadCache访问是**无锁的**

![ThreadCache2](picture/go语言要点/ThreadCache.png)

#### CentralCache

是所有线程共享的缓存，也是保存的空闲内存块链表，链表的数量与ThreadCache中链表数量相同，当ThreadCache内存块不足时，可以从CentralCache取，当ThreadCache内存块多时，可以放回CentralCache。由于CentralCache是共享的，所以它的访问是要**加锁的**

<img src="picture/go语言要点/CentralCache.png" alt="CentralCache" style="zoom: 25%;" />

#### Page Heap

PageHeap是堆内存的抽象，PageHeap存的也是若干链表，链表保存的是Span，当CentralCache没有内存的时，会从PageHeap取，把1个Span拆成若干内存块，添加到对应大小的链表中，当CentralCache内存多的时候，会放回PageHeap。如下图，分别是1页Page的Span链表，2页Page的Span链表等，最后是large span set，这个是用来保存中大对象的。毫无疑问，PageHeap也是要**加锁的**

<img src="picture/go语言要点/PageHeap.png" alt="PageHeap" style="zoom: 25%;" />



#### Pagemap and Spans

所有被TCMalloc从操作系统被申请到的内存会被划分成编译时就被确认好的Page大小,

pagemap用于查找一个page所属的Span，或标识给定对象的Size-class(例如一个大对象被分配了多个Page的Span,或者一个Span被拆分成Size-class大小后分配给小对象)。 TCMalloc使用2级或3级基数树来映射所有申请到的内存.

![Pagemap](picture/go语言要点/Pagemap.png)



### 1.2 Front-End (ThreadCache)

是一个内存缓存

程序需要内存时首先向Front-End申请

提供快速分配和重新分配内存给应用的功能



可以基于线程分配(per-thread cache) 或者 基于CPU分配(per-cpu cache) 内存

当申请的内存小于 `kMaxSize` 时，直接由前端分配内存，前端对申请的内存大小进行映射到对应的 `Size-Class`(如12KB映射到16KB)，然后将Size-Class分配给对象。

当申请的内存大于 `kMaxSize`时，会交给后端直接分配 `Span` 给大对象



前端高速缓存一次只能由单个线程(CPU)访问，因此无需加锁，分配和释放内存很快



### 1.3 Middle-End (CentralHeap)

如果前端已缓存适当大小的内存,它可以满足大部分的请求。但是如果某种特定大小的缓存为空，则前端将向中端(Middle-End)请求一批内存以重新填充缓存。 中间端包括Central Free List和Transfer Cache。



#### Transfer Cache(传输缓存)

当前端请求内存或返回内存时，它将到达传输缓存。

传输缓存包含一个指向可用内存的指针数组，可以快速将对象移动到该数组中，或者代表前端从该数组中获取对象。

传输缓存的存在是由于**一个线程正在分配由另一个线程释放的内存的情况**。 传输缓存允许内存在两个不同线程之间快速流动。

如果传输缓存无法满足内存请求，或者没有足够的空间容纳返回的对象，它将访问Central Free List



#### Central Free List

Central Free List是当Front-End内存不足时,提供内存供其使用

Central Free List 以Span为基础管理内存,当需要内存时,会从不同的Span中提取内存.

当对象返回到Central Free List时，每个对象都将映射到它所属的Span（使用pagemap，然后**释放到该Span中。如果驻留在特定Span中的所有对象都返回给它，则整个Span 返回到后端**）

如果中端已用尽，将请求后端分配Page填充到中端. 后端也称为PageHeap.

![CentralFreeList](picture/go语言要点/CentralFreeList.png)

### 1.4 Back-End (PageHeap)

TCMalloc的后端有三个任务:

- 管理未使用的内存。
- 当没有合适大小的可用内存来满足分配请求时, 它负责从操作系统获取内存。
- 将不需要的内存返回给操作系统。

TCMalloc有两个类型的后端:

- Legacy Pageheap

用于管理一定大小的Span.

- Hugepage Aware Allocator

用于管理大对象,通常超过2MB

当中端内存不够的请求来到后端时,后端会返回对应Span的内存给中端



### 1.5 TCMalloc的内存分配和回收

TCMalloc的定义：

1. 小对象大小：0~256KB
2. 中对象大小：257~1MB
3. 大对象大小：>1MB

小对象的分配流程：ThreadCache -> CentralCache -> PageHeap，大部分时候，ThreadCache缓存都是足够的( FreeList非空 )，不需要去访问CentralCache和HeapPage，无锁分配加无系统调用，分配效率是非常高的。

中对象分配流程：直接在PageHeap中选择适当的大小即可，128 Page的Span所保存的最大内存就是1MB。

大对象分配流程：从large span set选择合适数量的页面组成span，用来存储数据



应用程序调用free()或delete一个小对象时，仅仅是将其插入到ThreadCache中其size class对应的FreeList中而已，不需要加锁，因此速度也是非常快的。

只有当满足一定的条件时，ThreadCache中的空闲对象才会重新放回CentralCache中，以供其他线程取用。同样的，当满足一定条件时，CentralCache中的空闲对象也会还给PageHeap，PageHeap再还给系统。



## 2. Golang内存管理

宏观来看，Go的内存管理就是下图的样子，重点在于堆内存的部分

![Go内存管理](picture/go语言要点/Go内存管理.png)

Go这门语言抛弃了C/C++中的开发者管理内存的方式：主动申请与主动释放，增加了逃逸分析和GC，将开发者从内存管理中释放出来，让开发者有更多的精力去关注软件设计，而不是底层的内存问题。这是Go语言成为高生产力语言的原因之一



栈内存和堆内存的几点区别：

1. 栈的内存管理简单，分配比堆上快。
2. 栈的内存不需要回收，而堆需要，无论是主动free，还是被动的垃圾回收，这都需要花费额外的CPU。
3. 栈上的内存有更好的局部性，堆上内存访问就不那么友好了，CPU访问的2块数据可能在不同的页上，CPU访问数据的时间可能就上去了。

当我们说内存管理的时候，主要是指**堆内存的管理**，栈内存管理不需要程序去操心。

### 2.1 内存分配的基础概念

两种内存分配方式：

#### 1. 线性分配

线性分配非常高效。只需要维护一个指向内存特定位置的指针，当用户程序申请内存时，分配器检查剩余空闲内存，返回分配的内存区域并修改指针位置

缺陷：当已分配内存被回收后，单靠单个指针没有办法继续分配前面空闲的区域(红色区域)

![线性分配](picture/go语言要点/线性分配.png)



由此衍生的垃圾回收算法解决线性分配的缺陷：**标记压缩，标记复制**等。 通过拷贝的方式整理内存碎片，提升线性分配器的效率和性能



#### 2. 空闲链表分配

重用已经被释放的内存，在内部维护一个空闲链表。用户程序申请内存时，一次遍历空闲链表，找到足够大的内存，然后分配给用户，修改链表

常见的空闲链表分配方法：

- 首次适应：每次从头开始，找到第一个匹配的就返回
- 循环首次适应：从上一次结束位置开始遍历
- 最优适应：遍历整个链表，选择最合适的内存块
- 隔离适应：将内存分隔成多个链表，每个链表中内存块大小相同，申请内存时先找到满足条件的链表，再从链表中选择合适的内存块

#### 关于Golang内存分配

Golang的内存分配法和隔离适应算法类似，并且借鉴了TCMalloc的设计实现高速的内存分配。核心理念是**使用多级缓存将对象根据大小分类，按照类别实施不同的分配策略**

对象被分为三种大小：

- 小于16KB：微对象
- 16KB到32KB：小对象
- 大于32KB：大对象

Go语言也采用了TCMalloc的多级缓存，并且采用的是 `Per-Thread Cache` 模式，每个线程都有独立的缓存

![golang内存分配](picture/go语言要点/golang内存分配.png)



### 2.2 Go内存布局

Go1.10以前采用线性内存，管理简单，但是会造成C，Go混用导致程序崩溃，1.11之后，转为了稀疏内存管理。

![Go内存布局](picture/go语言要点/Go内存布局.png)

通过一个 `heapArena` 数组对内存进行管理

```go
type heapArena struct {
	// 前两者与线性分配中的bitmap以及spans对应,用于映射内存地址与Span.
	bitmap [heapArenaBitmapBytes]byte
	
	spans [pagesPerArena]*mspan
	
	// 一个位图用于表示在当前heapArena管理的Span中,哪些Span是正在被使用的.
	// 只使用了每个Span的第一个Page(Span可以由多页构成) 并且读写是原子性的
	pageInUse [pagesPerArena / 8]uint8
	
	// 与pageInUse类似,也是一个位图,用于表示哪些Span上有被标记的对象,用于垃圾回收.
	pageMarks [pagesPerArena / 8]uint8
	
	// 这个指针指向的是整个heapArena结构管理的首地址,通过CAS的方式进行修改
	zeroedBase uintptr
}
```



Go内存管理的许多概念在TCMalloc中已经有了，含义是相同的，只是名字有一些变化

![Go内存管理详解](picture/go语言要点/Go内存管理详解.png)

**Page：**

与TCMalloc中的Page相同，x64下1个Page的大小是8KB。上图的最下方，1个浅蓝色的长方形代表1个Page。

**Span：** 

与TCMalloc中的Span相同，**Span是内存管理的基本单位**，代码中为`mspan`，**一组连续的Page组成1个Span**，所以上图一组连续的浅蓝色长方形代表的是一组Page组成的1个Span，另外，1个淡紫色长方形为1个Span

**mcache：** 

mcache与TCMalloc中的ThreadCache类似，**mcache保存的是各种大小的Span，并按Span class分类，小对象直接从mcache分配内存，它起到了缓存的作用，并且可以无锁访问**。

但mcache与ThreadCache也有不同点，TCMalloc中是每个线程1个ThreadCache，Go中是**每个P拥有1个mcache**，因为在Go程序中，当前最多有GOMAXPROCS个线程在用户态运行，所以最多需要GOMAXPROCS个mcache就可以保证各线程对mcache的无锁访问，线程的运行又是与P绑定的，把mcache交给P刚刚好

**mcentral：**

mcentral与TCMalloc中的CentralCache类似，**是所有线程共享的缓存，需要加锁访问**，它按Span class对Span分类，串联成链表，当mcache的某个级别Span的内存被分配光时，它会向mcentral申请1个当前级别的Span。

但mcentral与CentralCache也有不同点，CentralCache是每个级别的Span有1个链表，mcache是**每个级别的Span有2个链表**，这**和mcache申请内存有关**

**mheap：**

mheap与TCMalloc中的PageHeap类似，**它是堆内存的抽象，把从OS申请出的内存页组织成Span，并保存起来**。当mcentral的Span不够用时会向mheap申请，mheap的Span不够用时会向OS申请，向OS的内存申请是按页来的，然后把申请来的内存页生成Span组织起来，同样也是需要加锁访问的。

但mheap与PageHeap也有不同点：**mheap把Span组织成了树结构，而不是链表，并且还是2棵树**，然后把Span分配到heapArena进行管理，它包含地址映射和span是否包含指针等位图，这样做的主要原因是为了更高效的利用内存：分配、回收和再利用



### 2.3 对象大小转换

1. **object size**：代码里简称`size`，指申请内存的对象大小。
2. **size class**：代码里简称`class`，它是size的级别，相当于把size归类到一定大小的区间段，比如size[1,8]属于size class 1，size(8,16]属于size class 2
3. **span class**：指span的级别，但span class的大小与span的大小并没有正比关系。span class主要用来和size class做对应，1个size class对应2个span class，2个span class的span大小相同，只是功能不同，1个用来存放包含指针的对象，一个用来存放不包含指针的对象，不包含指针对象的Span就无需GC扫描了。
4. **num of page**：代码里简称`npage`，代表Page的数量，其实就是Span包含的页数，用来分配内存。

<img src="picture/go语言要点/Go内存大小转换.png" alt="Go内存大小转换" style="zoom: 50%;" />

![Go内存分配表](picture/go语言要点/Go内存分配表.png)



size转换是没有固定的计算规律的，直接进行了硬编码，通过查表得到：

`size-class`：从1到66(从8byte到3Kb)，一共有66个，加上未使用的 size class 0，实际上有67个

`span-class`：一个size-class对应两个span-class，所以span-class有134个，从0到133，每个span class指向一个span，也就是说，每个mcache有最多134个 span class 链表

![Size转换](picture/go语言要点/Size转换.png)

### 2.4 Go内存分配

![Go内存对象分类](picture/go语言要点/Go内存对象分类.png)

Go中的内存分类并不像TCMalloc那样分成小、中、大对象，但是它的小对象里又细分了一个Tiny对象，Tiny对象指大小在1Byte到16Byte之间并且不包含指针的对象。小对象和大对象只用大小划定，无其他区分。

小对象是在mcache中分配的，而大对象是直接从mheap分配的

1. TCMalloc 会给每一个线程分配一个属于`线程本地的缓存(Thread Cache, mcache)`，这个本地缓存用于`小对象(小于32K)`的内存分配，**ThreadCache中分配和回收内存无需加锁**(线程私有)。在必要的时候，对象会从Central Heap (mheap) (备注：这个是多个线程分享的，操作的时候需要做加锁、解锁处理)移动到Thread Cache(mcache)
2. GC会周期性的将存储从 Thread Cache(mcache 默认是2M) 迁移到 Central Heap (mheap)，以便进行垃圾回收
3. 对于大对象(大于32K)则直接从 Central Heap(mheap)按照页面层次分配方式进行内存分配



### 2.5 微对象的分配

Go 语言运行时将**小于 16 byte**的对象划分为微对象，它会使用线程缓存上的微分配器提高微对象分配的性能，我们主要使用它来分配**较小的字符串以及逃逸的临时变量**。微分配器可以将多个较小的内存分配请求合入同一个内存块中，只有当内存块中的所有对象都需要被回收时，整片内存才可能被回收。

微分配器管理的对象不可以是指针类型

![微对象的分配](picture/go语言要点/微对象的分配.png)

如上图中,微对象分配器中已经被分配了12B的内存,现在**仅剩下4B空闲**, 如果此时有小于等于4B的对象需要被分配内存,那么这个对象会直接使用tinyoffset之后剩余的空间。如果大于4B，就要被算在小对象分配的过程中了



### 2.6 小对象的分配

#### 1. 为对象寻找span

小对象是大小从**16字节到32768字节的对象**，以及**小于16字节的指针类型对象**

根据小对象的`size`，映射所对应的 `SizeClass`，然后根据 SizeClass 和对象中是否包含指针，计算出SpanClass，获取该SpanClass对应的空闲Span空间



例如：24Byte不包含指针的对象，对应的 size-class 为 3 ( `(16,32]Byte` )

根据size-class计算span-class

```go
// noscan为true代表对象不包含指针
func makeSpanClass(sizeclass uint8, noscan bool) spanClass {
	return spanClass(sizeclass<<1) | spanClass(bool2int(noscan))
}
```

所以对应span-class为： 3<<1 | 1 = 7，所以该对象需要的是 span class 7 指向的 span



#### 2. 从span中分配对象空间

Span可以按对象大小切成很多份，这些都可以从映射表上计算出来，以size class 3对应的span为例，span大小是8KB，每个对象实际所占空间为32Byte，这个span就被分成了256块，可以根据span的起始地址计算出每个对象块的内存地址。

<img src="picture/go语言要点/Span内对象.png" alt="Span内对象" style="zoom: 67%;" />

随着内存的分配，span中的对象内存块，有些被占用，有些未被占用，比如上图，整体代表1个span，蓝色块代表已被占用内存，绿色块代表未被占用内存。

当分配内存时，只要快速找到第一个可用的绿色块，并计算出内存地址即可，如果需要还可以对内存块数据清零。



#### 3. mcache向mcentral申请span

当span内所有对应size-class的内存块都被占用时，没有剩余空间可以继续分配对象，这时 **mcache 会向 mcentral 申请1个span**，mcache拿到这个span后继续分配对象

mcentral和mcache一样，都是0~133这134个span class级别，但每个级别都保存了2个span list，即2个span链表：

1. `nonempty`：这个链表里的span，所有span都至少有1个空闲的对象空间。这些span是mcache释放span时加入到该链表的。
2. `empty`：这个链表里的span，所有的span都不确定里面是否有空闲的对象空间。当一个span交给mcache的时候，就会加入到empty链表。

这2个东西名称一直有点绕，建议直接把empty理解为没有对象空间就好了。

<img src="picture/go语言要点/2019-07-go-mcentral.png" alt="mcentral" style="zoom: 67%;" />

mcache向mcentral要span时，mcentral会先从`nonempty`搜索满足条件的span，如果没有找到再从`emtpy`搜索满足条件的span，然后把找到的span交给mcache。



#### 4. mcentral向mheap申请span

当mcentral没有满足条件的span时，如`empty`中也没有符合条件的span，则会向mheap申请span

mheap里保存了2棵**二叉排序树**，按span的page数量进行排序：

1. `free`：free中保存的span是空闲并且非垃圾回收的span。
2. `scav`：scav中保存的是空闲并且已经垃圾回收的span。

如果是垃圾回收导致的span释放，span会被加入到`scav`，否则加入到`free`，比如刚从OS申请的的内存也组成的Span。

<img src="picture/go语言要点/2019-07-go-mheap.png" alt="mheap" style="zoom: 67%;" />

mheap中还有arenas，有一组heapArena组成，每一个heapArena都包含了连续的`pagesPerArena`个span，这个主要是为mheap管理span和垃圾回收服务。

mheap本身是一个全局变量，它其中的数据，也都是从OS直接申请来的内存，并不在mheap所管理的那部分内存内



mcentral需要向mheap提供需要的**内存页数和span class级别**，然后它优先从`free`中搜索可用的span，如果没有找到，会从`scav`中搜索可用的span，如果还没有找到，它会向OS申请内存，再重新搜索2棵树，必然能找到span。如果找到的span比需求的span大，则把span进行分割成2个span，其中1个刚好是需求大小，把剩下的span再加入到`free`中去，然后设置需求span的基本信息，然后交给mcentral



#### 5. mheap向OS申请内存

当mheap没有足够的内存时，mheap会向OS申请内存，把申请的内存页保存到span，然后把span插入到`free`树 。

在32位系统上，mheap还会预留一部分空间，当mheap没有空间时，先从预留空间申请，如果预留空间内存也没有了，才向OS申请。



### 2.7 大对象的分配

大对象是指大于32K的对象，直接分配在 mcentral 中

大对象的分配比小对象省事多了，99%的流程与mcentral向mheap申请内存的相同



### 2.8 对象释放

Go使用垃圾回收收集不再使用的span，调用`mspan.scavenge()`把span释放给OS（并非真释放，只是告诉OS这片内存的信息无用了，如果你需要的话，收回去好了），然后交给mheap，mheap对span进行span的合并，把合并后的span加入`scav`树中，等待再分配内存时，由mheap进行内存再分配



注意，GC只是给内核提供一个建议：**这个内存地址区间的内存已经不再使用，可以回收。但内核是否回收，以及什么时候回收，这就是内核的事情了**。如果内核真把这片内存回收了，当Go程序再使用这个地址时，内核会重新进行虚拟地址到物理地址的映射。所以在内存充足的情况下，内核也没有必要立刻回收内存



## 3. GoGC

传统编程语言(c/c++)需要手动释放内存，存在内存泄漏或者误释放内存的问题。后来的变成需要都引入了语言层面的自动内存管理，内存的释放由`虚拟机(VM)`或者`运行时(runtime)`自动进行



### 3.1 常见GC算法和GoGC

- **标记清除：** 从根对象出发，将确定存活的对象进行标记，并清除可以回收的对象。简单快速，但会造成内存碎片，导致大对象创建不了。
- **复制算法：** 内存分为大小相同的两块，只使用一块，互相复制。解决了碎片问题，但是浪费资源，且移动耗费时间，效率低
- **标记整理：** 为了解决内存碎片问题而提出，在标记过程中，将对象尽可能整理到一块连续的内存上。性能低。
- **分代式：** 将对象根据存活时间的长短进行分类，存活时间小于某个值的为年轻代，存活时间大于某个值的为老年代，永远不会参与回收的对象为永久代。并根据分代假设（如果一个对象存活时间不长则倾向于被回收，如果一个对象已经存活很长时间则倾向于存活更长时间）对对象进行回收。
- **引用计数：** 根据对象自身的引用计数来回收，当引用计数归零时立即回收。计数频繁更新会带来很多额外开销，无法解决循环引用问题。



**GC的难点**在于，无法预料程序是否是暂停时间敏感的(批处理作业还是交互式程序)，在这种情况下为了平衡，最好使用一种吞吐量最大化的算法，即有用工作与用于垃圾回收的时间之比。没有一种垃圾回收算法是各方面都完美的。



Go程序通常是请求/响应处理，如HTTP服务器，表现出强烈的分代收集行为，但是Go并没有采用分代，而是用了古老的标记清除。这样做的好处是可以获得**非常短的暂停时间**。但是每轮GC都需要大量工作，吞吐量会降低。而且为了避免出现**并发模式故障**(垃圾生产速度大于GC清理速度)，Go默认堆开销为100%，使程序所需内存增加一倍。



目前Go语言使用的是：**无分代、不整理、并发** 的三色标记算法

无分代：Go 的编译器会通过**逃逸分析**将**大部分新生对象存储在栈上**（当 goroutine 死亡后栈也会被直接回收），只有那些需要长期存在的对象才会被分配到需要进行垃圾回收的堆中。分代会消耗额外的赋值器时间，Go团队认为下一个10年内存会比CPU增长快的多，所以不允许消耗一点CPU。

不整理：整理是为了解决内存碎片问题以及“允许”使用顺序内存分配器。但 Go 运行时的分配算法基于 **tcmalloc**，基本上没有碎片问题。这个问题对于Go来讲一点都不需要担心



### 3.2 三色标记法原理

三色标记算法：

![三色标记](picture/go语言要点/三色标记.png)

- **黑色对象：确定存活的对象**，已被回收器访问到的对象，其中所有的字段都已被扫描！黑色对象中的任何一个指针都不可能直接指向白色对象
- **灰色对象：波面，已被回收器访问到的对象**，但是回收器还得对其中的指针进行扫描，因为这些指针可能指向当前的某些白色对象
- **白色对象：未被回收器访问到的对象**，回收开始时所有对象都是白色，等标记结束后，白色对象就是不可达的垃圾对象

垃圾回收的过程可以理解为波面不断前进的过程。垃圾回收**开始时只有白色对象**，当**一个节点的所有子节点扫描完后，该节点会变成黑色节点**。等堆遍历完成后，只有黑色节点与白色节点，黑色为可到达对象，即存活，白色为不可到达对象，即死亡。

1. 开始：所有对象都是白色
2. 从根对象出发，扫描所有可达对象，标记为灰色，放入待处理队列
3. 从待处理队列取出灰色对象，将其引用的对象也标记为灰色并放入待处理队列中，完成后将自身标记为黑色
4. 重复3，直到待处理队列空，此时的白色对象即为不可达的垃圾

>根对象在垃圾回收的术语中又叫做根集合，它是垃圾回收器在标记过程时最先检查的对象，包括： 
>
>（1）全局变量：程序在编译期就能确定的那些存在于程序整个生命周期的变量。 
>
>（2）执行栈：每个 goroutine 都包含自己的执行栈，这些执行栈上包含栈上的变量及指向分配的堆内存区块的指针。 
>
>（3）寄存器：寄存器的值可能表示一个指针，参与计算的这些指针可能指向某些赋值器分配的堆内存区块



### 3.3 屏障机制

#### 1. 没有STW存在的问题

垃圾回收原则是不出现对象的丢失，不错误回收不需要回收的对象。

> 如果满足下面两个条件，**会破坏垃圾回收原则**，出现对象丢失
>
> 1. 赋值器修改对象图，导致一黑色对象突然引用白色对象(白色被挂在黑色下)
> 2. 从灰色对象出发，到达白色对象且未经访问的路径被赋值器破坏(灰色同时丢了该白色)



**没有STW的例子：**

假设灰色对象2(未被扫描)引用了白色对象3

此时黑色对象4(已扫描完毕)引用对象3的同时，灰色对象2删除了对对象3的引用

那么白色对象3就不会被扫描到，导致最后对象3是白色，被误当做垃圾清除

![NoSTW](picture/go语言要点/NoSTW.png)

可能的解决办法：整个过程STW，浪费资源且对用户程序影响大，所以引入了**屏障机制**

把回收器视为对象，把赋值器视为影响回收器这一对象的实际行为（即影响 GC 周期的长短），从而引入赋值器的颜色：

- 黑色赋值器：已经由回收器扫描过，不会再次对其进行扫描。
- 灰色赋值器：尚未被回收器扫描过或尽管已经扫描过，但仍需要重新扫描。



#### 2. 强弱三色不变式

**强三色不变式：** 黑色对象不能指向白色对象，只能指向灰色或黑色对象

![强三色不变式](picture/go语言要点/强三色不变式.png)

**弱三色不变式：** 黑色对象可以指向白色对象，但此白色对象链路的上游中必须有一个灰色对象(白色对象必须被灰色对象"保护")

![弱三色不变式](picture/go语言要点/弱三色不变式.png)



为了遵循上述的两个方式,Golang团队初步得到了如下具体的两种屏障方式“插入屏障”, “删除屏障”.



#### 3. 插入写屏障(Dijkstra)-灰色赋值器

GO V1.5 采用的方法 (GO V1.3是普通标记清除，整个过程STW，效率极低)

**具体操作**: 在A对象引用B对象的时候，B对象被标记为灰色。(将B挂在A下游，B必须被标记为灰色)

**满足**: **强三色不变式**. (不存在黑色对象引用白色对象的情况了， 因为白色会强制变成灰色)

>  破坏条件1（ 赋值器修改对象图，导致某一黑色对象引用白色对象；）因为在对象A 引用对象B 的时候，B 对象被标记为灰色



黑色对象的内存槽有两种位置, `栈`和`堆`. 栈空间的特点是容量小,但是**要求响应速度快**,因为函数调用弹出频繁使用, 所以“插入屏障”机制,在**栈空间的对象操作中不使用**. 而仅仅使用在堆空间对象的操作中



Dijkstra 插入屏障的好处在于可以立刻开始并发标记。但存在两个缺点：

- 由于 Dijkstra 插入屏障的“保守”，在一次回收过程中可能会残留一部分对象没有回收成功，只有在下一个回收过程中才会被回收；
- 在标记阶段中，每次进行**指针赋值操作**时，都需要**引入写屏障**，这无疑会增加大量性能开销；为了避免造成性能问题，Go 团队在最终实现时，**没有为栈上的指针的写操作启用写屏障**，而是当发生栈上的写操作时，将栈标记为灰色，但此举产生了**灰色赋值器**，将会需要在标记终止阶段 STW 时对这些栈进行**重新扫描**。

![插入屏障](picture/go语言要点/插入屏障.png)

但是如果栈不添加,当全部三色标记扫描之后,栈上有可能依然存在白色对象被引用的情况(如上图的对象9). 所以要对栈重新进行三色标记扫描, 但这次为了对象不丢失, 要对本次标记扫描启动STW暂停. 直到栈空间的三色标记结束

![插入屏障-STW](picture/go语言要点/插入屏障-STW.png)

#### 4. 删除写屏障(Yuasa)-黑色赋值器

**具体操作**: 被删除的对象，如果自身为灰色或者白色，那么被标记为灰色。

**满足**: **弱三色不变式**. (保护灰色对象到白色对象的路径不会断)

> 破坏条件2（从灰色对象出发，到达白色对象的、未经访问过的路径被赋值器破坏），因为被删除对象，如果自身是灰色或者白色，则被标记为灰色

**特点**：标记结束不需要STW，但是回收精度低(**一个对象即使被删除了最后一个指向它的指针也依旧可以活过这一轮，在下一轮GC中才会被清理掉**)，容易产生“冗余”扫描；

![删除屏障](picture/go语言要点/删除屏障.png)

#### 4. 混合写屏障

Go V1.8 新增的混合写屏障机制，**大大缩短了STW时间**

- **插入写屏障短板**：结束时需要STW来重新扫描栈，标记栈上引用的白色对象的存活；
- **删除写屏障短板**：基于起始快照，回收精度低，GC开始时STW扫描堆栈来记录初始快照，这个过程会保护开始时刻的所有存活对象。



混合写屏障机制整体流程：

- GC 开始将**栈上**的对象全部扫描并标记为**黑色** (之后不再进行第二次重复扫描，无需STW)
- GC 期间，任何在**栈上**创建的新对象，均为**黑色**
- 被删除的堆对象标记为灰色
- 被添加的堆对象标记为灰色

**满足**：变形的弱三色不变式

这里我们注意， **屏障技术是不在栈上应用的，因为要保证栈的运行效率**

Golang 中的混合屏障结合了删除写屏障和插入写屏障的优点，**只需要在开始时并发扫描各goroutine的栈，使其变黑并一直保持，标记结束后，因为栈空间在扫描后始终是黑色的，无需进行re-scan**，减少了STW 的时间。

另外注意：混合写屏障是GC的一种屏障机制，所以只是当程序执行GC的时候才会触发这种机制

**GC开始：** 扫描栈区，将栈区可达的对象全部标记为**黑色**

![混合屏障-准备](picture/go语言要点/混合屏障-准备.png)



**场景一：对象被一个堆对象删除引用，成为了栈对象的下游**

由于屏障的作用，对象7不会被误删除

![混合屏障-场景一](picture/go语言要点/混合屏障-场景一.png)



**场景二：对象被一个栈对象删除引用，成为栈对象的下游**

![混合屏障-场景二](picture/go语言要点/混合屏障-场景二.png)



**场景三：对象被一个堆对象删除引用，成为堆对象的下游**

![混合屏障-场景三](picture/go语言要点/混合屏障-场景三.png)



**场景四：对象被一个栈对象删除引用，成为另一个堆对象的下游**

![混合屏障-场景四](picture/go语言要点/混合屏障-场景四.png)

### 3.2 GoGC-标记清除算法

1.5版本及之后的GC主要分四个阶段，其中标记和清理都是并发执行的，但是标记前后需要STW来做**GC的准备工作和栈的rescan**



| 阶段             | 说明                                                       | 赋值器状态 |
| ---------------- | ---------------------------------------------------------- | ---------- |
| SweepTermination | 清扫终止阶段，为下一个阶段的并发标记做准备工作，启动写屏障 | STW        |
| Mark             | 扫描标记阶段，与赋值器并发执行，写屏障开启                 | 并发       |
| MarkTermination  | 标记终止阶段，保证一个周期内标记任务完成，停止写屏障       | STW        |
| GCoff            | 内存清扫阶段，将需要回收的内存归还到堆中，写屏障关闭       | 并发       |
| GCoff            | 内存归还阶段，将过多的内存归还给操作系统，写屏障关闭       | 并发       |

**GC触发：** 

- `GOGC threshold`: 当前分配的内存达到一定**阈值**时触发，这个阈值在每次GC过后都会根据堆内存的增长情况和CPU占用率来调整（目前唯一**可以调整的参数GCPercent, 以后可能会有MaxHeap**）

- `runtime.forcegcperiod`: 当一定时间没有执行过GC就强制触发GC（2分钟）

- `runtime.GC()`: 手动触发

#### 1. SweepTermination(STW)

栈标记阶段

所有P都启用写屏障后开始扫描(STW)，为每个P创建新的G用来GC调度。

> 每个P都有一个 `mcache` ，每个 `mcache` 都有1个`Span`用来存放 `TinyObject`，TinyObject 都是**不包含指针的对象**，所以这些对象可以直接标记为**黑色**，然后关闭 STW

每个P都有1个进行扫描标记的goroutine，可以进行并发标记，关闭STW后，这些goroutine就变成了可运行状态，接受 go scheduler 的调度，这些 goroutine被调度来进行栈扫描、标记、标记结束

> 栈扫描阶段就是把前面搜集的黑色Root对象的引用标记为灰色，加入 gcWork队列(gcWork队列保存了灰色对象)，每个灰色对象都是一个Work

#### 2. Mark(并发标记)

标记阶段是一个循环，不断从 gcWork 队列中取出 Work，将它所指向的所有对象标记为灰色，将它标记为黑色。循环直到队列为空

#### 3. MarkTermination

标记终止阶段，再次开启 STW，不同版本处理方式不同

Go1.7是 Dijkstra写屏障(插入写屏障)，只监控堆中指针变动，所以需要**再次扫描全局变量和栈**

Go1.8之后是混合写屏障，也不监控栈上指针变动，但是这个策略可以**无需再扫描栈和全局变量**，但是仍需要STW进行一些检查



标记结束阶段最后会关闭写屏障，然后关闭STW，唤醒负责垃圾清扫的 goroutine

#### 4. GCoff

负责清扫垃圾的goroutine 是 **应用程序启动后就立即创建的一个后台goroutine**，并且立刻进入睡眠，等待被唤醒后执行垃圾清理，把白色对象挨个清理掉，清扫goroutine 和 应用 goroutine 是并发进行的，清扫完成后再次进入睡眠状态，等待下次被唤醒

最后执行一些数据统计和状态修改的工作，设置好下一轮GC的阈值，把GC状态设置为 off



# 十三、GMP调度器

## 1. cpu和协程

### 1.1 CPU

物理cpu：主板上实际插入的 cpu 数量

cpu核心数：单块cpu上面能处理数据的芯片数量，例如双核、四核

逻辑cpu数：一般逻辑cpu数=物理cpu数×每颗的核数，如果不相等，说明cpu支持超线程技术，提高了系统性能，此时逻辑cpu数=物理cpu数×每颗cpu的核心数×2



多核 CPU 的每一个核心拥有自己**独立的运算单元、寄存器、L1核L2缓存**，所有核心**共用同一条内存总线**，同一段内存。



### 1.2 协程

在多进程/多线程的操作系统中，一个进程阻塞，cpu可以立即切换到其他进程中去执行，这样从宏观来看，似乎多个进程是在同时被运行

但新的问题就又出现了，进程拥有太多的资源，进程的创建、切换、销毁，都会占用很长的时间，CPU虽然利用起来了，但如果进程过多，**CPU有很大的一部分都被用来进行进程调度**

为了解决这个问题，就出现了用户态线程，一个用户态线程必须要绑定到一个内核态线程，但是cpu并不知道有用户态线程的存在，它只知道它在运行的是一个内核态线程。和内核线程(`thread`)不同，用户态线程叫做**协程**(`co-routine`)

下面统一用线程指代内核线程，协程指代用户线程



协程和线程的映射关系有三种：

**协程：线程 = N : 1，N个协程绑定1个线程**

![N-1模型](picture/go语言要点/N-1模型.png)

- 优点：协程在用户态线程完成了切换，不会陷入内核态，所以**切换轻量快速**
- 缺点：1个进程的所有协程都绑定在1个线程上，进程**用不了硬件的多核加速能力**，**一旦某个协程阻塞就会造成线程阻塞**，本进程的其他协程都无法执行了，没有了并发能力



**协程：线程 = 1 : 1，1个协程绑定1个线程**

![1-1模型](picture/go语言要点/1-1模型.png)

- 优点：最容易实现，协程的调度都由CPU完成了，不存在N:1的缺点
- 缺点：协程的创建、删除、切换的代价都由CPU完成，代价略显昂贵



**协程：线程 = M : N，M个协程绑定N个线程**

![M-N模型](picture/go语言要点/M-N模型.png)

这种模型克服了前面两种模型的缺点，但是实现起来最为复杂



### 1.3 Goroutine

goroutine来自协程的概念，一组协程运行在一个线程之上，即使有协程阻塞，该线程的其他协程也可以被`runtime`调度，转移到其他可运行的线程上。最关键的是，程序员看不到这些底层的细节，这就降低了编程的难度，提供了更容易的并发。 

goroutine非常轻量，一个goroutine只占几KB，并且这几KB就足够goroutine运行完，这就能在有限的内存空间内支持大量goroutine，支持了更多的并发。虽然一个goroutine的栈只占几KB，但实际是**可伸缩的**，如果需要更多内容，`runtime`会自动为goroutine分配。

特点：

- 占用内存更小：几KB
- 调度更灵活：runtime调度



### 1.4 GMP概念

G：goroutine

M：worker thread， or machine，即OS线程

P：processor，处理器，包含了运行goroutine的资源

如果线程M想运行goroutine，必须先获取P，P中包含了可运行的G队列，从队列中获得G



## 2. 早期的GM调度器

Go1.1版本之前就是用的GM模型

除了G和M之外，还有一个**全局协程队列**，存放的是多个处于**可运行状态的G**，M如果想要运行G，就必须访问全局队列

![早期的调度器](picture/go语言要点/早期的调度器.png)

M想要执行、放回G都必须访问**全局G队列**，并且M有多个，即多线程访问同一资源需要加锁进行保证互斥/同步，所以**全局G队列是有互斥锁进行保护的**。当并发量比较大的时候，这把大锁就成了性能瓶颈



缺点：

1. 创建、销毁、调度G都需要每个M获取锁，这就形成了**激烈的锁竞争**。

2. M转移G会造成**延迟和额外的系统负载**。比如当G中包含创建新协程的时候，M创建了G’，为了继续执行G，需要把G’交给M’执行，也造成了很差的局部性，因为G’和G是相关的，最好放在M上执行，而不是其他M'。

3. 系统调用(CPU在M之间的切换)导致频繁的线程阻塞和取消阻塞操作**增加了系统开销**。



## 3. GMP调度器

老的调度器存在诸多不足，于是就弃用了，Golang重新设计了调度器，在GM之间引进了`P`

P的加入，还带来了一个**本地协程队列**，跟前面的**全局队列**类似，也是用于存放可运行的G，M想要运行G的时候，会优先从本地队列拿，访问本地队列当然是无需加锁了。全局协程队列依然是存在的，但是功能被弱化了，不到万不得已的时候是不会去全局队列里拿G的

> **M想要运行G，就得先获取P，然后从P的本地队列获取G**

最终形成了GMP模型：

![GMP模型](picture/go语言要点/GMP模型.jpg)

1. **全局队列**：存放等待运行的 G。
2. **P 的本地队列**：同全局队列类似，存放的也是等待运行的 G，存的数量有限，` 不超过 256 个`。新建 G’时，G’优先加入到 P 的本地队列，如果本地队列满了，则会`把G'加入到全局队列(Go1.13)`。
3. **P 列表**：**所有的 P 都在程序启动时创建，并保存在数组中，最多有 GOMAXPROCS(可配置) 个**。
4. **M**：线程想运行任务就得获取 P，从 P 的本地队列获取 G，P 队列为空时，M 也会尝试从全局队列拿一批 G 放到 P 的本地队列，或从其他 P 的本地队列偷一半放到自己 P 的本地队列。M 运行 G，G 执行之后，M 会从 P 获取下一个 G，不断重复下去

![GMP](picture/go语言要点/GMP的本地队列.gif)



**新建G时**，G会优先加入到P的本地队列，如果本地队列满了，则会把本地队列中一半的G移动到全局队列。当P的本地队列为空的时候，就从全局队列去拿G：

![GMP的全局队列](picture/go语言要点/GMP的全局队列.gif)



如果**全局队列也为空**，M会从其他P的本地队列**偷(stealing)一半G**放到自己P的本地队列：

![GMP-stealing](picture/go语言要点/GMP-stealing.gif)



M运行G，当G执行之后，M会从P获取下一个G，不断重复下去



### 3.1 P和M的个数

**P的数量：**

启动时，由环境变量`GOMAXPROCS`或者是由 runtime 的方法 `GOMAXPROCS()` 决定。

这意味着在程序执行的任意时刻都只有 `GOMAXPROCS` 个 goroutine 在同时运行



**M的数量**

go语言的限制：go程序启动的时候，会设置M的最大值为10000，但是内核很难支持这么多线程，这个限制可忽略

runtime/debug中的 `SetMaxThreads` 函数，设置M的最大数量

**一个M阻塞了，会创建新的M**



**M的数量和P的数量没有绝对关系**，一个M阻塞，Ｐ就会取创建或者切换到另一个Ｍ，所以，即使Ｐ的数量是１，也有可能创建很多个M出来



### 3.2 P和M的创建时机

在确定了P的最大数量n后，runtime会根据这个值直接创建n个P出来

如果没有足够的M来关联P以运行G。比如所有的M阻塞了，P还有就绪任务G，P就会取寻找空闲的M，如果没有空闲的，就会创建新的M



### 3.3 调度器的设计策略

**复用线程**： 避免频繁的创建和销毁线程，而是直接对线程的复用

1. `work stealing机制`：当M**没有G可运行时**，会依次按照以下规则来窃取，而不是销毁线程
   1. 从P的**本地队列**中获取
   2. 从**全局队列**中获取
   3. 从网络轮询器(network poller)中获取
   4. 从**其他M的P的本地队列**中获取（从其他P的本地队列偷走一半的G）
2. `hand off机制`：当本线程的**G阻塞时**，会释放绑定的P，将P转移为其他空闲的线程执行

例如下面的例子，main函数在P1上运行并创建goroutine，当第一批goroutine进入P1的本地队列的时候，P0正在寻找任务。然而它的本地队列，全局队列和网络轮询器都是空的，所以最后就从P1中窃取任务，从这7个goroutine中偷走了4个，其中一个立马在P0执行了，剩余的3个放到P0的本地队列

![任务窃取](picture/go语言要点/9ca004b0ad5d83eccaed14ed671e20bd.png)

**利用并行**：`GOMAXPROCS` 设置 P 的数量，最多有 GOMAXPROCS 个线程分布在多个 CPU 上同时运行。GOMAXPROCS 也限制了并发的程度，比如 GOMAXPROCS = 核数/2，则最多利用了一半的 CPU 核进行并行



**抢占：** 在coroutine中要等待一个协程主动让出cpu才执行下一个协程。在go中，**一个goroutine最多占用CPU 10ms**，防止其他goroutine被饿死，这就是goroutine不同于coroutine的一个地方



**全局G队列**：在新的调度器中依然有全局G队列，但是功能被弱化了，当M执行work stealing 从其他P偷不到G时，它可以从全局队列中获取G



### 3.4 go func()调度流程

![GMP-gofunc调度流程](picture/go语言要点/GMP-gofunc调度流程.png)

1. go func() 创建一个 goroutine
2. 新创建的G会优先保存到P的本地队列中，如果本地队列满了则会保存到全局队列
3. G只能运行在M中，一个M必须持有P才能运行P中的G，P的本地队列为空的时候就会触发`work stealing`机制
4. 一个M调度G执行的过程是循环的，不断的从P中取出G执行
5. 如果M执行某一个Ｇ的时候发生了系统调用或其他阻塞操作，**M会阻塞，runtime会把这个M从P中摘除，然后去取出一个空闲的Ｍ来服务这个P (如果没有空闲M则创建M)**
6. 当M的系统调用完成后，会**尝试获取一个空闲的P，把G加入到P的本地队列。如果获取不到P，那么这个M就会休眠，加入到M的空闲队列，然后这个G会被放入全局队列中**



### 3.5 调度的生命周期，M0和G0

![GMP-调度的生命周期](picture/go语言要点/GMP-调度的生命周期.png)

**M0**：`M0`是go程序启动后的编号为0的主线程，这个M对应的实例在全局变量 `runtime.m0` 中，不需要在heap上分配，M0负责执行初始化操作和启动第一个G，即`G0`，之后M0就和其他M一样了

**G0**：`G0`是每次启动一个M都会第一个创建的goroutine，G0仅用于负责调度G，不指向任何可执行的 函数，**每个M都会有一个自己的G0**。在调度或者系统调用的时候会使用G0的栈空间，全局变量的G0是属于M0的G0



## 4. Go调度器场景解析

### 4.1 创建G

P 拥有 G1，M1 获取 P 后开始运行 G1，G1 使用 `go func()` 创建了 G2，为了局部性 G2 优先加入到 P1 的本地队列。

![GMP-场景1](picture/go语言要点/GMP-场景1.png)

### 4.2 本地切换G

G1 运行完成后 (函数：`goexit`)，M 上运行的 goroutine 切换为 **G0**，**G0 负责调度时协程的切换**（函数：`schedule`）。从 P 的本地队列取 G2，从 G0 切换到 G2，并开始运行 G2 (函数：`execute`)。实现了线程 M1 的复用。

![GMP-场景2](picture/go语言要点/GMP-场景2.png)



### 4.3 本地队列满了

G2想要创建6个G，假设P1的本地队列只能存放4个G，这时候，G3，G4，G5，G6都成功放下了，G7加入的时候发现P1的本地队列满了，于是执行**负载均衡，把P1中本地队列中的`前一半的G打乱顺序`，还有新创建的`G7`一起转移到全局队列**

![GMP-场景3](picture/go语言要点/GMP-场景3.png)

这时候，G2还要创建G8，**G8这时候P1的本地队列已经不满了**，于是直接加入到了P1的本地队列！！！

![GMP-场景3-2](picture/go语言要点/GMP-场景3-2.png)

### 4.4 创建G会唤醒MP

规定：**在创建 G 时，运行的 G 会尝试唤醒其他空闲的 P 和 M 组合去执行**。

![GMP-场景4](picture/go语言要点/GMP-场景4.png)

假设G2唤醒了M2，M2绑定了P2，开始运行G0，但是P2本地队列没有G，M2此时为自旋线程，不断的寻找G



### 4.5 从全局队列取G

由于`work stealing`机制，M2 尝试从全局队列 (简称 “GQ”) 取一批 G 放到 P2 的本地队列（函数：`findrunnable()`）。M2 从全局队列取的 G 数量符合下面的公式：

```go
n = min(len(GQ)/GOMAXPROCS + 1, len(GQ/2))
```

即：至少要从全局队列取出1个G，但是每次不要从全局队列移动太多的G到P的本地队列(不超过一半)，要给其他P留点，这时从全局队列到P本地队列的负载均衡

假设一共有4个P（`GOMAXPROCS=4`），那么我们允许最多能有4个P供M使用。所以M2只能从全局队列取出1个G移动到P2，然后完成从G0到G3的切换，运行G3

![GMP-场景5](picture/go语言要点/GMP-场景5.png)



### 4.6 从其他P窃取G

假设 G2 一直在 M1 上运行，经过 2 轮后，M2 已经把 G7、G4 从全局队列获取到了 P2 的本地队列并完成运行，全局队列和 P2 的本地队列都空了，如下图的左半部分

![GMP-场景6](picture/go语言要点/GMP-场景6.png)

全局队列已经没有 G，那 m 就要执行 work stealing (偷取)：从其他有 G 的 P 那偷取一半 G 过来，放到自己的 P 本地队列。P2 从 P1 的本地队列**尾部取一半的 G**，本例中一半则只有 1 个 G8，放到 P2 的本地队列并执行



### 4.7 自旋线程数量的限制

G1 本地队列 G5、G6 已经被其他 M 偷走并运行完成，当前 M1 和 M2 分别在运行 G2 和 G8，M3 和 M4 没有 goroutine 可以运行，M3 和 M4 处于自旋状态，它们不断寻找 goroutine

为什么要让 m3 和 m4 自旋，自旋本质是在运行，线程在运行却没有执行 G，就变成了浪费 CPU. 为什么不销毁现场，来节约 CPU 资源。因为创建和销毁 CPU 也会浪费时间，我们希望当有新 goroutine 创建时，立刻能有 M 运行它，如果销毁再新建就增加了时延，降低了效率。当然也考虑了过多的自旋线程是浪费 CPU，所以系统中**最多有 GOMAXPROCS 个自旋的线程** (当前例子中的 GOMAXPROCS=4，所以一共 4 个 P)，多余的没事做线程会让他们休眠（`notesleep()`）

![GMP-场景7](picture/go语言要点/GMP-场景7.png)



### 4.8 G阻塞的情况

假定当前除了 M3 和 M4 为自旋线程，还有 M5 和 M6 为空闲的线程 (没有得到 P 的绑定，注意我们这里**最多就只能够存在 4 个 P，所以 P 的数量应该永远是 P<=M**, 大部分都是 **M 在抢占需要运行的 P**)

G8 创建了 G9，G8 进行了阻塞的系统调用，**M2 和 P2 立即解绑**，P2 会执行以下判断：如果 P2 本地队列有 G 或全局队列有 G 或有空闲的 M，P2 都会立马唤醒 1 个 M 和它绑定，否则 P2 则会加入到空闲 P 列表，等待 M 来获取可用的 P。

本场景中，P2 本地队列有 G9，可以和其他空闲的线程 M5 绑定

![GMP-场景8](picture/go语言要点/GMP-场景8.png)





### 4.9 非阻塞的系统调用

G8 创建了 G9，假如 G8 进行了**非阻塞系统调用**。(CGo会是这种方式，见`cgocall()`)

![GMP-场景9](picture/go语言要点/GMP-场景9.png)

 M2 和 P2 会解绑，但 M2 会记住 P2，然后 G8 和 M2 进入系统调用状态。当 G8 和 M2 退出系统调用时，会尝试获取 P2，如果无法获取，则获取空闲的 P，如果依然没有，G8 会被记为可运行状态，并加入到全局队列，M2 因为没有 P 的绑定而变成休眠状态 (长时间休眠等待 GC 回收销毁)。

### 4.10 柔和抢占

Go调度在go1.12实现了抢占，应该更精确的称为**请求式抢占**，因为go调度器的抢占很柔和，不会说时间片到了或者更高优先级的任务到了就直接执行抢占调度，**go的抢占柔和到只给goroutine发送1个抢占请求，至于goroutine何时停下来，那就不管了**

发出抢占请求需要满足两个条件之一：

1. 当前G进行系统调用超过20us
2. G运行时间超过10ms

调度器在启动的时候会启动一个单独的线程`sysmon`，它负责所有的监控工作，其中一项就是抢占，发现满足抢占条件的G时，就发出抢占请求

