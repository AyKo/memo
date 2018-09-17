# 品質の悪いネットワークを模擬したい

## 動機
[TCPでネットワークの断絶を想定する](network_problem.md) で述べたような状況下を再現させプログラムをデバッグしたい。

## 回答
iptablesを用いれば一部の通信のみをブロックできる。netemを用いると品質の悪いネットワークを模擬できる。

### iptablesを用いて一部の通信のみをブロックする
iptablesコマンドを用いると様々なことができるが、もっとも大きな役割はファイアウォールである。
この機能を使って一部の通信のみを遮断できるので、不意なネットワークの断絶を模擬できる。

頻出パターンを以下に述べる。なお、詳細は[man iptables](https://linuxjm.osdn.jp/html/iptables/man8/iptables.8.html)で読める。
- 指定のIPアドレスのからの通信を遮断（例：192.168.10.1からの通信を遮断）  
  ```iptables -A INPUT -s 192.168.10.1 -j DROP```
- 指定のTCPポート番号への通信を遮断（例：TCP8080ポートへの通信を遮断）  
  ```iptables -A OUTPUT -p tcp --dport 8080 -j DROP```
- 現在の設定を見る  
  ```iptables -nv -L```  
　※上記を削除する場合「-A」の代わりに「-D」を指定する。

### netemを用いて品質の悪いネットワークを模擬する
tcコマンドを用いると帯域制御などをおこなうことができる。また、tcのモジュールの一つであるNetwork Emulator機能(netem)を用いると品質の悪いネットワークを模擬できる。

- 帯域制御をおこなう（eth1の出力を1Mbpsにする）  
  ``` tc qdisc add dev eth1 root tbf rate 1mbit latency 50ms burst 10kb ```
- 通信の遅延を模擬する（eth1の出力を200ms遅延させる）  
  ```tc qdisc add dev eth1 root netem delay 200ms```
- パケットロスを模擬する（eth1の出力を5%ロスする）  
  ```tc qdisc add dev eth1 root netem loss 5%```
- 帯域制御＋遅延＋パケットロス（eth1の出力を1Mbps・100ms遅延・5%ロス）  
  ```tc qdisc add dev eth1 root handle 1:1 tbf rate 1mbit latency 50ms buffer 10kb```  
  ```tc qdisc add dev eth1 parent 1:1 netem delay 100ms loss 5%```
- 設定を抹消する（eth1）  
  ```tc qdisc del dev eth1 root```
	
## 参考情報
コマンドの使い方
- [man iptables](https://linuxjm.osdn.jp/html/iptables/man8/iptables.8.html)
- [man tc](http://man7.org/linux/man-pages/man8/tc.8.html)、[man tc-tbf](http://man7.org/linux/man-pages/man8/tc-tbf.8.html)、[man tc-netem](http://man7.org/linux/man-pages/man8/tc-netem.8.html)など

Linuxの高度なルーティングや帯域制御の包括的な資料
- [Linux Advanced Routing & Traffic Control HOWTO](http://archive.linux.or.jp/JF/JFdocs/Adv-Routing-HOWTO/index.html)
