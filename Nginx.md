# Nginx基础与项目运用分享    

## [Nginx基础介绍]((https://zh.wikipedia.org/wiki/Nginx))  

## 反向代理  

### 介绍  
 * 反向代理在电脑网络中是代理服务器的一种。服务器根据客户端的请求，从其关系的一组或多组后端服务器（如Web服务器）上获取资源，然后再将这些资源返回给客户端，客户端只会得知反向代理的IP地址，而不知道在代理服务器后面的服务器集群的存在。   

### 正向代理
1. 要理解反向代理，首先需要知道什么是正向代理，正向代理所访问的URL是我们直接访问的网址，比如我们想要访问www.youtube.com这个网站，正常情况下，是访问不到的，需要一个代理服务器访问，此时我们直接访问的是youtube的网址，这时候的代理服务器就叫做正向代理，也称为前向代理。  

2. 而什么是反向代理呢，反向代理是指我们并不是直接访问的youtube网站，而是访问的是youtube的代理服务器，youtube真实的网站地址，我们是不知道的，具体代理服务器会把这个地址代理到何处，对于终端用户而言是隐藏的，总而言之，我认为反向代理只是一种说法，不用做过多纠结，知道是什么概念就可以了。   

```conf  
location /api {
        proxy_pass   http://b.domain.com:9000;  # 最终地址会加上/api，变成 /api/xxx
        #proxy_cookie_domain   b.domain.com a.domain.com; # 需要修改接口返回的cookie域名时使用
    }
```
## 负载均衡  

### 介绍  
* 负载平衡（Load balancing）是一种电子计算机技术，用来在多个计算机（计算机集群）、网络连接、CPU、磁盘驱动器或其他资源中分配负载，以达到优化资源使用、最大化吞吐率、最小化响应时间、同时避免过载的目的。 使用带有负载平衡的多个服务器组件，取代单一的组件，可以通过冗余提高可靠性。负载平衡服务通常是由专用软件和硬件来完成。 主要作用是将大量作业合理地分摊到多个操作单元上进行执行，用于解决互联网架构中的高并发和高可用的问题。  

1. 负载均衡最重要的一个应用是利用多台服务器提供单一服务,下面为大家画图展示。  
2. 负载均衡的常见算法  
    + round-robin — requests to the application servers are distributed in a round-robin fashion,  
    如果啥都不配置，nginx会默认采用此种负载均衡策略，这种策略最好理解，就是循环。  
    + least-connected — next request is assigned to the server with the least number of active connections,  
    此类策略的规则是当请求来了，nginx会找到所有负载的服务器中最闲的服务器来处理请求。 
        ```conf 
        upstream myapp1 {
            least_conn;
            server srv1.example.com;
            server srv2.example.com;
            server srv3.example.com;
        } 
        ```
    + ip-hash — a hash-function is used to determine what server should be selected for the next request (based on the client’s IP address).   
    这种策略是如果客户端一开始访问的是A服务器，那么他将一直访问A服务器。    
        ```conf  
        upstream myapp1 {
            ip_hash;
            server srv1.example.com;
            server srv2.example.com;
            server srv3.example.com;
        }
        ```
3. 负载均衡权重配置  
    * 如果想要对性能好一些的服务器访问多一些，可以在负载的服务器配置处配置权重，值越大，访问的就越多。
        ```conf  
        upstream myapp1 {
                server srv1.example.com weight=3;
                server srv2.example.com;
                server srv3.example.com;
            }
        ```  

## 动静分离  

### 介绍  
* Nginx本身就是一个网页服务器，可以解析超文本协议，给终端提供服务，所以静态页面的访问完全可以交给Nginx，而不用每次请求静态页面时都去访问Tomcat，新增Tomcat服务器的压力，而Nginx也是极其优秀的静态资源访问服务器，具有极好的性能，并且还有非常优秀的缓存功能，可以极大的提高用户响应速度。  

### 画图演示  

## Nginx 常用命令  
```cmd  
# 启动 
nginx  
# 关闭  
nginx -s stop  
# 重启  
nginx -s reload  
# 手动指定配置文件  
nginx -c /usr/local/nginx/conf/nginx.conf
```    

