# Java基础知识
## Java核心机制
* **java虚拟机**：解释型语言，.java编译成.class 提供给各个平台的java虚拟机翻译。**一次编译，随处运行**

* **垃圾收集机制**：java有自己的垃圾收集器，自动进行，程序员无法精确控制和干预。**提供了程序的健壮性**
## J2SDK和JRE
* J2SDK:Software Development Kit(软件开发包)。**开发需要JDK**
* JRE:Java Runtime Environment(JAVA运行环境)。**用户需要JRE运行环境**
## java程序结构
* Java源文件是.java。源文件的的基本组成是类(class)
* **一个源文件最多只能有一个public类。**其他类的个数不限，一个类对应一个.class文件。**源文件名必须和public类名一致。**
* **java程序执行的入口是main()方法。**格式：**public static void main(String args[]){...}**,args是变量名，不固定
* **java语言严格区分大小写**
* java方法有一条条语句构成，每个语句以分号结束。
## 安装与配置
* UltraEdit,jdk,eclipse,MarkdownPad
* **系统环境变量的配置**：copy jdk/bin文件夹目录(里面有javac.exe java.exe)，放path里的前面(系统找的时候从前往后)
* **查看javac版本**：cmd中输入javac -version
* path:windows系统执行命令时要搜寻的路径
* classpath:java在编译和运行时要找的class所在的路径
* UltraEdit->设置->文件处理:不备份  去掉.bak文件
##编译和运行
* 编译:  D:\>cd java   javac HelloWorld.java
* 运行:  D:\java>java HelloWorld

----
# Java基础语法
* 包：全部小写
* 类或接口：每个单词首字母大写
* 方法或变量：从第二个单词开始首字母大写
* 常量：每个字母大写，单词用_隔开
## 标识符
* 标识符由字母，下划线，美元符或者数字组成
* 标识符以字母，下划线，美元符开头
* Java标识符对大小写敏感，长度无限制
* Java标识符不能与Java语言的关键字重名
## Java常量
* 整型常量int，实型常量float，字符常量char(`a`)，逻辑常量(true false)，字符串常量string("asas") ,final(值不能改变的量)
## Java变量
* 变量使用前必须声明
* 本质上，变量是内存的一小块区域，使用变量名来访问这块区域
* **double d1,d2,d3=0.123; 表示d3=0.123，d1,d2并没有赋值**
* **程序执行过程中的内存管理**:

>1. code segment：存放代码


>1. data segment：存放静态变量、字符串常量

>1. stack(栈)：存放局部变量
>1. heap(堆)：存放new出来的东西
##Java变量的分类
* 按声明的位置划分：**局部变量**(方法或语句块内部定义的变量)、**成员变量**(方法外部、类的内部定义的变量)。**类外面不能有变量的声明**。  
**变量作用域只在它所在的那个大括号里**

* 按所属的数据类型划分：基本数据类型、引用数据类型  
### Java的数据类型
> * 基本数据型
>>数值型：整数类型(byte,short,int,long)、浮点类型(float,double)

>>字符型(char)

>>布尔型(boolean)

>* 引用数据型

>>类(class)

>>接口(interface)

>>数组  

* 整数类型  
  1. **三种表示形式**：  
      十进制整数：12  
      八进制整数：以0开头，如012。  
      十六进制整数：以0x或0X开头，如0x12。
  2. **整形常量默认为int类型，声明long时必须在常量后加'l'或者'L'如：int i1=600;long i2=88888888l;**
  3. **byte 8位1个字节(最大128-1)  short 2字节(最大32768-1)  int 4字节  long 8字节皆表示带符号的数**
* 浮点类型  
  1. **两种表示形式**：  
      十进制数：  
      科学计数法：3.14e2
  2. **浮点数默认为double型,float型后面加f或者F。double d=12345.6;  float f=12.3f;**  
     **float占4字节,double占8字节**
###数据类型的转换  
* boolean类型不可以转换为其他的数据类型
* 整形，字符型，浮点型的数据在混合运算中相互转换。  
	byte,short,char->int->long->float->double  
	**byte,short,char之间不会相互转换，计算时首先会转换成int**    
	容量大的往容量小的转换时要加强制转换符，但可能会使精度降低或溢出。
	多种类型混合运算时，系统先将所有数据转换成最大的哪一种数据类型运算  

* 强制转换:   
	**注意自动转换对变量和对常量的区别**   
	~~~java
	int i = 1,j = 2;  
	float f1 = (float)((i + j)*1.2)      
	byte b1 = 1;       
	byte b2 = 2;     
	byte b3 = (byte)(b1+b2);//系统自动将byte转换为int,  需要强制转换符  
	byte b1 += 1; //没有错，扩展的赋值运算符其实隐含了一个强制类型转换(b+=1不等价于b = b +1 而是等价于b = (b的类型)(b+1))</pre>  
	~~~
	但是如 byte a = 3+4;编译器先做3+4算出来的结果若不溢出则直接赋予a，而不会自动转换，溢出则报错。
* 强制转换时，大转小，直接丢弃高位，低位按有符号数的补码算。如  
	byte a = (bayte) 130;  00000000 00000000 00000000 10000010 变为10000010[补] 即11111110[原] = -126 
  
