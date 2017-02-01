# keepalived-cfg
lvs+keepalived配置脚本

通过脚本参数参数配置keepalived.conf，配置格式如下：（支持自定义模版的lvs健康检查）
```
! Configuration File for keepalived

global_defs {
   notification_email {
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 192.168.200.1
   smtp_connect_timeout 30
   router_id demo
}
vrrp_instance demo {
    state MASTER
    interface eth0
    virtual_router_id 10
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass demo
    }
    virtual_ipaddress {
        10.0.0.1
        10.0.0.2
    }
}
virtual_server 10.0.0.1 80 {
    delay_loop 6
    lb_algo rr
    lb_kind DR
    nat_mask 255.0.0.0
    persistence_timeout 60
    protocol TCP
    real_server 10.0.100.1 80 {
        weight 1
        TCP_CHECK {
            connect_port 80
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
    real_server 10.0.100.2 80 {
        weight 1
        TCP_CHECK {
            connect_port 80
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
}
virtual_server 10.0.0.2 8080 {
    delay_loop 6
    lb_algo wrr
    lb_kind DR
    nat_mask 255.0.0.0
    persistence_timeout 60
    protocol TCP
    real_server 10.0.200.1 8080 {
        weight 1
        TCP_CHECK {
            connect_port 8080
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
    real_server 10.0.200.2 8080 {
        weight 1
        TCP_CHECK {
            connect_port 8080
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
}
```

## 脚本参数说明
```
Usage:
  keepalived-cfg --init --rid router_id
  keepalived-cfg --add-vrrp --name vrrp_name --vid virtual_router_id --vip virtual_ipaddress [options]
  keepalived-cfg --edit-vrrp --name vrrp_name -P|--priority priority
  keepalived-cfg --delete-vrrp --name vrrp_name
  keepalived-cfg -A|E -t|u service-address [-s scheduler] [-M|--netmask netmask]
  keepalived-cfg -D -t|u service-address
  keepalived-cfg -a|e -t|u service-address -r server-address [options]
  keepalived-cfg -d -t|u service-address -r server-address
  keepalived-cfg -L|l
  keepalived-cfg -H -t|u service-address -r server-address
  keepalived-cfg -S -t|u service-address -r server-address
  keepalived-cfg -h

Commands:
Either long or short options are allowed.
  --init                      init keepalived.conf
  --add-vrrp                  add vrrp instance with options
  --edit-vrrp                 edit vrrp instance with options
  --delete-vrrp               delete vrrp instance
  --add-service     -A        add virtual service with options
  --edit-service    -E        edit virtual service with options
  --delete-service  -D        delete virtual service
  --add-server      -a        add real server with options
  --edit-server     -e        edit real server with options
  --delete-server   -d        delete real server
  --list            -L|-l     list with json
  --show-server     -S        show real server
  --hide-server     -H        hide real server
  --help            -h        display this help message

Options:
  --name                              vrrp instance
  --state                             vrrp state [MASTER|BACKUP]
                                      the default state is MASTER
  --interface    -I                   vrrp interface
  --vid                               vrrp virtual_router_id [1-255]
  --priority     -P                   vrrp priority [1-100]
                                      the default priority is 100
  --password                          authentication password
  --vip                               virtual_ipaddress is host
  --tcp-service  -t service-address   service-address is host[:port]
  --udp-service  -u service-address   service-address is host[:port]
  --scheduler    -s scheduler         one of rr|wrr|lc|wlc|lblc|lblcr|dh|sh|sed|nq,
                                      the default scheduler is wrr.
  --persistent   -p [timeout]         persistent service
                                      the default persistent timeout is 60
  --netmask      -M netmask           persistent granularity mask
  --real-server  -r server-address    server-address is host (and port)
  --gatewaying   -g                   gatewaying (direct routing) (default)
  --ipip         -i                   ipip encapsulation (tunneling)
  --masquerading -m                   masquerading (NAT)
  --weight       -w weight            capacity of real server [1-100]
                                      the default weight is 1
  --check        -c URL               check template of real-server
```
## 注意事项
```
需安装keepalived及ipvsadm
根据实际情况修改脚本中19-33行及48-60行内容
```
## Demo
```
1、初始化配置文件，指定router_id为demo（清空所有配置）
    keepalived-cfg --init --rid demo
2、添加vrrp_instance，指定名称为demo，声明为主服务器，使用网卡eth0，virtual_router_id为10，优先级100，认证密码demo，虚拟ip为10.0.0.1和10.0.0.2
    keepalived-cfg --add-vrrp --name demo --state MASTER -I eth0 --vid 10 --priority 100 --password demo --vip 10.0.0.1,10.0.0.2
3、添加virtual_server，TCP协议，80端口，加权轮询调度，DR模式，超时时间120秒
    keepalived-cfg -A -t 10.0.0.1:80 -M 255.0.0.0 -s wrr -g -p 120
4、添加real_server,权重为3，使用默认TCP_CHECK检查
    keepalived-cfg -a -t 10.0.0.1:80 -r 10.0.100.1 -w 3
5、添加real_server,权重为5，使用自定义模版，地址为http://www.example.com/demo
    keepalived-cfg -a -t 10.0.0.1:80 -r 10.0.100.2 -w 5 -c "http://www.example.com/demo"
    #模版内容：
    #[root@localhost ~]# curl -s http://www.example.com/demo
    #HTTP_GET {
    #    url {
    #        path /check.html
    #        digest cc2dbff2ec2178dfa42e20b5sa5dv39d
    #    }
    #    connect_timeout 3
    #    nb_get_retry 3
    #    delay_before_retry 3
    #}
6、删除vrrp_instance（将删除包含该vrrp虚拟IP的所有virtual_server）
    keepalived-cfg --delete-vrrp --name demo
7、删除virtual_server（将删除该virtual_server下所有real_server）
    keepalived-cfg -D -t 10.0.0.1:80
8、删除real_server
    keepalived-cfg -d -t 10.0.0.1:80 -r 10.0.100.2
9、隐藏/注释real_server（用于临时摘除real_server）
    keepalived-cfg -H -t 10.0.0.1:80 -r 10.0.100.1
10、显示/取消注释real_server（用于恢复临时摘除的real_server）
    keepalived-cfg -S -t 10.0.0.1:80 -r 10.0.100.1
11、修改vrrp优先级
    keepalived-cfg --edit-vrrp --name demo --priority 90
12、修改virtual_server调度模式
    keepalived-cfg -E -t 10.0.0.1:80 -s rr
13、修改real_server权重
    keepalived-cfg -e -t 10.0.0.1:80 -r 10.0.100.1 -w 5
```

## 脚本错误推出码说明
```
1 :无效选项
2 :参数格式错误
3 :缺少必要参数
4 :未找到
5 :无法访问Check模版地址
11:vrrp name已存在
12:vrrp name不存在
13:virtual route id已存在
15:virtual ipaddress已存在
16:virtual ipaddress不存在
21:virtual server已存在
22:virtual server不存在
31:real server已存在
32:real server不存在
```
