---
title: 数据库中间件Mycat  
date: 2019-12-29 22:38:06  
tags:  
- Mycat  
---
### 概述  

#### 官方文档
> 官网：http://www.mycat.io/   
源码：https://github.com/MyCATApache/Mycat-Server   
下载地址：https://github.com/MyCATApache/Mycat-download 

#### 什么是Mycat？
> Mycat 是**数据库中间件**，就是介于数据库与应用之间，进行数据处理与交互的中间服务，它是一个开源的分布式数据库系统，是一个**实现了 MySQL 协议的的 
Server**。其核心功能是**分表分库**，即将一个大表水平分割为 N 个小表，存储在后端 MySQL 服务器里或者其他数据库里。  
对于架构师来说，Mycat是一个强大是**数据库中间件**，不仅可以用作**读写分离、以及分表分库，容灾备份**，而且可以用于多 
租户应用开发、云平台基础设施、让你的架构具备很强的适应性和灵活性。  

#### Mycat的原理？  
> Mycat 的原理中最重要的一个动词是“拦截”，它拦截了用户发送过来的 SQL 语句，首先对 SQL 语句做了 
一些特定的分析：如分片分析、路由分析、读写分离分析、缓存分析等，然后将此 SQL 发往后端的真实数据库， 
并将返回的结果做适当的处理，最终再返回给用户  

![image](https://note.youdao.com/yws/api/personal/file/BA66B38BF33E4D469E104F1C10BE7267?method=download&shareKey=7286c2faad2fb1d577aaa5729b342afd)  

#### 应用场景？  
- 单纯的读写分离
- 分表分库 （对于超过 1000 万的表进行分片，最大支持 1000 亿的单表分片）  
- 多租户应用 （每个应用一个库，但应用程序只连接 Mycat，从而不改造程序本身，实现多租户化）  
- 报表系统 （借助于 Mycat 的分表能力，处理大规模报表的统计）  
- 替代 Hbase，分析大数据
- 作为海量数据实时查询的一种简单有效方案  

### DEMO
> 使用Mycat搭建MySQL读写读写分离

##### **搭建主从MySQL服务**

```
# 使用docker-compose搭建

# Master
# docker-compose.yml
version: '3'
services:
  master:
    restart: always
    image: mysql:5.7.22
    container_name: master
    ports:
      - 3306:3306
    environment:
      TZ: Asia/Shanghai
      MYSQL_ROOT_PASSWORD: 123456
    command:
      --character-set-server=utf8mb4
      --collation-server=utf8mb4_general_ci
      --explicit_defaults_for_timestamp=true
      --lower_case_table_names=1
      --max_allowed_packet=128M
      --sql-mode="STRICT_TRANS_TABLES,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION,NO_ZERO_DATE,NO_ZERO_IN_DATE,ERROR_FOR_DIVISION_BY_ZERO"
    volumes:
      - ./conf/my.cnf:/etc/my.cnf
      - mysql-data:/var/lib/mysql
volumes:
  mysql-data:

# my.cnf
[mysqld]
user=mysql
default-storage-engine=INNODB
character-set-server=utf8
server-id=1   # 设置server-id，注意唯一
log-bin=mysql-bin   # 开启二进制日志功能，以备Slave作为其它Slave的Master时使用
binlog-ignore-db=mysql   # 设置忽略数据库
binlog-ignore-db=information_schema    
binlog-do-db=testdb   # 设置需要负责的数据库
binlog_format=STATEMENT
[client]
default-character-set=utf8
[mysql]
default-character-set=utf8

# Slave 
# docker-compose.yml 相同
# my.cnf
[mysqld]
user=mysql
default-storage-engine=INNODB
character-set-server=utf8
server-id=2   # server-id,唯一
relay-log=mysql-relay  # 启动中继日志  
[client]
default-character-set=utf8
[mysql]
default-character-set=utf8
```

##### **设置主从MySQL** 
- Master

```
# 创建数据同步用户，并授权（Slave使用该用户进行复制）  
CREATE USER 'slave'@'%' IDENTIFIED BY '123456';
GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'slave'@'%';  
# 查看master状态
show master status;
```
![image](https://note.youdao.com/yws/api/personal/file/C61B852927594DC0865B54B05BFBAA45?method=download&shareKey=d968103cbc05bff70d6c862a1916c8a8)  

- Slave

```
# 登录MySQL
mysql -uroot -p  

# 设置Master信息（执行下面语句）
change master to master_host='192.168.213.141', 
master_user='slave', 
master_password='123456', 
master_port=3306, 
master_log_file='mysql-bin.000001', 
master_log_pos= 154, 
master_connect_retry=30;

# 开启主从复制
start slave
# 查看同步状态
show slave status \G;
```
> **master_host**： Master的地址  
**master_port**：Master的端口号  
**master_user**：用于数据同步的用户  
**master_password**：用于同步的用户的密码  
**master_log_file**：指定 Slave 从哪个日志文件开始复制数据，即上面图片中提到的 File 字段的值  
**master_log_pos**：从哪个 Position 开始读，即上面图片的Position字段的值  
**master_connect_retry**：如果连接失败，重试的时间间隔，单位是秒，默认是60秒  

![image](https://note.youdao.com/yws/api/personal/file/E4B1471721084C63824C193ABD2EB493?method=download&shareKey=bc0785d65224ecef9fee445411cd404f)  

> SlaveIORunning 和 SlaveSQLRunning 都为yes则设置成功，否则查看下面的错误信息。  


##### 配置Mycat
> 详细配置请查看官网的权威文档

```
# 修改用户信息（conf/server.xml）
<user name="mycat">
    <property name="password">123456</property>
    <property name="schemas">TESTDB</property>
</user>

# 修改 conf/schema.xml 文件（只保留下面内容）
<?xml version="1.0"?>
<!DOCTYPE mycat:schema SYSTEM "schema.dtd">
<mycat:schema xmlns:mycat="http://io.mycat/">

        <schema name="TESTDB" checkSQLschema="false" sqlMaxLimit="100" dataNode="dn1" >
        </schema>
        <dataNode name="dn1" dataHost="host1" database="db1" />
        <dataHost name="host1" maxCon="1000" minCon="10" balance="3"
                          writeType="0" dbType="mysql" dbDriver="native" switchType="1"  slaveThreshold="100">
                <heartbeat>select user()</heartbeat>
                <!-- can have multi write hosts -->
                <writeHost host="hostM1" url="192.r168.213.141:3306" user="root"
                                   password="123456">
                        <!-- can have multi read hosts -->
                        <readHost host="hostS1" url="192.168.213.142:3306" user="root" password="123456" />
                </writeHost>
        </dataHost>
</mycat:schema>
```

##### 启动、登录Mycat

```
# 控制台启动
./mycat console

# 后台启动
./mycat start

# 登录数据窗口  
mysql -umycat -p123456 -P 8066 -h 192.168.213.140

# 登录后台管理窗口 
mysql -umycat -p123456 -P 9066 -h 192.168.213.140
```




