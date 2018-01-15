

### For Redhat6/CentOS6, 
```
[root@srv-lambert-centos-test1 ~]# cat /etc/sysconfig/network-scripts/ifcfg-eth0 
DEVICE=eth0
ONBOOT=yes
BOOTPROTO=dhcp
TYPE=Ethernet
USERCTL=no
PEERDNS=yes
IPV6INIT=no
NM_CONTROLLED=no
DHCP_HOSTNAME=srv-lambert-sa-ubuntu-test1
SEARCH=bing.com
DNS1=8.8.8.8
DNS2=114.114.114.114
```

### For Ubuntu

```
root@srv-lambert-ubuntu-test1:~# cat /etc/network/interfaces.d/50-cloud-init.cfg 
# This file is generated from information provided by
# the datasource.  Changes to it will not persist across an instance.
# To disable cloud-init's network configuration capabilities, write a file
# /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg with the following:
# network: {config: disabled}
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet dhcp
dns-search bing.com
dns-nameservers 8.8.8.8
dns-nameservers 114.114.114.114
```