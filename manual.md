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

まずはローカルに Docker が実行できる環境を作りましょう。以下のドキュメントを参考にしてください。Mac では Docker Toolbox をインストールする方法と Docker for Mac をインストールする方法の2通りがあります。Windows では Docker Toolbox をインストールする方法と Docker for Windows をインストールする方法の2通りがあります。

OSのバージョンによってはインストールの選択肢がDocker Toolboxのみとなります。

#### macOSにDockerをセットアップする場合

macOSの場合、Docker Toolboxを入れる方法とDocker for Macを入れる方法があります。
既にVirtualBoxを使用している場合は[Docker Toolbox](https://www.docker.com/products/docker-toolbox)をダウンロードしてインストールします(ダウンロードに結構時間がかかるかもしれません...)。

仮想化ソフトウェアを一度もインストールしたことがないのでしたらDocker Toolboxを入れてもいいですし、[Docker for Mac](https://docs.docker.com/docker-for-mac/)を入れても構いません。※Docker Toolboxのインストーラの中でVirtualboxのインストールが行われます。
Docker for Mac は、macOSのxhyveを使っています。よく分からない場合やxhyveが信用ならない場合はDocker Toolboxでのセットアップを採用してください。

ここではDockerの詳しい説明は行いません。

#### WindowsにDockerをセットアップする場合

Windows7(64bit)以降であればWindowsでもDockerを動かすことが出来ます。
[Docker Toolbox](https://www.docker.com/products/docker-toolbox)のwindows版をインストールします(ダウンロードに結構時間がかかるかもしれません...)。※Docker Toolboxのインストーラの中でVirtualboxのインストールが行われます。
Windows10 Proに限ってはHyper-Vを利用する[Docker for Windows](https://docs.docker.com/docker-for-windows/)のインストールが簡単です。
※Docker for Windowsは2016/07/05時点ではWindows 10 Pro、Enterprise、Educationのみ対応ですが将来的には他のWindows10もサポートする予定だそうです。
※Docker for WindowsではHyper-Vを使います。お使いのPCの構成によってはHyper-Vの構成と再起動が入ります。
※Windows用のインストールでウィザード内でKitematicのインストールチェックボックスは外さないようにしてください。後で使います。

#### KitematicでDockerを試す

Dockerの便利ツールの中にKitematicというものがあります。Dockerのお手軽さを体験するために、Kitematicを使ってみましょう。
KitematicはDockerをGUIで操作するツールです。Dockerのインストール時に一緒に入ってるはずです。
※Docker for mac の場合は別途ダウンロードが必要です。ダウンロードしたものをアプリケーションフォルダに入れてください。
※WindowsでDocker Toolboxをインストールした場合でKitematicの最初の起動に失敗した場合、Hyper-Vを使おうとしている時があります、選択肢にVirtualboxを使うがありますので、そちらで再度起動を試してください。
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

Dockerコンテナの中身を分割する
=====

一つのDockerイメージに全てを詰め込むのはポータビリティが高いですが、本当はDockerの思想に反します。
一つのDOckerコンテナには一つの役割にするのが理想とされています。Webサーバー、DBサーバー、さらに分割するなら永続データと
コンテナを分けてみましょう。※PHPをApacheで動かす前提で説明しますので、ここではPHPとWebサーバー(Apache)は分けません。

### docker-composeを使う

上記のコンテナ分割を行うと、1つのアプリのために幾つものDockerコンテナをセットアップして起動しないといけなくなりちょっと煩雑になります。Dockerにはその煩雑さを解消するため、docker-composeというツールが有りますので、使ってみましょう。Dockerをインストールすると一緒についてきます。

細かい内容説明はドキュメントを参照してください。完成形をpullしましょう。
任意のディレクトリで以下のコマンドを実行し、githubからdocker-composeようの設定ファイルをcloneします。

```
git clone https://github.com/sakadonohito/im-compose.git 
```

cloneした内容に*docker-compose.yml*という設定ファイルがあります。ここにどういったDockerコンテナの起動に関する設定を連携させたいコンテナ分だけ記述します。YAMLファイルについての説明はここでは割愛します。
このdocker-compose.ymlで扱うMySQLとPHP(with Apache)のDocker起動設定を記載しています。また、データコンテナ用の設定も書いてあります。
実はこのcloneした内容には2つ足りないものが有りますので、別途用意してください、
- INTER-Mediatorで使うMySQLのサンプルデータを流し込むSQLファイル(sample_schema_mysql.sql)
  - INTER-Mediatorをダウンロードした際に内包しているsample_schema_mysql.txtファイルをsample_schema_mysql.sqlに拡張子変更してください。
  - sample_schema_mysql.sqlをmysql/tmpディレクトリに配置してください。
- INTER-Mediatorそのもの。
  - INTER-Mediatorをapache/webrootディレクトリに*im*という名前に変更して配置してください。
  
上記の追加作業を行ったら、以下のコマンドを実行してください。

```
docker-compose up -d
```

設定ファイルの内容に従って、イメージをpullしてデーモンで起動してくれます。
http://192.168.99.100/im/Samples にアクセスしてみてください。サンプルの画面が見えると思います。
今回の設定では、ローカルに配置したINTER-MediatorをDockerコンテナにマウントして使っていますので、ローカル上でINTER-Mediatorを編集すると、それが反映されます。htmlファイルを編集して画面を更新してみてください。

コンテナを停止して削除するには以下のコマンドを実行します。

```
docker-compose stop
docker-compose rm
```

データコンテナ用のDockerコンテナも削除するのかと聞いてきますが、ここでは「y」を選択してください。

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
- ArukasというDocker専用のPaaSサービス上で起動する。

他にも、開発環境やテスト環境として利用することも可能です。
手軽にINTER-Mediatorを動かす環境としてDockerを是非活用してください。
