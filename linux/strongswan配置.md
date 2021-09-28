# strongswan配置

## 环境

**strongswan版本**

```
]# strongswan --version
Linux strongSwan U5.7.2/K3.10.0-693.el7.x86_64
University of Applied Sciences Rapperswil, Switzerland
See 'strongswan --copyright' for copyright information.
```

**系统版本**

```
CentOS Linux release 7.4.1708 (Core)
3.10.0-693.el7.x86_64
```

**拓扑**

10.0.32.5/20 - 10.0.66.3/24(114.67.108.198) --- (116.196.103.113)192.168.21.11/24 - 192.168.1.171/20

## strongswan.conf

**/etc/strongswan/strongswan.conf配置**

```
# strongswan.conf - strongSwan configuration file
#
# Refer to the strongswan.conf(5) manpage for details
#
# Configuration changes should be made in the included files

charon {
        load_modular = yes
        i_dont_care_about_security_and_use_aggressive_mode_psk = yes
        plugins {
            include strongswan.d/charon/*.conf
        }
    filelog {
        charon {
            # path to the log file, specify this as section name in versions prior to 5.7.0
            path = /var/log/charon.log
            # add a timestamp prefix
            time_format = %b %e %T
            # prepend connection name, simplifies grepping
            ike_name = yes
            # overwrite existing files
            append = yes
            # increase default loglevel for all daemon subsystems
            default = 2
            # flush each line to disk
            flush_line = yes
        }
        stderr {
            # more detailed loglevel for a specific subsystem, overriding the
            # default loglevel.
            ike = 2
            knl = 3
        }
    }
}

include strongswan.d/*.conf

```



## IKEv1主模式配置

**10.0.66.3/24配置/etc/strongswan/swanctl/conf.d/ikev1_main.conf**

```
connections {

   conn-bj-sh-2 {
      version = 1
      aggressive = no
      proposals = aes128-sha1-modp1024

      local_addrs  = 10.0.66.3
      remote_addrs = 116.196.103.113

      local {
         id = 114.67.108.198
         auth = psk
      }
      remote {
         id = 116.196.103.113
         auth = psk
      }
      children {
         conn-bj-sh-2 {
            local_ts  = 10.0.32.0/20
            remote_ts = 192.168.0.0/20
            esp_proposals = aes128-sha1
            dpd_action = restart
            start_action = start
            close_action = start
         }
      }
   }
}

secrets {

   ike-conn-bj-sh-2 {
      id = 116.196.103.113
      secret = Rz2UhwQYYAs79AXs
   }
}
```

**192.168.21.11/24配置/etc/strongswan/swanctl/conf.d/ikev1_main.conf**

```
connections {

   conn-bj-sh-2 {
      version = 1
      aggressive = no
      proposals = aes128-sha1-modp1024

      local_addrs  = 192.168.21.11
      remote_addrs = 114.67.108.198

      local {
         id = 116.196.103.113
         auth = psk
      }
      remote {
         id = 114.67.108.198
         auth = psk
      }
      children {
         conn-bj-sh-2 {
            local_ts  = 192.168.0.0/20
            remote_ts = 10.0.32.0/20
            esp_proposals = aes128-sha1
            dpd_action = restart
            start_action = start
            close_action = start
         }
      }
   }
}

secrets {

   ike-conn-bj-sh-2 {
      id = 114.67.108.198
      secret = Rz2UhwQYYAs79AXs
   }
}

```

## IKEv1野蛮模式配置

>  注意：strongswan配置野蛮模式的时候，一定要做如下配置，否则会导致日志中报错：[IKE] <2> Aggressive Mode PSK disabled for security reasons

```
i_dont_care_about_security_and_use_aggressive_mode_psk = yes
```



**10.0.66.3/24配置/etc/strongswan/swanctl/conf.d/ikev1_aggresive.conf**

```
connections {

   conn-bj-sh-2 {
      version = 1
      aggressive = yes
      proposals = aes128-sha1-modp1024

      local_addrs  = 10.0.66.3
      remote_addrs = 116.196.103.113

      local {
         id = 114.67.108.198
         auth = psk
      }
      remote {
         id = 116.196.103.113
         auth = psk
      }
      children {
         conn-bj-sh-2 {
            local_ts  = 10.0.32.0/20
            remote_ts = 192.168.0.0/20
            esp_proposals = aes128-sha1
            dpd_action = restart
            start_action = start
            close_action = start
         }
      }
   }
}

secrets {

   ike-conn-bj-sh-2 {
      id = 116.196.103.113
      secret = Rz2UhwQYYAs79AXs
   }
}
```



**192.168.21.11/24配置/etc/strongswan/swanctl/conf.d/ikev1_aggresive.conf**

```
connections {

   conn-bj-sh-2 {
      version = 1
      aggressive = yes
      proposals = aes128-sha1-modp1024

      local_addrs  = 192.168.21.11
      remote_addrs = 114.67.108.198

      local {
         id = 116.196.103.113
         auth = psk
      }
      remote {
         id = 114.67.108.198
         auth = psk
      }
      children {
         conn-bj-sh-2 {
            local_ts  = 192.168.0.0/20
            remote_ts = 10.0.32.0/20
            esp_proposals = aes128-sha1
            dpd_action = restart
            start_action = start
            close_action = start
         }
      }
   }
}

secrets {

   ike-conn-bj-sh-2 {
      id = 114.67.108.198
      secret = Rz2UhwQYYAs79AXs
   }
}

```



## IKEv2配置

**10.0.66.3/24配置/etc/strongswan/swanctl/conf.d/ikev2.conf**

```
connections {
    conn-bj-sh {
        version = 2
        local_addrs = 10.0.66.3
        remote_addrs = 116.196.103.113
        proposals = aes128-sha1-modp1024
        dpd_delay = 20s
        local {
            id = 114.67.108.198
            auth = psk
        }
        remote {
            id = 116.196.103.113
            auth = psk
        }
        children {
            conn-bj-sh {
                local_ts = 10.0.32.0/20
                remote_ts = 192.168.0.0/20
                dpd_action = restart
                start_action = start
                close_action = start
                esp_proposals = aes128-sha1
            }
        }
    }
}
secrets {
    ike-conn-bj-sh {
        secret = Rz2UhwQYYAs79AXs
        id = 116.196.103.113
    }
}

```

**192.168.21.11/24配置/etc/strongswan/swanctl/conf.d/ikev2.conf**

```
connections {
    conn-bj-sh {
        version = 2
        local_addrs = 192.168.21.11
        remote_addrs = 114.67.108.198
        proposals = aes128-sha1-modp1024
        dpd_delay = 20s
        local {
            id = 116.196.103.113
            auth = psk
        }
        remote {
            id = 114.67.108.198
            auth = psk
        }
        children {
            conn-bj-sh {
                local_ts = 192.168.0.0/20
                remote_ts = 10.0.32.0/20
                dpd_action = restart
                start_action = start
                close_action = start
                esp_proposals = aes128-sha1
            }
        }
    }
}
secrets {
    ike-conn-bj-sh {
        secret = Rz2UhwQYYAs79AXs
        id = 114.67.108.198
    }
}
```



## swanctl命令的使用

```
swanctl --terminate --child <name> --child-id <id>

swanctl --list-sas         (-l)  list currently active IKE_SAs
swanctl --load-all         (-q)  load credentials, authorities, pools and connections
swanctl --list-conns       (-L)  list loaded configurations
swanctl --load-conns       (-c)  (re-)load connection configuration
```

