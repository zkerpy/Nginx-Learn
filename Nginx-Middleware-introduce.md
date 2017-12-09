## Nginx 中间件


### IO多路复用

多个描述符的I/O操作都能在一个线程内并发交替地顺序完成，这就叫I/O多路复用

这里的“复用”指的是复用同一个线程

IO多路复用的实现方式select，poll，epoll

select

缺点

1.  能够监视文件描述符的数量存在最大限制
2. 线性扫描效率低下

epoll

1. 每当FD就绪，采用系统的回调函数之间将fd放入，效率更高
2. 最大连接无限制

比喻生活场景：

我们在餐厅吃饭，要结账的生活，我们告诉服务员要结账，服务员告诉老板，但是老板不知道是那一桌要结账，只能一个一个去问这就是select模型

epoll模型就是服务员负责任的把要结账的桌号记住，并报告给老板

###  轻量级

功能模块化

代码模块化

### CPU亲和(affinity)

是一种把CPU核心和Nginx工作进程绑定方式，把每个worker进程固定在一个cpu上执行，减少切换cpu的cache miss，获得更好的性能

### sendfile 

静态文件

只通过内核空间(Kernel space )将文件传递给Socket，不需要通过用户空间(User space)

## 安装

参考

[Nginx  安装](http://nginx.org/en/linux_packages.html#stable)

修改yum软件源配置文件
```
    vim /etc/yum.repos.d/nginx.repo

    配置如下

    [nginx]
    name=nginx repo
    baseurl=http://nginx.org/packages/OS/OSRELEASE/$basearch/
    gpgcheck=0
    enabled=1

    修改OS为你当前的操作系统
    修改OSRELEASE为你当前的操作系统版本号

    查看有那些可供yum源使用的Nginx版本
    yum list | grep nginx

    yum install nginx
```

### 基本参数使用

安装目录

```
    nginx配置文件目录
    rpm -ql  nginx
```
| 路径 | 类型 | 作用 |
| --   | --  | -- |
| /etc/logrotate.d/nginx | 配置文件  | Nginx日志轮转，用于logrotate服务的日志切割 
| /etc/nginx/,/etc/nginx/nginx.conf,/etc/nginx/conf.d,/etc/nginx/conf.d/default.conf|  目录  配置文件 |  Nginx主配置文件    |
| /etc/nginx/fastcgi_params,/etc/nginx/uwsgi\_params,/etc/nginx/scgi\_params  | 配置文件 | cgi配置相关,fastcgi配置  |
| /etc/nginx/koi-utf,/etc/nginx/koi-win,/etc/nginx/win-utf   |  配置文件    |   编码转换映射转化文件 |
| /etc/nginx/mine.types | 配置文件 |  设置http协议的Content-Type与扩展名对应关系 |
| /usr/lib/systemd/system/nginx-debug.service, /usr/lib/systemd/system/nginx.service, /etc/sysconfig/nginx,/etc/sysconfig/nginx-debug |  配置文件 |  用于配置出系统守护进程管理器方式  |
| /usr/lib64/nginx/modules,/etc/nginx/modules | 目录 | Nginx模块目录 |
| /usr/sbin/nginx,/usr/sbin/nginx-debug |  命令  |  Nginx服务的启动管理的终端命令  |
| /usr/share/doc/nginx-1.12.0, /usr/share/doc/nginx-1.12.0,/usr/share/man/man8/nginx.8.gz | 文件，目录  |  Nginx的手册和帮助文件  |
| /var/cache/nginx  | 目录  | Nginx的缓存目录 |
| /var/log/nginx  | 目录  | Nginx的日志目录 |

编译参数

```
    nginx -V
```

Nginx基本语法

```
    systemctl restart/reload nginx
```

### Nginx 模块 

| 编译选项 |   作用  |
|  ---   |  ---   |
| --with-http_stub_status_module    |  Nginx的客户端状态     |      
| --with-http_random_index_module |  目录中选择一个随机主页   |
| --with-http_stub_module    |  HTTP内容替换   |     

```
    Nginx -V 输出前缀为 --with-*
```

#### limit_conn_module
- Syntax:  limit_conn_zone key zone=name:size;
- Default: --
- Context:http

限制变量 key(可以是IP)

- Syntax:  limit_conn zone number;
- Default: --
- Context:http,server,location

```
    修改/etc/nginx/conf.d/default.conf

    limit_conn_zone $binanry_remote_addr zone=conn_zone:1m;
    location / {
        root /opt/app/html;
        limit_conn conn_zone 1;
        index index.html;
    }

    nginx -t -c /etc/nginx/nginx.conf

    nginx -s reload -c /etc/nginx/nginx.conf

```
#### limit_req_module
- Syntax:  limit_req_zone key zone=name:size rate=rate;
- Default: --
- Context:http

rate 速率(一秒多少次请求)


- Syntax:  limit_req zone=name [burst=number] [nodelay];
- Default: --
- Context:http,server,location

```
    修改/etc/nginx/conf.d/default.conf

    limit_conn_zone $binanry_remote_addr zone=conn_zone:1m;
    #同一个客户端IP限制一秒一次
    limit_req_zone $binanry_remote_addr zone=req_zone:1m rate=1r/s;

    location / {
        root /opt/app/html;
        limit_conn conn_zone 1;
        #limit_req zone=req_zone;
        #limit_req zone=req_zone burst=3 ;
        limit_req zone=req_zone burst=3 nodelay;
        index index.html;
    }

    nginx -t -c /etc/nginx/nginx.conf

    nginx -s reload -c /etc/nginx/nginx.conf

```

burst 峰值 当客户端请求速率超过rate的时候，把3个请求放到下次执行，nodelay作用是其他多余的请求直接返回503

压力测试工具 ab(apachebench)
```s
    ab -n 50 -c 20 http://
    -n在测试会话中所执行的请求个数。默认时，仅执行一个请求。
    -c一次产生的请求个数。默认是一次一个。
```
[apache性能测试工具ab使用详解](http://www.jb51.net/article/59469.htm)


| HTTP协议版本 | 连接关系  |
| -- | -- |
| HTTP1.0  | TCP不能复用 | 
| HTTP1.1 | 顺序性TCP复用 | 
| HTTP2.0 | 多路复用TCP复用 | 

#### http_stub_status_module 配置
- Syntax: stub_status;
- Default: --
- Context:server,location
```
    在/etc/nginx/conf.d/default.conf添加如下配置

    location /mystatus {
        stub_status;
    }

    nginx -t -c /etc/nginx/nginx.conf

    nginx -s reload -c /etc/nginx/nginx.conf

```
在浏览器输入 ip/mystatus = >> 

```
    Active connections: 3 
    server accepts handled requests
    32 32 32 
    Reading: 0 Writing: 1 Waiting: 2 
```
#### http_random_index_module 配置
- Syntax: random_index on|off;
- Default: random_index off;
- Context: location;
```
    在/etc/nginx/conf.d/default.conf添加如下配置

    location / {
        随机页面的目录下
        root /opt/app/random_html;
        random_index on;
    }

    nginx -t -c /etc/nginx/nginx.conf

    nginx -s reload -c /etc/nginx/nginx.conf

```
#### http_stub_module 配置
- Syntax: stub_filter string replacement;
- Default: --
- Context:http,server,location

是否最新修改
- Syntax: stub_filter_last_modified on | off;
- Default: stub_filter_last_modified off;
- Context:http,server,location

配置一次
- Syntax: stub_filter_once on | off;
- Default: stub_filter_once on;
- Context:http,server,location
```
    修改/etc/nginx/conf.d/default.conf

    location / {
        root /opt/app/html;
        index index.html;
        sub_filter '<a>codefork ' '<a>codefork.me';
    }

    systemctl reload nginx
```