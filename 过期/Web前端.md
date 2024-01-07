# 简介

## B/S架构(Browser/Server)的资源分类
### 1. 静态资源：使用静态网页开发技术发布的资源
特点：所有用户都可以访问，得到的结果是一样的，如文本、图片、音频、视频、HTML、CSS、JavaScript           
用户请求静态资源时，服务器直接把静态资源发送给浏览器，浏览器进行解析和展示      
- HTML：用于搭建基础网页，展示页面的内容
- CSS：用于美化页面，布局页面
- JavaScript：控制页面的元素，让页面有一些动态的效果


### 2. 动态资源：使用动态网页开发技术发布的资源     
特点：不同用户访问得到的结果不一样，如 jsp/servlet、php、asp        
用户请求动态资源时，服务器会执行动态资源，转换为静态资源再发送给浏览器




# HTML  
超文本标记语言（英语：HyperText Markup Language，简称：HTML）是一种用于创建网页的标准标记语言       
- 超文本：是一种可以显示在电脑显示器或电子设备上的文本，现时超文本普遍以电子文档的方式存在，其中的**文字包含有可以链接到其他字段或者文档的超链接**，允许从当前阅读位置直接切换到超链接所指向的文字          
- 标记语言：由标签构成的语言 <标签名称>，如 html、xml       

~~~html
<!DOCTYPE html>           
<html>
<head>
<meta charset="UTF-8">
<title>页面标题</title>
</head>
<body>

<h1>我的第一个标题</h1>

<p>我的第一个段落。</p>

</body>
</html>

~~~

- DOCTYPE 声明 不区分大小写：HTML,html,Html都可以
- 对于中文网页需要使用 `<meta charset="utf-8">` 声明编码，否则会出现乱码。目前在大部分浏览器中，直接输出中文会出现中文乱码的情况，这时候需要在头部将字符声明为 UTF-8    
- html标签也不区分大小写，但是建议小写      
- 标签分为围堵标签 和 自闭和标签`<br/>` 

## 1. 标签

### 1.1 文件标签

| 标签         | 含义                                              |
| ------------ | ------------------------------------------------- |
| `<!DOCTYPE>` | 定义文档类型                                      |
| `<html>`     | html文档的根标签                                  |
| `<head>`     | 头标签，可以指定html 文档的一些属性，引入外部资源 |
| `<title>`    | 标题标签                                          |
| `<body>`     | 定义文档的主体                                    |

### 1.2 文本标签

| 标签           | 含义                                         |
| -------------- | -------------------------------------------- |
| `<!--...-->`   | 注释                                         |
| `<h1> to <h6>` | HTML 标题，一共六个等级                      |
| `<p>`          | 定义一个段落                                 |
| `<br>`         | 换行                                         |
| `<hr>`         | 显示一条水平线  color,width,size,align(对齐) |
| `<b>`          | 定义粗体文本                                 |
| `<i>`          | 定义斜体文本                                 |
| `<font>`       | 定义样式(过时)                               |
| `<center>`     | 居中(过时)                                   |
### 1.3 图片标签
| 标签           | 含义                                                          |
| -------------- | ------------------------------------------------------------- |
| `<img>	      ` | 定义图像                                                      |
| `<map>       ` | 定义图像映射                                                  |
| `<area>	     ` | 定义图像地图内部的区域                                        |
| `<canvas>    ` | 通过脚本（通常是 JavaScript）来绘制图形（比如图表和其他图像） |
| `<figcaption>` | 定义一个 caption for a `<figure>` element                     |
| `<figure>    ` | figure 标签用于对元素进行组合                                 |

