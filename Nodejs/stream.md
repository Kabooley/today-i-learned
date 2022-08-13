# Stream

この記事でまとめこと。

Node.jsのstreamについて学習してまとめるよ。

Node.jsのstreamを使ってweb上のコンテンツをダウンロードするプログラムを作るよ。

Node.js --version: v14.17.6

後ほど、`【Node.js】で、streamはいつ終了したんだい？`というタイトルで記事を作る

## 参考

https://nodejs.org/docs/latest-v14.x/api/

https://nodejs.org/docs/latest-v14.x/api/stream.html

## playground

Ubuntuの仮想環境にて。

```bash
~$ nodenv global 16.16.0
~$ cd nodejs/stream/
```

#### 寄り道：wslコマンド

まぁここを見ろという話ですが。

https://docs.microsoft.com/ja-jp/windows/wsl/basic-commands

```bash

# インストールされている Linux ディストリビューションを一覧表示する
wsl --list --verbose
# wslの状態を確認する
wsl --status
```

#### 寄り道：wsl2+UbuntuのUbuntuを初期状態に戻す

https://docs.microsoft.com/ja-jp/windows/wsl/install

https://docs.microsoft.com/ja-jp/windows/wsl/setup/environment

https://docs.microsoft.com/ja-jp/windows/wsl/basic-commands

https://qiita.com/PoodleMaster/items/b54db3608c4d343d27c4

- windows 10 の「設定」、「アプリ」、「アプリと機能」

- 検索窓から`Ubuntu`で初期化したいディストリビューションを選択

- 「詳細オプション」を選択

- 「リセット」ボタンを押す

...でアンインストールしないでまるっと初期状態にしてくれる。

```bash
# powershell
# リセットする前
$ wsl --list --verbose
  NAME                   STATE           VERSION
* Ubuntu-18.04           Stopped         2
  docker-desktop-data    Stopped         2
  docker-desktop         Stopped         2
  Ubuntu-20.04           Stopped         2
#   リセットした後
$ wsl --list --verbose
  NAME                   STATE           VERSION
* Ubuntu-18.04           Stopped         2
  docker-desktop-data    Stopped         2
  docker-desktop         Stopped         2
```

一旦リセットするとwslの利用可能ディストリビューションとして認識されていないのがわかる

Ubuntuをwindow 10のツールバー検索窓に入力して起動させる。

するとUbuntuが起動して初期処理が始まる。

rootユーザを登録。

以上でリセット終了。

```bash
# powershell
# 
# 起動させたら、wslでディストリビューションとして認識されているのがわかる
$ wsl --list --verbose
  NAME                   STATE           VERSION
* Ubuntu-18.04           Stopped         2
  docker-desktop-data    Stopped         2
  Ubuntu-20.04           Running         2
  docker-desktop         Stopped         2
```

初期化処理が終わったら以下の公式ベストプラクティスを設定するといいかも。

https://docs.microsoft.com/ja-jp/windows/wsl/setup/environment

```bash
$ sudo apt update && sudo apt upgrade
```


#### 寄り道：wsl2 + Ubuntu環境でスナップショットを取る

毎度何かあったら全部初めからやり直しはスッゴクタイヘンなので

ある程度ベースとなる環境を整え終えたらスナップショットを取ると安心＆便利。



#### 寄り道：wsl2+Ubuntu+anyenv環境の構築

手順：

- homebrewをインストールする
- anyenvをインストールする

NOTE: env系で環境開発するときは他のenv系がない状態で環境構築すべき

NOTE: env系もとか何もインストールされていない前提で話を進める

参考：

https://docs.brew.sh/Homebrew-on-Linux

https://qiita.com/amenoyoya/items/ca9210593395dbfc8531#docker-%E7%92%B0%E5%A2%83%E6%A7%8B%E7%AF%89

1. Homebrewをインストールする

Qiitaの記事のほうではLinuxbrewと呼ばれているけど

公式を確認したら、homebrewをインストールすればいいみたい

前準備：

https://docs.brew.sh/Homebrew-on-Linux#requirements

ビルド関係で必要な奴らを予めインストールしておく

```bash
# ビルド関係
sudo apt-get install build-essential procps curl file git

# とにかくいろいろする前はアップデートです
$ sudo apt update && sudo apt upgrade -y

# 公式のコマンドをそのまま実行すればいいみたい
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

インストール完了したら

```bash
Warning: /home/linuxbrew/.linuxbrew/bin is not in your PATH.
  Instructions on how to configure your shell for Homebrew
  can be found in the 'Next steps' section below.
==> Installation successful!

==> Homebrew has enabled anonymous aggregate formulae and cask analytics.
Read the analytics documentation (and how to opt-out) here:
  https://docs.brew.sh/Analytics
No analytics data has been sent yet (nor will any be during this install run).

==> Homebrew is run entirely by unpaid volunteers. Please consider donating:
  https://github.com/Homebrew/brew#donations

