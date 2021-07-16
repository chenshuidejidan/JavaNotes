# 一、gorm

## 1. quickStart

安装：

```
go get -u gorm.io/gorm
go get -u gorm.io/driver/sqlite
```

连接MySQL数据库：

```go
import (
  "gorm.io/driver/mysql"
  "gorm.io/gorm"
)

func main() {
  dsn := "user:pass@tcp(127.0.0.1:3306)/dbname?charset=utf8mb4&parseTime=True&loc=Local"
  db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})
}
```

MySQL驱动提供的高级配置：

```go
db, err := gorm.Open(mysql.New(mysql.Config{
  DSN: "gorm:gorm@tcp(127.0.0.1:3306)/gorm?charset=utf8&parseTime=True&loc=Local", // DSN data source name
  DefaultStringSize: 256, // string 类型字段的默认长度
  DisableDatetimePrecision: true, // 禁用 datetime 精度，MySQL 5.6 之前的数据库不支持
  DontSupportRenameIndex: true, // 重命名索引时采用删除并新建的方式，MySQL 5.7 之前的数据库和 MariaDB 不支持重命名索引
  DontSupportRenameColumn: true, // 用 `change` 重命名列，MySQL 8 之前的数据库和 MariaDB 不支持重命名列
  SkipInitializeWithVersion: false, // 根据当前 MySQL 版本自动配置
}), &gorm.Config{})
```



## 2. model和表格

```go
type student struct {
	gorm.Model        //id, CreatedAt, UpdatedAt,DeletedAt
	Sid        string `gorm:"index"`
  Name       string
	Sex        bool
  Age        int `gorm:"default:18"`
}


//根据model自动创建表格
db.AutoMigrate(&student{}) //自动创建表格
```



## 3. 增加元素

添加**单个记录**：

```go
res := db.Create(&student)
if res.Error != nil || res.ID == 0 {
  fmt.Errorf("addStudent error!")
  return false
}
```



根据**Slice添加多条记录**：gorm会生成一个单一的sql语句插入所有的数据，并**回填所有值，钩子函数也会被调用**

```go
students := []student{{...},{...},...}
db.Create(&students)

for _, student := students {
  fmt.Println(student.ID)
}
```



根据**Map创建记录**，map创建的方式不会调用钩子函数，而且不会回填值(传入的不是对象)，gorm.Model的字段也不会被自动填充

```go
	db.Model(&student{}).Create(map[string]interface{}{
		"Sid":  "1120110144",
		"Name": "滴滴44",
		"Sex":  false,
		"Age":  20,
	})
	db.Model(&student{}).Create([]map[string]interface{}{
		{"Sid": "1120110155",
			"Name": "滴滴55",
			"Sex":  false,
			"Age":  20,
		}, {
			"Sid":  "1120110156",
			"Name": "滴滴56",
			"Sex":  false,
			"Age":  20,
		},
	})
```



**钩子函数**：可以创建四种Hooks method：`BeforeSave`, `BeforeCreate`, `AfterSave`, `AfterCreate`

```go
func (s student) BeforeCreate(tx *gorm.DB) (err error) {
	if s.Age < 10 || s.Age > 30 {
		return fmt.Errorf("invalid age")
	}
	return
}
```

**忽略钩子函数：** 使用`SkipHooks` session

```go
db.Session(&gorm.Session{SkipHooks: true}).Create(&students)
```



**关联创建：** 关联创建使得组合对象会被自动创建

**忽略关联创建：** 

- 忽略单个对象  `db.Omit("CreditCard").Create(&student)`
- 忽略全部关联对象  `db.Omit(clause.Associations).Create(&student)`



## 4. 删除元素

**删除单个元素**：删除单个元素需要指定主键，否则会触发批量删除

```go
// email 的 ID 是 `10`
db.Delete(&email)
// DELETE from emails where id = 10;

// 带额外条件的删除
db.Where("name = ?", "jinzhu").Delete(&email)
// DELETE from emails where id = 10 AND name = "jinzhu";
```



**根据主键删除**：

```go
db.Delete(&User{}, 10)
// DELETE FROM users WHERE id = 10;

db.Delete(&User{}, "10")
// DELETE FROM users WHERE id = 10;

db.Delete(&users, []int{1,2,3})
// DELETE FROM users WHERE id IN (1,2,3);
```



