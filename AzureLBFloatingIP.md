##Azure LoadBalancer的Floating IP

###介绍：
文档是这样说的：Azure LB是不支持端口复用的，如果想使用端口复用，需要启用Floating IP。我看了好多遍，还是看不大懂；[参考文档](https://docs.microsoft.com/en-us/azure/load-balancer/load-balancer-multivip-overview#rule-type-2-backend-port-reuse-by-using-floating-ip)
> 于是我搭了个实验环境测一把；

> Haproxy是一个可以工作在七层或者四层的负载均衡器，类似于Applicaiton Gateway和LB。这里通过使用Haproxy来模拟其它类似于SQL Always-on功能的Cluster.

> Keepalived是一个基于虚拟路由冗余协议(VRRP)实现的高可用软件；这里通过使用Keepalived来模拟windows failover cluster。

###先说一下在私有云如何部署高可用的负载均衡器:
请看下图：为了实现网站的高可用，我们通常会在网站(应用)的入口部署多个负载均衡服务器(srv-lambert-haproxy1&srv-lambert-haproxy2)，如果主的负载均衡器挂掉，备用的负载均衡器就充当负载分发的角色。但是面临一个问题就是，前端的防火墙或路由设备如何做NAT呢，如果80的端口映射到haproxy1上，haproxy1挂掉，手动映射到haproxy2. 这样岂不是太傻。如果后端存在一个虚拟IP地址，然后将80的请求NAT到这个虚拟IP地址上，这样做岂不是很聪明吗。因此Keepalived诞生了，keepalive的实现是基于VRRP，两个keepalive进程会定时做心跳检测。分别将Keepalived安装到两台VM上，通过配置，让其对haproxy的进程做健康检查，如果主服务器上的keepalived检测上haproxy挂掉，主的keeplived会通知从服务器的keepalived进程，然后从服务器就把虚拟IP地址绑定到自己的网络接口上。这样就可以达到高可用的目的；

![](https://raw.githubusercontent.com/Leejung168/azure/master/keepalive.png)


###再说一下在私有云如何部署高可用的关系型数据库：
像Mysql, SqlServer这些数据库软件是属于关系型数据库，为保证数据一致性，在同一时刻一个事务的写操作必须只能在同一台数据库服务器上完成。 但是为了实现数据的备份和应用的高可用，我们往往会搭建两台数据库服务器，一主一备， 备用的服务器会实时同步主服务器上的数据库。如果主的服务器挂掉，我们期望从服务器能承担主服务器的角色，来完成用户的写操作。 因此有两种方法实现：

1. 应用的代码控制，发现主服务器的数据库进程连不上，就自动调整到从服务器的数据库进程。
2. 运维层面控制，通过keepalived，其实现方式和上述描述一样。应用代码的数据库连接请求指向到虚拟IP地址即可。


###再来说一下在云上如何实现keepalived：
在云上，我们可以配置Keepalived单播去实现和私有云一样的效果，但是虚拟出来的IP地址默认是不被路由的，意思就是无法从外界访问的。那怎么办呢，Azure考虑到这一块，因此在LB的实现上提供一个叫做Floating IP的功能。

请参考下图:
我在srv-lambert-haproxyvm0和srv-lambert-haproxyvm1上安装了haproxy和keepalived，配置haproxy使其向后端srv-lambert-app1和srv-lambert-app2分发80的请求，配置keepalived使其周期性的对各自haproxy做健康检查，

![](https://raw.githubusercontent.com/Leejung168/azure/master/Azure-LB.png)

1. LB的配置截图： 注意这里在配置LB rule的时候开启了floating IP. 在配置probe的时候我直接用80，用80不好，因为后端的两台VM上都有80的监听，因此lb的健康检查都过，这样会导致用户的请求分发到没有虚拟IP地址的VM上。但是只是单纯的做实验，生产环境下像我们的SQL always-on的probe设置的是59999,59999在同一时刻只会监听的主的SQL服务器上。所以用户的请求只会分发到主的sql服务器上。

![](https://raw.githubusercontent.com/Leejung168/azure/master/lb-mainpage.png)
![](https://raw.githubusercontent.com/Leejung168/azure/master/lb-backend.png)
![](https://raw.githubusercontent.com/Leejung168/azure/master/lb-probe.png)
![](https://raw.githubusercontent.com/Leejung168/azure/master/lb-rule.png)

2. keepalive配置文件

```
root@srv-lambert-haproxyvm0:~$ cat /etc/keepalived/keepalived.conf
vrrp_script chk_appsvc {    
    script /usr/local/sbin/keepalived-check-appsvc.sh   #这是配置健康检查的脚步，对haproxy的健康进行周期性探测
    interval 1
    fall 2
    rise 2
}

vrrp_instance VI_1 {
    interface eth0

    authentication {
        auth_type PASS
        auth_pass secr3t
    }

    virtual_router_id 51

    virtual_ipaddress {
        20.184.11.91        # 这里配置了LB的frontend IP地址
    }

    track_script {
        chk_appsvc
    }

    notify /usr/local/sbin/keepalived-action.sh
    notify_stop "/usr/local/sbin/keepalived-action.sh INSTANCE VI_1 STOP"


    state MASTER
    priority 101

    unicast_src_ip 10.0.0.6
    unicast_peer {
        10.0.0.7
    }

}
-
-
root@srv-lambert-haproxyvm1:~# cat /etc/keepalived/keepalived.conf
vrrp_script chk_appsvc {
    script /usr/local/sbin/keepalived-check-appsvc.sh
    interval 1
    fall 2
    rise 2
}

vrrp_instance VI_1 {
    interface eth0

    authentication {
        auth_type PASS
        auth_pass secr3t
    }

    virtual_router_id 51

    virtual_ipaddress {
        20.184.11.91
    }

    track_script {
        chk_appsvc
    }

    notify /usr/local/sbin/keepalived-action.sh
    notify_stop "/usr/local/sbin/keepalived-action.sh INSTANCE VI_1 STOP"


    state BACKUP
    priority 100

    unicast_src_ip 10.0.0.7
    unicast_peer {
        10.0.0.6
    }

}
```

3. 检查网络接口，发现20.184.11.91只绑定在srv-lambert-haproxyvm0的网络接口上，srv-lambert-haproxyvm1的网络接口没有.


```
root@srv-lambert-haproxyvm0:~# ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0d:3a:a2:32:38 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.6/24 brd 10.0.0.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet 20.184.11.91/32 scope global eth0    #请看这里
       valid_lft forever preferred_lft forever
    inet6 fe80::20d:3aff:fea2:3238/64 scope link
       valid_lft forever preferred_lft forever
-     
-
root@srv-lambert-haproxyvm1:~# ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0d:3a:a3:2d:56 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.7/24 brd 10.0.0.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::20d:3aff:fea3:2d56/64 scope link
       valid_lft forever preferred_lft forever
```

4. 网站访问
![](https://raw.githubusercontent.com/Leejung168/azure/master/access_test1.png)

5. 停掉srv-lambert-haproxyvm0的haproxy服务，发现ip地址飘到srv-lambert-haproxyvm2上了。
![](https://raw.githubusercontent.com/Leejung168/azure/master/stop_haproxy_onvm1.png)

6. 再次访问网站
![](https://raw.githubusercontent.com/Leejung168/azure/master/access_test2.png)

####总结:
由于在云上，IP地址的资源需要申请才能被路由，借助Azure LB的Floating IP这一功能和Windows的Failover Cluster或开源的Keepalived等工具可以实现虚拟IP地址的浮动。 请注意在我的实验环境IP在同一时刻Azure LB的frontend IP地址只会在其中一台VM的网络接口上。但是我们是可以同时配置同一个frontend IP在多台backend VM。







