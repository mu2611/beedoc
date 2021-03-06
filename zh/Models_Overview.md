# 模型（Models）－ beego ORM

[![Build Status](https://drone.io/github.com/astaxie/beego/status.png)](https://drone.io/github.com/astaxie/beego/latest)

beego ORM 是一个强大的 Go 语言 ORM 框架。她的灵感主要来自 Django ORM 和 SQLAlchemy。

目前该框架仍处于开发阶段，可能发生任何导致不兼容的改动。

**已支持数据库：**

* MySQL：[github.com/go-sql-driver/mysql](https://github.com/go-sql-driver/mysql)
* PostgreSQL：[github.com/lib/pq](https://github.com/lib/pq)
* Sqlite3：[github.com/mattn/go-sqlite3](https://github.com/mattn/go-sqlite3)

以上数据库驱动均通过基本测试，但我们仍需要您的反馈。

**ORM 特性：**

* 支持 Go 的所有类型存储
* 轻松上手，采用简单的 CRUD 风格
* 自动 Join 关联表
* 跨数据库兼容查询
* 允许直接使用 SQL 查询／映射
* 严格完整的测试保证 ORM 的稳定与健壮

更多特性请在文档中自行品读。

**安装 ORM：**

	go get github.com/astaxie/beego/orm

### 修改日志

* 2013-08-27: [自动建表](Models_Cmd#自动建表)继续改进
* 2013-08-19: [自动建表](Models_Cmd#自动建表)功能完成
* 2013-08-13: 更新数据库类型测试
* 2013-08-13: 增加 Go 类型支持，包括 int8、uint8、byte、rune 等
* 2013-08-13: 增强 date／datetime 的时区支持

### 快速入门

#### 简单示例

```go
package main

import (
	"fmt"
	"github.com/astaxie/beego/orm"
	_ "github.com/go-sql-driver/mysql" // import your used driver
)

// Model Struct
type User struct {
	Id   int    `orm:"auto"`
	Name string `orm:"size(100)"`
}

func init() {
	// register model
	orm.RegisterModel(new(User))

	// set default database
	orm.RegisterDataBase("default", "mysql", "root:root@/my_db?charset=utf8", 30)
}

func main() {
	o := orm.NewOrm()

	user := User{Name: "slene"}

	// insert
	id, err := o.Insert(&user)

	// update
	user.Name = "astaxie"
	num, err := o.Update(&user)

	// read one
	u := User{Id: user.Id}
	err = o.Read(&u)

	// delete
	num, err = o.Delete(&u)	
}
```
	
#### 关联查询

```go
type Post struct {
	Id    int    `orm:"auto"`
	Title string `orm:"size(100)"`
	User  *User  `orm:"rel(fk)"`
}

var posts []*Post
qs := o.QueryTable("post")
num, err := qs.Filter("User__Name", "slene").All(&posts)
```

#### SQL 查询

当您无法使用 ORM 来达到您的需求时，也可以直接使用 SQL 来完成查询／映射操作。

```go
var maps []Params
num, err := o.Raw("SELECT id FROM user WHERE name = ?", "slene").Values(&maps)
if num > 0 {
	fmt.Println(maps[0]["id"])
}
```

#### 事务处理

```go
o.Begin()
...
user := User{Name: "slene"}
id, err := o.Insert(&user)
if err != nil {
	o.Commit()
} else {
	o.Rollback()
}
```

#### 调试查询日志

在开发环境下，您可以使用以下指令来开启查询调试模式：

```go
func main() {
	orm.Debug = true
...
```

开启后将会输出所有查询语句，包括执行、准备、事务等。

例如：

```go
[ORM] - 2013-08-09 13:18:16 - [Queries/default] - [    db.Exec /     0.4ms] - 	[INSERT INTO `user` (`name`) VALUES (?)] - `slene`
...
```

注意：我们不建议您在部署产品后这样做。

### 文档索引

1. [Orm 使用方法](Models_ORM)
	- [数据库的设置](Models_ORM#数据库的设置)
		* [驱动类型设置](Models_ORM#registerdriver)
		* [参数设置](Models_ORM#registerdatabase)
		* [时区设置](Models_ORM#时区设置)
	- [注册模型](Models_ORM#注册模型)
	- [ORM 接口使用](Models_ORM#orm-接口使用)
	- [调试模式打印查询语句](Models_ORM#调试模式打印查询语句)
2. [对象的CRUD操作](Models_Object)
3. [高级查询](Models_Query)
	- [使用的表达式语法](Models_Query#expr)
	- [支持的操作符号](Models_Query#operators)
	- [高级查询接口使用](Models_Query#高级查询接口使用)
4. [使用SQL语句进行查询](Models_RawSQL)
5. [事务处理](Models_Transaction)
6. [模型定义](Models_Models)
	- [自定义表名](Models_Models#自定义表名)
	- [设置参数](Models_Models#设置参数)
	- [表关系设置](Models_Models#表关系设置)
	- [模型字段与数据库类型的对应](Models_Models#模型字段与数据库类型的对应)
7. [命令模式](Models_Cmd)
	- [自动建表](Models_Cmd#自动建表)
	- [打印建表SQL](Models_Cmd#打印建表sql)
8. [Test ORM](Models_Test)
9. [自定义字段](Models_Fields)
10. [FAQ](Models_Faq)


### API 文档

请移步 [Go Walker](http://gowalker.org/github.com/astaxie/beego/orm)。