------
## 运算符  
- 算数运算符:+,-,*,/,%(求余),++,--
- 关系运算符:>,<,>=,<=,==,!=
- 逻辑运算符:!,&,|,^(异或),&&,||
- 位运算符：&,|,^,~,>>,<<,>>>
- 赋值运算符：=
- 扩展赋值运算符：+=,-=,*=,/=
- 字符串连接运算符:+  

> int i = (i2++);表示 i=i2,i2再加1  
int i = (++i2);表示 i2加1，i=i2   

	int a = (b = 1);//表示b=1,留下b, a=b;注意区别c语言

- **&和&&的区别**：&要将条件计算完，&&计算到第一个是false直接停止运算  

- **字符串连接符**:  
> 1. "+"可以用于字符串连接(String s = "hello"+"world";)
> 2. "+"运算符两侧的操作数只要有一个是字符串类型，系统自动将另一个转换为字符串再连接  
> 3. **打印时，无论是何种类型，都自动转换为字符串类型进行打印**    
> > int a = 12;
System.out.println("c="+c);      

## 条件语句
### if语句
* 只有一句需要执行的语句时，可以省略{}  
###switch语句  
* **不要忘了每句话后面加 break;**   
* **switch后面的条件不能是long(因为byte char等可以自动转int，而long不可以)**  
~~~java
public class TestSwitch {
	public static void main(String[] args) {
		int i = 18;
		switch(i) {
			case 4 :
			case 8 :
				System.out.println("A"); //8和4都输出A
				break;
			case 2 :
				System.out.println("B");
				break;
			case 9 :
				System.out.println("C");
				break;
			default:
				System.out.println("error"); //default提高程序健壮性
		}
	}
}
~~~

## 循环语句
### for循环语句  
* for(表达式1;表达式2;表达式3;){语句;}
* 首先执行表达式1，接着执行表达式2，表达式2是boolean为true时执行语句，再计算表达式3，再判断表达式2的值...
### while和do while语句  
* while(逻辑表达式){语句;...;}  先判断再执行，最少执行0次  
* **do{语句;...;}(逻辑表达式);** **注意最后有分号**,先执行再判断，最少执行1次  
### break和continue  
* break终止语句块，跳出循环
* contiue终止当次循环，开始下一次循环 
## 方法  
* 方法类似于其他语言的函数，声明格式:   
  **[修饰符1 修饰符2 ...] 返回值类型 方法名(形式参数列表)｛java语句;...｝**  

* **形参:**在方法被调用时用于接收外界输入的数据   

* **实参:**调用方法时实际传给方法的数据   
* **返回值:**方法在执行完毕后返还给调用它的环境的数据，根据返回值类型返回      

* **返回值类型:**事先约定的返回值的数据类型，**如无返回值，必须给返回值类型void(表示不返回值)**
* **return 语句;**  终止方法的运行，并指定要返回的数据，不再执行后面的语句  return;表示直接返回   

* java进行函数调用中传递参数时，遵循**值传递的原则**：基本类型传递的是该数据值本身。引用类型传递的是对对象的引用，而不是对象本身   
* **方法重载：**在同一个类中，允许存在一个以上的同名方法，只要它们的参数个数或者参数类型不同即可    

* **方法重载的特点：**  
	>1. 与返回值类型无关，只看方法名和参数列表  
	>1. 在调用时，虚拟机通关参数列表的不同来区分同名方法  
