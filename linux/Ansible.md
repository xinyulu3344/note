## ansible介绍和架构

### ansible特性

* 模块化：调用特定的模块，完成特定的任务
* Paramiko（Python对ssh的实现），PyYAML，Jinja2（模板语言）三个关键模块
* 支持自定义模块，可使用任何编程语言写模块
* 基于Python语言实现
* 部署简单，基于Python和SSH，agentless，无需代理，不依赖PKI（无需ssl）
* 安全，基于OpenSSH
* 幂等性：一个任务执行一次和执行n次效果一样，不因重复执行带来意外情况
* 支持playbook编排任务，YAML格式，编排任务，支持丰富的数据结构
* 较强大的多层解决方案role

### ansible架构

#### ansible组成

![image-20210904184354845](images/image-20210904184354845.png)

中间的绿框里的组件，可以理解为是ansible命令工具，其为核心执行工具

* INVENTORY：Ansible管理主机的清单`/etc/ansible/hosts`
* MODULES：Ansible执行命令的功能模块，多数为内置核心模块，也可以自定义
* PLUGINS：模块功能的补充，如连接类型插件、循环插件、变量插件、过滤插件等，不常用
* API：供第三方程序调用的应用程序编程接口

#### ansible命令执行来源

* USER：普通用户，即系统管理员
* PLAYBOOKS：任务剧本（任务集合），编排定义Ansible任务集的配置文件，由Ansible顺序依次执行，通常是JSON格式的文件
* CMDB（配置管理数据库）API调用
* USER -> Ansible playbook ->Ansible

#### 注意事项

* 执行ansible的主机一般称为主控端，中控，master或堡垒机
* 主控端Python版本需要2.6以上
* 被控端Python版本小于2.4需要安装python-simplejson
* 被控端如果开启SELinux需要安装libselinux-python
* windows不能作为主控端

## ansible安装

yum安装和源码编译安装

### yum安装

EPEL源的rpm包安装

```bash
yum install ansible
```



### 源码编译安装

```bash
yum -y install python-jinja2 PyYAML python-paramiko python-babel python-crypto
tar -zxf ansible-1.5.4.tar.gz
cd ansible-1.5.4
python setup.py build
python setup.py install
mkdir /etc/ansible
cp -r examples/* /etc/ansible
```

### Git方式安装

```bash
git clone git://github.com/ansible/ansible.git --recursive
cd ./ansible
source ./hacking/env-setup
```



### pip安装

```bash
sudo pip3 install ansible
```



### 确认安装

```bash
ansible --version
```

## Ansible相关文件

### 配置文件

| 文件                     | 作用                            |
| ------------------------ | ------------------------------- |
| /etc/ansible/ansible.cfg | 主配置文件，配置ansible工作特性 |
| /etc/ansible/hosts       | 主机清单                        |
| /etc/ansible/roles/      | 存放角色的目录                  |



### ansible主配置文件

Ansible的主配置文件`/etc/ansible/ansible.cfg`，其中大部分的配置内容无需配置

```ini
[defaults]
# 主机列表配置文件
inventory       = /etc/ansible/hosts

# 库文件存放目录
library         = ~/.ansible/plugins/modules:/usr/share/ansible/plugins/modules

# 临时py命令文件存放的远程主机目录
remote_tmp      = ~/.ansible/tmp

# 本机的临时命令执行目录
local_tmp       = ~/.ansible/tmp

# 默认并发数，是ansible连接目标机的并发数
forks           = 5

# 默认sudo用户，用该账号连接目标主机
sudo_user      = root
# 每次执行ansible命令是否询问ssh密码
ask_sudo_pass  = True

ask_pass       = True

# 目标服务器ssh服务端口
remote_port    = 22

# 检查对应服务器的host_key，建议取消注释
host_key_checking = False

# 日志文件，建议启用
log_path = /var/log/ansible.log

# 默认模块，可以修改为shell模块
module_name = command
```

### inventory主机清单

默认是`/etc/ansible/hosts`

**主机清单文件格式**

```ini
# Ex 1: Ungrouped hosts, specify before any group headers:

green.example.com
blue.example.com
192.168.100.1
192.168.100.10

# Ex 2: A collection of hosts belonging to the 'webservers' group:

[webservers]
alpha.example.org:80
beta.example.org
192.168.1.100
192.168.1.110

# If you have multiple hosts following a pattern, you can specify
# them like this:

www[001:006].example.com

# Ex 3: A collection of database servers in the 'dbservers' group:

[dbservers]
db01.intranet.mydomain.net
db02.intranet.mydomain.net
10.25.1.56
10.25.1.57

# Here's another example of host ranges, this time there are no
# leading 0s:

db-[99:101]-node.example.com

```

## Ansible相关工具

| 工具             | 作用                                                       |
| ---------------- | ---------------------------------------------------------- |
| Ad-Hoc           | 利用ansible命令，主要用于临时命令的使用场景                |
| Ansible-playbook | 主要用于长期规划好的，大型项目的场景，需要有前期的规划过程 |
| ansible-doc      | 显示模块帮助                                               |



### ansible-doc

1. 列出所有模块

   ```bash
   ansible-doc -l
   ```

   

2. 查看指定模块用法

   ```bash
   ansible-doc ping
   ```

   

3. 查看指定模块用法片段

   ```bash
   ansible-doc -s ping
   ```

   

### ansible

**格式**

```
ansible <host-pattern> [-m module_name] [-a args]
```

| 格式   | 解释                      | 范例                                                   |
| ------ | ------------------------- | ------------------------------------------------------ |
| all    | 表示所有inventory中的主机 | ansible all -m ping                                    |
| 通配符 |                           | ansible "*" -m ping<br />ansible  192.168.1.\* -m ping |
| 逻辑或 |                           |                                                        |
| 逻辑与 |                           |                                                        |

**示例**

```
$ ansible all -m ping -k
SSH password: 
192.168.1.171 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}

```