**钩子函数**：`BeforeDelete`、`AfterDelete`



**批量删除**：如果指定的值不包括主键，那么gorm会执行批量删除，删除所有匹配的记录

```go
db.Where("name like ?", "%滴滴10%").Delete(&student{})
// DELETE from students where name LIKE "%滴滴10%";

ddb.Delete(&student{}, "name like ?", "%9")
// DELETE from students where name LIKE "%9";
```



**执行原生sql** ：原生sql的删除语句会直接删除记录，而不是像orm一样软删除

```go
db.Exec("Delete from students WHERE name=?", "滴滴8")
```



**软删除**：如果model包含了一个 `gorm.DeletedAt` 字段（`gorm.Model` 已经包含了该字段)，它将自动获得软删除的能力

```go
// user 的 ID 是 `111`
db.Delete(&user)
// UPDATE users SET deleted_at="2013-10-29 10:23" WHERE id = 111;
```



**查询软删除的记录：**

```go
db.Unscoped().Where("age = 20").Find(&users)
// SELECT * FROM users WHERE age = 20;
```



**永久删除：**

```go
db.Unscoped().Delete(&order)
// DELETE FROM orders WHERE id=10;
```





## 5. 更新元素



**更新单列：**

```go
// 条件更新
db.Model(&User{}).Where("active = ?", true).Update("name", "hello")
// UPDATE users SET name='hello', updated_at='2013-11-17 21:34:10' WHERE active=true;

// User 的 ID 是 `111`
db.Model(&user).Update("name", "hello")
// UPDATE users SET name='hello', updated_at='2013-11-17 21:34:10' WHERE id=111;

// 根据条件和 model 的值进行更新
db.Model(&user).Where("active = ?", true).Update("name", "hello")
// UPDATE users SET name='hello', updated_at='2013-11-17 21:34:10' WHERE id=111 AND active=true;
```



**更新多列** : 通过 struct 或者 map 更新

```go
// 根据 `struct` 更新属性，只会更新非零值的字段
db.Model(&user).Updates(User{Name: "hello", Age: 18, Active: false})
// UPDATE users SET name='hello', age=18, updated_at = '2013-11-17 21:34:10' WHERE id = 111;

// 根据 `map` 更新属性
db.Model(&user).Updates(map[string]interface{}{"name": "hello", "age": 18, "actived": false})
// UPDATE users SET name='hello', age=18, actived=false, updated_at='2013-11-17 21:34:10' WHERE id=111;
```



**更新选定字段**：select，忽略字段omit

```go
// 只更新name
db.Model(&user).Select("name").Updates(map[string]interface{}{"name": "hello", "age": 18, "actived": false})
// UPDATE users SET name='hello' WHERE id=111;

// 除了name以外的都更新
db.Model(&user).Omit("name").Updates(map[string]interface{}{"name": "hello", "age": 18, "actived": false})
// UPDATE users SET age=18, actived=false, updated_at='2013-11-17 21:34:10' WHERE id=111;

// Select 和 Struct （可以选中更新零值字段）
db.Model(&result).Select("Name", "Age").Updates(User{Name: "new_name", Age: 0})
// UPDATE users SET name='new_name', age=0 WHERE id=111;
```



**where批量更新：**

```go
db.Model(User{}).Where("role = ?", "admin").Updates(User{Name: "hello", Age: 18})
// UPDATE users SET name='hello', age=18 WHERE role = 'admin;
```



**获取更新的记录数**：通过 `RowsAffected` 得到更新的记录数

```go
result := db.Model(User{}).Where("role = ?", "admin").Updates(User{Name: "hello", Age: 18})
// UPDATE users SET name='hello', age=18 WHERE role = 'admin;

result.RowsAffected // 更新的记录数
result.Error        // 更新的错误
```



**通过sql表达式更新**

```go
dulti := 2
db.Model(&student{}).Where("sex=1").Update("age", gorm.Expr("age * ?", multi))

db.Model(&product).Updates(map[string]interface{}{"price": gorm.Expr("price * ? + ?", 2, 100)})
// UPDATE "products" SET "price" = price * 2 + 100, "updated_at" = '2013-11-17 21:34:10' WHERE "id" = 3;
```



