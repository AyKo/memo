# プログラム動作中に内部状態を調査したい

## 動機
プログラムの動作が何かおかしい。不正終了するわけではないが、プログラムの内部状態を調査したい。

## 回答
動作中のプログラムにgdbを接続するか、gcoreコマンドを使ってメモリのスナップショット(coreダンプ)をとる。
ただし、どちらにしてもプログラムに多少の影響を与えるので、運用中のプログラムに実施する場合は注意すること(責任者に許可をもらう)。

## 動作中のプログラムにgdbを接続する
『gdb -p [PID] [対象実行ファイル]』を実行する。

デバッガでアタッチする、ともいう。プログラムは一時停止する。
デバッガ内で再開(continue)するか、デタッチ(detach)するか、デバッガを終了すると再開する。

なお、デバッガ内で再開する場合は、SIGPIPEの補足が邪魔になることが多い（プログラム内で無視指定してもデバッガが補足して勝手に一時停止する）。
その場合は、あらかじめ「handle SIGPIPE nostop nopass」を実行しておけば邪魔されない。

## メモリのスナップショットをとる
『gcore [PID]』を実行する。仮想メモリの内容をすべてファイルに出力するため、仮想メモリサイズ(VSZ)に比例して時間がかかる。
ファイルに出力中はプログラムは一時停止している。

# 参考情報
[man gdb](https://linuxjm.osdn.jp/html/GNU_gdb/man1/gdb.1.html)
