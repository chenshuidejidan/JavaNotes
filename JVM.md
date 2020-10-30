# 1. JVM简介
## 1.1 跨平台的语言和跨语言的平台
![java从编码到执行](https://img-blog.csdnimg.cn/20200622135255828.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzQyNDgzMzQx,size_1,color_FFFFFF,t_70)

![JVM](https://img-blog.csdnimg.cn/20200622135505663.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzQyNDgzMzQx,size_1,color_FFFFFF,t_70)

- **JVM与Java独立不相关，其他语言也可以通过特定的编译生成class字节码文件，在JVM上运行** 

- **JVM是一种规范**，是虚构出来的一台计算机：字节码指令集(汇编语言)、内存管理(堆、栈、方法区等)

- **常见JVM的实现：** HotSpot(oracle官方)、Jrockit(BEA，曾号称世界上最快的JVM，后被oracle收购，合并到hotspot)、J9(IBM)、Microsoft VM、TaobaoVM(HotSpot的深度定制版)、LiquidVM(直接针对硬件)、azul zing(最新垃圾回收的业界标杆)


## 1.2 JVM,JRE,JDK
- JDK = JRE + development kit
- JRE = JVM + core lib

![wOvzYF.png](https://s1.ax1x.com/2020/09/22/wOvzYF.png)

- `JRE`：JRE是指java运行环境。**光有JVM还不能运行class文件**，因为在解释class的时候JVM需要调用解释所需要的类库lib。在JDK的安装目录里你可以找到jre目录，里面有两个文件夹bin和lib,在这里可以认为bin里的就是jvm，lib中则是jvm工作所需要的类库，而jvm和 lib和起来就称为jre
- `JDK`：JDK是java开发工具包。在目录下面有六个文件夹、一个src类库源码压缩包、和其他几个声明文件。其中，真正在运行java时起作用的是以下四个文件夹：`bin、include、lib、 jre`。现在我们可以看出这样一个关系，JDK包含JRE，而JRE包含JVM。
  - bin:最主要的是编译器(javac.exe)
  - include:java和JVM交互用的头文件
  - lib：类库
  - jre:java运行环境（注意：这里的bin、lib文件夹和jre里的bin、lib是不同的）

## 1.3 JVM整体结构
- JVM整体结构：
![JVM整体结构](https://s1.ax1x.com/2020/10/26/BK2hZj.png)
- Java代码执行流程
![Java代码执行流程](https://s1.ax1x.com/2020/10/26/BK24ds.png)
JIT编译器可以对反复执行的热点代码直接编译为机器指令

## 1.4 基于栈的指令集架构
- 由于**跨平台性**的设计，**java的指令都是根据栈来设计的**，不需要硬件支持，避开了寄存器分配难题，可以实现跨平台。不同平台CPU架构不同，所以不能设计成基于寄存器的。
- **优点：跨平台，指令集小，编译器容易实现**
- 缺点：性能下降，实现同样的功能需要更多的指令

## 1.4 JVM的生命周期
**JVM的启动**：JVM的启动是由`引导类加载器(bootstrap class loader)`创建一个`初始类(initial class)`来完成的，这个类由虚拟机的具体实现来指定，不同虚拟机不一样

**JVM的执行**：运行中的JVM的任务是`执行java程序`，因此程序开始执行时JVM才运行，程序结束时JVM就停止。**执行java程序的时候，真真正正在执行的是JVM进程**

**JVM的退出**：正常执行结束；程序异常终止；操作系统错误；**某线程调用Runtime或System的exit方法，或者Runtime的halt方法**



# 2. Class文件格式
- 魔数：每个Class文件的头4个字节被成为魔数(Magic Number)，它的作用是确定这个文件是为一个能被JVM接收的Class文件：0xCAFEBABE
- 版本号：紧接着魔数的4个字节存储的是Class文件的版本号。前两个字节是次版本号(Minor Version)，后两个字节是主版本号(Major Version)，Java版本号从45开始，每个大版本加1(jdk1.0~1.1使用45.0~45.1)，高级版本的JDK能向下兼容以前版本的Class文件，但不能运行以后版本的Class文件。虚拟机必须拒绝执行超过其版本号的Class文件   
（IDEA可以安装BinEd插件查看字节码，安装jclasslib插件分析class文件的内容）
- 常量池：版本号之后是常量池入口，通常是占据Class文件空间最大的数据项目之一，是Class文件中第一个出现的表类型数据项目。入口需要放置一项u2类型的数据，代表常量池容量计数值(constant_pool_count)。第0个常量空出来是为了后面某些指向常量池的索引值的数据在特定情况下需要表达”不引用任何一个常量池项目“的含义
- 访问标志：常量池之后，两个字节，用于识别类或者接口层次的访问信息。包括：这个Class是类还是接口、是否是public、是否是abstract、类是否final等
![class文件结构](https://s1.ax1x.com/2020/09/22/wLPUfO.png)




# 3. 类的生命周期
- **Class文件加载过程：** 硬盘上的class文件 -> loading -> linking （verfication -> preparation -> resolution） -> initializing -> gc

![类的声明周期](https://s1.ax1x.com/2020/10/27/BQe6JS.png)

~~~java
public class Test {
    public static void main(String[] args) {
        System.out.println(T.count);
    }
}
class T {
    public static T t = new T();
    public static int count = 2;

    private T() {
        count++;
        System.out.println("--" + count);
    }
}
/** --1
 *  2
 * /
~~~
静态成员加载时，在preparation阶段被赋予了默认值：t=null, count = 0          
到Initializing阶段时被赋予初始值：t=new T()，此时count依然是0，所以count++ 值为1；然后count又被赋予初始值2

## 3.1 Loading
### 3.1.1 什么是类的加载
- 类的加载指的是**将类的.class文件中的二进制数据读入到内存中**，将其放在运行时数据区的**方法区**内，然后在堆中**创建一个java.lang.Class对象作为方法区这个类的各种数据访问入口**，用来封装方法区的数据结构。类的加载的**最终产物是堆中的Class对象**，Class对象封装了类在方法区的数据结构，并且向Java程序员提供了访问方法区内的数据结构的接口

### 3.1.2 类加载子系统的作用
- **加载.class文件的方式**
     - 从本地系统中直接加载
     - 通过网络下载.class文件
     - 从zip，jar，war等归档文件中加载.class文件
     - 运行时计算生成，如动态代理
     - 从专有数据库中提取.class文件
     - 将Java源文件动态编译为.class文件
- ClassLoader只负责class文件的加载，至于是否能运行，则由Execution Engine来决定
- 加载的信息存放到`方法区`，除了类的信息外，方法区还有`运行时常量池`，`字符串字面量`和`数字常量`


![类加载](https://s1.ax1x.com/2020/10/27/BQm9JO.jpg)
- 一开始先判断类是否加载(未加载则ClassLoader加载)，然后开始链接，初始化

### 3.1.3 类加载器的种类
BootStrap ClassLoader， Extension ClasssLoader，System ClassLoader，....三者具有层级关系，但不是继承的关系

- JVM支持两种类加载器：`引导类加载器(Bootstrap ClassLoader)`和自定义类加载器(User-Defined ClassLoader)，`Extension ClasssLoader`和`System ClassLoader`都属于自定义类加载器。(继承自ClassLoader)
- `启动类加载器BootstrapClassLoader` 是使用C/C++编写的，并不继承自`java.lang.ClassLoader`，没有父加载器，**只负责加载包名为java，javax，sun等开头的类** 
- `扩展类加载器ExtensionClassLoader` 是由java语言编写，继承自`ClassLoader`，父类是启动类加载器，**用于加载java.ext.dirs指定的目录的类库，或者从jdk安装目录的jre/lib/ext下加载类库，如果用户创建的jar放在此目录，也会自动由扩展类加载器加载**
- `系统类加载器/应用程序类加载器AppClassLoader` 继承自ClassLoader，**负责加载环境变量或者系统属性java.class.path指定路径下的类库，是程序默认的类加载器**
--------
- **自定义类** 的类加载器是系统类加载器`SystemClassLoader`
- **Java核心类** 的类加载器是引导类加载器`BootStrapClassLoader`
~~~java
public class Test {
    public static void main(String[] args) {
        //获取系统类加载器
        ClassLoader systemClassLoader = ClassLoader.getSystemClassLoader();
        System.out.println(systemClassLoader);  //sun.misc.Launcher$AppClassLoader@18b4aac2

        //获取系统类加载器的上层：扩展类加载器
        ClassLoader extClassLoader = systemClassLoader.getParent();
        System.out.println(extClassLoader);  //sun.misc.Launcher$ExtClassLoader@7f31245a

        //获取扩展类加载器的上层：BootstrapClassLoader 获取不到
        ClassLoader bootstrapClassLoader = extClassLoader.getParent();
        System.out.println(bootstrapClassLoader);  //null

        //用户自定义类的类加载器：发现就是系统类加载器
        ClassLoader classLoader = Test.class.getClassLoader();
        System.out.println(classLoader);   //sun.misc.Launcher$AppClassLoader@18b4aac2

        //Java核心类的类加载器是BootstrapClassLoader
        ClassLoader classLoader1 = String.class.getClassLoader();
        System.out.println(classLoader1);   //null
    }
}
~~~

### 3.1.4 双亲委派机制
- JVM规范允许类加载器在预料`某个类将要被使用就预先加载`(并不是要等到首次使用才加载)，如果预加载出现错误，必须`在程序首次主动使用该类时才报告错误`(LinkageError)，如果该类一直未被程序主动使用，则类加载器不会报错
- JVM采用按需加载，加载某个类时，JVM采用的是`双亲委派机制`，即**类加载器收到类加载请求时，不会自己去加载，而是把请求交给父类的加载器去处理，直到达到顶层的启动类加载器，如果无法加载，再往下委托**，是一种任务委派模式

- 在JVM中判断两个class对象是否为同一个类的两个必要条件：**全限定类名相同，类的ClassLoader也相同**，也就是说两个类对象来源于同一个Class文件，只要不是被同一个ClassLoader加载，这两个对象就是不同的
- **如果一个对象是被用户类加载器加载的，JVM会把这个类加载器的引用作为类信息的一部分保存到方法区中**，要保证类加载器不变

![双亲委派机制](https://s1.ax1x.com/2020/10/27/BQJfOK.png)

~~~java
package java.lang;

public class String {
    static{
        System.out.println("我是自定义的String类");
    }
}


public class Test {
    public static void main(String[] args) {
        java.lang.String s = new java.lang.String();
    }
}
~~~
发现没有输出，因为首先系统类加载器交给拓展类加载器，拓展类加载器交给引导类加载器，引导类加载器发现是java.lang包的String，于是就把真正的String类给加载了，就不会加载我们自定义的String类了

~~~java
package java.lang;

public class String {
    static{
        System.out.println("我是自定义的String类");
    }

    public static void main(String[] args) {
        System.out.println("111");
    }
}

//错误: 在类 java.lang.String 中找不到 main 方法, 请将 main 方法定义为:public static void main(String[] args)
//否则 JavaFX 应用程序类必须扩展javafx.application.Application
~~~

**双亲委派的优势**： 避免类重复加载，保护程序安全，防止核心API被随意篡改
**破坏双亲委派机制**：

## 3.2 Linking
- Linking分为三个步骤：格式校验verfication、静态变量赋予默认值preparation、解析resolution
### 3.2.1 verfication
确保Class文件的字节流包含信息复合当前虚拟机的要求，保证被加载类的正确性，不会危害虚拟机自身安全

- 主要包括四种验证：文件格式验证，元数据验证，字节码验证，符号引用验证


### 3.2.2 preparation 半初始化

- 未变量分配内存，并设置为默认初始值，即零值
- **不包含用final修饰的static变量**，因为final常量在编译的时候就会分配了
- 不会为实例变量初始化

静态变量赋予默认值
- 举例：Double Check Singleton
~~~java
public class Singleton {
    private Singleton() {}  //私有构造函数
    private volatile static Singleton instance = null;  //单例对象
    //静态工厂方法
    public static Singleton getInstance() {
        if (instance == null) {      //双重检测机制
            synchronized (Singleton.class){  //同步锁
                if (instance == null) {     //双重检测机制，避免第一次判空的线程拿到锁后new出对象
                    instance = new Singleton();
                }
            }
        }
        return instance;
      }
}
~~~
**高并发时的问题：** 假设单例对象中有变量 count，初始值设为1000，线程A首先判断单例对象为null，创建对象，但是count的初始化分为两步            
- 第一步在preparation的半初始化阶段赋予默认值count=0
- 第二步在Initializing初始化时才被赋予初始值count=1000
如果线程A在第一步时，线程B来了，判断instance!=null，于是直接返回了count=0的instance，发生了错误

- **volatile防止该变量初始化时指令重排**，确保引用指向内存前实例初始化完毕，而可见性已经由synchronized保证了(https://blog.csdn.net/FU250/article/details/79721197)     
- 其实`instance = new Singleton()` 可以拆分成三部分：
   - a.`new #2 <T>`分配对象的内存空间，**半初始化**对象(java中申请内存就会进行默认初始化)
   - b.`invokespecial #3 <T.<init>>` 初始化，调用了构造方法
   - c.`astore_1` 建立关联，将引用指向对象的地址        
- a—>b—>c顺序执行不会有什么问题，但是如果JVM和CPU把指令顺序优化为a—>c—>b，当执行完a,c后，可能另一个线程在第一次判断singleton=null，但此时不为空了(已被赋予默认值)，不用进入synchroniezd，于是就**将未初始化完毕的instance对象返回**了

### 3.2.3 resolution
- 静态解析，**将常量池内的符号引用转换为直接引用**的过程
符号引用就是一组符号来描述所引用的目标，直接引用就是直接指向目标的指针、相对偏移量或一个间接定位到目标的句柄
- 解析动作主要针对类或接口、字段、类方法、接口方法、方法类型等。对应常量池中的`CONSTANT_Class_info`、`CONSTANT_Fieldref_info`、`CONSTANT_Methodref_info`等


## 3.3 Initializing
静态变量赋予初始值
- 初始化阶段就是执行**类的构造器方法**`<clinit>()`的过程
- 此方法是javac编译器自动收集类中的所有`类变量(静态变量)的赋值动作`和`静态代码块中的语句`合并而来。没有类变量和静态代码块就不会产生`<clinit>()`
- `<clinit>()`不同于类的构造器`<init>`
- 若该类具有父类，JVM会保证子类的`<clinit>()`执行前，父类的`<clinit>()`已经执行完毕
- JVM必须保证一个类的`<clinit>()`方法在多线程下被同步加锁，**保证一个类只被加载一次**



# 4 JVM内存模型-运行时数据区

![JVM内存模型](https://s1.ax1x.com/2020/10/26/BKNUWn.png)

## 4.1 PC寄存器
- 唯一一个在JVM规范中没有规定OOM情况的区域
- 没有GC
- 记录指令地址，便于线程的上下切换

## 4.2 虚拟机栈
- 每个线程创建的时候都会创建一个虚拟机栈，其内部保存着一个个的`栈帧(Stack Frame)`，对应着一次次的Java方法调用，是线程私有的
- JVM栈中保存：**局部变量(基本数据类型+引用数据类型的地址)，部分结果，方法的调用和返回**
- **栈的访问速度仅次于PC**
- 对于栈来说，不存在GC，但是有可能会OOM
- **栈中可能的异常**： 
  - `StackOverflowError`：采用固定大小的虚拟机栈，线程请求分配的栈容量超过java虚拟机栈允许的最大容量
  - `OutOfMemeryError`：采用动态扩展的虚拟机栈，线程尝试扩展时无法申请到足够的内存，或者线程创建时没有足够内存去创建对应的虚拟机栈
    - 设置栈的大小：`-Xss256k` 设置线程的最大栈空间，决定了函数调用的最大可达深度

### 4.2.1 栈帧
- JVM栈中的数据都是以栈帧(Stack Frame)的格式存在，**线程上正在执行的每个方法都各自对应一个栈帧**
- **不同线程所包含的栈帧不可以相互引用**
- 栈帧的结构：
  - **局部变量表**
  - **操作数栈（或表达式）**
  - 动态链接（或指向运行时常量池的方法引用）
  - 方法返回地址（或方法正常退出或异常退出的定义）
  - 一些附加信息

![栈帧](https://s1.ax1x.com/2020/10/27/Ble5Ax.png)

### 4.2.2 局部变量表
- 也称为局部变量数组，或者本地变量表。定义为一个`数字数组`，存储`方法参数和方法体内的局部变量`，数据的类型是基本数据类型，对象的引用和 returnAddress类型。
- **局部变量表的基本存储单元**是`Slot`，`32位以下的类型占用一个slot(也包括returnAdderss)，64位类型占用两个slot(long和double)`，另外byte、short、char都被转换为int存储，boolean也被转换为int 0，1
- JVM为每个Slot分配了访问索引，**方法调用时，方法参数和局部变量按声明顺序被复制到局部变量表的每个Slot**
- 如果当前帧是`构造方法或者实例方法`，Slot的`index0`会存放`该对象的引用this`，其余参数按顺序继续存放。故而静态方法不能用this，因为栈帧中根本就没有这个变量
- **Slot可以重复利用：** 当局部变量过了作用域后，重新声明的新局部变量就可以复用过期局部变量的槽位，节省资源
- 局部变量表的大小在编译期就确定了，保存在`code`属性的`maximum local variables`数据项中
- 当方法调用结束后，随着方法栈帧的销毁，局部变量表也随之销毁
- **局部变量表中的变量是重要的垃圾回收根节点，被局部变量表中直接或间接引用的对象不会被回收**

![局部变量表](https://s1.ax1x.com/2020/10/27/Bldfrn.png)


### 4.2.3 操作数栈
- `操作数栈`**用于保存计算过程的中间结果，同时作为计算过程中变量临时的存储空间**。
- 在方法执行过程中，根据字节码指令，往栈中写入数据或者提取数据，即入栈出栈。某些指令将值压栈，由其他指令将操作数取出栈，运算后再把结果压栈
- **操作数栈的深度**在编译期就确定好了，保存在`Code`属性的`max_stack`数据项中
- 和slot类似，32bit的类型占用一个栈深度，64bit的类型占用两个栈深度
- 对于`byte`范围内的数，直接按`bipush`单字节入栈，`short`类型就按`sipush`双字节入栈，但是存入局部变量表中都是int了
- 由于**JVM操作码是一个字节的零地址指令**，这意味着指令集的操作码总数不可能超过256条,大部分指令都没有支持 byte、char 和 short 类型，甚至没有任何指令支持 boolean 类型。编译器会在编译器或运行期将 byte 和 short 类型带符号扩展为 int 类型， boolean 和 char 类型零位扩展为相应的 int 类型，
**因此，大多数对于 boolean、byte、char 和 short 类型数据的操作，实际都是使用 int 类型作为运算类型**
![操作数栈演示](https://s1.ax1x.com/2020/10/28/B1EUsA.png)

### 4.2.4 动态链接
- 动态链接就是**指向运行时常量池的方法引用**
- 每个栈帧内部包含一个指向该方法所在类的`运行时常量池`的引用，目的是为了支持当前方法的代码能够实现`动态链接`
- 在java源文件被编译到字节码文件时，所有的`变量和方法引用`都作为`符号引用(symbolic reference)`保存在class文件的常量池中，**动态链接的作用就是将这些符号引用转换为调用方法的直接引用**
- **符号引用**通常是用文本/字符串形式来表示的引用关系
- **直接引用**就是JVM所能`直接使用`的形式(指向目标的指针、相对偏移量或一个间接定位到目标的句柄)
- 方法区中的常量池就是为了提供一些符号和常量，便于指令的识别

![动态链接举例](https://s1.ax1x.com/2020/10/28/B1Lbon.png)

### 4.2.5 方法的调用
- JVM中，将符号引用转换为调用方法的直接引用与方法的绑定机制相关
- **静态链接**：字节码文件装载进JVM，如果被调用的`目标方法在编译期可知，且运行期保持不变`，这时将调用方法的符号引用转换为直接引用的过程就是静态链接
- **动态链接**：被调用方法在编译期无法被确定下来，也就是`只能在运行期将调用方法的符号引用转换为直接引用`，这种引用转换过程称为动态链接
- 面向对象的语言都具备多态特性，具备早期绑定和晚期绑定两种方式
- 类型构造器和实例构造器：

~~~java
class myclass{
  static int x;
  //java中不可以用static修饰构造方法，可以用静态代码块达到相同的目的
  static myclass(){ //类型构造器 不能有参数，不能有访问限定符（public,private等）
    x=100;//初始化类静态变量
  }

  private int y; 
  public myclass(int y){  //实例构造器，可以有参数，可以有限定符
    thix.y = y；//初始化实例成员，调用了具体实例this
  }
}


~~~
- **非虚方法**：编译期就确定具体调用版本，该版本运行时不可改变：`静态方法，私有方法，final方法，实例构造器，父类方法`
- **虚方法**：其他在运行时可以体现多态特性的方法
-----
- JVM提供了几种调用方法的指令：
  - `invokestatic`：**非虚方法**。调用静态方法，解析阶段确定唯一方法版本
  - `invokespecial`：**非虚方法**。调用`<init>`方法、`私有方法`和`父类方法`，解析阶段确定唯一方法版本
  - `invokevirtual`：调用所有虚方法，final修饰的方法也会用invokevirtual，除此之外都是虚方法
  - `invokeinterface`：调用接口方法
  - `invokedynamic`：动态解析出需要调用的方法，然后执行。java8的lambda表达式就是通过该指令调用
- 面向对象由于多态特性，会频繁的使用动态分派，如果每次动态分派都要重新在类的方法元数据中搜索到合适的目标的话就会影响到执行效率。
因此为了提高性能，JVM在类的方法区建立了`虚方法表`，表中存放着各个方法的实际入口，使用索引表来替代查找

### 4.2.6 方法返回地址
- **存放该方法的调用者的pc寄存器的值**
- **方法正常退出时**，调用者的pc寄存器的值作为返回地址，这样就可以返回到该方法被调用的位置继续执行
- **方法异常退出时**，返回地址通过异常表来确定，栈帧中一般不会保存这部分信息
- 字节码指令中，返回指令包含 `ireturn`(boolean-int), `lreturn`, `freturn`, `dreturn`, `areturn`(引用类型), `return`(void方法，构造方法)

### 4.2.7 附加信息
根据JVM的具体实现而定，可选项，可能包含对程序调试者提供支持的信息

## 4.3 本地方法栈

- `native方法`就是Java调用非Java代码的接口。本地接口的作用是融合不同的语言为java所用
- `Java虚拟机栈`用于管理Java方法的调用，而`本地方法栈`用于管理本地方法的调用
- 本地方法栈也是线程私有的
- 可以设计成固定大小或者可扩展大小。内存溢出方面和java虚拟机栈的情况是一样的，同样会出现StackOverFlowError和OutOfMemoryError
- 当线程调用一个本地方法时，他就进入了一个全新的**不再受虚拟机限制**的世界，它和虚拟机拥有同样的权限！
  - 本地方法可以通过本地方法接口来`访问虚拟机内部的运行时数据区`
  - **可以直接使用本地处理器中的寄存器**
  - **可以直接从本地内存的堆中分配任意数量的内存**
- 并不是所有的java虚拟机都支持本地方法栈
- **HotSpot虚拟机中，直接将本地方法栈和虚拟机栈合二为一了**

## 4.4 堆
- **一个JVM实例只存在一个堆内存**，堆内存是java内存管理的核心区域，**在JVM启动的时候就被创建了，大小也确定了(可调节)**

堆在物理上可以不连续，但是逻辑上应被视为连续的
方法结束后，堆中的对象不会立即消失，要等到gc后才会消失
- 进程的所有线程共享Java堆，但是还可以划分**线程私有的缓冲区**：`TLAB (Thread Local Allocation Buffer)`
- jdk1.7及之前，堆分为：`新生代(Eden:Survivor0:Survivor1=8:1:1)，老年代`，`永久代`属于方法区
- jdk1.8及之后，**堆分为：新生代，老年代**。`元空间`属于方法区

![堆](https://s1.ax1x.com/2020/10/29/BJ1tIS.png)

### 4.4.1 设置堆的大小
- 设置堆区起始内存大小：`-Xms`或者`-XX:InitialHeapSize`，默认大小是：电脑物理内存大小/64
- 设置堆区最大内存大小：`-Xmx`或者`-XX:MaxHeapSize`，默认大小是：电脑物理内存大小/4

设置起始堆大小是只包含新生代和老年代的，不包含永久代/元空间
一旦堆区内存大小超过设置的最大内存大小，就会出现OOM     
- 通常将起始内存和最大内存设置相同的值，`不需要在gc清理完堆后重新计算调整堆区大小`，从而提高性能

**查看堆大小相关命令：**
- `jps`：查看java进程的pid
- `jstat -gc pid`：查看pid的内存信息
- 或者直接添加运行参数`-XX:+PrintGCDetails`，在程序执行完显示

![设置堆大小](https://s1.ax1x.com/2020/10/29/BG9Tw6.png)
![查看堆大小2](https://s1.ax1x.com/2020/10/29/BGPJPS.png)

### 4.4.2 年轻代和老年代
- JVM中的对象可分为两类：一类生命周期较短，创建消亡都很快。另一类生命周期很长。分别存放到年轻代和老年代区域
- 默认`新生代：老年代=1：2`，新生代中`Eden：from：to=8：1：1`

![堆](https://s1.ax1x.com/2020/10/29/BJ1tIS.png)

- 调整新生代和老年代的占比(一般不去调)：`-XX:NewRatio=4`，表示新生代占1，老年代占4，新生代占整个堆的1/5
  - 查看当前进程新生代和老年代的占比：`jinfo -flag NewRatio pid`
  - 设置新生代空间的大小(一般不设置，用比例就好)：`-Xmn`
- 调整新生代中Eden和Survivor的占比：`-XX:SurvivorRatio=8`，表示Eden占8，两个Survivor各占1
  - 默认有自适应比例的机制：`-XX:-UseAdaptiveSizePolicy`关闭自适应分配策略
  - 查看当前比例：`jinfo -flag SurvivorRatio pid`
- 设置晋升老年代的年龄：`-XX:MaxTenuringThreshold=<N>`，默认值是15
- **当Eden区满的时候就会触发YGC/MinorGC，将Eden和Survivor一起回收，但是Survivor区满的时候不会触发YGC！！！！！**


### 4.4.3 对象分配过程
- **当新对象太大，Eden放不下，YGC之后依然放不下，就会直接晋升到老年代区**
- **当YGC时，Survivor区不够放时，对象会直接晋升到老年代区**
- **动态对象年龄判断：** 如果Sruvior区中`相同年龄的所有对象大小总和大于Survivor空间的一半`，则年龄大于等于该年龄的对象直接进入老年代，无需达到MaxTenuringThreshold

![新对象的分配过程](https://s1.ax1x.com/2020/10/29/BGNXaq.png)

### 4.4.4 常用调优工具
- JDK命令
- Eclipse：Memory Analyzer Tool
- Jconsole
- VisualVM
- Jprofiler
- Java Flight Recorder
- GCViewer
- GC Easy

### 4.4.5 MinorGC、MajorGC、FullGC
**JVM进行GC时，并不是每次都对三个内存区域(新生代，老年代；永久代/元空间)一起回收的，大部分时候回收的都是新生代**

针对HotSpot的实现，它里面把GC按照`回收区域`分为两类：`部分收集(Partial GC)`，`完全收集(Full GC)`

- **部分收集**：不是完整收集整个堆的垃圾
  - `新生代收集(Minor GC/Young GC)`：只收集新生代的垃圾
  - `老年代收集(Major GC/Old GC)`：只收集老年代的垃圾
    - 目前只有`CMS GC`会单独收集老年代。**多数时候MajorGC会和FullGC混淆使用，要具体分辨是老年代回收还是整个堆的回收**
  - `混合收集(Mixed GC)`：收集整个新生代以及部分老年代的垃圾。目前只有`G1 GC`会有这种行为
- **整堆收集**：`Full GC`**收集整个java堆和方法区的垃圾**

### 4.4.6 TLAB
- `Thread Local Allocation Buffer(TLAB)`：由于堆区是线程共享的，为了避免多线程操作同一地址，通常需要使用加锁等机制，但是降低了效率。
**于是JVM在Eden区中为每个线程分配了一个私有的缓存区域**，使用TLAB可以避免线程安全问题，提升内存分配的吞吐量，这就是`快速分配策略`
- TLAB区域非常小，只占整个Eden区的1%，**JVM将TLAB作为内存分配的首选**，可以通过`-XX:TLABWasteTargetPercent`设置TLAB空间占Eden的百分比大小
- 一旦对象在TLAB空间分配内存失败时，JVM就会尝试使用`加锁机制`确保数据操作的原子性，从而`直接在Eden空间中分配内存`
- `-XX:UseTLAB`查看是否开启TLAB，默认是开启的

![TLAB](https://s1.ax1x.com/2020/10/29/BJJQFf.png)


### 4.4.7 堆的参数设置总结
`-XX:+PrintFlagsInitial`：查看所有的参数的默认初始值
`-XX:+PrintFlagsFinal`：查看所有参数的实际值
`-Xms`：设置初始堆空间大小(默认是物理内存/64)
`-Xmx`：设置最大堆空间大小(默认是物理内存/4)
`-Xmn`：设置新生代的大小
`-XX:NewRatio`：设置新生代和老年代的比例(老年代/新生代，默认是2)
`-XX:SurvivorRatio`：设置Eden和Survivor的比例
`-XX:MaxTenuringThreshold`：设置新生代垃圾的最大年龄
`-XX:+PrintGCDetails`：输出详细的GC日志
`-XX:HandlePromotionFailure`：是否设置空间分配担保，jdk6以前

- jdk6以前，Minor GC前会检查老年代最大可用连续空间是否大于新生代所有对象的总空间，如果小于就要进行担保机制检查
如果有担保机制再检查老年代最大可用连续空间是否大于立即晋升到老年代的对象的平均大小，大于则尝试MinorGC，否则进行FGC
![允许担保失败机制](https://s1.ax1x.com/2020/10/29/BGjYNQ.png)
- 但是JDK1.6之后，**只要Old区连续空间大于新生代对象总大小或者历次晋升的平均大小，就会进行MinorGC，否则进行FullGC**，担保失败的参数不再影响虚拟机的空间分配担保策略

### 4.4.8 堆不是对象分配的唯一选择
- 随着`JIT编译器`的发展和`逃逸分析`技术逐渐程数，`栈上分配、标量替换优化技术`有可能导致对象分配到栈上
- 如果经过`逃逸分析(Escape Analysis)`后发现，一个对象`没有逃逸出方法`的话，就可能被优化成`栈上分配`，这样就无需在堆上分配，无需进行垃圾回收了

**逃逸：** 如果对象在方法中定义后，只在`方法内部`使用，则没有发生逃逸，否则认为发生了逃逸。

**编译器对未逃逸的代码的优化**：
- **栈上分配：** JIT编译器在编译期间根据逃逸分析的结果，对于没有逃逸的对象，就可能被优化成栈上分配。(目前HotSpot只有标量替换)
- **同步省略：** JIT编译器在借助逃逸分析判断同步块所使用的`锁对象`是否`只能被一个线程访问而没有被发布到其他线程`，如果是，JIT编译器在编译这个同步代码块时就取消对这部分代码的同步，提高性能，也叫`锁消除`
- **分离对象或标量替换：** 有的对象不会被外界访问到，那么对象的部分或全部可以不存储在内存（堆），而是存储到CPU寄存器（栈）中。即`JIT优化把对象拆解成若干标量`，存储到栈上，而不会创建对象
  - 标量：不可以再拆分成更小的数据的数据。Java中原始数据类型都是标量
  - 聚合量：可以拆分为其他聚合量和标量。如Java中的类
- 逃逸分析技术还不成熟(逃逸分析本身消耗性能)，**目前HotSpot并没有栈上分配，但是有标量替换**
- **所以目前为止，Java中对象实例都是分配在堆上的**

## 4.5 方法区

### 4.5.1 堆、栈、方法区的交互
![堆栈方法区](https://s1.ax1x.com/2020/10/29/BJJKTP.png)

### 4.5.2 方法区的理解
- 《Java虚拟机规范》中提到，方法区逻辑上是堆的一部分，但是具体的实现可能不会对方法区进行垃圾回收。对于HotSpot虚拟机而言，方法区还有一个别名叫`非堆(Non-Heap)`，目的就是要和堆分开
- 方法区和堆一样，**是线程共享的内存区域**
- 方法区的实际物理内存空间和堆一样**可以是不连续的**
- 方法区的大小跟堆一样，**可以固定大小或者可扩展**
- **方法区的大小决定系统可以保存多少个类**，如果系统中定义了太多类(加载大量第三方jar包)，导致方法区溢出，同样会有OOM(PermGen或者Meta space)

jdk8以前的`永久代依然用的是JVM的内存`，容易OOM，所以jdk8使用在`本地内存中实现的元空间`替代了之前的永久代

### 4.5.3 设置方法区的大小
- jdk7设置永久代大小
  - `-XX:PermSize`设置初始大小，默认20.75MB
  - `-XX:MaxPermSize`设置最大空间大小，32位机器默认64MB，64位机器默认82MB
- jdk8设置元空间大小（一般设置最大值。。。）
  - `-XX:MetaspaceSize`默认值是21M
  - `-XX:MaxMetaspaceSize`默认值是-1，即无限制


### 4.5.4 方法区的内部结构
- **方法区存放**：`类型信息`(类，接口，枚举，注解，域信息，方法信息...)，`常量`(运行时常量池)，`静态变量`，`JIT编译后的代码缓存`等

1. **类型信息** 对每个加载的类型，JVM必须在方法区中存储以下类型信息
   - `该类型的全限定名`
   - `该类型直接父类的全限定名`，对于interface和 java.lang.Object 没有父类
   - `该类型的修饰符`，public, abstract, final
   - `该类型的直接接口的有序列表`

2. **域信息**：方法区中保存类型的所有域信息以域的声明顺序。包括域名，域类型，域修饰符
   - 要注意的是，**static的域信息在编译后不会被赋值，等到类加载的Linking的Preparation阶段赋默认值，Initializing时赋初值**
    **而static final的域信息是全局常量，编译为.class后直接就被赋予初值了**

3. **方法信息**
   - 方法名
   - 返回值类型
   - 参数的数量和类型
   - 方法的修饰符
   - 方法的字节码、操作数栈、局部变量表及大小(abstract和native方法除外)
   - 方法的异常表(abstract和native方法除外)：每个异常处理的开始位置，结束位置，处理异常的代码位置，被捕获的异常类的常量池索引
  
4. **运行时常量池**
   - **常量池**是字节码文件中的 Constant Pool：包括各种`字面量（数值，字符串值）`和`对类型、域、方法的符号引用`
   - 将字节码文件加载到方法区后，常量池加载到方法区，就是运行时常量池
   - **JVM为每个已加载的类型(类，接口)维护一个常量池**，池中的数据项像数组一样，可以直接通过索引访问(#)
   - **运行时常量池中的符号引用已经在运行期被解析为真实地址了**(动态链接)

### 4.5.5 方法区结构的演进
首先，只有HotSpot才有永久代，JRockit和J9等没有永久代，Java虚拟机规范没有管束具体方法区的实现细节

- jdk1.6及之前：有永久代，**静态变量存放在永久代上**
- jdk1.7：有永久代，**字符串常量池、静态变量从永久代移除，保存在堆空间中**
- jdk1.8之后：无永久代，**字符串常量池、静态变量保存在堆中。类型信息、字段、方法、常量保存在本地内存的元空间**

**不使用永久代的原因：**
- 永久代空间大小难确定，小了OOM，大了浪费。
- 对永久代调优困难

**字符串常量池调整到堆的原因：**
- 永久代回收效率很低，Full GC时才会触发。而Full GC是老年代的空间不足、永久代不足才会触发，这就导致经常创建的字符串常量被回收的效率不高，`放到堆里可以及时进行回收`！

**静态变量调整到堆的原因：**
- 注意：静态变量是指`引用`，对象始终都是在堆中
- 同样是为了提高回收效率吗？

![方法区演进](https://s1.ax1x.com/2020/10/29/BJLhLT.png)

### 4.5.6 方法区的垃圾回收
jvm规范并没有规定要堆方法区的垃圾进行回收，HotSpot对方法区也实现了gc。方法区中的类回收效果不好，条件苛刻
- 对常量，不再使用即可回收
- 对类而言，情况比较复杂，需要同时满足三个条件：
  - 该类的所有实例都已被回收，Java堆中不存在该类及其任何派生子类的实例
  - 该类的类加载器已经被回收(类加载器中记录了加载了哪些类)
  - 该类对应的java.lang.Class对象没有任何地方被引用，无法在任何地方通过反射访问该类的方法

## 4.5 对象实例化和内存布局

### 4.5.1 对象实例化
<center>

![对象实例化](https://s1.ax1x.com/2020/10/30/BYDzAe.png)

![对象内存布局](https://s1.ax1x.com/2020/10/30/BYgh0x.png)

![对象内存图](https://s1.ax1x.com/2020/10/30/BYhvY6.png)
</center>

# 5. Java多线程内存模型
- **每个线程由自己独立的PC，栈，本地方法栈**
- **线程间共享方法区，堆**
![多线程内存模型](https://s1.ax1x.com/2020/10/16/0bj25t.png)

- 每个Thread有一个属于自己的工作内存
- 所有Thread共用一个主内存
- 线程对共享变量的所有操作必须在工作内存中进行，不能直接操作主内存
- 不同线程间不能访问彼此的工作内存中的变量，线程间变量值的传递都必须经过主内存          

如果一个线程1对共享变量x的修改对线程2可见的话，需要经过下列步骤：       
a. 线程1将更改x后的值更新到主内存        
b. 主内存将更新后的x的值更新到线程2的工作内存中x的副本    

**JMM数据原子操作：**
1. read：从主内存读取数据
2. load：将主内存读取的数据写入工作内存
3. use：从工作内存读取数据来计算
4. assign：将计算好的值重新赋值给工作内存
5. store：将工作内存的数据写入主内存
6. write：将store过去的变量赋值给主内存中的变量
7. lock：将主内存变量加锁，标识为线程独占状态
8. unlock：将主内存变量解锁，解锁后其他线程可以锁定该变量
![JMM数据原子操作](https://s1.ax1x.com/2020/10/16/0bxOjU.png)


# 6. JVM执行引擎




# 7.  GC和GC tuning
- jvirtualvm工具，java自带，查看正在运行的java程序的资源情况
- Arthas，阿里开发的调优工具，常用
## 7.1 GC基础知识(Garbage Collector)
- **没有可以解决STW(stop the world)的垃圾收集器** 
- 常见垃圾回收器：整个Java发展过程中的10中垃圾回收器
    - 左边6种分类模型：区分新生代和老年代
    - 后面的3种不再区分新生代和老年代
    - Epsilon 用来调试jdk，生产环境一般不用

![垃圾回收器](https://s1.ax1x.com/2020/09/21/wqMZlt.png)
- G1的STW是可控的，使STW的时间最短
### 7.1.1 什么是垃圾(garbage)：
- C语言(malloc free)，C++（new delete），Java(new ?)
- Java内存自动回收，编程简单，避免：**忘记回收(内存泄漏)**，**多次回收**
- 垃圾：没有任何引用指向的对象，或者循环指向的一堆对象

### 7.1.2 如何找到垃圾
- **引用计数(Reference Count)**：记录指向对象的引用数量，不能解决循环指向的问题
- **根可达算法(Root Searching)**：通过一系列名为”GC Roots”的对象作为起始点，从这些节点开始向下搜索，搜索所走过的路径称为引用链(Reference Chain)，当一个对象到GC Roots没有任何引用链相连时，则证明此对象是垃圾
  - **根对象(GC roots)：** **线程栈变量**(JVM stack引用的对象)、**静态变量**(方法区中类静态属性引用的对象)、**常量池**(方法区中常量引用的对象)、**JNI指针**(本地方法栈中Java Native Interface引用的对象)
### 7.1.3 常见的GC算法
- **标记清除(Mark-Sweep)：** 直接标记区域为垃圾，表示内存区域可用。特点：**位置不连续，产生碎片**
- **拷贝(Copying)：** 内存一分为二，需要回收时将前一半里的存活对象拷贝到另一半，清除前一半。特点：**没有碎片，位置连续，但是浪费空间(同一时间只能用到一半的空间)**
- **标记压缩(Mark-Compact)：** 特点：**没有碎片，但效率较低(每块内存的移动都要进行线程同步)**

### 7.1.4 JVM分代模型(jdk1.8)
部分垃圾回收器使用的模型
- **新生代** young(1)：eden区(8)+survivor(1)+survivor(1)
  - 由于**YGC**需要经常回收垃圾，使用了**拷贝算法**回收垃圾（第一次YGC将eden中数据拷贝到Survivor0，第二次YGC将eden+Survivor0中存活的对象拷贝到survivor1，第三次YGC将eden+Survivor1中存活对象拷贝到survivor0）
  - 每回收一次，年龄增长，年龄足够则进入老年代区
  - new对象时，eden区空间不够(单个对象超过eden区大小的一半)，直接进入到老年代区，同理回收时对象超过Survivor区的一半也会直接进入老年代
  - survivor区空间不够时直接进入老年代区
- **老年代** ol-d(3)：
  - 老年代满了之后进行**FGC(Full GC)**，FGC需要将整个内存区域进行回收，会产生**停顿现象STW**(stop the world)，内存越大停顿时间越长，应该尽量避免FGC
  - **GC Tuning 的目标就是尽量减少FGC**
  - MinorGC=YGC
  - MajorGC=FGC
- 永久代 Permanent(1.7)、**元数据区** Metaspace(1.8)：都是存放Class对象，区别主要是永久代必须指定大小限制(可能出现内存溢出)，但是元数据区是无上限的(可以设置也可以不设置)


## 7.2 常见的垃圾回收器
- **jdk1.8 默认的垃圾回收器是：Parallel Scavenge + Parallel Old** 
![垃圾回收器](https://s1.ax1x.com/2020/09/21/wqMZlt.png)
### 7.2.1 Serial
- a stop-the-world, copying collector which uses a single GC thread.
- 运行一段时间后，STW停顿，进行垃圾回收，之后继续运行，一段时间后再进行STW
- **应用于年轻代，串行回收**

### 7.2.2 Parallel Scavenge
- a stop-the-world, copying collector which uses multiple GC threads.
- **应用于年轻代，多线程并行回收**

### 7.2.3 ParNew
- a stop-the-world, copying collector which uses multiple GC threads.
- **应用于年轻代，配合CMS的并行回收**，Parallel Scavenge不能配合CMS，于是有了ParNew

### 7.2.4 Serial Old
- 将Serial 应用到老年代区
### 7.2.5 Parallel Old
- 将Parallel Scavenge 应用到老年代区
- Serial Old 和 Parallel Old 的 FGC 可以长到几个小时....于是有了CMS提高STW时间
### 7.2.6 CMS (ConcurrentMarkSweep)
- 垃圾回收器和应用程序在不同的线程中同时运行
- **应用于老年代，并发的，垃圾回收和应用程序同时运行，降低STW的时间(200ms以内)**

### 7.2.7 G1 (Garbage First)
![G1](https://s1.ax1x.com/2020/09/21/wqGjEV.png)
- **把堆内存分为很多个Region**，Region之间可以是不连续的
- 每个Region中都有 Eden、Survivor、Old 以及存放大对象的 Humongous
- 各个Region可以进行并发的回收
- 一个内存区域并不会是固定的 Eden 或者 Old，不再需要指定年轻代和老年代所占的比例，程序启动时默认新生代占比5%，如果new的很多很快会自动增长
- **可以对STW进行控制：** `-XX:MaxGCPauseMills 200`  默认最大响应时间200ms
- **更灵活：** 分Region 进行回收，优先回收花费时间少、垃圾比例高的Region
- Region的大小：取值(1,2,4,8,16,32M 六种)，手动指定： `-XX:G1HeapRegionSize`

### 7.2.8 ZGC
- 1ms

### 7.2.9 Shenandoah

### 7.2.10 Epsilon

## 7.3 GC Tuning
- JVM的命令行参数参考：https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html
- JVM命令的参数分类
> 标准参数：- 开头，所有的 HotSpot 都支持
> 非标准参数：-X 开头，特定版本的HotSpot支持特定命令
> 不稳定参数：-XX 开头，下个版本可能取消
> - 常用的-XX： 
> - -XX:+PrintCommandLineFlags  打印参数
> - -XX:+PrintFlagsFinal  打印最终参数值
> - -XX:+PrintFlagsInitial  打印默认参数值