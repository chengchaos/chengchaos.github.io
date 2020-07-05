---
title: MongoDB Yaml config 实例
key: 2020-07-05
tags: mongo 
---

转自： https://www.cnblogs.com/guangshan/p/4842911.html



boolean 值为 `true` 或者 `false ，首字母不能大写

`systemLog.path` 为一个文件名，不能为一个文件夹，如果该文件已存在，会创建一个新的带日期的文件。有了这个基础，启动时就可以看启动日志文件来查看到底是哪里的配置有问题不能启动

`processManagement.pidFilePath` 为一个文件地址，不存在也没问题。

`net.ssl` 最好全部注释掉，暂时不用 ssl

`security` 下最好也注释掉，否则需要配置全部安全内容，以及 `keyFile` 一定要存在。

`storage.repairPath` 一定要是 `storage.dbPath` 的子目录，且目录需要存在。　　

`replication` 是复制，副本启动，注释掉

记得注释掉一个引擎

这样就可以启动了



<!--more-->

```yaml
systemLog:
   verbosity: 0
   quiet: false
#  traceAllExceptions: <boolean>
   syslogFacility: user
   path: "/data/mongodb/log"
   logAppend: false
   logRotate: rename
   destination: file
   timeStampFormat: iso8601-local
   component:
      accessControl:
         verbosity: 0
      command:
         verbosity: 0
      # COMMENT some component verbosity settings omitted for brevity
      storage:
         verbosity: 0
         journal:
            verbosity: 0
      write:
         verbosity: 0
processManagement:
   fork: false
   pidFilePath: "/var/run/mongodb/mongod.pid"
net:
   port: 27017
   bindIp: "127.0.0.1"
   maxIncomingConnections: 65536
   wireObjectCheck: true
   ipv6: false
   unixDomainSocket:
      enabled: true
      pathPrefix: "/tmp"
      filePermissions: 0700
   http:
      enabled: true
      JSONPEnabled: false
      RESTInterfaceEnabled: false
#   ssl:
#     sslOnNormalPorts: <boolean>  # deprecated since 2.6
#      mode: disabled
#      PEMKeyFile: <string>
#      PEMKeyPassword: <string>
#      clusterFile: <string>
#      clusterPassword: <string>
#      CAFile: <string>
#      CRLFile: <string>
#      allowConnectionsWithoutCertificates: <boolean>
#      allowInvalidCertificates: <boolean>
#      allowInvalidHostnames: <boolean>
#      FIPSMode: <boolean>
#security:
#   keyFile: "/var/lib/mongo/mongodb-keyfile"
#   clusterAuthMode: keyFile
#   authorization: disabled
#   javascriptEnabled:  true
#   sasl:
#      hostName: <string>
#      serviceName: <string>
#      saslauthdSocketPath: <string>
#setParameter:
#   <parameter1>: <value1>
#   <parameter2>: <value2>
storage:
   dbPath: "/data/db"
   indexBuildRetry: true
   repairPath: "/data/db/tmp"
   journal:
      enabled: true 
   directoryPerDB: false
   syncPeriodSecs: 60
   engine: mmapv1
   mmapv1:
      preallocDataFiles: true
      nsSize: 16
      quota:
         enforced: false
         maxFilesPerDB: 8
      smallFiles: false
      journal:
         debugFlags: 1
         commitIntervalMs: 100
#   wiredTiger:
#      engineConfig:
#         cacheSizeGB: 1
#         statisticsLogDelaySecs: 0
#         journalCompressor: snappy
#         directoryForIndexes: false
#      collectionConfig:
#         blockCompressor: snappy
#      indexConfig:
#         prefixCompression: true
operationProfiling:
   slowOpThresholdMs: 100
   mode: off
#replication:
#   oplogSizeMB: 50
#   replSetName: repl_test
#   secondaryIndexPrefetch: all
#sharding:
#   clusterRole: <string>
#   archiveMovedChunks: <boolean>
#auditLog:
#   destination: file
#   format: JSON
#   path: "/data/mongodb/log"
#   filter: <string>
#snmp:
#   subagent: <boolean>
#   master: <boolean>

#mongos only
#replication:
#   localPingThresholdMs: <boolean>
#sharding:
#   autoSplit: <boolean>
#   configDB: <string>
#   chunkSize: <int>
```



---

Power by TeXt.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>





