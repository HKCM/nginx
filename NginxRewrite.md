## Rewrite命令

### "地址重写"与"地址转发"

重写和转发的区别:

```
地址重写浏览器地址会发生变化而地址转发则不变
一次地址重写会产生两次请求而一次地址转发只会产生一次请求
地址重写到的页面必须是一个完整的路径而地址转发则不需要
地址重写因为是两次请求所以request范围内属性不能传递给新页面而地址转发因为是一次请求所以可以传递值
地址转发速度快于地址重写
```

### Rewrite常用全局变量

| 变量                 | 说明                                                                                                                     |
|--------------------|------------------------------------------------------------------------------------------------------------------------|
| $args              | 变量中存放了请求URL中的请求指令。比如http://192.168.200.133:8080?arg1=value1&args2=value2中的"arg1=value1&arg2=value2"，功能和$query_string一样 |
| $http_user_agent   | 变量存储的是用户访问服务的代理信息(如果通过浏览器访问，记录的是浏览器的相关版本信息)                                                                            |
| $host              | 变量存储的是访问服务器的server_name值                                                                                               |
| $document_uri      | 变量存储的是当前访问地址的URI。比如http://192.168.200.133/server?id=10&name=zhangsan中的"/server"，功能和$uri一样                              |
| $document_root     | 变量存储的是当前请求对应location的root值，如果未设置，默认指向Nginx自带html目录所在位置                                                                 |
| $content_length    | 变量存储的是请求头中的Content-Length的值                                                                                            |
| $content_type      | 变量存储的是请求头中的Content-Type的值                                                                                              |
| $http_cookie       | 变量存储的是客户端的cookie信息，可以通过add_header Set-Cookie 'cookieName=cookieValue'来添加cookie数据                                       |
| $limit_rate        | 变量中存储的是Nginx服务器对网络连接速率的限制，也就是Nginx配置中对limit_rate指令设置的值，默认是0，不限制。                                                       |
| $remote_addr       | 变量中存储的是客户端的IP地址                                                                                                        |
| $remote_port       | 变量中存储了客户端与服务端建立连接的端口号                                                                                                  |
| $remote_user       | 变量中存储了客户端的用户名，需要有认证模块才能获取                                                                                              |
| $scheme            | 变量中存储了访问协议                                                                                                             |
| $server_addr       | 变量中存储了服务端的地址                                                                                                           |
| $server_name       | 变量中存储了客户端请求到达的服务器的名称                                                                                                   |
| $server_port       | 变量中存储了客户端请求到达服务器的端口号                                                                                                   |
| $server_protocol   | 变量中存储了客户端请求协议的版本，比如"HTTP/1.1"                                                                                          |
| $request_body_file | 变量中存储了发给后端服务器的本地文件资源的名称                                                                                                |
| $request_method    | 变量中存储了客户端的请求方式，比如"GET","POST"等                                                                                         |
| $request_filename  | 变量中存储了当前请求的资源文件的路径名                                                                                                    |
| $request_uri       | 变量中存储了当前请求的URI，并且携带请求参数，比如http://192.168.200.133/server?id=10&name=zhangsan中的"/server?id=10&name=zhangsan"             |



### rewrite_log

```properties
rewrite_log on;
error_log  logs/error.log notice;
```

## 域名跳转

### 域名跳转携带URI

修改配置信息

```
server {
	listen 80;
	server_name www.itheima.com;
	rewrite ^(.*) http://www.hm.com$1 permanent;
}
```

### 多域名跳转

修改配置信息

```
server{
	listen 80;
	server_name www.360buy.com www.jingdong.com;
	rewrite ^(.*) http://www.jd.com$1 permanent;
}
```

### 合并目录

实现SEO优化

```
server {
	listen 80;
	server_name www.web.name;
	location /server{
		rewrite ^/server-([0-9]+)-([0-9]+)-([0-9]+)-([0-9]+)\.html$ /server/$1/$2/$3/$4/$5.html last;
	}
}
```

此时访问 http://127.0.0.1/server-11-22-33-44-20.html 就会跳转到 http://127.0.0.1/server/11/22/33/44/20.html


### 防盗链

```properties
# 特定文件防盗链
location ~* ^.+\.(gif|jpg|png|swf|flv|rar|zip)${
    valid_referers none blocked www.baidu.com 192.168.200.222 *.example.com example.*  www.example.org  ~\.google\.;
    if ($invalid_referer){
        rewrite ^/ http://www.web.com/images/forbidden.png;
    }
    root /usr/local/nginx/html;
}

# 目录防盗链
location /images {
    valid_referers none blocked www.baidu.com 192.168.200.222 *.example.com example.*  www.example.org  ~\.google\.;
    if ($invalid_referer){
        rewrite ^/ http://www.web.com/images/forbidden.png;
    }
    root /usr/local/nginx/html;
}
```