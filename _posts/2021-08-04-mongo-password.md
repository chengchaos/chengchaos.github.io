---
title: MongoDB 设置用户名密码
key: 2021-08--4
tags: mongo
---

## 超级管理员

### 设置管理员密码

```javascript
use admin
db.createUser({
    user: 'admin',
    pwd: '123456',
    roles: [{
        role: 'root',
        db: 'admin'
    }]
});
Successfully added user: {
        "user" : "admin",
        "roles" : [
                {
                        "role" : "root",
                        "db" : "admin"
                }
        ]
}

show users;
{
        "_id" : "admin.admin",
        "userId" : UUID("93397630-0c36-476f-9c0f-3982da23e540"),
        "user" : "admin",
        "db" : "admin",
        "roles" : [
                {
                        "role" : "root",
                        "db" : "admin"
                }
        ],
        "mechanisms" : [
                "SCRAM-SHA-1",
                "SCRAM-SHA-256"
        ]
}

```

### 开启验证

1, 停止 MongoDB
2, 修改配置文件, 添加:

```yaml
security:
  authorization: enabled
```

3, 启动 MongoDB.

### 修改密码

```javascript
use admin;
db.auth('admin', '123456');
1
db.changeUserPassword('admin', 'test123');
```

## 普通用户

我们除了可以设置数据库的超级管理员以外，还可以给每个数据库设置单独的管理员。其只有操作单独数据的一定权限

```javascript
use test;
db.createUser({
    user: 'foobar',
    pwd: 'VigWyBap]Rp_NNOKON1Ui7]3',
    roles:[{
        role: 'readWrite',
        db: 'test'
    }]
})
```

## 常用命令

```sh
show users  // 查看当前库下的用户

db.dropUser('testadmin')  // 删除用户

db.updateUser('admin', {pwd: '654321'})  // 修改用户密码

db.auth('admin', '654321')  // 密码认证
```

## MongoDB 数据库默认角色

- 数据库用户角色：read、readWrite
- 数据库管理角色：dbAdmin、dbOwner、userAdmin
- 集群管理角色：clusterAdmin、clusterManager、clusterMonitor、hostManager
- 备份恢复角色：backup、restore
- 所有数据库角色： readAnyDatabase、readWriteAnyDatabase、userAdminAnyDatabase、
dbAdminAnyDatabase
- 超级用户角色：root

EOF