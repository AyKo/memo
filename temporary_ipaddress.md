# 一時的にIPアドレスの付与・削除をしたい

## 動機
試験用に一時的にIPアドレスの付与・削除したい。

## 回答
一時的なIPアドレスの追加は次のようにコマンドを入力する。ただし、そのネットワークでIPアドレスの重複が起きないように注意する：

    ifconfig [デバイス名]:[任意の英数字] [IPアドレス]/[マスク値]
　
上記で追加したIPアドレスの削除は次のようにコマンドを入力する。

    ifconfig [デバイス名]:[追加で指定した英数字] down』

### 例
IPアドレスの表示：

    [root@virt-centos6 ~]# ifconfig
    eth0      Link encap:Ethernet  HWaddr XX:XX:XX:XX:XX:XX 
            inet addr:192.168.10.8  Bcast:192.168.10.255  Mask:255.255.255.0
            inet6 addr: fe80::a00:xxxx:xxxx:xxxx/64 Scope:Link
            UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
            RX packets:150 errors:0 dropped:0 overruns:0 frame:0
            TX packets:88 errors:0 dropped:0 overruns:0 carrier:0
            collisions:0 txqueuelen:1000 
            RX bytes:18808 (18.3 KiB)  TX bytes:13364 (13.0 KiB)

    lo        Link encap:Local Loopback  
            inet addr:127.0.0.1  Mask:255.0.0.0
            inet6 addr: ::1/128 Scope:Host
            UP LOOPBACK RUNNING  MTU:65536  Metric:1
            RX packets:0 errors:0 dropped:0 overruns:0 frame:0
            TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
            collisions:0 txqueuelen:0 
            RX bytes:0 (0.0 b)  TX bytes:0 (0.0 b)

一時的にIPアドレスを追加：

    [root@virt-centos6 ~]# ifconfig eth0:TEST 192.168.250.250/24
    [root@virt-centos6 ~]# ifconfig
    eth0      Link encap:Ethernet  HWaddr XX:XX:XX:XX:XX:XX 
            inet addr:192.168.10.8  Bcast:192.168.10.255  Mask:255.255.255.0
            inet6 addr: fe80::a00:xxxx:xxxx:xxxx/64 Scope:Link
            UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
            RX packets:434 errors:0 dropped:0 overruns:0 frame:0
            TX packets:272 errors:0 dropped:0 overruns:0 carrier:0
            collisions:0 txqueuelen:1000 
            RX bytes:43256 (42.2 KiB)  TX bytes:32868 (32.0 KiB)

    eth0:TEST Link encap:Ethernet  HWaddr XX:XX:XX:XX:XX:XX 
            inet addr:192.168.250.250  Bcast:192.168.250.255  Mask:255.255.255.0
            UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1

    lo        Link encap:Local Loopback  
            inet addr:127.0.0.1  Mask:255.0.0.0
            inet6 addr: ::1/128 Scope:Host
            UP LOOPBACK RUNNING  MTU:65536  Metric:1
            RX packets:0 errors:0 dropped:0 overruns:0 frame:0
            TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
            collisions:0 txqueuelen:0 
            RX bytes:0 (0.0 b)  TX bytes:0 (0.0 b)

IPアドレスの削除：

    [root@virt-centos6 ~]# ifconfig eth0:TEST down
    [root@virt-centos6 ~]# ifconfig
    eth0      Link encap:Ethernet  HWaddr XX:XX:XX:XX:XX:XX 
            inet addr:192.168.10.8  Bcast:192.168.10.255  Mask:255.255.255.0
            inet6 addr: fe80::a00:xxxx:xxxx:xxxx/64 Scope:Link
            UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
            RX packets:503 errors:0 dropped:0 overruns:0 frame:0
            TX packets:313 errors:0 dropped:0 overruns:0 carrier:0
            collisions:0 txqueuelen:1000 
            RX bytes:48782 (47.6 KiB)  TX bytes:37162 (36.2 KiB)

    lo        Link encap:Local Loopback  
            inet addr:127.0.0.1  Mask:255.0.0.0
            inet6 addr: ::1/128 Scope:Host
            UP LOOPBACK RUNNING  MTU:65536  Metric:1
            RX packets:0 errors:0 dropped:0 overruns:0 frame:0
            TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
            collisions:0 txqueuelen:0 
            RX bytes:0 (0.0 b)  TX bytes:0 (0.0 b)

