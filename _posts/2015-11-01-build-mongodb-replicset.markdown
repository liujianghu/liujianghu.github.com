---
layout:     post
title:      "搭建Mongodb复制功能"
subtitle:   "在Centos 6.5上搭建MongoDb 3.0复制(读写分离)功能"
date:       2015-11-01 19:00:00
author:     "极品拖拉机"
header-img: "img/post-bg-01.jpg"
tags:
    - nosql
---

使用MongoDB有段时间了，之前因为项目请求量不大，数据也不大，所以一直是单实例。 但刚做的一个项目，当批量修改(1-10w)，会导致服务器CPU达到100%， 影响读取功能，搞了很就都没找到解决方法，最后就用了Mongodb Replic Set功能来实现读写分离。

## 准备环境

1. 准备2台服务器以上。如果只是实现复制功能，2台就足够了，但如果要做副本集，就需要另外加一台做Arbiter（裁判).
2. 3台服务器的IP分别是192.168.0.1,192.168.0.2,192.168.0.3， 其中0.1用来做master, 0.2, 0.3都用来做SECONDARY, 也可以拿一个来做Arbiter。

## 安装MongoDB

1. 下载，安装,参考  https://docs.mongodb.org/manual/tutorial/install-mongodb-on-red-hat/
  也可以直接下载执行包，直接运行。
2. 创建数据存储目录

```shell
    mkdir /data/mongodb
    mkdir /data/mongodb/data
    mkdir /data/mongodb/log
    chown -R mongod:mongod /data/mongodb/data
    nano /data/mongodb/log/mongod.log      #保存此文件
    chown -R mongod:mongod /data/mongod/log/mongod.log
```
3. 修改配置
   如果是执行第一步安装的，默认配置在/etc/mongod.conf. 如果是第二步安装的，则需要手动创建一个配置文件。

   ```yaml
    systemLog:
        destination: file
        logAppend: true
        quiet: true
        path: /data/mongodb/log/mongod.log
    storage:
        dbPath: /data/mongodb/data
        journal:
       enabled: true
     directoryPerDB: true
     engine: wiredTiger
     wiredTiger:
        engineConfig:
            cacheSizeGB: 5
        collectionConfig:
            blockCompressor: zlib
    processManagement:
     fork: true  # fork and run in background
     pidFilePath: /var/run/mongodb/mongod.pid  # location of pidfile
    net:
     port: 27017
    replication:
        oplogSizeMB: 500
        replSetName: rs0

   ```

## 配置Replic Set

1. 启动master
  在192.168.0.1上配置完成后，启动。（备注：我这里的时候，启动primary时，一定要先把secondary服务停掉，否则怎么都加不进来，被折腾了半天。）

```shell
    service mongod start   # (或者在下载的包里: bin/mongod -f mongod.conf)
```
2.  初始化Replic Set
  连接Mongodb:

```shell
  mongo --port 27017
```
  默认进入的是test数据库，执行如下命令：

```shell
  config = {
             "_id": "rs0",
             members: [
                        { "_id": 0,
                         "host": "192.168.0.1:27017"
                        }
                      ]
           }

     rs.initiate(config);

```
  这样就把本机加入到复制群里。 也可以直接执行rs.initiate();
3. 启动其他2台机器后，再回到0.1 Master机器，连接到mongodb后，执行以下命令，加入2台SECONDARY:

```shell
  rs.add("192.168.0.2");  
  rs.add("192.168.0.3");  # 如果是Arbiter则： rs.add("192.168.0.3", {arbiterOnly: true});

```
  然后执行rs.conf()可以看到其他的机器已经加入到replica set里。  执行rs.status()查看所有的状态。
  	如果正确的话，0.1,0.2的statusStr 应该是secodary.
  	修改各自的priority权重：

```shell
      var cfg=rs.conf()
	    cfg.members[0].priority = 4
	    rs.reconfig(cfg)
```
4. 测试同步
    
```shell

  use bookstore
	db.books.insert({"title":"男人和女人的故事", "elapsedms":600000});

```
  切换到任意secondary机器上，连接mongodb: mongod --port 27017
	默认从节点是不能执行查询操作的，执行如下命令：db.getMongo().setSlaveOk();
	再查询bookstore的数据是否已经同步过来。

## 启用安全认证
1. 先在primary上，创建一个超级用户：

```shell

  db.createUser(
    {
      user: "root",
      pwd: "root",
      roles: [
         { role: "dbAdmin", db: "admin" },
         { role: "dbOwner", db: "admin" },
         { role: "clusterAdmin", db: "admin" },
         { role: "userAdminAnyDatabase", db: "admin" }
      ]
    });

```
2. 停止各个mongodb服务。

```shell

    use admin
    db.shutdownServer();

```
3. 生成ssl  keyfile文件
  * 执行openssl rand -base64 753. 会生成很大串一个字符串。
  * 保存ssl： 复制保存到  /data/mongodb/key
  * 设置文件的权限: chmod 6000 /data/mongodb/key
  * 修改文件的属主: chown mongod:mongod /data/mongodb/key
  * 把此文件内容复制到另外2台从节点上去。
4.  修改mongod.conf配置启用安全认证

```shell
  security:
	authorization: enabled
  keyFile: /data/mongodb/key
  ```
5. 启动所有的节点

```shell
  mongodb --port 27017  #因为启用了安全认证，所有会提示没有权限。
	use admin
	db.auth("root","root");
	rs.status();     #会发现所有的服务器已经在线。
```

## c#连接
1. 写主的连接字符串： mongodb://bookuser:123456@192.168.0.1:27017/bookstore?replicaSet=mymongo&w=1
2. 优先从secondary读取：  
mongodb://bookuser:123456@192.168.0.2:27017/bookstore?replicaSet=mymongo&readPreference=secondaryPreferred
3. 注意： 如果是字符串连接放入web.config的话，需要对&符号进行转义：   &amp;
4. 关于连接字符串的配置，请参考：  https://docs.mongodb.org/manual/reference/connection-string/

#### 备注
1. 有时候mongodb 会提示Getting connection refused because too many open connections: 819
  * 修改ulimit -n
```shell
    nano /etc/security/limits.conf #添加如下：
    	*                soft    nofile          65535
    	*                hard    nofile          65535
```
  * 在mongodb.conf节点下增加 maxIncomingConnections: 5000
  * 因为mongodb3.0的语法是yaml, 所以每个格式后面都是加空格，不能按tab键。
  * 安装numactl命令： yum install -y numactl
  * 按第一步安装的时候，服务脚本里面已经加了numactl， 所以如果是手动启动的话，需要加上numactl:

```shell
    numactl --interleave=all  bin/mongod -f mongod.conf
```
2. 关于mongod配置文件的说明： https://docs.mongodb.org/manual/reference/configuration-options
3. mongodb创建用户的说明： https://docs.mongodb.org/manual/tutorial/manage-users-and-roles/
4. mongodb角色的说明： https://docs.mongodb.org/v3.0/reference/built-in-roles/#superuser-roles
5. c# 连接mongodb的一些文档
  mongo c# linq document: http://mongodb.github.io/mongo-csharp-driver/1.10/linq/
  mongo c# driver document: https://docs.mongodb.org/getting-started/csharp/query/
  mongo c# driver 2.0 document: http://mongodb.github.io/mongo-csharp-driver/2.0/what_is_new/
  https://mongodb-documentation.readthedocs.org/en/latest/ecosystem/tutorial/use-linq-queries-with-csharp-driver.html
