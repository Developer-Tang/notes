## 系统信息

### info查看系统信息

```shell
dokcer info
```

## 镜像仓库相关

### search查询镜像

```shell
docker search imageName[:tag]
# imageName： 镜像名称，必填
# tag：版本号/版本标识，选填
# 例：
# docker search redis:5.0
# docker search rabbitmq:3.8-management
```

### pull拉取镜像

```shell
docker pull imageName[:version]
# imageName： 镜像名称，必填
# tag：版本号/版本标识，选填，不填默认拉去latest版本
# 例：
# docker pull redis
```

## 本地镜像相关

### images查看本地镜像

```shell
docker images [options][imageName[:tag]]
# imageName：填写可查询指定的镜像，非模糊匹配
# options：
#   -a：列出所有镜像
#   -q：只列出镜像ID
# 例：
# docker images
```

### rmi删除本地镜像

```shell
docker rmi [options] image[:tag] [imagel..]
# image: 镜像名
# options：
#   -f：强制删除，当镜像有运行容器时，默认不能直接删除
# 例：
# docker rmi redis:5.0 rabbitmq
```

### save导出镜像

```shell
docker save [options] image
# image：镜像名
# options：
#   -o：输出至文件
# 例：
# docker save -o fileName.tar redis:5.0 rabbitmq
```

### load导入镜像

```shell
docker load [options]
# options：
#   -i：指定输入文件
# 例：
# docker load < /path/file.tar
# docker load -i /path/file.tar
```

### import导入镜像

```shell
docker import file|url|- [name[:tag]]
# name[:tag]：自定义指定的名称和版本标识
# file：文件路径
# url：远程路径
# 例：
# docker import ./redis_save.tar my_redis:5.0
```

## 容器相关

### run创建并运行容器

```shell
docker run [options] image
# options：
#   -a stdin: 指定标准输入输出内容类型，可选 STDIN/STDOUT/STDERR 三项；
#   -d: 后台运行容器，并返回容器ID
#   -i: 以交互模式运行容器，通常与 -t 同时使用
#   -P: 随机端口映射，容器内部端口随机映射到主机的端口
#   -p: 指定端口映射，格式为：主机(宿主)端口:容器端口
#   -t: 为容器重新分配一个伪输入终端，通常与 -i 同时使用
#   --name="nginx-lb": 为容器指定一个名称
#   --dns 8.8.8.8: 指定容器使用的DNS服务器，默认和宿主一致
#   --dns-search example.com: 指定容器DNS搜索域名，默认和宿主一致
#   -h "mars": 指定容器的hostname
#   -e username="ritchie": 设置环境变
#   --env-file=[]: 从指定文件读入环境变量
#   --cpuset="0-2" or --cpuset="0,1,2": 绑定容器到指定CPU运行
#   -m :设置容器使用内存最大值
#   --net="bridge": 指定容器的网络连接类型，支持 bridge/host/none/container: 四种类型
#   --link=[]: 添加链接到另一个容器
#   --expose=[]: 开放一个端口或一组端口
#   --volume , -v: 绑定一个卷
# 例：
# docker run -itd -p 6379:6379 redis
```

### create仅创建容器

```shell
docker create [options] image
# 与docker run语法相同
```

### start启动容器

```shell
docker start name|id
# name：容器名
# id：容器ID
# 例：
# docker start ea7bb20a2d4a
```

### stop停止容器

```shell
docker stop name|id
# name：容器名
# id：容器ID
# 例：
# docker stop ea7bb20a2d4a
```

### restart重启容器

```shell
docker restart name|id
# name：容器名
# id：容器ID
# 例：
# docker restart ea7bb20a2d4a
```

### diff检查容器里文件结构

```shell
docker diff name/id
# name：容器名
# id：容器ID
# 例：
# docker diff rabbitmq
```

### cp容器与主机间数据拷贝

```shell
docker cp path|container:path
# path：主机路径
# container:path：容器名/ID:容器内路径
```

> 后面一个参数的名称可以自定义，示例：

```shell
# 向容器内拷贝数据
docker cp ./hello.txt centos7:/hello1.txt
docker cp centos7:/hello1.txt ./hello2.txt
```

### rm删除容器

```shell
docker rm [options] name|id
# name：容器名
# id：容器ID
# options：
#   -f：强制删除，运行状态删除需要该参数
#   -v：删除同时删除挂载在主机的目录
# 例：
# docker rm -fv rabbitmq
```

### pause暂停容器进程

```shell
docker pause name|id
# name：容器名
# id：容器ID
# 例：
# docker pause rabbitmq
```

### unpause恢复容器进程

```shell
docker unpause name|id
# name：容器名
# id：容器ID
# 例：
# docker unpause rabbitmq
```

### exec进入容器执行命令

```shell
docker exec [options] name|id command 
# name：容器名
# id：容器ID
# command：命令
# options：
#   -i：即使没有附加也保持STDIN 打开
#   -t：分配一个伪终端
# 例：
# docker exec -it nginx /bin/bash
```

### ps查询容器

```shell
docker ps [options] [name|id]
# name：容器名
# id：容器ID
# options：
#   -a：显示所有的容器，包括未运行的，不带就查在运行的
#   -l：显示最近创建的容器
#   -n：列出最近创建的n个容器
#   -q：静默模式，只显示容器编号
```

### inspect查询容器信息

```shell
docker inspect name|id
# name：容器名
# id：容器ID
```

### top查询运行容器中的进程

```shell
docker top name|id
# name：容器名
# id：容器ID
```

### logs查询容器日志

```shell
docker logs [options] [name|id]
# name：容器名
# id：容器ID
# options：
#   -f：跟踪日志输出
#   --since：显示某个开始时间的所有日志，--since "时间字符串"
#   -t：显示时间戳
#   --tail：列出最新N条容器日志，--tail 行数
```

### wait阻塞容器运行

> 阻塞运行直到容器停止，然后打印出它的退出代码

```shell
docker wait [name|id]
# name：容器名
# id：容器ID
```

### export导出容器文件系统

```shell
docker export [options] [name|id]
# name：容器名
# id：容器ID
# options：
#   -o：将输入内容写到文件
# 例：
# docker export -o redis-`date +%Y%m%d`.tar redis
```

### port列出端口映射

```shell
docker port [name|id]
# name：容器名
# id：容器ID
```