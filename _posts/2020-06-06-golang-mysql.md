---
title: Golang MySQL
key: 2020-06-06
tags: go golang mysql
---



![2839526_203626093214_2.jpg](https://upload-images.jianshu.io/upload_images/1881763-4c4fdeaae686f126.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



这是一个描述 go 语言中如何操作 MySQL 数据库的初级教程。适合刚刚接触 go 语言的开发者。



<!--more-->



## 库：

### database/sql :

database/sql 是 golang 的标准库之一，它提供了一系列接口方法，用于访问关系数据库。它并不会提供数据库特有的方法，那些特有的方法交给数据库驱动去实现。

database/sql库提供了一些 type。这些类型对掌握它的用法非常重要。

- DB :  数据库对象。 

`sql.DB` 类型代表了数据库连接池。和其他语言不一样，它并不是数据库连接。golang 中的连接来自内部实现的连接池，连接的建立是惰性的，当你需要连接的时候，连接池会自动帮你创建。通常你不需要操作连接池。一切都有go来帮你完成。

它提供了一些跟数据库交互的函数，同时管理维护一个数据库连接池，帮你处理了单调而重复的管理工作，并且在多个goroutines也是十分安全。

sql.DB表示是数据库抽象，因此你有几个数据库就需要为每一个数据库创建一个sql.DB对象。**因为它维护了一个连接池，因此不需要频繁的创建和销毁。它需要长时间保持，因此最好是设置成一个全局变量以便其他代码可以访问。**

- Results: 结果集。

数据库查询的时候，都会有结果集。`sql.Rows` 类型表示查询返回多行数据的结果集。`sql.Row` 则表示单行查询结果的结果集。当然，对于插入更新和删除，返回的结果集类型为 `sql.Result` 。

- Statements:

语句。`sql.Stmt` 类型表示 SQL 查询语句，例如 DDL，DML 等类似的 SQL 语句。可以把当成 prepare 语句构造查询，也可以直接使用 `sql.DB` 的函数对其操作。

### go-sql-driver/mysql

对于其他语言，查询数据的时候需要创建一个连接，而对于 golang 则是需要创建一个数据库抽象对象。连接将会在查询需要的时候，由连接池创建并维护。使用`sql.Open` 函数创建数据库对象。它的第一个参数是数据库驱动名，第二个参数是一个连接字串（符合DSN风格，可以是一个 TCP 连接，一个  unix socket 等）。

创建数据库对象需要引入标准库 `database/sql`，同时还需要引入驱动 `go-sql-driver/mysql`。使用 `_` 表示引入驱动的变量，这样做的目的是为了在你的代码中不至于和标注库的函数变量 namespace 冲突。

### 连接池

只使用 `sql.Open` 函数就可以创建连接池，可是此时只是初始化了连接池，并没有创建任何连接。连接创建都是**惰性**的，只有当你真正使用到连接的时候，连接池才会创建连接。连接池很重要，它直接影响着你的程序行为。

使用连接池工作相当简单。当你的函数(例如Exec，Query)调用需要访问底层数据库的时候，函数首先会向连接池请求一个连接。如果连接池有空闲的连接，则返回给函数。否则连接池将会创建一个新的连接给函数。一旦连接给了函数，连接则归属于函数。函数执行完毕后，要么把连接所属权归还给连接池，要么传递给下一个需要连接的（Rows）对象，最后使用完连接的对象也会把连接释放回到连接池。

请求一个连接的函数有好几种，执行完毕处理连接的方式稍有差别，大致如下：

- `db.Ping()` 调用完毕后会马上把连接返回给连接池。
- `db.Exec()` 调用完毕后会马上把连接返回给连接池，但是它返回的 `Result` 对象还保留这连接的引用，当后面的代码需要处理结果集的时候连接将会被重用。
- `db.Query()` 调用完毕后会将连接传递给 `sql.Rows` 类型，当然后者迭代完毕或者显示的调用 `.Close()` 方法后，连接将会被释放回到连接池。
- `db.QueryRow()` 调用完毕后会将连接传递给 `sql.Row` 类型，当 `.Scan()` 方法调用之后把连接释放回到连接池。
- `db.Begin()` 调用完毕后将连接传递给 `sql.Tx` 类型对象，当 `.Commit()` 或 `.Rollback()` 方法调用后释放连接。

因为每一个连接都是惰性创建的，如何验证 `sql.Open` 调用之后，`sql.DB` 对象可用呢？通常使用 `db.Ping()` 方法初始化：

```go
db, err := sql.Open("driverName", "dataSourceName")
if err != nil{
    log.Fatalln(err)
}

defer db.Close()

err = db.Ping()
if err != nil{
   log.Fatalln(err)
}
```


调用了 Ping 之后，连接池一定会初始化一个数据库连接。当然，实际上对于失败的处理，应该定义一个符合自己需要的方式，现在为了演示，简单的使用`log.Fatalln(err)` 表示了。

### 连接池配置

golang 直到 1.2 版本以后才有一些简单的配置。可是1.2版本的连接池有一个bug，请升级更高的版本。

配置连接池有两个的方法：

- `db.SetMaxOpenConns(n int) `

设置打开数据库的最大连接数。包含正在使用的连接和连接池的连接。如果你的函数调用需要申请一个连接，并且连接池已经没有了连接或者连接数达到了最大连接数。此时的函数调用将会被 block，直到有可用的连接才会返回。设置这个值可以避免并发太高导致连接mysql出现 too many connections 的错误。该函数的默认设置是 `0`，表示无限制。

- `db.SetMaxIdleConns(n int)` 

设置连接池中的保持连接的最大连接数。默认也是 `0` ，表示连接池不会保持释放会连接池中的连接的连接状态：即当连接释放回到连接池的时候，连接将会被关闭。这会导致连接再连接池中频繁的关闭和创建。

对于连接池的使用依赖于你是如何配置连接池，如果使用不当会导致下面问题：

- 大量的连接空闲，导致额外的工作和延迟。
- 连接数据库的连接过多导致错误。
- 连接阻塞。
- 连接池有超过十个或者更多的死连接，限制就是10次重连。

大多数时候，如何使用 `sql.DB` 对连接的影响大过连接池配置的影响。这些具体问题我们会再使用sql.DB的时候逐一介绍。


```go
package main

import (
	"database/sql"
	"log"

	_ "github.com/go-sql-driver/mysql"
)

type Mytb struct {
	Id    int
	Name  string
	Email string
}

func main() {

	db, err := sql.Open("mysql", "root:mp9b80:oNO4i@tcp(192.168.0.112:3306)/chaos")
	if err != nil {
		panic(err)
	}
	db.SetMaxOpenConns(20)
	db.SetMaxIdleConns(300)

	err = db.Ping()
	if err != nil {
		log.Fatalln(err)
	}
	defer db.Close()

	insertResult, err := db.Exec("insert into mytb (`name`, `email`) values (? , ?)", "handong", "handong222@163.com")

	if err != nil {
		log.Fatalln(err)
	}

	id, err := insertResult.LastInsertId()
	affected, _ := insertResult.RowsAffected()

	log.Printf("get id ===> %d, affected =====> %d\n", id, affected)

	rows, err := db.Query("select id, name, email from chaos.mytb")

	if err != nil {
		log.Fatalln(err)
	}

	defer rows.Close()

	for rows.Next() {
		mytb := Mytb{}

		err = rows.Scan(&mytb.Id, &mytb.Name, &mytb.Email)
		if err != nil {
			log.Printf(">>>>>>>>>>> db %v\n", db)
			log.Fatalln(err)
		}
		log.Printf("found row containing ..%v %v %v \n", mytb.Id, mytb.Name, mytb.Email)

	}
}

```

<< EOF >>

If you like TeXt, don't forget to give me a star :star2:.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