**通过子查询更新**

```go
db.Table("users as u").Where("name = ?", "jinzhu").Update("company_name", db.Table("companies as c").Select("name").Where("c.id = u.company_id"))
```



## 6. 查询元素

**查询单个对象**：`First`、`Take`、`Last`，使用时默认添加了`limit 1`条件

```go
// 获取第一条记录（主键升序），若没有主键则用第一个字段 
db.First(&user)
// SELECT * FROM users ORDER BY id LIMIT 1;

// 获取一条记录，没有指定排序字段
db.Take(&user)
// SELECT * FROM users LIMIT 1;

// 获取最后一条记录（主键降序）
db.Last(&user)
// SELECT * FROM users ORDER BY id DESC LIMIT 1;

result := db.First(&user)
result.RowsAffected // 返回找到的记录数
result.Error        // returns error

// 检查 ErrRecordNotFound 错误
errors.Is(result.Error, gorm.ErrRecordNotFound)
```



**根据主键检索：**

```go
db.First(&user, 10)
// SELECT * FROM users WHERE id = 10;

db.First(&user, "10")
// SELECT * FROM users WHERE id = 10;

db.Find(&users, []int{1,2,3})
// SELECT * FROM users WHERE id IN (1,2,3);
```



**检索全部记录：**

```go
// 获取全部记录
result := db.Find(&users)
// SELECT * FROM users;

result.RowsAffected // 返回找到的记录数，相当于 `len(users)`
```



**string条件检索：** 也可以使用struct，map的条件来检索

```go
// 获取第一条匹配的记录
db.Where("name = ?", "jinzhu").First(&user)
// SELECT * FROM users WHERE name = 'jinzhu' ORDER BY id LIMIT 1;

// 获取全部匹配的记录
db.Where("name <> ?", "jinzhu").Find(&users)
// SELECT * FROM users WHERE name <> 'jinzhu';

// IN
db.Where("name IN ?", []string{"jinzhu", "jinzhu 2"}).Find(&users)
// SELECT * FROM users WHERE name IN ('jinzhu','jinzhu 2');

// LIKE
db.Where("name LIKE ?", "%jin%").Find(&users)
// SELECT * FROM users WHERE name LIKE '%jin%';

// AND
db.Where("name = ? AND age >= ?", "jinzhu", "22").Find(&users)
// SELECT * FROM users WHERE name = 'jinzhu' AND age >= 22;
```



**struct检索会忽略零值：0, '', false, ...** 因为0值就是struct未初始化字段的默认值，防止歧义

```go
db.Where(&User{Name: "jinzhu", Age: 0}).Find(&users)
// SELECT * FROM users WHERE name = "jinzhu";
```



**直接使用Find, First来inline检索**：

```go
db.First(&user, "id = ?", "string_primary_key")
// SELECT * FROM users WHERE id = 'string_primary_key';

// Plain SQL
db.Find(&user, "name = ?", "jinzhu")
// SELECT * FROM users WHERE name = "jinzhu";

db.Find(&users, "name <> ? AND age > ?", "jinzhu", 20)
// SELECT * FROM users WHERE name <> "jinzhu" AND age > 20;

// Struct
db.Find(&users, User{Age: 20})
// SELECT * FROM users WHERE age = 20;

// Map
db.Find(&users, map[string]interface{}{"age": 20})
// SELECT * FROM users WHERE age = 20;
```



**Not条件**

```go
db.Not([]int64{1,2,3}).First(&user)
// SELECT * FROM users WHERE id NOT IN (1,2,3) ORDER BY id LIMIT 1;
```



**Or条件**

```go
db.Where("role = ?", "admin").Or("role = ?", "super_admin").Find(&users)
```



**选择特定字段**

```go
db.Select("name", "age").Find(&users)
// SELECT name, age FROM users;

db.Select([]string{"name", "age"}).Find(&users)
// SELECT name, age FROM users;
```



**Order**

```go
db.Order("age desc, name").Find(&users)
// SELECT * FROM users ORDER BY age desc, name;
```



**Group, Having**

```go
db.Model(&User{}).Select("name, sum(age) as total").Group("name").Having("name = ?", "group").Find(&result)
// SELECT name, sum(age) as total FROM `users` GROUP BY `name` HAVING name = "group"
```



**Join**

