## 简介

Dockerfile 文件是用于构建 Docker 镜像的配置文件，定义文件规范文件名 `Dockerfile` 没有文件后缀

## 内容格式

!> FROM 为必要参数 ，其他项皆可不写

```dockerfile
# 基础镜像 【作为容器中的基础系统，需要基础镜像中包含操作系统，未指定版本的默认 latest 版本】
FROM imageName
# 工作目录 【进入容器后的默认路径】
WORKDIR path 
# 复制文件夹或文件到容器 【前面为宿主机路径，后面为容器内路径】
COPY  path targetPath
# 执行命令 【多条可以写多条命令 && 分割，换行用 \】
RUN command -? param
# 定义变量 【格式 key=value , key value都可以】
ENV  varName=varValue
# 声明容器对外挂载的路径 【非必须，最终挂载取决于 docker run 命令】
VOLUME path
# 声明对外暴露的端口 【非必须，最终映射取决于 docker run 命令， protocol 默认 TCP ，可以给多个】
EXPOSE port/protocol
# 使用的用户
USER root  
## 容器启动后执行的命令
CMD: [ "command","-?","${varName}","...." ] 
```

## 示例

### Alpine+JDK8

搭建较小的JDK运行环境

```dockerfile
FROM alpine
RUN apk update
RUN apk add openjdk8 busybox tzdata curl
RUN cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
RUN echo Asia/Shanghai > /etc/timezone
RUN apk del tzdata
RUN  rm -rf /tmp/* /var/cache/apk/*
WORKDIR /opt
```

也可以在构建镜像的过程中加入 jar 包，在文件后面追加 java 启动命令就可以容器创建运行时，自动启动 java 项目
