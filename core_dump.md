# プログラムが不正終了するがCoreダンプがない

## 動機
プログラムが意図しないタイミングで終了した。
多分そのプログラムの問題と思われ解析したかったが、Coreダンプが出力されていない。

## 回答
意図しないプログラムダウン時にでCoreダンプが出力されない場合、我々のシステムでよくあるケースは次の２つ：
- Core File Sizeの設定が初期値(0)のままになっている
- SIGPIPEでダウンした

次点で：
- SIGHUPでダウンした
- メモリ枯渇でOOM-Killerが発生

それ以外の場合はほとんど見たことがない（あるいは気づいていない）。

### Core File Sizeの設定が初期値(0)のまま
ディフォルトではCore File Size（Coreファイル出力サイズの最大値）の設定が0になっているので異常終了時でもCoreダンプは出力されない。
該当プログラムを起動するシェルスクリプトで『ulimit -c unlimited』をあらかじめ実行しておく。

ulimitは別シェルやプロセスで実行しても意味がない。この設定はプロセスごとの設定であり、また子プロセス側にしか伝搬しないため。
また、安全に後付けで設定することはできない（gdbを用いて強引に変える方法はある）。
動作中のプロセスがどのような設定になっているかを知るためには、「/proc/<PID>/limits」をcatコマンド等で見ればよい。

### SIGPIPEでダウンした
上記の設定でCore File Size の設定をしてあったとしても、プログラムダウンの原因がSIGPIPEの場合はコアダンプが出力されず単に終了する。

特にネットワークを取り扱う場合は、SIGPIPEは日常的に発生すると考えること。
ほとんどの場合はプログラム起動時に無視するように設定しておくこと。
無視しないようにした場合、write()やsend()で -1 が返ってくるようなケース、例えばTCP接続で相手が通信終了後にデータを送信しようとしたりするとSIGPIPEが発生する。

### SIGHUPでダウンした
Teratermなどの端末からプログラムを起動した場合、端末の終了時にその端末から起動したプログラムが終了させられることがある。
フォアグラウンド起動の場合はもちろん、たとえバックグラウンド起動していたとしても終了するときは終了させられる。

なお、もしこのようにバックグラウンドで動作させる場合は、nohupコマンドを使うとよい。SIGHUPを無視してプログラムを起動する。

### メモリ枯渇でOOM-Killerが発生
メモリが枯渇してカーネルがメモリ確保ができなくなった。
その場合、カーネルはOOM-Killerという機能を起動してプログラムを強制終了させる。
ただし、メモリ枯渇の原因となったプロセスが終了させられるとは限らない。
OOM-Killerによって終了した場合は、syslogに残るので確認する。

## 参考情報
ulimitについて
- [man bash](https://linuxjm.osdn.jp/html/GNU_bash/man1/bash.1.html)
- [man setrlimit](https://linuxjm.osdn.jp/html/LDP_man-pages/man2/getrlimit.2.html)

SIGPIPE
- [man 7 signal](https://linuxjm.osdn.jp/html/LDP_man-pages/man7/signal.7.html)  
  各シグナルのディフォルト動作が書いてある。
- [man 2 write](https://linuxjm.osdn.jp/html/LDP_man-pages/man2/write.2.html)  
  「EPIPE」の欄

SIGHUP  
- [なぜnohupをバックグランドジョブとして起動するのが定番なのか？(擬似端末, Pseudo Terminal, SIGHUP他)](https://www.glamenv-septzen.net/view/854)
