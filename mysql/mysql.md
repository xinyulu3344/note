

## MySQL目录结构

**查看数据库文件的存放路径：**

```sql
show variables like 'datadir';
+---------------+-----------------+
| Variable_name | Value           |
+---------------+-----------------+
| datadir       | /var/lib/mysql/ |
+---------------+-----------------+
1 row in set (0.03 sec)
```

**MySQL自带的数据库：**

| 数据库             | 用途                                                         |
| ------------------ | ------------------------------------------------------------ |
| mysql              | MySQL 系统自带的核心数据库，它存储了MySQL的用户账户和权限信息，一些存储过程、事件的定义信息，一些运行过程中产生的日志信息，一些帮助信息以及时区信息等。 |
| information_schema | MySQL 系统自带的数据库，这个数据库保存着MySQL服务器维护的所有其他数据库的信息，比如有哪些表、哪些视图、哪些触发器、哪些列、哪些索引。这些信息并不是真实的用户数据，而是一些描述性信息，有时候也称之为 元数据 。在系统数据库 information_schema 中提供了一些以innodb_sys 开头的表，用于表示内部系统表。 |
| performance_schema | 主要保存MySQL服务器运行过程中的一些状态信息，可以用来 监控 MySQL 服务的各类性能指标 。包括统计最近执行了哪些语句，在执行过程的每个阶段都花费了多长时间，内存的使用情况等信息。 |
| sys                | 主要是通过 视图 的形式把 information_schema 和 performance_schema 结合起来，帮助系统管理员和开发人员监控 MySQL 的技术性能。 |

**MySQL表在文件系统中的表示：**

| 存储引擎      | 文件                         | 作用                                                         | MySQL版本                    |
| ------------- | ---------------------------- | ------------------------------------------------------------ | ---------------------------- |
| InnoDB/MyISAM | /var/lib/mysql/库名/表名.frm | 描述表结构                                                   | MySQL8.0之后合并到了.ibd文件 |
| InnoDB        | /var/lib/mysql/库名/表名.ibd | 独立表空间，MySQL5.6之后的版本中使用独立表空间存储表中的数据和索引 |                              |
|               | /var/lib/mysql/库名/db.opt   | 存放字符集、比较规则等                                       | MySQL8.0之后合并到了.ibd文件 |
|               | /var/lib/mysql/ibdata1       | 系统表空间，MySQL5.6之前的版本，会将各个表的数据存储到系统表空间中。 |                              |
| MyISAM        | /var/lib/mysql/库名/表名.MYD | 存储数据MYData                                               |                              |
| MyISAM        | /var/lib/mysql/库名/表名.MYI | 存储索引                                                     |                              |

**查看和配置系统表空间和独立表空间**

```ini
[server]
nnodb_file_per_table=0 # 0：代表使用系统表空间； 1：代表使用独立表空间
```

```sql
show variables like 'innodb_file_per_table';
+-----------------------+-------+
| Variable_name         | Value |
+-----------------------+-------+
| innodb_file_per_table | ON    |
+-----------------------+-------+
1 row in set (0.02 sec)
```

## 用户与权限管理

