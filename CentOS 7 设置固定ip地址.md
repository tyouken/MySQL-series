- CentOS 7 设置固定ip地址：
```
cd /etc/sysconfig/network-scripts/
vim ifcfg-ens33
```
- 修改下面2个属性值：
```
BOOTPROTO="static"
IPADDR="192.168.100.100"
```
- 保存后退出vim

- 重启网络：
```
service network restart
```
