# Coreダンプ調査（コマンド自動化）

## 動機
Coreダンプを調査していて、解析のためある変数の内容を表示させたいが、手動で入力するには現実的でない量である。

## 回答
PythonスクリプトでGDBを制御できるのでこれを利用する。

gdb上で「source <pythonソースファイル名>」で実行することができる。
通常のpythonスクリプトとして実行するが、gdb.execute(<gdbコマンド>)を用いることでgdbへコマンドを送ることができる。

## 実行例（簡単な例）
以下の例では、

    print('Exeucte [bt] command.')
    gdb.execute('bt')

と書かれたPythonスクリプトを呼び出している。ここでは単にバックトレースを表示する。

    (gdb) source test1.py
    Execute [bt] command.
    #0  0x00007f2a02af67f0 in __read_nocancel () from /lib64/libc.so.6
    #1  0x00007f2a02a8cbc8 in _IO_new_file_underflow () from /lib64/libc.so.6
    #2  0x00007f2a02a8e6ce in _IO_default_uflow_internal () from /lib64/libc.so.6
    #3  0x00007f2a02a89d0c in getchar () from /lib64/libc.so.6
    #4  0x0000000000400707 in main (argc=1, argv=0x7ffe09269678) at testprogram9.c:25

## 実行例（gdbの出力結果の取り込み）
「gdb.exexute(<gdbコマンド>, to_string=True)」で、gdbコマンドで出力した文字列を取り込むことができる。
以下の例では、

    s = gdb.execute('x/2gx list', to_string=True)
    s = s.split()
    print('hex1={0} hex2={1}'.format(s[1], s[2]))

と書かれたPythonスクリプトを呼び出している。
list変数に対するメモリダンプをgdbで表示し、値を取り込んで編集して表示している。
	
    (gdb) x/2gx list
    0x1a1e1f0:      0x0000000001a1e1d0      0x0000000071c31196
    (gdb) source test2.py
    hex1=0x0000000001a1e1d0 hex2=0x0000000071c31196

## 実行例（高度な例：ポインタ連結されたリストの内容の表示）
例えば、ポインタで連結したリンクドリストのようなデータ構造の内容をgdbから確認したい場合は非常に煩雑になる。データの内容を見るのに「x」コマンドを打ち、それで次のポインタのアドレスが分かったら、そのアドレスに対してまた「x」コマンドを入力し・・、１０個先のデータを見るのにも数分かかるだろう。
こういった際には、Pythonスクリプトを使う必要がある。

例えば、以下のような構造体によるリンクドリストで作られたlistという名前の変数があるとする：

    struct list_t {
        struct list_t* next; // 次のリストのポインタ。データがない場合はNULLとする
        uint64_t data;       // データ
    };
    struct list_t* list = NULL;
    ・・・
    list = append_list(list, data);   // list にデータを追加する
　
このデータ構造に対して、逐一コマンドを打ってdataフィールドの内容を表示しようと考えたならば、例えば以下のようにコマンドを入力していくことになるだろう（「x/2gx」コマンドで、引数のアドレスを、２つ・８バイト区切りで・１６進数で表示する。ここでは、１番目のデータがnext、２つめのデータがdataになる）：

    (gdb) x/2gx list
    0x1c6f1f0:      0x0000000001c6f1d0      0x0000000071c31196
    (gdb) x/2gx 0x0000000001c6f1d0
    0x1c6f1d0:      0x0000000001c6f1b0      0x000000002fc54c6b
    (gdb) x/2gx 0x0000000001c6f1b0
    0x1c6f1b0:      0x0000000001c6f190      0x0000000027cfa7a4
    ・・・

これを、Pythonスクリプトで自動化する。この場合、以下のようなスクリプトを準備する：

    addr = "list"
    while addr != "0x0000000000000000":
        command = 'x/2gx {0}'.format(addr)
        r = gdb.execute(command, to_string=True).strip().split('\t')
        print(r[2])
        addr = r[1]
　
このスクリプトでは、「x/2gx」を実行しその出力文字列を保存する。
２番目のデータを表示し、１番目のデータを次のアドレスとして「x/2gx」をさらに実行する動作をNULLに到達するまで次々とおこなう。

	(gdb) source test3.py
	0x0000000071c31196
	0x000000002fc54c6b
	0x0000000027cfa7a4
	0x000000006a017395
	0x000000006ec338b5
	0x000000005b8ca77d
	0x0000000032a11777
	0x0000000050874f62
	0x0000000011af4cb1
	0x00000000580ef4f2
	・・・
	
必要ならば、「set logging on」コマンドと組み合わせてファイルに保存してもいいかもしれない。

### 参考文書
gdbからPythonを呼び出した場合に用いることのできるAPIの詳細
- [Debugging with GDB - 23.2.2 Python API](http://www.geocities.jp/harddiskdive/gdb/gdb_264.html#Python-API)
