# 在ubuntu上搭建vpn

根据大佬的博客：[vps ubuntu上搭建pptp服务](https://my.oschina.net/mn1127/blog/380941)，顺利地搭建好了vpn，这里把过程记录下来（虽然步骤是一模一样的）。

```bash
apt-get update
apt-get install pptpd iptables iptables-persistent
```

```bash
vim /etc/pptpd.conf
#如下修改，localip为vps公网ip;remoteip为给vpn用户分配的ip段
localip ...
remoteip ...
```

```bash
vim /etc/ppp/pptpd-options
#如下修改
ms-dns 8.8.8.8
ms-dns 8.8.4.4
```

```bash
vim /etc/ppp/chap-secrets
#设置vpn的帐号和密码
#用户名 服务 密码 允许ip(可用正则)
...	pptpd ...	*
```

```bash
/etc/init.d/pptpd restart
```

```bash
vim /etc/sysctl.conf
#取消下面的注释，ipv4的转发开关
net.ipv4.ip_forward=1
```

```bash
sysctl -p
```

```bash
#添加NAT
iptables -t nat -A POSTROUTING -s 10.100.0.0/24 -o <vps公网网卡接口> -j MASQUERADE
#设置MTU
iptables -A FORWARD -s 10.100.0.0/24 -p tcp -m tcp --tcp-flags SYN,RST SYN -j TCPMSS --set-mss 1200
#添加NAT
iptables -t nat -A POSTROUTING -s 10.100.0.0/24 -j SNAT --to-source <vps IP>
#保存iptables规则
iptables-save > /etc/iptables-rules
```

```bash
vim /etc/network/interfaces
#编辑网卡文件，加载网卡时自动加载规则
pre-up iptables-restore < /etc/iptables-rules
```

```bash
#iptables配置持久化
apt-get install iptables-persistent
service netfilter-persistent start
```

