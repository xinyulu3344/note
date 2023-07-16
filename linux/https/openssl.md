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

查看/etc/pki/tls/openssl.cnf文件，该文件定义了CA的一些默认配置
```bash
dir		        = /etc/pki/CA		        # Where everything is kept
certs		    = $dir/certs		        # Where the issued certs are kept
database	    = $dir/index.txt	        # database index file.
new_certs_dir   = $dir/newcerts		        # default place for new certs.
certificate	    = $dir/cacert.pem 	        # The CA certificate
serial		    = $dir/serial 		        # The current serial number
private_key	    = $dir/private/cakey.pem    # The private key
```
### 创建私有CA

1. 创建CA的私钥
```bash
(umask 066;openssl genrsa -out /etc/pki/CA/private/cakey.pem 2048)
```

2. 生成CA证书
```bash
openssl req -new -x509 -key /etc/pki/CA/private/cakey.pem -out /etc/pki/CA/cacert.pem -days 3650
```

3. 查看CA证书
```bash
openssl x509 -in /etc/pki/CA/cacert.pem -noout -text
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
(umask 066;openssl genrsa -out service.key 4096)
```

2. 客户端生成证书请求
```bash
openssl req -new -key service.key -out service.csr

# 查看请求
openssl req -text -in xinyulu.csr -noout
```

3. 将请求发给CA服务器
4. CA签署证书
```bash
openssl ca -in /etc/pki/CA/csr/service.csr -out /etc/pki/CA/certs/service.crt -days 100
```

5. 查看证书
```bash
openssl x509 -in /etc/pki/CA/certs/service.crt -noout -text
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
