# nginx软件安装

## yum安装

### 使用官方yum源

```
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true
```



```
[root@realserver11 ~]# nginx -V
nginx version: nginx/1.16.1
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-39) (GCC) 
built with OpenSSL 1.0.2k-fips  26 Jan 2017
TLS SNI support enabled
configure arguments: --prefix=/usr/share/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --http-client-body-temp-path=/var/lib/nginx/tmp/client_body --http-proxy-temp-path=/var/lib/nginx/tmp/proxy --http-fastcgi-temp-path=/var/lib/nginx/tmp/fastcgi --http-uwsgi-temp-path=/var/lib/nginx/tmp/uwsgi --http-scgi-temp-path=/var/lib/nginx/tmp/scgi --pid-path=/run/nginx.pid --lock-path=/run/lock/subsys/nginx --user=nginx --group=nginx --with-file-aio --with-ipv6 --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-stream_ssl_preread_module --with-http_addition_module --with-http_xslt_module=dynamic --with-http_image_filter_module=dynamic --with-http_sub_module --with-http_dav_module --with-http_flv_module --with-http_mp4_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_random_index_module --with-http_secure_link_module --with-http_degradation_module --with-http_slice_module --with-http_stub_status_module --with-http_perl_module=dynamic --with-http_auth_request_module --with-mail=dynamic --with-mail_ssl_module --with-pcre --with-pcre-jit --with-stream=dynamic --with-stream_ssl_module --with-google_perftools_module --with-debug --with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -specs=/usr/lib/rpm/redhat/redhat-hardened-cc1 -m64 -mtune=generic' --with-ld-opt='-Wl,-z,relro -specs=/usr/lib/rpm/redhat/redhat-hardened-ld -Wl,-E'
```





### 使用非官方yum源

## 编译安装

1. 安装依赖

   ```
   yum install gcc pcre-devel zlib-devel
   ```

   

2. 下载nginx文件

   ```bash
   wget http://nginx.org/download/nginx-1.16.1.tar.gz
   ```

3. 解压

   ```bash
   tar -zxf nginx-1.16.1.tar.gz
   ```

4.  编译

   ```
   ./configure --prefix=/usr/local/nginx
   ```

5. 编辑配置文件

6. 启动nginx

   ```
   ./nginx -c xxx.conf
   ```

   

### 开机启动

```
[Unit]
Description=The nginx HTTP and reverse proxy server
After=network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/run/nginx.pid
# Nginx will fail to start if /run/nginx.pid already exists but has the wrong
# SELinux context. This might happen when running `nginx -t` from the cmdline.
# https://bugzilla.redhat.com/show_bug.cgi?id=1268621
ExecStartPre=/usr/bin/rm -f /run/nginx.pid
ExecStartPre=/usr/local/nginx/sbin/nginx -t
ExecStart=/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
ExecReload=/bin/kill -s HUP $MAINPID
KillSignal=SIGQUIT
TimeoutStopSec=5
KillMode=process
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```



# nginx配置文件

## 主配置文件结构

```
# 第一部分：配置文件主区域配置
user nginx;  # 定义worker进程管理的用户
worker_processes auto; # 定义有几个worker进程 == CPU核数 / 核数的两倍
error_log /var/log/nginx/error.log; # 定义错误日志
pid /run/nginx.pid;  # 定义pid文件路径信息
# Load dynamic modules. See /usr/share/doc/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;

# 第二部分：配置文件事件区域
events {
    worker_connections 1024;  # 一个worker进程可以同时接受1024个访问请求
}

# 第三部分：配置http区域
http {
    # 定义日志格式
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    access_log  /var/log/nginx/access.log  main; # 日志路径信息
    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;  # 超时时间
    types_hash_max_size 2048;
    include             /etc/nginx/mime.types;
    default_type        application/octet-stream; # 指定默认识别文件类型
    
    # 第四部分：server区域信息（配置一个网站 --- 虚拟主机）
    include /etc/nginx/conf.d/*.conf;
}
```

## 扩展配置结构

```
server {
    listen       8080;   #指定监听的端口
    server_name  114.67.96.178;   #指定网站域名  
    root         /usr/share/nginx/html; # 定义站点目录的位置
    index        index.html # 定义首页文件
    error_page 500 502 503 504 /50x.html; # 优雅显示页面信息
}
```

# 负载均衡配置

## 两个模块

1. ngx_http_proxy_module
2. ngx_http_upstream_module

```
upstream lbServer {   
    # ip_hash;
    server 192.168.0.13:80 weight=2 max_fails=2 fail_timeout=10;
    server 192.168.0.12:80 weight=1 max_fails=2 fail_timeout=10;
}

server {
    # keepalive_requests 120; #单连接请求上限次数。
    listen       192.168.0.11:8080;   #监听IP:端口
    server_name  192.168.0.11;   #访问域名
    location  / {
        proxy_pass  http://lbServer;  #请求转向lbServer 定义的服务器列表
        proxy_set_header Host $host;  #获取真实Host头
        proxy_set_header X-Forwarded-For $remote_addr; # 获取真实ip
        proxy_next_upstream error timeout http_403 http_404 http_500 http_502 http_503 http_504;
    } 
}
```

