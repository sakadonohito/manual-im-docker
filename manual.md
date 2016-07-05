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
```
と入力して検索してみましょう。INTER-Mediatorを動かすDockerのサンプルが出てきます。これをNewしましょう。
ローカルにDockerイメージがダウンロードされますのでそれを起動してみましょう。
起動したらブラウザでアクセスしてみましょう。
http://localhost/im/Samples
INTER-MediatorのサンプルTOPページが表示されましたでしょうか。

一体何がどうなったのでしょうか？仕組みを説明します。

1. INTGER-Mediatorが動く状態のPHP、MySQL、Apacheが組み込まれたDockerイメージがDockerHubにホストされていた。
1. 手元の環境にDockerやDockerのGUIツールKitematicをインストールした。
1. KitematicからDOckerHubにアクセスし、INTER-MediatorのDockerイメージをダウンロードした。
1. KitematicからダウンロードしたDockerイメージを起動した。
1. ブラウザから起動したDockerイメージ上のINTER-Mediatorにアクセスした。

という流れです。
なぜ、先述のURLでDocker上のINTER-Mediatorにアクセス出来たのかは、そうなるようにDocker上のApacheの設定を行っているからです。
お手軽ですね。

IMを動かすDockerイメージの作成
=====

前の項でKitematicを使って作成済みのDockerイメージを起動することが出来ました。
次に自作のINTER-MediatorアプリをDockerイメージ化するにはどうしたらよいでしょうか。
簡単に説明します。

#### Dockerイメージの設計書？Dockerfile

詳しくはDockerの公式を参照して欲しいのですが、`Dockerfile`という名前のテキストファイルに
1. ベースとなるLinuxディストリビューションやDockerイメージ名の指定
1. 作成者の名前
1. Dockerイメージにインストールしたいライブラリ等のインストールコマンド、事前に用意したファイルを取り込むコマンドなど

といった内容を記述します。

INTER-Mediator用のイメージを作りには
1. ベースとなるLinuxディストリビューションやDockerイメージ名の指定
1. 作成者の名前
1. PHPのインストールと設定
1. MySQLのインストールと設定
1. Apacheのインストールと設定
1. INTER-MediatorディレクトリをWeb公開用の場所に配置

のようになります。

終わったら記述したDockerfileからイメージを作成します。

Dockerfileのあるディレクトリで以下のようなコマンドでイメージを作成します。
```
docker build -t <name>/<app>:<tag>
```
nameはDockerHubの登録アカウント名、appは今回ならINTER-Mediatorと分かるような名称がいいでしょう。
tagはバージョン番号になります。

注意点としては、Dockerは仮想サーバーではありません。Dockerイメージを起動したからといって、自動起動のサービスが勝手に起動したりはしません。
Dockerイメージの起動時に渡すコマンドで目的のサービスを起動します。

作成したDockerイメージは例として以下の様なコマンドで起動できます。
```
docker run -d --name im -p 18880:80 hogehoge/im:1.0
```
例はDocker(以降 コンテナ)を「im」という名前でバックグラウンドプロセスで起動。ローカルの18880ポートをコンテナの80ポートにマッピング、Dockerイメージは「hogehoge/im:1.0」という意味になります。
正しくDockerイメージが作成できていればhttp://localhost:18880/ でコンテナ内で起動したApacheのルートが表示されます。

これでいつでも手元でINTER-Mediatorを起動できるようになりました。


DockerHubの登録手順
=====

手元でDockerのイメージが作成出来るようになって、ローカルで起動もできるようになりました。
このイメージを他の環境でも使いたい場合はどうしたらよいでしょうか？
DockerfileをGithubに置いておいて、都度DockerfileをpullしてDockerイメージをbuildして構いません。
ですが、作成したイメージをDockerHubに登録しておいて、使いたい環境でpullするのが使う時はお手軽です。

[Docker Hub](https://hub.docker.com/)にアクセスし、アカウントを持っていない人は登録します。
アカウントの登録ができていれば後は手元からDockerイメージをpushするだけです。
手元のターミナルで
```
docker login
```
してでまずはターミナルでDockerHubにログインします。次に
```
docker push hogehoge/im:1.0
```
のようにコマンドを入力します。hogehoge/im:1.0は前項の例の流用です。dockerイメージ名:tagという構文です。
※latestというtagでイメージを作成してpushした場合はDockerHubからpullする際にtagを省略してDockerイメージをpull出来るようになります。

さいごに
=====

INTER-MediatorのDockerイメージの作成についての説明は以上になります。
さてこうして作成したDockerイメージはどこで使えるのでしょうか？
以下の様な使い方ができます。

- 手元でデモ環境として起動する。
- 他の人にINTER-Mediatorを手軽に体験してもらうためにDockerHubに登録しておく
- 作成したDockerイメージを本番用Linuxサーバー上で起動して使う
- AWSのElasticBeanstalk上で起動する。
- AWSのElasticContainerService上で起動する。
- HerokuというPaaSサービス上で起動する。

他にも、開発環境やテスト環境として利用することも可能です。
手軽にINTER-Mediatorを動かす環境としてDockerを是非活用してください。
