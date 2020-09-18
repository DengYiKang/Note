# Hadoop配置

### centos7网络配置

```shell
vi /etc/sysconfig/network-scripts/ifcfg-ens33
```

```shell
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens33
UUID=560d28c7-e216-49ec-9163-3694a2a20c9e
DEVICE=ens33
ONBOOT=yes
IPADDR=192.168.200.131
GATEWAY=192.168.200.2
NETMASK=255.255.255.0
DNS1=8.8.8.8
DNS2=61.147.37.1
```

```shell
vi /etc/resolv.conf
```

```shell
nameserver 8.8.8.8
nameserver 61.147.37.1
```

```shell
service network restart
```

```shell
#如果需要外界工具连接centos7，那么还需要关闭防火墙
#临时关闭防火墙
systemctl stop firewalld
#禁止防火墙开机启动
systemctl disable firewalld
```

