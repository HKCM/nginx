# Nginx命令

查看安装Nginx的版本及相关配置信息
```shell
./nginx -V
./nginx -s reload
```
```
-?和-h:显示帮助信息
-v:打印版本号信息并退出
-V:打印版本号信息和配置信息并退出
-t:测试nginx的配置文件语法是否正确并退出
-T:测试nginx的配置文件语法是否正确并列出用到的配置文件信息然后退出
-q:在配置测试期间禁止显示非错误消息
-s:signal信号，后面可以跟 ：
    - stop[快速关闭，类似于TERM/INT信号的作用]
    - quit[优雅的关闭，类似于QUIT信号的作用]
    - reopen[重新打开日志文件类似于USR1信号的作用]
    - reload[类似于HUP信号的作用]
-p:prefix，指定Nginx的prefix路径，(默认为: /usr/local/nginx/)
-c:filename,指定Nginx的配置文件路径,(默认为: conf/nginx.conf)
```


| 信号       | 作用                                |
|----------|-----------------------------------|
| TERM/INT | 立即关闭整个服务                          |
| QUIT     | "优雅"地关闭整个服务                       |
| HUP      | 重读配置文件并使用服务对新配置项生效                |
| USR1     | 重新打开日志文件，可以用来进行日志切割               |
| USR2     | 平滑升级到最新版的nginx                    |
| WINCH    | 所有子进程不在接收处理新连接，相当于给work进程发送QUIT指令 |

1. 发送TERM/INT信号给master进程，会将Nginx服务立即关闭。

    ```shell
    kill -TERM PID / kill -TERM `cat /usr/local/nginx/logs/nginx.pid`
    kill -INT PID / kill -INT `cat /usr/local/nginx/logs/nginx.pid`
    ```

2. 发送QUIT信号给master进程，master进程会控制所有的work进程不再接收新的请求，等所有请求处理完后，在把进程都关闭掉。

    ```shell
    kill -QUIT PID / kill -TERM `cat /usr/local/nginx/logs/nginx.pid`
    ```

3. 发送HUP信号给master进程，master进程会把控制旧的work进程不再接收新的请求，等处理完请求后将旧的work进程关闭掉，然后根据nginx的配置文件重新启动新的work进程

    ```shell
    kill -HUP PID / kill -TERM `cat /usr/local/nginx/logs/nginx.pid`
    ```

4. 发送USR1信号给master进程，告诉Nginx重新开启日志文件

    ```shell
    kill -USR1 PID / kill -TERM `cat /usr/local/nginx/logs/nginx.pid`
    ```

5. 发送USR2信号给master进程，告诉master进程要平滑升级，这个时候，会重新开启对应的master进程和work进程，整个系统中将会有两个master进程，并且新的master进程的PID会被记录在`/usr/local/nginx/logs/nginx.pid`而之前的旧的master进程PID会被记录在`/usr/local/nginx/logs/nginx.pid.oldbin`文件中，接着再次发送QUIT信号给旧的master进程，让其处理完请求后再进行关闭
    
    ```shell
    kill -USR2 PID / kill -USR2 `cat /usr/local/nginx/logs/nginx.pid`
    ```
6. 发送WINCH信号给master进程,让master进程控制不让所有的work进程在接收新的请求了，请求处理完后关闭work进程。注意master进程不会被关闭掉

    ```
    kill -WINCH PID /kill -WINCH`cat /usr/local/nginx/logs/nginx.pid`
    ```

