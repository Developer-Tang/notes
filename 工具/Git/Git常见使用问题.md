## GitHub访问慢解决方法

### 1. 查询Git最快的IP

通过 https://www.ipaddress.com/ 这个网站来获取当前github最新的ip

分别获取以下两个域名的IP地址：

```
github.global.ssl.fastly.net 例如：151.101.185.194 
github.com 例如：140.82.114.3
```

### 2. 修改 hosts 配置

Windows环境：

```
C:\Windows\System32\drivers\etc
```

Linux/Mac环境 :

```
vi /etc/hosts
```

修改hosts文件，增加以下内容:

```
151.101.185.194 github.global.ssl.fastly.net 
140.82.114.3 github.com
```

### 3. 刷新DNS缓存

Windows环境：

```
ipconfig /flushdns
```

Linux环境 :

```
/etc/init.d/nscd restart
```

Mac环境 :

```
sudo killall -HUP mDNSResponder
```
