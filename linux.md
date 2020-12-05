# 计算机基础
- 冯诺依曼结构：运算器、控制器、存储器、输入设备、输出设备
## 1. CPU    
- 精简指令集RISC(Reduced Instruction Set Computer)：指令集精简，单个指令执行时间短，指令执行效能较佳，复杂功能需要右多个指令完成。如ARM
- 复杂指令集CISC(Complex Instruction Set Computer)：单个指令可以执行一些低阶的硬件操作，指令数目多且复杂，每条指令长度不同，单个指令执行时间较长。如x86(起源于8086)
- 位：cpu一次读取数据的位数
- 多核：一个cpu中封装了多个运算核心
- 字组大小(word size)：cpu每次处理的数据量，也就是我们称的位数，32，64
- 北桥称为系统总线，是内存传输的主要信道，因此速度比较快；南桥称为IO总线，主要联系硬盘、USB、网卡等设备。

## 2. 内存
- 带宽：以频率1660MHz的内存为例，每次传输数据量为64位，则一次可以从内存中取得的带宽为：1660MHz*64bit = 12.8GB/s
- DRAM(Dynamic Random Access Memory)：需要定时刷新，记忆短。一个bit只需要一个电容和一个晶体管，数据存储在电容中，所以需要定刷新，速度比SRAM慢，但造价便宜很多，主要用作计算机的内存
- SRAM(Static Random Access Memory)：只要保持供电，数据可以持久保持。速度快，整合到cpu成为高速缓存(L2)。但是一个bit需要6个晶体管保存，造假贵，一般就几个MB大小，做高速缓存
- ROM(Read Only Memory)：非挥发性内存

## 3. 操作系统概念
- 操作系统的核心层是直接参考硬件规格携程的，所以统一操作系统程序不能再不一样的硬件架构下运作
- 操作系统只管理硬件资源：CPU、内存、输入输出设备及文件系统文件
- 应用程序的开发都是参考操作系统提供的开发接口，所以针对某一操作系统开发的程序只能在该操作系统上运行

### 核心的功能  
- 系统呼叫接口(System Call Interface)：方便程序开发者与核心沟通而提供的接口
- 程序管理：控制多个程序的执行顺序，cpu的资源分配
- 内存管理：
- 文件系统管理：数据的输入输出、不同文件格式的支持
- 装置的驱动(Device drivers)：硬件厂商按照操作系统的接口开发驱动程序，由操作系统来控制


# Linux
## 1. 文件与目录
### 1.1 文件权限  
dr-xr-xr-x.   5 root root 4096 Feb 18  2020 boot           
- 第一个字符代表文件类型，7种：其中 b c s p 是伪文件，不占用磁盘空间  
  - d : 目录
  - \- : 文件，分为纯文本文档(ASCII)、二进制文件(binary)、数据格式文件(data)
  - l : link file，相当于快捷方式
  - b : block 区块设备文件，装置文件(device)里面的可供储存的接口设备(可随机存取装置，如硬盘、软盘)       
  - c : character 字符设备文件，装置文件里面的串行端口设备，例如键盘、鼠标(一次性读取装置)
  - s : sockets 套接字文件
  - p : pipe 管道文件
- 接下来的字符三个一组，分别表示拥有者权限、群组权限、其他人权限


|权限|<center>对文件的作用</center>|
|:---:|---|
|r|可读取文件中的实际内容，可对文件执行cat、more、less、head、tail等命令查看|
|w|可编辑、新增、修改文件内容，可使用vim、echo等修改，但不能删除文件|
|x|文件具有被系统执行的权限，文件能否执行由x权限决定，跟后缀名没有必然的关系|

|权限|<center>对文件夹的作用</center>|
|:---:|---|
|r|可读取目录结构列表的权限，可以使用ls命令查看目录中的内容|
|w|可以在此目录下新建、删除、移动、更名文件和文件夹，可使用touch、rm、cp、mv等命令|
|x|可以进入目录，也就是说可以使用cd命令，没有x权限的目录无法进入|
  - rwx分别为 4 2 1分
  - 操作所需最低权限：**读取文件内容 r**、**修改文件内容 rw**、**执行文件 rx**、**删除文件，需要文件所在目录具有 wx**
