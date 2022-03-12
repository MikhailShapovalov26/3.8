# 8. Компьютерные сети, лекция 3
## 1.
Вот запись в таблице маршрутизации

        route-views>show ip route 188.243.183.194
        Routing entry for 188.243.0.0/16
        Known via "bgp 6447", distance 20, metric 0
        Tag 3333, type external
        Last update from 193.0.0.56 2d20h ago
        Routing Descriptor Blocks:
        * 193.0.0.56, from 193.0.0.56, 2d20h ago
            Route metric is 0, traffic share count is 1
            AS Hops 4
            Route tag 3333
            MPLS label: none

CEF

        route-views>show ip cef 188.243.183.194 detail 
        188.243.0.0/16, epoch 2, flags [rib only nolabel, rib defined all labels]
        recursive via 193.0.0.56
            recursive via 128.223.51.1
            attached to TenGigabitEthernet0/0/0

        route-views>show bgp 188.243.183.194
        BGP routing table entry for 188.243.0.0/16, version 313229960
        Paths: (1 available, best #1, table default)
        Not advertised to any peer
        Refresh Epoch 1
        3333 1103 20562 35807
            193.0.0.56 from 193.0.0.56 (193.0.0.56)
            Origin IGP, localpref 100, valid, external, atomic-aggregate, best
            Community: 20562:45 20562:3120 20562:4001 20562:65000 20562:65020
            path 7FE113CCBC88 RPKI State not found
            rx pathid: 0, tx pathid: 0x0
## 2.
К сожалению не получилось запустить интерфейс dummy0
Нашёл инструкцию в сети интернет
        sudo modprobe -v dummy numdummies=2
Далее
        lsmod | grep dummy
        dummy                  16384  
Но интерфейс не поднялся 

        $ifconfig -a | grep dummy (Вывода не было)

            $ ifconfig -a 
    enp3s0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
            ether 74:46:a0:b8:1c:c9  txqueuelen 1000  (Ethernet)
            RX packets 0  bytes 0 (0.0 B)
            RX errors 0  dropped 0  overruns 0  frame 0
            TX packets 0  bytes 0 (0.0 B)
            TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
            device interrupt 18  

    lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
            inet 127.0.0.1  netmask 255.0.0.0
            inet6 ::1  prefixlen 128  scopeid 0x10<host>
            loop  txqueuelen 1000  (Локальная петля (Loopback))
            RX packets 243068  bytes 16012880 (16.0 MB)
            RX errors 0  dropped 0  overruns 0  frame 0
            TX packets 243068  bytes 16012880 (16.0 MB)
            TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

    wlx7898e8c1c8b3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
            inet 192.168.88.238  netmask 255.255.255.0  broadcast 192.168.88.255
            inet6 fe80::f3a5:1bb:cb78:a5fc  prefixlen 64  scopeid 0x20<link>
            ether 78:98:e8:c1:c8:b3  txqueuelen 1000  (Ethernet)
            RX packets 314395  bytes 297279494 (297.2 MB)
            RX errors 0  dropped 0  overruns 0  frame 0
            TX packets 144629  bytes 23767413 (23.7 MB)
            TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
К сожалению интерфейс не поднялся

Дополнительно*

                sudo modprobe -v dummy
                insmod /lib/modules/5.4.0-91-generic/kernel/drivers/net/dummy.ko numdummies=0 
                lsmod | grep dummy
                dummy                  16384  0
                Результат проверки
                sudo ifconfig -a | grep dummy(вывода не было)
                sudo ip addr add 192.168.1.150/24 dev dummy0
                Cannot find device "dummy0"
** Дополнение 2

                sudo modprobe -v dummy numdummies=2
                [sudo] пароль для mikhail:         
                $ lsmod | grep dummy
                dummy                  16384  0
                $ ifconfig -a | grep dummy
                $ (вывода нет)

Я перешёл на другую операционную систему. А именно попробовал данные команды на orangepi
-------------------------------

                root@orangepione:~# modprobe -v dummy numdummies=2
                insmod /lib/modules/5.15.25-sunxi/kernel/drivers/net/dummy.ko.xz numdummies=0 numdummies=2
                root@orangepione:~# lsmod | grep dummy
                dummy                  16384  0
                root@orangepione:~# ifconfig -a
                dummy0: flags=130<BROADCAST,NOARP>  mtu 1500
                        ether 36:a7:8a:88:d9:cb  txqueuelen 1000  (Ethernet)
                        RX packets 0  bytes 0 (0.0 B)
                        RX errors 0  dropped 0  overruns 0  frame 0
                        TX packets 0  bytes 0 (0.0 B)
                        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

                dummy1: flags=130<BROADCAST,NOARP>  mtu 1500
                        ether 1e:29:93:ea:b3:a6  txqueuelen 1000  (Ethernet)
                        RX packets 0  bytes 0 (0.0 B)
                        RX errors 0  dropped 0  overruns 0  frame 0
                        TX packets 0  bytes 0 (0.0 B)
                        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0


## 3.
Смотрим открытые порты TCP -t (у меня mint)

        ss -tlpn
        State    Recv-Q   Send-Q     Local Address:Port     Peer Address:Port  Process  
        LISTEN   0        5              127.0.0.1:631           0.0.0.0:*              
        LISTEN   0        4096       127.0.0.53%lo:53            0.0.0.0:*              
        LISTEN   0        5                  [::1]:631              [::]:*  

    TCP-порт 53 использует протокол управления передачей данных (TCP), который является одним из основных протоколов в сетях TCP/IP. TCP является протоколом с установлением соединения и требует квитирования для установки сквозной связи. Только после установления соединения пользовательские данные могут пересылаться в обоих направлениях

        lsof | grep 4096
        systemd      808                          mikhail  cwd       DIR                8,2      4096          2 /
## 4.
        sudo ss -ulpn
        State          Recv-Q         Send-Q                                               Local Address:Port                    Peer Address:Port         Process                                            
        UNCONN         0              0                                                      224.0.0.251:5353                         0.0.0.0:*             users:(("chrome",pid=2056,fd=75))                 
        UNCONN         0              0                                                      224.0.0.251:5353                         0.0.0.0:*             users:(("chrome",pid=2104,fd=62))                 
        UNCONN         0              0                                                          0.0.0.0:5353                         0.0.0.0:*             users:(("avahi-daemon",pid=562,fd=12))            
        UNCONN         0              0                                                          0.0.0.0:39987                        0.0.0.0:*             users:(("avahi-daemon",pid=562,fd=14))            
        UNCONN         0              0                                                          0.0.0.0:41775                        0.0.0.0:*             users:(("chrome",pid=2104,fd=81))                 
        UNCONN         0              0                                                    127.0.0.53%lo:53                           0.0.0.0:*             users:(("systemd-resolve",pid=544,fd=12))         
        UNCONN         0              0                                                   192.168.88.238:123                          0.0.0.0:*             users:(("ntpd",pid=734,fd=23))                    
631 порт к примеру Сетевая печать (я не весь листинг сюда закинул)
## 5.
https://github.com/MikhailShapovalov26/3.8/blob/e2f78e3c829974b94845eb80e56896dcabfb5eb4/Untitled%20Diagram.drawio
## 6.
Честно, очень интересна эта тема с nginx но у меня с ним не складывается. Не понимаю я его. Нашёл в сети листинг 
        stream {
    upstream mysql_read {
        server read1.example.com:3306 weight=5;
        server read2.example.com:3306;
        server 10.10.12.34:3306 backup;
    }

    server {
        listen 3306;
        proxy_pass mysql_read;
    }
}
 Тут мы пытаемся балансировать нагрузку по порту 3306 и выполнять нагрузку между двумя репликами на чтение данных из MySQL. В случае, если 1 данные будут остановлены, мы начнём считывать из резервной копии.
 В итоге я вставил данный код sudo nano /etc/nginx/conf.d/upstreams.conf

        systemctl restart nginx.service 
        Job for nginx.service failed because the control process exited with error code.
        See "systemctl status nginx.service" and "journalctl -xe" for details.

        systemctl status nginx.service 
        ● nginx.service - A high performance web server and a reverse proxy server
            Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
            Active: failed (Result: exit-code) since Mon 2022-02-21 10:24:09 MSK; 6min ago
            Docs: man:nginx(8)
            Process: 160341 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_process on; (code=exited, status=1/FAILURE)

        фев 21 10:24:09 HP systemd[1]: Starting A high performance web server and a reverse proxy server...
        фев 21 10:24:09 HP nginx[160341]: nginx: [emerg] "stream" directive is not allowed here in /etc/nginx/conf.d/upstreams.conf:1
        фев 21 10:24:09 HP nginx[160341]: nginx: configuration file /etc/nginx/nginx.conf test failed
        фев 21 10:24:09 HP systemd[1]: nginx.service: Control process exited, code=exited, status=1/FAILURE
        фев 21 10:24:09 HP systemd[1]: nginx.service: Failed with result 'exit-code'.
        фев 21 10:24:09 HP systemd[1]: Failed to start A high performance web server and a reverse proxy server.
Могли бы Вы мне помочь в нём разобраться? Очень он мне нужен и интересен. 
