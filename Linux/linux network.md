# Linux Network
## firewalld和iptables

firewall和iptables都属于Linux系统的防火墙，二者之间存在一定的不兼容性，当使用iptables时，需要确保firewall未使用，而使用firewall时无需考虑iptables。

## Centos配置接入DHCP

1.配置ip地址前首先ifconfig或ip addr查看网卡信息并获取到网卡的名称

![image-20230528192057232](https://raw.githubusercontent.com/Hxhao2000/ImagesBed/master/Images/image-20230528192057232.png)

2.进入到网卡配置目录

cd /etc/sysconfig/network-scripts/，找到配置文件为ifcfg-em2

![image-20220302165627133](https://raw.githubusercontent.com/Hxhao2000/ImagesBed/master/Images/image-20220302165627133.png)

修改对应的网卡，例如em2对应ifcfg-em2，在保证配置文件与网卡数相等的情况下，重启network.service

==当配置文件与网卡数不相等时，重启发生错误：/etc/sysconfig/network-scripts/ifup-eth[8540]:  设备eth0似乎不存在, 初始化延迟操作被延迟==

ifcfg-em2文件内容：

```shell
静态IP:
TYPE="Ethernet" # 网络类型，不用改的，默认就是Ethernet，以太网的意思
PROXY_METHOD="none" # 代理方式:关闭状态
BROWSER_ONLY="no" # 只是浏览器:否
BOOTPROTO="DHCP" # 网卡的引导协议:DHCP[中文名称: 动态主机配置协议]
DEFROUTE="yes" # 默认路由:是
IPV4_FAILURE_FATAL="yes" # 是不开启IPV4致命错误检测:否
IPV6INIT="yes" # IPV6是否自动初始化: 是
IPV6_AUTOCONF="yes" # IPV6是否自动配置:是
IPV6_DEFROUTE="yes" # IPV6是否可以为默认路由:是
IPV6_FAILURE_FATAL="no" # 是不开启IPV6致命错误检测:否
IPV6_ADDR_GEN_MODE="stable-privacy" # IPV6地址生成模型:stable-privacy
NAME="em2" # 网卡物理设备名称
UUID="xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxxx" # 通用唯一识别码, 每一个网卡都会有, 不能重复, 否两台linux只有一台网卡可用
DEVICE="em2" # 网卡设备名称, 必须和 `NAME` 值一样
ONBOOT="yes" # 是否开机启动, 要想网卡开机就启动或通过 `systemctl restart network`控制网卡
IPADDR="xxx.xxx.xxx.xxx" # IP地址
PREFIX="24" #配置子网掩码
GATEWAY="xxx.xxx.xxx.xxx" #网关
DNS1="114.114.114.114" #DNS
IPV6_PRIVACY="no"

DHCP:
TYPE=Ethernet # 网络类型，不用改的，默认就是Ethernet，以太网的意思
BOOTPROTO=dhcp
DEFROUTE=yes
PEERDNS=yes
PEERROUTES=yes
IPV4_FAILURE_FATAL=yes
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_PEERDNS=yes
IPV6_PEERROUTES=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=em2
UUID=f2fe7752-1559-4c6c-89f3-4a5586c264e9
DEVICE=em2
ONBOOT=yes
```

4.修改完成后重启网卡

```shell
systemctl restart network
```

==Tips: 手动清除网卡IP地址==

```shell
ip addr flush dev 网卡名(IFNAME)
```

