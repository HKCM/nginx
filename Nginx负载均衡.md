# Nginx负载均衡

## 基础知识

- 应用层：为应用程序提供网络服务。
- 表示层：对数据进行格式化、编码、加密、压缩等操作。
- 会话层：建立、维护、管理会话连接。
- 传输层：建立、维护、管理端到端的连接，常见的有TCP/UDP。
- 网络层：IP寻址和路由选择
- 数据链路层：控制网络层与物理层之间的通信。
- 物理层：比特流传输。

所谓四层负载均衡指的是OSI七层模型中的传输层，主要是基于IP+PORT的负载均衡

```
实现四层负载均衡的方式：
硬件：F5 BIG-IP、Radware等
软件：LVS、Nginx、Hayproxy等
```

所谓的七层负载均衡指的是在应用层，主要是基于虚拟的URL或主机IP的负载均衡

```
实现七层负载均衡的方式：
软件：Nginx、Hayproxy等
```

四层和七层负载均衡的区别

```
四层负载均衡数据包是在底层就进行了分发，而七层负载均衡数据包则在最顶端进行分发，所以四层负载均衡的效率比七层负载均衡的要高。
四层负载均衡不识别域名，而七层负载均衡识别域名。
```

处理四层和七层负载以为其实还有二层、三层负载均衡，二层是在数据链路层基于mac地址来实现负载均衡，三层是在网络层一般采用虚拟IP地址的方式实现负载均衡。

实际环境采用的模式

```
四层负载(LVS)+七层负载(Nginx)
```

## Nginx七层负载均衡实现

代理服务器在负责均衡调度中的状态有以下几个：

| 状态           | 概述                     |
|--------------|------------------------|
| down         | 当前的server暂时不参与负载均衡     |
| backup       | 预留的备份服务器               |
| max_fails    | 允许请求失败的次数              |
| fail_timeout | 经过max_fails失败后, 服务暂停时间 |
| max_conns    | 限制最大的接收连接数             |


服务端设置

```
server {
    listen   9001;
    server_name localhost;
    default_type text/html;
    location /{
    	return 200 '<h1>192.168.200.146:9001</h1>';
    }
}
server {
    listen   9002;
    server_name localhost;
    default_type text/html;
    location /{
    	return 200 '<h1>192.168.200.146:9002</h1>';
    }
}
server {
    listen   9003;
    server_name localhost;
    default_type text/html;
    location /{
    	return 200 '<h1>192.168.200.146:9003</h1>';
    }
}
```

负载均衡器设置

```properties
upstream backend{
	server 192.168.200.146:9091 down;
	server 192.168.200.146:9092 backup;
	server 192.168.200.146:9093 max_fails=3 fail_timeout=15;
}
server {
	listen 8083;
	server_name localhost;
	location /{
		proxy_pass http://backend;
	}
}
```

### 负载均衡策略

- 轮询
- weight: 权重
- ip_hash
- url_hash
- least_conn:最少连接数
- fair:响应时间 

```properties
upstream backend{
	# least_conn;
    # hash &request_uri;
    # ip_hash;
    server 192.168.200.146:9001 weight=10;
    server 192.168.200.146:9002 weight=5;
    server 192.168.200.146:9003 weight=3;
}
server {
	listen 8083;
	server_name localhost;
	location /{
		proxy_pass http://backend;
	}
}
```

### 负载均衡案例

#### 案例一：对所有请求实现一般轮询规则的负载均衡

```properties
upstream backend{
	server 192.168.200.146:9001;
	server 192.168.200.146:9002;
	server 192.168.200.146:9003;
}
server {
	listen 8083;
	server_name localhost;
	location /{
		proxy_pass http://backend;
	}
}
```

#### 案例二：对所有请求实现加权轮询规则的负载均衡

```properties
upstream backend{
	server 192.168.200.146:9001 weight=7;
	server 192.168.200.146:9002 weight=5;
	server 192.168.200.146:9003 weight=3;
}
server {
	listen 8083;
	server_name localhost;
	location /{
		proxy_pass http://backend;
	}
}
```

#### 案例三：对特定资源实现负载均衡

```properties
upstream videobackend{
	server 192.168.200.146:9001;
	server 192.168.200.146:9002;
}
upstream filebackend{
	server 192.168.200.146:9003;
	server 192.168.200.146:9004;
}
server {
	listen 8084;
	server_name localhost;
	location /video/ {
		proxy_pass http://videobackend;
	}
	location /file/ {
		proxy_pass http://filebackend;
	}
}
```

#### 案例四：对不同域名实现负载均衡

```properties
upstream itcastbackend{
	server 192.168.200.146:9001;
	server 192.168.200.146:9002;
}
upstream itheimabackend{
	server 192.168.200.146:9003;
	server 192.168.200.146:9004;
}
server {
	listen	8085;
	server_name www.itcast.cn;
	location / {
		proxy_pass http://itcastbackend;
	}
}
server {
	listen	8086;
	server_name www.itheima.cn;
	location / {
		proxy_pass http://itheimabackend;
	}
}
```

#### 案例五：实现带有URL重写的负载均衡

```properties
upstream backend{
	server 192.168.200.146:9001;
	server 192.168.200.146:9002;
	server 192.168.200.146:9003;
}
server {
	listen	80;
	server_name localhost;
	location /file/ {
		rewrite ^(/file/.*) /server/$1 last;
	}
	location / {
		proxy_pass http://backend;
	}
}
```

## Nginx四层负载均衡

### 添加stream模块的支持

Nginx默认是没有编译这个模块的，需要使用到stream模块，那么需要在编译的时候加上`--with-stream`。

完成添加`--with-stream`的实现步骤:

```
》将原有/usr/local/nginx/sbin/nginx进行备份
》拷贝nginx之前的配置信息
》在nginx的安装源码进行配置指定对应模块  ./configure --with-stream
》通过make模板进行编译
》将objs下面的nginx移动到/usr/local/nginx/sbin下
》在源码目录下执行  make upgrade进行升级，这个可以实现不停机添加新模块的功能
```

nginx.conf配置

```properties
http {
    ...
}

# stream和http是同级
stream {
    upstream redisbackend {
        server 192.168.200.146:6379;
        server 192.168.200.146:6378;
    }
    upstream tomcatbackend {
        server 192.168.200.146:8080;
    }
    server {
        listen  81;
        proxy_pass redisbackend;
    }
    server {
        listen	82;
        proxy_pass tomcatbackend;
    }
}
```