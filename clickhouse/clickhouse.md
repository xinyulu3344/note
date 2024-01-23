# clickhouse

## 部署

### 环境准备

hosts文件配置

```bash
cat >> /etc/hosts << EOF
192.168.30.8 node1
192.168.30.9 node2
192.168.30.10 node3
192.168.30.11 node4
EOF
```



操作系统语言显示环境

```bash
echo "export LANG=en_US.UTF8" >> ~/.bash_profile
cat ~/.bash_profile
```

准备文件系统

```bash
yum install lvm2 -y
pvcreate /dev/vdb
vgcreate datavg /dev/vdb
lvcreate -n datalv -L 200000M datavg
mkfs.ext4 /dev/datavg/datalv
```

挂载

```bash
mkdir /clickhouse
echo "UUID=xxx /clickhouse ext4 defaults,nobarrier,noatime 0 0" >> /etc/fstab
mount -a
mkdir /clickhouse/soft
```

创建用户、组，授权

```bash
groupadd -g 60001 clickhouse
useradd -u 61001 -g clickhouse clickhouse
chown -R clickhouse:clickhouse /clickhouse
chmod -R 775 /clickhouse
echo "clickhouse" | passwd --stdin clickhouse
```

启动级别修改

```bash
systemctl set-default multi-user.target
```

修改资源限制参数

```bash
cat >> /etc/security/limits.conf << EOF
root soft nofile 1048576
root hard nofile 1048576
clickhouse soft nproc 1048576
clickhouse hard nproc 1048576
clickhouse soft nofile 1048576
clickhouse hard nofile 1048576
clickhouse soft stack 10240
clickhouse hard stack 32768
clickhouse soft memlock unlimited
clickhouse hard memlock unlimited
EOF
```

修改内核参数

```bash
cat >> /etc/sysctl.conf << EOF
fs.aio-max-nr = 1048576
fs.file-max = 6815744
net.ipv4.ip_local_port_range = 10000    65535
net.ipv4.tcp_mem = 786432 2097152 3145728
net.ipv4.tcp_rmem = 4096 4096 16777216
net.ipv4.tcp_wmem = 4096 4096 16777216
vm.swappiness = 1
vm.min_free_kbytes = 204800
vm.overcommit_memory = 0
EOF

sysctl -p
```

修改时区

```bash
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

禁用selinux

关闭防火墙

关闭透明大页(Huge Pages)

```bash
cat /sys/kernel/mm/transparent_hugepage/defrag
cat /sys/kernel/mm/transparent_hugepage/enabled

# vi /etc/rc.d/rc.local
if test -f /sys/kernel/mm/transparent_hugepage/defrag; then
    echo never > /sys/kernel/mm/transparent_hugepage/defrag
fi

if test -f /sys/kernel/mm/transparent_hugepage/enabled; then
    echo never > /sys/kernel/mm/transparent_hugepage/enabled
fi

chmod -x /etc/rc.d/rc.local
```

### 单机版部署

创建目录

```bash
su - clickhouse
mkdir /clickhouse/{app,data,log,etc,soft}
```

上传软件包到`/clickhouse/soft`

解压改名

```bash
tar zxf /clickhouse/soft/clickhouse-common-static-23.3.2.37-amd64.tgz
tar zxf /clickhouse/soft/clickhouse-common-static-dbg-23.3.2.37-amd64.tgz
tar zxf /clickhouse/soft/clickhouse-server-23.3.2.37-amd64.tgz
tar zxf /clickhouse/soft/clickhouse-client-23.3.2.37-amd64.tgz
mv /clickhouse/soft/clickhouse-common-static-23.3.2.37 /clickhouse/app/clickhouse-common-static
mv /clickhouse/soft/clickhouse-common-static-dbg-23.3.2.37 /clickhouse/app/clickhouse-common-static-dbg
mv /clickhouse/soft/clickhouse-server-23.3.2.37 /clickhouse/app/clickhouse-server
mv /clickhouse/soft/clickhouse-client-23.3.2.37 /clickhouse/app/clickhouse-client

