# PPTP Server Ubuntu 21.04 Hosting
由于Ubuntu的默认防火墙是UFW所以只有UFW的防火墙配置

(全部都是以root权限运行)

## Stept 01-安装
##### 安装PPTP服务器
```apt-get install pptpd```

## Stept 02-配置
修改```/etc/pptpd.conf```文件

#### 将文件末尾中的下面语句取消注释
```
#localip 192.168.0.234-238,192.168.0.245
#remoteip 192.168.1.234-238,192.168.1.245
```
变成
```
localip 10.0.0.0      --PPTP服务器地址/本机地址
remoteip 10.0.0.1-255 --分配给远程连接的客户端的IP地址
```

## Stept 03-更改DNS服务器
```/etc/ppp/pptpd-options```

去到```# Network and Routing```的设置寻找下面这两行
```
ms-dns X.X.X.X
ms-dns X.X.X.X
```
*```X.X.X.X```的默认是微软官方DNS服务器地址

## Stept 04-添加客户端用户
修改```/etc/ppp/chap-secrets```文件
>client：连接用户名

>server：连接的服务

>secret：连接密码

>IP addresses：可连接的IP，*表示所有

例子
>```
># Secrets for authentication using CHAP
># client        server  secret                  IP addresses
>用户名     *       密码      *
>```
>>用户名与密码之间的*是键盘上的TAB键来区隔
>>在用户名前面放#代表暂停/停止用户使用该账户

## Stept 05-开启网络转发功能
修改
```/etc/sysctl.conf```
，找到

```**#net.ipv4.ip_forward=1**```

，将注释去掉

例子:

```net.ipv4.ip_forward=1```

然后运行指令立即生效

```sysctl -p```

## Stept 06-UFW防火墙配置
在```/etc/default/ufw```中，修改默认的转发规则
```DEFAULT_FORWARD_POLICY="ACCEPT"```

修改```/etc/ufw/before.rules```，添加如下配置到 *filter 之前
```
# nat Table rules
*nat
:POSTROUTING ACCEPT [0:0]

# Allow traffic from clients to eth0
-A POSTROUTING -s 10.0.0.0/24 -o eth0 -j MASQUERADE

# don.t delete the .COMMIT. line or these nat table rules won.t be processed
COMMIT
```
打开端口 1723
```ufw allow 1723```
重启 UFW防火墙
```
ufw disable
ufw enable
```

## Stept 04-启动
启动PPTP服务:```systemctl start pptpd.service```

重启PPTP服务:```service pptpd restart```

开机自动启动:```systemctl enable pptpd.service```
