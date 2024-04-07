# dhcp

## 配置

### 接口下配置

```
dhcp enable
interface GigabitEthernet0/0/0
 ip address 192.168.1.1 255.255.255.0
 dhcp select interface
 dhcp server ip-range 192.168.1.3 192.168.1.20
 dhcp server gateway-list 192.168.1.1 
 dhcp server lease day 0 hour 12 minute 0 
 dhcp server dns-list 103.224.222.222 103.224.222.223 
```

### 全局地址池配置

```
dhcp enable
ip pool pool1
 gateway-list 192.168.2.1 
 network 192.168.2.0 mask 255.255.255.0 
 lease day 0 hour 12 minute 0 
 dns-list 103.224.222.222 103.224.222.223
interface GigabitEthernet0/0/0
  ip address 192.168.2.1 255.255.255.0
  dhcp select global
```



## 查看命令

```
[R1]dis ip pool interface GigabitEthernet0/0/0  
  Pool-name        : GigabitEthernet0/0/0
  Pool-No          : 0
  Lease            : 0 Days 12 Hours 0 Minutes
  Domain-name      : -
  DNS-server0      : 103.224.222.222 
  DNS-server1      : 103.224.222.223 
  NBNS-server0     : -               
  Netbios-type     : -               
  Position         : Interface       
  Status           : Unlocked
  Gateway-0        : 192.168.1.1     
  Network          : 192.168.1.0
  Mask             : 255.255.255.0
  VPN instance     : --
  Logging          : Disable
  Conflicted address recycle interval: -
  Address Statistic: Total       :18        Used        :1          
                     Idle        :17        Expired     :0          
                     Conflict    :0         Disabled    :0      

 -------------------------------------------------------------------------------------
  Section ID: 0 
         Start           End       Total    Used Idle(Expired) Conflict Disabled
 -------------------------------------------------------------------------------------
     192.168.1.3    192.168.1.20      18       1         17(0)       0     0
 -------------------------------------------------------------------------------------
```