## 数组
### 数组的定义：
* **动态初始化**：只定义长度，系统给出初值：**int[] array = new int[3];**   
	new为数组分配分配内存空间。  
	array是数组在堆内存的首地址值，array[i]才是数组中元素的索引     
	![avatar](https://s2.ax1x.com/2019/03/20/AKxQBT.png)  
	**系统给出的初值均为0**  

* **静态初始化**：只给出初值，系统给出长度：**int[] array = new int[]{1,2,3}** 或者 **int[] array = {1,2,3};**   
    定义时仍然被初始化为0 (new的过程)，之后立马被赋值！    
* **注意：**不要同时动态和静态进行：**int array = new int[3]{1,2,3};//错误**  

* **java数组可以互相被赋值**： int[] array2 = array; 则array和array2指向同一片堆内存区域！
* **java中的内存分配：**java程序为了调高程序的效率，对数据进行了不同空间的分配
	>1. 栈内存：存放**局部变量**(在方法定义中或者方法声明上的变量)，栈内存的数据用完就释放    
	>2. 堆内存：存放所有new出来的东西   
	堆内存的特点：   
	a. 每一个new出来的东西都有地址值   
	b. 每个变量都有默认值    
		byte,short,int,long 0   
		float,double 0.0   
		char '\u0000'    
		boolean false  
		引用类型 null  
	c. 使用完毕就变成了垃圾，但是把那个么有理解回收，会在垃圾回收器空闲的时候回收	
	>3. 方法区：
	>4. 本地方法区：(和系统相关)  
	>5. 寄存器：(CPU使用)  
	
### 数组的遍历
* 直接遍历 
~~~java
for(int i = 0; i < array.length; i++){  
       System.out.println(array[i]);     
}
~~~

* 函数遍历  
~~~java
public static void printArray(int[] array){
	System.out.print("[")
	for(int i = 0; i < array.length; i++){
		if(i == array.length - 1){
			System.out.println(array[i]+"]");
		}
		else
			System.out.print(ayyay[i]+",");
	}
}
~~~

* 二维数组的遍历 
~~~java
public static void printArray(int[][] array){
	for(int i = 0; i < array.length i++){
		for(int j = 0 j < array[i].length; j++){
			System.out.print(array[i][j]+" ");
		}
		System.out.println();
	}
}
~~~



-----

# 面向对象编程  

## 对象和类   
* **对象**用计算机语言对问题域中事物的描述，通过"属性(attribute)"和"方法(method)"来分别对应事物所具有的静态属性和动态属性    

* **类**是用于描述同一类型的对象的一个抽象概念，类中定义了这一类对象所应具有的静态和动态属性    

* 类可以看作是一类对象的模版，对象可以看成该类的一个具体实例   

* **类之间的关系**：关联、继承(..是一种..)、聚合(..是..的一部分，分聚集和组合)、实现、多态
* 对象是Java程序的核心，对象可以看成是静态属性(成员变量)和动态属性(方法)的封装体，在类中定义该类对象所应具有的成员变量及方法。 

* **匿名对象**：调用一次方法，匿名对象不必使用栈内存，调用完毕就可以被垃圾回收器回收。 new Student.show();   
* 匿名对象可以直接作为实际参数传递 new StudentDemo().method(new Student());    

* **成员变量**：在类中方法处，在**堆内存**，随对象生存，**有默认初值**   
* **局部变量**：在方法定义中或者方法声明上，在**栈内存**，方法调用完毕消失，**没有初始值**，必须赋值才能使用，**变量名可以和成员变量相同**，按就近原则。  


### 引用、对象、内存分析
* **成员变量定义时可以不初始化，默认值为0(boolean为false,引用类型为null)，局部变量不会默认初始化，必须必须初始化**  

* 对象是new出来的，位于堆内存，类的每个成员变量在不同的对象中有不同的值(除了静态变量)，而**方法只有一份，执行的时候才占用内存** 
* 引用对象的成员变量：对象(引用).成员变量  
* 调用对象的方法：对象(引用).方法  
* **同一个类每个对象有不同的成员变量存储空间，但是每个对象共享该类的方法**   

* **封装**:隐藏对象的属性和实现细节，仅对外提供公共方法来访问，提高了代码的复用性和安全性    
* 封装的常见应用：把成员变量用private修饰，提供对应的getXx()和setXx()方法  

### 构造方法(构造函数)
* 使用**new 构造方法** 创建一个新的对象
* 构造函数是定义在Java类中的一个用来初始化对象的函数
* 构造函数与类同名且**没有返回值**  
* 没有定义构造方法时，系统自动给空的无参构造方法，自定义构造方法之后系统不再自动给无参构造方法！      
* **构造方法可以重载**  

		public class Person{
			int id;
			int age;
			Person(int id,int age){
				this.id = id;
				this.age = age;
			}
		}  



* **没有定义构造方法时,编译器为类自动添加 类名(){} 的空构造方法，定义时使用 Person = new Person(); 的形式**   

### 工具类
* **工具类**： 只提供方法的类， **把构造方法定义为静态**，则只能通过类名调用工具类的方法    
* 从java文件提取说明书的格式(帮助文档)：

		javadoc -d 目录 -author -version Xxx.java   


* 说明书的制作：

		/**
		* 这是针对数组进行操作的工具类
		* @author 作者
		* @version V.10
		*/
		public class ArrayTool {
	
			//把构造方法私有，外界就不能在创建对象了
			/**
			* 这是私有构造
			*/
			private ArrayTool(){}
	
			/**
			* 这是遍历数组的方法，遍历后的格式是：元素1, 元素2, 元素3, ...
			* @param arr 这是要被遍历的数组
			*/
			public static void printArray(int[] arr) {
			....
			}
		
			/**
			* 获取指定元素在数组中第一次出现的索引，如果元素不存在，就返回-1
			* @param arr 被查找的数组 
			* @param value 要查找的元素
			* @return 返回元素在数组中的索引，如果不存在，返回-1
			*/
			public static int getIndex(int[] arr,int value) {
			...
			}
		}   



### 代码块  
* 局部代码块：局部位置，用于限定变量的生命周期   
* 构造代码块：在类中的成员位置，用{}括起来的代码块，**每次调用构造方法执行前都会先执行构造代码块**，可以把多个构造方法中的共同代码放到一起，对对象进行初始化   
* 静态代码块：在类中的成员位置，用{}括起来的代码块，但是它用static修饰，**只在第一次调用构造方法时执行**，先于构造代码块执行，一般是对类进行初始化   
* 同步代码块：
* 执行顺序：静态代码块(只执行一次)---构造代码块---构造函数(**与三个代码的相对位置无关！**)   
### 约定俗成的命名方法
* **类名的首字母大写**
* **变量名和方法名的首字母小写**
* **驼峰规则**:名字里后面每个单词首字母大写   

### 方法的重载(OverLoad)  
* 方法的重载是指一个类中可以定义有相同的名字，但参数不同(数量或者参数类型不同)的多个方法。调用时，会根据不同的参数表选择对应的方法  
* **构造方法也可以重载**   

## 关键字
### this关键字
* 在类的方法定义中使用的this关键字代表使用该方法的对象的引用
* 当必须指出当前使用方法的对象是谁时要用this
* 有时使用this可以处理方法中成员变量和参数重名的情况
* this可以看作是一个变量，它的值是当前对象的引用    
### static关键字  
* **静态成员变量：**在类中，用static声明的成员变量为静态成员变量，它是该类的公用变量，也称为**类变量**(成员变量又称为实例变量，对象变量)，在第一次使用时被初始化，**对于该类的所有对象来说，static成员变量只有一份，被所有对象共享，可以通过类名调用！！（main方法被虚拟机直接用过类名调用），保存在方法区的静态区**
* **静态方法：**static声明的方法为静态方法，**不需要创建对象就可以加载静态方法**，调用时，不会将对象的引用传递给它，所以**在static方法中不可访问非static的变量和方法**（静态比对象先存在，this随着对象的创建而存在，所以static函数中不能有this关键字）  
* **静态方法不能重写**，方法随类加载，父类和子类中含有的其实是两个没有关系的方法，它们的行为也并不具有多态性  
* 可以通过对象引用或类名访问静态成员
* **静态内部类(嵌套类)**：    
	1. 内部静态类不需要有指向外部类的引用，但非静态内部类需要有对外部类的引用    
	2. 非静态内部类能够访问外部类的静态和非静态成员(包括私有)，**静态内部类只能访问外部类的静态成员**，不能访问外部类的非静态成员，但是自身的成员函数可以是静态的也可以是非静态的     
	3. 外部类访问内部类必须通过对象访问   
	4. 一个非静态内部类不能脱离外部类的对象被创建，一个非静态内部类可以访问外部类的数据和方法，因为他就在外部类里面   
	5. **一般来说，内部类用private或public static修饰，private保证数据安全性，不让外界直接访问，static让数据访问更方便**    
	6. **局部内部类 访问局部变量，需要局部变量被声明为final最终变量**
		
		* 因为局部变量会随着方法的调用完毕而消失，这时局部对象并没有立马从堆内存中消失，还要继续使用那个变量，为了让数据还能继续使用，只能用final修饰，这样再堆内存里面存储的其实是一个常量值         
	7. 非静态内部类和静态内部类的创建    
	
			//静态内部类的创建，不能通过外部类的对象创建，而只能通过外部类创建内部对象
		OuterClass.StatcInnerClass inner1 = new OuterClass.StaticInnerClass();
	
			//为了创建非静态内部类，我们需要外部类的实例  
			OuterClass outer = new OuterClass();
		OuterClass.InnerClass inner2 = outer.new InnerClass();
		
			//也可以一步创建非静态内部类
			OuterClass.InnerClass inner3 = new OuterClass().new InnweClass();
	
* **static修饰的成员随着类的加载而加载，优先于对象存在**   
* static不能修饰局部变量：**没有静态局部变量**   
* **为什么main前面要加static**：static关键字，告知编译器main函数是一个静态函数。也就是说main函数中的代码是存储在静态存储区的，静态方法在内存中的位置是固定的，即当定义了类以后这段代码就已经存在了。如果main()方法没有使用static修饰符，那么编译不会出错，但是如果你试图执行该程序将会报错，提示main()方法不存在。因为包含main()的类并没有实例化（即没有这个类的对象）。而使用static修饰符则表示该方法是静态的，不需要实例化即可使用。    


### final关键字(c++中的const)  
* final修饰的**变量**的值不能被重新赋值，因为这个变量其实是常量！  
* final修饰的**方法**不能够被重写
* final修饰的**类**不能够被继承 

* final修饰引用类型时，不能修改它所指向的地址值，但是该对象堆内存的值是可以改变的！
  
### package语句和import语句   
* 为了便于管理大型软件系统中的数目众多的类，解决类的命名冲突问题，java引入了包(package)机制，提供类的多重命名空间  
* **包的定义**：
	*  package 包名;   //多级包用.分开
	*  package语句必须是程序的第一条可执行的代码
	*  package语句在一个java文件中只能有一个
	*  如果没有package，默认表示无包名    
* 约定俗成package名用公司域名反过来写 com.xxx.xxx    
* **带包的编译和运行**
	* 手动式：javac直接编译，再创建相应文件夹，将.class文件放到相应目录后带包运行
	* 自动式：javac编译时带上-d  ，带包运行 
	
			javac -d.xxxx.java
			java xx.xx.xx  //带包运行

* **必须保证类的class文件位于所在package的正确目录下，否则会被认为使用的是裸体类**
* **访问相应的class时，在他之前必须要跟上对应的包(文件位置)，或者在第一句加入import语句导包(导入的是类)！！** 

		import com.java.* //引入com/java/下的所有的类   
		import com.java.Cat //引入com/java/下的Cat类


* **class文件的最上层包的父目录必须位于classpath下**   
* **package必须位于第一条可执行语句，且只能由一个，import必须位于class之前，可以有多个**
  
### J2SDK中主要的包   
* java.lang 包含java语言的核心类，如String、Math、Integer、System和Thread，提供常用功能
* java.awt 包含构成抽象窗口工具集的多个类，用来构建和管理应用程序的图形用户界面(GUI)  
* java.applet 包含applet运行所需的一些类  
* java.net 包含执行与网络相关的操作的类  
* java.io 包含能提供多种输入/输出功能的类  
* java.util 包含一些实用工具，如系统特性，使用与日期日历相关的函数  
* **java.lang中的类使用时不需要引入，其他需要引入！**
* 定义自己的包：在class文件的最上层包的父目录执行命令提示符：**jar cvf test.jar \*.\***     

### 访问控制   
Java的权限修饰符置于类的成员定义前，用来限定其他对象对该对象成员的访问权限  
      
* **private** 类内部  
* **default** 类内部、同一个包  
* **protected** 类内部、同一个包、不同包的子类  
* **public** 类内部、同一个包、不同包的子类、不同包无关类    
* **对class的修饰权限只可以用public和default(public类可以在任意地方被访问，default类只可以被同一个包内部的类访问)**   
* 构造方法只能使用权限修饰符，不能用static、final、abstract修饰     

## 类的继承和权限控制  
* Java中使用**extends**关键字实现类的继承机制，语法规则为：          **<modifier\> class <name\> extends <superclass\> {...}**   
* 通过继承，子类自动拥有了基类(superclass)的所有成员(成员变量和方法)  
* 提高了代码的复用性和维护性，让类与类之间产生了关系，是**多态的前提**，但是产生了弊端：类的耦合增强了！  
* Java只支持单继承，不允许多继承：**一个子类只能有一个基类，一个基类可以派生出多个子类**  
* Java支持**多层继承**，形成继承体系   
* 继承时private可以继承，但是无法访问  
### 方法的重写  

* 子类中可以根据需要对从基类中继承来的方法进行重写  
* 重写方法必须和被重写方法具有**相同方法名称、参数列表和返回类型**  
* **重写方法不能使用比被重写方法更严格的访问权限**  
	1. 父类方法如果是public 子类不能使用默认访问权限
	2. 父类方法使用默认访问权限，子类可以使用public权限  
	3. 最好直接使用相同的访问权限   
	4. 父类静态方法，子类也重写时也必须通过静态方法重写
	5. 父类是非静态方法时，子类重写也不能定义为静态方法  
* **super关键字**：super.f()引用基类的成分  **this代表本类对应的引用，super代表父类对应的引用**(可以理解为父类的引用，操作父类成员，但并不是父类的引用！)   
* 子类不能重写父类的私有方法(不会报错，但是相当于新写一个方法，不属于方法重写)   
* **重写和重载**：重写是子类出现了和父类中方法声明一模一样的方法，不能修改返回值类型，而重载要求同一类中方法名一样，参数列表不一样，而且可以改变返回值类型！  

### 继承中的构造方法   

* **子类的所有构造方法都会自动调用其父类的无参构造方法**    
  1. 因为子类继承父类的数据，可能还会使用父类数据，所以子类初始化前一定要先完成父类数据的初始化。    
  2. 若父类没有无参构造方法，子类创建对象时会报错。这时子类可以通过super(argument_list)调用父类的带参构造方法，使父类初始化，或者使用this去调用本类的其他成功调用父类构造方法的本类构造方法。    
* this(...) 表示调用本类构造方法，this.成员方法调用本类成员方法
* super(...)表示调用父类构造方法，super.成员方法调用父类的成员方法
* **子类每一个构造方法的第一条语句默认都是：super()** ，表示父类构造方法
* **this(...)或者super(...)，必须写在子类构造方法的第一行** ，否则可能对父类数据进行多次初始化  
* **若一个类的构造方法私有或被final修饰，则该类无法被继承**     
* 如果子类的构造方法中没有**显示调用基类构造方法**，则系统默认**调用基类无参构造方法**，如果基类中也没有无参数的构造方法则编译出错   

* 当new一个子类对象时，会调用子类的构造函数,在子类的构造函数中，第一条语句默认会执行**super();**这条语句**调用父类的无参构造函数** (因为会继承父类的变量,所以有必要调用其构造函数,因为有可能构造函数内会对其变量进行初始化)，如果父类的无参构造函数不存在就会出现编译时错误(如果父类手动重写了一个有参构造函数，那么就不存在默认的无参构造函数了，也需要再显示写一个)。但是可以**手动写super(参数...)**显示调用父类的有参构造函数，这样只要父类有对应的构造函数就不会报错了。


* 若子类直接继承父类，而父类定义中把带参构造函数相关的变量定义为私有，如   

		class Person{			
			private String name;
			private int age;
			public Person(){}
			public Person(String name, int age){
				this.name = name;
				this.age = age;
			}
		}
这时，子类不能访父类的成员，则不能直接使用父类的带参构造：

		class Student extends Person{
			public Student(){}
			public Student(String name, int age){
				ths.name = name;
				this.age = age;   //不能访问父类的私有成员，报错
			}
		}
应该写成如下形式才能使用带参构造：      


		class Student extends Person{
			public Student(){}
			public Student(String name, int age){
				super(name, age); //显示调用父类的构造方法
			}
		}

* 例题：

		class Fu {
			static {
				System.out.println("静态代码块Fu");
			}
			{
				System.out.println("构造代码块Fu");
			}
			public Fu() {
				System.out.println("构造方法Fu");
			}
		}
		
		class Zi extends Fu {
			static {
				System.out.println("静态代码块Zi");
			}
			{
				System.out.println("构造代码块Zi");
			}
			public Zi() {
				System.out.println("构造方法Zi");
			}
		}
		
		class ExtendsTest2 {
			public static void main(String[] args) {
				Zi z = new Zi();
			}
		}
		
		/*结果是：
		静态代码块Fu  (加载子类先加载父类，加载父类class则执行静态代码块，对类进行初始化)
		静态代码块Zi  (加载子类则执行子类的静态代码块，初始化子类)
		构造代码块Fu  (子类加载完则创建子类对象，子类对象构造则先进行父类对象构造)
		构造方法Fu    (对象构造时先执行构造代码块，后执行构造方法)
		构造代码块Zi  (父类构造完，进行子类对象构造)
		构造方法Zi    (静态代码块>构造代码块>构造方法)
		*/


* 例题

		class X {
			Y b = new Y();
			X() {
				System.out.print("X");
			}
		}
		
		class Y {
			Y() {
				System.out.print("Y");
			}
		}
		
		public class Z extends X {
			Y y = new Y();
			Z() {
				//super();
				System.out.print("Z");
			}
			public static void main(String[] args) {
				new Z(); 
			}
		}
		
		/*
		结果是YXYZ
		虽然子类构造方法默认有一个super()
		初始化的时候不是按照那个顺序进行的
		而是按照分层初始化进行的
		它仅仅表示要先初始化父类数据，再初始化子类数据
		*/

* 小结： 
  
	1.无继承时，初始化类的执行顺序：    

    	1. 静态成员变量
    	2. 静态代码块
    	3. 普通成员变量
    	4. 构造代码块
    	5. 构造方法  
  
    2.有继承时，初始化类的执行顺序：    

		1. 父类的静态成员变量
    	2. 父类的静态代码块
		3. 子类的静态成员变量
		4. 子类的静态代码块
		5. 父类的成员变量
		6. 父类的构造代码块
		7. 父类的构造方法
		8. 子类的成员变量
		9. 子类的构造代码块
		10. 子类的构造方法

### 对象转型  
* 一个基类的引用类型变量可以指向其子类的对象
* 基类的引用不可以访问其子类对象新增加的属性和方法  
* 可以使用  **引用变量 instanceof 类名** 来判断该引用型变量所指向的对象是否属于该类或该类的子类
* 子类的对象可以当作基类的对象来使用称作向上转型，反之称为向下转型
  

![instanceof](https://pic2.zhimg.com/v2-767f3a58c370f7f505aeb6619d98c98d_b.png)

### 多态  
* 多态：某一个事物，在不同时刻表现出来的不同状态   
* 多态的前提：    
	1. 要有继承关系
	2. 要有方法重写
	3. 要有父类引用指向子类对象  fu f = new zi();    
* 多态的实现方法
	1. 具体类多态(几乎不用)
	2. 抽象类多态(常用)
	3. 接口多态(最常用)
* **多态的成员访问特点**：         
	1. 成员变量：编译看左边(不能访问父类没有的变量)，运行看左边(访问的仍然是父类成员)
	2. 构造方法：创建子类对象的时候，访问父类的构造方法，对父类的数据进行初始化    
	3. 成员方法：编译看左边(不能访问父类没有的方法)，运行看右边(父类方法被子类重写了)  
	4. 静态方法：编译看左边，运行看左边(静态方法不能重写，所以访问的还是左边的)，静态成员是类的成员存放在栈中，类可以直接调用；实例成员是对象的成员，存放在堆中，只能被对象调用。 **重写的目的在于根据创造对象的所属类型不同而表现出多态**。因为静态方法无需创建对象即可使用。没有对象，重写所需要的“对象所属类型” 这一要素不存在，因此无法被重写。    
	5. 其实，由于只有成员方法存在方法重写，所以它运行看右边    
* 多态的弊端：不能使用子类的特有功能！  
	* 解决办法：将父类引用强转为子类引用(**向下转型**)，赋值给子类的引用：

		Fu f = new Zi();
		Zi z = (Zi)f;
		z.show();  //子类父类均含有show()
		z.method();  //method()是子了特有的   

* 对象的转型问题：
	1. **向上转型**：
	   
			Fu f = new Zi();   

	2. **向下转型**： 

			Zi z = (Zi)f; //要求该f必须是能够转换为Zi的

## 抽象类  
* 在java中，一个**没有方法体的方法**应该定义为**抽象方法**，而类中如果**有抽象方法**，该类就必须被定义为**抽象类**，用**abstract**修饰  
* 抽象类中不一定有抽象方法，但是有抽象方法的类一定要定义为抽象类    
* 含有抽象方法的类必须被声明为抽象类，**抽象类必须被继承，抽象方法必须重写**  
* **抽象类不能被实例化**  
* **抽象方法只需声明，不需要实现**
* 抽象类**有构造方法**，但是不能实例化，构造方法用于**子类访问父类数据的初始化**     
* 抽象类的**子类可以是抽象类也可以是具体类**：
	* 如果不重写抽象方法，该子类必须定义为抽象类    
	* 重写所有的抽象方法，这时子类是一个具体的类
* 抽象类的实例化其实是依靠具体的子类是实现的，是多态的方式  
* **区分空方法体和没有方法体**：
		
		abstract class Animal{
			public abstract void eat(){} //空方法体，报错
			public abstract void run(); //没有方法体，正确的写法
			public Animal(){}  //抽象类的构造方法
		}  

* abstract不能和下列关键字共存
	> private   (private修饰的方法是私有方法，不能被子类访问，不能重写)    
	> final		(final也不能被重写)   
	> static	(无意义，静态方法可以直接被类名访问，而抽象方法是没有方法体的)    

## 接口interface  
* 接口是抽象方法和常量值的定义的集合

		interface 接口名{}   
  
* 接口的子类
	* 可以是抽象类，意义不大
	* 具体类，必须重写接口的方法
	* 接口的实现：

			interface Inter{
				public void eat();
			}
			class InterImpl implements Inter{   //接口的实现
				public void eat(){...}
			}
	
* 本质上讲，接口是一种特殊的抽象类，**只包含常量(final)和方法的定义（全是抽象方法,不用加abstract）**，而没有变量和方法的实现，且**没有构造方法**，成员方法均默认为抽象方法，不能有方法体      

		public interface Runner{
			public static final int id = 1;  //等价于 public int id = 1
			public void start();
			public void run();
			public void stop();
		}
  
* 接口中的成员变量只能是常量(final)，并且是静态的(可以通过接口名访问)   
* 接口**没有构造方法**，其实是所有的类都默认继承自**Object类**(无参构造)   
* **接口可以多重实现(一个类继承多个接口)，并且可以在继承一个类的同时实现多个接口**     
* **接口中声明的属性默认且只能为public static final**    
* **接口中只能定义抽象方法，默认且只能为public**
* **接口可以继承其它的接口，并添加新的属性和抽象方法(接口继承用extends，接口实现用implements，并且可以多继承多实现)**  

### 抽象类和接口的区别：   
* 成员区别：
	* 抽象类
		* 成员变量：可以是变量也可以是常量
		* 构造方法：有
		* 成员方法：可以抽象，可以非抽象
	* 接口
		* 成员变量：只可以是常量
		* 构造方法：无
		* 成员方法：只可以抽象
* 关系区别：
	* 类与类
		* 继承：单继承(extends)
	* 类与接口
		* 实现：单实现，多实现(implements)
	* 接口与接口
		* 继承：单继承，多继承(extends)
* 设计理念区别
	* 抽象类被继承体现的是：" is a "的关系，**抽象类中定义的是该继承体系的共性功能**  
	* 接口被实现体现的是：" like a "的关系，**接口中定义的是该继承体系的扩展功能**   

------

# java中的常用类

## Object类

* **Object类是所有JAva类的根基类**，每个类都直接或者间接继承自Object类  

### toString方法

    public String toString(); //返回对象的字符串表示形式
    //等价于getClass().getName() + '@' + Integer.toHexString(hashCode());  

* **可以根据需要在用户自定义类型中重写toString()方法**：把该类的所有成员变量值组成返回即可   

* 要进行String与其它类型数据的连接操作时(如System.out.println("info"+person))，将自动调用该对象的toString方法
  


#### hashCode()方法

	public int hashCode();  //返回对象的哈希值
	//哈希值是根据地址值由哈希算法计算出来的值，不是地址，但可以理解为地址 

### getClass()方法

	public final Class getClass();  //返回此Object的运行时类  
	//Class类的方法 public String getName();以String形式返回此Class对象所表示的实体名称   

### equals()方法  
* **==比较的是对象的引用(引用的地址，比较是否指向同一对象)，equals可以比较对象的内容(需要重写，否则和==等价)**   

		public boolean equals(Object obj){
			if(obj == null) return false;
			if(obj == this) return true;
			else{
				if(obj instanceof Cat){  //判断obj对象是否是Cat类的，若不是则无法转型
					Cat c = (Cat)obj;
					if(c.color.equals(this.color) && c.height.equals(this.height)){  //String	的equals自带
						return true;
				}
				else retuen false;
			}
		}
	}     

* **还是自动生成的好！！**

## Scanner类

* Scanner类位于java.util 使用需要导包，用于接收键盘录入数据

### Scanner的构造方法
* **Scanner(InputStream source);** 

		Scanner(System.in);

### hasNextXxx 方法
* **public boolean hasNextXxx();**  //判断是否是某种类型的元素    
	
		public boolean hasNextInt();   //判断是否是int

### nextXxx 方法
* **public Xxx nextXxx();**  //获取元素

		public int nextInt();  //获取int元素   
		public String nextLine();  //获取一个String    
	
	先获取数值，再获取String，会将回车接收到String，**解决办法**：
		
		//1.获取数值后创建一个新的键盘录入对象获取字符串
		Scanner sc = new Scanner(System.in);
		int a = sc.nextInt();
		Scanner sc0 = new Scanner(System.in);
		String s = sc0.nextLine();
	
		//2.将所有的数据都按String接收，再根据需要进行转换
		Scanner sc = new Scanner(System.in);
		String a1 = sc.nextLine();
		String sc1 = sc.nextLine();


## String 类  
* 字符串就是由多个字符组成的一串数据，可以看成是一个字符数组   
* 字符串字面值"abc"也可以看成是一个字符串对象   
* **字符串是常量，一旦被赋值，就不能改变**：字符串拼接实际上是重新创建了一个字符串，它是之前的字符串拼接起来形成的新字符串     

###String的构造方法
* **public String();**  //无参构造
* **public String(byte[] byte);**  //把字节数组转成字符串

		byte[] bys = {97,98,99,100};
		String s = new String(bys); 
		System.out.println(s); //得到abcd

* **public String(byte[] byte,int index,int length);** //把字节数组的一部分转换成字符串,**注意参数length是长度，不是结束位置**

		String s = new String(bys,2,2);
		System.out.println(s); //得到cd

* **public String(char[] value);** //把字符数组转成字符串
* **public String(char[] value, int index; int length);** //把字符数组的一部分转成字符串    
* **public String(String original);**  //把字符串常量转换成字符串  
	* String s = new String("hello");和String s = "hello"的区别：
	* 前者会创建两个对象(堆内存和方法区)，后者只创建一个对象(方法区)
	* [![ATa7Ie.png](https://s2.ax1x.com/2019/04/10/ATa7Ie.png)](https://imgchr.com/i/ATa7Ie)
	* **字符串如果是变量相加，先开辟内存空间，再拼接**
	* **字符串如果是常量相加，先拼接，再在方法区常量池里找，若有直接返回，否则创建**
	
			String s1 = "hello";
			String s2 = "world";
			String s3 = "helloworld";
			System.out.println(s3 == s1 + s2);// false
			System.out.println(s3.equals((s1 + s2)));// true
	
			System.out.println(s3 == "hello" + "world");// true
			System.out.println(s3.equals("hello" + "world"));// true
		
			// 通过反编译看源码，我们知道这里已经做好了处理。
			// System.out.println(s3 == "helloworld");
			// System.out.println(s3.equals("helloworld"));

### String类的判断功能  

* **boolean equals(Object obj);** //比较字符串内容是否相同
* **boolean equalsIgnoreCase(String str);** //比较字符串内容是否相同，忽略大小写   
* **boolean contains(String str);** //模式匹配
* **boolean startsWith(String str);** //判断字符串是否以某个指定字符串开头
* **boolean endsWith(String str);** //判断字符串是否以某个指定字符串结尾   
* **boolean isEmpty();**  //判断字符串内容是否为空   

### String类的获取功能 

* **int length();** //获取指定字符串的长度    
* **char charAt(int index);** //获取指定索引位置的字符   
* **int indexOf(int ch);** //获取指定字符在次字符串中那个第一次出现处的索引(既可以int的97也可以char的'a')    
* **int indexOf(String str);** //返回指定字符串在此字符串中第一次出现处的索引    
* **int indexOf(int ch, int fromIndex);** //返回指定字符在此字符串中指定位置之后第一次出现处的索引   
* **int indexOf(String str, int fromIndex);** //返回指定字符串在此字符串中指定位置之后第一次出现处的索引    
* **String substring(int start);** //从指定的开始位置截取字符串(**包含**)
* **String substring(int start, int end);** //从指定的开始和结束位置截取字符串(**包含start，不包含end索引！！！！**)   

### String类的转换功能  

* **byte[] getBytes();**  //把字符串转换为字节数组
* **char[] toCharArray();**  //把字符串转换为字符数组
* **static String valueOf(char[] chs);**  //字符数组转换为字符串
* **static String valueOf(int i);** //把int转换为字符串
	* **static String valueOf(Xxx);** 可以把任意类型数据转为字符串
* **String toLowerCase();** //把字符串转换为小写
* **String toUpperCase();** //把字符串转换为大写
* **String concat(String str);** //字符串拼接

### 替换功能

* **String replace(char old, char new);** 
* **String replace(String old, String new);**     

### 去除字符串两空格

* **String trim();** //去掉前后两段空格，中间的空格保留

### 按字典顺序比较两个字符串

* **int compareTo(String str);**  //只输出**第一次**不同的位置的ASCII码差值，如果比较直到一个字符串的结尾都一直相同，则输出左边比右边多出来的字符数量
* **int compareToIgnoreCase(String str);**  //

# 异常  
## 异常的概念  
* Java异常是指**运行期**出现的错误
* **观察错误的名字和行号最重要**   
* Java程序执行过程中如出现异常事件，可以生成一个异常类对象，该异常对象封装了异常事件的信息并将被提交给Java运行时系统，这个过程成为**抛出(throw)异常**  
## 异常的分类
![异常的分类](https://pic1.zhimg.com/v2-1aafd1d72be6ebf3695aa2aead4defdc_b.png)    


* 异常的根类Throwable类：Error类(处理不了的错误)和Exception类(可以处理可以catch的错误)，Exception包含RuntimeException(经常出的错误，可以catch可以不catch)
* Error称为错误，有Java虚拟机生成并抛出，包括动态连接失败、虚拟机错误等，程序对其不作处理
* Exception是所有异常类的父类，其子类对应各种可能出现的异常事件，一般需要用户显式声明或捕获
![异常的抛出](https://pic3.zhimg.com/v2-1ecab71c252ac0b8ad507e6f13bc58ee_b.png)
* RuntimeException是一类特殊异常，如被0除，数组下标超范围等，其产生比较麻烦，处理麻烦，如果显式的声明或捕获将会对程序可读性和运行效率影响很大。因此由系统自动检测并将它们交给缺省的异常处理程序(用户可不必对其处理)    





