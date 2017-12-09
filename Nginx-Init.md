## Nginx

- 代理服务
- 动态缓存
- 动静分离
- 负载均衡
- Nginx与LUA的开发

### 中间件

- Nginx应用层的安全防护
- 基于Nginx的中间件架构性能优化的问题
- 对sql的注入防攻击
- 对请求的访问控制
- 对请求的频率控制
- 对防爬虫

### Nginx中间件性能优化

- http性能压测
- 性能瓶颈分析
- 系统性能优化
- 基于Nginx的性能配置优化

### 贯彻了技术原理

http协议原理

linux系统原理


### 教程目录

####  基础篇

- 快速安装
- 配置语法
- 默认模块
- Nginx的log
- 访问限制

    - HTTP的请求和连接
    - 请求限制与连接限制
    - access模块配置语法
    - 请求限制局限性
    - 基本安全认证
    - auth模块配置语法
    - 安全认证局限性

#### 场景实践篇

- 静态资源WEB服务
    
    - 什么是静态资源
    - 静态资源服务场景
    - 静态资源服务配置
    - 客户端缓存
    - 静态资源压缩
    - 防盗链
    - 跨域访问 

- 代理服务
- 负载均衡
- 缓存服务

#### 深度学习篇

- 动静分离
- rewrite规则
- 进阶模块的配置
- HTTPS服务

    - HTTPS协议
    - 配置语法
    - Nginx的HTTPS服务
    - Apple要求的HTTP服务(CA证书)

- Nginx与LUA开发

#### 架构篇

- 常见问题
- Nginx中间件性能优化

    - 如何调适性能优化
    - 性能优化影响因素
    -  操作系统性能优化
    - Nginx性能优化

- Nginx与安全
- 新版本特性
- 中间件架构设计

### 环境调试

#### 四项确认

1. 确认系统网络
2. 确认yum可用
3. 确认关闭iptables规则
4. 确认停用selinux  (Ubuntu下要安装 selinux-utils)

```
    1. ping www.baidu.com
    2. yum list | grep gcc
    3. 查看是否有iptables规则: iptables -L 
        关闭: iptables -F
    4. gentenforce 
        查看是否是Disabled,
        如果是Enabled的话,
        关闭命令是:setenforce 0
```

#### 两项安装

> Centos 环境下
```
    yum -y install gcc gcc-c++ autoconf pcre pcre-devel make automake
    yum -y install wget httpd-tools vim
```

> Ubuntu 环境下
```
    sudo apt-get install apache2-dev
```

#### 一次初始化

```
    opt(Optional application software packages)
    cd /opt;
    mkdir app download logs work backup
```