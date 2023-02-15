## 正向代理

```properties
server {
    listen  82;
    resolver 8.8.8.8;
    location /{
        proxy_pass http://$host$request_uri;
    }
}
```

## 反向代理

```
server {
	listen 80;
	server_name localhost;
	location /{
		#proxy_pass http://192.168.200.146;
		proxy_pass http://192.168.200.146/;
	}
}
```
如果`location`是`/`, 当客户端访问 http://localhost/index.html 在代理后面加不加`/`效果是一样的


```properties
server{
	listen 80;
	server_name localhost;
	location /server{
		#proxy_pass http://192.168.200.146;
		proxy_pass http://192.168.200.146/;
		# 防止暴露真实服务器地址
		proxy_redirect http://192.168.200.146 http://192.168.200.133;
	}
}
```
如果`location`后面有值，当客户端访问 http://localhost/server/index.html

第一个proxy_pass就变成了http://localhost/server/index.html

第二个proxy_pass就变成了http://localhost/index.html

### 示例


代理服务器
```properties
server {
        listen          8082;
        server_name     localhost;
        location /server1 {
                proxy_pass http://192.168.200.146:9001/;
        }
        location /server2 {
                proxy_pass http://192.168.200.146:9002/;
        }
        location /server3 {
                proxy_pass http://192.168.200.146:9003/;
        }
}
```

服务端
```properties
server1
server {
        listen          9001;
        server_name     localhost;
        default_type text/html;
        return 200 '<h1>192.168.200.146:9001</h1>'
}
server2
server {
        listen          9002;
        server_name     localhost;
        default_type text/html;
        return 200 '<h1>192.168.200.146:9002</h1>'
}
server3
server {
        listen          9003;
        server_name     localhost;
        default_type text/html;
        return 200 '<h1>192.168.200.146:9003</h1>'
}
```


