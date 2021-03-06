# 綺麗なコード

## 動機
綺麗なコードとは何か。綺麗に書くにはどうしたらよいか。

## 回答
私は、次の３つを満たすコードが綺麗であると考える：
- 論理的な構造を持つ
- 各構成要素が適切な分量
- 名称が一貫していて・明瞭・簡潔

綺麗に書くためには、これらを意識して記述すればよい。コーディングにおける心得や規約はこれらを支える具体的な実現方法となる。

「論理的な構造をもつ」とは、モジュール（クラス・関数など）の分け方・構成が適切であることをいう。日本語でいえば、章立てや段落のつけ方が適切で読みやすいことを指す。
「各構成要素が適切な分量」とは、モジュールが長すぎないことをいう。
「選択した言葉が一貫していて・明瞭・簡潔」とは、変数名や関数名などが全体としてブレがなく、また分かりやすいことをいう。

コードの綺麗かどうかに関する良し悪しは、読みやすさ・理解しやすさと同義と考えてよい。
読みやすさ・理解しやすさの観点でいえば、自然言語、たとえば日本語とコードは同じであるとみなせる。
ゆえに文章をどのように綺麗に書くかの考え方は、ほとんどそのまま綺麗なコードを書くことに転用できる。
上記の３つの条件は、読みやすく・理解しやすい文章を書く条件でもある。

### なぜ綺麗に書く必要があるか
綺麗に書かれていないコードは、読み手を意識しておらず読みづらく・理解しづらい。
そのため：
- 記述時は、自身によるミスに気が付かずバグが混入しやすい
- レビューの際は、他人の時間をより多く使い、かつバグの検出がしづらい
- デバッグ時は、読みづらいためバグの箇所の特定が難しく、かつ修正しづらい

また、
- 不必要に複雑である可能性が高い（作られた複雑さ）

よって、コードは綺麗に書く必要がある。

### 論理的な構造を持つ
コードは論理的な構造を持たねばならない。論理的な構造とは「グルーピングが適切であり、またそれぞれのグループには目的が１つある。さらにこの目的を支えるための構成要素は過不足なく、かつそれぞれの抽象的なレベルはかけ離れていない」ことをいう。

コードの上での論理的構造でいえば：
- モジュールの分け方が適切である
- モジュールの目的は１つ  
（目的が１つでないと、目的がブレてはっきりしない。よって２つ以上の目的は持つべきでない。その場合はこのモジュールの上位にさらにモジュールがあるべきだろう）
- そのモジュールの記述に無関係ないことが混じっていないこと
- そのモジュールの記述に細かすぎることが書いていないこと  
（それは別の子モジュールになっているべきだろう）

ことをいう。

例えば関数であれば、以下のようになるだろう：
- 関数の分け方が適切で、
- 関数呼ぶことによって発生することがハッキリしていて、
- 関数を構成するコードは過不足なく、
（目的達成のためのコードはすべて書かれていて[少なすぎず]、かつ関係ないことをしていない[多すぎない]）
- 細かすぎることが書かれていない
（その関数の目的にとって細かすぎる内容は別の関数として切り離す）

### 各構成要素が適切な分量
ある構造を構成する要素は適切な分量でなければならない。
適切な分量とは、人が一度に理解ができる分量のことを指す。
例えば数１００行あるような関数は、人が一度に理解できる分量をはるかに超えている。

### 名称が一貫していて・明瞭・簡潔
モジュールや変数などの名称が、プログラム全体（可能ならもっと広い範囲）でブレがなく、
一見してそれが何を示しているのかが分かるようでなければならない。
　
- 一貫した単語  
例えば、「艦艇」を示す英単語はfleet・vessel・shipなどで表現できるが、プログラム内で同じ意味でこれらが複数種類を無秩序に使われていたら混乱することは明らかである。
- 明瞭  
例えば、「メッセージ送信に成功した」ことを記録する変数があるとして、これが「flag」だった場合は明瞭でない（何かのフラグである以上の情報がない。さらにどんな値なら成功？そもそも送信したことを示している？）。「isSuccessSendMessage」なら、ある程度は変数の目的もはっきりしているだろう。
- 簡潔  
例えば、上記の変数名「isSuccessSendMessage」について、ほんの２～３行のみで有効でならば「isSuccess」だけで意味は通じるし端的であるのでそれでもいいだろう。言葉が短く端的である方が認識が早いので場合に応じて短くする（もちろん、明瞭さを失われない範囲で）。  
もちろん逆に有効範囲が広いならば、例えばグローバル変数であったら変数名「isSuccessSendMessage」は簡潔すぎる可能性がある（例えば、何のメッセージか、どこへのメッセージなのかの情報が抜けている）。 
なお略語についても、全体として一貫していれば使っても問題ない。Messageという単語も、Msgで表せるかもしれない。略語は常に使われていたり、あるいは慣習として一般的であれば問題ないだろう。  
例えば、WikipediaでTCPを検索すると、TCP/IPのTCPであったり、そういう名称のテレビ制作会社での略称であったりと、複数の意味を取りうるらしい。しかし、LANで通信するプログラム内で「TCP」と書いてあったら、普通、TCP/IPのTCPだと認識するだろう。不明瞭だからと、ここで「TransmissionControlProtocolInInternetProtocolSuite」と記述するのはやりすぎである。

## 参考文書

論理的に書くための考え方
- バーバラ・ミント（1999）『新版　考える技術・書く技術　問題解決力を伸ばすピラミッド原則』ダイヤモンド社
- 木下是雄（1981）『理科系の作文技術』中央新書

具体的にコードにどのように書くか、名称に関する改善など
- ダスティン・ボズウェル、トレバー・フォシェ（2012）『リーダブルコード　より良いコードを書くためのシンプルで実践的なテクニック』