**属性**
![abx5NR.png](https://s1.ax1x.com/2020/08/10/abx5NR.png)

### 1.4 列表标签

| 标签        | 含义                                                   |
| ----------- | ------------------------------------------------------ |
| `<ul>`      | 定义一个无序列表，type：disc,square,circle             |
| `<ol>`      | 定义一个有序列表，type：1,a,A,I,i                      |
| `<li>`      | 定义一个列表项                                         |
| `<dir>`     | HTML5不再支持。 HTML 4.01 已废弃。 定义目录列表。      |
| `<dl>`      | 定义一个定义列表                                       |
| `<dt>`      | 定义一个定义定义列表中的项目。                         |
| `<dd>`      | 定义定义列表中项目的描述。                             |
| `<menu>`    | 定义菜单列表。                                         |
| `<command>` | 定义用户可能调用的命令（比如单选按钮、复选框或按钮）。 |

### 1.5 链接标签
| 标签      | 含义                     |
| --------- | ------------------------ |
| `<a>	   ` | 定义一个链接             |
| `<link>	` | 定义文档与外部资源的关系 |
| `<main>	` | 定义文档的主体部分       |
| `<nav>  ` | 定义导航链接             |

a 中的属性：    
![aq9FKK.png](https://s1.ax1x.com/2020/08/10/aq9FKK.png)

### 1.6 块标签

| 标签     | 含义                                   |
| -------- | -------------------------------------- |
| `<span>` | 包裹，默认没有效果，行内标签，不换行   |
| `<div>`  | 包裹，默认没有效果，块级标签，沾满一行 |

### 1.7 语义化标签
- h5为了提高程序可读性提供的一些标签
| 标签       | 含义         |
| ---------- | ------------ |
| `<header>` | 定义头标签   |
| `<footer>` | 定义脚注标签 |

### 1.8 表格标签
| 标签         | 含义                           |
| ------------ | ------------------------------ |
| `<table>	  ` | 定义一个表格                   |
| `<caption>	` | 定义表格标题                   |
| `<th>	     ` | 定义表格中的**表头单元格**     |
| `<tr>	     ` | 定义表格中的**行**             |
| `<td>	     ` | 定义表格中的**单元**           |
| `<thead>	  ` | 定义表格中的表头内容           |
| `<tbody>	  ` | 定义表格中的主体内容           |
| `<tfoot>	  ` | 定义表格中的表注内容（脚注）   |
| `<col>	    ` | 定义表格中一个或多个列的属性值 |
| `<colgroup>` | 定义表格中供格式化的列组       |

- width : 宽度
- broder : 边框
- border-collapse: collapse; 设置边框合并
- cellpadding : 定义内容和单元格的距离
- cellspacing : 定义单元格之间的距离
- bgcolor : 定义表格的背景色    
- colspan : 合并列
- rowspan : 合并行

### 1.9 表单标签    
| 标签         | 含义                                               |
| ------------ | -------------------------------------------------- |
| `<form>	   ` | 定义一个 HTML 用户输入表单，只有表单内的内容被提交 |
| `<input>	  ` | 定义一个输入控件，可以定义 type                    |
| `<textarea>` | 定义多行的文本输入控件                             |
| `<button>	 ` | 定义按钮                                           |
| `<select>	 ` | 定义选择列表（下拉列表）                           |
| `<optgroup>` | 定义选择列表中相关选项的组合                       |
| `<option>	 ` | 定义选择列表中的选项                               |
| `<label>	  ` | 定义 input 元素的标注                              |
| `<fieldset>` | 定义围绕表单中元素的边框                           |
| `<legend>	 ` | 定义 fieldset 元素的标题                           |
| `<datalist>` | 规定了 input 元素可能的选项列表                    |
| `<keygen>  ` | 规定用于表单的密钥对生成器字段                     |
| `<output>  ` | 定义一个计算的结果                                 |

- 表单的属性    
![aqNMAU.png](https://s1.ax1x.com/2020/08/10/aqNMAU.png)

- 表单中的数据要被提交，则必须指定其 **name 属性**      

**method参数 get 和 post 的区别：** 
- get请求参数会在地址栏中显示，会封装在请求行中，请求参数大小有限制，不太安全           
- post请求参数不会在地址栏中显示，会封装在请求体中，请求参数大小没有限制，较安全    

**input:**  


1. 可以通过type属性改变元素展示的样子     
   - text ： 默认为文本输入框      
   - password ： 密码，密文显示        
   - radio : 单选框，name 属性一样才能单选，使用 value 属性表示提交后的值          
   - checkbox ： 多选框，同样通过指定 value 属性表示提交后的值  
     - checked ： 指定默认值
   - file ：上传文件的选框          
   - hidden ： 隐藏域，用于提交一些信息   
   - submit ： 提交按钮，可以提交表单       
   - button ： 普通按钮     
   - image : 图片提交按钮      
   - color ： 定义拾色器        
   - date ：  日期      
   - email ： 邮箱格式          
     
2. placeholder：placeholder 属性规定可描述输入 `<input>` 字段预期值的简短的提示信息，输入框有内容时自动被清空       
3. label ： 指定输入向的文字描述信息， label 的 for 属性和 input 的 id 对应，点击 label 区域可以让 input 输入框获取焦点         

**select：**
- 一定要加name，否则无法提交到服务器
- value设置提交的值
~~~html
<select name="provice">
    <option value="">--请选择--</option>
    <option value="beijing">北京</option>
    <option value="shanghai">上海</option>
    <option value="shenzhen">深圳</option>
</select>
~~~

**textarea：**
- 定义一个多行输入的文本域
- cols 指定列数(每行字符数)
- rows 指定行数

# CSS
Cascading Style Sheets 层叠样式表
- 层叠：多个样式作用在同一个html 的元素上，同时生效    
- 可以将内容展示和样式控制分离：解耦，分工协作，提高开发效率            
## 1. CSS和HTML的结合方式
### 1.1 内联样式    
- 标签内通过style控制
~~~html
<div style="color:red;">一些文字</div>
~~~

### 1.2 内部样式 
- 可以控制当前页面所有div
~~~html
<style>
    div{
        color:blue;
    }
</style>

<div>一些文字</div>
~~~   

### 1.3 外部样式
- 在head标签中定义 link标签引入外部资源文件 

~~~css
div{
    color:red;
}
~~~

~~~html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <link rel="stylesheet" href="css/a.css">
</head>
<body>
正文
<div>被css控制的样式</div>>
</body>
</html>
~~~ 

- CSS 的格式

~~~html
选择器{
    属性1: 值1;
    属性2: 值2;
    ..
}
~~~
选择器可以筛选具有相似特征的元素                            

## 2. CSS 选择器
### 2.1 基础选择器
- id选择器 `#` ：选择具体的 id 属性值的元素 
- 元素选择器 ： 选择具有相同标签名称的元素，id选择器优先级高于元素选择器  
- 类选择器 `.` ：选择具有相同 class 属性的元素      
~~~html
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        #d123{
            color:red;
        }
        div{
            color:green;
        }
        .cls123{
            color:blue;
        }
    </style>
</head>
<body>
正文
<div id="d123">被 id 控制的样式
    <p class="cls123">被class控制的段落</p>
    被 id 控制的样式
</div>
<div>另一个被元素选择器控制的样式</div>
</body>
</html>
~~~

### 2.2 扩展选择器
- 选择所有元素：`*` 
- 并集选择器：`div,p` 逗号连接多个元素
- 子选择器：`选择器1 选择器2{}` 选择选择器1的子选择器2
- 父选择器：`选择器1>选择器2{}` 选择选择器2的父选择器1
- 属性选择器：`元素名称[属性名=属性值]{}` 
- 伪类选择器：`元素：状态{}`选择一些元素具有的状态
~~~html
如 <a>中的状态
    a:link{
        color:black;  <!--初始状态 -->
    }
    a:hover{
        color:blue;  <!--鼠标悬停状态 -->
    }
    a:active{
        color:red;  <!--点击状态 -->
    }a:visited{
        color:yellow;  <!--点击后状态 -->
    }
~~~

## 3. CSS 属性      

### 3.1 字体、文本
- font-size ： 单位为像素 px
- color     
- text-align : 文本对齐方式    
- line-height ： 文本行高       
### 3.2 背景
- backgroud ： 复合属性：图片，重复，对齐等
### 3.3 边框
- border ： 设置边框，复合属性：粗细，线的样式，颜色等      
### 3.4 尺寸
- width : 宽度          
- height : 高度
### 3.5 盒子模型：控制布局
- margin : 外边距 auto值表示水平居中
- padding ： 内边距，默认情况下内边距会影响盒子大小，设置 `box-sizing:border-box` 可以不再影响          
- float ： 浮动 











-----------
# JavaScript        
客户端脚本语言，浏览器自带了JS的解析引擎，JS可以控制html元素，让页面有一些动态的效果，增强用户体验      
JavaScript 包含 ECMAScript 标准和 JS特有的 BOM、DOM             

## 1. ECMAScript            
- 动态类型语言(Dynamically Typed Language) ： 运行时才做数据类型检查的语言(JavaScript、Ruby、Python)       
- 静态类型语言(Statically Typed Language) : 编译时进行数据类型检查，写程序时必须声明变量的数据类型(JAVA, C, C++, C#)  
- 强类型语言，一个变量不经过强制转换，它永远是这个数据类型，不允许隐式的类型转换。举个例子：如果你定义了一个double类型变量a,不经过强制类型转换那么程序 int b = a 无法通过编译。典型代表是Java、Python
- 弱类型语言：它与强类型语言定义相反,允许编译器进行隐式的类型转换，典型代表C/C++  
- JS是动态弱类型语言
### 1.1 与html结合的方式
- 内部JS            
可以在html中任意位置定义 `<script>` ，可定义多个，按顺序执行，阻塞式

- 外部JS        
通过 src 属性引入外部 js 文件       

### 1.2 数据类型
1. 原始数据类型 
- number : 整数/小数/NaN
- string ： 没有字符，单双引号均可
- boolean       
- null : typeof(null) 返回object      
- undefined ：未定义。变量未初始化时，值为undefined，null==undefined返回true

2. 引用数据类型

### 1.3 变量
~~~js
var a = 1;
document.write(a + "---" + typeof (a) + '<br>');
a = NaN;
document.write(a + "---" + typeof (a) + '<br>');
a = "123";
document.write(a + "---" + typeof (a) + "<br>");
a = null;
document.write(a + "---" + typeof (a) + "<br>");
a = undefined;
document.write(a + "---" + typeof (a) + "<br>");
/*
1---number
NaN---number
123---string
null---object
undefined---undefined
*/

~~~

### 1.4 运算符      
1. 一元运算
- 如果运算数不是运算符要求的类型，会自动进行转换      
- **其他类型转 number**      
~~~js
//string转number，按字面值转换，字面值不是数字时转为 NaN
a = +"123";
document.write(a + "---" + typeof (a) + "<br>");    //123---number
a = +"qwq1";
document.write(a + "---" + typeof (a) + "<br>");    //NaN---number

//booolean转number，true转1，false转0
//null 转 number 为 0
//其他都转为 NaN
~~~

2. 比较运算符
- 类型相同直接比，不相同则转换  
- === 全等于，先判断类型，类型不同直接false
~~~js
a = "1"==1;
document.write(a + "---" + typeof (a) + "<br>");    //true---boolean
a = "1"===1;
document.write(a + "---" + typeof (a) + "<br>");    //false---boolean
a = "a">1;
document.write(a + "---" + typeof (a) + "<br>");    //"a"转为NaN 结果：false---boolean
a = "a"<1;
document.write(a + "---" + typeof (a) + "<br>");    //"a"转为NaN 结果：false---boolean
~~~

- **其他类型转 boolean**
~~~java
//number 转 boolean：0和 NaN 为假，其他为真 
a = !!-3;
document.write(a + "---" + typeof (a) + "<br>");    //true---boolean
//string 转 boolean：只要不是空字符串，均转为真
a = !!"0";
document.write(a + "---" + typeof (a) + "<br>");    //true---boolean
//对象 转 boolean：只要不是 null，均为真
a = !!new Date();
document.write(a + "---" + typeof (a) + "<br>");    //true---boolean
~~~

### 1.5 流程控制
- if...else...
- switch
- while
- do...while
- for

### 1.6 常用对象
**Function对象：**
- `function 方法名(参数列表){方法体}`       
- `var 方法名 = funciton(参数列表){方法体}`
- JS 方法生命中有一个内置数组对象，`arguments` 接收所有的实际参数
~~~js
    //第一种方法
    function add(a,b){
        return a+b;
    }

    //第二种方法
    var sub = function(a,b){
        return a-b;
    }

    document.write(add(1,"2kajskj")+"<br>");    //12kajskj
    document.write(sub(9,"2"));                //7
~~~

**Array**
- 传入一个参数的时候只能指定数组的长度          
- 元素类型可变
~~~js
var arr1 = new Array(1,"abc",true);        //1,abc,true
var arr2 = new Array(5);            //,,,,
var arr3 = new Array(5,);           //,,,,
var arr4 = [1,2,3];                 //1,2,3

document.write(arr1[10]+"<br>");    //undefined
document.write(arr1.length+"<br>"); //3
arr1[10]=false;
document.write(arr1[10]+"<br>");    //false
document.write(arr1.length+"<br>"); //11  
~~~
- `join()` 将数组元素按指定分隔符转换为字符串
- `push()` 从尾部添加元素

**Date**    

- `toLocaleString()`
- `getTime()` : 返回相对时间的毫秒值

~~~js
var date = new Date();
document.write(date.toLocaleString()+"<br>");  //2020/8/11 下午7:17:32

document.write(date.getTime()+"<br>");         //1597144717013
~~~

**Math**
- 不用创建对象
- `Math.PI` 
- `random()` 返回 [0.0,1) 的随机数
- `rand()`
- `floor()`
- `ceil()`

**RegExp**
- `var re = new RegExp("regular expression")`
- `var re = /regular expression/`
- `test(string)` : 判断该string是否符合re 
- 一般用第二种方式定义，第一种传入字符串，用 \ 时需要转义很麻烦     
~~~js
var re1 = new RegExp("^[a,b](1)*");
var re2 = /1*\w$/;
document.write(re1.test("b")+"<br>");   //true
document.write(re2.test("b")+"<br>");   //true
~~~

**Global:**
- Global 全局对象，方法不需要对象就可以直接调用
- `encodeURI()` : 返回字符串的URI编码，不会对下列字符编码： **ASCII字母 数字 ~!@#$&*()=:/,;?+'**
- `decodeURI()`
- `encodeURIComponent()` : 编码范围更大，不会对下列字符编码： **ASCII字母 数字 ~!*()'**
- `decodeURIComponent()`
- 如果只是编码字符串，不和URL有关系，那么用escape就可以
- 如果需要编码整个URL，然后使用这个**URL**，那么用**encodeURI**，因为encodeURIComponent会把 :/ 也编码，破坏了url
- 使用需要编码URL中的参数的时候，用**encodeURIComponent**方法，

~~~js
字符串 = encodeURI("滴滴")
document.write( 字符串 + "<br>");   //%E6%BB%B4%E6%BB%B4

encodeURI("http://www.baidu.com");          //http://www.baidu.com
encodeURIComponent("http://www.baidu.com"); //http%3A%2F%2Fwww.baidu.com
~~~
- `parseInt()` ： 将字符串转数字，逐一判断每个字符是否是数字，是则转，直到遇到第一个非数字就停止    
- `isNaN()` : 因为NaN参与的比较全都是false，所以只能用这个函数判断，**NaN==NaN的结果也是false** 
- `eval()` : 将字符串转换为JS代码执行   


## 2. BOM(Browser Object Model)
### 2.1 Window 窗口
- 不需要创建，直接使用 window.方法 调用，也可以直接方法调用，直接获取属性       
- `alert()` : 弹窗显示
- `confirm()` : 弹窗显示，有确认和取消按钮(返回true和false)，防止用户误操作           
- `prompt()` : 显示信息，并获取用户输入           
- `open()` : 打开新窗口
- `close()` : 关闭当前窗口(调用的window对象)
- `setTimeout()` : 一次性定时器，参数：对象,毫秒值；返回定时器的唯一标识(用于取消定时器)
- `clearTimeout()` : 取消定时器，传入定时器的唯一标识     
- `setInterval()` ：循环定时器
- `history` ： window的属性，直接获取对应的BOM对象
- `navigator`
- `location`
- `screen`
- `document` : 直接获取document对象  
### 2.2 Navigator 浏览器
### 2.3 Screen 显示器
### 2.4 History 历史记录

### 2.5 Location 地址栏
- `reload()` : 重新加载当前文档：刷新
- `href` ：href属性，获取页面的网址(可以通过点击修改 href 实现点击跳转功能)     


## 3. DOM(Document)
- 将html 文档封装为树结构，可以控制 html 文档的内容，DOM定义了 html 和 xml 文档的标准                 
- 分为 核心DOM、HTML DOM、XML DOM        

**Document 文档对象**
- `document.getElementById("id值")` ： 获取当前页面标签(元素)对象               
    通过 `element.属性` 来获取修改属性值，通过`element.innerHTML`来获取修改标签内部的值
- `getElementsByTagName()` ： 根据元素名获取对象，返回数组      
- `getElementsByClassName()`
- `getElementsByName()`
- `crementElement()`
- `crementAttribute()`
- `crementTextNode()`
- `crementComment()`
**Element 元素对象**
- 通过document创建和获取    
- `removeAttribute()`   
- `setAttribute(属性,属性值)` 设置元素的属性
- `innerHTML` : 直接通过 = 修改或者 += 增加HTML，是HTML DOM 元素对象的属性
- `style` ： 通过 style 属性可以直接获取 CSS 中的属性并修改样式

**Attribute 属性对象**


**Text 文本对象**

**Comment 注释对象**    

**Node 节点对象(其他对象的父类对象)**
- 可以是元素节点、属性节点、文本节点等，所有的DOM对象都可以被认为是一个节点     
- `appendChild()` ： 向节点的子节点列表的结尾添加新的子节点
- `removeChild()` : 删除并返回当前节点指定的子节点
- `replaceChild()` : 用新的节点替换一个子节点                   
- `parentNode` : 获取父节点



## 4. 事件 
**概念**
- 事件：用户操作
- 事件源：组件，如按钮，文本输入框
- 监听器：代码
- 注册监听：将时间，事件源，监听器结合在一起，当事件源上发生事件时，出发执行监听源的代码
- 可以直接定义在标签中(不推荐)，或者通过 JS 获取元素对象，指定事件属性，设置相应的函数  
**点击事件**
- `onclick`
- `ondbclick` ： 双击

**焦点事件**
- `onblur` : 失去焦点(表单校验)
- `onfocus` : 元素获得焦点
**加载**
- `onload` : 一张页面或者一幅图完成加载(可以直接用window.onload 等待页面加载完成)     

**鼠标事件**
- `onmousedown` : 鼠标按钮按下(定义形参接受event对象，event.button的值代表了不同鼠标按键：左键0，滚轮1，右键2)
- `onmouseup`   ：鼠标按钮松开
- `onmousemove` ：鼠标在元素内被移动
- `onmouseover` ：鼠标移到某元素之上
- `onmouseout`  ：鼠标从某元素移开

**键盘事件**
- `onkeyup`     : 某个键盘按键松开
- `onkeydown`   ：某个键盘按键按下(event.keyCode的值代表不同按键：回车13)
- `onkeypress`  ：某个键盘按键按下并松开

**选择和改变**
- `onchange` : 域的内容被改变(可以用于下拉列表修改后执行...)
- `onselect` : 文本被选中

**表单事件**
- `onsubmit` : 确认按钮被点击(检查格式，格式不正确时阻止 form 表单的提交)
- `onreset` : 重置按钮被点击


## 5. HTML DOM 样式控制
1. 使用元素的 style 属性设置，通过 style 属性可以直接获取 CSS 中的属性并修改样式     
~~~js
div1.style.border = "1px solid red";
div1.style.width = "200px";
div1.style.fontSize = "20px";
~~~

2. 提前定义好类选择器的样式，通过元素的className属性值来设置其class属性值

~~~js
div2.className = "d1";
~~~

## 5. 案例：动态表格
~~~html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>DynamicTable</title>
    <style>
        #input {
            margin-left: 39%;
            margin-top: 10%;
        }

        table {
            height: auto;
            width: auto;
            margin-left: 36%;
            margin-top: 3%;
            text-align: center;
            border: black solid thick;
            border-collapse: collapse;
        }

        th, td {
            height: auto;
            width: 150px;
            border: black solid thin;
        }

        input, select, button {
            height: 30px;
        }

    </style>
</head>
<body>
<div id="input" class="input" name="input">
    <input name="inputID" id="inputID" class="inputValue" placeholder="   请输入编号">
    <input name="inputName" id="inputName" class="inputValue" placeholder="   请输入姓名">
    <select id="inputGender" name="inputGender" class="inputGender">
        <option value="Male">男</option>
        <option value="Female">女</option>
    </select>
    <button id="addButton">添加</button>
</div>
<table>
    <caption>学生信息表</caption>
    <div id="tableHead" class="tableHead" name="tableHead">
        <tr>
            <th class="tableText" name="tableTextHead">编号</th>
            <th class="tableText" name="tableTextHead">姓名</th>
            <th class="tableText" name="tableTextHead">性别</th>
            <th class="tableText" name="tableTextHead">操作</th>
        </tr>
    </div>
    <tr>
        <td>1</td>
        <td>小明</td>
        <td>男</td>
        <td><a href="javascript:void(0)" onclick="del(this)">删除</td>
    </tr>
    <tr>
        <td>2</td>
        <td>小李</td>
        <td>男</td>
        <td><a href="javascript:void(0)" onclick="del(this)">删除</td>
    </tr>
    <tr>
        <td>3</td>
        <td>小蓝</td>
        <td>女</td>
        <td><a href="javascript:void(0)" onclick="del(this)">删除</td>
    </tr>

</table>

<script>
    //使用DOM
    var addButton = document.getElementById("addButton");
    addButton.onclick = function () {
        var inputID = document.getElementById("inputID").value;
        var inputName = document.getElementById("inputName").value;
        var inputGender = document.getElementById("inputGender").value;
        inputGender = inputGender == "Male" ? "男" : "女";
        var table = document.getElementsByTagName("table")[0];

        //创建tr
        var new_tr = document.createElement("tr");
        table.appendChild(new_tr);

        //创建td
        var new_td_id = document.createElement("td");
        //文本不能直接当作节点加入，创建文本节点
        new_td_id.appendChild(document.createTextNode(inputID));

        var new_td_name = document.createElement("td");
        new_td_name.appendChild(document.createTextNode(inputName));

        var new_td_gender = document.createElement("td");
        new_td_gender.appendChild(document.createTextNode(inputGender));

        var new_td_del_text = document.createTextNode("删除");
        var new_td_del_a = document.createElement("a");
        new_td_del_a.setAttribute("href", "javascript:void(0)")
        new_ta_del_a.setAttribute("onclick", "del(this)")
        new_td_del_a.appendChild(new_td_del_text);

        new_tr.appendChild(new_td_id);
        new_tr.appendChild(new_td_name);
        new_tr.appendChild(new_td_gender);
        new_tr.appendChild(new_td_del_a);
    }

    var del = function(obj){
        var tr = obj.parentNode.parentNode;
        var table = tr.parentNode;
        table.removeChild(tr);
    }

</script>
</body>
</html>
~~~

此处如果使用HTML DOM 中的 innerHTML 则简便不少
~~~html
<script>
    var addButton = document.getElementById("addButton");
    addButton.onclick = function () {
        var inputID = document.getElementById("inputID").value;
        var inputName = document.getElementById("inputName").value;
        var inputGender = document.getElementById("inputGender").value;
        inputGender = inputGender == "Male" ? "男" : "女";
        var table = document.getElementsByTagName("table")[0];


        table.innerHTML += "    <tr>\n" +
            "        <td>" + inputID + "</td>\n" +
            "        <td>" + inputName + "</td>\n" +
            "        <td>" + inputGender + "</td>\n" +
            "        <td><a href=\"javascript:void(0)\" onclick=\"del(this)\">删除</td>\n" +
            "    </tr>";
    }

    var del = function(obj){
        var tr = obj.parentNode.parentNode;
        var table = tr.parentNode;
        table.removeChild(tr);
    }
</script>
~~~

## 6. JQuery
- JQuery 是一个JS 框架，简化JS 的开发
- 1.x版本，兼容ie678，使用最广泛，最终版1.12.4
- 2.x版本，不兼容ie678，很少使用，最终版本2.2.4
- 3.x版本，不兼容ie678，老的JQuery插件不支持该版本

### 6.1 JQuery和JS的区别和转换
- 获取多个元素都可以当作数组操作
- 方法不通用
- js转jq： `$(js对象)`
- jq转js： `jq对象[索引]` 或者 `jq对象.get(索引)`

~~~html
<body>

<div id="div1" class="div_class">div1....</div>
<div id="div2" class="div_class">div2....</div>

<script>
    var divs = document.getElementsByTagName("div");
    for (var i = 0; i < divs.length; i++) {
        divs[i].innerHTML = "aaa";
    }
    alert(divs.length);
    var $divs = $("div");
    $divs.html("bbb");
    alert($divs.length);

    $(divs)            //jq转js   
    $divs[0]           //js转jq   
    $divs.get(0)       //js转jq   
</script>
</body>
~~~

### 6.2 JQuery选择器        

#### 1. 基本选择器
  1. 标签选择器（元素选择器）
		*  $("html标签名") 获得所有匹配标签名称的元素
  1. id选择器 
		*  $("#id的属性值") 获得与指定id属性值匹配的元素
  1. 类选择器
		*  $(".class的属性值") 获得与指定的class属性值匹配的元素
  1. 并集选择器：
		*  $("选择器1,选择器2....") 获取多个选择器选中的所有元素
#### 2. 层级选择器
  1. 后代选择器
		*  $("A B ") 选择A元素内部的所有B元素		
  1. 子选择器
		*  $("A > B") 选择A元素内部的所有B子元素
#### 3. 属性选择器
  1. 属性名称选择器 
		*  $("A[属性名]") 包含指定属性的选择器
  1. 属性选择器
		*  $("A[属性名='值']") 包含指定属性等于指定值的选择器
  1. 复合属性选择器
		*  $("A[属性名='值'][]...") 包含多个属性条件的选择器
#### 4. 过滤选择器
  1. 首元素选择器 
		*  :first 获得选择的元素中的第一个元素
  1. 尾元素选择器 
		*  :last 获得选择的元素中的最后一个元素
  1. 非元素选择器
		*  :not(selector) 不包括指定内容的元素
  1. 偶数选择器
		*  :even 偶数，从 0 开始计数
  1. 奇数选择器
		*  :odd 奇数，从 0 开始计数
  1. 等于索引选择器
		*  :eq(index) 指定索引元素
  1. 大于索引选择器 
		*  :gt(index) 大于指定索引元素
  1. 小于索引选择器 
		*  :lt(index) 小于指定索引元素
  1. 标题选择器
		*  :header 获得标题（h1~h6）元素，固定写法
#### 5. 表单过滤选择器
  1. 可用元素选择器 
		*  :enabled 获得可用元素
  1. 不可用元素选择器 
		*  :disabled 获得不可用元素
  1. 选中选择器 
		*  :checked 获得单选/复选框选中的元素
  1. 选中选择器 
		*  :selected 获得下拉框选中的元素



## 7. AJAX  
- ASynchronous JavaScript And XML 异步的JS和XML




## 8. JSON
- JavaScript Object Notation  JS对象标识法
- json用于存储和交换文本信息，进行数据的传输，比xml更小更快，更易解析

### 8.1 语法规则        
- 数据由键值对构成：键一般用引号引起来(单双皆可)，也可以不加引号。值可以是**数字、字符串(引号)、逻辑值、数组、json对象、null**    
- 键值对由逗号分割
- 花括号保存对象
- 方括号保存数组            

如：
~~~json
{
    "persons":[
        {"name":"张三", "age":13}, 
        {"name":"李四","age":14}
        ]
}
~~~

### 8.2 获取json的数据      
- 获取键的值：通过 `对象.键名` 或者 `对象["键名"]`
- 获取数组元素：`对象[索引]`
~~~js
var p = {
    "persons":[
        {"name":"张三", "age":13}, 
        {"name":"李四","age":14}
        ]
}

var name = p.persons[0].name;           //张三
var name = p["persons"][0]["name"];     //张三
~~~

- json的遍历：通过 `key in person` 获得的 key 是 **"字符串"**，所以不能用 person.key 来获取值

~~~js
for(var i=0; i<p.persons.length; i++){
    var person = p.persons[i];
    for(var key in person){
        alert(key + "--->" + person[key]);
    }
}
~~~

### 8.3 json数据和java对象的转换

#### 8.3.1 java对象转json
- json解析器：Jsonlib、Gson、fastjson、**jackson**

- 核心对象：`ObjectMapper`
- `writeValue(File, Object)` ： 将objct对象转json字符串，并保存到指定的文件中
- `writeValue(Writer, Object)` ： object转换为json字符串，并将json数据填充到字符输出流中      
- `writeValue(OutputStream, Object)` ： object转换为json字符串，并将json数据填充到字节输出流中
- `writeValueAsString(Object)` ： 将object对象转换为json字符串
- 在类中的成员上加注解 `@JsonIgnore` 可以在转换时忽略该成员
- 在类中的成员上加注解 `@JsonFormat(pattern="yyyy-MM-dd")` 实现格式化
~~~java
    public void test1() throws JsonProcessingException {
        Person p = new Person();
        p.setName("张三");
        p.setAge(13);
        p.setAddress("深圳");

        //创建json的核心对象
        ObjectMapper mapper = new ObjectMapper();
        String json = mapper.writeValueAsString(p);
        System.out.println(json);       //{"name":"张三","age":13,"address":"深圳"}

        mapper.writeValue(new File("D:\\a.txt"),p);
    }

    //list转json：转为数组格式的字符串
    public void test2() throws JsonProcessingException {
        Person p1 = new Person("张三",13,"深圳");
        Person p2 = new Person("李四",14,"深圳");
        List<Person> list = new ArrayList<>();
        list.add(p1);
        list.add(p2);
        String json = new ObjectMapper().writeValueAsString(list);
        System.out.println(json);   //[{"name":"张三","age":13,"address":"深圳"},{"name":"李四","age":14,"address":"深圳"}]
    }

    //map转json：和对象格式一致
    public void test3() throws JsonProcessingException {
        Map<String, Object> map = new HashMap<>();
        map.put("name", "张三");
        map.put("age", 13);
        map.put("address", "深圳");
        String json = new ObjectMapper().writeValueAsString(map);
        System.out.println(json);   //{"address":"深圳","name":"张三","age":13}
    }
~~~

#### 8.3.2 json转java对象
- `readValue(String, Class<T>)`       
- `readValue(File, Class<T>)`         
- `readValue(Reader, Class<T>)`           
- `readValue(InoutStream, Class<T>)`      

~~~java
    public void test4() throws IOException {
        String json = "{\"address\":\"深圳\",\"name\":\"张三\",\"age\":13}";
        Person person = new ObjectMapper().readValue(json, Person.class);
        System.out.println(person);
    }
~~~

### 8.4 response设置相应json格式

- json格式的字符串响应时设置 mime 类型

~~~java
response.setContextType("application/json;charset=utf-8")
~~~








# BootStrap             
前端开发框架，基于HTML、CSS、JS，定义了很多的 CSS 样式和 JS 插件        
响应式布局：同一页面，根据分辨率自动调整，兼容不同分辨率的设备      

## 1. Quick Start
~~~html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <!-- 上述3个meta标签*必须*放在最前面，任何其他内容都*必须*跟随其后！ -->
    <title>Bootstrap 101 Template</title>

    <!-- Bootstrap -->
    <link href="css/bootstrap.min.css" rel="stylesheet">


    <!-- jQuery (Bootstrap 的所有 JavaScript 插件都依赖 jQuery，所以必须放在前边) -->
    <script src="js/jquery-3.5.1.min.js"></script>
    <!-- 加载 Bootstrap 的所有 JavaScript 插件。你也可以根据需要只加载单个插件。 -->
    <script src="js/bootstrap.min.js"></script>
</head>
<body>
<h1>你好，世界！</h1>

</body>
</html>
~~~
## 2. 响应式布局：网格系统(Grid)

- Bootstrap 包含了一个响应式的、移动设备优先的、不固定的网格系统，可以随着设备或视口大小的增加而适当地扩展到 12 列      

**网格工作原理**    
- 定义容器：container(有留白的宽度)，container-fluid(占满屏幕)
- 行 row 必须放置在 container 内，以便获得适当的对齐（alignment）和内边距（padding）
- 元素(内容)应该放置在列内，且唯有列可以是行的直接子元素        
- 指定元素在不同设备上所占的网格数目： col-设备代号-占格子数目

设备代号：
- xs :　超小屏幕 手机(<768px>) ： col-xs-12
- sm ： 小屏幕  平板(768px起)
- md ： 中型设备  台式电脑(992px起)
- lg ： 大型设备  大台式电脑(1200px起)

注意：
- 网格类适用于宽度大于等于分界点大小的设备，更大尺寸屏幕上没有定义则按最接近的小尺寸的定义显示
- 设备尺寸小于定义中的最小尺寸时，默认会一个元素占满一整行          
~~~html
<!--定义容器-->
<div class="container">
    <!--定义行-->
    <div class="row">
        <!--定义元素，大屏幕上每个元素占一格(一行12个元素)，小屏幕每个占2格(一行6个)，超小屏幕每个占4格(一行3个)，超过自动换行-->
        <div class="col-lg-1 col-sm-2 col-xs-4 inner">栅格</div>
        <div class="col-lg-1 col-sm-2 col-xs-4 inner">栅格</div>
        <div class="col-lg-1 col-sm-2 col-xs-4 inner">栅格</div>
        <div class="col-lg-1 col-sm-2 col-xs-4 inner">栅格</div>
        <div class="col-lg-1 col-sm-2 col-xs-4 inner">栅格</div>
        <div class="col-lg-1 col-sm-2 col-xs-4 inner">栅格</div>
        <div class="col-lg-1 col-sm-2 col-xs-4 inner">栅格</div>
        <div class="col-lg-1 col-sm-2 col-xs-4 inner">栅格</div>
        <div class="col-lg-1 col-sm-2 col-xs-4 inner">栅格</div>
        <div class="col-lg-1 col-sm-2 col-xs-4 inner">栅格</div>
        <div class="col-lg-1 col-sm-2 col-xs-4 inner">栅格</div>
        <div class="col-lg-1 col-sm-2 col-xs-4 inner">栅格</div>
    </div>
</div>
~~~

## 3. CSS样式   
**按钮 :** 
- `class="btn btn-default"`

**图片 :** 
- `class="img-responsive"` 图片在任意尺寸都占100%
- `class="img-rounded"` rounded：带圆角的方形  circle：圆的  thumbnail：相框        

**表格**
- `table-striped` 条纹状表格
- `table-bordered` 带边框的表格
- `table-hover` 鼠标悬停

**表单**
- 单独的表单控件会被自动赋予一些全局样式。所有设置了 .form-control 类的 `<input>`、`<textarea>` 和 `<select>` 元素都将被默认设置宽度属性为 width: 100%;。 将 label 元素和前面提到的控件包裹在 .form-group 中可以获得最好的排列



## 4. 组件  
**导航条**


**分页条**

## 5. 插件
**轮播图**


# XML       
- Extensible Markup Language ： 可扩展标记语言      
- 可扩展 ：自定义标签       
- 主要用作存储数据，作配置文件，在网络中传输，跨平台

## 1. 基本语法
- 第一行必须定义为文档声明 `<?xml version="1.0"?>`，空行也不能有
- 有且仅有一个根标签
- 标签必须闭合，有结束标签(也可以定义自闭合的)
- 属性值必须使用引号(单双都可以)
- 标签名称区分大小写

## 2. 组成部分
### 2.1 文档声明
- 格式：`<?xml 属性列表 ?>`
- 属性 `version`：版本号，必须的属性，1.0
- 编码 `encoding` : 默认ISO-8859-1 
- 独立 `standalone` : 是否依赖其他文件，一般不用
- `<?xml version='1.0' encoding='utf-8'?>`
### 2.2 指令
- 结合CSS，一般不用 `<?xml-stylesheet type="text/css" href="a.css"?>`
### 2.3 标签
- 自定义标签的命名规则：可以是字母数字及其他字符、不能以数字或符号开头、不能以xml开头、不能包含空格
### 2.4 属性
- id 属性值唯一
### 2.5 文本
- CDATA区：在该区域的文本会被原样展示，无需转义     

~~~xml
<![CDATA[xxxxxxxxxxxxx]>
~~~

### 2.6 约束 
- 软件提供的约束文档规定了xml文档的书写规则      
- DTD ： 简单约束
- Schema ： 复杂约束，可以限定属性的值  
- DTD文档的引入：.dtd                 
    内部DTD(不常用)  `<!DOCTYPE 根标签 [dtd约束]>`          
    外部本地DTD  `<!DOCTYPE 根标签名 SYSTEM "dtd文件位置">`          
    外部网络DTD  `<!DOCTYPE 根标签名 PUBLIC "dtd文件名字" "dtd文件位置URL">`

- Schema文档的引入： .xsd  

## 3. xml 解析

### 3.1 解析方式        
- DOM ：将标记语言文档一次性加载进内存，在内存中形成一颗DOM树。方便查找和修改，但是内存占用大，一般用于服务器端    
- SAX ：逐行读取，基于事件驱动。不占内存，但是只能读取，不能增删改，一般用作移动端    

### 3.2 常用解析器  
- JAXP ： sun 提供的解析器，支持DOM 和 SAX，性能不好
- DOM4J ： 优秀的解析器，常用     
- Jsoup ： Java 的 HTML 解析器，提供强力API，可通过DOM，CSS以及类似jQuery的操作方法来取出和操作数据    
- PULL ： Android 操作系统内置的解析器，按 SAX 方式解析 

### 3.3 Jsoup_QuickStart   
步骤：
- 导入jar包
- 获取 Document 对象
- 获取对应的标签 Element 对象
- 获取数据

~~~java
public class JsoupDemo1 {
    public static void main(String[] args) throws IOException {
        //获取 xml 的 path，根据 path 获取 Document 对象
        String path = JsoupDemo1.class.getClassLoader().getResource("xml/jsoup/student.xml").getPath();

        //解析xml文档，加载文档进内存，获取dom树
        Document document = Jsoup.parse(new File(path), "utf-8");

        //获取元素对象
        Elements elements = document.getElementsByTag("name");

        elements.forEach(element -> {
            System.out.println(element.text());
        });
    }
}
~~~

### 3.4 Jsoup_Jsoup 对象
- Jsoup 是一个工具类，可以解析 html 和xml ，返回Document
- `static Document parse(File in, String charsetName)` : 解析xml、html文件
- `static Connection connect(String url)` ： 解析String 格式的 xml、html    
- `static Document parse(URL url, int timeoutMillis)` : 解析网络路径获得的html、xml     

~~~java
public class JsoupDemo2 {
    public static void main(String[] args) throws IOException {
        URL url  = new URL("https://www.baidu.com/");
        Document document = Jsoup.parse(url, 3000);
        //System.out.println(document);
        Elements elements = document.getElementsByClass("mnav");
        System.out.println(elements.size());
        elements.forEach(element -> {
            System.out.println(element.text());
        });
    }
}
~~~

### 3.5 Jsoup_Document 对象
- Document 继承自 Element
- `Elements getElementsByTag(String tagName)`
- `Element getElementById(String id)`
- `Elements getElementsByClass(String className)`
- `Elements getElementsByAttribute(String key)`
- `Elements getElementsByAttributeValue(String key, String value)` 根据属性名和属性值获得对象  

### 3.6 Jsoup_Elements 和 Element对象
- Elements 是 Element 对象的集合，继承自 ArrayList
- `String attr(String key)` 根据属性名称获取属性值
- `String text()` 获取文本内容   
- `String html()` 获取标签体的所有内容(包括子标签的标签和文本)
- Node 节点对象 是Element 的父类

## 3.7 Jsoup 快捷查询
- selector : 选择器， `Elements select(String cssQuery)`

![d9ZyOx.png](https://s1.ax1x.com/2020/08/13/d9ZyOx.png)
![d9nnw6.png](https://s1.ax1x.com/2020/08/13/d9nnw6.png)
~~~java
public class JsoupDemo3 {
    public static void main(String[] args) throws IOException {
        String path = JsoupDemo3.class.getClassLoader().getResource("xml/jsoup/student.xml").getPath();

        Document document = Jsoup.parse(new File(path),"utf-8");

        Elements elements = document.select("student[number='0001'] age");

        System.out.println(elements);
    }
}

~~~



