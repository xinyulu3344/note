# clickhouse

## 部署

### 环境准备

hosts文件配置

```bash
cat >> /etc/hosts << EOF
192.168.30.8 node1
192.168.30.9 node2
192.168.30.10 node3
192.168.30.11 node3
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

执行安装

```bash
/clickhouse/app/clickhouse-server/install/doinst.sh
```

软链接配置文件

```bash
cd /etc
mv /etc/clickhouse-* /clickhouse/etc/
ln -sf /clickhouse/etc/* .
```

安装客户端

```bash
/clickhouse/app/clickhouse-client/install/doinst.sh
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
```

手工启动

```bash
nohup clickhouse-server --config=/etc/clickhouse-server/config.xml &
```

客户端登录

```bash
clickhouse-client -h 192.168.30.8 --port 9000 --user default --password
```

