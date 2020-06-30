---
title: MongoDB 运行 js 脚本
key: 2020-06-12
tags: iPhone Mac socks
---



我们通常通过 Mongo  Shell 访问 MongoDB Server，在 shell 中执行指令以完成各种操作，比如说复制集初始化、用户添加等等。但是，在实际过程中运维过程中有些操作是固定常用的，类似这些操作我们可以将其写入 js 文件，在Linux的 shell 中执行 `mongo xxx.js` 这样指令完成我们的操作。



<!--more-->

### 添加用户

**1:  编写脚本：**

```javascript
// file name create-user.js
//

//var conn = new Mongo("t420i");
//var db = conn.getDB("admin")

var userName = "chengchao"

var console = console || {}
console.log = console.log || print
console.json = console.json || printjson
console.debug = console.debug || print

let dbAdmin = db.getSiblingDB("admin")
print(" ========= ")
var useExists = (dbAdmin.getUser(userName) != null)
if (useExists) {
    console.log("用户已经存在")
} else {
    printjson(dbAdmin.runCommand({
        createUser :  userName,
        pwd : "A12345^7b",
        roles : [{
            role : "root",
            db : "admin"
        }]
       
    }))
}


var users = dbAdmin.getUsers()

users.map(x => { 
    var u = x.user
    console.log("=>", u)
});

print(" ========= ")
```



**2: 运行命令：**

```bash
$ mongo --host=t420i create-user.js 
MongoDB shell version v4.2.6
connecting to: mongodb://t420i:27017/?compressors=disabled&gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("44ced561-219b-4f3b-ae21-700e9e44cf99") }
MongoDB server version: 4.2.1
...
```



需要特别注意的是在 Mongo Shell 中使用的 `use admin`、 `show  users`、`show collections` 指令以及类似的指令不能出现在 js 文件中。因为它不是` JavaScript` 语法，我们只能在 js 文件调用对应的方法去执行相应的操操作。



```javascript
// 切换数据库，类似于 use 操作
db.getSiblingDB("xxx")

// 在对应的数据库上执行指令
db.getSiblingDB("xxx").runCommand({xxxx})

// 倘若是需要在 admin 数据库上执行指令，更简单
db.adminCommand({xxx})
```



参考： 

- https://docs.mongodb.com/manual/mongo/



EOF

---

Power by TeXt.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>





