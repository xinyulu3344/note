## 加密算法

对称加密和非对称加密

### 对称加密

des、3des、aes、Blowfish、Twofish、IDEA、RC6、CAST5

### 非对称加密
RSA、DSA、ELGamal

## hash算法

md5、sha1sum(160bit)    sha224sum  sha256sum  sha384sum  sha512sum
## 密钥交换(IKE)
DH(Deffie-Hellman)
## openssl命令

### 加解密

```bash
# enc加密
openssl enc -e -des3 -a -salt -in person.txt -out person.txt.enc

# enc解密
openssl enc -d -des3 -a -salt -in person.txt.enc -out person.txt.denc
```
### 计算摘要
```bash
# 计算摘要
openssl dgst -md5 -hex persion.txt
```
### 生成密钥对

```bash
# 生成私钥
(umask 066;openssl genrsa -out xxx.key 2048)
# 生成公钥
openssl rsa -in xxx.key -pubout -out xxx.pubkey
```
## TLS实现过程

实现分为握手阶段和应用阶段：

- 握手阶段（协商阶段）：客户端和服务端认证对方身份（依赖于PKI体系，利用数字证书进行身份认证），并协商通信中使用的安全参数、密码套件以及主密钥。后续通信使用的所有密钥都是通过MasterSecret生成
- 应用阶段：在握手阶段完成后进入，在应用阶段通信双方使用握手阶段协商好的密钥进行安全通信

目前密钥交换+签名有三种主流选择：

- RSA密钥交换、RSA数字签名
- ECDHE密钥交换、RSA数字签名
- ECDHE密钥交换、ECDSA数字签名

**RSA密钥交换、RSA数字签名：**

1. Client给出协议版本号、一个客户端随机数（Client random），以及客户端支持的加密方法
2. Server确认双方使用的加密方法，以及一个服务器生成的随机数（Server random）
3. Server发送数字证书给Client
4. Client确认数字证书有效（查看证书状态且查询证书吊销列表），并使用信任的CA的公钥解密数字证书获得Server的公钥，然后生成一个46字节随机数（称为预备主密钥Pre-master Secret），并使用Server的公钥加密预备主密钥发送给Server
5. Server使用自己的私钥，解密Client发来的预备主秘钥
6. Client和Server双方都具有了（客户端随机数、服务端随机数、预备主密钥），两边都根据约定的加密算法，使用这三个随机数生成对称密钥——主秘钥（也称为会话密钥session key），用来加密后续的对话过程
7. 在双方验证完session key的有效性后，SSL握手机制就算结束了。之后所有的数据只需要使用“对话密钥”（此密钥不是session key）加密即可，不再需要多余的加密机制

>注意：
>
>1. 在SSL握手机制中，需要三个随机数
>2. RSA密钥交换有一个很大的问题：没有前向安全性Forward Secrecy。这意味着攻击者可以把监听到的加密流量先存起来，后续一旦拿到了私钥，之前的所有流量都可以成功解密

**目前大部分HTTPS流量用的都是ECDHE密钥交换。**ECDHE是使用椭圆曲线（ECC）的DH（Diffie-Hellman）算法。与RSA密钥交换相比，DH由传递加密的预备主密钥，变成了传递DH算法所需的Parameter，然后双方各自算出预备主密钥。由于预备主密钥不需要交换，中间人即使获得私钥也无法解密之前的流量，但可以实施中间人攻击解密之后的流量。

RSA：既可以用于密钥交换，又可以用于数字签名；

ECC：ECDHE用于密钥交换，ECDSA用于数字签名

## 自建CA