## Nginx 常用指令  
```properties   
root        /www/wwwroot/www.example.com; # 全局定义，表示在该server下web的根目录，注意要和locate {}下面定义的区分开来。
index       index.php index.html index.htm; # 全局定义访问的默认首页地址。  
charset     utf-8; # 设置网页的默认编码格式。  

client_max_body_size 1000m;         # 设置请求体大小，常用于配置上传文件大小控制。  
add_header Cache-Control no-store;  # 禁用缓存，常用于开发模式，生产模式不需要加     

sendfile on; # 用于开启高效文件传输模式。

tcp_nopush on; # 用于防止网络阻塞。
tcp_nodelay on; # 用于防止网络阻塞。

gzip on; # gzip 压缩，用来对静态资源进行压缩，需要客户端同时支持才有效。
gzip_disable "MSIE [1-6]\.(?!.*SV1)"; # IE6的某些版本对gzip的压缩支持很不好,故关闭。
gzip_http_version 1.0; # HTTP1.0以上的版本都启动gzip
gzip_types text/plain application/javascript application/x-javascript text/javascript text/css application/xml; # 指定哪些类型的相应才启用gzip压缩，多个用空格分隔
gzip_comp_level 5; # 压缩等级，可选1-9，值越大压缩时间越长压缩率越高，通常选2-5

keepalive_timeout  6500;            # 设置客户端连接保持活动的超时时间，在超过这个时间之后，服务器会关闭该连接 。KeepAlive 在一段时间内保持打开状态，它们会在这段时间内占用资源。占用过多就会影响性能。
client_header_timeout 6400;         # 客户端向服务端发送一个完整的 request header 的超时时间。如果客户端在指定时间内没有发送一个完整的 request header，Nginx 返回 HTTP 408（Request Timed Out）。
client_body_timeout 6400;           # 指定客户端与服务端建立连接后发送 request body 的超时时间。如果客户端在指定时间内没有发送任何内容，Nginx 返回 HTTP 408（Request Timed Out）
proxy_read_timeout 6400;            # nginx接收upstream server数据超时, 默认60s, 如果连续的60s内没有收到1个字节, 连接关闭
proxy_send_timeout 6400;            # nginx发送数据至upstream server超时, 默认60s, 如果连续的60s内没有发送1个字节, 连接关闭

deny all;           # 禁用所有IP访问
deny 192.168.1.1;    # 禁用某个IP访问

allow all;          # 允许所有IP访问
allow 10.100.95.10; # 允许某个IP访问  

```   

## Nginx配置文件组成  

### 全局块  

* 从配置文件开始到events块之间的内容，主要影响服务器整体运行的配置指令，比如worker_process 1; worker_process 值越大，能处理的并发数量也越多，那他能不能无限大呢？不能，最好的配置是值得数量是CPU核数，并且每个worker都与指定的核绑定。  

### events块  

* events块涉及到的指令主要影响Nginx服务器与用户的网络连接，比如worker_connections 1024; 支持的最大连接数   

### http块   

* http块是最常配置的位置  
1. server  

```conf  
server {
    listen       443;
    server_name  127.0.0.1;
    ssl on;
    ssl_certificate             my.pem;    # 替换成自己的证书
    ssl_certificate_key         my.key;    # 替换成自己的证书
    ssl_session_timeout         5m;
    ssl_protocols SSLv3 TLSv1.2;
    ssl_ciphers ECDHE-RSA-AES256-SHA384:AES256-SHA256:RC4:HIGH:!MD5:!aNULL:!eNULL:!NULL:!DH:!EDH:!AESGCM;
    ssl_prefer_server_ciphers   on;

    location / {}
}
```  
2. 反向代理  
```conf  
 location /api {
        proxy_pass   http://b.domain.com:9000;  
    }
```

3. 允许它站跨域访问,只需在location中添加如下指令即可  
```properties    
add_header Access-Control-Allow-Origin *;  # *表示允许所有站跨域访问（不安全，建议指定具体允许的域名如：http://b.domain.com:9000（注意格式：http(s):// + domain + port，末尾也不能加/）
add_header Access-Control-Allow-Credentials true;  #此项为允许带cookie跨域访问，若设置true，上面域名配置不能为*，必须指定具体域名
```  
4. 开启gzip压缩  
```conf  
http {
    include          mime.conf;
    default_type     application/octet-stream;
    # ....

    gzip             on;
    gzip_min_length  10k;
    gzip_comp_level  5;
    gzip_types       text/plain text/css application/x-javascript application/javascript text/javascript;

    server {
        # ....
    } 
```   
5. return指令  
    * 如果想把请求直接访问状态码，例如返回404页面，500页面，可直接 return 404;  

6. 请求404，500等通用页面配置  
    ```conf  
    server {
        listen 80;
        server_name www.zhangguoye.com;
        index index.html index.htm;
        root /home/wwwroot;
    
        error_page 404 /404.html;
        location = /404.html {
                    root   /home/wwwroot/;  # 在此目录下添加自定义的404.html
        }
    ```   

## Nginx进程结构  
![Nginx进程结构](./process.png "Nginx进程结构")  

* Nginx启动进程时，会首先启动Master进程，随后由Master进程启动worker进程，其次还有Cache manager以及Cache loader  

## 日志切割  

* Nginx日志数量没有动态生成每天日志的功能，如果想要日志每天生成，也有相应的办法，首先把日志名称重名令，然后运用命令nginx -s reopen重新加载配置文件，最后把以上命令写在定时程序中，每天执行即可。  

## 扩展  
1. 如果获取用户真实IP？
    * 我们使用程序获取到的IP实际上是请求头为remote_addr,remote_port的值，这个IP很可能就是127.0.0.1，而Nginx可以采用http协议中的proxy_protocol的数据结构获取到用户真实的IP地址，此处需要Nginx启用ngx_stream_realip_module模块，默认此模块是不会启用，需要使用命令-with-stream_realip_module启用功能。  
2. 如何限制客户端的并发连接数？   
    * 模块ngx_steam_limit_conn_module,通过-with-stream_limit_conn_module 启用模块    
3. Nginx强大的缓存功能  
    * [Nginx缓存参考文章](https://segmentfault.com/a/1190000020475756)
## Nginx 项目上的运用  
### 西南巡检  
1. 负载均衡  
### 西部天然气  
1. 反向代理 
2. 连接超时配置  
3. 文件上传大小配置    
