## 环境

目标：

​		外网只开放80端口

​		外网IP:www.test.com

​		内网IP:192.168.10.161

本机：

​		出口IP:36.11.14.121

通过webshell查看

```
netstat -nao
```

```
  TCP    0.0.0.0:135            0.0.0.0:0              LISTENING       2240
  TCP    0.0.0.0:445            0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:902            0.0.0.0:0              LISTENING       7256
  TCP    0.0.0.0:912            0.0.0.0:0              LISTENING       7256
  TCP    0.0.0.0:5040           0.0.0.0:0              LISTENING       11252
  TCP    0.0.0.0:7680           0.0.0.0:0              LISTENING       6016
  TCP    0.0.0.0:49664          0.0.0.0:0              LISTENING       2000
  TCP    0.0.0.0:49665          0.0.0.0:0              LISTENING       1896
  TCP    0.0.0.0:49666          0.0.0.0:0              LISTENING       2824
  TCP    0.0.0.0:49668          0.0.0.0:0              LISTENING       6236
  TCP    0.0.0.0:49673          0.0.0.0:0              LISTENING       1964
  TCP    0.0.0.0:55869          0.0.0.0:0              LISTENING       1144
  TCP    0.0.0.0:55889          0.0.0.0:0              LISTENING       16592
  TCP    192.168.10.161:80     36.11.14.121:2343      ESTABLISHED     804
  TCP    127.0.0.1:4709         0.0.0.0:0              LISTENING       1144
  TCP    127.0.0.1:8680         0.0.0.0:0              LISTENING       14264
  TCP    127.0.0.1:49962        0.0.0.0:0              LISTENING       4608
  TCP    127.0.0.1:50010        0.0.0.0:0              LISTENING       8208
  TCP    127.0.0.1:53197        0.0.0.0:0              LISTENING       17260
  TCP    [::]:135               [::]:0                 LISTENING       2240
  TCP    [::]:445               [::]:0                 LISTENING       4
  TCP    [::]:7680              [::]:0                 LISTENING       6016
  TCP    [::]:49664             [::]:0                 LISTENING       2000
  TCP    [::]:49665             [::]:0                 LISTENING       1896
  TCP    [::]:49666             [::]:0                 LISTENING       2824
  TCP    [::]:49668             [::]:0                 LISTENING       6236
  TCP    [::]:49673             [::]:0                 LISTENING       1964
  TCP    [::]:55869             [::]:0                 LISTENING       1144
  TCP    [::]:55889             [::]:0                 LISTENING       16592
  TCP    [::1]:51194            [::]:0                 LISTENING       3828

```

我们出口IP`36.11.14.121`-> `192.168.10.161:80`

## 使用

### Windows

`help`

```cmd
C:\Users\Desktop>ReuseSocks_Server.exe ReuseSocks_Server.exe
ReuseSocks_Server.exe [lhost] [reuse prot] [md5(myip)]
myip is we link to its md5(IP)
it cannot reuse service port like IIS,RDP, but can reuse Mysql,Apache and so on
Use it carefully and bear the consequences.
```

#### 开启端口复用SOCKS5

目标系统执行服务端，注意使用管理员权限执行

```
C:\Users\Desktop>ReuseSocks_Server.exe [lhost]=192.168.10.161 [reuse prot]=80 [md5(myip)]=MD5(192.168.10.1)
```

```cmd
C:\Users\Desktop>ReuseSocks_Server.exe 192.168.10.161 80 1bac027cf67efcc4d10125724221fc48
```

本地执行客户端

```cmd
C:\Users\Desktop>ReuseSocks_client.exe -h
Usage of ReuseSocks_client.exe:
  -listen string
        listen loacl scoks5 (default "0.0.0.0:1080")
  -remote string
        Forward Remote Server eg: 192.168.0.1:80
```

连接目标（默认监听本地1080端口）

```cmd
C:\Users\Desktop>ReuseSocks_client.exe -remote 192.168.10.161:80
```

配置socks5代理为:`127.0.0.1:1080`即可

注意事项

​		当服务端启动之后请2分钟之内使用配置的出口IP(myip)访问一下,否则服务端会退出。

​		默认一个小时服务端程序退出，以免未知原因导致业务端口不能访问。

### linux

`help`

```bash
root@ReuseSock:/opt/lcx# ./ReuseSocks_Server ./ReuseSocks_Server 
./ReuseSocks_Server [Listen prot]
Use it carefully and bear the consequences.
```

#### 开启端口复用

**添加**

将来自[you_ip] IP地址的TCP流量，如果目标端口是80，将其重定向到端口ReuseSocks_Server监听的端口

```bash
sudo iptables -t nat -A PREROUTING -p tcp -s [you_ip] --dport 80[web端口] -j REDIRECT --to-port [ReuseSocks_Server监听的端口]
```

删除

```
sudo iptables -t nat -D PREROUTING -p tcp -s [you_ip] --dport 80[web端口] -j REDIRECT --to-port [ReuseSocks_Server监听的端口]
```

eg:

```
sudo iptables -t nat -A PREROUTING -p tcp -s 192.168.10.1 --dport 80 -j REDIRECT --to-port 1081
```

```bash
root@ReuseSock:/opt/lcx# ./ReuseSocks_Server 1081
2023/07/26 17:54:53 Listen on 0.0.0.0:1081
```

本地执行客户端

```cmd
C:\Users\Desktop>ReuseSocks_client.exe -h
Usage of ReuseSocks_client.exe:
  -listen string
        listen loacl scoks5 (default "0.0.0.0:1080")
  -remote string
        Forward Remote Server eg: 192.168.0.1:80
```

连接目标（默认监听本地1080端口）

```cmd
C:\Users\Desktop>ReuseSocks_client.exe -remote 192.168.10.161:80
```

配置socks5代理为:`127.0.0.1:1080`即可

注：使用完之后记得删除规则