查看`/etc/pki/tls/openssl.cnf`文件，该文件定义了CA的一些默认配置
```bash
[ ca ]
default_ca      = CA_default            # 默认的CA配置用CA_default这个section

[ CA_default ]
dir             = /etc/pki/CA           # CA配置所在的目录
certs           = $dir/certs            # 签发的证书所在目录
crl_dir         = $dir/crl              # 吊销证书所在目录
database        = $dir/index.txt        # 签发过的证书的索引文件
#unique_subject = no                    # Set to 'no' to allow creation of
                                        # several ctificates with same subject.
new_certs_dir   = $dir/newcerts         # 新签发证书的目录

certificate     = $dir/cacert.pem       # CA自签证书
serial          = $dir/serial           # 当前证书的序列号（下一个签发证书的序列号）
crlnumber       = $dir/crlnumber        # the current crl number
                                        # must be commented out to leave a V1 CRL
crl             = $dir/crl.pem          # The current CRL
private_key     = $dir/private/cakey.pem# CA自己的私钥
RANDFILE        = $dir/private/.rand    # private random number file

x509_extensions = usr_cert              # The extentions to add to the cert

# Comment out the following two lines for the "traditional"
# (and highly broken) format.
name_opt        = ca_default            # Subject Name options
cert_opt        = ca_default            # Certificate field options

default_days    = 365                   # 签发证书的默认有效期
default_crl_days= 30                    # 
default_md      = sha256                # 
preserve        = no                    # 


policy          = policy_match          # 对于证书请求，提供的信息匹配策略使用policy_match这个section

# match: 必须和ca一致
# optional: 可选
# supplied: 必须提供
[ policy_match ]
countryName             = match
stateOrProvinceName     = match
organizationName        = match
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional

[ policy_anything ]
countryName             = optional
stateOrProvinceName     = optional
localityName            = optional
organizationName        = optional
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional
```
### 创建私有CA

国家代码查询网站：https://country-code.cl/

1. 创建CA的私钥
```bash
(umask 066;openssl genrsa -out /etc/pki/CA/private/cakey.pem 2048)
```

2. 生成CA证书
```bash
# openssl req -new -x509 -key /etc/pki/CA/private/cakey.pem -out /etc/pki/CA/cacert.pem -days 3650
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:CN
State or Province Name (full name) []:shanghai
Locality Name (eg, city) [Default City]:shanghai
Organization Name (eg, company) [Default Company Ltd]:JD
Organizational Unit Name (eg, section) []:devops
Common Name (eg, your name or your server's hostname) []:ca.xinyulu3344.cn
Email Address []:xinyulu3344@163.com
```

3. 查看CA证书
```bash
# openssl x509 -in cacert.pem -noout -text
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            b2:19:bc:a0:d0:bd:59:1b
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: C=CN, ST=shanghai, L=shanghai, O=JD, OU=devops, CN=ca.xinyulu3344.cn/emailAddress=xinyulu3344@163.com
        Validity
            Not Before: Jul 18 09:36:13 2023 GMT
            Not After : Jul 15 09:36:13 2033 GMT
        Subject: C=CN, ST=shanghai, L=shanghai, O=JD, OU=devops, CN=ca.xinyulu3344.cn/emailAddress=xinyulu3344@163.com
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:aa:6b:d4:4d:e6:96:41:cf:96:42:b1:30:38:a0:
                    4b:3e:30:cb:80:b0:c5:fe:9f:84:88:4b:5e:0b:41:
                    00:31:27:a2:03:c0:c1:0b:f2:c1:ab:c8:c7:35:b4:
                    fe:be:d5:5f:b3:8b:c3:67:fb:26:fe:3e:d7:9e:8b:
                    24:94:b9:de:07:ff:ec:61:8c:bf:f0:7c:34:b4:6b:
                    59:eb:42:53:f6:ee:f3:2b:d7:66:ca:cf:01:70:1d:
                    0a:f0:5d:ba:b9:04:82:fe:23:1a:fa:5e:7e:c6:84:
                    c2:3d:a4:dd:66:35:4f:db:88:36:8c:29:81:c8:b4:
                    37:34:a9:f5:1d:95:2a:64:0c:ad:bb:0b:a3:0c:fe:
                    07:1e:2d:1c:13:73:7d:2e:96:82:75:19:93:6e:93:
                    a9:00:29:d9:83:f0:dc:40:49:1f:c3:00:bb:46:95:
                    05:f0:48:aa:c9:06:3f:3e:30:78:ac:9f:97:fe:a3:
                    20:fd:58:bb:81:0c:7e:94:38:5f:25:e9:a4:aa:38:
                    63:04:c4:3e:26:7b:80:79:02:13:9a:d6:60:71:05:
                    1b:58:7b:7b:6c:9c:fb:ca:e8:e0:21:8e:31:95:62:
                    5c:5d:ae:23:91:5f:ca:55:bf:41:d9:b9:3e:32:9d:
                    48:55:e9:78:65:6a:3a:6b:92:6e:ba:3f:3f:8f:f5:
                    0d:e3
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Subject Key Identifier:
                CD:C1:DA:A7:84:B2:79:64:92:98:BA:2F:E0:F3:4C:07:A6:8D:66:27
            X509v3 Authority Key Identifier:
                keyid:CD:C1:DA:A7:84:B2:79:64:92:98:BA:2F:E0:F3:4C:07:A6:8D:66:27

            X509v3 Basic Constraints:
                CA:TRUE
    Signature Algorithm: sha256WithRSAEncryption
         7d:a9:80:5b:48:1f:f5:e5:b0:43:f2:ea:fa:7c:fc:5a:c8:ab:
         ba:51:63:40:c5:ed:6a:9b:bb:1e:38:e8:d2:02:ef:24:88:50:
         36:85:90:17:c9:58:e3:ee:2a:a2:23:4d:ee:bc:f5:f6:35:c6:
         65:20:fc:57:8d:d4:75:11:96:20:76:2c:2d:95:7b:5a:2f:b1:
         74:8a:16:4a:c6:2c:74:ed:95:03:25:84:6c:cd:74:de:78:f2:
         0c:07:28:77:bf:73:6e:c0:2f:86:69:7d:50:6a:05:b5:38:19:
         7e:87:a8:da:12:05:7d:b6:be:bd:fd:80:eb:e6:8a:62:f9:87:
         e9:0f:bc:dd:ec:a0:02:ea:c9:d5:7d:13:01:56:69:3a:70:5c:
         01:fa:48:79:f2:5f:9d:59:72:f7:25:1b:42:3b:2e:69:56:a6:
         8b:b4:f4:0e:02:5d:4f:d4:f5:7f:39:af:f7:a4:92:52:84:74:
         69:b5:86:56:29:bc:5a:15:83:ea:3d:82:1f:e3:9e:f7:c6:17:
         2e:cd:cf:ff:40:c4:e1:19:a8:d3:85:79:6e:fb:35:c2:ec:d7:
         fc:69:a9:a9:88:36:2f:4f:7d:d4:4b:12:51:51:73:78:05:b9:
         93:d7:d1:68:d6:28:86:e0:98:5a:52:69:86:73:d7:5a:95:04:
         a4:0b:e3:9c
```