- 第二栏表示表示有多少名连接到此节点
- 第三栏是文件的拥有者账号
- 第四栏是所属群组
- 第五栏是文件的容量大小，默认单位是Bytes
- 第六栏是最近修改时间，时间太久会只显示年，要显示详细时间可以用`ls -l --full-time`
- 最后一栏就是文件的名字了，带 . 表示隐藏文件

### 1.2 权限变更
- `chgrp` 改变所属群组，可选参数 -R 递归变更
- `chown` 改变拥有者(所属用户)
- `chmod` 改变文件的权限

~~~
把群组改为users
[root@study ~]# chgrp users initial-setup-ks.cfg   
[root@study ~]# ls -l
-rw-r--r--. 1 root users 1864 May 4 18:01 initial-setup-ks.cfg

把拥有者改为bin
[root@study ~]# chown bin initial-setup-ks.cfg     
[root@study ~]# ls -l
-rw-r--r--. 1 bin users 1864 May 4 18:01 initial-setup-ks.cfg

将 initial-setup-ks.cfg 的拥有者与群组改回为 root：chown [-R] 账号名称:组名 文件或目录
[root@study ~]# chown root:root initial-setup-ks.cfg
[root@study ~]# ls -l
-rw-r--r--. 1 root root 1864 May 4 18:01 initial-setup-ks.cfg
~~~

- 修改权限命令：`chmod 777 .bashrc` 即改为-rwxrwxrwx；而-rwxr-xr-x 就是 755，其他人只能读不能写
- 还可以使用 u、g、o、a（a代表all） 和 +、-、= 来设定，如 `chmod u=rwx,go=rx .bashrc`就设定了一个755
- 只有r没有x权限的目录，仅能查询到目录下的文件名列表，其他信息均不可见，一般也不会这么设置，没什么意义
- 有意义的三种**目录权限**：**0(---)， 5(r-x)， 7(rwx)**，**最常用755**，严谨一点可以**750或者700**
- 对**普通文件**而言，**最常用644**， 严谨一点可以**640或者600**
- 对**可执行文件**而言，**最常用754或755**，严谨一点可以**750，740，700**
- 文件所有者必须为6或者7，不然就傻逼了
~~~

[dmtsai@study tmp]$ ls -l testing/
ls: cannot access testing/testing: Permission denied
total 0
?????????? ? ? ? ? ? testing
[dmtsai@study tmp]$ cd testing/
-bash: cd: testing/: Permission denied
~~~

### 1.3 文件名和文件扩展名
- 文件是否具有可执行能力由x权限决定，但是能不能执行成功就得看文件的实际内容了。于是以适当的扩展名表示文件的种类：
- .sh ： 脚本或者批处理文件
- Z, .tar, .tar.gz, .zip, .tgz : 打包和压缩文件
- .html, .php : 网页相关的文件
- linux文件名限制为255Bytes

### 1.4 目录结构
- FHS要求必须存在的目录

|目录|放置文件内容|
|---|---|
|/bin|可执行文件，bin下的指令可以被root和一般账号使用，主要有cat,chmod,chown,data,mv,mkdir,cp,bash等|
|/boot|包含linux核心文件和开机所需配置文件等|
|/dev|装置和接口设备以文件的形态存在于该目录中，比较重要的有/dev/null, /dev/zero, /dev/tty,/dev/loop*, /dev/sd*等等|
|/etc|系统主要的配置文件，例如账号密码文件等，另外FHS规范了几个重要的目录存放在/etc/目录下：<br> /etc/opt(必要)：第三方软件/opt 的相关配置文件|
|/lib|开机时会用到的函数库，以及在/bin或/sbin下指令会呼叫的函数库|
|/media|可移除的媒体装置，软盘光盘dvd等|
|/mnt|临时挂载额外装置的目录|
|/opt|第三方软件放置目录|
|/run|开机后产生的各项信息|
|/sbin|开机、修复、还原系统所需要的指令，只有root用户才能使用|
|/srv|service，网络服务启动后所需要的数据目录|
|/tmp|临时文件，定期清理|

