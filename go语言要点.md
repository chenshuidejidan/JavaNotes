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



### 3.4 变量的生命周期

- 包级别声明的变量的生命周期和整个程序的生命周期一致
- 局部变量有动态的生命周期：从创建该局部变量起，到该变量不再有引用为止。。因此局部变量可能超出其局部作用域，在函数返回之后依然存在。。**对于逃逸的局部变量，必须分配在堆上**，对于没有逃逸的局部变量，编译器可以选择分配在栈上。（也可以分配在堆上由gc进行回收）

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

文本字符串通常被解释成UTF-8编码的Unicode码点（rune）序列

字符串是**不可改变的字节序列**，len函数返回字符串的**字节数目**！所以第i个字节并不一定是字符串的第i个字符，因为对于非ASCII字符的UTF8编码会要两个或多个字节

```go
	s := "left foot"
	t := s
	s += "left foot, right foot"[9:]     //切片操作
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

**utf8解码器**可以帮助我们解码原生的utf8编码的字符串，另外range循环处理字符串的时候会隐式的解码utf8

```go
	s := "a中国"
	for i := 0; i < len(s); {
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
```

**utf8编码的字符串转换为[]rune的unicode码点序列([]int32)**：程序内部使用rune序列更方便，rune大小一致，支持数组索引和方便切割。

- 直接进行[]rune类型和string类型的转换即可
- []rune实际上就是[]int32类型

```go
	s = "a中🐶"
	fmt.Printf("% x\n", s) // "61 e4 b8 ad f0 9f 90 b6"
	r := []rune(s)
	fmt.Printf("%x\n", r) // "[61 4e2d 1f436]"
	fmt.Println(string(r)) // "a中🐶"
```

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
	b := [...]int{1, 2}
  c := [3]int{1,2}
  fmt.Println(a == b)   //true
  fmt.Println(a == c)   //Invalid operation: a == c (mismatched types [2]int and [3]int)
}
```

数组传的也是值，传递引用要传递&

```go
	a := [2]int{1, 2}	
	changeArr(a)
	fmt.Println(a)  //[1 2]

func changeArr(arr [2]int) {
	arr = [2]int{9, 9}
}
```

数组类型非常僵化，因为**数组的类型中封装了数组的长度**，只有相同长度相同类型的数组才可以进行函数的参数传递，所以很少使用，除非是对特定长度的数组进行操作。。一般使用Slice来代替数组



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



### 合并两个切片

```go
	a := [5]int{0, 1, 2, 3, 4}
	b := a[1:]
	c := a[1:2]
	c = append(c, b...)   //解构，将一个切片的所有元素追加到另一个切片里
	fmt.Println(b)  //[1 2 3 4]
	fmt.Println(c)  //[1 1 2 3 4]
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



## 3. Map

map类型可以写为map[K]V，一个map中所有的key都是相同的类型，所有的value也都是相同的类型

**key**可以是任何值，但**必须支持==操作**，**切片、函数以及包含切片的结构体**类型具有引用语义，不能作为map的key

结构体支持==操作的前提是，结构体的每个成员都支持==操作

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

## 3. 匿名函数

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

```go
//注意defer加载和传值的时间

a:=10
defer func(){fmt.Println(a)}()
defer func(a int){fmt.Println(a)}(a)
a = 100

/*
10
100
*/
```



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

- 直接调用内置的panic函数也会引发panic异常，某些不该发生的场景发生时，我们可以主动调用panic表示路径不可达

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

```go
//更标准的处理方式应该是：
defer func(){
    if err := recover(); err!=nil{
        fmt.Println(err)   //处理错误信息
    }
}()
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





# 七、Goroutines和Channels

Go语言中的并发程序可以用两种手段来实现：

1. 本章的goroutine和channel，其支持“顺序通信进程”（communicating sequential processes）或被简称为CSP。CSP是一种现代的并发编程模型，在这种编程模型中值会在不同的运行实例（goroutine）中传递，尽管大多数情况下仍然是被限制在单一实例中。
2. 覆盖更为传统的并发模型：多线程共享内存



## 1. Goroutines-并发

Go语言中，每一个并发的执行单元叫做一个goroutine

当一个程序启动的时候，其主函数即在一个单独的goroutine中运行（main goroutine），使用go 语句创建一个新的goroutine.



**修改逻辑处理器的数量**

```go
runtime.GOMAXPROCS(1)      //分配一个逻辑处理器给调度器使用
```



```go
runtime.GOMAXPROCS(runtime.NumCPU())   //分配逻辑处理器的数量=cpu核心数，每个cpu核心都分配一个逻辑处理器
```











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



**关于nil值的channel：** 向nil的channel读或者写都会被永远阻塞，所以select一个nil的channel会永远都select不到，所以可以用来**禁用或者激活case**





### goroutine并发的退出

go语言并没有提供在一个goroutine中终止另一个goroutine的方法，因为这样会导致goroutine之间的共享变量落在为定义的状态上





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



## goroutines和线程

每一个**OS线程**都有一个**固定大小的内存**块（一般会是2MB）来做栈，这个栈会用来存储当前正在被调用或挂起（指在调用其它函数时）的函数的内部变量。

一个goroutine会以一个很小的栈开始其生命周期，**一般只需要2KB**。一个goroutine的栈，和操作系统线程一样，会保存其活跃或挂起的函数调用的**本地变量**，但是和OS线程不太一样的是，一个goroutine的**栈大小并不是固定的**；栈的大小会根据需要动态地伸缩。而**goroutine的栈的最大值有1GB**





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



## 1. 测试函数

参数`-v`可用于打印**每个测试函数的名字和运行时间**

参数`-run`对应一个正则表达式，只有**测试函数名被它正确匹配的测试函数**才会被`go test`测试命令运行



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



## ...



# 十、反射



## 1. reflect.Type 和 reflect.Value

函数 reflect.TypeOf 接受任意的 interface{} 类型，并以 reflect.Type 形式返回其动态类型：

```Go
t := reflect.TypeOf(3)  // a reflect.Type
fmt.Println(t.String()) // "int"
fmt.Println(t)          // "int"
```

这里将3作为接口传递给TypeOf函数，即**将一个具体的值转为接口类型**，会有一个**隐式的接口转换**操作，它会创建一个包含两个信息的接口值：操作数的**动态类型**（这里是 int）和它的**动态的值**（这里是 3）

- reflect.Type是一个**接口类型**

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



## 2. 三大法则

 Go 语言反射的三大法则：

1. 从 `interface{}` 变量可以反射出反射对象；
2. 从反射对象可以获取 `interface{}` 变量；
3. 要修改反射对象，其值必须是可设置的；



详细介绍：

1. reflect.TypeOf() 和 reflect.ValueOf() 可以**将go语言的 interface{} 对象转化为反射对象**，他们的参数类型都是 interface{}。。所以这可能会进行隐式的类型转换。 如果我们认为 **Go 语言的类型** 和 **反射类型**处于两个不同的世界，那么这两个函数就是连接这两个世界的桥梁

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



2. 反射的第二法则是我们可以从反射对象可以获取 `interface{}` 变量。既然能够将接口类型的变量转换成反射对象，那么一定需要其他方法将反射对象还原成接口类型的变量， `reflect.Value.Interface`就能完成这项工作：

![golang-reflection-to-interface](picture/go语言要点/golang-reflection-to-interface.png)

```go
v := reflect.ValueOf(1)
v.Interface().(int)   //通过类型断言，将 interface{} 类型转化为了 int 类型
```



**无论是接口值到反射对象还是反射对象到接口值，都是需要两次类型转换的**：

![img](picture/go语言要点/golang-bidirectional-reflection.png)

当然如果变量本身就是 `interface{}` 类型的，那么它不需要类型转换，因为类型转换这一过程一般都是隐式的，所以我不太需要关心它，只有在我们需要将反射对象转换回基本类型时才需要显式的转换操作



3. 最后一条法则是与值是否可以被更改有关，如果我们想要更新一个 `reflect.Value`，那么它持有的值一定是可以被更新的

```go
func main() {
	i := 1
	v := reflect.ValueOf(i)
	v.SetInt(10)     //panic: reflect: reflect.flag.mustBeAssignable using unaddressable value
	fmt.Println(i)
}
```

由于go语言的函数调用都是**值传递**，**我们得到的反射对象和最开始的变量没有任何关系**，所以直接修改反射对象无法改变原始变量，程序为了防止错误就会崩溃

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





# 十一、标准库



## 1. log包

log.Println()  打印到标准日志记录器

log.Fatalln()  调用后会紧接着调用os.Exit(1)

log.Panicln() 调用后会紧接着调用panic()



## 2. json相关



**两种编码和解码方式：**

1. 编码
   `json.NewEncoder(<Writer>).encode(v)`    
   `json.Marshal(&v)`  返回字节切片 []byte ，为了让编码后的json更加易读，还可以使用 `MatshalIndent(&v, "", "\t")`   第一个字符串是指定前缀，后面一个字符串指定一个制表符
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