```go
db.Model(&User{}).Select("users.name, emails.email").Joins("left join emails on emails.user_id = users.id").Scan(&result{})
// SELECT users.name, emails.email FROM `users` left join emails on emails.user_id = users.id
```



**Scan**：和Find功能类似，但是可以查结果到一个特定的结构体中

```go
type Result struct {
  Name string
  Age  int
}

var result Result
db.Table("users").Select("name", "age").Where("name = ?", "Antonio").Scan(&result)

// Raw SQL
db.Raw("SELECT name, age FROM users WHERE name = ?", "Antonio").Scan(&result)
```



**Locking: for update, share mode**

```go
db.Clauses(clause.Locking{Strength: "UPDATE"}).Find(&users)
// SELECT * FROM `users` FOR UPDATE

db.Clauses(clause.Locking{
  Strength: "SHARE",
  Table: clause.Table{Name: clause.CurrentTable},
}).Find(&users)
// SELECT * FROM `users` FOR SHARE OF `users`
```



**Count**

```go
var total int64
// Count with Distinct
db.Model(&User{}).Distinct("name").Count(&total)
// SELECT COUNT(DISTINCT(`name`)) FROM `users`

db.Table("deleted_users").Select("count(distinct(name))").Count(&total)
// SELECT count(distinct(name)) FROM deleted_users
```





# 二、rpc框架

一个函数需要能够被远程调用，需要满足如下五个条件：

- the method’s type is exported.  //**方法所属类型是导出的**
- the method is exported.  //**方法是导出的**
- the method has two arguments, both exported (or builtin) types. //**两个参数类型是导出的**
- the method’s second argument is a pointer.  //**第二个参数是指针类型**
- the method has return type error.  //**返回error**

```go
func (t *T) MethodName(argType T1, replyType *T2) error
```

## 1. 传输协议

```c
 *   0     1     2     3     4        5     6     7     8         9          10      11     12  13  14   15 16
 *   +-----+-----+-----+-----+--------+----+----+----+------+-----------+-------+----- --+-----+-----+-------+
 *   |   magic   code        |version | full length         | messageType| codec|compress|    RequestId       |
 *   +-----------------------+--------+---------------------+-----------+-----------+-----------+------------+
 *   |                                                                                                       |
 *   |                                         body                                                          |
 *   |                                                                                                       |
 *   |                                        ... ...                                                        |
 *   +-------------------------------------------------------------------------------------------------------+
 * 4B  magic code（魔法数）   1B version（版本）   4B full length（消息长度）    1B messageType（消息类型）
 * 1B compress（压缩类型） 1B codec（序列化类型）   4B  requestId（请求的Id）
```





## 2. 结构体和服务的映射

客户端发来的请求包含ServiceMethod和Argv

```go
{
    "ServiceMethod"： "T.MethodName"
    "Argv"："0101110101..." // 序列化之后的字节流
}
```

硬编码需要每个方法进行判断，编写很多`switch case`代码

好的办法可以借助**反射**，获取结构体的方法，通过方法获取方法的所有参数类型和返回值

例如：

```go
func main() {
	var wg sync.WaitGroup
	typ := reflect.TypeOf(&wg)
	for i := 0; i < typ.NumMethod(); i++ {
		method := typ.Method(i)
		argv := make([]string, 0, method.Type.NumIn())
		returns := make([]string, 0, method.Type.NumOut())
		// j 从 1 开始，第 0 个入参是 wg 自己。
		for j := 1; j < method.Type.NumIn(); j++ {
			argv = append(argv, method.Type.In(j).Name())
		}
		for j := 0; j < method.Type.NumOut(); j++ {
			returns = append(returns, method.Type.Out(j).Name())
		}
		log.Printf("func (w *%s) %s(%s) %s",
			typ.Elem().Name(),
			method.Name,
			strings.Join(argv, ","),
			strings.Join(returns, ","))
    }
}

//结果：
//func (w *WaitGroup) Add(int)
//func (w *WaitGroup) Done()
//func (w *WaitGroup) Wait()
```

**通过反射实现Service**

