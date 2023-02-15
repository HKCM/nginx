# Nginx缓存

- 成本最低的一种缓存实现
- 减少网络带宽消耗
- 降低服务器压力
- 减少网络延迟，加快页面打开速度

## 浏览器缓存

### header字段

HTTP协议中和页面缓存相关的字段

| header        | 说明                      |
|---------------|-------------------------|
| Expires       | 缓存过期的日期和时间              |
| Cache-Control | 设置和缓存相关的配置信息            |
| Last-Modified | 请求资源最后修改时间              |
| ETag          | 请求变量的实体标签的当前值，比如文件的MD5值 |

### 缓存流程

（1）用户首次通过浏览器发送请求到服务端获取数据，客户端是没有对应的缓存，所以需要发送request请求来获取数据；

（2）服务端接收到请求后，获取服务端的数据及服务端缓存的允许后，返回200的成功状态码并且在响应头上附上对应资源以及缓存信息；

（3）当用户再次访问相同资源的时候，客户端会在浏览器的缓存目录中查找是否存在响应的缓存文件

（4）如果没有找到对应的缓存文件，则走(2)步

（5）如果有缓存文件，接下来对缓存文件是否过期进行判断，过期的判断标准是(Expires),

（6）如果没有过期，则直接从本地缓存中返回数据进行展示

（7）如果Expires过期，接下来需要判断缓存文件是否发生过变化

（8）判断的标准有两个，一个是ETag(Entity Tag),一个是Last-Modified

（9）判断结果是未发生变化，则服务端返回304，直接从缓存文件中获取数据

（10）如果判断是发生了变化，重新从服务端获取数据，并根据缓存协商(服务端所设置的是否需要进行缓存数据的设置)来进行数据缓存。

### 缓存响应指令

```
Cache-control: must-revalidate
Cache-control: no-cache
Cache-control: no-store
Cache-control: no-transform
Cache-control: public
Cache-control: private
Cache-control: proxy-revalidate
Cache-Control: max-age=<seconds>
Cache-control: s-maxage=<seconds>
```

| 指令               | 说明                      |
|------------------|-------------------------|
| must-revalidate  | 可缓存但必须再向源服务器进行确认        |
| no-cache         | 缓存前必须确认其有效性             |
| no-store         | 不缓存请求或响应的任何内容           |
| no-transform     | 代理不可更改媒体类型              |
| public           | 可向任意方提供响应的缓存            |
| private          | 仅向特定用户返回响应              |
| proxy-revalidate | 要求中间缓存服务器对缓存的响应有效性再进行确认 |
| max-age=<秒>      | 响应最大Age值                |
| s-maxage=<秒>     | 公共缓存服务器响应的最大Age值        |


### 示例

```properties
location ~ .*\.(html|js|png)$ {\
    expires 10d; # 缓存10天
    add_header Cache-control no-cache;  # 每次必须确认其有效性（弱缓存）
}
```

## Nginx缓存服务


### Nginx缓存配置

Nginx的web缓存服务主要是使用`ngx_http_proxy_module`模块相关指令集来完成

path:缓存路径地址,如：

```
/usr/local/proxy_cache
```

levels: 指定该缓存空间对应的目录，最多可以设置3层，每层取值为1|2如 :

```
levels=1:2   缓存空间有两层目录，第一次是1个字母，第二次是2个字母
举例说明:
itheima[key]通过MD5加密以后的值为 43c8233266edce38c2c9af0694e2107d
levels=1:2   最终的存储路径为/usr/local/proxy_cache/d/07
levels=2:1:2 最终的存储路径为/usr/local/proxy_cache/7d/0/21
levels=2:2:2 最终的存储路径为??/usr/local/proxy_cache/7d/10/e2
```

keys_zone:用来为这个缓存区设置名称和指定大小，如：

```
keys_zone=itcast:200m  # 缓存区的名称是itcast,大小为200M,1M大概能存储8000个keys
```

inactive:指定缓存的数据多次时间未被访问就将被删除，如：

```
inactive=1d  # 缓存数据在1天内没有被访问就会被删除
```

max_size:设置最大缓存空间，如果缓存空间存满，默认会覆盖缓存时间最长的资源，如:

```
max_size=20g
```

配置实例1:

```properties
http{
	proxy_cache_path /usr/local/proxy_cache keys_zone=itcast:200m  levels=1:2:1 inactive=1d max_size=20g;
    proxy_cache_key $scheme$proxy_host$request_uri;
    
    # 状态码从上到下依次生效
    # 为200和302的响应URL设置10分钟缓存，为404的响应URL设置1分钟缓存
    proxy_cache_valid 200 302 10m;
    proxy_cache_valid 404 1m;
    # 对所有响应状态码的URL都设置1分钟缓存
    proxy_cache_valid any 1m;
    # 设置资源被访问10次后被缓存
    proxy_cache_min_uses 10;
    # 缓存GET和HEAD HTTP方法
    proxy_cache_methods GET HEAD; 
}
```

配置实例2:

```properties
http{
	proxy_cache_path /usr/local/proxy_cache levels=2:1 keys_zone=itcast:200m inactive=1d max_size=20g;
	upstream backend{
		server 192.168.200.146:8080;
	}
	server {
		listen       8080;
        server_name  localhost;
        location / {
        	proxy_cache itcast;
            proxy_cache_key itheima;
            proxy_cache_min_uses 5;
            proxy_cache_valid 200 5d;
            proxy_cache_valid 404 30s;
            proxy_cache_valid any 1m;
            # 会在header中显示是否命中
            add_header nginx-cache "$upstream_cache_status";
        	proxy_pass http://backend/js/;
            
            # 不缓存js文件
            if ($request_uri ~ /.*\.js$){
                set $nocache 1;
            }
            # 不缓存
            proxy_no_cache $nocache $cookie_nocache $arg_nocache $arg_comment;
            # 不从缓存中获取数据
            proxy_cache_bypass $nocache $cookie_nocache $arg_nocache $arg_comment;
        }
	}
}
```

### Nginx缓存清除

1. 删除缓存目录 `proxy_cache_path /usr/local/proxy_cache`
2. purge模块




