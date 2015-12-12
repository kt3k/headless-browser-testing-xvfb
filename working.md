This is a Japanese translation of [Headless Browser Testing With Xvfb](http://tobyho.com/2015/01/09/headless-browser-testing-xvfb/) by [Toby Ho](http://tobyho.com/).

[Toby Ho](http://tobyho.com/) は JavaScript のテストフレームワーク [testem](https://github.com/testem/testem) の作者。Google が開発する [Karma](https://github.com/karma-runner/karma) フレームワークと並んで、実ブラウザ上でテストケースを走らせられるツールとして非常に人気です。

# Xvfb を利用したヘッドレスブラウザテスト

by [Toby Ho](https://github.com/airportyh), 2015/01/09

いまどき "ヘッドレスブラウザ" と言うと、PhantomJS がすぐに思い浮かぶかもしれませんが、実は他にも選択肢があります。この記事では Linux で使える Firefox, Chrome などの実ブラウザのヘッドレスな実行の仕方の紹介をします。(そのために Xvfb というツールを使います)

These days, when the phrase "headless browser" is mentioned, you immediately think of PhantomJS, but - there are alternatives. In this article, I am going to introduce a nice alternative you can use on Linux which allows you to run real browsers - Firefox and Chrome, headless - using Xvfb.

## PhantomJS の何が問題か？

What's Wrong With PhantomJS?

PhantomJS は素晴らしいツールです。世界中の企業 / エンジニアに使われています。特に、JavaScript のテストを走らせるためによく採用されています。しかし、PhantomJS に欠点がないわけではありません。PhantomJS は Chrome や Safari と同じ WebKit ベースで作られているものの、微妙に違う点もいくつか存在します。そのような違いは、テスト失敗の誤検知 (本当は成功しているテストが、PhantomJS 固有の問題で失敗すること) であったり、テストケースのパフォーマンス劣化を招いたりします。実際のエンドユーザーがいないという意味で、PhantomJS は実際のブラウザではないため、PhantomJS 固有の問題を解決することは、実際のプロダクトを改善することとは別な、テストケースを維持するためのコストとなります。ただでさえ、きちんとテストを書き続けることは難しいことですが、そのメンテナンスが、テスト失敗の誤検知などでさらに面倒になってしまうと、開発者のモチベーションに悪影響です。

PhantomJS is great tool and is widely used by companies and developers around the world. It has particularly been widely adopted for running JavaScript test suites. However, using PhantomJS isn't without drawbacks. Although it is built on top on WebKit - the same rendering engine both Safari and Chrome use, it still behaves in subtly different ways from those browsers, which can cause false positives (tests failing when there is not a real defect) and in some cases performance degradation of your test suite. Since PhantomJS isn't a real browser in the sense that no end-user actually use it, fixing these issues specifically for PhantomJS becomes an upkeep cost of the test suite, rather than something that directly improves the product. Getting developers to write tests consistently is hard enough, but if maintaining the tests becomes annoying - which false positives definitely are - it can become demoralizing.

## 実ブラウザをヘッドレスに実行する

Run Real Browsers Headless

以上のような理由から、私はある時から JavaScript のテストを走らせるために、実ブラウザだけを使うことに決めました。このことは、開発中にテストを走らせるための別ブラウザ窓をいつも開いている (実際はバックグラウンドに隠しているかもしれませんが) ということを意味します。私にとってはこれで OK  です。しかし、多くの場合 CI サーバは Linux です。そしてディスプレイを持っていません。そこで Xvfb が出てきます。

Because of the above reasons, at one point in my career I decided to use real browsers only to run JavaScript test suites. This has meant opening up a separate browser window to run the tests during development which I would simply hide in the background. I am okay with this. However, in many cases the continuous integration servers we use are Linux servers, and do not have displays. This is where Xvfb comes in.

Xvfb というのは X (Linux 用の GUIシステム) のための仮想ディスプレイフレームバッファというものです。それは、GUI アプリが書き込むための偽のディスプレイバッファを提供します。つまりは、すべてのプログラムをヘッドレス (GUI 無し) に実行することができます。

Xvfb is a virtual display framebuffer for X - the display system used by Linux. It provides a fake display buffer for graphical programs to write to, thus allowing any program to run headlessly.

### Xvfb のインストール
Installing Xvbf

Debian か Ubuntu 上で apt-get を使っている場合はインストールはシンプルです

If you are running Debian or Ubuntu and are using apt-get, installing is simply as:

    apt-get install xvfb

Redhat 系 OS で yum を使う場合

If you are on CentOS and using yum, it's

    yum install xorg-X11-server-Xvfb


### ブラウザのインストール

Installing the Browsers

Ubuntu 上で Chrome と Firefox をインストールするのは簡単でした。

On Ubuntu, I found installing Chrome and Firefox painless.

Chrome に関しては単純にダウンロードページに行って `.deb` ファイルを落として、`dpkg -i <the path to the .deb file>` コマンドを実行してください。

For Chrome simply go to the Chrome download page, download the .deb file, then do dpkg -i <the path to the .deb file> to install.

Firefox については、`api-get install firefox` でインストールできます。
For Firefox, it's just apt-get install firefox.

CentOS の場合、いろいろ問題が見つかりました。
On CentOS, I have had more trouble.

まずそもそも、Chrome は CentOS ではサポートされていません。Chromium をビルドするためのシンプルな方法も見つけることができなかったため、私は諦めました。
For starters, Chrome is not supported on CentOS and I haven't even found a simply way to build Chrominum and I gave up.

Firefox についても、少し問題がありました。まず `yum install firefox` をしたところエラーで起動できず、gdk-pixbuf2 を `yum install gdk-pixbuf2` でインストールする必要がありました。
Installing firefox also had a little hiccup: I ran yum install firefox, but then running firefox resulted in an error, which I solved by also installing gdk-pixbuf2 via yum install gdk-pixbuf2.

その他の Linux に関しては、Chrome の Download ページと Firefox on Linux ページを参照してください。
If you are on other Linux distros I haven't mentioned, checkout the Chrome download page and the Firefox on Linux page.

### Xvfb の使い方
Xvfb Usage

Xvfb の使い方は:
The usage of Xvfb is:

    xvfb-run <some command>

以上だけです！例えば、次のコマンドで Firefox を Xvfb で使えます:
That's all you need to know! For example, you can run Firefox within Xvfb:

    xvfb-run firefox http://google.com

上のコマンドは、単純に Firefox をバックグラウンドで起動します。よく分からない警告がコンソール上に流れるかもしれませんが、それ以外は特に何も見えないはずです。
This should just start a firefox process in the background. You may see a cryptic warning in the terminal, but otherwise it's not very exciting because you don't actually see anything.

Testem を使っている場合:
If you use Testem to run tests, you can do:

    xvfb-run testem -l firefox

で、Firefox 上で test が走り、
to run your tests in Firefox or

    xvfb-run testem -l chrome

で、Chrome のテストが走ります、ヘッドレスに！
to run them in Chrome - headless!

もちろん、これは Testem に限ったことではありません。Karma, Grunt, Gulp や実ブラウザを起動する他のどんなテストランナーでも xvfb-run で実行することができ、単純にうまくいきます。
Of course, this is not limited to Testem, you can use xvfb-run with Karma, Grunt, Gulp or any other test runner that spawns a real browser and it will just work.


## 結論

Implications

もし開発に Linux を使っている場合、これは素敵なソリューションです。これを使って、余計なウインドウを開くことなくテストが実行できます。セットアップも簡単で、使うのも簡単です。しかし、開発に Linux を使っていないにしても、CI では Linux を大抵使っているでしょうし、その場合、今の所 PhantomJS を使ってテストを走らせてるかもしれません。もしそれがうまくいっているなら、それはそれで素晴らしいですが、もし PhantomJS が余計な開発のオーバヘッドを生みだしつつあるようなら、このセットアップにスイッチすることを検討してみてはどうでしょうか。

If you use Linux for development, this is an attractive solution. It allows you to run your tests without having an unwanted window and it's easy to setup and use. However, even if you don't use Linux for development, you may still use Linux for continuous integration, in which case, maybe you are currently using PhantomJS to run tests. If that's working well for you, great. But if PhantomJS is starting to create unwanted overhead for you, consider switching to this setup.