```

修改`clickhouse-server/install/doinst.sh`配置

```bash
CLICKHOUSE_DATADIR=${CLICKHOUSE_DATADIR:=/clickhouse/data}
CLICKHOUSE_LOGDIR=${CLICKHOUSE_LOGDIR:=/clickhouse/log}
```

修改`/clickhouse/app/clickhouse-server/etc/clickhouse-server/config.xml`配置

```xml
<clickhouse>
    <logger>
        <log>/clickhouse/log/clickhouse-server.log</log>
        <errorlog>/clickhouse/log/clickhouse-server.err.log</errorlog>
    </logger>
    <path>/clickhouse/data</path>
    <tmp_path>/clickhouse/data/tmp/</tmp_path>
    <user_files_path>/clickhouse/data/user_files/</user_files_path>
    <format_schema_path>/clickhouse/data/format_schemas/</format_schema_path>
    <user_directories>
        <local_directory>
            <path>/clickhouse/data/access/</path>
        </local_directory>
    </user_directories>
    <listen_host>::</listen_host>
    <timezone>Asia/Shanghai</timezone>
</clickhouse>

```

修改`/clickhouse/app/clickhouse-server/etc/clickhouse-server/users.xml `配置

```xml
<clickhouse>
    <users>
        <default>
            <access_management>1</access_management>
        </default>
    </users>
