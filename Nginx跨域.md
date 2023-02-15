同源:  协议、域名(IP)、端口相同即为同源




## 示例
```properties
server{ # ServerB
    listen  8080;
    server_name localhost;
    location /getUser{
        default_type application/json;
        return 200 '{"id":1,"name":"TOM","age":18}';
    }
}
server{ # ServerA
    listen 	80;
    server_name localhost;
    location /{
        root html;
        index index.html;
    }
}
```

此时从ServerA访问ServerB会出现错误，错误信息：`Access-Control-Allow-Origin`

解决方案

在B服务器添加以下头信息
```properties
add_header Access-Control-Allow-origin http://192.168.200.133,http://192.168.200.134; 
add_header Access-Control-Allow-Methods GET,POST,PUT,DELETE;
```

```properties
server{ # ServerB
    listen  8080;
    server_name localhost;
    location /getUser{
        add_header Access-Control-Allow-origin *; # 允许所有
        add_header Access-Control-Allow-Methods GET,POST,PUT,DELETE;
        default_type application/json;
        return 200 '{"id":1,"name":"TOM","age":18}';
    }
}
server{ # ServerA
    listen 	80;
    server_name localhost;
    location /{
        root html;
        index index.html;
    }
}
```

