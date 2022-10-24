## Nginx静态资源代理

> 用于访问本地静态文件

```nginx
server {
    listen 80;
    server_name www.xxxx.com;
    location / {
        root /www/index;
    }
    location /image/ {
        root /www/image/;
    }
}
```

## Nginx反向代理

> 用于代理访问接口，如访问其他服务其端口、访问限制外网访问的本地接口等

```nginx
server {
    listen 80;
    server_name www.xxxx.com;
    location / {
        proxy_pass https://www.yyy.com/;
    }
}
```


## Nginx负载均衡

> 为了避免服务器崩溃，大家会通过负载均衡的方式来分担服务器压力。将对台服务器组成一个集 群，当用户访问时，先访问到一个转发服务器，再由转发服务器将访问分发到压力更小的服务器

### Nginx负载均衡策略

#### 轮询

> 每个请求按时间顺序逐一分配到不同的后端服务器，如果后端某个服务器宕机，能自动剔除故障系统

```nginx
upstream backserver {
    server 192.168.0.12;
    server 192.168.0.13;
}
```

#### 权重weight

> weight的值越大分配到的访问概率越高，主要用于后端每台服务器性能不均衡的情况下。其次是为在主从的情况下设置 不同的权值，达到合理有效的地利用主机资源

```nginx
upstream backserver {
    server 192.168.0.12 weight=2;
    server 192.168.0.13 weight=8;
}
```

#### ip_hash

> 每个请求按访问IP的哈希结果分配，使来自同一个IP的访客固定访问一台后端服务器， 并且**可以有效解决动态网页存在的session共享问题**

```nginx
upstream backserver {
    ip_hash;
    server 192.168.0.12:88;
    server 192.168.0.13:80;
}
```

## Nginx高可用配置

> 当上游服务器(真实访问服务器)，一旦出现故障或者是没有及时相应的话，应该直接轮训到下一台服务器，保证服务器的高可用

```nginx
server {
    listen 80;
    server_name www.xxxx.com;
    location / {
        # 指定上游服务器负载均衡服务器 
        proxy_pass http://backServer;
        # nginx与上游服务器(真实访问的服务器)超时时间 后端服务器连接的超时时间_发起握手等 候响应超时时间 
        proxy_connect_timeout 1s;
        # nginx发送给上游服务器(真实访问的服务器)超时时间 
        proxy_send_timeout 1s;
        # nginx接受上游服务器(真实访问的服务器)超时时间 
        proxy_read_timeout 1s;
        index index.html index.htm; 
    }
}
```