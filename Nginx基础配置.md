## listen

```
listen 127.0.0.1:8000; // listen localhost:8000 监听指定的IP和端口
listen 127.0.0.1;	监听指定IP的所有端口
listen 8000;	监听指定端口上的连接
listen *:8000;	监听指定端口上的连接
```

## server_name

server_name的配置方式有三种，分别是：

- 精确匹配
- 通配符匹配
- 正则表达式匹配

### 精确匹配
```properties
server {
	listen 80;
	server_name www.itcast.cn www.itheima.cn;
}
```

### 通配符匹配
```properties
server {
	listen 80;
	server_name  *.itcast.cn	www.itheima.*;
	# www.itcast.cn abc.itcast.cn www.itheima.cn www.itheima.com
}
```

### 正则表达式匹配
```properties
server{
        listen 80;
        server_name ~^www\.(\w+)\.com$;
        default_type text/plain;
        return 200 $1;
}
# 注意 ~后面不能加空格，括号可以取值
```

### 常见的正则表达式

| 代码    | 说明                                         |
|-------|--------------------------------------------|
| ^     | 匹配搜索字符串开始位置                                |
| $     | 匹配搜索字符串结束位置                                |
| .     | 匹配除换行符\n之外的任何单个字符                          |
| \     | 转义字符，将下一个字符标记为特殊字符                         |
| [xyz] | 字符集，与任意一个指定字符匹配                            |
| [a-z] | 字符范围，匹配指定范围内的任何字符                          |
| \w    | 与以下任意字符匹配 A-Z a-z 0-9 和下划线,等效于[A-Za-z0-9_] |
| \d    | 数字字符匹配，等效于[0-9]                            |
| {n}   | 正好匹配n次                                     |
| {n,}  | 至少匹配n次                                     |
| {n,m} | 匹配至少n次至多m次                                 |
| *     | 零次或多次，等效于{0,}                              |
| +     | 一次或多次，等效于{1,}                              |
| ?     | 零次或一次，等效于{0,1}                             |


### 匹配执行顺序

由于server_name指令支持通配符和正则表达式，因此在包含多个虚拟主机的配置文件中，可能会出现一个名称被多个虚拟主机的server_name匹配成功，当遇到这种情况，当前的请求交给谁来处理呢？

```
server{
	listen 80;
	server_name ~^www\.\w+\.com$;
	default_type text/plain;
	return 200 'regex_success';
}

server{
	listen 80;
	server_name www.itheima.*;
	default_type text/plain;
	return 200 'wildcard_after_success';
}

server{
	listen 80;
	server_name *.itheima.com;
	default_type text/plain;
	return 200 'wildcard_before_success';
}

server{
	listen 80;
	server_name www.itheima.com;
	default_type text/plain;
	return 200 'exact_success';
}

server{
	listen 80 default_server;
	server_name _;
	default_type text/plain;
	return 444 'default_server not found server';
}
```

结论：

```
No1:准确匹配server_name
No2:通配符在开始时匹配server_name成功
No3:通配符在结束时匹配server_name成功
No4:正则表达式匹配server_name成功
No5:被默认的default_server处理，如果没有指定默认找第一个server
```

## location

不带符号，要求必须以指定模式开始

```
server {
	listen 80;
	server_name 127.0.0.1;
	location /abc{
		default_type text/plain;
		return 200 "access success";
	}
}
以下访问都是正确的
http://192.168.200.133/abc
http://192.168.200.133/abc?p1=TOM
http://192.168.200.133/abc/
http://192.168.200.133/abcdef
```

= :  用于不包含正则表达式的uri前，必须与指定的模式精确匹配

```
server {
	listen 80;
	server_name 127.0.0.1;
	location =/abc{
		default_type text/plain;
		return 200 "access success";
	}
}
可以匹配到
http://192.168.200.133/abc
http://192.168.200.133/abc?p1=TOM
匹配不到
http://192.168.200.133/abc/
http://192.168.200.133/abcdef
```

~ ： 用于表示当前uri中包含了正则表达式，并且区分大小写
~*:  用于表示当前uri中包含了正则表达式，并且不区分大小写

换句话说，如果uri包含了正则表达式，需要用上述两个符合来标识

```
server {
	listen 80;
	server_name 127.0.0.1;
	location ~^/abc\w${
		default_type text/plain;
		return 200 "access success";
	}
}
server {
	listen 80;
	server_name 127.0.0.1;
	location ~*^/abc\w${
		default_type text/plain;
		return 200 "access success";
	}
}
```

^~: 用于不包含正则表达式的uri前，功能和不加符号的一致，唯一不同的是，如果模式匹配，那么就停止搜索其他模式了。

```
server {
	listen 80;
	server_name 127.0.0.1;
	location ^~/abc{
		default_type text/plain;
		return 200 "access success";
	}
}
```

## 动静分离

```properties
upstream webservice{
   server 192.168.200.146:8080;
}
server {
        listen       80;
        server_name  localhost;

        #动态资源
        location /demo {
                proxy_pass http://webservice;
        }
        #静态资源
        location ~/.*\.(png|jpg|gif|js){
                root html/web;
                gzip on;
        }

        location / {
            root   html/web;
            index  index.html index.htm;
        }
}
```

## 下载站点

```properties
location /download{
    root /usr/local;
    autoindex on;  # 开启下载功能
    autoindex_exact_size off; # 带有单位显示文件大小
    autoindex_format html; # 默认就好
    autoindex_localtime on; # 显示服务器时间，默认显示文件更新时间
}

```

## 用户认证

需要使用`htpasswd`工具生成

```shell
yum install -y httpd-tools
```

```shell
# 创建一个新文件记录用户名和密码
htpasswd -c /usr/local/nginx/conf/htpasswd username 
# 在指定文件新增一个用户名和密码
htpasswd -b /usr/local/nginx/conf/htpasswd username password
# 从指定文件删除一个用户信息
htpasswd -D /usr/local/nginx/conf/htpasswd username 
# 验证用户名和密码是否正确
htpasswd -v /usr/local/nginx/conf/htpasswd username 
```

```properties
location /download{
    root /usr/local;
    autoindex on;
    autoindex_exact_size on;
    autoindex_format html;
    autoindex_localtime on;
    auth_basic 'please input your auth';
    # 指定密码文件
    auth_basic_user_file htpasswd; 
}
```