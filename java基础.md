# java内存划分   
## 1. 栈：存放方法的局部变量
局部变量：方法的参数和方法内部的变量    
作用域：超出作用域，立刻从栈内存中消失   
## 2. 堆：new出来的东西，成员变量
堆内存里的东西都有一个地址值：16进制   
堆内存里的数据都有默认值，规则：  
    整数（0）   浮点数（0.0）   字符（'\u0000'）    布尔(false)       引用类型（null）
## 3.方法区(Method area)：存储.class相关信息，包含方法的信息
## 4.本地方法栈(Native method stack)：与操作系统相关
## 5.寄存器(pc Register)：CPU

# Scanner 中的坑
- `nextInt, nextDouble, nextFloat` : 获取一个指定类型的数据，遇到结束符不再获取
- `next()` : 不能得到带空格的字符串，一定要读取到有效字符后才可以结束，结束条件是非有效字符(空格、tab、回车\r)      
- `nextLine()` : 以回车为结束符，返回回车之前的所有字符，可以获得空白，但是可能会获得上一次输入后的回车键而直接结束       


​          
~~~java
        Scanner sc = new Scanner(System.in);
        int a = sc.nextInt();
        String s1 = sc.next();
        String s2 = sc.nextLine();
        System.out.println("a:"+a);
        System.out.println("s1:"+s1);
        System.out.println("s2:"+s2);
~~~
输入输出：
~~~java
    11111			123you		jkk
a:11111
s1:123you
s2:		jkk
~~~

Scanner是一个扫描器，它扫描数据都是去内存中一块缓冲区中进行扫描并读入数据的，而我们在控制台中输入的数据也都是被先存入缓冲区中等待扫描器的扫描读取。这个扫描器在扫描过程中判断停止的依据就是“结束符”，空格，回车，tab 都算做是结束符
可以看到 `nextInt` 和 `next` 并不会获取到空格，全部自动跳过，而且会把这些结束符留在缓冲区中(nextDouble、nextFloat同理)，当 `nextLine` 读取的时候就会读取到

~~~java
    int a = sc.nextInt();
    String s = sc.nextLine();
    System.out.println("a:"+a);
    System.out.println("s:"+s);
~~~

此时，因为第一次输入完后回车了，不管第二次输入什么，s 的值都将为""值，因为s一开始就读取到了结束符  
解决的办法是使用一个 `nextLine()` 把回车吸收掉      

~~~java
    int a = sc.nextInt();
    sc.nextLine();//提前吞掉空格，防止后面吞掉
    String s = sc.nextLine();
~~~

所以如果不是必须要获得空格和tab，尽量不要使用 `nextLine()`



# 数组
## 1. 初始化  
~~~java
    int[] a = new int[10];  //动态初始化，指定长度
    int[] b = new int[]{1,2,3}; //静态初始化，指定内容
    int[] c = {1,2,3,4};  //静态初始化，省略格式
~~~
## 2. 数组长度
~~~java
    a.length
~~~

# 面向对象
## 1. 类  
### 1.1 访问控制
- **private:** 类内部
- **default:** 类内部、同一个包
- **protected:** 类内部、同一个包、不同包的子类
- **public:** 类内部、同一个包、不同包的子类、不同包无关类 
- **对class的修饰权限只可以用public和default(public类可以在任意地方被访问，default类只可以被同一个包内部的类访问)** 

- 外部类可以用public/default   
  成员内部类用public/protected/default/private    
  **局部内部类什么都不能写**（但并不是default）
  



### 1.2 构造方法

- 格式：public 类名称(参数)
- 名字必须和类完全一样，没有返回值，可以重载
~~~java
    public class Student {
        public Student(int age, String name){
            this.age = age;
            this.name = name;
        }

        public Student(String name){
            this.name = name;
        }
        
        public Student(){

        }
    }
~~~

- 没有编写构造方法时，编译器会产生默认构造方法，没有参数，但是一旦写了构造方法，编译器便不会再自动产生构造方法
### 1.3 标准类
- 所有成员变量使用private修饰
- 为每一个成员变量编写一对儿get/set方法
- 编写一个无参构造和一个全参构造

### 1.4 J2SDK中主要的包
- java.lang 包含java语言的核心类，如String、Math、Integer、System和Thread，提供常用功能
- java.awt 包含构成抽象窗口工具集的多个类，用来构建和管理应用程序的图形用户界面(GUI)  
- java.applet 包含applet运行所需的一些类  
- java.net 包含执行与网络相关的操作的类  
- java.io 包含能提供多种输入/输出功能的类  
- java.util 包含一些实用工具，如系统特性，使用与日期日历相关的函数  
- **java.lang中的类使用时不需要引入，其他需要引入！**


### 1.5 String类


1. 字符串是常量，一旦创建，不能改变
2. 3+1种创建方式

~~~java
    String s1 = new String();     //空字符串

    char[] c = new char[]{'a','b','q'};
    String s2 = new String(c);    //根据字符数组创建

    byte[] b = {97, 99, 98};
    String s3 = new String(b);    //根据byte数组创建
    
    String s4 = "hello";          //直接创建
~~~

3. **字符串常量池**：

- 字符串常量池在堆内存中，保存的是byte值
~~~java
    String s1 = "abc";
    String s2 = "abc";
    char[] c = {'a','b','c'};
    String s3 = new String(c);
    System.out.println(s1 == s2);          //true
    System.out.println(s1 == s3);          //false
    System.out.println(s2 == s3);          //false
    System.out.println(s1.equals(s3));     //true
~~~
4. 常用方法  
~~~java
    //截取
    String s1 = "java";
    String s2 = "hellooooasoaosa";
    System.out.println(s1.length());     //获取长度//2
    System.out.println(s1.concat(s2));   //拼接//java
    System.out.println(s1.charAt(1));    //获取指定索引的字符//'a'
    System.out.println(s1.indexOf('j')); //获取字符第一次出现的索引//0
    System.out.println(s1.substring(1,3)); // [1,3)
    
    //替换
    char[] c = s1.toCharArray();          //字符串拆分为字符数组返回
    byte[] b = s1.getBytes();             //获得底层字节数组
    String s3 = s2.replace("o","*");      //内容替换//hell****as*a*sa

    //分割
    String[] s4 = s2.split("o");    //返回字符数组,按正则匹配切分
~~~
### 1.6 说明书和工具类
* **工具类**： 只提供方法的类， **把构造方法定义为静态**，则只能通过类名调用工具类的方法    
* 从java文件提取说明书的格式(帮助文档)：

~~~java
javadoc -d 目录 -author -version Xxx.java   
~~~
* 说明书的制作：
~~~java
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
~~~


### 1.7 Arrays数组工具类
- public static String toString(数组)：将参数数组变成字符串（格式：[元素1，元素2，元素3...]）
- public static void sort(数组)：将数组升序排序（在原数组修改）

~~~java
    int[] a = {20, 30, 10};
    String s = Arrays.toString(a);   //将数组元素变为字符串
    System.out.println(s);

    Arrays.sort(a);                 //升序排序
    System.out.println(Arrays.toString(a));

运行结果：
[20, 30, 10]
[10, 20, 30]
~~~

### 1.8 Math数学工具类
- public static double abs(double num)   //绝对值
- public static double ceil(double num)  //向上取整
- public static double floor(double num) //向下取整
- public static long round(double num)   //四舍五入

## 2. 局部变量和成员变量   

- 局部变量作用域在方法内，没有默认值，存放在**栈内存**里，随着方法出栈消失  
- 成员变量作用域在整个类，有默认值，存放在**堆内存**里，随着对象被垃圾回收而消失
- static不能修饰局部变量：**没有静态局部变量** 
- **为什么main前面要加static**：static关键字，告知编译器main函数是一个静态函数。也就是说main函数中的代码是存储在静态存储区的，静态方法在内存中的位置是固定的，即当定义了类以后这段代码就已经存在了。如果main()方法没有使用static修饰符，那么编译不会出错，但是如果你试图执行该程序将会报错，提示main()方法不存在。**因为包含main()的类并没有实例化（即没有这个类的对象）**。而使用static修饰符则表示该方法是静态的，不需要实例化即可使用。
### 2.1 静态成员变量
在类中，用static声明的成员变量为静态成员变量，它是该类的公用变量，也称为**类变量**(成员变量又称为实例变量，对象变量)，在第一次使用时被初始化，**对于该类的所有对象来说，static成员变量只有一份，被所有对象共享，可以通过类名调用！！（main方法被虚拟机直接用过类名调用），保存在方法区的静态区**
### 2.2 静态方法
static声明的方法为静态方法，**不需要创建对象就可以加载静态方法**，调用时，不会将对象的引用传递给它，所以**在static方法中不可访问非static的变量和方法**（静态比对象先存在，this随着对象的创建而存在，所以static函数中不能有this关键字）  
* **静态方法不能重写**，方法随类加载，父类和子类中含有的其实是两个没有关系的方法，它们的行为也并不具有多态性  
* 可以通过对象引用或类名访问静态成员

- **静态代码块：** 属于所在类，随类加载

- **执行顺序：** 静态代码块、构造代码块、构造函数
~~~java
public class HelloA {
    public HelloA(){//构造函数
        System.out.println("A的构造函数");    
    }
    {//构造代码块
        System.out.println("A的构造代码块");    
    }
    static {//静态代码块
        System.out.println("A的静态代码块");        
    }
    public static void main(String[] args) {
        HelloA a=new HelloA();
        HelloA b=new HelloA();
    }

}

运行结果：
A的静态代码块
A的构造代码块
A的构造函数
A的构造代码块
A的构造函数

~~~

### 2.3 内部类
- 外部类权限可以用public/default   
  成员内部类权限用public/protected/default/private    
  **局部内部类什么都不能写**（但并不是default）
1. 成员内部类: 



- 内用外可以随意使用，外用内需要有内部类的对象
- 可以在外部类的方法中创建内部类的对象调用内部类方法，实现通过外部类的方法调用内部类的方法
- 直接访问： 外部类.内部类 = new 外部类().new 内部类()
~~~java
修饰符 class 类名称{    //外部类
    private int num = 10;
    修饰符 class 类名称{   //成员内部类
        num = 20
        ...
    }
    ...
}
~~~

~~~java

public class OuterClass {
    int num = 10;
    public class Inner{
        int num = 20;
        public void method(){
            int num = 30;
            System.out.println(num); //30
            System.out.println(this.num); //20
            System.out.println(OuterClass.this.num); //10
        }
    }
}
~~~

2. 局部内部类（包含匿名内部类）:定义在方法中
- 只有当前所属的方法才能使用它
- 局部内部类如果要访问所在方法的局部变量，那么这个局部变量必须是**有效final的**（java8开始final可以省略不写），因为**局部变量在栈内存中，方法结束后就消失了，局部内部类的对象在堆内存中持续存在直到垃圾回收消失，为了避免冲突，局部内部类copy了一份局部变量，必须保证局部变量不会变。**
~~~java
修饰符 class 类名称{    //外部类

    修饰符 返回值类型 方法名(参数){

        calsss 类名称{  //局部内部类，定义在方法中
            ...
        }
    }
    ...
}
~~~


3. **匿名内部类**（和匿名对象概念区分开）

- 定义格式
~~~java
接口名称 对象名 = new 接口名称(){
    //重写接口的所有抽象方法
};  //最后一定要加;

~~~
- {...} 这些内容才是匿名内部类的内容
- 匿名内部类在创建对象时，只能使用一次，需要多次定义对象时必须单独定义实现类
- 匿名对象只能调用一次方法

- 例子
~~~java
public interface NoName {
    abstract void method();
}

public class NoNameDemo {
    public static void main(String[] args) {
        NoName a = new NoName() {
            @Override
            public void method() {
                System.out.println("匿名内部类实现了方法");
            }
        };
        a.method();    //可以少定义一个类，使用其方法
    }
}
~~~



~~~java
public interface NoName {
    abstract void method1();
    abstract void method2();
}

public class NoNameDemo {
    public static void main(String[] args) {
        NoName a = new NoName() {  //匿名内部类
            @Override
            public void method1() {
                System.out.println("匿名内部类实现了方法1");
            }

            @Override
            public void method2() {
                System.out.println("匿名内部类实现了方法2");
            }

        };
        a.method1();
        a.method2();

        new NoName(){  //匿名对象，只能调用一次方法
            @Override
            public void method1() {
                System.out.println("匿名对象实现了方法1");
            }

            @Override
            public void method2() {
                System.out.println("匿名对象实现了方法2");
            }
        }.method1();
    }
}

~~~


### 2.4 静态内部类(嵌套类)    
1. 内部静态类不需要有指向外部类的引用，但非静态内部类需要有对外部类的引用    
2. 非静态内部类能够访问外部类的静态和非静态成员(包括私有)，**静态内部类只能访问外部类的静态成员**，不能访问外部类的非静态成员，但是自身的成员函数可以是静态的也可以是非静态的     
3. 外部类访问内部类必须通过对象访问   
4. 一个非静态内部类不能脱离外部类的对象被创建，一个非静态内部类可以访问外部类的数据和方法，因为他就在外部类里面   
5. **一般来说，内部类用private或public static修饰，private保证数据安全性，不让外界直接访问，static让数据访问更方便**    
6. **局部内部类 访问局部变量，需要局部变量被声明为final最终变量**
    * 因为局部变量会随着方法的调用完毕而消失，这时局部对象并没有立马从堆内存中消失，还要继续使用那个变量，为了让数据还能继续使用，只能用final修饰，这样再堆内存里面存储的其实是一个常量值         
7. 非静态内部类和静态内部类的创建    
~~~java
    //静态内部类的创建，不能通过外部类的对象创建，而只能通过外部类创建内部对象
    OuterClass.StaticInnerClass inner1 = new OuterClass.StaticInnerClass();

    //为了创建非静态内部类，我们需要外部类的实例  
    OuterClass outer = new OuterClass();
    OuterClass.InnerClass inner2 = outer.new InnerClass();

    //也可以一步创建非静态内部类
    OuterClass.InnerClass inner3 = new OuterClass().new InnweClass();
~~~






## 3. 三大特征
### 3.1 封装
- 隐藏细节信息，对外界不可见
1. 方法就是一种封装
2. 关键字private也是一种封装

### 3.2 继承

- 继承是多态的前提，没有继承就没有多态

### 3.3 多态

## 4. 继承
- **继承时private可以继承，但是无法访问**
### 4.1 重名变量
1. 直接通过子类对象访问成员变量：就近原则，子类没有就去父类找
2. 间接通过成员方法访问成员变量：方法属于谁就优先用谁，没有则向上找
3. this关键字，super关键字
### 4.2 方法的重写
- **必须要保证子类的方法参数列表和父类完全一样，返回值只能变为更小(或一样)的范围**
- @override 可以检测是否是正确的重写
- **重写方法的权限必须比父类原方法松(或相等)**
- 子类构造方法中可以**重载**父类的构造函数，重载语句必须是子类构造函数的第一个语句。**不重载时默认调用父类的无参构造。如果父类没有无参构造，必须通过super重载父类构造。**

### 4.3 super关键字的用法
1. 子类的成员方法中访问父类的成员变量和成员方法
2. 子类构造方法中访问父类的构造方法
3. 操作父类的成员，但super不是父类的引用
### 4.4 this关键字
1. 在类的方法定义中使用的this关键字代表使用该方法的对象的引用
2. 当必须指出当前使用方法的对象是谁时要用this
3. 有时使用this可以处理方法中成员变量和参数重名的情况
4. this可以看作是一个变量，它的值是当前对象的引用 

### 4.5 final关键字(c++中的const)  
1. final修饰的**变量不能被重新赋值**，因为这个变量其实是常量！
2. final修饰的**成员变量在初始化时不会被自动赋予初值**，要直接赋值或通过构造函数赋值  
3. final修饰的**方法不能够被重写**
4. final修饰的**类不能够被继承** 
5. final修饰**引用类型时，不能修改它所指向的地址值**，但是该对象堆内存的值是可以改变的！
6. abstract和final不能同时修饰类，互相矛盾

### 4.6 继承中的构造方法
1. **子类的所有构造方法都会自动调用其父类的无参构造方法**    
  - 因为子类继承父类的数据，可能还会使用父类数据，所以子类初始化前一定要先完成父类数据的初始化。    
  - 若父类没有无参构造方法，子类创建对象时会报错。这时子类可以通过super(argument_list)调用父类的带参构造方法，使父类初始化，或者使用this去调用本类的其他成功调用父类构造方法的本类构造方法。 
2. this(...) 表示调用本类构造方法，this.成员方法调用本类成员方法
3. super(...)表示调用父类构造方法，super.成员方法调用父类的成员方法
4. **this(...)或者super(...)，必须写在子类构造方法的第一行** ，否则可能对父类数据进行多次初始化
5. **若一个类的构造方法私有或被final修饰，则该类无法被继承**  
6. 当new一个子类对象时，会调用子类的构造函数,在子类的构造函数中，第一条语句默认会执行**super();**这条语句**调用父类的无参构造函数** (因为会继承父类的变量,所以有必要调用其构造函数,因为有可能构造函数内会对其变量进行初始化)，如果父类的无参构造函数不存在就会出现编译时错误(如果父类手动重写了一个有参构造函数，那么就不存在默认的无参构造函数了，也需要再显示写一个)。但是可以**手动写super(参数...)
* 若子类直接继承父类，而父类定义中把带参构造函数相关的变量定义为私有，如   
~~~java
		class Person{			
			private String name;
			private int age;
			public Person(){}
			public Person(String name, int age){
				this.name = name;
				this.age = age;
			}
		}
~~~
这时，子类不能访父类的成员，则不能直接使用父类的带参构造：
~~~java
		class Student extends Person{
			public Student(){}
			public Student(String name, int age){
				ths.name = name;
				this.age = age;   //不能访问父类的私有成员，报错
			}
		}
~~~
应该写成如下形式才能使用带参构造：      
~~~java
		class Student extends Person{
			public Student(){}
			public Student(String name, int age){
				super(name, age); //显示调用父类的构造方法
			}
		}
~~~

### 4.7 执行顺序小结
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

~~~java
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
~~~


* 例题
~~~java
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
~~~

### 4.8 对象转型
* 一个基类的引用类型变量可以指向其子类的对象
* 基类的引用不可以访问其子类对象新增加的属性和方法  
* 可以使用  **引用变量 instanceof 类名** 来判断该引用型变量所指向的对象是否属于该类或该类的子类
* 子类的对象可以当作基类的对象来使用称作向上转型，反之称为向下转型
  

