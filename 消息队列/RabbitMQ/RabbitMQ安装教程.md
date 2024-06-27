## 软件下载地址

RabbitMQ官方下载路径： [https://github.com/rabbitmq/rabbitmq-server/releases](https://github.com/rabbitmq/rabbitmq-server/releases)

我这里用的Docker安装演示，其他环境安装后面补充

## Docker安装RabbitMQ

官方提供的Docker脚本

默认登录账号及密码 `guest/guest` ，默认管理页面 [http://127.0.0.1:15672](http://127.0.0.1:15672) ，如果自定义映射端口则是自定的端口

```shell
docker run -itd --rm --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:3.8-management
```

## Linux安装RabbitMQ