==> Next steps:
- Run these two commands in your terminal to add Homebrew to your PATH:
    echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> /home/username/.profile
    eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"
- Install Homebrew's dependencies if you have sudo access:
    sudo apt-get install build-essential
  For more information, see:
    https://docs.brew.sh/Homebrew-on-Linux
- We recommend that you install GCC:
    brew install gcc
- Run brew help to get started
- Further documentation:
    https://docs.brew.sh
```

という表示が出る。

とにかくPATHにhomebrewを登録しようという話である。

PATHに登録すればbrewコマンドをいつでも呼び出せるよねってやつ。

ご丁寧にコマンドを示してくれているのでその通りにやればいいんだと思うよ。

公式でもPATHに追加する方法を公開している。

https://docs.brew.sh/Homebrew-on-Linux

> Follow the Next steps instructions to add Homebrew to your PATH and to your bash shell profile script, either ~/.profile on Debian/Ubuntu or ~/.bash_profile on CentOS/Fedora/Red Hat.

ということで、

Ubuntu系は.profile/のscriptに、とのこと。

PATHを通す：

```bash
# こっちのコマンドはredhat系でもdebian系でもどっちでも行ける
test -d ~/.linuxbrew && eval "$(~/.linuxbrew/bin/brew shellenv)"
test -d /home/linuxbrew/.linuxbrew && eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"
test -r ~/.bash_profile && echo "eval \"\$($(brew --prefix)/bin/brew shellenv)\"" >> ~/.bash_profile
echo "eval \"\$($(brew --prefix)/bin/brew shellenv)\"" >> ~/.profile
# もしくはインストール後に示してくれたこっちを実行しろと
echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> /home/teddy/.profile
eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"
```

最後に

```bash
$ brew doctor
Your system is ready to brew.
```

でエラーがなければいいそうです

homebrewインストール・PATH登録完了

Qiitaの記事では、Linuxbrewをインストールしたら

> curl や git などは、最新版を使う方が良いため、改めて Linuxbrew で導入しなおす

ということで開発ツールを改めてインストールするといいそうです

```bash
$ brew install curl git wget gcc zlib libzip bzip2 readline openssl pkg-config autoconf
```

2. anyenvをインストールする

https://github.com/anyenv/anyenv

> env系開発環境をまとめて管理できるツール
> env系開発環境とは、pyenv, nodenv など、各プログラミング言語の複数バージョンを切り替えて使用可能とする環境のこと
> 独自に導入した env系開発環境がある場合は、それらを削除してから導入すること

だそうで。


```bash
$ brew install anyenv
$ anyenv install --init
## Do you want to checkout ? [y/N]: <= y

# anyenv 初期化スクリプトを .bashrc に記述
$ echo 'eval "$(anyenv init -)"' >> ~/.bashrc
$ source ~/.bashrc

# https://github.com/znz/anyenv-update
# 
# Plugin
mkdir -p $(anyenv root)/plugins
git clone https://github.com/znz/anyenv-update.git $(anyenv root)/plugins/anyenv-update
anyenv update
```

#### 寄り道：node.js開発環境構築

- nodenvの導入
- yarnの導入
- Node.jsの導入

1. nodenvの導入

https://github.com/nodenv/nodenv

公式のinstallationにはanyenvでのインストール方法は載っていない。

```bash
$ anyenv install nodenv
$ exec $SHELL -l
```

2. yarn導入

nodenv-yarn-installを導入する

https://github.com/pine/nodenv-yarn-install


```bash
$ mkdir -p "$(nodenv root)/plugins"
$ git clone https://github.com/pine/nodenv-yarn-install.git "$(nodenv root)/plugins/nodenv-yarn-install"
```

3. Node.jsの導入

```bash
# list all available versions:
$ nodenv install -l

# install a Node version:
$ nodenv install 0.10.26
```

公式で16.16.0が推奨版らしい

完了

#### 寄り道；WSL2でVSCodeを使えるようにする

https://docs.microsoft.com/ja-jp/windows/wsl/tutorials/wsl-vscode

Linuxディストリビューションから`code .`と入力するだけ


## 重要な概念、ポイント

- highWaterMark
- consumer
- drain
- streamはいつ終了するのか？

## stream

`stream`はNode.jsでストリーミングデータを扱うための抽象的なインタフェイスである。

`stream`には多くのストリームオブジェクトが存在する。


#### Types of streams

- `Writabel`: データを書き込むことができるストリーム
- `Readable`: データを読み取ることができるストリーム
- `Duplex`: `Readable`でも`Writable`でもあるストリーム
- `Transform`: データの書き込みおよび読み取り時にデータを変更または変換できるストリーム

#### Object mode

> Node.js APIによって作成されたすべてのストリームは、文字列とBuffer（またはUint8Array）オブジェクトでのみ動作します。
> ただし、ストリームの実装が他のタイプのJavaScript値で機能することは可能です（ストリーム内で特別な目的を果たすnullを除く）。
> このようなストリームは、「オブジェクトモード」で動作すると見なされます。

つまり、streamは通常文字列型とBufferオブジェクトに対して使えるものであるが、

streamをオブジェクトモードで運用するならば、JavaScriptにおけるnumber型など、

文字列型やBUffer型以外に対してもstreamを利用することができるという話らしい。

streamにはオブジェクトモードという特別な`mode`が存在する

このオブジェクトモードになるにはstreamインスタンスを生成するときに

`objectMode`オプションを使うことでなることができる。

#### Buffering

`Writable`, `Readable`両ストリームは内部的なバッファにデータを格納する。

バッファリングされる可能性のあるデータ量は、

ストリームのコンストラクタに渡された`highWaterMark`オプションに依存する。

`highWaterMark`オプションは、

通常のストリームなら総バイト量を指定する。

オブジェクトモードストリームなら総オブジェクト量を指定する。


実装が`stream.push(chunk)`を呼び出したときは、データはReadable`ストリームにバッファされる

