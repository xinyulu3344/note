# pxe+kickstart实现批量装机

| PXE服务器环境       |
| ------------------- |
| 操作系统：CentOS8.2 |
| dhcp-server         |
| tftp-server         |
| nginx               |

## 安装软件包

```bash
dnf -y install dhcp-server tftp-server nginx syslinux-nonlinux
systemctl enable --now nginx tftp
```

## 配置DHCP服务

编辑配置文件：`/etc/dhcp/dhcpd.conf`

```bash
# 对应于/etc/resolv.conf中的search配置
option domain-name "";
# 对应于/etc/resolv.conf中的nameserver配置
option domain-name-servers 8.8.8.8;
# 默认租期
default-lease-time 86400;
# 最大租期
max-lease-time 106400;
log-facility local7;
# 地址池范围
subnet 192.168.0.0 netmask 255.255.0.0 {
    # 分配地址的范围
    range 192.168.0.3 192.168.0.100;
    range 192.168.0.110 192.168.0.200;
    option routers 192.168.0.1;
    # tftp服务器地址
    next-server 192.168.3.96;
    # bootloader启动文件的名称
    filename "pxelinux.0";
}

# 主机MAC绑定IP
host yhnode1 {
    hardware ethernet MAC;
    fixed-address IP;
    # 对应于/etc/resolv.conf中的search配置
    option domain-name "";
    # 对应与/etc/resolv.conf中的nameserver
    option domain-name-servers 8.8.8.8;
    # 默认租期
    default-lease-time 86400;
    # 最大租期
    max-lease-time 106400;
}

host yhnode2 {
    hardware ethernet MAC;
    fixed-address IP;
}
```

```bash
systemctl start dhcpd
```

准备yum源和相关目录

```bash
mkdir -pv /var/www/html/centos/{6,7,8}/os/x86_64
mount -o loop -t iso9660 /export/os/CentOS-8.2.2004-x86_64-dvd1.iso /var/www/html/centos/8/os/x86_64
```

准备kickstart文件

```bash
mkdir /var/www/html/ks/
vim /var/www/html/ks/centos8.cfg
```

centos8.cfg

```bash
#version=RHEL8
ignoredisk --only-use=sda
zerombr
text

# 安装完成后重启
reboot

# 清理所有分区信息
clearpart --all --initlabel
# 禁用selinux
selinux --disabled
# 禁用防火墙
firewall --disabled

# 安装光盘
url --url="http://192.168.3.96/centos/8/os/x86_64/"

# 键盘布局
keyboard --vckeymap=us --xlayouts='us'
# System language
lang en_US.UTF-8
# Network information
network  --bootproto=dhcp --device=ens33 --ipv6=auto --activate
network  --hostname=centos82
# Root password “`123qwe”           #root用户的密码"`123qwe"
rootpw --iscrypted $6$lYV8xqOfEWbJZ/8Y$YXDve/YNwljyh6BnrMHkKm.18kyUqZkHBpA4DHA/nEhXckJfNPLpclgNcQUS64XQSDZRAdlWzjPsI8sCe1dPo0

# Run the Setup Agent on first boot
#firstboot disable      #初次启动设置
firstboot --disable
# System services
services --enabled="chronyd"
# System timezone
timezone Asia/Shanghai --isUtc --nontp

# License agreement     #同意授权协议
eula --agreed
#Add a new user named tyschool password “`123qwe” group whell
# user --groups=wheel --name=tyschool --password=$6$GnUoYHa8rYy7XfrK$0OSNtZwn7.mq4mTeEXQvebx8AEKMV7/PYBS0qfLXUUMJFQozCVMGZY3c0gYcfaKhkCDVQAcxCpi01A5f6a9XC. --iscrypted --gecos="tyschool_com_cn"

part / --fstype="xfs" --ondisk=sda --size=102400
part /boot --fstype="ext4" --ondisk=sda --size=1024
part /boot/efi --fstype="ext4" --ondisk=sda --size=1024


%packages
# @^graphical-server-environment
@^minimal-environment
kexec-tools
%end

%addon com_redhat_kdump --enable --reserve-mb='auto'
%end

%anaconda
pwpolicy root --minlen=6 --minquality=1 --notstrict --nochanges --notempty
pwpolicy user --minlen=6 --minquality=1 --notstrict --nochanges --emptyok
pwpolicy luks --minlen=6 --minquality=1 --notstrict --nochanges --notempty
%end

%post
%end
```



准备PXE启动相关文件

```bash
mkdir /var/lib/tftpboot/centos{6,7,8}
cp /var/www/html/centos/8/os/x86_64/isolinux/vmlinuz /var/lib/tftpboot/centos8/
cp /var/www/html/centos/8/os/x86_64/isolinux/initrd.img /var/lib/tftpboot/centos8/
cp /usr/share/syslinux/pxelinux.0 /var/lib/tftpboot/
cp /usr/share/syslinux/menu.c32 /var/lib/tftpboot/

# centos8必须的文件
cp /var/www/html/centos/8/os/x86_64/isolinux/ldlinux.c32 /var/lib/tftpboot/
cp /var/www/html/centos/8/os/x86_64/isolinux/libcom32.c32 /var/lib/tftpboot/
cp /var/www/html/centos/8/os/x86_64/isolinux/libutil.c32 /var/lib/tftpboot/
```

```bash
mkdir /var/lib/tftpboot/pxelinux.cfg/
cp /var/www/html/centos/8/os/x86_64/isolinux/isolinux.cfg /var/lib/tftpboot/pxelinux.cfg/default
```

修改default文件

```
default menu.c32
timeout 600

menu title CentOS Linux Install

label linux8
  menu label ^Install CentOS Linux 8
  kernel centos8/vmlinuz
  append initrd=centos8/initrd.img ks=http://192.168.3.96/ks/centos8.cfg

label manual
  menu label ^Manual Install CentOS Linux 8
  kernel centos8/vmlinuz
  append initrd=centos8/initrd.img inst.repo=http://192.168.3.96/centos/8/os/x86_64/

label rescue
  menu label ^Rescue a CentOS Linux system 8
  kernel centos8/vmlinuz
  append initrd=centos8/initrd.img inst.repo=http://192.168.3.96/centos/8/os/x86_64/

label local
  menu default
  menu label Boot from ^local drive
  localboot 0xffff
```

