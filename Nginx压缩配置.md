## nginx_gzip.conf
```properties
gzip            on; # 开启压缩 默认只对html生效
gzip_types   application/javascript text/css application/json; # 对js生效
gzip_comp_level 6;  # gzip 压缩级别
gzip_vary       on; # 在响应头中添加Vary字段
gzip_min_length 2k; # 小于2k的数据就不压缩
gzip_disable   "MSIE [1-6]\.";   # 对某些浏览器禁止gzip 对IE6以下禁止gzip
#gzip_buffers 32 4k；# 申请32个缓冲区每个4k 建议使用默认值 

#gzip_http_version 1.1; # 针对不同的HTTP协议版本，可以选择性地开启和关闭Gzip 默认1.1

#gzip_proxied
```

## nginx.conf
```
include nginx_gzip.conf
```

## Gzip和sendfile共存问题

开启sendfile以后，在读取磁盘上的静态资源文件的时候，可以减少拷贝的次数，可以不经过用户进程将静态文件通过网络设备发送出去

但是Gzip要想对资源压缩，是需要经过用户进程进行操作的。所以如何解决两个设置的共存问题。

可以使用`ngx_http_gzip_static_module`模块的`gzip_static`指令来解决。

## 添加模块到Nginx的实现步骤

(1)查询当前Nginx的配置参数

```
nginx -V
```

(2)将nginx安装目录下sbin目录中的nginx二进制文件进行更名

```
cd /usr/local/nginx/sbin
mv nginx nginxold
```

(3) 进入Nginx的安装目录

```
cd /root/nginx/core/nginx-1.16.1
```

(4)执行make clean清空之前编译的内容

```
make clean
```

(5)使用configure来配置参数

```
# 带上 nginx -V 查询到的参数加上with-http_gzip_static_module
./configure --with-http_gzip_static_module
```

(6)使用make命令进行编译

```
make
```

(7) 将objs目录下的nginx二进制执行文件移动到nginx安装目录下的sbin目录中

```
mv objs/nginx /usr/local/nginx/sbin
```

(8)执行更新命令(在nginx的安装目录执行)

```
make upgrade
```

##### gzip_static测试使用

(1)直接访问`http://192.168.200.133/jquery.js`

(2)提前使用gzip命令将资源进行压缩

```
cd /usr/local/nginx/html
gzip jquery.js  # 会生成jquery.js.gz 文件
```

(3)再次访问`http://192.168.200.133/jquery.js` 可以看到`Content-Encoding:gzip`头消息 