![instanceof](https://pic2.zhimg.com/v2-767f3a58c370f7f505aeb6619d98c98d_b.png)

## 5. 多态
- 多态：一个对象，在不同时刻表现出来的不同状态     

        父类名称 对象名 = new 子类名称();

### 5.1 多态的前提：

1. 要有继承关系
2. 要有方法重写
3. 要有父类引用指向子类对象 fu f = new zi();

### 5.2 多态的成员访问特点

1. **成员变量：编译看左边(不能访问父类没有的变量)，运行看左边(访问的仍然是父类成员)**
2. **构造方法：创建子类对象的时候，访问父类的构造方法，对父类的数据进行初始化**
3. **成员方法：编译看左边(不能访问父类没有的方法)，运行看右边(父类方法被子类重写了)**
4. 静态方法：编译看左边，运行看左边(静态方法不能重写，所以访问的还是左边的)，静态成员是类的成员存放在栈中，类可以直接调用；实例成员是对象的成员，存放在堆中，只能被对象调用。 重写的目的在于根据创造对象的所属类型不同而表现出多态。因为静态方法无需创建对象即可使用。没有对象，重写所需要的“对象所属类型” 这一要素不存在，因此无法被重写。

- **其实，由于只有成员方法存在方法重写，所以它运行看右边**

### 5.3 多态的弊端

- 不能使用子类的特有方法！但是可以通过对象转型将父类引用强转为子类引用，赋值给子类的引用
~~~java
    Fu f = new Zi(); 
    Zi z = (Zi)f; 
    z.show(); //子类父类均含有show() 
    z.method(); //method()是子类特有的
~~~
### 5.4 对象转型
1. 向上转型(多态写法)：向上转型一定是安全的
~~~java
    Fu f = new Zi();  
~~~
2. 向下转型：其实是**还原**，必须要转型前使用向上转型创建的，将父类对象还原成为原本的子类对象
~~~java
    Zi z = (Zi)f; //要求该f必须是能够转换为Zi的
~~~



## 6. 抽象类
- 在java中，一个没有方法体的方法应该定义为抽象方法，而类中如果有抽象方法，该类就必须被定义为抽象类，用abstract修饰

~~~java
    public abstract void eat();
~~~
- 抽象类中不一定有抽象方法，但是有抽象方法的类一定要定义为抽象类
- **抽象类不能实例化，必须被继承再使用，抽象方法必须全部重写**
- 抽象类的子类可以是抽象类也可以是具体类：如果不重写抽象方法，该子类必须定义为抽象类，重写所有的抽象方法后，子类才是一个具体的类
~~~java
  abstract class Animal{
  	public abstract void eat(){} //空方法体，报错
  	public abstract void run(); //没有方法体，正确的写法
  	public Animal(){}  //抽象类的构造方法
  }  
~~~
- abstract不能和下列关键字共存

    1. private (private修饰的方法是私有方法，不能被子类访问，不能重写)
    2. final	(final也不能被重写)
    3. static	(无意义，静态方法可以直接被类名访问，而抽象方法是没有方法体的)

## 7. 接口
1. 接口是**抽象方法和常量值**(java8以上还有默认方法和静态方法，java9以上可以有私有方法)的集合，是多个类的公共规范。
- **一个类可以实现多个接口，如果实现的多个接口存在重名抽象方法，必须重写一次。(子接口只需要重写重名的default方法，重名的抽象方法可以不重写)**
- **如果父类和接口中存在重名的方法，会优先用父类的方法**
~~~java
    interface Inter{
        public static final int STUDENT_ID = 1;//常量名称全部大写
        int STUDENT_AGE = 10;  //public static final可以省略，必须初始化
        public abstract void eat();
        void sleep();  //public abstract可以省略
    }
    class InterImpl implements Inter{   //接口的实现，重写全部抽象方法
        public void eat(){...}
        public void sleep(){...}
    }
~~~

2. 本质上讲，接口是一种特殊的抽象类，只包含常量(final)和方法的定义（全是抽象方法,不用加abstract），而没有变量和方法的实现，且**没有构造方法和静态代码块**，成员方法均默认为抽象方法，不能有方法体

3. **java8新增的默认方法用于接口升级**，在接口的实现类里面不需要重写默认方法（也可以重写）
~~~java
    interface Inter{
        void eat();
        void sleep();
        public default void learn(){ //升级接口，不需要修改接口的实现了，否则还要在每个实现类里面重写抽象方法
            System.out.println("learn....")
        } 
    }
    class InterImpl implements Inter{   //接口的实现，重写全部抽象方法
        public void eat(){...}
        public void sleep(){...}
    }
~~~

4. **java8接口中可以定义静态方法**，但是只能通过接口调用，实现类不能调用！
~~~java
    interface Inter{
        public static void methodStatic(){
        }
    }
    public class InterImpl implements Inter{   
    //接口里没有抽象方法，不需要重写
    }
    public class Demo{
        public static void main(String[] args){
            InterImpl impl = new InterImpl();
            //impl.methodStatic(); 错误，不能通过实现类调用接口的静态方法
            Inter.methodStatic(); //直接通过接口调用
        }
    }
~~~

5. **java9接口中允许定义私有方法**
- 普通私有方法可以解决多个默认方法之间重复代码的问题 private void 方法名(){...}
- 静态私有方法可以解决多个静态方法之间重复代码的问题 private static void 方法名(){...}
~~~java 
public interface MyInterface {
    public static void methodS1(){
        System.out.println("静态方法1");
        methodSCommon();
    }
    public static void methodS2(){
        System.out.println("静态方法2");
        methodSCommon();
    }
    private static void methodSCommon(){
        System.out.println("静态方法共有的部分代码");
    }
    
    public default void methodD1(){
        System.out.println("默认方法1");
    }
    public default void methodD2(){
        System.out.println("默认方法2");
    }
    private void methodDCommon(){
        System.out.println("默认方法共有的部分代码");
    }
}
~~~

## 8. Object类
- 是所有类的根基类，每个类都直接或间接继承自Object类
### 8.1 toString() 方法

~~~java
public String toString(); //返回对象的字符串表示形式
//等价于
getClass().getName() + '@' + Integer.toHexString(hashCode());  
~~~

- 可以根据需要在用户自定义类型中重写toString()方法：把该类的所有成员变量值组成返回即可

- 要进行String与其它类型数据的连接操作时(如System.out.println("info"+person))，或者直接打印该对象，将自动调用该对象的toString()方法

- IDEA可以自动生成toString()的重写代码

### 8.2 eauals() 方法

- ==比较的是对象的引用(引用的地址，比较是否指向同一对象)，equals可以比较对象的内容(**需要重写，否则和==等价**)

~~~java
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Person person = (Person) o;
        return age == person.age &&
                Objects.equals(name, person.name);   //空指针安全
    }
~~~

## 9. 时间相关类
### 9.1 Date类: java.util.Date

- 时间原点：1970年1月1日00：00：00(中国+8小时)

1. `System.currentTimeMillis()` 计算当前时间到原点经历的毫秒数   
2. 无参构造:当前系统时间
3. `toLocalString()`方法：按当地习惯打印时间
~~~java
    Date date = new Date();
    System.out.println(date);//打印当前时间Tue Jul 28 20:57:42 CST 2020
~~~
4. 带参数构造：传入相对原点的毫秒，打印其对应时间
~~~java
    Date date = new Date(0L);
    System.out.println(date); //Thu Jan 01 08:00:00 CST 1970
    Date date1 = new Date(System.currentTimeMillis());
    System.out.println(date1); //Tue Jul 28 21:02:32 CST 2020
~~~

5. `long getTime(Date)` 把Date类型的日期转换为毫秒，无参数时相当于System.currentTimeMillis() 

### 9.2 DateFormat类: java.text.DateFormat
- 是一个抽象类。继承自Format抽象类。
- 可以使用它的子类：**SimpleDateFormat类**
- 格式化日期为文本、解析文本为日期
1. `String format(Date date)`: 按照指定模式把日期解析为文本
2. `Date parse(String source)`: 把符合标准的字符串解析为Date日期
3. `SimpleDateFormat`中   
    |符号|含义|
    |----|----|
    |y|年|
    |M|月|
    |d|日|
    |H|时|
    |m|分|
    |s|秒|

    如："yyyy-MM-dd HH:mm:ss", "yyyy年MM月dd日 HH时mm分ss秒"
~~~java
    SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    Date date= new Date();  //得到当前时间
    String standardDate = simpleDateFormat.format(date);
    System.out.println(standardDate);  //2020-07-28 21:24:36

    date = simpleDateFormat.parse(standardDate);   //必须和构造方法中的格式一样
    System.out.println(date);   //Tue Jul 28 21:28:38 CST 2020
~~~

### 9.3 Calendar类: java.util.calendar
- 是一个抽象类，提供了很多操作日历字段的方法

- **无法直接创建对象，里面有一个getInstance()方法返回一个Canlendar类的子类对象**

- `public int get(int field)`: 返回给定日历字段的值
- `public void set(int field , int value)`: 将给定的日历字段设置为给定值
- `public abstract void add(int field, int amount)`: 根据日历规则，给日历字段增减指定的时间量
- `public Date getTime()`: 返回表示此Calendar时间毫秒数的Date对象


    |字段值|含义|
    |----|----|
    |YEAR|年|
    |MONTH|月(从0开始，+1使用)|
    |DAY_OF_MONTH|几号|
    |HOUR|12小时制|
    |HOUR_OF_DAY|24小时制|
    |MINUTE|分|
    |SECOND|秒|


~~~java
    Calendar c = Calendar.getInstance();  //返回Calendar类的子类对象，多态
    int year = c.get(Calendar.YEAR);    //2020，YEAR是常量1，是field
    int month = c.get(Calendar.MONTH);  //6
    int date = c.get(Calendar.DATE);    //28
    System.out.println(year+"-"+(month+1)+"-"+date);  //2020-7-28
~~~

~~~java
    Calendar c = Calendar.getInstance();  //返回Calendar类的子类对象，多态

    c.set(Calendar.YEAR, 2222);       //单独设置某个字段
    c.set(3333, 11,12);               //全部一起设置
    year = c.get(Calendar.YEAR);
    month = c.get(Calendar.MONTH);
    date = c.get(Calendar.DATE);
    System.out.println(year+"-"+(month+1)+"-"+date);  //3330-12-12
~~~

~~~java
    Date ddate = c.getTime();
    System.out.println(ddate);   //Tue Dec 12 22:48:25 CST 3330
~~~

## 10. System类: java.lang.System
- 含有大量的静态方法
- `public static long currentTimeMillis()`: 返回以毫秒为单位的当前时间
- `public static void arraycopy(Object src, int srcPos, Object dest, int destPos, int length)`: 将数组中指定数据拷贝到另一个数组

~~~java
    long startTime = System.currentTimeMillis();
    demo01();
    long endTime = System.currentTimeMillis();
    System.out.println("\n"+"demo01执行了："+(endTime-startTime)+"ms");
~~~

~~~java
    int[] a = {1,2,3,4,5};
    int[] b = {6,7,8,9,10};
    System.arraycopy(a,1,b,0,3);
    System.out.println("a:"+ Arrays.toString(a)+"\nb:"+Arrays.toString(b));
    /*
    a:[1, 2, 3, 4, 5]
    b:[2, 3, 4, 9, 10]
    */
~~~

## 11. StringBuilder类(字符串缓冲区)： java.lang.StringBuilder
- String类的底层定义的是被final修饰的byte数组，创建后不能改变，字符串操作后，内存中会存在大量的空间浪费，效率低下
- StringBuilder可以提高字符串的操作效率，底层数组没有被final修饰
- 构造方法： 无参(空字符串)，带参（传入字符串）
- `public StringBuilder append(...)`: 参数可以是任意类型，返回当前对象自身，无需接受返回值
- `public StringBuilder reverse()`： 反转内容
- `String toString()`: 将缓冲区内容转换为字符串
~~~java
    StringBuilder s1 = new StringBuilder();
    StringBuilder s2 = new StringBuilder("abc");
    StringBuilder s3 = s1.append(s2);  //返回当前对象自身
    System.out.println(s1+"\n"+(s1==s3));

    s1.append(1).append(true).append('啊');
    System.out.println(s1);

    s1.reverse();
    String s4 = s1.toString().toUpperCase();
    System.out.println(s4);
    /*
    abc
    true
    abc1true啊
    啊EURT1CBA
    */
~~~

## 12. 包装类 java.lang
- 基本数据类型的包装类，使基本数据类型像对象一样操作，提供更多的功能
### 12.1 装箱与拆箱
1. 装箱：基本类型->包装类
- 使用构造方法(过时)    
    ~~Integer(int value)~~  
    ~~Integer(String s)~~      //s必须是基本数据类型的字符串
- 使用静态方法  
    `static Integer valueOf(int i)`  
    `static Integer valueOf(String s)`
2. 拆箱  
- 使用成员方法  
    int intValue()
~~~java
    Integer in1 = new Integer(10); //方法过时了
    Integer in2 = new Integer("10"); //方法过时了
    System.out.println(in1+"\n"+in2);

    Integer in3 = Integer.valueOf(1);   //使用静态方法
    int i = in3.intValue();
~~~

### 12.2 自动装箱与拆箱
- 基本类型与包装类之间自动相互转换(jdk1.5)
~~~java
    Integer in = 1  ///自动装箱
    in = in + 2 //自动拆箱后+2，再自动装箱
~~~

- ArrayList集合无法直接存储整数，可以存储Integer包装类，传入整数的时候其实就发生了自动装箱

### 12.3 基本类型和字符串之间的转换
1. 字符串转基本类型
- `public static byte parseByte(String s)`: 字符串转byte
- `public static short parseShort(String s)`: 字符串转short
- `public static int parseInt(String s)`: 字符串转int
...

**除了Character类外**，其他所有包装类都有parseXxx静态方法将字符串转换为对应基本类型

2. 基本类型转字符串
- 基本类型+""：直接与空字符串连接   
- `static toString(基本类型数据)`：包装类的toString()方法，要用对应的包装类
- `static valueOf(基本类型数据)`：String类的静态方法，重载了很多个，不管传入什么基本类型都可以
~~~java
    int i1 = Integer.parseInt("123");
    String s1 = i1+"";
    String s2 = Integer.toString(i1);
    String s3 = String.valueOf(i1);
~~~


# 容器

