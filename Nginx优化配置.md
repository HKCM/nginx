# Nginx配置文件

```properties
user www; # 与目录访问权限有关
worker_processes  1; # 也可以设置为auto
# daemon on; # 以守护进程的方式启动
# pid file; # 默认值/usr/local/nginx/logs/nginx.pid

keepalive_timeout 75s;
keepalive_requests 100;

events {
    accept_mutex on; # Nginx网络连接序列化 解决"惊群"问题
    multi_accept on; # 用来设置是否允许同时接收多个网络连接 默认off
    worker_connections  1024; # 不仅仅前端用户建立的连接数，而是包括所有可能的连接数
    use epoll; # epoll IO多路复用
}

http {
    include       mime.types;
    default_type  application/octet-stream; # 浏览器处理字节流的默认方式是下载

    sendfile        on; # 开启零拷贝
    tcp_nopush      on; # 数据存满再发送
    tcp_nodelay     on; # 一有数据就发送

    keepalive_timeout  65;

    server {
        listen       80;
        server_name  localhost;
        location / {
            root   html;
            index  index.html index.htm;
        }

        location /get_text {
            default_type text/html; # 返回简单文本
            return 200 "This is nginx's text";
        }

        location /get_json{
            default_type application/json; # 返回json
            return 200 '{"name":"TOM","age":18}';
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

    }

}
```