```go
func (m *methodType) newArgv() reflect.Value {
	var argv reflect.Value
	if m.ArgType.Kind() == reflect.Ptr {
		argv = reflect.New(m.ArgType.Elem()) // reflect.Type.Elem() 获取指针类型的值的类型，相当于 * 操作
	} else {
		argv = reflect.New(m.ArgType).Elem()
	}
	return argv
}

func (s *service) registerMethods() {
	s.method = make(map[string]*methodType)
	for i := 0; i < s.typ.NumMethod(); i++ {
		method := s.typ.Method(i)
		mType := method.Type
		if mType.NumIn() != 3 || mType.NumOut() != 1 {
			continue
		}
		if mType.Out(0) != reflect.TypeOf((*error)(nil)).Elem() {
			continue
		}
		argType, replyType := mType.In(1), mType.In(2)
		if !isExportedOrBuiltinType(argType) || !isExportedOrBuiltinType(replyType) {
			continue
		}
		s.method[method.Name] = &methodType{
			method:    method,
			ArgType:   argType,
			ReplyType: replyType,
		}
		log.Printf("server [registerMethods] rpc server: register %s.%s\n", s.name, method.Name)
	}
}
```



## 3. 超时处理

超时处理是RPC框架的基本能力，缺少超时处理机制的话，无论是客户端还是服务端都容易因为网络或者其他错误导致资源耗尽，大大降低服务的可用性。

整个远程调用中，需要客户端处理的超时有：

1. 与服务端建立连接导致的超时
2. 发送请求到服务端，写报文导致的超时
3. 等待服务端处理时，等待处理导致的超时
4. 从服务端接收响应，读报文导致的超时

需要服务端处理的超时有：

1. 读取客户端请求报文时，读报文导致的超时
2. 发送响应报文时，写报文导致的超时
3. 调用映射服务的方法时，处理报文导致的超时



添加3个超时处理机制：

1. 客户端创建连接时
2. 客户端 `Client.Call()` 整个过程导致的超时（发送，等待处理，接收报文）
3. 服务端处理报文，即 `Server.handleRequest` 超时



## 4. 负载均衡策略

假设有多个服务实例，每个实例都提供相同的功能，为了提高整个系统的吞吐量，每个实例部署在不同的容器中。客户端可以选择任意一个实例进行调用，获取想要的结果。有一下几种策略：

1. 随机选择
2. 轮询(Round Robin)：依次调度不同的服务器，每次调度执行i = (i+1) mod n
3. 加权轮询(Weight Round Robin)：在轮询算法的基础上，为每个服务实例设置一个权重，高性能的机器赋予更高的权重，也可以根据服务实例的当前负载情况做出动态调整
4. 哈希/一致性哈希：根据请求的某些特征，计算hash值，根据hash值将请求发送到对应的机器。一致性hash还可以解决服务实例动态添加情况下调度抖动的问题





## 5. 服务发现和注册中心

负载均衡的前提是有多个服务实例，就需要服务发现模块

![geerpc registry](picture/go框架/registry.jpg)

有了注册中心之后，客户端和服务端都无需感知对方的存在，只需要感知注册中心的存在即可

1. 服务端启动后，向注册中心发送注册消息，注册中心得知该服务已经启动，处于可用状态。一般来说，服务端还需要定期向注册中心发送心跳，证明自己可用
2. 客户端向注册中心询问哪些服务可用，注册中心将可用的服务列表返回给客户端
3. 客户端根据注册中心得到的服务列表，选择其中一个发起调用

**如果没有注册中心，客户端需要硬编码服务端的地址，而没有机制保证客户端是否处于可用状态**。当然注册中心还有其他功能，比如配置的动态同步、通知机制等，比较常见的注册中心有`etcd`、`zookeeper`、`consul`，一般比较出名的微服务或RPC框架，这些主流的注册中心都是支持的



使用zookeeper作为注册中心

创建永久节点：

```go
servicePath = ZK_REGISTER_ROOT_PATH + "/" + rpcServiceName + inetSocketAddress
```



当我们的服务被注册进ZooKeeper时，我们将完整的服务名称作为根节点，子节点是对应的服务地址(ip:port)



客户端根据完整的服务名称便可以找到对应的服务地址，查出来的地址可能不止一个，就可以根据负载均衡策略选出一个服务地址



# 三、分布式缓存框架

## 1. 缓存淘汰策略

缓存淘汰策略：FIFO、LFU、LRU
	