### 1.5 环境变量
- 查看环境变量：`echo $PATH`
- 新增环境变量：`PATH="${PATH}:/root"` 将/root加入到环境变量中
- 配置环境变量：`vi /etc/profile`，修改完之后需要重新加载：`source /etc/profile`
~~~
配置java的环境变量
JAVA_HOME=/usr/local/jdk/jdk1.8.0_261
CLASSPATH=.:$JAVA_HOME/lib.tools.jar
PATH=$JAVA_HOME/bin:$PATH
export JAVA_HOME CLASSPATH PATH 
~~~


### 1.6 文件变动的时间
- modification time **(mtime)：** 当该文件的『内容数据』变更时，就会更新这个时间！内容数据指的是文件的内容，而不是文件的属性或权限
- status time (**ctime**)：当该文件的『状态 (status)』改变时，就会更新这个时间，举例来说，像是权限与属性被更改了，都会更新这个时间
- access time (**atime)**：当『该文件的内容被取用』时，就会更新这个读取时间 (access)。举例来说，我们使用 cat 去读取/etc/man_db.conf ， 就会更新该文件的 atime 了
- 默认情况下显示的是mtime
- 完全复制`cp -a`，仅仅复制了mtime和atime，而ctime则是新建文件的当前时间 
~~~
[root@study ~]# date; ls -l /etc/man_db.conf ; ls -l --time=atime /etc/man_db.conf ; \
> ls -l --time=ctime /etc/man_db.conf # 这两行其实是同一行喔！用分号隔开
Tue Jun 16 00:43:17 CST 2015 # 目前的时间
-rw-r--r--. 1 root root 5171 Jun 10 2014 /etc/man_db.conf # 在 2014/06/10 建立的内容(mtime)
-rw-r--r--. 1 root root 5171 Jun 15 23:46 /etc/man_db.conf # 在 2015/06/15 读取过内容(atime)
-rw-r--r--. 1 root root 5171 May 4 17:54 /etc/man_db.conf # 在 2015/05/04 更新过状态(ctime)
~~~

### 1.7 文件预设权限：umask
- 目前用户在建立文件或目录的时候权限的默认值
- 建立文件的默认最高权限：-rw-rw-rw- 即666
- 建立文件夹默认最高权限：drwxrwxrwx 即777
- umask表示减去一定的权限，如：023表示建立的文件为644，文件夹为754

~~~
[root@iZwz91j9t2admw7xtplq6bZ ~]# umask 002
[root@iZwz91j9t2admw7xtplq6bZ ~]# touch test_file
[root@iZwz91j9t2admw7xtplq6bZ ~]# mkdir test_dir
[root@iZwz91j9t2admw7xtplq6bZ ~]# ll -d test*
drwxrwxr-x 2 root root 6 Sep  7 16:29 test_dir
-rw-rw-r-- 1 root root 0 Sep  7 16:29 test_file
~~~

### 1.8 常见打包和压缩格式
- `*.Z`     compress 程序压缩的文件(过时)；
- `*.zip`   zip 程序压缩的文件；
- `*.gz`    gzip 程序压缩的文件(取代compress)；
- `*.bz2`   bzip2 程序压缩的文件(取代gzip)；
- `*.xz`    xz 程序压缩的文件(更优秀的压缩比)；
- `*.tar`   tar 程序打包的数据，并没有压缩过；
- `*.tar.gz` tar 程序打包的文件，其中并且经过 gzip 的压缩
- `*.tar.bz2` tar 程序打包的文件，其中并且经过 bzip2 的压缩
- `*.tar.xz` tar 程序打包的文件，其中并且经过 xz 的压缩

## 2. vim
- 三种模式：一般命令模式、编辑模式、底行命令模式
### 2.1 一般命令模式
- 移动光标