4. 创建CA索引文件
```bash
touch /etc/pki/CA/index.txt
```

5. 填写初始serial
```bash
echo 99 > /etc/pki/CA/serial
```
### 客户端申请证书

1. 客户端生成私钥
```bash
(umask 066;openssl genrsa -out app1.key 4096)
```

2. 客户端生成证书请求
```bash
# openssl req -new -key app1.key -out app1.csr
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:CN
State or Province Name (full name) []:shanghai
Locality Name (eg, city) [Default City]:shanghai
Organization Name (eg, company) [Default Company Ltd]:JD
Organizational Unit Name (eg, section) []:it
Common Name (eg, your name or your server's hostname) []:app1.xinyulu3344.cn
Email Address []:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:


# 查看请求
# openssl req -text -in app1.csr -noout
Certificate Request:
    Data:
        Version: 0 (0x0)
        Subject: C=CN, ST=shanghai, L=shanghai, O=JD, OU=it, CN=app1.xinyulu3344.cn
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (4096 bit)
                Modulus:
                    00:bb:fc:2e:d6:01:6a:a7:bc:7d:03:a1:ca:31:f2:
                    83:32:f1:fe:50:9e:1e:41:fc:eb:f7:de:26:1a:45:
                    2f:4d:9a:31:6e:cb:3e:31:8b:a1:73:ae:b3:91:6a:
                    d1:ec:e7:54:2b:78:bb:e2:b9:9d:be:75:38:c4:b9:
                    5f:53:58:cb:64:0c:2c:bd:5b:7f:29:54:c3:db:e7:
                    d5:d5:e7:83:30:78:85:f9:93:36:db:8f:aa:32:7a:
                    a6:2d:a3:f6:bf:67:d5:b0:a8:d7:51:b7:bc:dd:e1:
                    15:2d:c4:4e:e6:74:d6:e6:48:29:28:98:f0:7b:5b:
                    02:8a:85:c9:b8:69:08:a1:f3:71:6b:89:77:e9:94:
                    91:24:f5:f8:32:31:a7:37:6c:03:da:68:23:22:eb:
                    e6:d8:c2:99:03:05:38:99:42:b2:7c:ab:db:11:50:
                    c5:a1:b3:9a:7f:bd:ab:33:87:3a:23:1e:9c:72:8e:
                    ab:a2:ed:2b:92:e3:27:d5:1a:d4:68:9b:9f:dc:94:
                    36:9d:3b:7f:25:d6:22:d5:5d:50:11:98:43:10:d6:
                    71:fe:4b:14:ce:e5:8e:23:9f:a1:d3:a5:17:aa:8d:
                    ad:db:42:14:47:f8:1d:75:20:08:9b:82:3f:80:fe:
                    27:64:46:69:77:59:db:f2:53:7f:6b:68:90:af:c6:
                    05:63:d2:7a:82:5e:66:01:33:76:89:3d:4f:01:35:
                    e6:33:0c:5e:3b:b3:c4:0f:2a:a6:96:45:83:c9:fe:
                    84:e7:f4:b0:c0:df:4e:d3:43:13:6a:c4:3b:97:b9:
                    58:3b:e3:72:29:39:ae:34:93:e5:dd:df:4d:8d:78:
                    a4:1d:d5:53:9e:71:b0:b0:f2:f7:1a:ba:20:27:db:
                    f3:8c:78:5d:52:1d:bd:2f:81:3e:6f:82:17:52:b8:
                    8e:0e:44:c8:ac:d0:c1:18:f8:98:f9:a5:ac:d9:f7:
                    90:73:55:49:2d:6c:8a:fb:27:e7:4f:a3:2e:14:6f:
                    a4:03:10:e1:df:68:14:f2:a1:5e:24:ca:21:18:23:
                    a7:f7:da:66:ae:e0:51:39:f9:58:50:3c:8b:b2:ac:
                    05:52:b9:ba:8f:cc:7f:8b:bd:3c:4a:8e:92:47:d0:
                    fa:94:9c:67:22:f9:b5:ae:04:ff:81:fc:59:7f:d8:
                    37:d3:26:79:e8:b8:a2:cf:23:22:4d:92:d0:35:49:
                    67:e1:ea:19:d6:7e:a2:7a:8d:76:65:50:60:46:ee:
                    32:df:9d:c7:e2:f6:d9:26:da:e1:2b:ca:1c:03:ef:
                    6a:08:ca:86:b9:e9:df:68:18:c8:74:25:5c:c4:53:
                    90:9f:e7:93:06:20:8b:4b:22:15:a6:80:04:39:a4:
                    12:dc:ab
                Exponent: 65537 (0x10001)
        Attributes:
            a0:00
    Signature Algorithm: sha256WithRSAEncryption
         40:15:8f:2c:06:bf:b5:83:65:29:dc:59:00:f5:39:a6:f6:7b:
         f9:44:4f:dc:2e:c9:b1:aa:b6:a5:84:84:c2:c0:48:a6:8a:1b:
         18:2d:a4:eb:27:7f:2f:ca:c6:4b:1b:20:6c:69:1b:71:db:ca:
         99:68:7e:6c:37:61:6d:22:b2:d0:45:b3:eb:47:a8:2a:bd:4c:
         12:1f:c0:d8:8b:da:5d:30:f4:cf:11:ab:24:d3:2b:e3:a6:14:
         e8:f9:ce:4b:2f:d7:93:4b:6e:ed:f9:0c:dc:8d:19:d1:60:e4:
         b6:36:9c:28:0a:78:b5:b3:20:79:8f:1b:00:62:c3:b9:5f:bb:
         9b:2c:1c:f2:c3:6d:ec:60:2d:e4:1a:b8:5f:53:49:11:d8:14:
         06:8f:00:df:93:de:05:e3:b4:90:f2:43:7e:ce:f3:be:af:6c:
         63:96:20:e7:95:da:74:79:08:08:b9:db:32:72:b0:96:ab:fc:
         3d:d6:9f:35:78:b7:94:b0:31:63:05:93:3b:e9:25:04:d3:7c:
         0e:8c:9a:20:f4:dd:84:37:1b:11:fc:ab:bb:48:ab:d8:6b:65:
         10:91:72:c6:7c:fb:55:a8:b6:29:7a:d9:47:81:30:5a:39:1d:
         33:cf:9e:61:29:43:46:9c:4d:20:3a:99:d2:30:e6:bd:5f:50:
         50:04:db:c3:3c:a2:44:4f:2e:de:8a:76:1b:e1:96:a6:8c:99:
         3e:f7:83:54:d7:a2:f7:05:ed:83:71:38:25:df:32:a7:a8:ec:
         c9:ed:a0:37:3a:51:e0:1c:32:a1:83:f9:55:b1:3f:30:bd:bc:
         d4:2e:16:34:07:9a:6d:9e:6a:53:df:c9:3a:e6:59:98:31:e7:
         3f:45:3c:d0:cb:16:d7:ee:10:4d:96:d0:be:37:69:86:80:f8:
         67:4a:45:61:67:13:98:c7:8e:93:19:81:df:23:b1:e8:0c:81:
         13:c0:b1:c7:17:47:2f:c5:5e:c2:38:c7:30:75:8f:30:81:cc:
         9a:33:4e:c0:2b:64:11:8c:28:e8:ff:8e:f4:f3:0e:93:50:db:
         3c:71:29:40:2a:02:f8:e2:a4:40:0c:1d:54:d1:01:63:3b:8d:
         17:36:95:1f:f5:8e:ba:46:bf:d4:af:15:7a:76:e3:0b:9b:15:
         dc:eb:8a:51:35:cd:20:f9:08:0d:49:10:10:b5:00:9b:ee:ff:
         cb:2e:1a:eb:14:46:6a:5e:36:e0:11:f0:19:c8:0f:9a:58:5f:
         33:b6:e5:54:f2:6a:7b:de:45:08:a5:31:6e:14:b9:65:e8:8c:
         1d:ae:75:b5:59:ee:52:92:e0:f6:5e:43:3d:d1:2f:58:32:41:
         b6:9d:e8:47:f8:c5:0f:9d
```

