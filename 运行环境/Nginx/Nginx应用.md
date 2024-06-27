## 什么是Nginx

Nginx 是目前负载均衡技术中的主流方案，几乎绝大部分项目都会使用它，Nginx 是一个轻量级的高性能HTTP反向代理服务器，同时它也是一个通用类型的代理服务器，支持绝大部分协议，如 TCP 、 UDP 、 SMTP 、 HTTPS 等

Nginx 与 Redis 相同，都是基于多路复用模型构建出的产物，因此它与Redis同样具备`资源占用少、并发支持高`的特点，在理论上单节点的Nginx同时支持5W并发连接，而实际生产环境中，硬件基础到位再结合简单调优后确实能达到该数值

## 静态资源代理

用于访问本地静态文件

```nginx
server {
    listen 80; # 监听端口
    server_name localhost; # 监听域名
    location / {
        index index.html; # 静态页面的首页
        root /www/index; # 静态页面的本地目录路径
    }
    location /image/ {
        root /www/image/; # 图片的本地目录路径
    }
}
```

## 反向代理

用于代理访问接口，如访问其他服务其端口、访问限制外网访问的本地接口等

```nginx
server {
    listen 80;
    server_name localhost; # 监听域名
    location / {
        proxy_pass https://www.yyy.com/; # 代理到该网络请求路进行下，也可以代理到本地接口 http://127.0.0.1:xxx
    }
}
```

## 负载均衡

为了避免服务器崩溃导致服务不可用，大家会通过负载均衡的方式来分担服务器压力。将多台服务器组成一个集群，当用户访问时，先访问到一个转发服务器，再由转发服务器将访问分发到压力更小的服务器

<!-- tabs:start -->
<!-- tab:轮询 -->
每个请求按时间顺序逐一分配到不同的后端服务器，如果后端某个服务器宕机，能自动剔除故障系统

```nginx
upstream backserver { # backserver为自定义负载均衡组
    server 192.168.0.12:80;
    server 192.168.0.13:80;
}
```

<!-- tab:加权轮询 -->
weight的值越大分配到的访问概率越高，主要用于后端每台服务器性能不均衡的情况下。其次是为在主从的情况下设置 不同的权值，达到合理有效的地利用主机资源

```nginx
upstream backserver {
    server 192.168.0.12:80 weight=2;
    server 192.168.0.13:80; # weight 默认为1
}
```

<!-- tab:最少链接 -->
将请求发送给当前连接数较少的服务器

```nginx
upstream backserver {
    least_conn;
    server 192.168.0.12:80;
    server 192.168.0.13:80;
}
```

<!-- tab:ip_hash -->
每个请求按访问IP的哈希结果分配，使来自同一个IP的访客固定访问一台后端服务器， 并且 **可以有效解决动态网页存在的session共享问题**

```nginx
upstream backserver {
    ip_hash;
    server 192.168.0.12:80;
    server 192.168.0.13:80;
}
```

<!-- tabs:end -->

用法如下

```nginx
upstream backserver {
    // ... 具体负载配置
}
server {
    listen 9001;
    server_name  0.0.0.0;
    location /api {
        proxy_pass http://backserver;
    }
}
```

!> 以下部分摘取自 [微信推文](https://mp.weixin.qq.com/s/OkqrVuuY7wVl6Xjqf1Mk6g)

## 资源压缩

| 参数                | 参数值                                             | 说明                              |
|-------------------|-------------------------------------------------|---------------------------------|
| gzip              | `on` / `off`                                    | 是否开启压缩机制                        |
| gzip_types        | `image/png text/css application/javascript ...` | 需要压缩的文件类型                       |
| gzip_comp_level   | `1` - `9`                                       | 压缩级别，越高压缩效果越好，但越耗时              |
| gzip_vary         | `on` / `off`                                    | 是否再响应头中添加 Vary: Accept-Encoding |
| gzip_buffers      | `16 8k`                                         | 设置处理压缩请求的缓冲区数量与大小               |
| gzip_disable      | `.*Chrome.*`                                    | 针对不同客户端的请求来设置是否开启压缩             |
| gzip_http_version | `1.1`                                           | 指定压缩响应所需的最低 HTTP 请求版本           |
| gzip_min_length   | `512k`                                          | 设置触发压缩的文件最低大小                   |
| gzip_proxied      | `off` 丨 `expired` 丨 `no-cache` 丨 ...            | 对于后台服务器的响应结果是否开启压缩              |                                 |

**gzip_proxied**

| 参数值              | 说明                                       |
|------------------|------------------------------------------|
| off              | 关闭 Nginx 对后台服务器的响应结果进行压缩                 |
| expired          | 如果响应头中包含 Expires 信息，则开启压缩                |
| no-cache         | 如果响应头中包含 Cache-Control:no-cache 信息，则开启压缩 |
| no-store         | 如果响应头中包含 Cache-Control:no-store 信息，则开启压缩 |
| private          | 如果响应头中包含 Cache-Control:private 信息，则开启压缩  |
| no_last_modified | 如果响应头中不包含 Last-Modified 信息，则开启压缩         |
| no_etag          | 如果响应头中不包含 ETag 信息，则开启压缩                  |
| auth             | 如果响应头中包含 Authorization 信息，则开启压缩          |
| any              | 无条件对后端的响应结果开启压缩机制                        |

## 缓冲区

| 参数                         | 说明                                                                                              |
|----------------------------|-------------------------------------------------------------------------------------------------|
| proxy_buffering            | 是否启用缓冲机制，默认为 on ,开启状态                                                                           |
| client_body_buffer_size    | 设置缓冲客户端请求数据的内存大小                                                                                |
| proxy_buffers              | 为每个请求/连接设置缓冲区的数量和大小，默认4 4k/8k                                                                   |
| proxy_buffer_size          | 设置用于存储响应头的缓冲区大小                                                                                 |
| proxy_busy_buffers_size    | 在后端数据没有完全接收完成时，Nginx 可以将 busy 状态的缓冲返回给客户端，该参数用来设置 busy 状态的 buffer 具体有多大，默认为 proxy_buffer_size*2 |
| proxy_temp_path            | 当内存缓冲区存满时，可以将数据临时存放到磁盘，该参数是设置存储缓冲数据的目录                                                          |
| proxy_temp_file_write_size | 设置每次写数据到临时文件的大小限制                                                                               |
| proxy_max_temp_file_size   | 设置临时的缓冲目录中允许存储的最大容量                                                                             |

具体配置如下：

```nginx
http{  
    proxy_connect_timeout 10;  
    proxy_read_timeout 120;  
    proxy_send_timeout 10;  
    proxy_buffering on;  
    client_body_buffer_size 512k;  
    proxy_buffers 4 64k;  
    proxy_buffer_size 16k;  
    proxy_busy_buffers_size 128k;  
    proxy_temp_file_write_size 128k;  
    proxy_temp_path /soft/nginx/temp_buffer;  
}
```