|命令|含义|
|---|---|
|移动光标     |h←， j↓， k↑， l→，可以使用nj nk
|翻页         |Ctrl+f 向下翻页   Ctrl+b 向上翻页
|切行         |+ 移动到非空格的下一行， - 移动到非空格的上一行
|n\<space>    |如20 空格， 光标向后移动20个字符
|0或Home      |移动到列首
|$或End       |移动到列尾
|G            |移动到最后一列
|gg           |移动到第一列
|n\<Enter>    |光标向下移动n行

- 查找与替换

|命令|含义|
|---|---|
|/word|向光标之下寻找名称为word的字符串|
|?word|向上查找|
|:n1,n2s/word1/word2/g|在n1和n2行之间查找word1并替换为word2|
|:n1,n2s/word1/word2/gc|在n1和n2行之间查找word1并替换为word2，并提示确认|

- 删除与复制粘贴

|命令|含义|
|---|---|
|x,X|x删除后继字符，相当于del，X删除前一个字符，相当于backspace|
|dd|删除一行|
|ndd|向下删除n行|
|yy|复制游标所在行|
|nyy|向下复制n行|
|pP|p 粘贴到光标所在列的下一列， P粘贴到光标所在列的上一列|
|u|复原前一个动作|
|Ctrl+r|重做上一个动作，与u配合使用。。即向前向后撤销|
|.|重复上一个动作|

### 2.2 编辑模式
|命令|含义|
|---|---|
|i,I|进入插入模式，从光标所在处插入，I为从非空格字符处插入|
|a,A|进入插入模式，从光标所在处的下一个字符开始插入，A为在该列尾插入|
|o,O|进入插入模式，从光标所在列的下一列插入新列，O为在上一列插入新列|
|r,R|进入取代模式，r只取代光标所在的字符一次，R会一直取代光标所在字符直到ESC|

### 2.3 底行命令模式
|命令|含义|
|---|---|
|:w|保存到硬盘|
|:w!|若文件为只读时，强制写入。不过能不能写入还要看文件权限|
|:q|离开vi|
|:q!|不保存，强制离开|
|:wq|存储后离开|
|ZZ|若文件未更改则不存储离开，已更改则存储后离开|
|:w filename|另存为|

