# INTER-MediatorをDockerで動かす

はじめに
=====

### Dockerとは

> Docker（ドッカー[2]）はソフトウェアコンテナ内のアプリケーションのデプロイメントを自動化するオープンソースソフトウェアである。
> [Wikipediaより](https://ja.wikipedia.org/wiki/Docker)

Dockerを使うことにより、例えば同じ設定内容のWebサーバーを素早く追加して並列化したり、
プロダクション環境と同じWebサーバーを開発環境に用意することが迅速にできます。もちろんWebサーバーに限らずDBサーバーにも使うことが出来ます。

ここではそんなDockerをINTER-Mediatorを動かす環境としてセットアップしてみます。


Dockerのインストール
=====

### Dockerのインストール

Dockerはお手元のOSによってインストールの方法が別々にあります。

#### macOSにDockerをセットアップする場合

macOSの場合、Docker Toolboxを入れる方法とDocker for Macを入れる方法があります。
既にVirtualBoxを使用している場合は[Docker Toolbox](https://www.docker.com/products/docker-toolbox)をダウンロードしてインストールします。

仮想化ソフトウェアを一度もインストールしたことがないのでしたらVirtualBoxをインストールの上、Docker Toolboxを入れてもいいですし、[Docker for Mac](https://docs.docker.com/docker-for-mac/)を入れても構いません。
Docker for Mac は、macOSのxhyveを使っています。よく分からない場合やxhyveが信用ならない場合はDocker Toolboxでのセットアップを採用してください。

ここではDockerの詳しい説明は行いません。

#### WindowsにDockerをセットアップする場合

Windows7(64bit)以降であればWindowsでもDockerを動かすことが出来ます。
[Docker Toolbox](https://www.docker.com/products/docker-toolbox)のwindows版をインストールします。
Windows10 Proに限ってはHyper-Vを利用する[Docker for Windows](https://docs.docker.com/docker-for-windows/)のインストールが簡単です。
※Docker for Windowsは2016/07/05時点ではWindows 10 Pro、Enterprise、Educationのみ対応ですが将来的には他のWindows10もサポートする予定だそうです。
※Windows用のインストールでウィザード内でKitematicのインストールチェックボックスは外さないようにしてください。後で使います。

#### KitematicでDockerを試す

Dockerの便利ツールの中にKitematicというものがあります。Dockerのお手軽さを体験するために、Kitematicを使ってみましょう。
KitematicはDockerをGUIで操作するツールです。Dockerのインストール時に一緒に入ってるはずです。
Kitematicを起動すると真ん中にDockerHubで公開されているDockerイメージのお薦めなどが表示されています。
ここでは検索して任意のDockerイメージを探すことも出来ます。
虫眼鏡マークの検索欄に
```
sakadonohito/im



IMを動かすDockerの作成
=====

さて、Dockerをセットアップできました。次にDocker上で動かすINTER-Mediatorを準備しましょう。
ここでは例としてgit cloneしたINTER-Mediatorをそのまま使います。
もしも既にカスタマイズしたINTER-Mediatorで作成したWebアプリがあるようでしたらそれを使って構いません。

Dockerhubの登録手順
=====

Docker

Docker pull と Kitematicの操作
=====