![aKLR0J.png](https://s1.ax1x.com/2020/07/30/aKLR0J.png)

java容器可分为两大类：
- Collection 
    - List
        - ArrayList
        - LinkedList
        - ~~Vector~~
    - Set
        - HashSet
            - LinkedHashSet
        - TreeSet
- Map
    - HashMap
        - LinkedHashMap
    - TreeMap
    - ConcurrentHashMap
    - ~~Hashtable~~


## 1. Collection集合
- java提供的一种容器，可以存储多个数据，长度可变，存储的都是对象(数组只能存储基本数据类型)
- Collection继承自Iterable接口，Iterable接口允许对象成为for each循环的目标

- `Collection` 三大接口: List,  Set (两种接口), Queue     

1. `List` ：有序，元素可重复，有索引
- `ArrayList`：底层由数组实现，查询快，增删慢
- `LinkedList`：底层由链表实现，查询慢，增删快
- `Vector`：   

2. `Set` : 无序，元素不可重复，无索引
- `TreeSet` : 底层由二叉树实现，一般用于排序
- `HashSet` : 底层由哈希表和红黑树实现
- `LinkedHashSet` : 底层由哈希表和链表实现，可以保证存取顺序

3. `Queue` ： 除Collection基本操作外，提供队列操作
- `Deque` : 
- `PriorityQueue` : 

### Collection常用功能
1.1 添加
- `boolean add(Object o)` :  向集合中添加元素
- `boolean addAll(Collection c)` : 添加一个集合的元素  

1.2 删除
- `boolean remove(O o)` : 从集合中删除元素(最先查找到的，只删除一次)
- `boolean removeAll(Collection c)` : 移除一个集合的元素(至少移除一个元素则true)  
- `void clear()` : 清空集合

1.3 判断
- `boolean contains(O o)` : 是否含有某个元素
- `boolean containsAll(Collection c)` : 是否包含一个集合的所有元素 
- `boolean isEmpty()` : 判断集合是否为空      

1.4 其他
- `int size()` : 获取集合长度
- `object[] toArray` : 集合转数组
- `Iterator<E> iterator()` ： 获取该集合的迭代器对象

~~~java
    Collection<String> coll = new ArrayList<>();
    coll.add("小明");
    coll.add("小芳");
    coll.add("小明");
    coll.remove("小明");
    System.out.println(coll); //[小芳, 小明]  重写了toString()方法
~~~



## 2. Iterator迭代器接口和for each循环
- Collection通用的集合元素获取方式，取出元素前判断是否为空    
- Iterator也是接口，在具体集合中以内部类的方式实现，该集合给出返回Iterator实现类对象的方法   
常用方法：
- `public E next()` : 返回迭代的下一个元素
- `public boolean hasNext()` : 是否还有元素可以迭代
- `Iterator<E> Collection.iterator()` : 返回一个在此集合上的迭代器

~~~java
    //使用迭代器遍历
    Collection<String> coll = new HashSet<>();
    coll.add("小明");
    coll.add("小红");
    coll.add("小芳");
    coll.add("小王");
    Iterator<String> iter = coll.iterator(); //迭代器元素类型必须和集合一样
    while(iter.hasNext()){
        System.out.println(iter.next());
    }
    /*
    小明
    小王
    小红
    小芳
    */
~~~


~~~java
    //使用for each循环遍历(底层也是iterator迭代器)
    for (String s : coll) {
        System.out.println(s);

    }
~~~

## 3. 泛型
- 将数据类型参数化，如 `ArrayList` 定义的时候使用泛型，可以放入指定的任意引用类型   
### 3.1 含有泛型的类和接口
~~~java
public class 类名<E>{
    private E 成员变量名;
    public E 成员方法名(){
        ...
    }
    ...
}
~~~
### 3.2 含有泛型的方法   
`修饰符 <泛型> 返回值类型 方法名(参数列表(含泛型)){}`
~~~java
    public <E> void method(int i, E e){
        for(j = 0; j < i; j++>){
            System.out.println(e)
        }
    }
~~~

### 3.3 泛型通配符
- ？ 代表任意的数据类型，不能创建对象时使用，只能作为方法的参数使用(当参数是含有泛型的类的对象时，增强通用性)
- 因为泛型是不能向上转型的，需要使用泛型通配符
~~~java
    //遍历ArrayList
    public void printArrayList(ArrayList<?> list){       //不知道具体使用时传入的泛型类型
        Iterator<?> iter = list.iterator();
        while(iter.hasNext()){
            System.out.println(iter.next());
        }
    }
~~~
- 泛型上下限的限定：   
`? extends E` : 上限限定，使用的泛型只能是E类型的子类和自身   
`? super E` : 下限限定，使用的泛型只能是E类型的父类和自身   

## 4. Collections工具类
不属于Java框架继承树上的内容，是单独的分支，只包含静态方法，操作或返回 `Collection` 

### 4.1 包装
- 将自动同步(线程安全)添加到任意集合    
`public static Collection synchronizedCollection(Collection c);`   
`public static Set synchronizedSet(Set s);`    
`public static List synchronizedList(List list);`   
`public static <K,V> Map<K,V> synchronizedMap(Map<K,V> m);`   
`public static SortedSet synchronizedSortedSet(SortedSet s);`   
`public static <K,V> SortedMap<K,V> synchronizedSortedMap(SortedMap<K,V> m);`          

- 不可修改的包装，使集合不可变     
`public static Collection unmodifiableCollection(Collection<? extends T> c);`   
`public static Set unmodifiableSet(Set<? extends T> s);`   
`public static List unmodifiableList(List<? extends T> list);`    
`public static <K,V> Map<K, V> unmodifiableMap(Map<? extends K, ? extends V> m);`     
`public static SortedSet unmodifiableSortedSet(SortedSet<? extends T> s);`     
`public static <K,V> SortedMap<K, V> unmodifiableSortedMap(SortedMap<K, ? extends V> m);`   

### 4.2 工具
- 调用一个空List,Set,Map  
`public static final List EMPTY_LIST = new EmptyList<>();`   
`public static final Map EMPTY_MAP = new EmptyMap<>();`   
`public static final Set EMPTY_SET = new EmptySet<>();`

- addAll:向指定的集合c中加入多个特定的元素elements   
`public static <T> boolean addAll(Collection<? super T> c, T… elements)`    

~~~java
    //例如
    Collections.addAll(list, "s5","s7",null,"s9");
~~~

- 二分查找指定的元素binarySearch   
`public static <T> int binarySearch(List<? extends Comparable<? super T>> list, T key)`     

- 排序sort   
`public static <T extends Comparable<? super T>> void sort(List<T> list) `

- 置乱shuffle   
`public static void shuffle(List<?> list）`

- 反转reverse    
`public static void reverse(List<?> list)`

## 5. List
- 有序，元素可重复，有索引
### 5.1 List中带索引的方法
- List的共有方法   
`public void add(int index, E element)`      
`public E set(int index, E element)`   
`public E get(int index)` ： 返回旧值    
`public E remove(int index)` ： 删减数组元素不会改变数组的容量，可以调用 `trimToSIze()`修改  

### 5.2 ArrayList
- List接口实现的大小可变的动态数组    
- 两种`add`方法，直接添加到尾部(底层自动判断溢出和扩容)，或者指定位置(插入，未溢出时直接复制位置之后的元素到下一个位置)
- `newCapacity = oldCapacity + (oldCapacity >> 1);` 每次扩容增加50%，扩容之后调用`copyOf()`方法复制到新数组， `copyOf`调用了`System.arraycopy()`   
- 可以完全替代Vector，只是ArryList线程不安全，多线程使用Vector或使用`Collections.synchronizedList`来实现同步   
~~~java
    List list = Collections.synchronizedList(new ArrayList(...));
~~~
- `private static final int DEFAULT_CAPACITY=10;` 初始容量为10
- 查询快，增删慢，但并不是绝对，在中间位置和尾部增删依然比LinkedList快

### 5.3 Vector      
- 所有单列集合的父类   
- 每次扩容增加一倍(与ArrayList不同)     
- 是同步的，线程安全的容器，对内部每个方法都上锁，开销大，效率远低于ArrayList



### 5.4 LinkedList
- List接口实现的**双向链表**，可以存储任何元素(包括null)，非线程安全   
- 多线程要使用`List list = Collections.synchronizedList(new LinkedList(...));`   
- LinkedList实现了`Deque`接口，可以像**队列和栈**一样操作LinkedList   
- 构造方法： 
    `public LinkedList()` : 建立一个空链表
    `public LinkedList(Collection<? extends E> C)` ： 从集合创建链表(其实就是使用Collection的`addAll`方法)
- 特有方法：    
    `public void addFirst(E e)`     
    `public void addLast(E e)`      
    `public void push(E e)` ： 将元素压栈到列表所表示的栈


    `public E getFirst()`       
    `public E getLast()`        
    `public E removeFirst()` : 返回被删掉的头节点值     
    `public E removeLast()`  
    `public E get(int index)` : 根据index选择从头或尾开始遍历    
    `public E set(int index, E element)` : 遍历阶段同上  
    `public E pop()`        
    `public boolean isEmpty()`



### 5.5 Stack和Deque

- Stack继承自Vector类，提供了`push`和`pop`操作，以及栈顶的`peek`方法，`empty`方法，搜寻与栈顶元素距离的`search`方法 
- 需要注意的是`push`、`pop`、`peek`失败时均会抛出异常
- Stack类一般不建议使用，因为继承自Vector，vector是动态数组，所以**Stack可以对栈内任意位置的元素进行添加删除操作**，违背了栈设计的初衷，破坏了栈的结构，引发安全问题
- 所以基于Vector的栈应该手动进行实现
~~~java
public class Stack<E>{
    private Vector<E> v = new Vector<E();
    //实现栈的相关方法
}
~~~

- 可靠的栈操作由`Deque`接口和它的实现类提供，Deque是双端队列，能在两端进行插入和删除，当然也能在一端进行插入删除操作  
~~~java
    Deque<Integer> stack = new ArrayDeque<Integer>();
~~~
- 然而，我们声明的仍然**是一个Deque**，可以在两端进行插入和删除！！
- 目前java官方只做到这个份上。一般用Deque，为了更安全可以自己再封装一层

<center>

![dohevd.png](https://s1.ax1x.com/2020/08/28/dohevd.png)
</center>

### 5.6 Queue和Deque、ArrayDeque
#### 5.6.1 Queue
- 队列(queue)是一种常用的数据结构，可以将队列看做是一种特殊的线性表，该结构遵循的先进先出原则。
- Queue接口提供了添加元素操作`offer`、删除并返回第一个元素`poll`、返回第一个元素`peek`
- **注意：** `add`、`remove`、`element`也可以实现上述功能，但是对于空和满的状态**会抛出异常**，所以一般使用offer系列操作
~~~java
public interface Queue<E> extends Collection<E> {
    
    boolean add(E e);       //往队列插入元素，如果出现异常会抛出异常 
    boolean offer(E e);     //往队列插入元素，如果出现异常则返回false

    E remove();             //移除队列元素，如果出现异常会抛出异常
    E poll();               //移除队列元素，如果出现异常则返回null

    E element();            //获取队列头部元素，如果出现异常会抛出异常
    E peek();               //获取队列头部元素，如果出现异常则返回null
}
~~~

#### 5.6.2 Deque
- 双向队列(Deque),是Queue的一个子接口，双向队列是指该队列两端的元素既能入队(offer)也能出队(poll),如果将Deque限制为只能从一端入队和出队，则可实现栈的数据结构
- Java中，**LinkedList实现了Deaue接口，Deque继承自Queue**, 因为LinkedList进行插入、删除操作效率较高
~~~java
public interface Deque<E> extends Queue<E> {
    void addFirst(E e);     //插入头部，异常会报错
    boolean offerFirst(E e);//插入头部，异常返回false

    E getFirst();           //获取头部，异常会报错
    E peekFirst();          //获取头部，异常不报错
    
    E removeFirst();        //移除头部，异常会报错
    E pollFirst();          //移除头部，异常不报错
    
    void addLast(E e);      //插入尾部，异常会报错
    boolean offerLast(E e); //插入尾部，异常返回false
    
    E getLast();            //获取尾部，异常会报错
    E peekLast();           //获取尾部，异常不报错
    
    E removeLast();         //移除尾部，异常会报错
    E pollLast();           //移除尾部，异常不报错
}
~~~
- 一般直接使用LinkedList
  
~~~java
    Deque<String> deque = new LinkedList<String>();
~~~

#### 5.6.3 ArrayDeque 

- ArrayDeque也实现了Deque，拥有队列或者栈特性的接口
- 实现了Cloneable，拥有克隆对象的特性
- 实现了Serializable，拥有序列化的能力
~~~java
public class ArrayDeque<E> extends AbstractCollection<E>
                       implements Deque<E>, Cloneable, Serializable{}
~~~

- ArrayDeque 底层使用数组存储元素，同时还使用head和tail来表示索引，但注意tail不是尾部元素的索引，而是尾部元素的下一位，即下一个将要被加入的元素的索引
- 对于只操作头尾的情况，ArrayDeque的性能要优于LinkedList，尤其是对于数据量比较大的时候



### 5.7 PriorityQueue优先级队列
- AbstractQueue的实现类，元素自然排序或通过构造函数时期提供的Comparator排序，不允许null元素

- 优先级队列底层通过堆实现 **(完全二叉树实现的小顶堆)**

- 队列的头是指定顺序的最后一个元素 poll, remove, peek, element访问的都是队列头部
 
- 队列使用场景： Top K 问题、维护数据流中的中位数  


## 6. Set
- 无序，元素不可重复，无索引，不可用带索引的方法
- 使用迭代器Iterator或者for each遍历

1. HashSet：无序，允许为null，底层是HashMap(散列表+红黑树)，非线程同步
2. TreeSet：有序，不允许为null，底层是TreeMap(红黑树),非线程同步
3. LinkedHashSet：迭代有序，允许为null，底层是HashMap+双向链表，非线程同步

### 6.1 HashSet
- jdk1.8之前，哈希冲突用链表解决，1.8之后超过长度8的链表转为红黑树来处理哈希冲突
- `HashSet` 是 `HashMap` 的一个实例，**不保证集合的迭代顺序**
- 这个实现不是线程安全的，多线程应该使用`Collections.synchronizedSet()`方法重写
- 集合元素可以是null,但只能放入一个null
- HashSet集合判断两个元素相等的标准是两个对象通过equals方法比较相等，并且两个对象的hashCode()方法返回值相等,**所以在重写了equals方法之后也应该重写hashCode方法，equals返回true时，hashCode也应该相同**
- `HashSet` 底层实际上是一个 `HashMap` 实例，`value` 是一个 `Object` ，所有 `key` 的 `value` 都是它     

### 6.2 TreeSet
- 基于TreeMap的NavigableSet实现，使用自然排序或创建时提供的Comparator进行排序   
- 是SortedSet接口的唯一实现类，确保集合元素处于排序状态
- 为基本操作`add`、`remove`、`contains`提供了**logn**的时间复杂度
- 这个实现不是线程安全的。多线程使用`SortedSet s = Collections.synchronizedSortedSet(new TreeSet(...));`
- `TreeMap` 底层实际上是一个 `TreeMap` 实例， `value` 也是 `Object` 

### 6.3 LinkedHashSet

- 使用hashCode决定元素位置，同时使用链表维护元素次序，所以在遍历的时候性能比HashSet好，但是插入时不如HashSet
- 迭代次数不受容量影响，选择过高的初始容量的开销比HashSet小
- 底层实际上是 `HashMap` 和双向链表实现，其实就是 `LinkedHashMap` 

## 7. Map
- 元素是 key : value 的键值对，一个 key 只能对应最多一个 value，多个 key 可以对应相同的 value
- key 是唯一的， value 可以重复
### 7.1 Map集合的功能
- 添加：
- `V put(K key, V value)` ： 添加元素，如果 key 是第一次存储，返回null，否则替换并返回以前的 value 

- 删除：
- `void clear()`   
- `V remove(Object key)` : 根据 key 删除对应 value， 并返回 value

- 判断：
- `boolean containsKey(Object key)`   
- `boolean containsValue(Object value)`
- `boolean isEmpty`  

- 获取：
- `Set<Map.Entry<K key, V value>> entrySet()` : 返回此映射中包含的映射关系的Set视图，Map.Entry表示映射关系，entrySet()：迭代后可以通过e.getKey()，e.getValue()取key和value。返回的是Entry接口 。
- `V get(Object key)` : 根据 key 获得 value       
- `Set<K> keySet()` : 获取集合中所有 key 的集合 //key是唯一的，所以用Set
- `Collection<V> values()` : 获取集合中所有 value 的集合  //value不是唯一的，所以用Collection   

- `entrySet` 方式遍历 Map 的性能比 `keySet` 好


- 长度：
- `int size()`

- 遍历：

~~~java
//keySet方式遍历
    Map<String,String> map = new HashMap<String,String>();
                
    map.put("01", "zhangsan");
    map.put("02", "lisi");
    map.put("03", "wangwu");
            
    Set<String> keySet = map.keySet();//先获取map集合的所有键的Set集合

    Iterator<String> it = keySet.iterator();//有了Set集合，就可以获取其迭代器。
            
    while(it.hasNext()){
        String key = it.next();
        String value = map.get(key);//有了键可以通过map集合的get方法获取其对应的值。        
        System.out.println("key: "+key+"-->value: "+value);//获得key和value值
    }



//entrySet方式遍历
    Map<String,String> map = new HashMap<String,String>();
                
    map.put("01", "zhangsan");
    map.put("02", "lisi");
    map.put("03", "wangwu");

    //通过entrySet()方法将map集合中的映射关系取出（这个关系就是Map.Entry类型）
    Set<Map.Entry<String, String>> entrySet = map.entrySet();

    //将关系集合entrySet进行迭代，存放到迭代器中                
    Iterator<Map.Entry<String, String>> it2 = entrySet.iterator();
                
    while(it2.hasNext()){
        Map.Entry<String, String> me = it2.next();//获取Map.Entry关系对象me
        String key2 = me.getKey();//通过关系对象获取key
        String value2 = me.getValue();//通过关系对象获取value
                        
        System.out.println("key: "+key2+"-->value: "+value2);
    }

    //或者使用 for each 更方便
    for (Map.Entry<String, String> entry : map.entrySet()) {
        String s = "key: " + entry.getKey() + "---->value: " + entry.getValue();
        System.out.println(s);
    }


~~~





### 7.2 HashMap 
- 特点： 无序，允许null，非同步   
- 底层是: 数组+散列表+红黑树   
- 初始容量为16，最大容量 2^30， 默认装载因子为 0.75
- `TREEIFY_THRESHOLD = 8` ： 控制桶中链表最多的节点数，超出则转为树形结构，默认为 8
- `UNTREEIFY_THRESHOLD = 6` ： 树形结构转链表的阈值，默认为 6
- `MIN_TREEIFY_CAPARITY = 64` ： 转树形结构的最小散列表容量



- 构造方法：
- `public HashMap(int initialCapacity, float loadFactor)`       
- `public HashMap()`
### 7.3 HashTable 
- 和HashMap实现基本相同，但线程安全，不允许 key 和 value 为 null，过时的类，需要线程安全时用`ConcurrentHashMap`即可

### 7.4 LinkedHashMap  
- 特点：插入有序，允许null，不同步    
- 底层是散列表和双向链表
- 构造方法：
- `public LinkedHashMap(int initialCapacity, float loadFactor)`
- `public LinkedHashMap()`
- `public LinkedHashMap(Map<? extends K, ? extends V> m)`
- `public LinkedHashMap(int initialCapacity, float loadFactor, boolean accessOrder)`
- `accessOrder`：the ordering mode， true for access-order, false for insertion-order 默认均为false，按插入的顺序遍历
- 在access-order的情况下，访问一次就会修改顺序，最常用的放在最后面

### 7.5 TreeMap 
- TreeMap 实现了`NavigableMap` 接口，而 `NavigableMap` 接口继承自 `SortedMap` 接口，所以TreeMap是有序的  
- TreeMap 底层是红黑树，时间复杂度logn    
- 非同步
- 使用 `Comparator` 或者 `Comparable` 比较 key 是否相等和排序   
- 构造方法：
- `public TreeMap()` : 
- `public TreeMap(Comparator<? super K> comparator)`   
- `public TreeMap(Map<? extends K, ? extends V> m)` : 使用 putAll 将 Map 转 TreeMap   
- `public TreeMap(SortedMap<K, ? extends V> m)`  
- 如果 `Comparator` 不为 `null`，则使用 `Comparator.compare()` 比较，否则使用 `key` 作为比较器进行比较，但是 `key` 必须实现 `Comparable` 接口

~~~java

    //构造时传入Comparator的例子
    TreeMap<Student, String> map = new TreeMap<Student, String>(new Comparator<Student>() {
        @Override
        public int compare(Student o1, Student o2) {
            int num = o1.getAge() - o2.getAge();
            int num2 = (num == 0) ? o1.getName().compareTo(o2.getName()) : num;
            return num2;
        }
    });

    Student s1 = new Student("小明", 13);
    Student s2 = new Student("小红", 10);
    map.put(s1, "一班");
    map.put(s2, "二班");

    //map.put(null, "一班");   使用compareTo 比较，不能传入 null 的 key

    //遍历集合
    for ( Map.Entry<Student, String> entry : map.entrySet()) {
        System.out.println(entry.getKey() + "--->" + entry.getValue());
    }
~~~

### 7.6 ConcurrentHashMap



# 异常  

![aMVOtP.png](https://s1.ax1x.com/2020/07/30/aMVOtP.png)

## 1. Throwable 
- `Throwable` 类是Java所有 `Error` 和 `Exception` 的父类，只有 `Throwable` 的子类才可以被抛出    

- `uncheckedException` : 包含 `RuntimeException` 和 `errors`，指的是程序的瑕疵或逻辑错误，并且在运行时无法恢复，包括系统异常，语法上不需要声明抛出异常，不处理异常也能通过编译，JVM 会自行处理。

- checkedException：代表程序不能直接控制的无效外界情况（如用户输入，数据库访问，网络异常，文件访问和丢失等），除了Error和RuntimeException及其子类之外的异常， 需要try catch处理或throws声明抛出异常。

- 异常和错误的区别：异常能被程序本身处理，错误是无法处理

###  常用的Throwable方法
1. 抛出异常的详细信息  
`public string getMessage();`    
`public string getLocalizedMessage();`    
`public Throwable getCause();`   返回一个Throwable 对象代表异常原因。

2. 返回异常发生时的简要描述   
`public public String toString();`    
   
3. 打印异常信息到标准输出流   
`public void printStackTrace();`   打印错误输出流   

4. 记录栈帧的当前状态   
`public synchronized Throwable fillInStackTrace();`

## 2. Exception   
- `Exception` 继承自 `Throwable`， 有两种异常 `RuntimeException` 和 `CheckedException`，这两种异常都应该去捕获

### 2.1 常见的 RuntimeException 举例  
- 程序的瑕疵或逻辑错误，并且在运行时无法恢复。      
`ArrayIndexOutOfBoundsException`   
`IndexOutOfBoundsException`        
`NullPointerException`   
`IllegalArgumentException` : 非法参数     
`NegativeArraySizeException` : 数组长度为负     
`IllegalStateException`     
`ClassCastException` : 类型转换异常
`ArithmeticException` : 算数异常，如除0错误

### 2.2 常见的 CheckedException 举例
- 程序不能直接控制的无效外界情况(用户输入，数据库问题，网络异常，文件丢失等)，需要`try...catch`处理或`throws`声明抛出    
`NoSuchFieldException` : 请求的变量不存在     
`NoSuchMethodException` : 请求的方法不存在      
`IllegalAccessException` : 不允许访问某个类的异常      
`ClassNotFoundException` ： 应用程序试图加载类时，找不到相应的类，抛出该异常    
`InterruptedException` : 一个线程被另一个线程中断时抛出该异常    
`ServletException`    
`SQLException`      
`IOException`

### 2.3 throws和throw关键字   
- `throws` 在方法尾部，声明该方法会抛出这种类型的异常，使它的调用者知道要捕获这个异常     
- `throw` 在方法体内，表示抛出异常的具体动作，抛出一个异常的实例   
- 一个方法尾部可以声明抛出多个异常
- 如果我们不通过 `try…catch` 来处理该异常，那么我们就不得不在函数声明中通过 `throws` 标明该函数会抛出异常
~~~java
   public static void method() throws ArrayIndexOutOfBoundsException,IndexOutOfBoundsException{
        int i = new Random().nextInt(1);
        if (i == 0){
            throw new ArrayIndexOutOfBoundsException();
        }
        else{
            throw new IndexOutOfBoundsException();
        }
    }
~~~

### 2.4 try和catch关键字
- 使用 try 和 catch 关键字可以捕获异常。try/catch 代码块放在异常可能发生的地方。try/catch 代码块中的代码称为保护代码

- catch 语句包含要捕获异常类型的声明。当保护代码块中发生一个异常时，try 后面的 catch 块就会被检查。如果发生的异常包含在 catch 块中，异常会被传递到该 catch 块，这和传递一个参数到方法是一样。
~~~java
    public static void method(){   //使用try...catch 之后可以不再声明函数会抛出异常
        int[] a = {1,2};
        try{
            System.out.println(a[2]);
        }
        catch(Exception e){
            System.out.println("出错了：" + e);   //打印异常信息
            e.printStackTrace();                 //打印错误输出流
            throw e;                             //打印错误输出流并停止程序
        }
    }
~~~
- `throw` 是抛出异常给虚拟机处理，虚拟机处理的方式是打印异常，并在异常处终止程序的运行   
- 程序使用catch捕捉异常时，不能随心所欲地捕捉所有的异常，程序可在任意想捕捉的地方捕捉RuntimeException异常、Exception,但是对于其他Checked异常，只能当try块可能抛出异常单时(try块中调用的某个方法声明抛出了该Checked异常),catch块才能捕捉该checked异常

### 2.5 finally关键字

- `try...finally` 表示一段代码不管执行情况如何，都会执行finally中的代码    
- `try...catch...finally` 表示捕获一场之后再执行finally中的代码
- try 代码后不能既没 catch 块也没 finally 块
- 即使 catch 里面有 `return` 语句，finally中的代码依然会被执行
~~~java
    try{
        //待捕获代码
    }catch（Exception e）{
        System.out.println("catch is begin");
        return 1 ；
    }finally{
        System.out.println("finally is begin");   //先执行catch里面的代码后执行finally里面的代码最后才return
    }
    /*
    catch is begin
    finally is begin 
    */
~~~
- `finally` 中的代码均在 `catch` 中的 `return` 前执行
~~~java
    try{
    //待捕获代码    
    }catch（Exception e）{
        System.out.println("catch is begin");
        return 1 ；
    }finally{
        System.out.println("finally is begin");
        return 2 ;
    }
    //最后返回的是 2
~~~


### 2.6 自定义异常
- 所有异常都必须是 `Throwable` 的子类
- 如果希望写一个检查性异常类，则需要继承 `Exception` 类
- 如果你想写一个运行时异常类，那么需要继承 `RuntimeException` 类

# 反射    

## 1. 基本概念

### 1.1 动态语言和静态语言   
- 动态编程语言(Dynamic Programming Language) ： 是指程序在运行时可以改变其结构，新的函数可以引进，已有的函数可以被删除等结构上的变化，如增删函数、对象、代码等(JavaScript、Objective-C、Ruby、Python等)     
- 静态编程语言(StaticProgramming Language) : 在运行时不能改变其结构，尽管静态语言可以通过复杂的手段实现动态语言的特性，但是动态语言提供了直接的方法实现语言的动态性(C、C++) 
~~~javascript
    //动态语言举例（javascript）
    function Person(name, age, job)
    {
    this.name = name;
    this.age = age;
    this.job = job
    this.hello = function(name){
        alert("Hello, " + name);
    };

    person = new Person("Eric", 28, 'worker');
    alert(person.name + '' + person.age + '');
    person.hello("Alice");
    //为对象添加方法
    person.work = function(){
    alert('I am working');
    }
    person.work();

    //删除方法
    delete person.work;
    person.work();
~~~

- 动态类型语言(Dynamically Typed Language) ： 运行时才做数据类型检查的语言(JavaScript、Ruby、Python)       
- 静态类型语言(Statically Typed Language) : 编译时进行数据类型检查，写程序时必须声明变量的数据类型(JAVA, C, C++, C#)  
- 强类型语言，一个变量不经过强制转换，它永远是这个数据类型，不允许隐式的类型转换。举个例子：如果你定义了一个double类型变量a,不经过强制类型转换那么程序int b = a无法通过编译。典型代表是Java、Python
- 弱类型语言：它与强类型语言定义相反,允许编译器进行隐式的类型转换，典型代表C/C++  
~~~python
    #动态类型语言python
    def sum(a, b):

        return a + b;

    print sum(1,2);
    print sum("Hello ", "Word")
~~~
<center>   </center> 

![aQ9fIO.png](https://s1.ax1x.com/2020/07/31/aQ9fIO.png)   

</center>    

### 1.2 反射
- 反射机制：    
Java 反射机制是指在运行状态中，对于任意一个类都能够知道这个类所有的属性和方法；并且对于任意一个对象，都能够调用它的任意一个方法；这种动态获取信息以及动态调用对象方法的功能成为 Java 语言的反射机制。**其实就是将类的各个组成部分封装为其他对象**  

- 反射机制提供的功能    
    1. 运行时判断任意一个对象所属的类
    2. 运行时构造任意一个类的对象
    3. 运行时知道任意一个类的所有成员变量和方法   
    4. 运行时调用任意一个对象的方法

- Java通过反射机制，可以在程序运行时加载，探知和使用编译期间完全未知的类，并且可以生成相关类对象实例，从而可以调用其方法或者改变某个属性值。所以JAVA也可以算得上是一个半动态的语言

### 1.3 反射的应用场景：
- Java中很多对象都有两种类型：编译时类型和运行时类型，编译时类型由声明对象时的类型决定，运行时类型由实际赋值给对象的类型决定，例如：    
~~~java
    Person p = new Student();
    //编译时类型类Person，运行时类型为Student
~~~
- 所以程序需要在运行时找到类和对象的真实信息，但是编译时是无法知道的，只能依靠运行时信息，于是便需要反射机制    
### 1.4 Java代码在计算机中运行的三个阶段

1. 源代码阶段    
- `.java` 文件通过 `javac` 命令被编译为 `.class` 字节码文件，按照**成员变量**，**构造方法**和**成员方法**分为三个区域。        
2. Class类对象阶段     
- 通过类加载器 `ClassLoader` 将 `.class` 加载到内存中     
- 内存中通过 `Class` 类对象描述字节码文件:    
成员变量被封装为 `Field` 对象(Field[] fields)     
构造方法被封装为 `Constructor` 对象(Constructor[] cons)     
成员方法被封装为 `Method` 对象(Method[] methods)
3. Runtime运行时阶段   
- 通过Class类对象创建原本的类的对象


## 2. Class类
- Class 类没有 public 构造器   
### 2.1 获取Class对象   
1. 调用对象的 getClass()方法  
~~~java
    Person p = new Person();
    Class clazz = p.getClass();
~~~
2. 调用类的class属性
~~~java
    Class clazz = Person.class;
~~~
3. 使用Class类的 `forName()` 静态方法 **(最安全，性能最好)**   
~~~java
    Class clazz = Class.forName("java.lang.Thread"); //类的全路径 
~~~
### 2.2 Class类的方法
1. `String toString()` ：将对象转换为字符串
~~~java
    public String toString() {
        return (isInterface() ? "interface " : (isPrimitive() ? "" : "class "))
            + getName();
    }
~~~

2. `String toGenericString()` : 返回类的全限定名称，包括修饰符和类型参数信息
~~~java
    Class clazz = Class.forName("java.util.Random");
    System.out.println(clazz);  //class java.util.Random
    System.out.println(clazz.toGenericString());  //public class java.util.Random
~~~

3. `static Class<?> forName(String className)` : 根据类名获得一个Class对象，加载类   
~~~java
    Class t = Class.forName("java.lang.Thread"); //加载Thread类，获得Class对象
~~~

4. `T newInstance()` : 创建一个类的实例，代表这个类的对象， `forName()`对类进行初始化，`newInstance()`对类进行实例化,将Class类中的forName和newInstance配合使用，可以根据存储在字符串中的类名创建一个对象，但是**只能使用无参构造创建对象**
~~~java
    String s = "java.util.Date";
    Object m = Class.forName(s).newInstance();
~~~

5. `Method[] getDeclaredMethods()` : 获取类的所有方法信息   
6. `Method getDeclaredMethod(String name, Class<?>... parameterTypes)` : 获取某个方法
7. `Method[] getMethods()` : 获得该类所有公有方法
8. `Method getMethod(String name, Class<?>... parameterTypes)` : 获得该类某个共有方法
9. `Class<?>[] getDeclaredClasses()` : 获得类的所有类和接口(其他同上) 
10. `Field[] getDeclaredFields()` ： 获得类的所有属性(其他同上) 
11. `Constructor[] getDeclaredConstructors()` ： 获得类的所有构造方法信息(其他同上) 
12. `Annotation[] getDeclaredAnnotations()` ： 获得类的所有注解(其他同上)  


- 抛出错误     
`NoSuchMethodException `  原因：没有找到所要查询的Method对象      
`NullPointerException`   原因：所要查询的Method对象的名称为null    
`SecurityException`    原因：调用的类或其父类没有调用权限
~~~java
    //看看成员变量
    Class clazz = Class.forName("cn.reflect.Student");
    Field[] fieldsAll = clazz.getDeclaredFields();  //获取所有成员变量
    for(Field field:fieldsAll){
        System.out.println(field);
    }
    /*
    public java.lang.String cn.reflect.Student.id
    private java.lang.String cn.reflect.Student.name
    private java.lang.String cn.reflect.Student.sex
    public java.lang.String cn.reflect.Student.address
    */

    Field[] fields = clazz.getFields();          //获取public成员变量
    for(Field field:fields){
        System.out.println(field);
    }
    /*
    public java.lang.String cn.reflect.Student.id
    public java.lang.String cn.reflect.Student.address
    */


    //看看构造
    Constructor[] constructors = clazz.getConstructors();
    for(Constructor constractor:constructors){
        System.out.println(constractor);
    }
    /*
    public cn.reflect.Student()
    public cn.reflect.Student(java.lang.String,java.lang.String,java.lang.String,java.lang.String)
    */


    //创建对象
    Constructor constructor = clazz.getConstructor(String.class, String.class, String.class, String.class);
    Object student1 = constructor.newInstance("11S11111", "小明", "男", "深圳");

~~~

## 3. Field类  
- 提供类或接口中单独字段的信息，以及对单独字段的动态访问   
1. `equals(Object obj)` ： 属性与 obj 相等则返回 true
2. `get(Object obj)` ： 获得 obj 中对应的属性值   
3. `set(Object obj, Object value)` : 设置 obj 中对应的属性值  
4. 通过 `setAccessible(true)` 暴力反射，忽略权限修饰符，可以访问和设置私有成员(私有构造，私有方法同理)


~~~java
    
	//修改public的成员变量
    Field address = clazz.getDeclaredField("address");
    Object value = address.get(student1);
    System.out.println(value);    //深圳
    address.set(student1, "北京");
    System.out.println(student1);  //Student{id='11S111111', name='小明', sex='男', address='北京'}


    //暴力反射，获取修改私有成员
    Field sex = clazz.getDeclaredField("sex");
    System.out.println(sex);  //private java.lang.String cn.reflect.Student.sex
    //sex.get(s1);            //IllegalAccessException
    sex.setAccessible(true);  //暴力反射,忽略权限修饰符
    sex.set(student1, "女");
    System.out.println(student1);  //Student{id='11S111111', name='小明', sex='女', address='北京'}
~~~

## 4. Method类  
1. `invoke(Object obj, Object... args)` : 传递obj对象及参数调用该对象对应的方法  
2. `String getName()` : 获取方法名称
- 抛出错误   
`IllegalAccessException` 原因：Method对象强制Java语言执行控制 或 无权访问obj对象   
`IllegalArgumentException` 原因：方法是实例化方法，而指定需要调用的对象并不是实例化后的类或接口   

~~~java
	//调用一下like方法
    Method method1 = clazz.getMethod("like", String.class);
    System.out.println(method1);  //public void cn.reflect.Student.like(java.lang.String)
    System.out.println(method1.getName());   // like  
    method1.invoke(student1, "小红");     //小明喜欢小红
~~~



# Lambda表达式和Stream流   


## 1. Lambda表达式(jdk1.8)
- 函数式编程的思想    
- 把代码赋值给变量，在java8之前是做不到的，java8可以利用Lambda表达式的特性做：
~~~java
    printFunction = public void print(String s){
        System.out.println(s);
    }
~~~
- 这里面有很多多余的内容：`public`, `void`(编译器自动判断返回值类型), `print`(函数名已经赋值给printFunction), `{ }`在一行代码时不必写    
再加上一个`->`，于是得到表达式：`printFunction = (String s) -> System.out.println(s);`    
- 在 java8 里 `printFunction` 的类型是 `Interface` ，所以 Lambda 表达式**其实是一个接口的实现**，但是Lambda表达式只能重写一个方法，把这种只有一个接口函数需要实现的接口类型成为 **函数式接口**， 加上@FunctionalInterface   
- @FunctionalInterface注解,是对函数式接口的标识，他的作用是对接口进行编译级别的检查，如果一个接口使用了这个注解，但是写了两个抽象方法，会出现编译错误  

### 1.1 Lambda格式   
- `(parameters) -> expression`
- `(parameters) -> {statements;}`
- 可以显式声明参数类型，也可以由编译器自动从上下文推断参数类型
- 表达式正文如果只有一条语句，可以不加 { }
- 一个参数可以不加 ( ) :  `a -> a*a`
### 1.2 基本用法
~~~java
    // 接受2个参数,并返回他们的差值
    (x, y) -> x * y

    //接收2个int型整数,返回他们的和
    (int x, int y) -> x + y

    //接受一个 String 对象,打印
    (String s) -> System.out.print(s)

    //创建一个比较器
    Comparator c = (Person p1, Person p2) -> p1.getAge().compareTo(p2.getAge())
    //或者使用类型推断
    Comparator c = (p1, p2) -> p1.getAge().compareTo(p2.getAge())
~~~

- 容器中的例子，当方法的参数需要传入函数式接口对象时：
~~~java
    //TreeMap中的例子，构造时需要传入Comparator的实现类
    //按匿名内部类的做法，需要创建对象重写方法：
    TreeMap<Student, String> map = new TreeMap<Student, String>(new Comparator<Student>() {
        @Override
        public int compare(Student o1, Student o2) {
            int num = o1.getAge() - o2.getAge();
            int num2 = (num == 0) ? o1.getName().compareTo(o2.getName()) : num;
            return num2;
        }
    });

    
    //使用Lambda表达式：
    TreeMap<Student, String> map = new TreeMap<Student, String>((o1, o2) -> {
        int num = o1.getAge() - o2.getAge();
        int num2 = (num == 0) ? o1.getName().compareTo(o2.getName()) : num;
        return num2;
    });


    Student s1 = new Student("小明", 13);
    Student s2 = new Student("小红", 10);
    map.put(s1, "一班");
    map.put(s2, "二班");

    //map.put(null, "一班");   使用compareTo 比较，不能传入 null 的 key

    //遍历集合
    for ( Map.Entry<Student, String> entry : map.entrySet()) {
        System.out.println(entry.getKey() + "--->" + entry.getValue());
    }    
~~~
- Lambda表达式不会创建匿名内部类的 .class文件，节省了空间

### 1.3 函数式接口包 java.util.function.*  
- 在实际使用时，Java8为我们提供了丰富的函数式接口包，并不需要我们每次使用 Lambda 表达式就去定义一个 Interface 
1. `Consumer<T>` 消费型接口   
该接口的抽象方法是 `void accept(T t)`，是单参数无返回值的方法，参数是泛型类   
~~~java
    Consumer consumer = (a) -> System.out.println("this is " + a);
    consumer.accept("123");  //this is 123
~~~
2. `Supplier<T>` 供给型接口   
该接口的抽象方法是 `T get()`，无参数返回泛型类
~~~java
    Supplier<String> supplier = () -> "abc";
    String result = supplier.get();
    System.out.println(result);   //abc
~~~
3. `Predicate<T>` ，抽象方法 `boolean test(T t)` 接受一个参数，返回布尔值
4. `Function<T, R>` , 抽象方法 `R apply(T t)` 接收一个参数，返回一个参数  
5. `UnaryOperator<T>` , 接收和返回一个同类型参数   

6. `BiConsumer<T, U>` , 接收两个参数的消费型接口
7. `BiPredicate<T, U>`
8. `BiFunction<T, U, R>`   
9. `BooleanSupplioer`  

- `Consumer` 可以调用 `andThen` 方法进行两次消费：`con1.andThen(con2).accept(t)`，con1先消费，con2再消费   
- `Predicate` 可以调用 `and`、`or`、`negate`方法，进行判断的逻辑组合: `pre1.and(pre2).test(t)`    
- `Function` 也可以调用 `andThen` 方法    

### 1.4 Lambda表达式的延迟执行特性
- 使用 Lambda 表达式的主要原因是：将代码的执行延迟到一个合适的时间点，即调用的时候   
​所有的 Lambda 表达式都是延迟执行的，**因为匿名内部类的方法都是要等到调用的时候才会执行**   
~~~java
    public static void main(String[] args) {
        String s1 = "hello ";
        String s2 = "java";
        String s3 = "!";
        myFunction(new Random().nextInt(2), s1+s2+s3);   //无论int是否为1都会执行拼接

        myFunction(new Random().nextInt(2), ()->s1+s2+s3);   //只有当int为1才调用 get() 方法拼接字符串
    }

    static void myFunction(int i, String s){
        if(i == 1){
            System.out.println(s);
        }
    }
    static void myFunction(int i, Supplier<String> s){
        if(i == 1){
            System.out.println(s.get());
        }
    }
~~~

## 2. 方法引用
- 当我们使用 `Lambda` 表达式的时候，如果我们指定的操作方案，已经在其他地方被实现了，就没必要写重复逻辑，只需要引用对应的方法即可   
- 双冒号（::）操作符是 Java 中的方法引用。 当们使用一个方法的引用时，目标引用放在 :: 之前，目标引用提供的方法名称放在 :: 之后，即 `目标引用::方法`  
~~~java
    Comparator c = Comparator.comparing(Person::getAge);
~~~
- 方法引用结合Stream流可以让代码更优雅 

### 方法引用的使用

- 通过对象名引用成员方法   
- 通过类名引用静态成员方法
- 通过`super`直接引用父类方法
- 通过`this`引用本类方法
~~~java
    Stream<Integer> stream1 = Stream.of(1,2,3,4,5);
    stream1.map(new Random()::nextInt).forEach(System.out::print);
~~~

- 类的构造器引用  
~~~java
    @FunctionalInterface
    public interface StudentBuilder {
        //定义根据传递的姓名、年龄、性别创建Student对象的方法
        Student buildStudent(String name, int age, String sex);
    }
    public class Demo {
        public static void printStudent(String name, int age, String sex, StudentBuilder sb){
            Student s = sb.buildStudent(name, age, sex);
            System.out.println(s);
        }
        public static void main(String[] args) { 

            //传入Lambda表达式
            printStudent("小明",12,"男",(String name, int age, String sex)->new Student(name, age, sex));
            
            //传入方法引用
            printStudent("小红",13,"女",Student::new);
        }
    }

~~~


## 3. Stream流    
### 3.1 Stream流思想
- 几乎所有的集合都支持遍历，但是循环再循环，每次都要手动 `for each` 或使用 `Iterator` 迭代。 `Stream` 可以让这个过程更优雅，`Stream` 开启后可以直接对单个元素进行操作   
- `Stream.filter(Predicate<T>)` 需要传入一个`Predicate`函数式接口判断进行过滤  

~~~java
    List<Student> students = Arrays.asList(
            new Student("张小明", 10, "男"),
            new Student("刘小明", 13, "男"),
            new Student("陈小明", 15, "男"),
            new Student("张小红", 13, "女"),
            new Student("刘小红", 12, "女"),
            new Student("陈小红", 16, "女"),
            new Student("张小芳", 14, "女"),
            new Student("刘小芳", 11, "女"),
            new Student("陈小芳", 12, "女"));
    students.stream().filter(student->student.getName().startsWith("张"))
    .filter(student->student.getAge()>12)
    .filter(student->student.getSex()=="女")
    .forEach(System.out::println);
    /*
    Student{name='张小红', age=13, sex='女'}
    Student{name='张小芳', age=14, sex='女'}
    */
~~~
- 集合元素处理时是对模型进行操作，集合元素并未改变，只有终结方法执行的时候(需要使用模型中数据)，整个模型才会按照指定的策略执行操作，这得益于 `Lambda` 的延迟执行特性     
- `Stream` 流是一个集合元素的函数模型，并不是集合，也不是数据结构，**其本身并不会存储任何元素和元素地址**，`Stream` 流的元素只有在需求的时候才进行迭代计算，用户仅仅从流中提取到需要的值，所以只能使用 `for each` 循环遍历一次 `stream`，再次遍历会抛出异常   
- 流的数据来源可以是集合 `Collection` 、数组 `Array` 等

### 3.2 流的获取   
- 所有的 `Collection` 都可以通过自身的 `stream()` 方法获取流
- `public static<T> Stream<T> of(T... values)` : 通过`Stream` 接口的静态方法 `of` 可以获取数组对应的流，但是`Stream.of()`参数是泛型，**必须传入包装类的数组**(自动装箱只对基本类型，虽然Array存的是int，但是传的是Array本身，并不是基本类型，不能自动装箱)     
~~~java
    Integer[] a = {1,2,3,4,5};    //Stream.of()参数是泛型，必须传入包装类的数组
    Stream<Integer> stream1 = Stream.of(a);
    Stream<String> stream2 = Stream.of("a", "b", "c");

~~~
- `Map` 可以通过 `keySet`、 `values` 或者 `entrySet` 调用 `stream()`   

### 3.1 常用方法   
1. 延迟方法：返回值仍然是 `Stream` 接口自身类型，支持链式调用    
- `Stream<T> filter(Predicate<? super T> predicate)` : 利用 `filter` 进行谓词筛选，传入一个`Predicate` 
- `Stream<T> distinct()` : 利用`distinct()`可以进行去重   
- `Stream<T> limit(long maxSize)` : 截短流，只获取流中前 `maxSize` 个元素  
- `Stream<T> skip(long n)` : 跳过元素流中前 `n` 个元素     
- `<R> Stream<R> map(Function<? super T, ? extends R> mapper)` : 映射， `map()` 传入一个`Function`把一个对象转换为另一个   
~~~java  
    Stream<String> stream2 = Stream.of("1", "2", "3");
    stream2.map(s2i->Integer.parseInt(s2i)+100).forEach(System.out::println);
~~~

- `static <T> Stream<T> concat(Stream<? extends T> a, Stream<? extends T> b)` : 通过`Stream.concat()`静态方法把两个流合并为一个流，流中元素类型不必相同   
- `Stream<T> sorted()` : 排序
- `Stream<T> sorted(Comparator<? super T> comparator)` : 传入比较器排序  

~~~java
    Stream<Integer> stream1 = Stream.of(1,2,3,4,5);
    Stream<String> stream2 = Stream.of("上山", "打", "老虎");
    Stream.concat(stream1, stream2).forEach(System.out::print);
~~~
- `T reduce(T identity, BinaryOperator<T> accumulator)` : 聚合 
- `Optional<T> reduce(BinaryOperator<T> accumulator)` 

~~~java
    System.out.println("给定个初始值，求和");
    System.out.println(Stream.of(1, 2, 3, 4).reduce(100, (sum, item) -> sum + item));
    System.out.println(Stream.of(1, 2, 3, 4).reduce(100, Integer::sum));
    System.out.println("给定个初始值，求min");
    System.out.println(Stream.of(1, 2, 3, 4).reduce(100, Integer::min));
    System.out.println("给定个初始值，求max");
    System.out.println(Stream.of(1, 2, 3, 4).reduce(100, Integer::max));

    // 注意返回值，上面的返回是T,泛型，传进去啥类型，返回就是啥类型。
    // 下面的返回的则是Optional类型
    System.out.println("无初始值，求和");
    System.out.println(Stream.of(1, 2, 3, 4).reduce(Integer::sum).orElse(0));
    System.out.println("无初始值，求max");
    System.out.println(Stream.of(1, 2, 3, 4).reduce(Integer::max).orElse(0));
    System.out.println("无初始值，求min");
    System.out.println(Stream.of(1, 2, 3, 4).reduce(Integer::min).orElse(0));

    /*
    给定个初始值，求和
    110
    110
    给定个初始值，求min
    1
    给定个初始值，求max
    100
    无初始值，求和
    10
    无初始值，求max
    4
    无初始值，求min
    1
    */
~~~

2. 终结方法：返回值类型不再是 Stream接口自身类型，一旦调用终结方法，整个模型就会按照指定策略执行操作  
- `long count()` : 统计Stream流中的元素个数  
- `void forEach(Consumer<? super T> action)` ：    
~~~java   
    stream1.forEach(s1i->System.out.print(s1i));
    //或者使用方法引用
    stream1.forEach(System.out::print);
~~~
- `R collect(Collector<? super T, A, R> collector)` : 收集   
~~~java
    //收集到List
    stream.collect(Collectors.toList())

    //收集到Set
    stream.collect(Collectors.toSet())

    //收集到数组
    stream.toArray()
~~~


# IO    

- 在Java中，I/O流分为字节流和字符流，分类如下：
![IO流分类](https://s1.ax1x.com/2020/08/01/aGpR6e.png)

- 根据操作对象主要分为：
![IO按操作对象分类](https://s1.ax1x.com/2020/08/01/aGp2lD.png)

## 1. 字节流  
字节流可以传输任意文件数据，在操作流的时候，无论使用什么样的流对象，底层传输的始终都是二进制数据   

### 1.1 字节输出流抽象类 OutputStream   
`OutputStream`是`Object`的直接子类，此抽象类是表示输出字节流的所有类的父类，输出流接受输出字节并将这些字节发送到某个接收器。  
- **成员方法 ：**    
`void close()` : 关闭此输出流，释放相关的所有系统资源      
`void flush()` ： 刷新此输出流，并强制任何缓冲的输出字节被写出          
`void write(byte[] b)` ： 将 b.length 字节从指定的字节数组写入此输出流          
`void write(byte[] b, int off, int len)` ： 从偏移量 off 开始写入 len 字节到输出流      
`abstract void write(int b)` ： 将指定的字节输出流     

### 1.2 字节输入流抽象类 InputStream      
- **成员方法 ：**     
`int available ()` : 返回可读字节数量       
`abstract int read()` ：读取数据     
`int read(byte[] b, int off, int len)` : 读取指定长度到byte数组       
`long skip(long n)` ： 跳过指定个数字节       
`boolean markSupported()` : 是否支持 mark()/reset()         
`synchronized void mark(int readlimit)` : mark一个位置      
`synchronized void reset()` ： 重置位置为上次mark的位置         
`void close()` ： 关闭流，释放资源 

### 1.3 文件字节流

#### 1.3.1 文件字节输出流类 FileOutputStream   

- **构造方法：**   
`FileOutputStream(String name)` ： 创建一个向指定名称的文件中写入数据的文件输出流     
`FileOutputStream(File file)` ： 创建一个向指定 File 对象表示的文件中写入数据的文件输出流    
`FileOutputStream(String name, boolean append)` : 可以进行追加写        
`FileOutputStream(File file, boolean append)`      

- 构造方法的作用：  
创建`FileOutputStream`对象，根据构造方法中的文件路径/文件创建一个空的文件，把`FileOutputStream`对象指向创建好的文件    

- 写入数据的原理：    
java程序 --> JVM ---> OS ---> OS调用写数据的方法 ---> 将数据写入到文件中    

- 一次写入单个字符
~~~java
    //创建FileOutputStream对象，传入文件
    FileOutputStream fos = new FileOutputStream("D:\\a.txt");
    //写入数据
    fos.write('b');
    //释放资源
    fos.close();
~~~

- 一次写入多个字符   
如果第一个字节是正数(0-127)，直接查询 ASCII 表，如果第一个字节是负数，那么前两个字节组合，查询 GBK 表   
~~~java
    //使用字符串转字节数组写入
    byte[] in1 = "大家好才是真的好".getBytes();
    fos.write(in1,0,21);   //大家好才是真的，UTF-8编码
    fos.close();
~~~

- 换行符号    
Windows: \r\n    
Linux: /n   
MacOS: /r   

#### 1.3.2 文件字节输入流 FileInputStream      
- **构造方法：**    
`FileInputStream(String name)`      
`FileInputStream(File file)`         

- 按单个字节读入：     
`int read()`返回读取到的字节的`int`值
~~~java
    FileInputStream fis = new FileInputStream("D:\\a.txt");
    int readByte = 0;
    while((readByte = fis.read())!=-1){     //java中赋值语句会返回该值，读取到结束标记时read()会返回-1
        System.out.println(readByte);
    }
    fis.close();
~~~

- 一次读取多个字节：    
此处的 bytes 起到缓冲作用，存储每次读取到的多个字节，数组的长度定义为 1024 即可一次读取 1KB   
`int read(byte[])` 返回的是每次读取到的**有效字节个数**
~~~java
    FileInputStream fis = new FileInputStream("D:\\a.txt");    //a中存放abcde
    byte[] bytes = new byte[2];
    int len = 0;
    while(len>=0) {
        len = fis.read(bytes);
        System.out.println("len="+len+",bytes="+new String(bytes, 0, len));    //String的构造方法，从byte[]数组创建String
        //使用 String(bytes, 0, len)保证只把有效读取的字符转换为字符串~~~
    }
    fis.close();
    /*
    len=2,bytes=ab
    len=2,bytes=cd
    len=1,bytes=ed       //只读取到一个e,覆盖掉c
    len=-1,bytes=ed      //读取到结束标记，返回-1
    */
~~~


## 2. 字符流   
字节流读取中文字符的时候，由于中文一个字符可能占用多个字节存储，可能不会显示完整字符，所以java提供了字符流，以字符为单位读写数据，专门处理文本文件,**底层也是字节流**   

### 2.2 字符输入流抽象类 Reader     
- **成员方法：**  
`boolean ready()`     
`int read()` ：读取单个字符，可读返回读取的整型数，遇到文件末尾返回-1     
`int read(char[] cbuf)  ` : 读取到char数组       
`abstract int read(char[] cbuf, int off, int len)`    
`void reset()`         
`long skip(long n)`               
`abstract void close()`    

### 2.1 字符输出流抽象类 Writer
- **成员方法：**   
`abstract void flush()`    
`void write(int c)`   
`void write(char[] cbuf)`   
`abstract void write(char[] cbuf, int off, int len)`      
`void write(String str)`    
`void write(String str, int off, int len)`    

### 2.3 文件字符流   
`java.io.FileReader` extends `InputStreamReader` extends `Reader`   
`java.io.FileWriter` extends `OutputStreamWriter` extends `writer`
#### 2.3.1 文件字符输入流 FileReader  
- **构造方法：**    
`FileReader(File file)`   
`FileReader(String fileName)`   
- read 读取到的是字符的编码，需要转换为 char  
#### 2.3.2 文件字符输出流 FileWriter   
- **构造方法：**   
`FileWriter(String fileName)`    
`FileWriter(String fileName, boolean append)`        
`FileWriter(File file)`      
`FileWriter(File file, boolean append)`      
字符流输出的时候，write方法先把数据**写入到内存缓冲区中**，字符转换为字节，再利用flush刷新内存缓冲区中的数据，把数据写入到文件中，释放资源也会自动把内存缓冲区中的数据刷新到文件中  

- 字节输出流即使不调用`close()`方法，也会把数据写入到文件中，但是字符传输流如果不调用`close()`或者`flush()`，数据不会写入到文件，只在内存缓冲区中，程序运行结束时便消失了   

- 与 try catch 结合使用
~~~java
    FileWriter fw = null;      //提高变量作用域，使finally中可以使用
                     // 必须给初值，否则new出现异常无法执行finally
    try {
        fw = new FileWriter("D:\\a.txt");
        fw.write(97);   //写入int的ASCII码

        char[] chs = {'a', 'b', 'c', '\r', '\n'};
        fw.write(chs);    //写入字符数组

        String s = "写入字符串了\r\n";
        fw.write(s);    //写入字符串
        fw.write(s, 0, 5);

    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        if (fw != null) {   //不为null时才close，否则会出现空指针异常
            try {           //close()的异常再次进行捕获
                fw.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
~~~

## 3. 异常的处理   

### 3.1 jdk7新特性
- 在jdk7之后，`try`之后可以加`()`，**在括号中定义流对象**，此流对象作用域只在try中有效，try中代码执行完毕自动释放流对象**不需要再写finally**    

~~~java   
    public static void main(String[] args) {
        try (   //在try中创建流对象
                FileInputStream fis = new FileInputStream("D:\\lena.bmp");
                FileOutputStream fos = new FileOutputStream("D:\\lena_copy.bmp");
        ) {   
            byte[] bytes = new byte[1024];
            int len = 0;
            while ((len = fis.read(bytes)) != -1) {
                fos.write(bytes, 0, len);
            }                                       //try中代码执行完之后自动释放流对象
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
~~~

### 3.2 jdk9新特性
- 可以在 **try之前定义流对象**，在try的括号中引入流对象的名称，try的代码执行完毕之后，流对象也可以自动释放，不用写finally释放流对象    
~~~java
    public static void main(String[] args) throws FileNotFoundException {
        FileInputStream fis = new FileInputStream("D:\\lena.bmp");
        FileOutputStream fos = new FileOutputStream("D:\\lena_copy.bmp");
        try (fis;fos) {
            byte[] bytes = new byte[1024];
            int len = 0;
            while ((len = fis.read(bytes)) != -1) {
                fos.write(bytes, 0, len);
            }
        } catch (IOException e) {
            System.out.println("出错了" + e);
            e.printStackTrace();
        }
    }

~~~
## 4. File类   
- **构造函数：**    
`File(String directoryPath)`     
`File(String directoryPath, String filename)` : 操作同目录多个文件时比较方便      
`File(File dirObj, String filename)`        

- **常用方法：**   
`String getName()`      
`String getParent()` ： 返回父路径名    
`String getPath()` ： 路径名转字符串    
`boolean isFile()` : 此路径名是表示的文件是否是标准文件     
`boolean equals()`      
`String[] list()` ： 返回目录中所有文件和文件夹名的字符串数组
`boolean mkdir()` ： 创建路径对应的目录
`String getAbsolutePath()` ： 返回绝对路径名
`boolean exist()` 

~~~java
    File file = new File("D:\\b.txt");
    file.createNewFile(); //创建对应的文件
    System.out.println(File.pathSeparator);  //路径分隔符，Windows是;  linux是:
    System.out.println(File.separator);     //路径名称分隔符，Windows是\ linux是/

    if(file.exists()){      //删除文件
        file.delete();
    }


    //也可以对文件夹操作
    String fileName = "D" + File.separator + "aabb";
    File file1 = new File(fileName);
    file.mkdir();       //创建文件夹
    file.listFiles();   //列出所有文件，包括隐藏文件
    file.isDirectory();  //判断指定路径是否是目录
~~~

## 5. Properties集合    
- `Properties extends Hashtable<k, v> implements Map<k, v>`
- Properties 类表示了一个持久的属性集，是唯一和IO流相结合的集合，可以保存在流中或者从流中加载   
- Properties 集合的 key 和 value 默认都是字符串      
`Object setProperities(String key, String value)` : 调用 Hashtable 的 put 方法      
`String getPreperities(String key)` : 通过 key 找到对应 value       
`Set<String> ProperityNames()` : 返回所有的key的集合，相当于 Map.keySet()    
- 写    
`void store(Writer writer, String comments)` : 使用字符输出流输出(中文用字符流), comments是注解(unicode)，用中文会输出unicode         
`void store(OutputStream out, String comments)` ： 使用字节输出流输出，输出中文会得到unicode码       
- 读    
`synchronized void load(Reader reader)`           
`synchronized void load(InputStream inStream)`    
读取的时候，key和value的连接符可以是等号=，空格，以及其他符号      
存储的文件中可以使用 # 对键值对进行注释，被注释的键值对不再读取      
键值都是字符串，不用再加 ""

~~~java
    Properties pp = new Properties();
    pp.setProperty("11201209120","小明");
    pp.setProperty("11201209121","小红");
    pp.setProperty("11201209122","小蓝");
    pp.setProperty("11201209123","小绿");
    Set<String> keys= pp.stringPropertyNames();   //得到keys(names)
    System.out.println(keys);

    try(FileWriter fw = new FileWriter("D:\\a.txt");){   //字符流写入
        pp.store(fw,"中文some coments...");
    } catch (IOException e) {
        e.printStackTrace();
    }

    try(FileOutputStream fos = new FileOutputStream("D:\\b.txt");){  //字节流写入
        pp.store(fos, "中文some coments...");
    } catch (IOException e) {
        e.printStackTrace();
    }

    /*结果 

    a.txt
    #\u4E2D\u6587some coments...
    #Sun Aug 02 20:25:39 CST 2020
    11201209122=小蓝
    11201209121=小红
    11201209123=小绿
    11201209120=小明


    b.txt
    #\u4E2D\u6587some coments...
    #Sun Aug 02 20:25:39 CST 2020
    11201209122=\u5C0F\u84DD
    11201209121=\u5C0F\u7EA2
    11201209123=\u5C0F\u7EFF
    11201209120=\u5C0F\u660E            

    */
~~~
接下来加载一下文件到集合：
~~~java
    pp.clear();
    try(FileReader fr = new FileReader("D:\\b.txt")){
        pp.load(fr);
    } catch (IOException e) {
        e.printStackTrace();
    }
    for(Map.Entry<Object, Object> entry : pp.entrySet()){
        System.out.println(entry.getKey()+"--->"+entry.getValue());
    }
~~~

## 6. 缓冲流(高效流)
- 上面的字节流和字符流在每次读写数据时，都要经过jvm，os，读写文件，效率低下       
- 缓冲流给基本的输入输出流增加一个缓冲区(数组)，提高基本的字符输入流的读取效率     

### 6.1 字节缓冲流    
- 字节缓冲流：              
BufferedInputStream extends InputStream            
BufferedOutputStream extends OutputStream
- **构造方法：**    
`BufferedInputStream(InputStream in)`           
`BufferedInputStream(InputStream in, int size)` : 指定缓冲区的大小           
`BufferedOutputStream(OutputStream out)`            
`BufferedOutputStream(OutputStream out, int size)`            
~~~java
    long start = System.currentTimeMillis();
    try (
            BufferedInputStream bis = new BufferedInputStream(new FileInputStream("D:\\lena.bmp"));
            BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream("D:\\lena_copy.bmp"));
    ) {
        byte[] bytes = new byte[1024];
        int len = 0;
        while((len = bis.read(bytes))!=-1){
            bos.write(bytes, 0, len);
            bos.flush();
        }
    } catch (IOException e) {
        e.printStackTrace();
    }
    long end = System.currentTimeMillis();
    System.out.println("用时"+(end-start)+"ms");   // 3ms
~~~

### 6.2 字符缓冲流  
- 字符缓冲流        
BufferedWriter extends Writer           
BufferedReader extends Reader       
- **构造方法：**  
`BufferedReader(Reader in)`           
`BufferedReader(Reader in, int size)` : 指定缓冲区的大小           
`BufferedWriter(Writer out)`            
`BufferedWriter(Writer out, int size)`    
- 其他方法：   
`String readLine()` : 读取一行，不会读取换行符，到达文件末尾时返回 `null`          
`void newLine()` : 根据不同系统，写入一个行分割符(println也是调用的newLine)   
~~~java
    String line = null;
    while((line = br.readLine())!=null){
        System.out.println(line);
    }
~~~
- 注意：第一行直接回车，也能读取到换行符，并不会返回null结束读取

## 7. 转换流   
- 之前的`FileReader`和`FileWriter`只能按UTF-8编码        
**InputStreamReader** : 字节流向字符的桥梁，是FileReader的父类，可以指定编码方式      
**OutputStreamWriter** ： 是FileWriter的父类，可以指定编码方式写入      
- **构造方法：**            
`OutputStreamWriter(OutputStream out, String charsetName)`          
`OutputStreamWriter(OutputStream out, Charset cs)`     
`InputStreamReader(InputStream in, String charsetName)`         
`InputStreamReader(InputStream in, Charset cs)`

~~~java   
    //以GBK编码格式写入和读取
    try (BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(new FileOutputStream("D:\\a.txt"), Charset.forName("GBK")))) {
        bw.write("缓冲的带格式的写入方法".toCharArray());
        bw.flush();
    } catch (IOException e) {
        e.printStackTrace();
    }

    try (BufferedReader br = new BufferedReader(new InputStreamReader(new FileInputStream("D:\\a.txt"), Charset.forName("GBK")))) {
        String line = br.readLine();
        System.out.println(line);
    } catch (IOException e) {
        e.printStackTrace();
    }
~~~

## 8. 对象序列化          
### 8.1 序列化操作      
`ObjectOutputStream` extends `OutputStream`              
`ObjectInputStream` extends `InputStream`           
- 写对象，可以把对象转换为字节序列，写入到文件(序列化)，或者把文件中的对象读出来(反序列化)            
- 类必须实现 `Serializable` 接口，才能被序列化            
- **构造方法：**    
`ObjectOutputStream(OutputStream out)`      
`ObjectInputStream(InputStream in)`         

~~~java
    //序列化对象
    try (ObjectOutputStream oos = new ObjectOutputStream(
        new FileOutputStream(new File("D:" + File.separator + "a.txt")));
    ) {
        Student s1 = new Student("小明", 12, "男");     //Student必须实现Serializable接口
        oos.writeObject(s1);
    } catch (IOException e) {
        e.printStackTrace();
    }

    //反序列化  
    try(ObjectInputStream ois = new ObjectInputStream(new FileInputStream(new File("D:" + File.separator + "a.txt")))){
        Object s1 = ois.readObject();
        System.out.println(((Student)s1).getName());
        System.out.println(s1);
    } catch (ClassNotFoundException e) {
        e.printStackTrace();
    }   
~~~

### 8.2 瞬态关键字 transient
- 只要实现了`Serilizable`接口，对象就可以被序列化，但是有时候为了安全起见，类的一些敏感属性不希望被序列化后传输，这事就需要瞬态关键字                      
- 在对象序列化的时候，`transient`修饰的成员变量不能被序列化，**生命周期仅存在于调用者的内存中**，不会写到磁盘里持久化       
- `transient` 只能修饰变量，不能修饰方法和类
- `stastic`变量属于类，不管是否被`transient`修饰，都不能被序列化    
- 需要用两个进程分别测试序列化和反序列化，否则jvm已经把类加载进来了，可以读取到类的static变量信息   
测试：
~~~Java
    public class Student implements Serializable {
        private String name;
        private static int age;
        private transient String sex;
        constructor...getter and setter....
    }

    //序列化
    public class DemoObjectOutputStream {
        public static void main(String[] args) {
            try (ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(
                new File("D:" + File.separator + "a.txt")));
            ) {
                Student s1 = new Student("小明", 12, "男");
                oos.writeObject(s1);

            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

    //反序列化
    public class DemoObjectInputStream {
        public static void main(String[] args) {
            try(ObjectInputStream ois = new ObjectInputStream(new FileInputStream(
                new File("D:" + File.separator + "a.txt")))){
                Object s1 = ois.readObject();
                System.out.println(s1);
            } catch (ClassNotFoundException | IOException e) {
                e.printStackTrace();
            }
        }
    }

    // 结果：
    // Student{name='小明', age=0, sex='null'}

~~~

### 8.3 InvalidClassException           
- 当我们反序列化时，如果对应的 class 和序列化时相比已经发生了变化，就会出现`InvalidClassException`，class文件和序列化文件的`SerialVersionUID`不匹配
- 如果我们想修改class之后不让`SerialVersionUID`不发生变化，可以在类中通过显示声明`Static final long SerialVersionUID = ...` 定义自己的`SerialVersionUID`  
- IDEA也可以自动生成`serialVersionUID`，settings -> Editor -> Inspections -> Serialization issues -> Serialization class without 'serialVersionUID' 打上勾即可

### 8.4 writeObject()与readObject()    

- 对于被transient修饰的字段，除了将transitive关键字去掉之外，还可以在类中添加两个方法：writeObject()与readObject()，如下所示：
~~~java
//学生类，添加writeObject()和readObject()方法
public class Student implements Serializable {
    private static final long serialVersionUID = 1686151893029100782L;
    private String name;
    private static int age;
    private transient String sex;

    Constructor...

    private void writeObject(ObjectOutputStream oos) throws IOException {   //添加writeObject方法
        oos.defaultWriteObject();
        oos.writeChars(sex);
    }
    private void readObject(ObjectInputStream ois) throws IOException, ClassNotFoundException { //添加readObject方法
        ois.defaultReadObject();
        sex = String.valueOf(ois.readChar());
    }

    getter and setter...
}

public class DemoObjectOutputStream {
    public static void main(String[] args) throws IOException {
        try (ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(new File("D:" + File.separator + "a.txt")));
        ) {
            Student s1 = new Student("小明", 12, "男");
            oos.writeObject(s1);

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

public class DemoObjectInputStream {
    public static void main(String[] args) {
        try(ObjectInputStream ois = new ObjectInputStream(new FileInputStream(new File("D:" + File.separator + "a.txt")))){
            Object s1 = ois.readObject();
            System.out.println(s1);
        } catch (ClassNotFoundException | IOException e) {
            e.printStackTrace();
        }
    }
}

//结果
//Student{name='小明', age=12, sex='男'}
//成功的写入了。。。。
~~~

​      

## 9. 一点小问题
可以看到，java字节流和字符流的输入，都是在遇到文件末尾的时候返回-1，那如果我们直接往文件中写入-1的话，会发生什么情况呢？会不会读取不到之后的信息呢？   

~~~java
    FileOutputStream fos = new FileOutputStream("D:\\a.txt");
    fos.write(-1);
    fos.write(97);
    fos.write((97-256));
    fos.close();

    FileInputStream fis = new FileInputStream("D:\\a.txt");
    int len = 0;
    byte[] chs = new byte[10];
    len = fis.read(chs);
    System.out.println(Arrays.toString(chs));
    System.out.println(new String(chs, 0, len));
    fis.close();

    FileInputStream fis2 = new FileInputStream("D:\\a.txt");
    int value = 0;
    while((value=fis2.read())!=-1){
        System.out.print(value+"  ");
    }
    //最后得到的结果是
    //[-1, 97, 97, 0, 0, 0, 0, 0, 0, 0]
    //�aa
    //255  97  97  
~~~
可以看到，当我们以byte数组接收的时候，接收到的仍然是-1，以int接受的时候返回的是255，这是怎么回事呢？

原来，在java中，`byte`以一个字节表示的是有符号整数，也就是[-128,127]      
当我们传入-1的时候(1000 0001)b，存储的是-1的补码(1111 1111)b    
在`int`形式输出的时候，`int`是32位，自然就是255了，而`read()`方法返回值就是`int`啊，所以写入-1，实际上读取到的是`int`的255，所以不会停止读入    
而在用`byte[]`接收的时候又进行了`int`转`byte`，`byte`接收到(11111111)b，自然就输出-1了


我们如果按下面的方式读取，就不会读到任何东西了
~~~java
    FileInputStream fis2 = new FileInputStream("D:\\a.txt");
    byte value = 0;
    while((value=(byte)fis2.read())!=-1){
        System.out.print(value+"  ");
    }
~~~


# 多线程

## 1. 基本概念          
### 1.1 并发和并行
- 并发：同一时间段发生        
- 并行：同一时刻发生             
### 1.2 进程和线程      
1. 进程：内存中运行的应用程序，每个进程都有独立的内存空间，一个应用程序可以同时运行多个进程，进城是系统运行程序的基本单位，是程序的一次执行过程(进程的创建、运行、销毁)         
- 是资源分配的单位      
- 开销：每个进程由独立的代码和数据空间(进程上下文)，进程之间的切换由较大的开销  
- 所处环境：在操作系统中能同时运行多个任务(程序)        
- 分配内存：系统在运行时为每个进程分配不同的内存区域    
- 包含关系：没有线程的进程可以看作单线程，进程中的多个线程是同时执行的            

2. 线程：进程中的一个执行单元，负责当前进程中程序的执行。
- 是资源调度的基本单位和程序执行的基本单元        
- 开销：同一类线程共享代码和数据空间，每个线程有独立的运行栈和程序计数器，线程切换开销小        
- 所处环境：在同一应用程序中多个顺序流同时执行      
- 分配内存：线程使用所属进程的资源，线程组只能共享资源
- 包含关系：线程是进程的一部分              

3. 进程实现多处理机环境下的进程调度，分派，切换时， 都需要花费较⼤的时间和空间开销      
引⼊线程主要是为了提⾼系统的执⾏效率，减少处理机的空转时间和调度切换的时间，以及便于系统管
理。 使OS具有更好的并发性

## 1.3 线程的调度       
- 分时调度：所有线程轮流使用CPU，平均分配占用CPU的时间          
- 抢占式调度：按优先级调度，优先级高的线程先使用CPU，同优先级随机选择，Java使用抢占式调度       

## 1.4 主线程   
- 执行main方法的线程    

## 2. 线程的创建        

### 2.1 继承Thread类
1. 自定义一个类，继承`Thread`类，重写`run`方法      
2. 创建该类的对象，通过对象调用 `start` 方法执行线程    
- 结果是当前线程(main)和创建的新线程并发运行，多次启动同一线程是非法的，尤其是线程结束后，不能再启动        
- 调用`start`和`run`方法的区别：`run`方法在主线程的栈内存中直接调用，`start`方法会开辟新的栈空间由JVM来调用执行`run`方法，各个栈空间互不影响         
- 缺点：单继承，继承了Thread不能再继承其他类了

#### Thread类的常用方法
**构造方法：**          
`public Thread()` : 分配一个新的线程对象            
`public Thread(String name)` ： 分配一个指定名字的新线程对象            
`public Thread(Runnable target)` ： 带有指定目标的新线程           
`public Thread(Runnable target, String name)`           
`public Thread(ThreadGroup group, String name)`

**常用方法：**          
`public final String getName()`         
`public void start()`           
`public void run()`         
`public static void sleep(long millis)`             
`public static Thread currentThread()`          

### 2.2 实现Runnable接口
1. 自定义一个类，实现`Runnabel`接口的`run`方法
2. 传入该`Runnable`创建`Thread`对象，调用`start`方法(Runnable本身没有start方法)         
- 好处：实现Runnable还可以继承其他类，增强了程序扩展性，降低耦合性(设置线程任务和开启线程分离)，**将并发运⾏任务和运⾏机制解耦**        

~~~java
//定义一个实现Runnable接口的类，设置线程的任务
public class RunnableImpl implements Runnable{
    @Override
    public void run() {
        String name = Thread.currentThread().getName();
        for (int i = 0; i < 10; i++) {
            System.out.println(name+",执行第"+(i+1)+"次");
        }

    }
}

//将该类的对象用来构造Thread
public class DemoCreateThread {
    public static void main(String[] args) {
        Runnable runnable = new RunnableImpl();
        Thread t1 = new Thread(runnable,"线程1");
        Thread t2 = new Thread(runnable,"线程2");
        t2.start();
        t1.start();
    }
}
~~~

- 也可以直接传入匿名内部类或者Lambda表达式的方式(只能执行一次了)

~~~java
    public static void main(String[] args) {
        Thread t1 = new Thread(()-> {
            String name = Thread.currentThread().getName();
            for (int i = 0; i < 10; i++) {
                System.out.println(name + ",执行第"+(i+1)+"次");
            }
        }, "线程1");
        t1.start();
    }
~~~

### 2.3 实现Callable接口
- Runnable没有返回值，不能抛出受检查的异常，而Callable可以
- Callable的call方法返回值是Future接口类型，通过get方法获得返回值
- get方法具有阻塞性，提交任务之后主线程本来是继续运行了，运行到 future.get() 时便则阻塞住了，等待返回值。
- 详细使用见线程池部分

**run和start的区别：**
- `run()` : 仅仅是封装被线程执行的代码，直接调⽤是普通⽅法      
- `start()` : ⾸先启动了线程，然后再由jvm去调⽤该线程的run()⽅法      


## 3. 线程安全   
- 解决线程安全的几种方法：同步代码块，同步方法，Lock锁
### 3.1 同步代码块      
**同步代码块：**
- `synchronized` 关键字可以对用于某个方法中的某个区块，表示对这个区块资源实行互斥访问         
- 获取的是指定对象的锁
~~~java
    synchronized(Object 锁对象){
        需要同步操作的代码
    }
~~~
**同步锁：**            
- 锁对象可以是任意类型      
- 多个线程对象要使用同一把锁        
- 未获得锁的线程进入 BLOCKED 状态等待

### 3.2 同步方法  
- 使用 synchronized 修饰的方法就是同步方法，同一时刻只能有一个线程进入该方法       
- 获取的是当前对象的锁       
~~~java
    public synchronized void method(){
        可能产生线程安全问题的代码
    }
~~~
**同步方法中的同步锁：**
- 对于非static方法，同步锁就是实现类对象，也就是 this
- 对于static方法，使用的是当前方法所在类的Class对象(本类的class属性)            
- 获取了类锁和获取了对象锁的线程是不冲突的      

**锁的释放时机：**
- 当方法执行完毕后会自动释放锁，不需要任何的操作
- 当一个线程的代码出现异常时，其持有的锁会自动释放
- 不会由于异常出现导致死锁现象
~~~java
public class DemoSynchronized {
    public synchronized void method() throws InterruptedException {//非静态方法
        for (int i = 0; i < 3; i++) {
            Thread.sleep(1000);
            System.out.println("method running");
        }
    }

    public static synchronized void staticMethod() throws InterruptedException { //静态方法
        for (int i = 0; i < 3; i++) {
            Thread.sleep(1000);
            System.out.println("static method running");
        }
    }

    public static void main(String[] args) {
        new Thread(()-> {
            try {
                staticMethod();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();

        new Thread(()->{
            try {
                new DemoSynchronized().method();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();
    }
}
/* 结果互不影响
method running
static method running
method running
static method running
method running
static method running
*/
~~~


### 3.3 Lock显式锁 
- java.util.locks.lock 接口, Lock锁提供了比 synchronized 代码块和 synchronized 方法更广泛的锁定操作，除了具备前两者的功能外，还有更强大的功能，更能体现面向对象的思想     
- Lock锁更灵活，但必须手动释放锁。synchronized加锁更方便，出错少

- Lock锁的加锁和释放锁进行方法化了：        
`public void lock()`        
`public void unlock()`      

- jdk1.5带来了Lock锁，性能比synchronized好，但从jdk1.6开始synchronized就进行了各种优化(适应自旋锁，消除锁，锁粗化，轻量级锁，偏向锁)。所以现在二者差别不大，大多数时候用synchronized锁就好了。

#### 3.3.1 AQS同步器
- juc : java.util.concurrent， 并发包           
- AQS：AbstractQueuedSynchronizer 类，常见的两个Lock锁都是基于AQS实现的           
- AQS是一个可以实现锁的框架，内部实现依靠队列和state状态        
- AQS是 ReentrantReadWriteLock 和 ReentrantLock 的基础，这两种 Lock 锁默认的实现都是在内部类Syn中，
⽽Syn是继承AQS的        

#### 3.3.2 重入锁Reentrantlock     
`java.util.concurrent.locks.ReentrantLock` implements `Lock`        
- 加锁和释放锁都在方法里进行，可以自由控制，比synchronized更灵活方便，还可以多重加锁
- 释放锁必须在finally里，否则可能导致锁不能正常被释放，卡死后续访问该锁的线程    
- ReentrantLock锁上使⽤的是state来表示同步状态(也可以表示重⼊的次数)       
- 构造方法：
~~~java
    public ReentrantLock() {    //默认使用非公平锁
        sync = new NonfairSync();
    }
~~~

- 基本使用方法
~~~java
    class X {
    private final ReentrantLock lock = new ReentrantLock();
    // ...
    public void m() {
        lock.lock();  // block until condition holds
        try {
        // ... method body
        } finally {
        lock.unlock()
        }
    }
    }
~~~

**公平锁：**
- 线程按照请求发出顺序来获取锁，(非公平锁就可以插队获取锁)。 
- Lock和synchronized默认都使用非公平锁，必要时才使用公平锁(额外的性能消耗)      

非公平锁比公平锁的性能更好，假设线程一是队列中的第一个，线程二是想要插队的线程，当占用锁的线程释放了锁时JVM有两种选择：
阻塞线程二，启动线程一，或者线程一的状态维持不变继续阻塞，线程二的状态也维持不变继续运行。线程状态的切换是耗费时间的，因此方案二的性能更好

**synchronized也是重入锁：**          
如下所示，两个方法都是synchronized修饰，add方法可以成功获取到当前线程operation方法已经获取到的锁        

~~~java
    public synchronized void operation(){
        add();
    }
    public synchronized void add(){

    }
~~~

#### 3.3.3 ReentrantReadWriteLock   
synchronized锁和ReentrantLock都是互斥锁(一次只能有一个线程进入被锁定区域)       
ReentrantReadWriteLock是读写锁，实现了ReadWriteLock接口 
- 读取数据的时候，可以多个线程同时进入到临界区(被锁定区域)      
- 写如数据的时候，无论读线程还是写线程都是互斥的
- 读锁不能升级为写锁，写锁可以降级为读锁    
`acquire(1)` : 获取写锁             
`acquireShared(int arg)` ： 获取读锁

### 3.4 原子性和易变性      

比如要在多线程任务里实现一个变量的自增，按照上面的方法，我们加上synchronized锁即可，但是对这种简单的操作用没必要用锁，这会使得别的线程在执行的时候当前线程只能等待
- 原子性是指某个操作，要么一次执行完，要么就不执行            
~~~java
public class DemoAtomic {
    public static void main(String[] args) throws InterruptedException {
        ExecutorService service = Executors.newCachedThreadPool();
        Count count = new Count();
        // 100个线程对共享变量进行加1
        for (int i = 0; i < 100; i++) {
            service.execute(() -> count.increase());
        }
        // 等待上述的线程执行完
        service.shutdown();
        service.awaitTermination(1, TimeUnit.DAYS);
        System.out.println(count.getCount());
    }

    static class Count {
        // 共享变量
        private AtomicInteger count = new AtomicInteger(0);

        public AtomicInteger getCount() {
            return count;
        }

        public synchronized void increase() {
            count.incrementAndGet();    //返回新值
            //count.getAndIncrement();  返回旧值
        }
    }
}
~~~
相比锁机制，会有不错的性能提升      

#### 3.4.1 CAS(compare and swap)
CAS即比较并交换，是实现并发算法时常用到的技术。CAS是一种原子操作，用于在多线程编程中实现不被打断的数据交换，从而避免多线程同时改写某一数据时由于执行顺序不确定性以及中断的不可预知性产生的数据不一致问题    
- Java并发包种的很多类都使用了CAS技术  
- CAS有3个操作数：内存值V，旧的预期值A，要修改的新值B。如果V与A相等，则将内存值改为B，否则重试或者等待      
- CAS 在 Unsafe 中调用了 Atomic::cmpxchg 方法，cmpxchg是汇编指令，作用是比较并交换操作数(Compare and Exchange)          

**CAS的缺点：**
CAS虽然很高效的解决了原子操作的问题，但是仍然存在三个问题：
- **循环时间长开销很大：** 如果CAS失败会循环进行CAS操作(循环的同时将期望更新为最新的)，长时间不成功的话会给CPU带来极大的开销(这种循环也成为**自旋**)，解决办法是限制自旋的次数，防止进入死循环      
- **只能保证一个共享变量的原子操作：** 对多个变量操作时，CAS无法保证操作的原子性，这时候可以用加锁的方式保证原子性，或者把多个共享变量合并成一个共享变量进行CAS操作     
- **ABA问题：**       

ABA问题：如果内存地址V初次读取的值是A，并且在准备赋值的时候检查到它的值仍然为A，那我们就能说它的值没有被其他线程改变过了吗？如果在这段期间它的值曾经被改成了B，后来又被改回为A，那CAS操作就会误认为它从来没有被改变过。这个漏洞称为CAS操作的“ABA”问题。Java并发包为了解决这个问题，提供了一个带有标记的原子引用类“AtomicStampedReference”，它可以通过控制变量值的版本来保证CAS的正确性。因此，在使用CAS前要考虑清楚“ABA”问题是否会影响程序并发的正确性，如果需要解决ABA问题，改用传统的互斥同步可能会比原子类更高效

ABA问题举例：线程1准备用CAS将变量的值由A替换为B，在此之前，线程2将变量的值由A替换为C，又由C替换为A，然后线程1执行CAS时发现变量的值仍然为A，所以CAS成功。但实际上这时的现场已经和最初不同了，尽管CAS成功，但可能存在潜藏的问题       
现有一个用单向链表实现的堆栈，栈顶为A，这时线程T1已经知道A.next为B，然后希望用CAS将栈顶替换为B      
在T1执行上面这条指令之前，线程T2介入，将A、B出栈，再pushD、C、A，而对象B此时处于游离状态        
此时轮到线程T1执行CAS操作，检测发现栈顶仍为A，所以CAS成功，栈顶变为B，但实际上B.next为null          
其中堆栈中只有B一个元素，C和D组成的链表不再存在于堆栈中，平白无故就把C、D丢掉了 


**CAS的应用：**         
CAS操作并不会锁住共享变量，也就是一种**非阻塞**的同步机制，CAS就是乐观锁的实现      
- 乐观锁：乐观锁总是假设最好的情况，每次去操作数据都认为不会被别的线程修改数据，所以在每次操作数据的时候都不会给数据加锁，即在线程对数据进行操作的时候，别的线程不会阻塞仍然可以对数据进行操作，只有在需要更新数据的时候才会去判断数据是否被别的线程修改过，如果数据被修改过则会拒绝操作并且返回错误信息给用户

- 悲观锁：悲观锁总是假设最坏的情况，每次去操作数据时候都认为会被的线程修改数据，所以在每次操作数据的时候都会给数据加锁，让别的线程无法操作这个数据，别的线程会一直阻塞直到获取到这个数据的锁。这样的话就会影响效率，比如当有个线程发生一个很耗时的操作的时候，别的线程只是想获取这个数据的值而已都要等待很久


#### 3.4.2 原子类           
Java中对变量的读取和赋值都是原子操作，但long、double类型除外，只有使用volatile修饰之后long、double类型的读取和赋值操作才具有原子性。除此之外Java还提供了几个常用的原子类，原子类的方法是具有原子性的方法，可以保证在执行操作的过程中不会被打断  

java.util.concurrent.atomic
- 基本类型：
    - AtomicBoolean：布尔型
    - AtomicInteger：整型
    - AtomicLong：长整型

- 数组：
    - AtomicIntegerArray：数组里的整型
    - AtomicLongArray：数组里的长整型
    - AtomicReferenceArray：数组里的引用类型

- 引用类型：
    - AtomicReference：引用类型
    - AtomicStampedReference：带有版本号的引用类型
    - AtomicMarkableReference：带有标记位的引用类型

- 对象的属性：
    - AtomicIntegerFieldUpdater：对象的属性是整型
    - AtomicLongFieldUpdater：对象的属性是长整型
    - AtomicReferenceFieldUpdater：对象的属性是引用类型

- JDK8新增DoubleAccumulator、LongAccumulator、DoubleAdder、LongAdder
    - 是对AtomicLong等类的改进。比如LongAccumulator与LongAdder在高并发环境下比AtomicLong更高效。

从原理上概述就是：Atomic包的类的实现绝大调用Unsafe的方法，而Unsafe底层实际上是调用C代码，C代码调用汇编，最后生成出一条CPU指令完成操作。这也就为啥CAS是原子性的，因为它是一条CPU指令，不会被打断     
~~~java
    // 共享变量(使用AtomicInteger来替代Synchronized锁)
    private AtomicInteger count = new AtomicInteger(0);
    
    public Integer getCount() {
        return count.get();
    }
    public void increase() {
        count.incrementAndGet();
    }
~~~
#### 3.4.3 易变性   
java的volatile关键字用于通知虚拟机这个变量具有易变性，易变性具有两层含义，一个是可见性，一个是有序性        


#### 3.4.4 可见性
如果一个线程对共享变量值的修改，能够及时的被其他线程看到，叫做共享变量的可见性。如果一个变量同时在多个线程的工作内存中存在副本，那么这个变量就叫共享变量            

首先来看Java内存模型：
- 每个Thread有一个属于自己的工作内存
- 所有Thread共用一个主内存
- 线程对共享变量的所有操作必须在工作内存中进行，不能直接操作主内存
- 不同线程间不能访问彼此的工作内存中的变量，线程间变量值的传递都必须经过主内存          

如果一个线程1对共享变量x的修改对线程2可见的话，需要经过下列步骤：       
a.线程1将更改x后的值更新到主内存        
b.主内存将更新后的x的值更新到线程2的工作内存中x的副本       

**1. synchronized实现可见性：**      
- 某一个线程进入synchronized代码块前后，执行过程入如下：          
a.线程获得互斥锁        
b.清空工作内存      
c.从主内存拷贝共享变量最新的值到工作内存成为副本        
d.执行代码      
e.将修改后的副本的值刷新回主内存中      
f.线程释放锁        

随后，其他代码在进入synchronized代码块的时候，所读取到的工作内存上共享变量的值都是上一个线程修改后的最新值。

**2. volatile实现可见性：**    
volatile变量每次被线程访问时，都强迫线程从主内存中重读该变量的最新值，而当该变量发生修改变化时，也会强迫线程将最新的值刷新回主内存中。这样一来，不同的线程都能及时的看到该变量的最新值          

volatile变量不会被缓存在寄存器或其他对处理器不可见的地方，因此在读取volatile类型的变量时总会返回最新写入的值

- volatile保证变量对所有线程的可见性，但不能保证原子性：        
比如number++，这个操作实际上是三个操作的集合（读取number，number加1，将新的值写回number），volatile只能保证每一步的操作对所有线程是可见的，但是假如两个线程都需要执行number++，那么这一共6个操作集合，之间是可能会交叉执行的，那么最后导致number 的结果可能会不是所期望的
- 所以对于number++这种非原子性操作，推荐用synchronized      

一般来说，volatile大多用于标志位上(判断操作),满足下面的条件才应该使用volatile修饰变量：
- 修改变量时不依赖变量的当前值(因为volatile是不保证原子性的)
- 该变量不会纳入到不变性条件中(该变量是可变的)
- 在访问变量的时候不需要加锁(加锁就没必要使用volatile这种轻量级同步机制了)      

通常volatile用做保存某个状态的boolean值       


这里记录一下我遇到的一个坑      
首先看没有volatile修饰的情况            

示例一：
~~~java
public class DemoVolatile {
    static class Task implements Runnable {
        public static long value;

        public void run() {
            while (DemoVolatile.flag) {
                value++;
            }
            System.out.println(num);
        }
    }

    public static boolean flag = true;
    public static int num;
    public static void main(String[] args) throws InterruptedException {
        new Thread(new Task()).start();
        Thread.sleep(1000);
        flag = false;
        //Thread.sleep(1000);
        num = 120;
        System.out.println("flag: " + flag);
        System.out.println("value: " + Task.value);
        Thread.sleep(1500);
        System.out.println("value: " + Task.value);
    }
}
/*
run: false
value: 831286841
value: 2064104499
*/
~~~
显然我们会想到，没有volatile修饰，主线程中 flag 的改变对子线程不可见，所以子线程会陷入死循环
结果也是如此    
~~~java
run: false
value: 831286841
value: 2064104499
~~~

接下来我们把flag修改为volatile修饰，我们期待子线程可以获得修改之后的flag，并跳出循环        

示例二：
~~~java
public class DemoVolatile {
    static class Task implements Runnable {
        public static long value;

        public void run() {
            while (DemoVolatile.flag) {
                value++;
            }
            System.out.println(num);
        }
    }

    public static volatile boolean flag = true;
    public static int num;
    public static void main(String[] args) throws InterruptedException {
        new Thread(new Task()).start();
        Thread.sleep(1000);
        flag = false;
        //Thread.sleep(1000);
        num = 120;
        System.out.println("flag: " + flag);
        System.out.println("value: " + Task.value);
        Thread.sleep(1500);
        System.out.println("value: " + Task.value);
    }
}

/*
120
flag: false
value: 839587639
value: 839587639
*/
~~~
没错，也按我们的预想输出了          

接下来，我们把flag的volatile去掉，转而把 value加上volatile，我们预想的是，value对主进程即时可见，而主进程修改flag对子进程依然不可见     
示例三：
~~~java
public class DemoVolatile {
    static class Task implements Runnable {
        public static volatile long value;

        public void run() {
            while (DemoVolatile.flag) {
                value++;
            }
            System.out.println(num);
        }
    }

    public static boolean flag = true;
    public static int num;
    public static void main(String[] args) throws InterruptedException {
        new Thread(new Task()).start();
        Thread.sleep(1000);
        flag = false;
        //Thread.sleep(1000);
        num = 120;
        System.out.println("flag: " + flag);
        System.out.println("value: " + Task.value);
        Thread.sleep(1500);
        System.out.println("value: " + Task.value);
    }
}


/*
120
flag: false
value: 172291888
value: 172291888

*/
~~~
怎么回事？ 子进程居然也获取到了主进程修改的 flag            

其实转而想想前面的 num， 为什么示例二、三的子进程都可以获得修改后的num呢？      
这个问题在JAVA并发编程实践一书中有提到：
> **volatile变量对可见性的影响所产生的价值远远高于变量本身，线程A向volatile变量写入值之后，线程B读取该变量，所有A执行写操作前可见的变量的值，在B读取了volatile变量后，成为了对B也是可见的。所以从内存可见性的角度看，写入volatile变量就像退出同步块，读取volatile变量就像进入同步块。**             

所以其实在一个线程访问volatile变量的时候，去主内存更新的数据并不只有 volatile修饰的变量！！！子线程在修改value之后，把value值同步给主线程，这时候也把主线程中的num给读了回去       

接下来，我们让主线程中的flag改变后，主线程睡一下，就会发现其实num的值在子线程里还是0            

示例四：
~~~java
public class DemoVolatile {
    static class Task implements Runnable {
        public static volatile long value;

        public void run() {
            while (DemoVolatile.flag) {
                value++;
            }
            System.out.println(num);
        }
    }

    public static boolean flag = true;
    public static int num;
    public static void main(String[] args) throws InterruptedException {
        new Thread(new Task()).start();
        Thread.sleep(1000);
        flag = false;
        Thread.sleep(1);
        num = 120;
        System.out.println("flag: " + flag);
        System.out.println("value: " + Task.value);
        Thread.sleep(1500);
        System.out.println("value: " + Task.value);
    }
}
/*
0
flag: false
value: 172842201
value: 172842201
*/
~~~

如果没有volatile修饰，就一定不可见吗？接下来再考虑下面的例子          
示例五：        
~~~java
public class DemoVolatile {
    static class Task implements Runnable {
        public static long value;

        public void run() {
            while (DemoVolatile.flag) {
                value++;
            }
            System.out.println(num);
        }
    }

    public static boolean flag = true;
    public static int num;
    public static void main(String[] args) throws InterruptedException {
        new Thread(new Task()).start();
        Thread.sleep(1);
        flag = false;
        num = 120;
        System.out.println("flag: " + flag);
        System.out.println("value: " + Task.value);
        Thread.sleep(1500);
        System.out.println("value: " + Task.value);
    }
}
~~~

如果没有Thread.sleep(1)，我们很明显想到flag的值在子线程读取的时候就修改为了false，不会进入while循环，输出的value值明显是0，那现在的情况的输出是什么样的呢？我们来看看           
~~~java
120
flag: false
value: 59771
value: 59771
~~~
这又是怎么回事？进入了while循环，并且成功的循环一段时间后出来了！！！
经过实验后发现当主线程停顿时间很极短（1～2ms）时，可以跳出循环，当主线程停顿时间较长时，无法跳出循环，尝试修改子线程循环一次的时间，发现**子线程循环次数少的时候可以跳出循环，循环次数多的时候无法跳出循环**，代码的执行结果居然跟循环次数有关？            

这与JIT即时编译优化有关。           
>关于JIT
当虚拟机发现某个方法或代码块运行特别频繁时，就会把这些代码认定为“Hot Spot Code”（热点代码），为了提高热点代码的执行效率，在运行时，虚拟机将会把这些代码编译成与本地平台相关的机器码，并进行各层次的优化，完成这项任务的正是 JIT 编译器。
运行过程中会被即时编译器编译的“热点代码”有两类：
1）被多次调用的方法。
2）被多次调用的循环体。

JIT优化后，只在进入循环之前读取了flag变量的值，后面无论flag怎么变化都不管了，JAVA内存模型中不能保证没有线程安全的字段将会看到更新，这个规定允许JIT进行这样的优化     

#### 3.4.5 有序性   
JMM允许编译器和处理器对指令重排序的，使用volatile关键词，可以禁止重排序，可以确保程序的“有序性”。

## 4. 线程池        
- 线程池为任务分配空闲的线程，任务执行完成后回到线程池等待下次任务(而不是销毁)，这样就实现了线程的重用      
- 线程池的好处：不必为每个请求都开一个新的线程，因为线程生命周期的开销非常高，创建和销毁线程所花费的时间和资源可能比处理任务花费更多，还会导致某些空闲线程占用资源，程序的稳定性和健壮性会下降，每个请求开一个线程，受到恶意攻击或过多请求容易内存不足，程序崩溃。      
### 4.1 Executor框架        
- `Executor` 提供了一种将**任务提交**和**任务执行**分离的机制(解耦)        
- `ExecutorService` 接口继承自Executor，提供了线程池生命周期管理的办法     
- `ThreadPoolExecutor` 类，是用的最多的线程池类            
- `ScheduledThreadPoolEcecutor` 额外提供了延迟和周期执行功能        
- `ForkJoinPool` jdk1.7新增，采用工作窃取算法，所有线程会 找到并执行已被提交到线程池或被其他线程创建的任务，这样很少有线程处于空闲状态，非常高效        

~~~java
public class MyCallable implements Callable<String> {
    private int number;
    public MyCallable(int number) {
        this.number = number;
    }

    @Override
    public String call() throws Exception {
        int sum = 0;
        for (int i = 0; i < number; i++) {
            sum+=i;
        }
        return String.valueOf(sum)+"\t"+Thread.currentThread().getName();
    }
}

public class DemoCallable {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        ExecutorService pool = Executors.newFixedThreadPool(2);

        Future<String> submit1 = pool.submit(new MyCallable(200));
        Future<String> submit2 = pool.submit(new MyCallable(100));
        //打印返回值
        System.out.println(submit1.get());
        System.out.println(submit2.get());
        //结束
        pool.shutdown();
    }
}
~~~

### 4.2 ThreadPoolExecutor
- `corePoolSize` : 运行线程数小于corePoolSize时，创建新线程来处理请求，即使其他线程是空闲的
- `maximumPoolSize` : 运行线程数介于corePoolSize和maximumPoolSize之间时，则仅当队列满时才创建新的线程，最大线程数小于核心线程数会直接抛出异常         
- `keepAliveTime` ： 线程数大于核心线程的时候，如果线程空闲时间大于该值，就会销毁，可以使用方法修改该值       
- 运行的线程少于核心线程数时，新任务会直接开线程，多于核心线程数时，则放入阻塞队列中，如果队列满了，会先开一个线程处理该任务，除非超过最大线程数，这样的话就直接拒绝该请求      
- **拒绝任务的情况**：线程池关闭 or 线程数量满了且队列饱和了            
- 四种拒绝任务的策略：直接抛出异常(默认)、在调用者所在的线程来执行任务、直接丢弃掉该任务、丢失最旧的一个任务        

**线程的状态：**
- RUNNING：可以接受新任务，可以对新添加的任务进行处理      
- SHOUDOWN：不可以接受新任务，可以对已添加任务进行处理，不再添加新任务
- STOP：不接受新任务，不处理已添加的任务，中断正在处理的任务        
- TIDYING：当所有任务已终止，ctl记录的任务数为0，线程变为TIDYING状态，执行钩子函数terminated()，是空的，用户想在TIDYING状态处理时，可以重载terminated()函数来实现       
- TERMINATED：线程池彻底终止的状态          
<center>            

|状态|COUNT_BITS高三位|工作队列workers中的任务|阻塞队列workQueue中的任务|未添加的任务|
|---|:----:|:----:|:----:|:-----:|
|RUNNING|111|继续处理|继续处理|添加|
|SHUTDOWN|000|继续处理|继续处理|不添加|
|STOP|001|尝试中断|不处理|不添加|
|TIDYING|010|处理完成|SHUTDOWN->TIDYING是处理完了，STOP->TIDYING是不处理|不添加|
|TERMINATED|011|处理完成|同上|不添加|

![状态转换](https://s1.ax1x.com/2020/08/05/ar0Nm8.png)

</center>

#### 4.2.1 常见的实现池   
**newFixedThreadPool**      
- 固定线程数的线程池，返回一个corePoolSize 和 maximumPoolSize 相等的线程池      
- 当需要运行的线程数量大体上变化不大时，适合使用这种线程池,一次性支付高昂的创建线程的开销，之后再使用的时候就不再需要这种开销       
~~~java
   public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
~~~
**newCachedThreadPool**     
- 非常有弹性的线程池，对于新任务，如果线程池没有空闲线程，线程池会毫不犹豫的创建新线程去处理，如果一个线程60秒之内没有被使用过，这个线程就会被销毁，通用性强，遇事不决，用缓存线程池        
~~~java
    public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
~~~

**SingleThreadExecutor**
- 使用单个worker线程的Executor
~~~java
    public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }
~~~
#### 4.2.2 线程池的使用         
- 对于 `Runnable` 任务，使用 `execute()` 方法执行
- 对于 `Callable` 任务，使用 `submit()` 方法执行(submit也重载了接收Runnable的方法，两者都可以)
- 最后不要忘了调用 `shutdown()` 方法关闭线程池          
`submit()` 方法还有一个`Future`类型的返回值，`Future`用于获取线程的返回值，`Future`是一个有泛型的类，泛型的类型与`Callable`的泛型相同，调用Future的 `get()` 方法可以获得返回的值       

~~~java
        //execute执行Runnable的例子
		ExecutorService cachedTP = Executors.newCachedThreadPool();
		cachedTP.execute(new ExampleThread());      //ExampleThread实现了Runnable接口
		cachedTP.shutdown();

    //造一个Callable 泛型是String，所以调用时返回的Future类型也是String
    class ExampleThread implements Callable<String> {
        @Override
        public String call() {
            return "Some Value";
        }
    }

        //执行一下
        ExecutorService cachedTP = Executors.newCachedThreadPool();
		Future<String> future = cachedTP.submit(new ExampleThread());   //submit返回的是Future
		try {
			String value = future.get();    //get()方法获得返回值
		} catch (InterruptedException e) {
			e.printStackTrace();
		} catch (ExecutionException e) {
			e.printStackTrace();
		}
		cachedTP.shutdownNow();
~~~
- `future.get()` 方法会阻塞当前线程，一直等到线程池里相应的线程执行结束的时候当前线程才会解除阻塞。因此**get()方法要在不得不用返回值的时候才调用**，否则会影响程序运行的效率

#### 4.2.3 线程池的关闭
`ThreadPoolExecutor` 提供了 `shutdown()` 和 `shutdownNow()` 两个方法来关闭线程池
- `shutdown()` ：锁定，设置线程池状态为**SHUTDOWN**并中断空闲线程，正在执行的线程执行完才中断，不可以再 submit 新的task                    
- `shutdownNow()` :  加锁，设置线程池状态为**STOP**并中断所有线程，正在执行的线程任务被停止，返回没被执行完的 task 列表      

#### 4.2.4 构造方法(自定义线程池)   
~~~java
    public ThreadPoolExecutor(int corePoolSize,     
                              int maximumPoolSize,  
                              long keepAliveTime,   
                              TimeUnit unit,       
                              BlockingQueue<Runnable> workQueue,    
                              ThreadFactory threadFactory,         
                              RejectedExecutionHandler handler) {   
                              /*
                              corePoolSize 指定核心线程数量
                              maximumPoolSize 指定最大线程数量
                              keepAliveTime 允许线程空闲时间
                              unit 时间对象
                              workQueue 阻塞队列
                              threadFactory 线程工厂
                              handler 任务拒绝策略
                              */
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
~~~

## 5. 线程的状态和控制
### 5.1线程的状态        
- **初始态(NEW)：** 创建Thread对象，但还未调用start启动线程，线程处于初始态       
- **运行态(RUNNABLE)：** 包括就绪态(ready)和运行中(running)两种状态       
- **就绪态(READY)：** 获得了执行所需的所有资源，只要CPU分配执行权就能运行，所有就绪态线程存放在就绪队列中     
- **运行中(RUNNING)：** 获得CPU执行权，正在执行，同一个CPU同一时刻只能有一个运行态线程        
- **阻塞态(BLOCKED)：** 请求排它锁失败时的状态        
- **等待态(WAITING)：** 当前线程调用 `wait`, `join`, `park` 函数时，线程进入无限等待态，释放CPU执行权，释放资源(如锁)，等待被其他线程调用 `notify` 方法显示唤醒       
- **超时等待态(TIMED_WAITING)：** 当运行中的线程调用 sleep(time), wait, join, parkNanos, parkUtil 时，进入该状态，释放CPU执行权和占有资源，与等待态的区别：无需等待被唤醒，到时间会由系统自动唤醒      
- **终止态(TERMINATED)：** 运行结束  
<center>

![线程的状态](https://s1.ax1x.com/2020/08/03/adSU9s.png) </center>



### 5.2 线程的控制        
`isAlive()` : 线程是否终止          
`getPriority()`
`setPriority()`
`Thread.sleep()` ： 当前线程进入TIME_WAIT状态，但不释放对象锁，milis后自动苏醒进入READY状态，给其他线程执行机会的最佳方式     
`join()` : 将当前线程与该线程合并，当前线程进入等待状态，等待该线程结束再恢复当前线程到就绪状态         
`yield()` ： 让出CPU，线程进入就绪队列等待调度      
`wait()` ： 当前线程进入对象的 wait pool            
`notify()/notifyAll()` : 唤醒对象的wait pool中的一个/所有等待线程       
`interrupt()` : 请求终止线程(只是请求，设置一个中断标志，需要被通知的线程自己处理)，这样可以安全的终止线程，阻塞线程调用interrupt方法会抛出异常     
- 如果阻塞线程调用了interrupt()方法，会抛出异常，设置中断标志为false，同时该线程退出阻塞，抛出异常  

~~~java
    public static void main(String[] args) {
        Demo main = new Demo();
        // 创建线程并启动
        Thread t = new Thread(main.runnable);
        t.start();
        try {
            Thread.sleep(3000);     //main睡3秒
        } catch (InterruptedException e) {
            System.out.println("In main");
            e.printStackTrace();
        }
        t.interrupt();      //中断t
    }

    Runnable runnable = () -> {
        int i = 0;
        try {
            while (i < 1000) {
                Thread.sleep(500);
                System.out.println(i++);
            }
        } catch (InterruptedException e) {
            // 判断该阻塞线程是否还在
            System.out.println(Thread.currentThread().isAlive());
            // 判断该线程的中断标志位状态
            System.out.println(Thread.currentThread().isInterrupted());
            System.out.println("In Runnable");
            e.printStackTrace();
        }
    };
    /*
    0
    1
    2
    3
    4
    true
    false
    In Runnable
    java.lang.InterruptedException: sleep interrupted
    */
~~~
#### 终结线程     
- **只有在运行和阻塞状态时才有被终结的机会，其它状态时都无法终结**
- 停止线程的执行，可以通过volatile变量设置标志，除了这种简单粗暴的方法外，还可以使用中断信号来停止正在运行的线程。
- 中断信号只是一个线程发给另一个线程的请求信号而不是命令，需要接方自己去处理        
- 可以通过 submit 返回的 **Future对象** 来发送中断信号 `Future.cancel(true)`
~~~java
class InterruptableThread implements Runnable {
    private int value;
    public void run() {
        Thread currentThread = Thread.currentThread();
        while(!currentThread.isInterrupted()) {     //没有收到中断信号执行循环
            value++;
        }
        System.out.println("Interrupted value = "+value); //收到中断信号打印
    }
}
class AlwaysRunThread implements Runnable {
    public void run() {     //不处理中断信号
        while(true) {
            try {
                Thread.currentThread().sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("Running");
        }
    }
}
public class InterruptRunningTest {
    public static void main(String[] args) {
        ExecutorService exec = Executors.newCachedThreadPool();
        Future interruptableFuture= exec.submit(new InterruptableThread());
        Future alwaysRunFuture= exec.submit(new AlwaysRunThread());
        exec.shutdown();
        interruptableFuture.cancel(true);   //Future调用 cancel(true) 方法向线程发送中断信号
        alwaysRunFuture.cancel(true);
    }
}
/*
java.lang.InterruptedException: sleep interrupted
Running
Interrupted value = 48768
Running
Running
*/
~~~



## 6. 线程通信      
多线程并发执行的时候，不同的线程执行的内容可能存在一些依赖关系，比如线程A执行某一部需要线程B里的某方法的结果，一个简单粗暴的办法是设置监控的布尔变量，缺点是等待依赖的条件的时候线程处于**忙等待**状态(运行状态，占用cpu时间)，却没有做有意义的事情，更好的办法是将其阻塞，阻塞状态不占用CPU的时间，从而提高CPU的利用率             
Java提供了线程间合作的机制，即Object.wait()方法、Object.notify()和Object.notifyAll()方法                   
`Object.wait()`：使当前线程阻塞，等待其它线程调用notify()方法，释放当前获取的锁       
`Object.notify()` ：唤醒一个等待着的线程，这个线程唤醒之后尝试获取锁，其它线程继续等待        
`Object.notifyAll()` ：唤醒所有等待着的线程尝试获取锁，这些线程排队等待锁       

**锁的调度：**      
- java中对象锁的模型：JVM会为一个使用内部锁（synchronized）的对象维护两个集合，Entry Set 和 Wait Set (锁池和等待池)     
- **Entry Set：** 如果线程A已经持有了对象锁，此时如果有其他线程也想获得该对象锁的话，它只能进入Entry Set，并且处于线程的BLOCKED状态。当对象锁被释放的时候，JVM会随机唤醒处于Entry Set中的某一个线程，这个线程的状态就从BLOCKED转变为RUNNABLE          
- **Wait Set：** 如果线程A调用了 wait() 方法，那么线程A会释放该对象的锁，进入到Wait Set，并且处于线程的WAITING状态。当对象的notify()方法被调用时，JVM会唤醒处于Wait Set中的某一个线程，这个线程的状态就从WAITING转变为RUNNABLE；或者当notifyAll()方法被调用时，Wait Set中的全部线程会转变为RUNNABLE状态。所有Wait Set中被唤醒的线程会被转移到Entry Set中
- 每当对象的锁被释放后，那些所有处于RUNNABLE状态的线程会共同去竞争获取对象的锁，最终会有一个线程（具体哪一个取决于JVM实现，队列里的第一个？随机的一个？）真正获取到对象的锁，而其他竞争失败的线程继续在Entry Set中等待下一次机会

**java中wait和sleep方法的不同**
1. 最大的不同是 wait 会释放锁，而 sleep 一直持有锁。
2. Wait 通常被用于线程间交互，sleep通常被用于暂停执行。
3. wait()方法会释放 CPU执行权和占有的锁。
4. sleep(long)方法仅释放CPU使用权，锁仍然占用；线程被放入超时等待队列，与Thread.yield() 相比，它会使线程较长时间得不到运行。
5. Thread.yield() 方法仅释放CPU执行权，锁仍然占用，线程会被放入就绪队列，会在短时间内再次执行。
6. wait 和 notify、notifyAll 必须配套使用，即必须使用同一把锁调用；
7. wait和notify必须放在一个同步块中调用，wait和notify的对象必须是他们所处同步块的锁对象

### 6.1 生产者消费者问题        
**等待方(消费者)和通知方(生产者)**   
~~~java
//等待方：
synchronized(obj){
	while(条件不满足){
 	    obj.wait();
    }
    消费;
}

//通知方：
synchronized(obj){
	改变条件;
	obj.notifyAll();
}

~~~
- `wait` 和 `notiy` 方法必须由同一个锁调用，`notify` 只能唤醒使用同一个锁对象调用 `wait` 方法的线程     
- 锁对象可以是任意对象，`wait` 和 `notify` 方法是 `Object` 类的方法         
- `wait` 和 `notify` 方法必须在同步代码块或同步方法中使用(要通过锁对象调用) 
- 当线程从wait方法中被唤醒时，它在重新请求锁时不具有任何特殊的优先性，而要与其他尝试进入同步代码块的线程一起正常地在锁上进行竞争        
- **每当线程从wait中唤醒时，都必须再次测试条件谓词**，在线程被唤醒到wait重新获取锁的这段时间里，可能有其他线程已经获取了这个锁，并修改了对象的状态。

#### 6.1.1 锁池和等待池
Java中，每个对象都有一个唯一与之对应的内部锁（Monitor）。Java虚拟机会为每个对象维护两个“队列”（姑且称之为“队列”，尽管它不一定符合数据结构上队列的“先进先出”原则）：一个叫**Entry Set（入口集、锁池）**，另外一个叫**Wait Set（等待集、等待池）**。对于任意的对象objectX，objectX的**Entry Set用于存储等待获取objectX对应的内部锁的所有线程**。objectX的**Wait Set用于存储执行了objectX.wait()/wait(long)的线程**    
- **锁池(Entry Set)：** 假设线程A已经拥有了某个对象(注意:不是类)的锁，而其它的线程想要调用这个对象的某个synchronized方法(或者synchronized块)，由于这些线程在进入对象的synchronized方法之前必须先获得该对象的锁的拥有权，但是该对象的锁目前正被线程A拥有，所以这些线程就进入了该对象的锁池中

- **等待池(Wait Set):** 假设一个线程A调用了某个对象的wait()方法，线程A就会释放该对象的锁(因为wait()方法必须出现在synchronized中，这样自然在执行wait()方法之前线程A就已经拥有了该对象的锁)，同时线程A就进入到了该对象的等待池中。如果另外的一个线程调用了相同对象的notifyAll()方法，那么处于该对象的等待池中的线程就会全部进入该对象的锁池中，准备争夺锁的拥有权。如果另外的一个线程调用了相同对象的notify()方法，那么仅仅有一个处于该对象的等待池中的线程(随机)会进入该对象的锁池


#### 6.1.2 notify 和 notifyAll的问题：
- 一个生产者两个消费者的情况：一开始三个线程都在Entry Set中竞争锁，两个消费者先后拿到锁并进入了Wait Set，此时生产者拿到锁并生产，然后唤醒消费者1，自己进入到Wait Set，消费者1拿到锁消费后进入Wait Set，并唤醒了消费者2(此时唤醒生产者是没有问题的，但是恰恰唤醒的是消费者2)，此时消费者2拿到锁，无法消费，进如Wait Set，至此，所有线程都进入到了Wait Set，死锁就发生了


- 考虑两个消费者和生产者的场景：如果我们代码中使用了notify()而非notifyAll()，假设消费者线程1拿到了锁，判断buffer为空，那么wait()，释放锁；然后消费者2拿到了锁，同样buffer为空，wait()，也就是说此时Wait Set中有两个线程；然后生产者1拿到锁，生产，buffer满，notify()了，那么可能消费者1被唤醒了，但是此时还有另一个线程生产者2在Entry Set中盼望着锁，并且最终抢占到了锁，但因为此时buffer是满的，因此它要wait()；然后消费者1拿到了锁，消费，notify()；这时就有问题了，此时生产者2和消费者2都在Wait Set中，buffer为空，如果唤醒生产者2，没毛病；但如果唤醒了消费者2，因为buffer为空，它会再次wait()，这就尴尬了，万一生产者1已经退出不再生产了，没有其他线程在竞争锁了，只有生产者2和消费者2在Wait Set中互相等待，那传说中的死锁就发生了     

### 6.2 阻塞队列
阻塞队列可以让我们在任何时刻向队列内扔一个对象，如果队列满了则当前线程阻塞；在任何时刻都可以从队列中取出一个对象，如果队列为空则当前线程阻塞。阻塞队列是线程安全的，使用它时无需加锁。此外其内部是使用显示锁实现的同步，使用Condition实现的线程阻塞。阻塞队列的接口是`BlockingQueue`,它有两个实现类：               
`ArrayBlockingQueue`：底层使用数组实现的队列，有固定长度，调用其构造方法时必须提供队列的最大长度              
`LinkedBlockingQueue`：底层使用链表实现的队列，理论上讲是没有最大长度的，使用时不用提供队列长度；但实际上这个队列的长度不能超过Integer.MAX_VALUE            

- 食堂打饭的例子：
~~~java
class Student implements Runnable {
	private Object wan = new Object();
	public void run() {
		try {
			System.out.println("学生：取到了一个碗");
			BlockingQueueTest.wanQueue.put(wan);
			System.out.println("学生：阿姨帮忙盛饭");
			wan = BlockingQueueTest.wanWithFanQueue.take();
			System.out.println("学生：吃饭");
		} catch (InterruptedException e) {}
	}
}
class CafeteriaWorker implements Runnable {
	public void run() {
		try {
			Object wan = BlockingQueueTest.wanQueue.take();
			System.out.println("阿姨：给学生盛饭");
			BlockingQueueTest.wanWithFanQueue.put(wan);
		} catch (InterruptedException e) {}
	}
}
public class BlockingQueueTest {
	public static BlockingQueue wanQueue = new LinkedBlockingQueue();
	public static BlockingQueue wanWithFanQueue = new LinkedBlockingQueue();
	public static void main(String[] args) {
		ExecutorService exec = Executors.newCachedThreadPool();
		exec.execute(new Student());
		exec.execute(new CafeteriaWorker());
		exec.shutdown();
	}
}
/*
学生：取到了一个碗
学生：阿姨帮忙盛饭
阿姨：给学生盛饭
学生：吃饭
*/
~~~

### 6.3 管道通讯
管道方式实现线程通信和阻塞队列类似，管道内没有数据的时候如果某个线程尝试读取数据就会被阻塞      
通过PipedWriter 和 Pipedreader 来实现对管道数据的读写，与阻塞队列不同点在于，阻塞队列中不同线程都是操作队列的同一种对象，管道可以让不同的线程使用不同的对象，只要他们注册为一个管道即可           
~~~java
class Sender implements Runnable {
    private PipedWriter writer;
    Sender(PipedWriter writer) {
        this.writer = writer;
    }
    public void run() {
        String str1 = new String("I love you\n");
        String str2 = new String("Do you love me\n");
        try {
            writer.write(str1.toCharArray());
            writer.write(str2.toCharArray());
        } catch (IOException e) {}
    }
}
class Receiver implements Runnable {
    private PipedReader reader;
    public Receiver(PipedReader reader) {
        this.reader = reader;
    }
    public void run() {
        try {
            while(true) {
                char c = (char)reader.read();
                System.out.print(c);
            }
        } catch (IOException e) {}
    }
}
public class PipeCommunicationTest {
    public static void main(String[] args) throws Exception {
        PipedReader reader = new PipedReader();
        PipedWriter writer = new PipedWriter(reader);
        ExecutorService exec = Executors.newCachedThreadPool();
        exec.execute(new Sender(writer));
        exec.execute(new Receiver(reader));
        Thread.sleep(1000);
        exec.shutdownNow();
    }
}
~~~

我们在主方法里先定义了一个PipedReader对象，然后将这个对象作为PipedWriter的构造方法的参数传给PipedWriter对象，这样就实现两个输入输出流的绑定，分别将两个流对象传给两个线程对象，主线程sleep一秒后调用了shutdownNow()方法，这个方法向所有运行着的线程发送中断信号，程序运行一秒后就退出了，我们可以看出中断信号打断了Receiver的阻塞状态，由此得出结论：**管道类阻塞可以被中断信号打断**，普通的IO是不能被interrupt中断的

## 7. 死锁      
造成死锁的原因：
- 当前线程拥有其他线程需要的资源
- 当前线程等待其他线程已拥有的资源
- 都不放弃自己拥有的资源

### 7.1 顺序死锁    
- 线程A --> 锁住left --> 尝试锁住right --> 永久等待      
- 线程B --> 锁住right --> 尝试锁住left --> 永久等待     

线程A调用leftRight()方法，得到left锁，同时线程B调用rightLeft()方法，得到right锁。线程A和线程B都继续执行，此时线程A需要right锁才能继续往下执行，线程B需要left锁才能继续往下执行。但是：线程A的left锁并没有释放，线程B的right锁也没有释放。所以他们都只能等待，而这种等待是无期限的-->永久等待-->死锁

### 7.2 动态顺序死锁        
- 由于方法入参由外部传递而来，方法内部虽然对两个参数按照固定顺序进行加锁，但是由于外部传递时顺序的不可控，而产生锁顺序造成的死锁，即动态锁顺序死锁      
~~~java
public static void transferMoney(Account fromAccount,
                                 Account toAccount,
                                 DollarAmount amount)
        throws InsufficientFundsException {
    synchronized (fromAccount) {
        synchronized (toAccount) {
            if (fromAccount.getBalance().compareTo(amount) < 0)
                throw new InsufficientFundsException();
            else {
                fromAccount.debit(amount);
                toAccount.credit(amount);
            }
        }
    }
}
~~~

如果线程A从X账户向Y账户转账，同时线程B从Y账户向X账户转账，就会发生死锁

**解决办法**：固定加锁的顺序，可以使用对象的hash值比较来按hash值大小顺序进行加锁

### 7.3 协作对象之间的死锁
~~~java
public class ThreadDeadLockTest{

    class Taxi{  
        private Point location,destination;  
        private final Dispatcher dispatcher;  
        public Taxi(Dispatcher dispatcher) {  
            this.dispatcher=dispatcher;  
        }  
        public synchronized Point getLocation(){ //获取同步锁 
            return location;  
        }  
        //获取Taxi同步锁，同时还会获取Dispatcher的同步锁
        public synchronized void setLocation(Point location){  
            this.location=location;  
            if(location.equals(destination))  
                dispatcher.notifyAvailable(this);  
        }  
    } 

    class Dispatcher{

        private final Set<Taxi> taxis;
        private final Set<Taxi> availableTaxis;

        public Dispatcher() {
            taxis = new HashSet<Taxi>();
            availableTaxis = new HashSet<Taxi>();
        }
        public synchronized void notifyAvailable(Taxi taxi) {//获取同步锁
            availableTaxis.add(taxi);
        }
        //获取同步锁，同时还会获取Taxi的同步锁
        public synchronized Image getImage() {
            Image image = new Image();
            for(Taxi t : taxis) {
                image.drawer(t.getLocation());
            }
            return image;
        }
    }
}
~~~

setLocation 和 getImage 方法都会获取两个锁。假如一个线程调用了 setLocation 方法，那么首先会获取到一个 Taxi 的锁，然后嵌套调用了 notifyAvailable，或获取到 Dispatcher 的锁。同样，另外一个线程调用了 getImage 方法，首先他会获取 Dispatcher 的锁，然后获取 Taxi 的锁。所以两个线程会以不同的顺序占用锁，那么就会产生死锁的情况
- 线程A --> Taxi的锁 --> Dispatcher 的锁
- 线程B --> Dispatcher的锁 --> Taxi 的锁            

**解决办法：** 开放调用(调用某个方法时不需要持有锁)，减少synchronized的范围，只保护那些调用共享状态的操作，避免死锁

~~~java
        //缩小锁的范围，不会同时去获取两个锁
        public void setLocation(Point location) {
            boolean isReached = false;
            //缩小锁的范围
            synchronized (this) {
                this.location = location;
                isReached = location.equals(destination);
            }
            if(isReached) {
                dispatcher.notifyAvailable(this);
            }
        }

        //缩小锁的范围，不会同时去获取两个锁
        public Image getImage() {
            Set<Taxi> copy;
            //缩小锁的范围
            synchronized(this) {
                copy = new HashSet<Taxi>(taxis);
            }
            Image image = new Image();
            for(Taxi t : copy) {
                image.drawer(t.getLocation());
            }
            return image;
        }
~~~
### 7.4 死锁的解决办法      
- **固定加锁顺序：** 针对顺序死锁
- **开放调用：** 针对协作对象之间造成的死锁
- **使用定时锁：** tryLock()， 如果等待获取锁超时，则抛出异常而不是一直等待