## 3. Linux上软件安装
详见 [Linux系统中安装软件的几种方式:https://blog.csdn.net/qq_36119192/article/details/82866329](https://blog.csdn.net/qq_36119192/article/details/82866329)
1. 二进制发布包：软件已经针对具体平台编译打包发布，只需要解压修改配置即可，但是个平台不兼容
2. RPM方式：软件按redhat的包管理工具规范RPM进行打包发布，需要获取到相应软件RPM发布包，然后用RPM命令安装，该方式不会为软件安装所需的依赖包
3. Yum在线安装：软件以RPM规范打包，发布在网络服务器上，可用yum在线安装服务器上的rpm软件，并且自动解决安装过程中的库依赖问题
4. 源码编译安装：软件以源码方式发布，需要获取到源码工程后用相应的开发工具进行编译打包部署

### 3.1 常用rpm命令
|命令|含义|
|---|---|
|rpm -qa|查询所有已安装软件的rpm包信息，列出包的版本|
|rpm  -ivh  包.rpm|i表示安装，v表示显示安装过程，h表示以‘#’作为进度，显示安装进度|
|rpm -e --nodeps 包名|卸载|


### 3.2 常用yum命令
|指令|含义|
|---|---|
|yum  clean all         |          清空缓存信息
|yum  list              |          列出所有包的信息
|yum  list  httpd       |          查看 httpd 是否安装
|yum  info httpd        |          显示 httpd 包的详细具体信息
|yum install httpd   -y |          安装 httpd 包
|yum remove httpd  -y   |          卸载 httpd 包
|yum search 关键词         |      根据关键词，在已发现的repo源中搜索包含关键词的rpm包
|yum provides 命令        |         根据命令，在已发现的repo源中搜索安装指令的rpm包
|yum history  list/info/undo/redo number|   history可以列出，查看，重装，反安装对应的包，但是是以yum指令的操作顺序为依据的，所以需要加指定的数字执行
|yum update -y    |                    升级所有包同时也升级软件和系统内核
|yum upgrade  -y  |                   只升级所有包，不升级软件和系统内核

### 3.3 yum安装mysql
- `yum list|grep mysql`
- `yum install mysql-server`
- `service mysqld start` 注意，启动mysql服务器是用mysqld
- `mysql -uroot` 登录，初始没有密码
- 设置远程访问权限
~~~sql
mysql> CREATE USER 'root'@'%' IDENTIFIED BY 'root';
mysql> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;
mysql> flush privileges;
~~~
- aliyun主机中记得手动设置开放的端口，安全组中增加3306端口

### 3.4 源码安装redis
- 安装gcc-c++：`yum install gcc-c++`
- 下载解压redis
- cd 到 redis-6.0.7 执行 make 命令
- 安装：`make PREFIX=/usr/local/redis install`
- 复制redis-6.0.7下的 redis.conf 配置文件到 bin 目录：`[root@iZw bin]# cp ../redis-6.0.7/redis.conf ./`
- 在bin中启动redis服务端：`./redis-server redis.conf`

## 指令大全
### 1. 目录和文件
|指令|含义|
|---|---|
|cd         |cd ~ 回到用户home目录(root用户回/root)<br>cd - 回到上一次访问的目录
|pwd [-P]   |显示当前目录<br> pwd -P 显示确切路径，而非link路径
|mkdir [-mp]|建立新的空目录<br>-m 配置文件权限：mkdir -m 755 test<br>-p 递归建立所需上层目录：mkdir -p test1/test2/test3/test4
|rmdir [-p] |删除一个**空目录**<br>-p 将上层空目录一起删除：rmdir -p test1/test2/test3/test4|
|ls [-aAdfFhilnrRSt]|-a 显示全部文件，包括隐藏文件<br>-A 显示全部文件，包括隐藏文件，但不包括.和..这两个目录<br>-d 仅列出目录本身，不列出目录内的文件数据<br>-F 在列出的文件名称后加一符号；例如可执行档则加 "*", 目录则加 "/"<br>-h 将文件容量以人类较易读的方式(例如 GB, KB 等等)列出来<br>-i 列出 inode 号码<br>-l 除文件名称外，亦将文件型态、权限、拥有者、文件大小等资讯详细列出<br>-R 连同子目录内容一起列出来，等于该目录下的所有文件都会显示出来<br>-S 以文件容量大小排序，而不是用档名排序<br>-t 将文件依建立时间之先后次序列出<br>--full-time 以完整时间模式 (包含年、月、日、时、分) 输出|
|cp [-adfilprsu]|复制，还可以建立链接档，比对文件新旧进行更新，以及复制整个目录<br>-a 相当于-dr --preserve=all 完全复制，完全一样<br>-i 若目标文件已经存在，会在覆盖时先询问动作的进行<br>-p 连同文件的属性(权限、用户、时间)一起复制，常用于备份<br>-r 递归持续复制，用于目录的复制<br>复制到当前目录`cp /var/log/wtmp .`用.即可|
|mv [-fiu]|移动，也可用作更名<br>-f 强制，目标文件已经存在则直接覆盖，不询问<br>-i 目标文件已经存在时会进行询问<br>-u 目标文件已经存在，且source比目标文件新时才会更新(update)<br>更名：`mv mvtest mvtest2`|
|rm [-fir]|删除(无需文件夹为空)<br>-f 强制删除，忽略不存在的文件，不会出现警告信息<br>-i 删除前询问<br>-r 递归删除|
|basename|取得文件的最后文件名|
|dirname|取得文件的目录名|
|touch [-acdmt]|修改文件时间或建立新文件<br>-a 仅修改access time(atime)<br>-c 仅修改ctime，若文件不存在则新建文件<br>-d 后面可以接日期而不使用当前日期<br>-m 仅修改mtime<br>-t (同时修订mtime和atime为目标值，而ctime则是记录当前时间) 后面可以接日期而不使用当前日期，格式为YYYYMMDDhhmm|
|chgrp [-R]| 改变所属群组 -R 递归变更|
|chown| 改变拥有者(所属用户)|
|chmod| 改变文件的权限|
|file|查看文件类型|

### 2. 查看文件内容

|指令|含义|
|---|---|
|cat|从第一行开始显示文件内容<br>-A 显示特殊字符而不是空白，可以区分开空格和tab，显示结尾的断行字符$<br>-b 显示行号(仅对非空白行显示，空白行不标行号)<br>-n 显示行号，连同空白行也显示<br>-T 将tab以^I 显示 |
|tac|从最后一行开始，反过来显示文件内容|
|nl|显示内容的时候，一并显示行号，并对行号样式进行设计|
|more|一页一页显示文件内容<br>空格：向下翻一页<br>Enter：向下翻一行<br>/字符串：向下搜寻字符串这个关键字<br>q：退出<br>b ：往回翻页|
|less|与more类似，但是可以往前翻页！使用pageup和pagedown翻页<br>/字符串：向下搜寻<br>?字符串：向上搜寻|
|head [-n]|只看头几行<br>-n 显示前n行，默认显示前10行`head -n 20 /man_db.conf`，负数表示不包括后n行|
|tail [-n]|只看尾几行<br>查看man_db.conf的11-20行：`head -n 20 /man_db.conf | tail -n 10`|
|od [-t TYPE]|查看非文本文件<br>-t 后面可以跟上各种TYPE的输出：<br>-t a 使用默认字符输出<br>-t c 使用ASCII字符输出<br>-t d[size] 使用十进制输出数据，每个整数占用size个bytes<br>-t f[size]， -t o[size]， -t x[size]<br>找到password的ASCII对照：`echo password | od -t oCc`|

### 3. 文件搜索
|指令|含义|
|---|---|
|which [-a]|搜索指令脚本的完整文件名<br>-a 将所有PATH目录中可以找到的指令均列出，而不止第一个找到的指令|
|whereis [-bmsu]|搜索文件或目录名的完整名称，速度比find快，因为只搜索几个特定目录(-l查看)<br>-l 查看whereis搜索的目录列表<br>-b 只搜索binary文件<br>-m 只找在man路径下的文件<br>-s 只找source来源文件<br>-u 搜索不再在述三个项目当中的其他特殊文件|
|locate [-ir] keyword|根据keyword查找文件和目录的完整名称<br>-i 忽略大小写<br>-c 仅计算查找到的文件数量，不输出文件名<br>-r 后面可以接正则表达式|
|find [PATH] [option] [action]|`find /home -user dmtsai` 搜寻 /home 底下属于 dmtsai用户的文件<br>`find / -name passwd` 全盘搜索名为passwd的文件<br>`find / -name "*passwd*"` 搜索包含passwd关键字的文件|
|grep [option] pattern [file]|在file中使用pattern进行正则匹配搜索内容<br>-i 不区分大小写<br>-l 只列出匹配的文件名<br>-w 只进行完整匹配|

### 4. 文件压缩
|指令|含义|
|---|---|
|gzip [-cdtv#]|gzip压缩，取代了compress，**默认情况下压缩后源文件不再存在**<br>-c 把压缩后的文件输出到标准输出设备，保留原始文件`gzip -c aaa > aaa.gz`<br>-d 解压缩，会删除原.gz文件<br>-v 显示压缩比等信息<br>-# #是数字，压缩等级，1最快，9最慢但压缩比最好，默认6|
|zcat、zmore、zless|查看.gz文件的源文件内容(文本文件)|
|bzip2 [-cdkzv#]|bzip压缩，比gzip压缩比更好，用法几乎相同<br>-d 解压缩<br>-k 保留原始文件|
|bzcat|查看.bz2文件的文本文件内容|
|tar [-zjJ][cv][-f name] file|打包指令<br>-z 通过gzip的支持进行压缩/解压缩，文件名*.tar.gz<br>-j 通过bzip2进行压缩/解压缩，文件名*.tar.bz2<br>-J 通过xz进行压缩/解压缩<br>-c 建立打包文件<br>-v 在压缩/解压缩时显示正在处理的文件名<br>-f 被处理的文件名(将要建立的压缩文件名或将要解压缩的文件名)<br>-p 保留备份数据的原本权限和属性，常用于备份重要的配置文件<br>-P 保留绝对路径，即允许备份数据中含有根目录存在<br>--exclude=xx/xx/xx 打包时排除某些文件<br>常用压缩：`tar -jcvf *.tar.bz2 文件名`|
|tar [-zjJ][tv][-f name]|查看压缩文件<br>-t 查看打包文件的内容，包含哪些文件名<br>常用查询：`tar -jtvf *.tar.bz2`|
|tar [-zjJ][xv][-f name][-C 目录]|解压缩<br>-x 解压缩或解打包<br>-C 目录：解压缩到指定的目录<br>常用解压缩：`tar -jxvf *.tar.bz2 -C 解压目录`|
- 解压缩单一文件的方法：
![解压缩单一文件.png](https://s1.ax1x.com/2020/09/07/wKPaKH.png)

### 5. 进程管理
|命令|含义|
|---|---|
|ps -ef|查看所有进程|
|ps -ef \| grep name|查找name名的进程|
|kill 2869|结束2869编号的进程|
|kill -9 2869|强制结束2869进程|

### 6. 网络操作
|命令|含义|
|---|---|
|hostname|查看主机名，也可以修改主机名(临时修改，重启失效)<br>永久生效需要修改/etc/sysconfig/network文件中的HOSTNAME字段|
|ipconfig|查看ip地址，也可以修改主机ip地址(临时修改)<br>永久修改：/etc/sysconfig/network-scripts/ifcfg-eth0 文件|
|域名映射|/etc/hosts|
|service network status|查看指定服务的状态|
|service network stop|停止指定服务|
|service network start|启动指定服务|
|service network restart|重启指定服务|
|service --status-all|查看系统中所有后台服务|
|netstat -nltp|查看系统中网络进程的端口监听情况|
|service iptables status/start/stop/off|防火墙状态/启动/关闭/禁止启动<br>注意：CentOS7,RHEL7,Fedora中防火墙由firewalld来管理<br>停止firewalld：`systemctl stop firewalld`<br>禁用firewalld：`systemctl mask firewalld`<br>开启firewalld：`systemctl unmask firewalld`<br>安装iptables：`yum install iptables-services`|

### 7. yum安装软件

|命令|含义|
|---|---|
|yum  clean all         |          清空缓存信息|
|yum  list              |          列出所有包的信息|
|yum  list  httpd       |          查看 httpd 是否安装|
|yum  info httpd        |          显示 httpd 包的详细具体信息|
|yum install httpd   -y |          安装 httpd 包|
|yum remove httpd  -y   |          卸载 httpd 包|
|yum search 关键词         |      根据关键词，在已发现的repo源中搜索包含关键词的rpm包|
|yum provides 命令        |         根据命令，在已发现的repo源中搜索安装指令的rpm包|
|yum history  list/info/undo/redo number|   history可以列出，查看，重装，反安装对应的包，但是是以yum指令的操作顺序为依据的，所以需要加指定的数字执行|
|yum update -y    |                    升级所有包同时也升级软件和系统内核|
|yum upgrade  -y  |                   只升级所有包，不升级软件和系统内核|

### 8. rpm包管理
|命令|含义|
|---|---|
|rpm -qa|查询所有已安装软件的rpm包信息，列出包的版本|
|rpm  -ivh  包.rpm|i表示安装，v表示显示安装过程，h表示以‘#’作为进度，显示安装进度|
|rpm -e --nodeps 包名|卸载|
|rpm  -q httpd   |  查看 httpd 是否安装|
|rpm -qi  httpd  |  列出 httpd 软件的详细信息|
||rpm -qc httpd   |  查看 httpd 的配置文件目录|
|rpm  -ql  httpd |  查看 httpd 所包含的文件|
|whereis  httpd  |  查看httpd的安装路径和可执行文件路径|

