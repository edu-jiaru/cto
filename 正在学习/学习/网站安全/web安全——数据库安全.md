# 简介
数据安全是现在互联网安全非常重要一个环节。而且一旦数据出现问题是不可逆的，甚至是灾难性的。
>有一些防护措施应该在前面几个博文说过了，就不再赘述。比如通过防火墙控制，通过系统的用户控制，通过web应用的控制等。
>想说的是，任何一个节点都不是单独存在的。

# 场景
1. 确保应用本身安全。
2. 控制系统用户对数据库的访问权限。
3. 控制数据库用户对数据库的访问权限。
4. 确保数据库敏感数据的安全。
5. 确保数据库整个数据的完整性。
6. 规范日常运维操作
7. 合理的划分业务。

# 解决方案
## 应用安全
### 删除默认的数据库和用户
mysql初始化后会自动生成空用户和test库，这会对数据库构成威胁，我们全部删除。
```
mysql> drop database test;
mysql> use mysql;
mysql> delete from db;
mysql> delete from user where not(host=”localhost” and user=”root”);
mysql> flush privileges;
```

### 禁止数据库从本地直接加载内容
在某些情况下，LOCAL INFILE命令可被用于访问操作系统上的其它文件(如/etc/passwd)，应使用下现的命令：
```
mysql> LOAD DATA LOCAL INFILE '/etc/passwd' INTO TABLE table1
# 更简单的方法是：
mysql> SELECT load_file("/etc/passwd")
``` 
为禁用LOCAL INFILE命令，应当在MySQL配置文件的[mysqld]部分增加下面的参数：
set-variable=local-infile=0

## 控制用户的权限
这里用户，指的是数据库里的用户。
###  控制访问的ip。
只允许信任的ip访问，其他的ip都应该拒绝。
比如：只允许办公网络，还有业务服务器对应的网络可以访问。
### 区分角色
区分角色，给不同的权限。角色的划分需要根据具体的使用场景。
下面简单举例：
1. 角色：view。权限：只允许查询数据，不允许做任何修改。场景：业务正确性验证时
2. 角色：update。权限：允许修改数据，但是不允许修改数据结构。场景：程序运行
3. 角色：operate。权限：允许修改表结构，允许新增和修改表，不允许删除表，不允许删库。场景：产品要发布的时候才可以使用，通过升级sql方式执行。
4. .....

## 加密敏感信息
要使用md5,sha等算法加密。这样即使数据丢失，也能减少损失。比如：登录密码，支付密码等。


## 保证数据的完整性
1. 解决单点故障。主从，主主。
2. 需要备份与还原。

## 规范日常操作
1. 如果没有特殊需求，应该使用最小的用户。比如只使用查看的用户。
2. 有需要修改数据或者结构的操作，可以考虑两人一起。或者可以考虑做成功能，减少`人为直接操作数据库`。
3. 在测试环境上测试OK，才往正式环境执行。

## 业务的划分
### 少用数据库
可以通过缓存，静态化。尽可能少的使用数据库。能不使用数据库是最安全。
### 分库分表
敏感的数据和常用的数据，最好从表的设计上隔离。比如：用户的详情信息和支付信息最好分开。
### 优化sql
这个也非常重要，往往就是因为不重要sql的优化，所以数据库对应的服务器资源吃满不提供服务。

# 验证方法
通过不同的账号操作，判断有没有对应的权限。
# 参考资料
1. [保障MySQL安全的十四个最佳方法](http://dev.yesky.com/429/35432929.shtml)
2. [Mysql安全配置](http://drops.wooyun.org/tips/2245)
3. 《高性能MySql》