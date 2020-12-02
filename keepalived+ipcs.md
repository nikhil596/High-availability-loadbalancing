#  KeepAllived + IPVS for High Availability Setup 


```


Keepalived and IPVS installed on lb1 and lb2  
VRPP router  ( Virtual-IP  provisined for keepalived ) 

Lb1            Lb2 
___            ___
| |  - - - - - | |
---            ---
 |   \          |
___   \        ___
| |  - - - - - | |
---            ---
Real1          Real2 

Connections from lb1 that holds the VirtualIP to both servers 





# Network Setting 

/etc/sysctl.conf 

net.ipv4.ip_forward=1
net.ipv4.conf.all.rp_filter=1
net.ipv4.icmp_echo_ignore_broadcasts=1
net.ipv4.conf.default.proxy_arp=1

net.ipv4.conf.all.promote_secondaries=1
net.ipv4.conf.all.arp_ignore = 1
net.ipv4.conf.all.arp_announce = 2

net.ipv4.conf.virbr0.arp_ignore = 1
net.ipv4.conf.virbr0.arp_announce = 2
net.ipv4.conf.eth1.arp_announce = 0

```


### Loadbalancer1 
```
! Configuration File for keepalived

global_defs {
   notification_email {
       your_email@admin.com
   }
   notification_email_from loadbalancer1@admin.com
   smtp_server localhost
   smtp_connect_timeout 30
! UNIQUE:
   router_id LVS_PRI
}

! ***********************************************************************
! *************************   WEB SERVICES VIP  *************************
! ***********************************************************************
vrrp_instance VirtIP_10 {
    state MASTER
    interface eth0
    virtual_router_id 10
! UNIQUE:
    priority 150
    advert_int 3
    smtp_alert
    authentication {
        auth_type PASS
        auth_pass MY_PASS
    }
    virtual_ipaddress {
        10.10.10.10
    }

    lvs_sync_daemon_interface eth0
}

! ************************   WEB SERVERS  **************************

virtual_server 10.10.10.10 80 {
    delay_loop 10
    lvs_sched wlc
    lvs_method DR
    persistence_timeout 5
    protocol TCP

    real_server 10.10.10.41 80 {
        weight 50
        TCP_CHECK {
            connect_timeout 3
        }
    }

    real_server 10.10.10.42 80 {
        weight 50
        TCP_CHECK {
            connect_timeout 3
        }
    }

}


```
### loadbalancer2 
```
! Configuration File for keepalived

global_defs {
   notification_email {
        your_email@admin.com
   }
   notification_email_from loadbalancer2@admin.com
   smtp_server localhost
   smtp_connect_timeout 30
! UNIQUE:
   router_id LVS_SEC
}

! ***********************************************************************
! *************************   WEB SERVICES VIP  *************************
! ***********************************************************************

vrrp_instance VirtIP_10 {
    state BACKUP
    interface eth0
    virtual_router_id 10
! UNIQUE:
    priority 50
    advert_int 3
    smtp_alert
    authentication {
        auth_type PASS
        auth_pass MY_PASS
    }
    virtual_ipaddress {
        10.10.10.10
    }

    lvs_sync_daemon_interface eth0
}

! ************************   WEB SERVERS  **************************

virtual_server 10.10.10.10 80 {
    delay_loop 10
    lvs_sched wlc
    lvs_method DR
    persistence_timeout 5
    protocol TCP

    real_server 10.10.10.41 80 {
        weight 50
        TCP_CHECK {
            connect_timeout 3
        }
    }

    real_server 10.10.10.42 80 {
        weight 50
        TCP_CHECK {
            connect_timeout 3
        }
    }
}
```

