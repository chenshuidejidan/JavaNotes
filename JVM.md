# 1. JVM简介
![java从编码到执行](https://img-blog.csdnimg.cn/20200622135255828.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzQyNDgzMzQx,size_1,color_FFFFFF,t_70)

![JVM](https://img-blog.csdnimg.cn/20200622135505663.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzQyNDgzMzQx,size_1,color_FFFFFF,t_70)

- **JVM与Java独立不相关，其他语言也可以通过特定的编译生成class字节码文件，在JVM上运行** 
- JVM是一种规范，是虚构出来的一台计算机：字节码指令集(汇编语言)、内存管理(堆、栈、方法区等)
- 常见JVM的实现：HotSpot(oracle官方)、Jrockit(BEA，曾号称世界上最快的JVM，后被oracle收购，合并到hotspot)、J9(IBM)、Microsoft VM、TaobaoVM(HotSpot的深度定制版)、LiquidVM(直接针对硬件)、azul zing(最新垃圾回收的业界标杆)

- JDK = JRE + development kit
- JRE = JVM + core lib
# 2. Class File format
- 魔数：每个Class文件的头4个字节被成为魔数(Magic Number)，它的作用是确定这个文件是为一个能被JVM接收的Class文件：0xCAFEBABE
- 版本号：紧接着魔数的4个字节存储的是Class文件的版本号。前两个字节是次版本号(Minor Version)，后两个字节是主版本号(Major Version)，Java版本号从45开始，每个大版本加1(jdk1.0~1.1使用45.0~45.1)，高级版本的JDK能向下兼容以前版本的Class文件，但不能运行以后版本的Class文件。虚拟机必须拒绝执行超过其版本号的Class文件   
（IDEA可以安装BinEd插件查看字节码，安装jclasslib插件分析class文件的内容）
- 常量池：版本号之后是常量池入口，通常是占据Class文件空间最大的数据项目之一，是Class文件中第一个出现的表类型数据项目。入口需要放置一项u2类型的数据，代表常量池容量计数值(constant_pool_count)。第0个常量空出来是为了后面某些指向常量池的索引值的数据在特定情况下需要表达”不引用任何一个常量池项目“的含义

![class文件结构](https://s1.ax1x.com/2020/09/22/wLPUfO.png)




# 3. Loading Linking Initializing
- **Class文件加载过程：** 硬盘上的class文件 -> loading -> linking -> initializing -> gc

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

## 3.2 Linking
- Linking分为三个步骤：格式校验verfication、静态变量赋予默认值preparation、解析resolution
### 3.2.1 verfication
格式校验
### 3.2.2 preparation 半初始化
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

- 其实`instance = new Singleton()` 可以拆分成三部分：
   - a. `memory =allocate();`分配对象的内存空间
   - b.`ctorInstance(memory);`在对象地址上初始化一个singleton对象
   - c.`instance =memory;` 将singleton引用指向对象地址； 
如果指令重排时，bc顺序反了，可能会导致某个线程获得第一个线程未初始化完毕的对象
### 3.2.3 resolution
静态解析

## 3.3 Initializing
静态变量赋予初始值
# 4. Java Memory Model




# 5. JVM的汇编指令集(JVM Instruction) 



# 6.  GC和GC tuning
## 6.1 GC基础知识(Garbage Collector)
- **没有可以解决STW(stop the world)的垃圾收集器** 
- 常见垃圾回收器：整个Java发展过程中的10中垃圾回收器
    - 左边6种分类模型：区分新生代和老年代
    - 后面的3种不再区分新生代和老年代
    - Epsilon 用来调试jdk，生产环境一般不用

![垃圾回收器](https://s1.ax1x.com/2020/09/21/wqMZlt.png)
- G1的STW是可控的，使STW的时间最短
### 6.1.1 什么是垃圾(garbage)：
- C语言(malloc free)，C++（new delete），Java(new ?)
- Java内存自动回收，编程简单，避免：**忘记回收(内存泄漏)**，**多次回收**
- 垃圾：没有任何引用指向的对象，或者循环指向的一堆对象

### 6.1.2 如何找到垃圾
- **引用计数(Reference Count)**：记录指向对象的引用数量，不能解决循环指向的问题
- **根可达算法(Root Searching)**：通过一系列名为”GC Roots”的对象作为起始点，从这些节点开始向下搜索，搜索所走过的路径称为引用链(Reference Chain)，当一个对象到GC Roots没有任何引用链相连时，则证明此对象是垃圾
  - **根对象(GC roots)：** **线程栈变量**(JVM stack引用的对象)、**静态变量**(方法区中类静态属性引用的对象)、**常量池**(方法区中常量引用的对象)、**JNI指针**(本地方法栈中Java Native Interface引用的对象)
### 6.1.3 常见的GC算法
- **标记清除(Mark-Sweep)：** 直接标记区域为垃圾，表示内存区域可用。特点：**位置不连续，产生碎片**
- **拷贝(Copying)：** 内存一分为二，需要回收时将前一半里的存活对象拷贝到另一半，清除前一半。特点：**没有碎片，位置连续，但是浪费空间(同一时间只能用到一半的空间)**
- **标记压缩(Mark-Compact)：** 特点：**没有碎片，但效率较低(每块内存的移动都要进行线程同步)**

### 6.1.4 JVM分代模型(jdk1.8)
部分垃圾回收器使用的模型
- **新生代** young(1)：eden区(8)+survivor(1)+survivor(1)
  - 由于**YGC**需要经常回收垃圾，使用了**拷贝算法**回收垃圾（第一次YGC将eden中数据拷贝到survior0，第二次YGC将eden+survior0中存活的对象拷贝到survivor1，第三次YGC将eden+survior1中存活对象拷贝到survivor0）
  - 每回收一次，年龄增长，年龄足够则进入老年代区
  - new对象时，eden区空间不够(单个对象超过eden区大小的一半)，直接进入到老年代区
  - survivor区空间不够时直接进入老年代区
- **老年代** old(3)：
  - 老年代满了之后进行**FGC(Full GC)**，FGC需要将整个内存区域进行回收，会产生**停顿现象STW**(stop the world)，内存越大停顿时间越长，应该尽量避免FGC
  - **GC Tuning 的目标就是尽量减少FGC**
  - MinorGC=YGC
  - MajorGC=FGC
- 永久代 Permanent(1.7)、**元数据区** Metaspace(1.8)：都是存放Class对象，区别主要是永久代必须指定大小限制(可能出现内存溢出)，但是元数据区是无上限的(可以设置也可以不设置)


## 6.2 常见的垃圾回收器
- **jdk1.8 默认的垃圾回收器是：Parallel Scavenge + Parallel Old** 
![垃圾回收器](https://s1.ax1x.com/2020/09/21/wqMZlt.png)
### 6.2.1 Serial
- a stop-the-world, copying collector which uses a single GC thread.
- 运行一段时间后，STW停顿，进行垃圾回收，之后继续运行，一段时间后再进行STW
- **应用于年轻代，串行回收**

### 6.2.2 Parallel Scavenge
- a stop-the-world, copying collector which uses multiple GC threads.
- **应用于年轻代，多线程并行回收**

### 6.2.3 ParNew
- a stop-the-world, copying collector which uses multiple GC threads.
- **应用于年轻代，配合CMS的并行回收**，Parallel Scavenge不能配合CMS，于是有了ParNew

### 6.2.4 Serial Old
- 将Serial 应用到老年代区
### 6.2.5 Parallel Old
- 将Parallel Scavenge 应用到老年代区
- Serial Old 和 Parallel Old 的 FGC 可以长到几个小时....于是有了CMS提高STW时间
### 6.2.6 CMS (ConcurrentMarkSweep)
- 垃圾回收器和应用程序在不同的线程中同时运行
- **应用于老年代，并发的，垃圾回收和应用程序同时运行，降低STW的时间(200ms以内)**

### 6.2.7 G1 (Garbage First)
![G1](https://s1.ax1x.com/2020/09/21/wqGjEV.png)
- **把堆内存分为很多个Region**，Region之间可以是不连续的
- 每个Region中都有 Eden、Survivor、Old 以及存放大对象的 Humongous
- 各个Region可以进行并发的回收
- 一个内存区域并不会是固定的 Eden 或者 Old，不再需要指定年轻代和老年代所占的比例，程序启动时默认新生代占比5%，如果new的很多很快会自动增长
- **可以对STW进行控制：** `-XX:MaxGCPauseMills 200`  默认最大响应时间200ms
- **更灵活：** 分Region 进行回收，优先回收花费时间少、垃圾比例高的Region
- Region的大小：取值(1,2,4,8,16,32M 六种)，手动指定： `-XX:G1HeapRegionSize`

### 6.2.8 ZGC
- 1ms

### 6.2.9 Shenandoah

### 6.2.10 Epsilon

## 6.3 GC Tuning
- JVM的命令行参数参考：https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html
- JVM命令的参数分类
> 标准参数：- 开头，所有的 HotSpot 都支持
> 非标准参数：-X 开头，特定版本的HotSpot支持特定命令
> 不稳定参数：-XX 开头，下个版本可能取消
> - 常用的-XX： 
> - -XX:+PrintCommandLineFlags  打印参数
> - -XX:+PrintFlagsFinal  打印最终参数值
> - -XX:+PrintFlagsInitial  打印默认参数值