3. 将请求发给CA服务器
4. CA签署证书
```bash
openssl ca -in app1.csr -out /etc/pki/CA/certs/app1.crt -days 3650
```

5. 查看证书
```bash
# openssl x509 -in /etc/pki/CA/certs/app1.crt -noout -text
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 15 (0xf)
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: C=CN, ST=shanghai, L=shanghai, O=JD, OU=devops, CN=ca.xinyulu3344.cn/emailAddress=xinyulu3344@163.com
        Validity
            Not Before: Jul 19 14:50:01 2023 GMT
            Not After : Jul 16 14:50:01 2033 GMT
        Subject: C=CN, ST=shanghai, O=JD, OU=it, CN=app1.xinyulu3344.cn
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (4096 bit)
                Modulus:
                    00:bb:fc:2e:d6:01:6a:a7:bc:7d:03:a1:ca:31:f2:
                    83:32:f1:fe:50:9e:1e:41:fc:eb:f7:de:26:1a:45:
                    2f:4d:9a:31:6e:cb:3e:31:8b:a1:73:ae:b3:91:6a:
                    d1:ec:e7:54:2b:78:bb:e2:b9:9d:be:75:38:c4:b9:
                    5f:53:58:cb:64:0c:2c:bd:5b:7f:29:54:c3:db:e7:
                    d5:d5:e7:83:30:78:85:f9:93:36:db:8f:aa:32:7a:
                    a6:2d:a3:f6:bf:67:d5:b0:a8:d7:51:b7:bc:dd:e1:
                    15:2d:c4:4e:e6:74:d6:e6:48:29:28:98:f0:7b:5b:
                    02:8a:85:c9:b8:69:08:a1:f3:71:6b:89:77:e9:94:
                    91:24:f5:f8:32:31:a7:37:6c:03:da:68:23:22:eb:
                    e6:d8:c2:99:03:05:38:99:42:b2:7c:ab:db:11:50:
                    c5:a1:b3:9a:7f:bd:ab:33:87:3a:23:1e:9c:72:8e:
                    ab:a2:ed:2b:92:e3:27:d5:1a:d4:68:9b:9f:dc:94:
                    36:9d:3b:7f:25:d6:22:d5:5d:50:11:98:43:10:d6:
                    71:fe:4b:14:ce:e5:8e:23:9f:a1:d3:a5:17:aa:8d:
                    ad:db:42:14:47:f8:1d:75:20:08:9b:82:3f:80:fe:
                    27:64:46:69:77:59:db:f2:53:7f:6b:68:90:af:c6:
                    05:63:d2:7a:82:5e:66:01:33:76:89:3d:4f:01:35:
                    e6:33:0c:5e:3b:b3:c4:0f:2a:a6:96:45:83:c9:fe:
                    84:e7:f4:b0:c0:df:4e:d3:43:13:6a:c4:3b:97:b9:
                    58:3b:e3:72:29:39:ae:34:93:e5:dd:df:4d:8d:78:
                    a4:1d:d5:53:9e:71:b0:b0:f2:f7:1a:ba:20:27:db:
                    f3:8c:78:5d:52:1d:bd:2f:81:3e:6f:82:17:52:b8:
                    8e:0e:44:c8:ac:d0:c1:18:f8:98:f9:a5:ac:d9:f7:
                    90:73:55:49:2d:6c:8a:fb:27:e7:4f:a3:2e:14:6f:
                    a4:03:10:e1:df:68:14:f2:a1:5e:24:ca:21:18:23:
                    a7:f7:da:66:ae:e0:51:39:f9:58:50:3c:8b:b2:ac:
                    05:52:b9:ba:8f:cc:7f:8b:bd:3c:4a:8e:92:47:d0:
                    fa:94:9c:67:22:f9:b5:ae:04:ff:81:fc:59:7f:d8:
                    37:d3:26:79:e8:b8:a2:cf:23:22:4d:92:d0:35:49:
                    67:e1:ea:19:d6:7e:a2:7a:8d:76:65:50:60:46:ee:
                    32:df:9d:c7:e2:f6:d9:26:da:e1:2b:ca:1c:03:ef:
                    6a:08:ca:86:b9:e9:df:68:18:c8:74:25:5c:c4:53:
                    90:9f:e7:93:06:20:8b:4b:22:15:a6:80:04:39:a4:
                    12:dc:ab
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Basic Constraints:
                CA:FALSE
            Netscape Comment:
                OpenSSL Generated Certificate
            X509v3 Subject Key Identifier:
                43:14:04:F8:1B:F8:3C:11:D9:C2:67:D3:B2:37:65:53:F1:CB:24:86
            X509v3 Authority Key Identifier:
                keyid:CD:C1:DA:A7:84:B2:79:64:92:98:BA:2F:E0:F3:4C:07:A6:8D:66:27

    Signature Algorithm: sha256WithRSAEncryption
         77:9c:5e:69:1a:ae:2c:94:f9:07:2f:d4:23:57:20:93:0e:c4:
         bb:54:fd:f1:31:af:d1:48:d8:74:f0:fe:31:ad:19:74:68:79:
         9a:ae:25:20:dd:cb:91:25:e5:b2:f4:34:1e:ac:28:b4:e9:52:
         1e:38:78:b0:1b:ae:d3:e5:4f:73:6f:24:09:bf:e7:7d:26:a3:
         63:a3:5e:83:c4:fe:cd:02:d4:7f:32:44:32:09:60:4a:45:fb:
         b6:86:a7:63:86:de:d4:f6:e6:54:a1:3a:98:7c:5f:e9:82:2f:
         d1:30:9a:a9:07:98:15:e7:ab:2b:cb:dd:70:a2:35:24:24:b2:
         7a:1c:9b:56:2a:3b:34:54:be:17:48:f8:80:74:33:0d:e7:5b:
         63:df:3f:a5:b1:7e:bf:38:10:93:47:bd:2c:34:76:73:32:5b:
         53:49:33:35:2f:59:cb:7e:57:c8:28:51:58:1f:ff:01:a6:d1:
         51:a3:ad:d0:e2:96:e9:c3:f4:77:10:38:b6:76:7a:11:70:31:
         19:b3:79:66:20:d0:ea:55:86:69:47:9f:bf:39:92:65:b8:23:
         bd:6f:6f:bc:6e:07:03:12:43:84:fd:c7:b1:ac:df:83:40:ce:
         d2:40:7b:7d:a8:3b:74:be:4d:2e:4e:80:7c:fc:35:bd:84:14:
         05:a8:ce:17
```

6. 将证书发给客户端
### 吊销证书

1. 初始化吊销证书列表数据库
```bash
echo 01 > /etc/pki/CA/crlnumber
```

2. 吊销证书
```bash
openssl ca -revoke /etc/pki/CA/newcerts/99.pem
```

3. 更新证书吊销列表数据库
```bash
openssl ca -gencrl -out /etc/pki/CA/crl.pem
```

4. 查看吊销证书
```bash
openssl crl -in /etc/pki/CA/crl.pem -noout -text
```