</clickhouse>
```

root执行安装

```bash
/clickhouse/app/clickhouse-common-static/install/doinst.sh
/clickhouse/app/clickhouse-common-static-dbg/install/doinst.sh
/clickhouse/app/clickhouse-server/install/doinst.sh
/clickhouse/app/clickhouse-client/install/doinst.sh
```

软链接配置文件

```bash
cd /etc
mv /etc/clickhouse-* /clickhouse/etc/
ln -sf /clickhouse/etc/* .
```

检查参数

```bash
/clickhouse/etc/clickhouse-server/config.xml
/clickhouse/etc/clickhouse-server/user.xml
```

修改权限

```bash
chown -R clickhouse:clickhouse /clickhouse
chmod -R 755 /clickhouse
```

启动服务

systemd启动

```bash
systemctl status clickhouse-server.service
systemctl start clickhouse-server.service

# 停止服务
systemctl stop clickhouse-server.service
```

手工启动

```bash
nohup clickhouse-server --config=/etc/clickhouse-server/config.xml &

# 停止服务：kill
```

客户端登录

```bash
clickhouse-client -h 192.168.30.8 --port 9000 --user default --password
```

### 多实例环境部署

配置config.xml文件

```bash
cp /clickhouse/etc/clickhouse-server/config.xml /clickhouse/etc/clickhouse-server/config9200.xml
```

修改`config9200.xml`

```xml
<clickhouse>
    <logger>
        <log>/clickhouse/log9200/clickhouse-server.log</log>
        <errorlog>/clickhouse/log9200/clickhouse-server.err.log</errorlog>
    </logger>
    <http_port>8223</http_port>
    <tcp_port>9200</tcp_port>
    <mysql_port>9204</mysql_port>
    <postgresql_port>9205</postgresql_port>
    <interserver_http_port>9209</interserver_http_port>
    
    <path>/clickhouse/data200</path>
    <tmp_path>/clickhouse/data9200/tmp/</tmp_path>
    <user_files_path>/clickhouse/data9200/user_files/</user_files_path>
    <format_schema_path>/clickhouse/data9200/format_schemas/</format_schema_path>
    <user_directories>
        <local_directory>
            <path>/clickhouse/data9200/access/</path>
        </local_directory>
    </user_directories>
    <listen_host>::</listen_host>
    <timezone>Asia/Shanghai</timezone>
</clickhouse>
```

创建对应的目录

```bash
mkdir -p /clickhouse/data9200
mkdir -p /clickhouse/log9200
chown -R clickhouse:clickhouse /clickhouse/data9200
chown -R clickhouse:clickhouse /clickhouse/log9200
```

启动

```bash
nohup clickhouse-server --config=/etc/clickhouse-server/config9200.xml &
```

客户端登录

```bash
clickhouse-client -h 192.168.30.8 --port 9200 --user default --password
```

### 分布式集群部署（分片）

MergeTree + Distribute

修改`config.xml`

```bash
vim /clickhouse/etc/clickhouse-server/config.xml
```

```xml
<clickhouse>
    <remote_servers incl="clickhouse_remote_servers">
    </remote_servers>
    <include_from>/clickhouse/etc/clickhouse-server/config.d/metrika.xml</include_from>
</clickhouse>
```

创建并添加`metrika.xml`

```bash
vim /clickhouse/etc/clickhouse-server/config.d/metrika.xml
```

```xml
<clickhouse>
    <clickhouse_remote_servers>
        <!--集群名称-->
        <ck_cluster1>
            <!--定义分片节点-->
            <shard>
                <internal_replication>false</internal_replication>
                <replica>
                    <host>node1</host>
                    <port>9000</port>
                </replica>
            </shard>
            <shard>
                <internal_replication>false</internal_replication>
                <replica>
                    <host>node2</host>
                    <port>9000</port>
                </replica>
            </shard>
            <shard>
                <internal_replication>false</internal_replication>
                <replica>
                    <host>node3</host>
                    <port>9000</port>
                </replica>
            </shard>
        </ck_cluster1>
    </clickhouse_remote_servers>
    <clickhouse_compression>
        <case>
            <min_part_size>10000000000</min_part_size>
            <min_part_size_ratio>0.01</min_part_size_ratio>
            <method>lz4</method>
        </case>
    </clickhouse_compression>
</clickhouse>
```

**clickhouse四种数据复制模式：**

非复制表：

- internal_replication = false：

  写分布式表：会被插入到所有副本的本地表中，副本表上的数据保持同步。写入单机表，不同服务器查询结果不同

- internal_replication = true

  写分布式表：数据只插入到一个本地表中，数据无法保证准确性，这种是不正确的配置方式，非复制表，不建议使用

复制表：

- internal_replication = false

  写分布式表/本地表：会被插入到副本的其中一个本地表中，会通过复制机制复制数据到另一个副本表上，数据保持同步。官方推荐

- internal_replication = true

  写分布式表：会被插入到副本的所有的本地表中㎡数据会重复2俱通过复制表的机制会删除重复数据，对性能造成大的影响。

打包

```bash
tar -zcf clickhouse.tar.gz app data etc log
```

拷贝到所有主机

删除数据目录

```bash
rm -rf /clickhouse/data/*
rm -rf /clickhouse/data9200/*
```

执行安装

```bash
/clickhouse/app/clickhouse-common-static/install/doinst.sh
/clickhouse/app/clickhouse-common-static-dbg/install/doinst.sh
/clickhouse/app/clickhouse-server/install/doinst.sh
/clickhouse/app/clickhouse-client/install/doinst.sh
```

修改权限

```bash
chown -R clickhouse:clickhouse /clickhouse
chmod -R 755 /clickhouse
```

软链接配置文件

```bash
cd /etc
rm -rf /etc/clickhouse-*
ln -sf /clickhouse/etc/* .
```

启动服务

```bash
nohup /usr/bin/clickhouse-server --config=/etc/clickhouse-server/config.xml &
```

集群验证

```bash
clickhouse-client -h 192.168.30.8 --port 9000 --user default --query "select * from system.clusters;" 
```

```bash
┌─cluster─────────┬─shard_num─┬─shard_weight─┬─replica_num─┬─host_name──┬─host_address──┬─port─┬─is_local─┬─user────┬─default_database─┬─errors_count─┬─slowdowns_count─┬─estimated_recovery_time─┐
│ ck_cluster1     │         1 │            1 │           1 │ node1      │ 192.168.30.8  │ 9000 │        1 │ default │                  │            0 │               0 │                       0 │
│ ck_cluster1     │         2 │            1 │           1 │ node2      │ 192.168.30.9  │ 9000 │        0 │ default │                  │            0 │               0 │                       0 │
│ ck_cluster1     │         3 │            1 │           1 │ node3      │ 192.168.30.10 │ 9000 │        0 │ default │         
```

MergeTree建表语法

```sql
create table [if not exists] [db.]table_name [on cluster_name cluster]
(
    name1 [type1] [default|materialized|alias expr1],
    name2 [type2] [default|materialized|alias expr2],
    ...
    index index_name1 expr1 type type1(...) granularity value1,
    index index_name2 expr2 type type2(...) granularyity value2
) engine = MergeTree()
[partition by expr]
[order by expr]
[primary key expr]
[sample by expr]
[settings name=value,...]
```

MergeTree关键字

| 关键字       | 解释                                                         |
| ------------ | ------------------------------------------------------------ |
| engine       | 引擎名称和参数                                               |
| partition by | 分区字段                                                     |
| order by     | 表的排序键                                                   |
| primary key  | 主键                                                         |
| sample by    | 采样表达式                                                   |
| ttl          | 定义行数据的存储时间，设置值的生命周期。既可以为整张表设置，也可以为每个列单独设置。如果ttl同时作用于表和字段，Clickhouse会使用先到期的那个 |

### 分布式集群部署（分片+副本复制+zk）

MergeTree + Distribute

安装JDK

```bash
su - clickhouse
mkdir -p /clickhouse/soft
mkdir -p /clickhouse/app
cat >> ~/.bash_profile << EOF
export JAVA_HOME=/clickhouse/jdk1.8.0_391
export PATH=/clickhouse/jdk1.8.0_391/bin:$PATH
export LANG=en_US.UTF8
EOF
source ~/.bash_profile

cd /clickhouse
tar -zxf /clickhouse/soft/jdk-8u391-linux-x64.tar.gz
java -version
```

安装zookeeper

```bash
su - clickhouse
cd /clickhouse/
tar -zxf /clickhouse/soft/apache-zookeeper-3.6.2-bin.tar.gz
mv apache-zookeeper-3.6.2-bin/ zookeeper
```

配置zookeeper

```bash
mkdir -p /clickhouse/zookeeper/data
mkdir -p /clickhouse/zookeeper/log
cd /clickhouse/zookeeper/conf
cp zoo_sample.cfg zoo.cfg

vi /clickhouse/zookeeper/conf/zoo.cfg
tickTime=2000
initLimit=30000
syncLimit=10
maxClientCnxns=2000
maxSessionTimeout=60000000
autopurge.snapRetainCount=10
autopurge.purgeInterval=1
globalOutstandingLimit=200
preAllocSize=131072
snapCount=3000000
leaderServes=yes
dataDir=/clickhouse/zookeeper/data
dataLogDir=/clickhouse/zookeeper/log
clientPort=2181
server.1=192.168.30.8:2888:3888
server.2=192.168.30.9:2888:3888
server.3=192.168.30.10:2888:3888

# 修改JVM内存参数，默认1G(生产改成内存的一半，不要超过32G)
vi /clickhouse/zookeeper/bin/zkEnv.sh
ZK_SERVER_HEAP="${ZK_SERVER_HEAP:-1000}"
```

将zoo.cfg和zkEnv.sh拷贝到其它节点

创建myid文件

```bash
# 3个节点分别创建
echo 1 > /clickhouse/zookeeper/data/myid
echo 2 > /clickhouse/zookeeper/data/myid
echo 3 > /clickhouse/zookeeper/data/myid
```

配置环境变量

```bash
cat >> ~/.bash_profile << EOF
export PATH=$PATH:/clickhouse/zookeeper/bin
EOF
source ~/.bash_profile
```

启动zk

```bash
zkServer.sh start
zkServer.sh stop
zkServer.sh restart
zkServer.sh status
jps
```

测试连接zk

```bash
zkCli.sh -server 192.168.30.8:2181
zkCli.sh -server 192.168.30.9:2181
zkCli.sh -server 192.168.30.10:2181
```

配置`/clickhouse/etc/clickhouse-server/config.d/metrika.xml`

```xml
<clickhouse>
    <clickhouse_remote_servers>
        <!--集群名称-->
        <ck_cluster01>
            <!--定义分片节点-->
            <shard>
                <internal_replication>true</internal_replication>
                <replica>
                    <host>node1</host>
                    <port>9000</port>
                </replica>
                <replica>
                    <host>node2</host>
                    <port>9000</port>
                </replica>
            </shard>
            <shard>
                <internal_replication>true</internal_replication>
                <replica>
                    <host>node3</host>
                    <port>9000</port>
                </replica>
                <replica>
                    <host>node4</host>
                    <port>9000</port>
                </replica>
            </shard>
        </ck_cluster01>
    </clickhouse_remote_servers>
    <zookeeper>
        <node index="1">
            <host>node1</host>
            <port>2181</port>
        </node>
        <node index="2">
            <host>node2</host>
            <port>2181</port>
        </node>
        <node index="3">
            <host>node3</host>
            <port>2181</port>
        </node>
    </zookeeper>
    <macros>
        <layer>01</layer>
        <shard>01</shard>
        <replica>ck_cluster01-01-01</replica>
    </macros>
    <networks>
        <ip>::/0</ip>
    </networks>
    <clickhouse_compression>
        <case>
            <min_part_size>10000000000</min_part_size>
            <min_part_size_ratio>0.01</min_part_size_ratio>
            <method>lz4</method>
        </case>
    </clickhouse_compression>
</clickhouse>
```

将`/clickhouse/etc/clickhouse-server/config.d/metrika.xml`拷贝到其它节点

每个节点修改macros配置

启动服务

```
┌─cluster─┬─shard_num─┬─shard_weight─┬─replica_num─┬─host_name────────────────────┬─host_address──┬─port─┬─is_local─┬─user────┬─default_database─┬─errors_count─┬─slowdowns_count─┬─estimated_recovery_time─┬─database_shard_name─┬─database_replica_name─┬─is_active─┐
│ default │         1 │            1 │           1 │ chi-terrabase-9hab0axfr8-0-0 │ 127.0.0.1     │ 9000 │        1 │ default │                  │            0 │               0 │                       0 │                     │                       │      ᴺᵁᴸᴸ │
│ default │         1 │            1 │           2 │ chi-terrabase-9hab0axfr8-0-1 │ 10.233.46.26  │ 9000 │        0 │ default │                  │            0 │               0 │                       0 │                     │                       │      ᴺᵁᴸᴸ │
│ default │         1 │            1 │           3 │ chi-terrabase-9hab0axfr8-0-2 │ 10.233.42.246 │ 9000 │        0 │ default │                  │            0 │               0 │                       0 │                     │                       │      ᴺᵁᴸᴸ │
│ default │         1 │            1 │           4 │ chi-terrabase-9hab0axfr8-0-3 │ 10.233.62.193 │ 9000 │        0 │ default │                  │            0 │               0 │                       0 │                     │                       │      ᴺᵁᴸᴸ │
│ default │         1 │            1 │           5 │ chi-terrabase-9hab0axfr8-0-4 │ 10.233.15.136 │ 9000 │        0 │ default │                  │            0 │               0 │                       0 │                     │                       │      ᴺᵁᴸᴸ │
│ default │         1 │            1 │           6 │ chi-terrabase-9hab0axfr8-0-5 │ 10.233.6.200  │ 9000 │        0 │ default │                  │            0 │               0 │                       0 │                     │                       │      ᴺᵁᴸᴸ │
│ default │         1 │            1 │           7 │ chi-terrabase-9hab0axfr8-0-6 │ 10.233.39.168 │ 9000 │        0 │ default │                  │            0 │               0 │                       0 │                     │                       │      ᴺᵁᴸᴸ │
│ default │         1 │            1 │           8 │ chi-terrabase-9hab0axfr8-0-7 │ 10.233.62.216 │ 9000 │        0 │ default │                  │            0 │               0 │                       0 │                     │                       │      ᴺᵁᴸᴸ │
└─────────┴───────────┴──────────────┴─────────────┴──────────────────────────────┴───────────────┴──────┴──────────┴─────────┴──────────────────┴──────────────┴─────────────────┴─────────────────────────┴─────────────────────┴───────────────────────┴───────────┘
```

## 

