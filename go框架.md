# 一、gorm

# 1. quickStart

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

