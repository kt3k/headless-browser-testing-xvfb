This is a Japanese translation of [Headless Browser Testing With Xvfb](http://tobyho.com/2015/01/09/headless-browser-testing-xvfb/) by [Toby Ho](http://tobyho.com/).

[Toby Ho](http://tobyho.com/) は JavaScript のテストフレームワーク [testem](https://github.com/testem/testem) の作者。Google が開発する [Karma](https://github.com/karma-runner/karma) フレームワークと並んで、実ブラウザ上でテストケースを走らせられるツールとして人気があります。

# Xvfb を利用したヘッドレスブラウザテスト

by [Toby Ho](https://github.com/airportyh), 2015/01/09

いまどき "ヘッドレスブラウザ" と言うと、PhantomJS がすぐに思い浮かぶかもしれませんが、実は他にも選択肢があります。この記事では Linux で使える Firefox, Chrome などの実ブラウザのヘッドレスな (GUI 無しでの) 実行の仕方の紹介をします。(そのために Xvfb というツールを使います)

## PhantomJS の何が問題か？

PhantomJS は素晴らしいツールです。世界中の企業 / エンジニアに使われています。特に、JavaScript のテストを走らせるためによく採用されています。しかし、PhantomJS に欠点がないわけではありません。PhantomJS は Chrome や Safari と同じ WebKit ベースで作られているものの、微妙に違う点もいくつか存在します。そのような違いは、テスト失敗の誤検知 (本当は成功しているが PhantomJS 固有の問題でテストが失敗すること) であったり、テストケースのパフォーマンス劣化を招いたりします。実際のエンドユーザーがいないという意味で、PhantomJS は実際のブラウザではないため、PhantomJS 固有の問題を解決することは、実際のプロダクトを改善することとは別な、テストケースを維持するためのコストとなります。ただでさえ、きちんとテストを書き続けることは難しいことですが、そのメンテナンスがテスト失敗の誤検知などでさらに面倒になってしまうと、開発者のモチベーションに悪影響です。

## 実ブラウザをヘッドレスに実行する

以上のような理由から、私はある時から JavaScript のテストを走らせるために、実ブラウザだけを使うことに決めました。このことは、開発中にテストを走らせるための別ブラウザ窓をいつも開いている (実際はバックグラウンドに隠しているかもしれませんが) ということを意味します。私にとってはこれで OK  です。しかし、多くの場合 CI サーバは Linux です。そしてディスプレイを持っていません。そこで Xvfb が出てきます。

Xvfb というのは X (Linux 用の GUIシステム) のための仮想ディスプレイフレームバッファというもので、それは、GUI アプリが書き込むための偽のディスプレイバッファを提供します。つまりは、すべてのプログラムをヘッドレス (GUI 無し) に実行することができます。

### Xvfb のインストール

Debian か Ubuntu 上で apt-get を使っている場合はインストールはシンプルです

    apt-get install xvfb

Redhat 系 OS で yum を使う場合

    yum install xorg-X11-server-Xvfb

### ブラウザのインストール

Ubuntu 上で Chrome と Firefox をインストールするのは簡単でした。

- Chrome に関しては単純にダウンロードページに行って `.deb` ファイルを落として、`dpkg -i <the path to the .deb file>` コマンドを実行してください。
- Firefox については、`api-get install firefox` でインストールできます。

CentOS の場合、いろいろ問題が見つかりました。

- まずそもそも、Chrome は CentOS ではサポートされていません。Chromium をビルドするためのシンプルな方法も見つけることができなかったため、私は諦めました。
- Firefox についても、少し問題がありました。まず `yum install firefox` をしたところエラーで起動できず、gdk-pixbuf2 を `yum install gdk-pixbuf2` でインストールする必要がありました。

その他の Linux に関しては、Chrome の Download ページと Firefox on Linux ページを参照してください。

### Xvfb の使い方

Xvfb の使い方は:

    xvfb-run <コマンド>

以上だけです！例えば、次のコマンドで Firefox を Xvfb で使えます:

    xvfb-run firefox http://google.com

上のコマンドは、単純に Firefox をバックグラウンドで起動します。よく分からない警告がコンソール上に流れるかもしれませんが、それ以外は特に何も見えないはずです。

Testem を使っているなら:

    xvfb-run testem -l firefox

で、Firefox 上で test が走り、

    xvfb-run testem -l chrome

で、Chrome のテストが走ります、ヘッドレスに！

もちろん、これは Testem に限ったことではありません。Karma、Grunt、Gulp や実ブラウザを起動する他のどんなテストランナーでも xvfb-run で実行することができ、単純にうまくいきます。

## 結論

もし開発に Linux を使っている場合、これは素敵なソリューションです。これを使って、余計なウインドウを開くことなくテストが実行できます。セットアップも簡単で、使うのも簡単です。しかし、開発に Linux を使っていないにしても、CI では大抵 Linux を使っているでしょうし、その場合、今の所 PhantomJS を使ってテストを走らせているかもしれません。もしそれがうまくいっているなら、それはそれで素晴らしいですが、もし PhantomJS が余計な開発のオーバヘッドを生みだしつつあるようなら、このセットアップにスイッチすることを検討してみてはどうでしょうか。