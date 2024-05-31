## 服务器端搭建

环境以CentOS为例，如果操作系统不一样请注意部分命令需要调整

### 证书管理软件安装及配置

```shell
yum -y install easy-rsa # 安装证书软件 
mkdir /opt/easy-rsa # 创建证书存放目录（可自定义）
cd /opt/easy-rsa # 进入证书存放目录
rpm -ql easy-rsa # 查看证书软件版本路径
cp -a /usr/share/easy-rsa/3.0.8/* . # 拷贝文件到工作目录（版本号改为上面查到的）
cp -a /usr/share/doc/easy-rsa-3.0.8/vars.example ./vars # （版本号改为上面查到的）
vars # 清空文件
vi vars
```

写入如下内容

```text
if [ -z "$EASYRSA_CALLER" ]; then
        echo "You appear to be sourcing an Easy-RSA 
'vars' file." >&2
        echo "This is no longer necessary and is 
disallowed. See the section called" >&2
        echo "'How to use this file' near the top 
comments for more details." >&2
       return 1
fi
set_var EASYRSA_DN "cn_only"
set_var EASYRSA_REQ_COUNTRY "CN"
set_var EASYRSA_REQ_PROVINCE "Shanghai"
set_var EASYRSA_REQ_CITY "Shanghai"
set_var EASYRSA_REQ_ORG "xxx"
set_var EASYRSA_REQ_EMAIL "abc123@126.comm"
set_var EASYRSA_NS_SUPPORT "yes"
```

继续执行命令

```shell
./easyrsa init-pki # 证书初始化
./easyrsa build-ca nopass # 创建根证书（提示输入直接回车就行）
./easyrsa gen-req server nopass # 生成server端证书和私钥文件（提示输入直接回车就行）
./easyrsa sign server server # server端证书签名（输入yes回车）
./easyrsa gen-dh # 创建Diffie-Hellman文件
./easyrsa gen-req client nopass # 生成client端证书和私钥文件（提示输入直接回车就行）
./easyrsa sign client client # client端证书签名（输入yes回车）
```

### 安装OpenVPN及配置

```shell
yum -y install openvpn
vi /etc/openvpn/server.conf
```

写入如下内容

```text
port 1194                                    # 端口
proto udp                                    # 协议
dev tun                                      # 采用路由隧道模式
ca /opt/easy-rsa/pki/ca.crt                  # ca证书的位置
cert /opt/easy-rsa/pki/issued/server.crt     # 服务端公钥的位置
key /opt/easy-rsa/pki/private/server.key     # 服务端私钥的位置
dh /opt/easy-rsa/pki/dh.pem                  # 证书校验算法  
server 10.8.0.0 255.255.255.0                # 给客户端分配的地址池
push "route 172.16.1.0 255.255.255.0"        # 允许客户端访问的内网网段（第一个ip改为自己服务器的网段）
ifconfig-pool-persist ipp.txt                # 地址池记录文件位置，未来让openvpn客户端固定ip地址使用的
keepalive 10 120                             # 存活时间，10秒ping一次，120秒如果未收到响应则视为短线
max-clients 100                              # 最多允许100个客户端连接
status openvpn-status.log                    # 日志位置，记录openvpn状态
log /var/log/openvpn.log                     # openvpn日志记录位置
verb 3                                       # openvpn版本
client-to-client                             # 允许客户端与客户端之间通信
persist-key                                  # 通过keepalive检测超时后，重新启动VPN，不重新读取
persist-tun                                  # 检测超时后，重新启动VPN，一直保持tun是linkup的，否则网络会先linkdown然后再linkup
duplicate-cn                                 # 客户端密钥（证书和私钥）是否可以重复
comp-lzo                                     # 启动lzo数据压缩格式

push "redirect-gateway def1 bypass-dhcp"     # 允许代理网络请求

script-security 3                                   # 允许使用自定义脚本
auth-user-pass-verify /etc/openvpn/check.sh via-env # 指定认证脚本
username-as-common-name                             # 用户密码登陆方式验证
```

配置用户校验脚本

```shell
touch /etc/openvpn/check.sh
chmod +x /etc/openvpn/check.sh
vi  /etc/openvpn/check.sh
```

写入如下内容

```shell
#!/bin/bash
PASSFILE="/etc/openvpn/openvpnfile"   #密码文件 用户名 密码明文
LOG_FILE="/var/log/openvpn-password.log"  #用户登录情况的日志
TIME_STAMP=`date "+%Y-%m-%d %T"`
if [ ! -r "${PASSFILE}" ]; then
    echo "${TIME_STAMP}: Could not open password file \"${PASSFILE}\" for reading." >> ${LOG_FILE}
    exit 1
fi
CORRECT_PASSWORD=`awk '!/^;/&&!/^#/&&$1=="'${username}'"{print $2;exit}'    ${PASSFILE}`
if [ "${CORRECT_PASSWORD}" = "" ]; then
    echo "${TIME_STAMP}: User does not exist: username=\"${username}\",password=\"${password}\"." >> ${LOG_FILE}
    exit 1
fi
if [ "${password}" = "${CORRECT_PASSWORD}" ]; then
    echo "${TIME_STAMP}: Successful authentication: username=\"${username}\"." >> ${LOG_FILE}
    exit 0
fi
echo "${TIME_STAMP}: Incorrect password: username=\"${username}\", password=\"${password}\"." >> ${LOG_FILE}
exit 1
```

配置用户列表

```shell
echo 'username password' >> /etc/openvpn/openvpnfile # 追加用户
# （后面添加/删除后只要执行 systemctl restart openvpn@server 即可）
```

```shell
systemctl start openvpn@server # 启动openvpn服务
systemctl enable openvpn@server # 设置开机启动
ip a s tun0 # 检查tun0网卡是否创建
ss -lntup|grep 1194 # 检查1194端口是否监听
ps -ef|grep openvpn # 检查openvpn进程是否启动
```

### 导出客户端连接文件

```shell
scp xxx@root:/opt/easy-rsa/pki/private/client.key ./
scp xxx@root:/opt/easy-rsa/pki/issued/client.crt ./ 
scp xxx@root:/opt/easy-rsa/pki/ca.crt ./
```

## 客户端链接

### windows客户端

从 [OpenVpN官网](https://openvpn.net/community-downloads/) 下载新版本安装程序，并安装

!> 需要注意 OpenVPN 官网不能在国内正常访问，请科学上网后下载

将上面 [导出](./关于OpenVPN的搭建?id=导出客户端连接文件) 的三个文件保存到 `C:\Users\用户名\OpenVPN\config` 或 `C:\Program Files\OpenVPN\config` 下，并新增文件 `client.ovpn` 如下

```text
client
dev tun
proto udp
remote xxx.xxx.xxx.xxx 1194 #服务端所在的公网IP和端口（请确保防火墙&云服务器安全组开放对应了的端口）
resolv-retry infinite
nobind
ca ca.crt
cert client.crt
key client.key
verb 3
persist-key
comp-lzo
auth-user-pass
```

运行程序 `OpenVPN GUI` 根据弹窗输入对应的用户名和密码即可实现连接
