
valid_referers:nginx会通就过查看referer自动和valid_referers后面的内容进行匹配，如果匹配到了就将$invalid_referer变量置0
如果没有匹配到，则将$invalid_referer变量置为1，匹配的过程中不区分大小写。

- none: 如果Header中的Referer为空，允许访问
- blocked:在Header中的Referer不为空，但是该值被防火墙或代理进行伪装过，如不带"http://" 、"https://"等协议头的资源允许访问。
- server_names:指定具体的域名或者IP允许访问
- string: 可以支持正则表达式和*的字符串。如果是正则表达式，需要以`~`开头表示

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