- **FIFO**：先进先出，淘汰最老的记录，世界使用队列即可，实现简单，但是很多时候最早添加的也经常被访问，却因为呆的时间太长被淘汰，导致缓存命中率降低
- **LFU**：最少使用，淘汰缓存中访问频率最低的记录，LFU认为数据过去被访问多次，将来被访问的频率也更高，需要维护一个按访问次数排序的队列。该方法命中率较高，但是维护队列成本大，此外如果数据访问模式发生变化，LFU需要花较大时间去适应(例如历史很高，最近很少使用)
- **LRU**：最近最少使用，相对于仅考虑时间的FIFO和仅考虑频率的LFU，LRU相对平衡，LRU认为最近被访问过的话，将来被访问的改率也会更高。实现简单，最近被访问的移动到队尾，每次从队首淘汰



LRU的核心数据结构：

![implement lru algorithm with golang](picture/go框架/lru.jpg)

- 字典map：存储键值对映射关系，使得根据key查找value的复杂度是O(1)，在字典中插入的复杂度也是O(1)
- 双向链表：队列，所有值存放在链表中，当访问某个值时，移动到队尾的复杂度是O(1)，在队尾新增和对头删除记录的复杂度也均为O(1)



## 2. 一致性哈希算法

对于分布式缓存来说，当一个节点接收到请求，如果该节点并没有存储缓存值，那么它面临的难题是，从谁那获取数据？自己，还是节点1, 2, 3, 4… 

我们可以对于一个给定的key，使用特定的hash算法（对key取hash后按模选择节点），使得该key每次都访问同一个节点，避免每次访问不同节点都不命中缓存，从而多次从数据源获取数据

![hash select peer](picture/go框架/hash_select.jpg)

但是有一个问题，**如果节点的数量发生了变化，那么缓存值对应的节点几乎都发生了变化**，即几乎所有的缓存值都失效了，节点收到请求时均需去数据源获取数据，容易引起`缓存雪崩`

> **缓存雪崩**：缓存在同一时刻全部失效，造成瞬时DB请求量大、压力骤增，引起雪崩。常因为缓存服务器宕机，或缓存设置了相同的过期时间引起。
>
> **缓存击穿**：一个存在的key，在缓存过期的一刻，同时有大量的请求，这些请求都会击穿到 DB ，造成瞬时DB请求量大、压力骤增。
>
> **缓存穿透**：查询一个不存在的数据，因为不存在则不会写到缓存中，所以每次都会去请求 DB，如果瞬间流量过大，穿透到 DB，导致宕机。

一致性哈希算法就是解决这个问题的。

一致性哈希算法将 key 映射到 2^32 的空间中，将这个数字首尾相连，形成一个环。

- 计算节点/机器(通常使用节点的名称、编号和 IP 地址)的哈希值，放置在环上。
- 计算 key 的哈希值，放置在环上，**顺时针寻找到的第一个节点，就是应选取的节点/机器**。

![一致性哈希添加节点 consistent hashing add peer](picture/go框架/add_peer.jpg)

一致性哈希算法，在新增/删除节点时，只需要重新定位该节点附近的一小部分数据，而不需要重新定位所有的节点，这就解决了上述的问题



**数据倾斜问题：**

如果服务器的节点过少，容易引起 key 的倾斜。例如上面例子中的 peer2，peer4，peer6 分布在环的上半部分，下半部分是空的。那么映射到环下半部分的 key 都会被分配给 peer2，key 过度向 peer2 倾斜，缓存节点间负载不均。

为了解决这个问题，引入了**虚拟节点**的概念，一个真实节点对应多个虚拟节点

假设 1 个真实节点对应 3 个虚拟节点，那么 peer1 对应的虚拟节点是 peer1-1、 peer1-2、 peer1-3（通常以添加编号的方式实现），其余节点也以相同的方式操作。

- 第一步，计算虚拟节点的 Hash 值，放置在环上。
- 第二步，计算 key 的 Hash 值，在环上顺时针寻找到应选取的虚拟节点，例如是 peer2-1，那么就对应真实节点 peer2。

虚拟节点扩充了节点的数量，解决了节点较少的情况下数据容易倾斜的问题。而且代价非常小，只需要增加一个字典(map)维护真实节点与虚拟节点的映射关系即可













