
# はじめに
本文書は、私の勤務先での技術や知識の伝達を目的とします。

勤務先での開発は、現状ではおおむね Redhat Enterprise Linux 6 および CentOS 6 を対象としています。
本文書の対象もこれらのOSとします。開発言語は C/C++ がメインとなります。
なお、開発にあたりほとんどはコンソール上からのアクセスであろうことを想定するため、
情報としてはコンソールアクセスによるものに偏っています。

これらの環境での開発をおこなうのに必要もしくは単に便利な情報を提供します。
他のOSでも有効なこともありますが、役に立たないこともあることもあるかと思います。
なお、私の経験や考えによるもの多く含むため、誤りおよび情報の偏りがあると思われるので、その点は注意が必要です。

本文書はこれらの情報に対し、見出しとその「動機」および「回答」、必要に応じてより詳しい情報を得るための「参考文書」を示します。

## 設計・コーディング

[複雑な設計・コード](complex_design_and_code.md)

[綺麗なコード](beautiful_code.md)

## デバッグ

[不正終了時のシグナル](fault_signal.md)

[デバッグ情報付きの実行ファイル](executable_with_debug_info.md)

[プログラムが不正終了するがCoreダンプがない](core_dump.md)

[プログラム動作中に内部状態を調査したい](inspect_prgoram_on_running.md)

[Coreダンプ調査（準備）](core_dump_1.md)

[Coreダンプ調査（基本）](core_dump_2.md)

[Coreダンプ調査（デッドロック）](core_dump_3.md)

[Coreダンプ調査（コマンド自動化）](core_dump_4.md)

[動的解析：不正メモリアクセス](dynamic_code_analysis.md)

[大量の仮想メモリを使っているが大丈夫か](huge_vss.md)

## ネットワーク

[必要なネットワーク系のコマンドは](important_network_command.md)

[一時的にIPアドレスの付与・削除をしたい](temporary_ipaddress.md)

[PING: 指定のIPアドレスと通信可能か確認したい(IP)](ping.md)

[ARPING: 指定のIPアドレスと通信可能か確認したい(Ethner)](arping.md)

[NETSTAT: 使っているTCP・UDPの状態を知りたい](netstat.md)

[TCPDUMP: コンソールからパケットキャプチャをしたい](tcpdump.md)

[受信ポートをエフェメラルポートの範囲内にしない](ephemeral_port.md)

[TCPポートがbindできないことがある](bind_failure.md)

[UDPではSO_REUSEADDRを使わない](udp_reuseaddr.md)

[TCPを閉じる場合はshutdown()を使う](tcp_shutdown.md)

[TCPでネットワークの断絶を想定する](network_problem.md)

[品質の悪いネットワークを模擬したい](tc_netem.md)

## ログ解析

[ログがリアルタイムで表示されない](stdio_buffering.md)

[ログ解析のためにフィルタしたい](log_filtering.md)

## その他

[ファイルサイズ０に気を付ける](filesize_zero.md)

