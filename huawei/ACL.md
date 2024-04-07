# ACL

## ACL分类

| 分类    | 编号范围  | 参数                                         |
| ------- | --------- | -------------------------------------------- |
| 基本ACL | 2000-2999 | 源IP地址等                                   |
| 高级ACL | 3000-3999 | 源IP地址、目的IP地址、源端口、目的端口、协议 |
| 二层ACL | 4000-4999 | 源MAC地址、目的MAC地址、以太帧协议类型等     |

## 配置

基本ACL配置

```
[R2]acl 2000
[R2-acl-basic-2000]rule 10 deny source 192.168.1.100 0.0.0.0
[R2-acl-basic-2000]int g0/0/0
[R2-GigabitEthernet0/0/0]traffic-filter inbound  acl 2000
```

高级ACL配置

```
[R2]acl 3000
[R2-acl-adv-3000]rule 10 deny icmp source 192.168.1.200 0.0.0.0 destination 192.168.4.100 0.0.0.0
```

## 限制说明

同一个接口的同一个方向不能挂接两个acl
