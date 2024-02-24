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

## 配置文件管理

- config.xml
- metrika.xml
- users.xml

### config.xml

```xml
<clickhouse>
    <logger>
        <!-- Possible levels [1]:
          - none (turns off logging)
          - fatal
          - critical
          - error
          - warning
          - notice
          - information
          - debug
          - trace
          - test (not for production usage)
        -->
        <level>trace</level>

        <!--日志文件, 包含所有日志条目-->
        <log>/clickhouse/log/clickhouse-server.log</log>

        <!--错误日志文件-->
        <errorlog>/clickhouse/log/clickhouse-server.err.log</errorlog>

        <!--日志文件大小。文件达到设置大小后，Clickhouse将对其存档并重命名，并创建一个新的日志文件-->
        <size>1000M</size>

        <!--已存档日志文件的数量上限-->
        <count>10</count>

        <!-- <console>1</console> --> <!-- Default behavior is autodetection (log to console if not daemon mode and is tty) -->

        <!--日志格式-->
        <formatting>
            <type>json</type>
            <names>
                <date_time>date_time</date_time>
                <thread_name>thread_name</thread_name>
                <thread_id>thread_id</thread_id>
                <level>level</level>
                <query_id>query_id</query_id>
                <logger_name>logger_name</logger_name>
                <message>message</message>
                <source_file>source_file</source_file>
                <source_line>source_line</source_line>
            </names>
        </formatting>
    </logger>


    <!--通过HTTP连接到服务器的端口。-->
    <http_port>8123</http_port>

    <!--通过TCP与客户端进行通信的端口。-->
    <tcp_port>9000</tcp_port>

    <!--通过MySQL协议与客户端通信的端口-->
    <mysql_port>9004</mysql_port>

    <!--通过pg协议与客户端通信的端口-->
    <postgresql_port>9005</postgresql_port>

    <!--通过https协议连接到ck-->
    <!-- <https_port>8443</https_port> -->

    <!--通过TCP协议与客户端进行通信的端口。与OpenSSL设置一起使用-->
    <!-- <tcp_port_secure>9440</tcp_port_secure> -->

    <!-- <tcp_with_proxy_port>9011</tcp_with_proxy_port> -->

    <!--Clickhouse服务器之间交换数据的端口-->
    <interserver_http_port>9009</interserver_http_port>
    <!-- <interserver_https_port>9010</interserver_https_port> -->

    <!--其他服务器可以用来访问该服务器的主机名。如果省略，则其定义方法与hostname -f命令相同。-->
    <!--<interserver_http_host>example.clickhouse.com</interserver_http_host>-->

    <!--副本之间通信的用户名密码, 默认不使用-->
    <!--<interserver_http_credentials>
        <user>interserver</user>
        <password></password>
    </interserver_http_credentials>-->

    <listen_host>::</listen_host>

    <!-- <interserver_listen_host>::</interserver_listen_host> -->

    <!-- <listen_try>0</listen_try> -->

    <!-- <listen_reuse_port>0</listen_reuse_port> -->

    <!-- <listen_backlog>4096</listen_backlog> -->

    <!--最大连接数-->
    <max_connections>4096</max_connections>

    <!--Clickhouse在关闭连接之前等待传入请求的秒数。默认3s-->
    <keep_alive_timeout>3</keep_alive_timeout>

    <!-- gRPC protocol (see src/Server/grpc_protos/clickhouse_grpc.proto for the API) -->
    <!-- <grpc_port>9100</grpc_port> -->
    <grpc>
        <enable_ssl>false</enable_ssl>

        <!-- The following two files are used only if enable_ssl=1 -->
        <ssl_cert_file>/path/to/ssl_cert_file</ssl_cert_file>
        <ssl_key_file>/path/to/ssl_key_file</ssl_key_file>

        <!-- Whether server will request client for a certificate -->
        <ssl_require_client_auth>false</ssl_require_client_auth>

        <!-- The following file is used only if ssl_require_client_auth=1 -->
        <ssl_ca_cert_file>/path/to/ssl_ca_cert_file</ssl_ca_cert_file>

        <!-- Default transport compression type (can be overridden by client, see the transport_compression_type field in QueryInfo).
             Supported algorithms: none, deflate, gzip, stream_gzip -->
        <transport_compression_type>none</transport_compression_type>

        <!-- Default transport compression level. Supported levels: 0..3 -->
        <transport_compression_level>0</transport_compression_level>

        <!-- Send/receive message size limits in bytes. -1 means unlimited -->
        <max_send_message_size>-1</max_send_message_size>
        <max_receive_message_size>-1</max_receive_message_size>

        <!-- Enable if you want very detailed logs -->
        <verbose_logs>false</verbose_logs>
    </grpc>

    <!--SSL客户端/服务端配置-->
    <openSSL>
        <server>
            <!-- openssl req -subj "/CN=localhost" -new -newkey rsa:2048 -days 365 -nodes -x509 -keyout /etc/clickhouse-server/server.key -out /etc/clickhouse-server/server.crt -->

            <!--PEM格式的客户端/服务器证书文件路径。如果privateKeyFile包含证书，则可以忽略它。-->
            <!-- <certificateFile>/etc/clickhouse-server/server.crt</certificateFile>-->

            <!--私钥路径-->
            <!--<privateKeyFile>/etc/clickhouse-server/server.key</privateKeyFile> -->

            <!--openssl dhparam -out /etc/clickhouse-server/dhparam.pem 4096-->
            <!--<dhParamsFile>/etc/clickhouse-server/dhparam.pem</dhParamsFile>-->

            <!--检查节点证书的方法。
                 none:
                 relaxed:
                 strict:
                 once:
            -->
            <verificationMode>none</verificationMode>

            <!--是否使用OpenSSL内置的CA证书。true/false-->
            <loadDefaultCAFile>true</loadDefaultCAFile>

            <!--启用或禁用缓存会话。必须与sessionIdContext结合使用。true/false-->
            <cacheSessions>true</cacheSessions>

            <!--不允许使用的协议-->
            <disableProtocols>sslv2,sslv3</disableProtocols>

            <!--首选服务器密码-->
            <preferServerCiphers>true</preferServerCiphers>
        </server>

        <client>
            <!--是否使用OpenSSL内置的CA证书。true/false-->
            <loadDefaultCAFile>true</loadDefaultCAFile>

            <!--启用或禁用缓存会话。必须与sessionIdContext结合使用。true/false-->
            <cacheSessions>true</cacheSessions>

            <!--不允许使用的协议-->
            <disableProtocols>sslv2,sslv3</disableProtocols>

            <!--首选服务器密码-->
            <preferServerCiphers>true</preferServerCiphers>

            <!---->
            <invalidCertificateHandler>
                <!-- Use for self-signed: <name>AcceptCertificateHandler</name> -->
                <name>RejectCertificateHandler</name>
            </invalidCertificateHandler>
        </client>
    </openSSL>

    <!-- Default root page on http[s] server. For example load UI from https://tabix.io/ when opening http://localhost:8123 -->
    <!--
    <http_server_default_response><![CDATA[<html ng-app="SMI2"><head><base href="http://ui.tabix.io/"></head><body><div ui-view="" class="content-ui"></div><script src="http://loader.tabix.io/master.js"></script></body></html>]]></http_server_default_response>
    -->

    <!-- The maximum number of query processing threads, excluding threads for retrieving data from remote servers, allowed to run all queries.
         This is not a hard limit. In case if the limit is reached the query will still get at least one thread to run.
         Query can upscale to desired number of threads during execution if more threads become available.
    -->
    <concurrent_threads_soft_limit_num>0</concurrent_threads_soft_limit_num>
    <concurrent_threads_soft_limit_ratio_to_cores>0</concurrent_threads_soft_limit_ratio_to_cores>

    <!--同时处理的最大请求数-->
    <max_concurrent_queries>100</max_concurrent_queries>

    <!--最大内存使用量。0为不限制-->
    <max_server_memory_usage>0</max_server_memory_usage>

    <!--最大线程池-->
    <max_thread_pool_size>10000</max_thread_pool_size>

    <!-- Configure other thread pools: -->
    <!--
    <background_buffer_flush_schedule_pool_size>16</background_buffer_flush_schedule_pool_size>
    <background_pool_size>16</background_pool_size>
    <background_merges_mutations_concurrency_ratio>2</background_merges_mutations_concurrency_ratio>
    <background_merges_mutations_scheduling_policy>round_robin</background_merges_mutations_scheduling_policy>
    <background_move_pool_size>8</background_move_pool_size>
    <background_fetches_pool_size>8</background_fetches_pool_size>
    <background_common_pool_size>8</background_common_pool_size>
    <background_schedule_pool_size>128</background_schedule_pool_size>
    <background_message_broker_schedule_pool_size>16</background_message_broker_schedule_pool_size>
    <background_distributed_schedule_pool_size>16</background_distributed_schedule_pool_size>
    -->

    <!--可用于Clickhouse的，服务器的总物理RAM量的比例-->
    <max_server_memory_usage_to_ram_ratio>0.9</max_server_memory_usage_to_ram_ratio>

    <!-- Simple server-wide memory profiler. Collect a stack trace at every peak allocation step (in bytes).
         Data will be stored in system.trace_log table with query_id = empty string.
         Zero means disabled.
      -->
    <total_memory_profiler_step>4194304</total_memory_profiler_step>

    <!-- Collect random allocations and deallocations and write them into system.trace_log with 'MemorySample' trace_type.
         The probability is for every alloc/free regardless to the size of the allocation.
         Note that sampling happens only when the amount of untracked memory exceeds the untracked memory limit,
          which is 4 MiB by default but can be lowered if 'total_memory_profiler_step' is lowered.
         You may want to set 'total_memory_profiler_step' to 1 for extra fine grained sampling.
      -->
    <total_memory_tracker_sample_probability>0</total_memory_tracker_sample_probability>

    <!--打开的最大文件数, 默认最大值-->
    <!-- <max_open_files>262144</max_open_files> -->

    <!--表引擎从MergeTree使用的未压缩数据的缓存大小（以字节为单位，8G）。服务器有一个共享缓存，内存是按需分配的。如果启用，则使用高速缓存。在个别情况下，未压缩的缓存对于非常短的查询是有利的。-->
    <uncompressed_cache_size>8589934592</uncompressed_cache_size>

    <!--标记缓存的大小，用于MergeTree系列的表中。  以字节为单位，共享服务器的缓存，并根据需要分配内存。缓存大小必须至少为5368709120（5G）。 -->
    <mark_cache_size>5368709120</mark_cache_size>


    <!-- If you enable the `min_bytes_to_use_mmap_io` setting,
         the data in MergeTree tables can be read with mmap to avoid copying from kernel to userspace.
         It makes sense only for large files and helps only if data reside in page cache.
         To avoid frequent open/mmap/munmap/close calls (which are very expensive due to consequent page faults)
         and to reuse mappings from several threads and queries,
         the cache of mapped files is maintained. Its size is the number of mapped regions (usually equal to the number of mapped files).
         The amount of data in mapped files can be monitored
         in system.metrics, system.metric_log by the MMappedFiles, MMappedFileBytes metrics
         and in system.asynchronous_metrics, system.asynchronous_metrics_log by the MMapCacheCells metric,
         and also in system.events, system.processes, system.query_log, system.query_thread_log, system.query_views_log by the
         CreatedReadBufferMMap, CreatedReadBufferMMapFailed, MMappedFileCacheHits, MMappedFileCacheMisses events.
         Note that the amount of data in mapped files does not consume memory directly and is not accounted
         in query or server memory usage - because this memory can be discarded similar to OS page cache.
         The cache is dropped (the files are closed) automatically on removal of old parts in MergeTree,
         also it can be dropped manually by the SYSTEM DROP MMAP CACHE query.
      -->
    <mmap_cache_size>1000</mmap_cache_size>

    <!-- Cache size in bytes for compiled expressions.-->
    <compiled_expression_cache_size>134217728</compiled_expression_cache_size>

    <!-- Cache size in elements for compiled expressions.-->
    <compiled_expression_cache_elements_size>10000</compiled_expression_cache_elements_size>

    <!--数据目录的路径-->
    <path>/clickhouse/data/node1</path>

    <!-- Multi-disk configuration example: -->
    <!--
    <storage_configuration>
        <disks>
            <default>
                <keep_free_space_bytes>0</keep_free_space_bytes>
            </default>
            <data>
                <path>/data/</path>
                <keep_free_space_bytes>0</keep_free_space_bytes>
            </data>
            <s3>
                <type>s3</type>
                <endpoint>http://path/to/endpoint</endpoint>
                <access_key_id>your_access_key_id</access_key_id>
                <secret_access_key>your_secret_access_key</secret_access_key>
            </s3>
            <blob_storage_disk>
                <type>azure_blob_storage</type>
                <storage_account_url>http://account.blob.core.windows.net</storage_account_url>
                <container_name>container</container_name>
                <account_name>account</account_name>
                <account_key>pass123</account_key>
                <metadata_path>/var/lib/clickhouse/disks/blob_storage_disk/</metadata_path>
                <cache_enabled>true</cache_enabled>
                <cache_path>/var/lib/clickhouse/disks/blob_storage_disk/cache/</cache_path>
                <skip_access_check>false</skip_access_check>
            </blob_storage_disk>
        </disks>

        <policies>
            <all>
                <volumes>
                    <main>
                        <disk>default</disk>
                        <disk>data</disk>
                        <disk>s3</disk>
                        <disk>blob_storage_disk</disk>

                        <max_data_part_size_bytes></max_data_part_size_bytes>
                        <max_data_part_size_ratio></max_data_part_size_ratio>
                        <perform_ttl_move_on_insert>true</perform_ttl_move_on_insert>
                        <prefer_not_to_merge>false</prefer_not_to_merge>
                        <load_balancing>round_robin</load_balancing>
                    </main>
                </volumes>
                <move_factor>0.2</move_factor>
            </all>
        </policies>
    </storage_configuration>
    -->


    <!--用于处理大型查询的临时数据的路径-->
    <tmp_path>/clickhouse/data/node1/tmp/</tmp_path>

    <!-- Disable AuthType plaintext_password and no_password for ACL. -->
    <allow_plaintext_password>1</allow_plaintext_password>
    <allow_no_password>1</allow_no_password>
    <allow_implicit_no_password>1</allow_implicit_no_password>

    <!-- Complexity requirements for user passwords. -->
    <!-- <password_complexity>
        <rule>
            <pattern>.{12}</pattern>
            <message>be at least 12 characters long</message>
        </rule>
        <rule>
            <pattern>\p{N}</pattern>
            <message>contain at least 1 numeric character</message>
        </rule>
        <rule>
            <pattern>\p{Ll}</pattern>
            <message>contain at least 1 lowercase character</message>
        </rule>
        <rule>
            <pattern>\p{Lu}</pattern>
            <message>contain at least 1 uppercase character</message>
        </rule>
        <rule>
            <pattern>[^\p{L}\p{N}]</pattern>
            <message>contain at least 1 special character</message>
        </rule>
    </password_complexity> -->

    <!-- Policy from the <storage_configuration> for the temporary files.
         If not set <tmp_path> is used, otherwise <tmp_path> is ignored.

         Notes:
         - move_factor              is ignored
         - keep_free_space_bytes    is ignored
         - max_data_part_size_bytes is ignored
         - you must have exactly one volume in that policy
    -->
    <!-- <tmp_policy>tmp</tmp_policy> -->

    <!--包含用户文件的目录，在表函数file()中使用-->
    <user_files_path>/clickhouse/data/node1/user_files</user_files_path>

    <!-- LDAP server definitions. -->
    <ldap_servers>
        <!-- List LDAP servers with their connection parameters here to later 1) use them as authenticators for dedicated local users,
              who have 'ldap' authentication mechanism specified instead of 'password', or to 2) use them as remote user directories.
             Parameters:
                host - LDAP server hostname or IP, this parameter is mandatory and cannot be empty.
                port - LDAP server port, default is 636 if enable_tls is set to true, 389 otherwise.
                bind_dn - template used to construct the DN to bind to.
                        The resulting DN will be constructed by replacing all '{user_name}' substrings of the template with the actual
                         user name during each authentication attempt.
                user_dn_detection - section with LDAP search parameters for detecting the actual user DN of the bound user.
                        This is mainly used in search filters for further role mapping when the server is Active Directory. The
                         resulting user DN will be used when replacing '{user_dn}' substrings wherever they are allowed. By default,
                         user DN is set equal to bind DN, but once search is performed, it will be updated with to the actual detected
                         user DN value.
                    base_dn - template used to construct the base DN for the LDAP search.
                            The resulting DN will be constructed by replacing all '{user_name}' and '{bind_dn}' substrings
                             of the template with the actual user name and bind DN during the LDAP search.
                    scope - scope of the LDAP search.
                            Accepted values are: 'base', 'one_level', 'children', 'subtree' (the default).
                    search_filter - template used to construct the search filter for the LDAP search.
                            The resulting filter will be constructed by replacing all '{user_name}', '{bind_dn}', and '{base_dn}'
                             substrings of the template with the actual user name, bind DN, and base DN during the LDAP search.
                            Note, that the special characters must be escaped properly in XML.
                verification_cooldown - a period of time, in seconds, after a successful bind attempt, during which a user will be assumed
                         to be successfully authenticated for all consecutive requests without contacting the LDAP server.
                        Specify 0 (the default) to disable caching and force contacting the LDAP server for each authentication request.
                enable_tls - flag to trigger use of secure connection to the LDAP server.
                        Specify 'no' for plain text (ldap://) protocol (not recommended).
                        Specify 'yes' for LDAP over SSL/TLS (ldaps://) protocol (recommended, the default).
                        Specify 'starttls' for legacy StartTLS protocol (plain text (ldap://) protocol, upgraded to TLS).
                tls_minimum_protocol_version - the minimum protocol version of SSL/TLS.
                        Accepted values are: 'ssl2', 'ssl3', 'tls1.0', 'tls1.1', 'tls1.2' (the default).
                tls_require_cert - SSL/TLS peer certificate verification behavior.
                        Accepted values are: 'never', 'allow', 'try', 'demand' (the default).
                tls_cert_file - path to certificate file.
                tls_key_file - path to certificate key file.
                tls_ca_cert_file - path to CA certificate file.
                tls_ca_cert_dir - path to the directory containing CA certificates.
                tls_cipher_suite - allowed cipher suite (in OpenSSL notation).
             Example:
                <my_ldap_server>
                    <host>localhost</host>
                    <port>636</port>
                    <bind_dn>uid={user_name},ou=users,dc=example,dc=com</bind_dn>
                    <verification_cooldown>300</verification_cooldown>
                    <enable_tls>yes</enable_tls>
                    <tls_minimum_protocol_version>tls1.2</tls_minimum_protocol_version>
                    <tls_require_cert>demand</tls_require_cert>
                    <tls_cert_file>/path/to/tls_cert_file</tls_cert_file>
                    <tls_key_file>/path/to/tls_key_file</tls_key_file>
                    <tls_ca_cert_file>/path/to/tls_ca_cert_file</tls_ca_cert_file>
                    <tls_ca_cert_dir>/path/to/tls_ca_cert_dir</tls_ca_cert_dir>
                    <tls_cipher_suite>ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:AES256-GCM-SHA384</tls_cipher_suite>
                </my_ldap_server>
             Example (typical Active Directory with configured user DN detection for further role mapping):
                <my_ad_server>
                    <host>localhost</host>
                    <port>389</port>
                    <bind_dn>EXAMPLE\{user_name}</bind_dn>
                    <user_dn_detection>
                        <base_dn>CN=Users,DC=example,DC=com</base_dn>
                        <search_filter>(&amp;(objectClass=user)(sAMAccountName={user_name}))</search_filter>
                    </user_dn_detection>
                    <enable_tls>no</enable_tls>
                </my_ad_server>
        -->
    </ldap_servers>

    <!-- To enable Kerberos authentication support for HTTP requests (GSS-SPNEGO), for those users who are explicitly configured
          to authenticate via Kerberos, define a single 'kerberos' section here.
         Parameters:
            principal - canonical service principal name, that will be acquired and used when accepting security contexts.
                    This parameter is optional, if omitted, the default principal will be used.
                    This parameter cannot be specified together with 'realm' parameter.
            realm - a realm, that will be used to restrict authentication to only those requests whose initiator's realm matches it.
                    This parameter is optional, if omitted, no additional filtering by realm will be applied.
                    This parameter cannot be specified together with 'principal' parameter.
         Example:
            <kerberos />
         Example:
            <kerberos>
                <principal>HTTP/clickhouse.example.com@EXAMPLE.COM</principal>
            </kerberos>
         Example:
            <kerberos>
                <realm>EXAMPLE.COM</realm>
            </kerberos>
    -->

    <!--定义用户配置文件-->
    <user_directories>
        <users_xml>
            <!--预定义用户配置文件路径-->
            <path>users.xml</path>
        </users_xml>
        <local_directory>
            <!--由SQL命令创建的用户目录路径-->
            <path>/clickhouse/data/node1/format_schemas</path>
        </local_directory>

        <!-- To add an LDAP server as a remote user directory of users that are not defined locally, define a single 'ldap' section
              with the following parameters:
                server - one of LDAP server names defined in 'ldap_servers' config section above.
                        This parameter is mandatory and cannot be empty.
                roles - section with a list of locally defined roles that will be assigned to each user retrieved from the LDAP server.
                        If no roles are specified here or assigned during role mapping (below), user will not be able to perform any
                         actions after authentication.
                role_mapping - section with LDAP search parameters and mapping rules.
                        When a user authenticates, while still bound to LDAP, an LDAP search is performed using search_filter and the
                         name of the logged in user. For each entry found during that search, the value of the specified attribute is
                         extracted. For each attribute value that has the specified prefix, the prefix is removed, and the rest of the
                         value becomes the name of a local role defined in ClickHouse, which is expected to be created beforehand by
                         CREATE ROLE command.
                        There can be multiple 'role_mapping' sections defined inside the same 'ldap' section. All of them will be
                         applied.
                    base_dn - template used to construct the base DN for the LDAP search.
                            The resulting DN will be constructed by replacing all '{user_name}', '{bind_dn}', and '{user_dn}'
                             substrings of the template with the actual user name, bind DN, and user DN during each LDAP search.
                    scope - scope of the LDAP search.
                            Accepted values are: 'base', 'one_level', 'children', 'subtree' (the default).
                    search_filter - template used to construct the search filter for the LDAP search.
                            The resulting filter will be constructed by replacing all '{user_name}', '{bind_dn}', '{user_dn}', and
                             '{base_dn}' substrings of the template with the actual user name, bind DN, user DN, and base DN during
                             each LDAP search.
                            Note, that the special characters must be escaped properly in XML.
                    attribute - attribute name whose values will be returned by the LDAP search. 'cn', by default.
                    prefix - prefix, that will be expected to be in front of each string in the original list of strings returned by
                             the LDAP search. Prefix will be removed from the original strings and resulting strings will be treated
                             as local role names. Empty, by default.
             Example:
                <ldap>
                    <server>my_ldap_server</server>
                    <roles>
                        <my_local_role1 />
                        <my_local_role2 />
                    </roles>
                    <role_mapping>
                        <base_dn>ou=groups,dc=example,dc=com</base_dn>
                        <scope>subtree</scope>
                        <search_filter>(&amp;(objectClass=groupOfNames)(member={bind_dn}))</search_filter>
                        <attribute>cn</attribute>
                        <prefix>clickhouse_</prefix>
                    </role_mapping>
                </ldap>
             Example (typical Active Directory with role mapping that relies on the detected user DN):
                <ldap>
                    <server>my_ad_server</server>
                    <role_mapping>
                        <base_dn>CN=Users,DC=example,DC=com</base_dn>
                        <attribute>CN</attribute>
                        <scope>subtree</scope>
                        <search_filter>(&amp;(objectClass=group)(member={user_dn}))</search_filter>
                        <prefix>clickhouse_</prefix>
                    </role_mapping>
                </ldap>
        -->
    </user_directories>

    <access_control_improvements>
        <!-- Enables logic that users without permissive row policies can still read rows using a SELECT query.
             For example, if there two users A, B and a row policy is defined only for A, then
             if this setting is true the user B will see all rows, and if this setting is false the user B will see no rows.
             By default this setting is false for compatibility with earlier access configurations. -->
        <users_without_row_policies_can_read_rows>false</users_without_row_policies_can_read_rows>

        <!-- By default, for backward compatibility ON CLUSTER queries ignore CLUSTER grant,
             however you can change this behaviour by setting this to true -->
        <on_cluster_queries_require_cluster_grant>false</on_cluster_queries_require_cluster_grant>

        <!-- By default, for backward compatibility "SELECT * FROM system.<table>" doesn't require any grants and can be executed
             by any user. You can change this behaviour by setting this to true.
             If it's set to true then this query requires "GRANT SELECT ON system.<table>" just like as for non-system tables.
             Exceptions: a few system tables ("tables", "columns", "databases", and some constant tables like "one", "contributors")
             are still accessible for everyone; and if there is a SHOW privilege (e.g. "SHOW USERS") granted the corresponding system
             table (i.e. "system.users") will be accessible. -->
        <select_from_system_db_requires_grant>false</select_from_system_db_requires_grant>

        <!-- By default, for backward compatibility "SELECT * FROM information_schema.<table>" doesn't require any grants and can be
             executed by any user. You can change this behaviour by setting this to true.
             If it's set to true then this query requires "GRANT SELECT ON information_schema.<table>" just like as for ordinary tables. -->
        <select_from_information_schema_requires_grant>false</select_from_information_schema_requires_grant>

        <!-- By default, for backward compatibility a settings profile constraint for a specific setting inherit every not set field from
             previous profile. You can change this behaviour by setting this to true.
             If it's set to true then if settings profile has a constraint for a specific setting, then this constraint completely cancels all
             actions of previous constraint (defined in other profiles) for the same specific setting, including fields that are not set by new constraint.
             It also enables 'changeable_in_readonly' constraint type -->
        <settings_constraints_replace_previous>false</settings_constraints_replace_previous>

        <!-- Number of seconds since last access a role is stored in the Role Cache -->
        <role_cache_expiration_time_seconds>600</role_cache_expiration_time_seconds>
    </access_control_improvements>

    <!-- Default profile of settings. -->
    <default_profile>default</default_profile>

    <!-- Comma-separated list of prefixes for user-defined settings. -->
    <custom_settings_prefixes></custom_settings_prefixes>

    <!-- System profile of settings. This settings are used by internal processes (Distributed DDL worker and so on). -->
    <!-- <system_profile>default</system_profile> -->

    <!-- Buffer profile of settings.
         This settings are used by Buffer storage to flush data to the underlying table.
         Default: used from system_profile directive.
    -->
    <!-- <buffer_profile>default</buffer_profile> -->

    <!--默认数据库-->
    <default_database>default</default_database>

    <!-- Server time zone could be set here.

         Time zone is used when converting between String and DateTime types,
          when printing DateTime in text formats and parsing DateTime from text,
          it is used in date and time related functions, if specific time zone was not passed as an argument.

         Time zone is specified as identifier from IANA time zone database, like UTC or Africa/Abidjan.
         If not specified, system time zone at server startup is used.

         Please note, that server could display time zone alias instead of specified name.
         Example: Zulu is an alias for UTC.
    -->
    <!--时区配置-->
    <timezone>Asia/Shanghai</timezone>

    <!-- You can specify umask here (see "man umask"). Server will apply it on startup.
         Number is always parsed as octal. Default umask is 027 (other users cannot read logs, data files, etc; group can only read).
    -->
    <!-- <umask>022</umask> -->

    <!-- Perform mlockall after startup to lower first queries latency
          and to prevent clickhouse executable from being paged out under high IO load.
         Enabling this option is recommended but will lead to increased startup time for up to a few seconds.
    -->
    <mlock_executable>true</mlock_executable>

    <!-- Reallocate memory for machine code ("text") using huge pages. Highly experimental. -->
    <remap_executable>false</remap_executable>

    <![CDATA[
         Uncomment below in order to use JDBC table engine and function.

         To install and run JDBC bridge in background:
         * [Debian/Ubuntu]
           export MVN_URL=https://repo1.maven.org/maven2/com/clickhouse/clickhouse-jdbc-bridge/
           export PKG_VER=$(curl -sL $MVN_URL/maven-metadata.xml | grep '<release>' | sed -e 's|.*>\(.*\)<.*|\1|')
           wget https://github.com/ClickHouse/clickhouse-jdbc-bridge/releases/download/v$PKG_VER/clickhouse-jdbc-bridge_$PKG_VER-1_all.deb
           apt install --no-install-recommends -f ./clickhouse-jdbc-bridge_$PKG_VER-1_all.deb
           clickhouse-jdbc-bridge &

         * [CentOS/RHEL]
           export MVN_URL=https://repo1.maven.org/maven2/com/clickhouse/clickhouse-jdbc-bridge/
           export PKG_VER=$(curl -sL $MVN_URL/maven-metadata.xml | grep '<release>' | sed -e 's|.*>\(.*\)<.*|\1|')
           wget https://github.com/ClickHouse/clickhouse-jdbc-bridge/releases/download/v$PKG_VER/clickhouse-jdbc-bridge-$PKG_VER-1.noarch.rpm
           yum localinstall -y clickhouse-jdbc-bridge-$PKG_VER-1.noarch.rpm
           clickhouse-jdbc-bridge &

         Please refer to https://github.com/ClickHouse/clickhouse-jdbc-bridge#usage for more information.
    ]]>
    <!--
    <jdbc_bridge>
        <host>127.0.0.1</host>
        <port>9019</port>
    </jdbc_bridge>
    -->

    <!--远程服务器，分布式表引擎和集群表功能使用的集群的配置。-->
    <remote_servers incl="clickhouse_remote_servers">
    </remote_servers>

    <!-- The list of hosts allowed to use in URL-related storage engines and table functions.
        If this section is not present in configuration, all hosts are allowed.
    -->
    <!--<remote_url_allow_hosts>-->
        <!-- Host should be specified exactly as in URL. The name is checked before DNS resolution.
            Example: "clickhouse.com", "clickhouse.com." and "www.clickhouse.com" are different hosts.
                    If port is explicitly specified in URL, the host:port is checked as a whole.
                    If host specified here without port, any port with this host allowed.
                    "clickhouse.com" -> "clickhouse.com:443", "clickhouse.com:80" etc. is allowed, but "clickhouse.com:80" -> only "clickhouse.com:80" is allowed.
            If the host is specified as IP address, it is checked as specified in URL. Example: "[2a02:6b8:a::a]".
            If there are redirects and support for redirects is enabled, every redirect (the Location field) is checked.
            Host should be specified using the host xml tag:
                    <host>clickhouse.com</host>
        -->

        <!-- Regular expression can be specified. RE2 engine is used for regexps.
            Regexps are not aligned: don't forget to add ^ and $. Also don't forget to escape dot (.) metacharacter
            (forgetting to do so is a common source of error).
        -->
    <!--</remote_url_allow_hosts>-->

    <!--metrika.xml的路径-->
    <include_from>/clickhouse/etc/clickhouse-server/config.d/metrika.xml</include_from>


    <!-- Reloading interval for embedded dictionaries, in seconds. Default: 3600. -->
    <builtin_dictionaries_reload_interval>3600</builtin_dictionaries_reload_interval>


    <!--最大会话超时，单位s-->
    <max_session_timeout>3600</max_session_timeout>

    <!--默认会话超时，单位s-->
    <default_session_timeout>60</default_session_timeout>

    <!-- Sending data to Graphite for monitoring. Several sections can be defined. -->
    <!--
        interval - send every X second
        root_path - prefix for keys
        hostname_in_path - append hostname to root_path (default = true)
        metrics - send data from table system.metrics
        events - send data from table system.events
        asynchronous_metrics - send data from table system.asynchronous_metrics
    -->
    <!--
    <graphite>
        <host>localhost</host>
        <port>42000</port>
        <timeout>0.1</timeout>
        <interval>60</interval>
        <root_path>one_min</root_path>
        <hostname_in_path>true</hostname_in_path>

        <metrics>true</metrics>
        <events>true</events>
        <events_cumulative>false</events_cumulative>
        <asynchronous_metrics>true</asynchronous_metrics>
    </graphite>
    <graphite>
        <host>localhost</host>
        <port>42000</port>
        <timeout>0.1</timeout>
        <interval>1</interval>
        <root_path>one_sec</root_path>

        <metrics>true</metrics>
        <events>true</events>
        <events_cumulative>false</events_cumulative>
        <asynchronous_metrics>false</asynchronous_metrics>
    </graphite>
    -->

    <!-- Serve endpoint for Prometheus monitoring. -->
    <!--
        endpoint - mertics path (relative to root, statring with "/")
        port - port to setup server. If not defined or 0 than http_port used
        metrics - send data from table system.metrics
        events - send data from table system.events
        asynchronous_metrics - send data from table system.asynchronous_metrics
        status_info - send data from different component from CH, ex: Dictionaries status
    -->
    <!--
    <prometheus>
        <endpoint>/metrics</endpoint>
        <port>9363</port>

        <metrics>true</metrics>
        <events>true</events>
        <asynchronous_metrics>true</asynchronous_metrics>
        <status_info>true</status_info>
    </prometheus>
    -->

    <!-- Query log. Used only for queries with setting log_queries = 1. -->
    <query_log>
        <database>system</database>
        <table>query_log</table>
        <!--
            PARTITION BY expr: https://clickhouse.com/docs/en/table_engines/mergetree-family/custom_partitioning_key/
            Example:
                event_date
                toMonday(event_date)
                toYYYYMM(event_date)
                toStartOfHour(event_time)
        -->
        <partition_by>toYYYYMM(event_date)</partition_by>
        <!--
            Table TTL specification: https://clickhouse.com/docs/en/engines/table-engines/mergetree-family/mergetree/#mergetree-table-ttl
            Example:
                event_date + INTERVAL 1 WEEK
                event_date + INTERVAL 7 DAY DELETE
                event_date + INTERVAL 2 WEEK TO DISK 'bbb'

        <ttl>event_date + INTERVAL 30 DAY DELETE</ttl>
        -->

        <!-- Instead of partition_by, you can provide full engine expression (starting with ENGINE = ) with parameters,
             Example: <engine>ENGINE = MergeTree PARTITION BY toYYYYMM(event_date) ORDER BY (event_date, event_time) SETTINGS index_granularity = 1024</engine>
          -->

        <!--将数据从内存中的缓冲区刷新到表的时间间隔-->
        <flush_interval_milliseconds>7500</flush_interval_milliseconds>

        <!-- example of using a different storage policy for a system table -->
        <!-- storage_policy>local_ssd</storage_policy -->
    </query_log>

    <!-- Trace log. Stores stack traces collected by query profilers.
         See query_profiler_real_time_period_ns and query_profiler_cpu_time_period_ns settings. -->
    <trace_log>
        <database>system</database>
        <table>trace_log</table>

        <partition_by>toYYYYMM(event_date)</partition_by>
        <flush_interval_milliseconds>7500</flush_interval_milliseconds>
    </trace_log>

    <!-- Query thread log. Has information about all threads participated in query execution.
         Used only for queries with setting log_query_threads = 1. -->
    <query_thread_log>
        <database>system</database>
        <table>query_thread_log</table>
        <partition_by>toYYYYMM(event_date)</partition_by>
        <flush_interval_milliseconds>7500</flush_interval_milliseconds>
    </query_thread_log>

    <!-- Query views log. Has information about all dependent views associated with a query.
         Used only for queries with setting log_query_views = 1. -->
    <query_views_log>
        <database>system</database>
        <table>query_views_log</table>
        <partition_by>toYYYYMM(event_date)</partition_by>
        <flush_interval_milliseconds>7500</flush_interval_milliseconds>
    </query_views_log>

    <!-- Uncomment if use part log.
         Part log contains information about all actions with parts in MergeTree tables (creation, deletion, merges, downloads).-->
    <part_log>
        <database>system</database>
        <table>part_log</table>
        <partition_by>toYYYYMM(event_date)</partition_by>
        <flush_interval_milliseconds>7500</flush_interval_milliseconds>
    </part_log>

    <!-- Uncomment to write text log into table.
         Text log contains all information from usual server log but stores it in structured and efficient way.
         The level of the messages that goes to the table can be limited (<level>), if not specified all messages will go to the table.
    <text_log>
        <database>system</database>
        <table>text_log</table>
        <flush_interval_milliseconds>7500</flush_interval_milliseconds>
        <level></level>
    </text_log>
    -->

    <!-- Metric log contains rows with current values of ProfileEvents, CurrentMetrics collected with "collect_interval_milliseconds" interval. -->
    <metric_log>
        <database>system</database>
        <table>metric_log</table>
        <flush_interval_milliseconds>7500</flush_interval_milliseconds>
        <collect_interval_milliseconds>1000</collect_interval_milliseconds>
    </metric_log>

    <!--
        Asynchronous metric log contains values of metrics from
        system.asynchronous_metrics.
    -->
    <asynchronous_metric_log>
        <database>system</database>
        <table>asynchronous_metric_log</table>
        <flush_interval_milliseconds>7000</flush_interval_milliseconds>
    </asynchronous_metric_log>

    <!--
        OpenTelemetry log contains OpenTelemetry trace spans.
    -->
    <opentelemetry_span_log>
        <!--
            The default table creation code is insufficient, this <engine> spec
            is a workaround. There is no 'event_time' for this log, but two times,
            start and finish. It is sorted by finish time, to avoid inserting
            data too far away in the past (probably we can sometimes insert a span
            that is seconds earlier than the last span in the table, due to a race
            between several spans inserted in parallel). This gives the spans a
            global order that we can use to e.g. retry insertion into some external
            system.
        -->
        <engine>
            engine MergeTree
            partition by toYYYYMM(finish_date)
            order by (finish_date, finish_time_us, trace_id)
        </engine>
        <database>system</database>
        <table>opentelemetry_span_log</table>
        <flush_interval_milliseconds>7500</flush_interval_milliseconds>
    </opentelemetry_span_log>


    <!-- Crash log. Stores stack traces for fatal errors.
         This table is normally empty. -->
    <crash_log>
        <database>system</database>
        <table>crash_log</table>

        <partition_by />
        <flush_interval_milliseconds>1000</flush_interval_milliseconds>
    </crash_log>

    <!-- Session log. Stores user log in (successful or not) and log out events.

        Note: session log has known security issues and should not be used in production.
    -->
    <!-- <session_log>
        <database>system</database>
        <table>session_log</table>

        <partition_by>toYYYYMM(event_date)</partition_by>
        <flush_interval_milliseconds>7500</flush_interval_milliseconds>
    </session_log> -->

    <!-- Profiling on Processors level. -->
    <processors_profile_log>
        <database>system</database>
        <table>processors_profile_log</table>

        <partition_by>toYYYYMM(event_date)</partition_by>
        <flush_interval_milliseconds>7500</flush_interval_milliseconds>
    </processors_profile_log>

    <!-- Log of asynchronous inserts. It allows to check status
         of insert query in fire-and-forget mode.
    -->
    <asynchronous_insert_log>
        <database>system</database>
        <table>asynchronous_insert_log</table>

        <flush_interval_milliseconds>7500</flush_interval_milliseconds>
        <partition_by>event_date</partition_by>
        <ttl>event_date + INTERVAL 3 DAY</ttl>
    </asynchronous_insert_log>

    <!-- <top_level_domains_path>/var/lib/clickhouse/top_level_domains/</top_level_domains_path> -->
    <!-- Custom TLD lists.
         Format: <name>/path/to/file</name>

         Changes will not be applied w/o server restart.
         Path to the list is under top_level_domains_path (see above).
    -->
    <top_level_domains_lists>
        <!--
        <public_suffix_list>/path/to/public_suffix_list.dat</public_suffix_list>
        -->
    </top_level_domains_lists>

    <!-- Configuration of external dictionaries. See:
         https://clickhouse.com/docs/en/sql-reference/dictionaries/external-dictionaries/external-dicts
    -->
    <dictionaries_config>*_dictionary.xml</dictionaries_config>

    <!-- Configuration of user defined executable functions -->
    <user_defined_executable_functions_config>*_function.xml</user_defined_executable_functions_config>

    <!-- Path in ZooKeeper to store user-defined SQL functions created by the command CREATE FUNCTION.
     If not specified they will be stored locally. -->
    <!-- <user_defined_zookeeper_path>/clickhouse/user_defined<user_defined_zookeeper_path> -->

    <!-- Uncomment if you want data to be compressed 30-100% better.
         Don't do that if you just started using ClickHouse.
      -->
    <!--
    <compression>
        <!- - Set of variants. Checked in order. Last matching case wins. If nothing matches, lz4 will be used. - ->
        <case>

            <!- - Conditions. All must be satisfied. Some conditions may be omitted. - ->
            <min_part_size>10000000000</min_part_size>        <!- - Min part size in bytes. - ->
            <min_part_size_ratio>0.01</min_part_size_ratio>   <!- - Min size of part relative to whole table size. - ->

            <!- - What compression method to use. - ->
            <method>zstd</method>
        </case>
    </compression>
    -->

    <!-- Configuration of encryption. The server executes a command to
         obtain an encryption key at startup if such a command is
         defined, or encryption codecs will be disabled otherwise. The
         command is executed through /bin/sh and is expected to write
         a Base64-encoded key to the stdout. -->
    <encryption_codecs>
        <!-- aes_128_gcm_siv -->
            <!-- Example of getting hex key from env -->
            <!-- the code should use this key and throw an exception if its length is not 16 bytes -->
            <!--key_hex from_env="..."></key_hex -->

            <!-- Example of multiple hex keys. They can be imported from env or be written down in config-->
            <!-- the code should use these keys and throw an exception if their length is not 16 bytes -->
            <!-- key_hex id="0">...</key_hex -->
            <!-- key_hex id="1" from_env=".."></key_hex -->
            <!-- key_hex id="2">...</key_hex -->
            <!-- current_key_id>2</current_key_id -->

            <!-- Example of getting hex key from config -->
            <!-- the code should use this key and throw an exception if its length is not 16 bytes -->
            <!-- key>...</key -->

            <!-- example of adding nonce -->
            <!-- nonce>...</nonce -->

        <!-- /aes_128_gcm_siv -->
    </encryption_codecs>

    <!-- Allow to execute distributed DDL queries (CREATE, DROP, ALTER, RENAME) on cluster.
         Works only if ZooKeeper is enabled. Comment it if such functionality isn't required. -->
    <distributed_ddl>
        <!-- Path in ZooKeeper to queue with DDL queries -->
        <path>/clickhouse/task_queue/ddl</path>

        <!-- Settings from this profile will be used to execute DDL queries -->
        <!-- <profile>default</profile> -->

        <!-- Controls how much ON CLUSTER queries can be run simultaneously. -->
        <!-- <pool_size>1</pool_size> -->

        <!--
             Cleanup settings (active tasks will not be removed)
        -->

        <!-- Controls task TTL (default 1 week) -->
        <!-- <task_max_lifetime>604800</task_max_lifetime> -->

        <!-- Controls how often cleanup should be performed (in seconds) -->
        <!-- <cleanup_delay_period>60</cleanup_delay_period> -->

        <!-- Controls how many tasks could be in the queue -->
        <!-- <max_tasks_in_queue>1000</max_tasks_in_queue> -->
    </distributed_ddl>

    <!-- Settings to fine tune MergeTree tables. See documentation in source code, in MergeTreeSettings.h -->
    <!--
    <merge_tree>
        <max_suspicious_broken_parts>5</max_suspicious_broken_parts>
    </merge_tree>
    -->

    <!--删除表的限制，默认50G，0表示不限制。如果MergeTree表的大小超过max_table_size_to_drop（单位字节），则无法通过DROP将其删除。如果仍然需要删除而不重新启动Clickhouse，需要创建<clickhouse-path>/flags/force_drop_table文件后再删除-->
    <!-- <max_table_size_to_drop>0</max_table_size_to_drop> -->
    <!-- <max_partition_size_to_drop>0</max_partition_size_to_drop> -->

    <!-- Example of parameters for GraphiteMergeTree table engine -->
    <graphite_rollup_example>
        <pattern>
            <regexp>click_cost</regexp>
            <function>any</function>
            <retention>
                <age>0</age>
                <precision>3600</precision>
            </retention>
            <retention>
                <age>86400</age>
                <precision>60</precision>
            </retention>
        </pattern>
        <default>
            <function>max</function>
            <retention>
                <age>0</age>
                <precision>60</precision>
            </retention>
            <retention>
                <age>3600</age>
                <precision>300</precision>
            </retention>
            <retention>
                <age>86400</age>
                <precision>3600</precision>
            </retention>
        </default>
    </graphite_rollup_example>

    <!-- Directory in <clickhouse-path> containing schema files for various input formats.
         The directory will be created if it doesn't exist.
      -->
    <format_schema_path>/clickhouse/data/node1/format_schemas</format_schema_path>

    <!-- Default query masking rules, matching lines would be replaced with something else in the logs
        (both text logs and system.query_log).
        name - name for the rule (optional)
        regexp - RE2 compatible regular expression (mandatory)
        replace - substitution string for sensitive data (optional, by default - six asterisks)
    <query_masking_rules>
        <rule>
            <name>hide encrypt/decrypt arguments</name>
            <regexp>((?:aes_)?(?:encrypt|decrypt)(?:_mysql)?)\s*\(\s*(?:'(?:\\'|.)+'|.*?)\s*\)</regexp>
            <replace>\1(???)</replace>
        </rule>
    </query_masking_rules> -->

    <!-- Uncomment to use custom http handlers.
        rules are checked from top to bottom, first match runs the handler
            url - to match request URL, you can use 'regex:' prefix to use regex match(optional)
            methods - to match request method, you can use commas to separate multiple method matches(optional)
            headers - to match request headers, match each child element(child element name is header name), you can use 'regex:' prefix to use regex match(optional)
        handler is request handler
            type - supported types: static, dynamic_query_handler, predefined_query_handler
            query - use with predefined_query_handler type, executes query when the handler is called
            query_param_name - use with dynamic_query_handler type, extracts and executes the value corresponding to the <query_param_name> value in HTTP request params
            status - use with static type, response status code
            content_type - use with static type, response content-type
            response_content - use with static type, Response content sent to client, when using the prefix 'file://' or 'config://', find the content from the file or configuration send to client.

    <http_handlers>
        <rule>
            <url>/</url>
            <methods>POST,GET</methods>
            <headers><pragma>no-cache</pragma></headers>
            <handler>
                <type>dynamic_query_handler</type>
                <query_param_name>query</query_param_name>
            </handler>
        </rule>

        <rule>
            <url>/predefined_query</url>
            <methods>POST,GET</methods>
            <handler>
                <type>predefined_query_handler</type>
                <query>SELECT * FROM system.settings</query>
            </handler>
        </rule>

        <rule>
            <handler>
                <type>static</type>
                <status>200</status>
                <content_type>text/plain; charset=UTF-8</content_type>
                <response_content>config://http_server_default_response</response_content>
            </handler>
        </rule>
    </http_handlers>
    -->

    <send_crash_reports>
        <!-- Changing <enabled> to true allows sending crash reports to -->
        <!-- the ClickHouse core developers team via Sentry https://sentry.io -->
        <!-- Doing so at least in pre-production environments is highly appreciated -->
        <enabled>false</enabled>
        <!-- Change <anonymize> to true if you don't feel comfortable attaching the server hostname to the crash report -->
        <anonymize>false</anonymize>
        <!-- Default endpoint should be changed to different Sentry DSN only if you have -->
        <!-- some in-house engineers or hired consultants who're going to debug ClickHouse issues for you -->
        <endpoint>https://6f33034cfe684dd7a3ab9875e57b1c8d@o388870.ingest.sentry.io/5226277</endpoint>
    </send_crash_reports>

    <!-- Uncomment to disable ClickHouse internal DNS caching. -->
    <!-- <disable_internal_dns_cache>1</disable_internal_dns_cache> -->

    <!-- You can also configure rocksdb like this: -->
    <!--
    <rocksdb>
        <options>
            <max_background_jobs>8</max_background_jobs>
        </options>
        <column_family_options>
            <num_levels>2</num_levels>
        </column_family_options>
        <tables>
            <table>
                <name>TABLE</name>
                <options>
                    <max_background_jobs>8</max_background_jobs>
                </options>
                <column_family_options>
                    <num_levels>2</num_levels>
                </column_family_options>
            </table>
        </tables>
    </rocksdb>
    -->

    <!-- Configuration for the query cache -->
    <!-- <query_cache> -->
    <!--     <max_size>1073741824</max_size> -->
    <!--     <max_entries>1024</max_entries> -->
    <!--     <max_entry_size>1048576</max_entry_size> -->
    <!--     <max_entry_rows>30000000</max_entry_rows> -->
    <!-- </query_cache> -->

    <!-- Uncomment if enable merge tree metadata cache -->
    <!--merge_tree_metadata_cache>
        <lru_cache_size>268435456</lru_cache_size>
        <continue_if_corrupted>true</continue_if_corrupted>
    </merge_tree_metadata_cache-->

    <!-- This allows to disable exposing addresses in stack traces for security reasons.
         Please be aware that it does not improve security much, but makes debugging much harder.
         The addresses that are small offsets from zero will be displayed nevertheless to show nullptr dereferences.
         Regardless of this configuration, the addresses are visible in the system.stack_trace and system.trace_log tables
         if the user has access to these tables.
         I don't recommend to change this setting.
    <show_addresses_in_stack_traces>false</show_addresses_in_stack_traces>
    -->

    <!-- On Linux systems this can control the behavior of OOM killer.
    <oom_score>-1000</oom_score>
    -->
</clickhouse>
```

## 存储引擎

数据库存储引擎

表存储引擎





