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
