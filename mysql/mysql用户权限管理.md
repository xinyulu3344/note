# 用户权限管理

官方文档：https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html

# 1. 用户

## 1.1 用户的作用

登录管理数据库对象

## 1.2 用户在MySQL中的表现形式

```
'用户名'@'白名单'

'白名单': 允许哪些ip使用该用户名登录数据库

'白名单'允许的格式：

xinyulu@'10.0.0.%'    
xinyulu@'%'
xinyulu@'10.0.0.200'
xinyulu@'localhost'
xinyulu@'db02'
xinyulu@'10.0.0.5%'  // 表示10.0.0.50-10.0.0.59
xinyulu@'10.0.0.0/255.255.255.0'
```

## 1.3 用户的增删改查

```
增：
3306 [(none)]>create user xinyulu@'%' identified by '123';
Query OK, 0 rows affected (0.00 sec)
查：
3306 [(none)]> desc mysql.user;    ---->  authentication_string
3306 [(none)]> select user ,host ,authentication_string from mysql.user
改:
3306 [(none)]> alter user xinyulu@'%' identified by '456';
删：
3306 [(none)]> drop user xinyulu@'%';
```

# 2. 权限

## 2.1 常用权限介绍

```
ALL:
SELECT,INSERT, UPDATE, DELETE, CREATE, DROP, RELOAD, SHUTDOWN, PROCESS, FILE, REFERENCES, INDEX, ALTER, SHOW DATABASES, SUPER, CREATE TEMPORARY TABLES, LOCK TABLES, EXECUTE, REPLICATION SLAVE, REPLICATION CLIENT, CREATE VIEW, SHOW VIEW, CREATE ROUTINE, ALTER ROUTINE, CREATE USER, EVENT, TRIGGER, CREATE TABLESPACE
ALL : 以上所有权限，一般是普通管理员拥有的
with grant option：超级管理员才具备的，给别的用户授权的功能
```

## 2.2 权限管理

### 2.2.1 授予权限

```
grant 权限 on 允许操作的对象 to 用户 identified by 用户密码;

grant: 授权命令
权限: 要授予的权限
允许操作的对象: *.*、库.*、库.表
用户: '用户'@'白名单'
用户密码: 相当于把这个用户的登录密码改了

示例:
grant all on *.* to 'xinyulu'@'%' identified by '333';
```

### 2.2.2 查看权限

```
show grants for 用户;

例如：
show grants for xinyulu@'%';
```

### 2.2.3 回收权限

```
revoke 权限 on 权限对象 from 用户;

例如：
revoke select on app.* from 'xinyulu'@'%';
```

### 2.2.4 8.0新特性

```
建用户和授权分开了
grant 不再支持自动创建用户了，不支持改密码
授权之前，必须要提前创建用户。
```

### 2.2.5 root密码忘了怎么办

1. 先停止mysql服务

```
systemctl stop mysqld
```

2. 不加载权限表，关闭tcp连接mysql功能，启动mysql

```
mysqld_safe --skip-grant-tables --skip-networking &
```

3. 将权限表刷入内存

```
flush privileges;
```

4. 修改密码

```
alter user root@'localhost' identified by '123456';
```

5. 重启mysql

```
pkill mysqld
systemctl start  mysqld
```