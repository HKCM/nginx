SSL(Secure Socket Layer)安全套接层

```properties
server {
    listen       80;
    server_name  localhost;
    location / {
        # 重定向到https
        rewrite ^(.*) https://localhost$1；
    }
}
server {
    listen       443 ssl;
    server_name  localhost;

    ssl_certificate      server.cert; # 证书所在位置
    ssl_certificate_key  server.key; # 

    ssl_session_cache    shared:SSL:1m; # 共享缓存，缓存名称SSL，缓存大小1m
    ssl_session_timeout  5m; # 5分钟

    ssl_ciphers  HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers  on; # 优先使用服务端密码

    location / {
        root   html;
        index  index.html index.htm;
    }
}
```


TLS(Transport Layer Security)传输层安全