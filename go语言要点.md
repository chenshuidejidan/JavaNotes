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
%v          变量的自然形式（natural format）
%T          变量的类型
%%          字面上的百分号标志（无操作数）
```





# 一、程序结构

## 1. 命名

函数内部定义的变量只在函数内部有效

函数外部定义的变量在整个包中都有效，变量名的开头字母大小写决定了变量在包外的可见性：对于首字母大写的变量名，它将是**导出的**，即可以被其他包访问到（函数名同理），而对于首字母小写的变量，只能在包内被访问



## 2. 声明

四种类型的生命 var、const、type、func



## 3. 变量

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

### 3.7 包和文件

对于在包级别声明的变量，如果有初始化表达式则用表达式初始化，还有一些没有初始化表达式的，例如某些表格数据初始化并不是一个简单的赋值过程。在这种情况下，我们可以用一个特殊的init初始化函数来简化初始化工作。每个文件都可以包含多个init初始化函数

```go
func init() { /* ... */ }
```

这样的init初始化函数除了不能被调用或引用外，其他行为和普通函数类似。在每个文件中的init初始化函数，在程序开始执行时按照它们声明的顺序被自动调用



初始化工作是自下而上进行的，main包最后被初始化。以这种方式，可以确保在main函数执行之前，所有依赖的包都已经完成初始化工作了。同时p包若导入了q包，p包初始化的时候就认为q包已经初始化完成了

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
func Contains(s, substr string) bool
func Count(s, sep string) int
func Fields(s string) []string
func HasPrefix(s, prefix string) bool
func Index(s, sep string) int
func Join(a []string, sep string) string
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

const声明

iota常量生成器

## 无类型常量

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



## 2. Slice

Slice（切片）代表变长的序列，序列中每个元素都有相同的类型。一个slice类型一般写作[]T，其中T代表slice中元素的类型。Slice没有固定的长度

一个slice由三个部分构成：指**针、长度和容量**。指针指向第一个slice元素对应的底层数组元素的地址，要注意的是slice的第一个元素并不一定就是数组的第一个元素。长度对应slice中元素的数目；长度不能超过容量，容量一般是从slice的开始位置到底层数据的结尾位置。内置的`len`和`cap`函数分别返回slice的长度和容量

如果切片操作超出cap(s)的上限将导致一个panic异常，但是超出len(s)则是意味着扩展了slice，因为新slice的长度会变大

和数组不同的是，**slice之间不能比较**，因此我们不能使用==操作符来判断两个slice是否含有全部相等元素。不过标准库提供了高度优化的`bytes.Equal`函数来判断两个字节型slice是否相等（[]byte），但是对于其他类型的slice，我们必须自己展开每个元素进行比较

- 为nil的Slice没有底层数组，len和cap都是0，可以进行比较。但是非nil的Slice也可以len和cap为0，与普通Slice一样

```go
var s []int    // len(s) == 0, s == nil
s = nil        // len(s) == 0, s == nil
s = []int(nil) // len(s) == 0, s == nil
s = []int{}    // len(s) == 0, s != nil
```

除了和nil相等比较外，**一个nil值的slice的行为和其它任意0长度的slice一样**；例如reverse(nil)也是安全的。除了文档已经明确说明的地方，所有的Go语言函数应该以相同的方式对待nil值的slice和0长度的slice

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



## 3. Map

map类型可以写为map[K]V，一个map中所有的key都是相同的类型，所有的value也都是相同的类型

key对应的value必须支持==操作

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

var dilbert Employee
```

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



## 5. defer

对于函数或方法前加了difer关键字的，当执行到该条语句时，函数和参数表达式得到计算，但**直到包含该defer语句的函数执行完毕时，defer后的函数才会被执行**，不论包含defer语句的函数是通过return正常结束，还是由于panic导致的异常结束。你可以在一个函数中执行多条defer语句，它们的**执行顺序与声明顺序相反**。注意**在difer之后执行完毕**才会执行difer语句

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

## 6. Panic异常

- Go的类型系统会在编译时捕获很多错误，但有些错误只能在运行时检查，如数组访问越界、空指针引用等。这些运行时错误会引起painc异常

一般而言，当panic异常发生时，程序会**中断运行**，并立即执行在该goroutine中**被延迟的函数**（defer 机制）。随后，程序**崩溃并输出日志信息**。日志信息包括panic value和函数调用的堆栈跟踪信息

- 直接调用内置的panic函数也会引发panic异常，某些不该发生的场景发生时，我们可以主动调用panic表示路径不可达

## 7. recover捕获异常





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

## 1. 基于指针的方法

1. 不管你的method的receiver是指针类型还是非指针类型，都是可以通过指针/非指针类型进行调用的，编译器会帮你做类型转换。
2. 在声明一个method的receiver该是指针还是非指针类型时，你需要考虑两方面的因素，第一方面是这个对象本身是不是特别大，如果声明为非指针变量时，调用会产生一次拷贝；第二方面是如果你用指针类型作为receiver，那么你一定要注意，这种指针类型指向的始终是一块内存地址，就算你对其进行了拷贝

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





# 六、接口

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

interface{}类型，它没有任何方法，空接口类型对实现它的类型没有要求，所以我们可以将任意一个值赋给空接口类型

```go
var any interface{}
any = true
any = 12.34
any = "hello"
any = map[string]int{"one": 1}
any = new(bytes.Buffer)
```