ストリームの消費者が`stream.read()`を呼び出さなかった場合、内部的なQueueに消費されるときまで格納される。

ひとたび内部的な読み取りバッファが`highWaterMark`の指定する域にまで達したならば、

ストリームは一時的にデータの読み取りを停止する、現在のデータが消費されるまで。

継続的に`Wrtaibel.write(chunk)`が呼び出されたとき、データは`Writable`ストリームへ格納される。

内部の書き込みバッファの総量が`highWaterMark`の閾値を下回っている限り,`writable.write()`は`true`を返す。

バッファ総量が`highWaterMark`を超えたら`false`を返す

`stream`APIのゴールへのカギは、

とりわけ`stream.pipe()`メソッドのようなメソッドには

ストリーム元とストリーム先の速度の違いが、確保できているメモリを圧迫しないように

バッファデータ量を制限することにある。

highWaterMarkオプションはしきい値であり、制限ではありません。

ストリームが追加のデータの要求を停止する前にバッファリングするデータの量を指定します。一般に、厳密なメモリ制限は適用されません。特定のストリーム実装では、より厳しい制限を適用することを選択できますが、そうすることはオプションです。

NOTE: Duplexの説明項目は割愛。

内部バッファの構造は内部実装であり、またいつでも変更される可能性がある。

とはいえ、特定の高度な実装においては内部バッファは`writable.writableBuffer`, `readable.readableBuffer`を使うことで結びつけることができる
これらドキュメント公開されていないプロパティを使うことは非推奨である。

#### API for stream consumers

どんなに小さいアプリケーションでも、ほとんどすべてのNode.jsアプリはいくつかのマナーに従ってstreamを使う。

```JavaScript
const http = require('http');

const server = http.createServer((req, res) => {
  // `req` is an http.IncomingMessage, which is a readable stream.
  // `res` is an http.ServerResponse, which is a writable stream.

  let body = '';
  // Get the data as utf8 strings.
  // If an encoding is not set, Buffer objects will be received.
  req.setEncoding('utf8');

  // Readable streams emit 'data' events once a listener is added.
  req.on('data', (chunk) => {
    body += chunk;
  });

  // The 'end' event indicates that the entire body has been received.
  req.on('end', () => {
    try {
      const data = JSON.parse(body);
      // Write back something interesting to the user:
      res.write(typeof data);
      res.end();
    } catch (er) {
      // uh oh! bad json!
      res.statusCode = 400;
      return res.end(`error: ${er.message}`);
    }
  });
});

server.listen(1337);

// $ curl localhost:1337 -d "{}"
// object
// $ curl localhost:1337 -d "\"foo\""
// string
// $ curl localhost:1337 -d "not json"
// error: Unexpected token o in JSON at position 1
```

上記のアプリケーションのやっていること：

----

サーバを1337ポートで立てて

データ通信があったら、

req.on()というイベントエミッターメソッドが発火してデータを取得して

その細切れのデータ（chunk)をbody変数へ格納していく。

通信が終了したらdata変数へ書き込んで終了する。
----

`Writable`ストリームは(例の中では`res`)`write()`と`end()`メソッドというストリームへ書き込むメソッドを公開している。

`Readable`ストリームは、`EventEmitter`という、ストリームからデータを読み取ることができることをアプリケーションに通知してくれるAPIを使う。

取得できるデータは様々な方法でストリームから読み取ることができる。

`readable`も`Writable`も`EventEmitter`APIを使ったいろんな方法で、現在の状態のストリームと通信する。

ストリームにデータを書き込んだり、ストリームからデータを消費したりするアプリケーションは、ストリームインターフェイスを直接実装する必要はなく、通常、require（'stream'）を呼び出す理由はありません。

新しいタイプのストリームを実装したい開発者は、ストリーム実装者向けのセクションAPIを参照する必要があります(https://nodejs.org/docs/latest-v14.x/api/stream.html#stream_api_for_stream_implementers)

