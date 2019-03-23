## nat技术

>(Network Address Translation)地址转换, 包括SNAT和DNAT. NAT的功能都是隐藏内部私有网络.

>公网是不会路由私有网段地址, 所以如果私有网段内的ip要上公网, 需要共享一个公网的ip.

>在 iptables中的nat表设置规则, 内核中会自动维护对应的关系. 

>linux下需要设置开启转发功能. `sysctl -w net.ipv4.ip_forward=1`.

### 本页大纲

* [SNAT](#SNAT)
* [DNAT](#DNAT)
* [MASQUERADE](#MASQUERADE)
* [REDIRECT](#REDIRECT)

### SNAT

(Source Network Address Translation) 源地址转换

SNAT规则只能存在于POSTROUTING和INPUT链中. 

可以这样理解: 本机发出的报文, POSTROUTING链是iptables中报文发出的最后一个"关卡", 如果不在这里进行修改, 就没有机会了. 同样的道理, 来自其他主机访问本机, INPUT链是进入本机的最后一道"关卡", 如果不在这里进行修改也就没机会了.

```
iptables -t nat -A POSTROUTING -s 10.1.0.0/16 -j SNAT --to-source 192.168.118.12
```

把来自10.1.0.0/16网段内的地址的报文, 把源地址改为192.168.118.12.


### DNAT

(Destination Network Address Translation) 目的地址转换

DNAT规则只能存在于PREROUTING和OUTPUT链中.

比如公司只有一个公网ip. 这个公网ip分配在公司的网关上, 公司内部有多个私有服务器. web服务器, ftp服务器. 这些服务如何从外网访问, 那么我们就可以在公司的网关上设置DNAT. 对外宣称公司网关(公网ip)上有web服务器, ftpf服务器等. 我们必要再网关上做DNAT. 把访问公网ip的访问重新转发到公司虚拟机的真实的服务器上.

```
iptables -t nat -I PREROUTING -d 192.168.118.12 -p tcp --dport 801 -j DNAT --to-destination 10.1.0.1:80
```

### MASQUERADE

假如每次我们从运行商获得的wlan的ip总是发生变化, 比如我们家庭使用的pppoe方式. 我们都需要重新配置SNAT, 所有MASQUERADE解决这个问题, 会动态的将源地址转换为可用的IP地址. (msquerade 指定的是网卡.)

```
iptables -t nat -I POSTROUTING -s 10.1.0.0/16 -o eth0 -j MASQUERADE
```

MASQUERADE理解为动态的, 自动化的SNAT. 如果没有动态SNAT需求, 就没有必要使用. SNAT更加的高效.


### REDIRECT

在本机上进行端口映射

```
iptables -t nat -A PREROUTING -p tcp --dport 320 -j REDIRECT --to-ports 8080
```

将本机的8080端口映射到本机的320端口. 当访问本机的8080端口. 报文会被重定向到本机的320端口.