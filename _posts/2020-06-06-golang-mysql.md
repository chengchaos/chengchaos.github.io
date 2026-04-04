---
title: Golang MySQL
key: 2020-06-06
tags: go golang mysql
---



![2839526_203626093214_2.jpg](https://upload-images.jianshu.io/upload_images/1881763-4c4fdeaae686f126.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这是一个描述 go 语言中如何操作 MySQL 数据库的初级教程。适合刚刚接触 go 语言的开发者。

<!--more-->

## 库

### sql.Register

这个存在于 `database/sql` 的函数时用来注册数据库驱动的，当第三方开发者开发数据库驱动的时候，都会实现 `init` 函数，在 `init` 里面会调用这个 `Register(name string, driver driver.Driver)` 完成驱动的注册。

下面是 mymysql , sqlite3 的驱动的调用：

```go

// https://github.com/mattn/go-sqlte3
func init() {
    sql.Register("sqlite3", &SQLiteDriver{})
}

// https://github.com/mikespook/mymysql
// Driver autumatically registered in database/sql
var d = Driver{ 
    proto : "tcp",
    raddr : "127.0.0.1:3306",
}

func init() {
    Register("SET NAMES utf8")
    sql.Register("mymysql", &d)
}
```

### driver.Driver

Driver 是一个数据库驱动的接口，他定义了一个方法：`Open(name string)` ，这个方法返回一个而数据库的 `Conn` 接口。

```go
type Driver interface {
    Open(name string) (Conn, error)
}
```

返回的 `Conn` 只能用来进行一次 goroutine 的操作。

第三方驱动都会定义这个函数，解析 name 参数获取相关数据库的连接信息，然后使用他们来初始化一个 `Conn` 并返回。

### driver.Conn

`Conn` 是一个数据库链接的接口定义，它定义了一系列的方法，这个 `Conn` 只能应用在一个 goroutine 里面，不能用在多个 goroutine 里面。

```go
type Conn interface {

    // 与当前链接相关的执行 SQL 语句的准备状态，可以进行查询、删除等操作。
    Prepare(query string) (Stmt, error)
    // 关闭当前的连接，执行释放链接所持有的资源等清理工作。因为驱动都实现了 
    // database/sql  里面建议的连接池，所以用户不用再去实现缓存 conn 之类的工作。
    Close() error
    // 返回一个代表事务处理的 `Tx`，可以通过它进行查询、更新等操作，
    // 或者对事务进行提交、回滚等操作。
    Begin() (Tx, error)
}
```

### driver.Stmt

`Stmt` 是一种准备好的状态，和 `Conn` 相关联，而且只能应用于一个 goroutine 中。

```go
type Stmt interface {

    // Close 方法关闭当前的连接状态，但是如果当前正在执行 Query，
    // Query 还是有效反回 Rows 数据。
    Close() error

    // 返回当前预留参数的个数，当返回值 大于等于 0 时，数据库驱动会
    // 智能检查调用者的参数，当数据库驱动不知道预留参数的时候，
    // 返回 -1 。
    NumInput() int

    // 执行 Prepare 准备好的 SQL，传入参数执行
    // update / insert 等操作， 返回 Result 。
    Exec(args []Value) (Result, error)

    // 执行 Prepare 准备好的 SQL，传入参数执行 
    // select 操作，返回 Rows 。
    Query(args []Value) (Rows, error)
}
```

### driver.Tx

事务处理一般就两个过程，提交或者回滚。

```go
type Tx interface {
    Commit() error
    Rollback() error
}
```

### driver.Execer

这是一个 `Conn` 可选择实现的接口。

```go
type Execer interface {
    Exec(query string, args []Value) (Result, error)
}
```

如果这个接口没有定义，那么在调用 `DB.Exec` 时，就会先调用 `Prepare` 返回 `Stmt`，然后再执行 `Stmt` 的 `Exec` ，最后关闭 `Stmt` 。

### driver.Result

这是执行 update / insert 等擦欧总返回的结果的接口定义。

```go
type Result interface {

    // 返回由数据库执行插入操作得到的自增 ID
    LastINsertId() (int6t4, error)

    // 返回执行操作影响的数据条目数。
    RowsAffected() (int64, error)
}
```

### driver.Rows

这是执行查询操作返回的结果集接口定义。

```go
type Rows interface {

    // 返回查询数据库表的字段信息，和 SQL 查询的字段一一对应。
    Columns() []string

    // 关闭 Rows 迭代器。
    Close() error

    // 用来返回下一条数据，把结果赋值给 dest。
    // dest 里面的元素必须是 driver？
    // Value 的值除了 string，返回的数据中所有的 string 都 
    // 必须要转换成 []byte 。
    // 如果没有数据了，Next 函数最后返回 io.EOF。
    Next(dest []Value) error
}
```

### driver.RowsAffected

`RowsAffected` 其实就是一个 int64 的别名，但是它实现了 `Result` 接口，以底层实现 `Result` 的表示方式。

```go
type RowsAffected int64

func (RowsAffected) LastInsertId() (int64, error)
func (v RowsAffected) RowsAffected() (int64, error)
```

### driver.Value

其实就是一个空接口，它可以容纳任何数据。

```go
type Value interface{}
```

drive 的 Value 是驱动必须能够操作的 Value，Value 要么是 nil，要么是下面的任意一种：

- int64
- float64
- bool
- []byte
- string [*] 除了 Rows.Next 返回的不能是 string。
- time.Time

### driver.ValueConverter

`ValueConverter` 接口定义了如何把一个普通的值转化成 `driver.Value` 的接口。

```go
type ValueConverter interface {
    ConvertValue(v interface{}) (Value, error)
}
```

在开发的数据库驱动包中实现这个接口的函数在很多地方会使用到。例如：

- 转化 `driver.value` 到数据库表相应的字段，（例如 int64 如何转换成 uint16 ）
- 把数据库查询结果转化成 `driver.Value`  。
- 在 `Scan` 函数里面如何把 `driver.Value` 值转化成用于定义的值。

### driver.Valuer

`Valuer` 接口定义了返回一个 `driver.Value` 的方法。

```go
type Valuer interface {
    Value() (Value, error)
}
```

很多类型都实现了这个 `Value` 方法，用来自身于 `driver.Value` 的转化。

### database/sql

database/sql 是 golang 的标准库之一，它在 `dataqbase/sql/driver` 提供的接口的基础上定义了一些更高阶的方法，用以简化数据库操作。

（它并不会提供数据库特有的方法，那些特有的方法交给数据库驱动去实现。）

database/sql库提供了一些 type。这些类型对掌握它的用法非常重要。

- DB :  数据库对象。

上面我了解到， `Open` 方法返回的 DB 对象，里面有一个 freeConn，它就是那个简易的连接池。他的实现相当简单或者说简陋，就是当执行 `DB.prepare` 的时候会 `defer db.putConn(ci, err)` ，也就是把这个连接放入连接池，每次调用 conn 的时候会先判断 freeConn 的长度是否大于 0，是则说明有可以复用的 conn，直接拿出来用；否则创建一个 conn ，然后再返回。

```go
type DB struct {
    driver driver.Driver
    dsn string
    mu sync.Mutex // protects freeConn and closed
    freeConn []driver.Conn
    closed bool
}
```

`sql.DB` 类型代表了数据库连接池。和其他语言不一样，它并不是数据库连接。golang 中的连接来自内部实现的连接池，连接的建立是惰性的，当你需要连接的时候，连接池会自动帮你创建。通常你不需要操作连接池。一切都有go来帮你完成。

它提供了一些跟数据库交互的函数，同时管理维护一个数据库连接池，帮你处理了单调而重复的管理工作，并且在多个goroutines也是十分安全。

sql.DB表示是数据库抽象，因此你有几个数据库就需要为每一个数据库创建一个sql.DB对象。**因为它维护了一个连接池，因此不需要频繁的创建和销毁。它需要长时间保持，因此最好是设置成一个全局变量以便其他代码可以访问。**

- Results: 结果集。

数据库查询的时候，都会有结果集。`sql.Rows` 类型表示查询返回多行数据的结果集。`sql.Row` 则表示单行查询结果的结果集。当然，对于插入更新和删除，返回的结果集类型为 `sql.Result` 。

- Statements:

语句。`sql.Stmt` 类型表示 SQL 查询语句，例如 DDL，DML 等类似的 SQL 语句。可以把当成 prepare 语句构造查询，也可以直接使用 `sql.DB` 的函数对其操作。

## go-sql-driver/mysql

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

- `db.SetMaxOpenConns(n int)`

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

## SQLite

SQLite 是一个开源的嵌入式关系型数据库，实现自包容、零配置、支持事务的 SQL 数据库引擎。

### 驱动

SQLite Administrator :  [http://sqliteadmin.orbmu2k.de/](http://sqliteadmin.orbmu2k.de/)

<< EOF >>
