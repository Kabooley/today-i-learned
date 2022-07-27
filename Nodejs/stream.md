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

streamとか内部バッファつかうし全然理解していないから通常の環境で使うと大変なことになりそう。

仮想環境に開発環境を立てて実験する。

ちょっとwsl2 + Ubuntu メモ

```bash
# windows powershell

# インストールされている Linux ディストリビューションを一覧表示する
wsl --list --verbose
# wslの状態を確認する
wsl --status
# PowerShellから特定のLinuxディストリビューションを実行する
wsl --distribution <Distribution Name> --user <User Name>
```
WSLを使用してLinuxディストリビューションを実行する方法：

https://docs.microsoft.com/ja-jp/windows/wsl/install#ways-to-run-multiple-linux-distributions-with-wsl

とにかくWindows10のツールバーの検索窓からLinuxディストリビューションを入力すればディストリビューションコマンドプロンプトが開くからとにかくこれで開く

anyenvでnodeﾊﾟｯｹｰｼﾞﾏﾈｰｼﾞｬを管理してもらうので、homebrewをインストールする

```bash
# そもそもすでにインストール済か確認する
brew --version

Command 'brew' not found, did you mean:

  command 'qbrew' from deb qbrew (0.4.1-8build1)
  command 'brec' from deb bplay (0.991-10build1)

Try: sudo apt install <deb name>

anyenv -v
anyenv: command not found

# とにかくいろいろする前はアップデートです
sudo apt get && sudo apt upgrade -y

# 公式のコマンドをそのまま実行すればいいみたい
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

https://docs.brew.sh/Homebrew-on-Linux

NOTE: Ubuntu系は.profile/のscriptに、とのこと

```bash
test -d ~/.linuxbrew && eval "$(~/.linuxbrew/bin/brew shellenv)"
test -d /home/linuxbrew/.linuxbrew && eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"
test -r ~/.bash_profile && echo "eval \"\$($(brew --prefix)/bin/brew shellenv)\"" >> ~/.bash_profile
echo "eval \"\$($(brew --prefix)/bin/brew shellenv)\"" >> ~/.profile

# https://github.com/znz/anyenv-update
mkdir -p $(anyenv root)/plugins
git clone https://github.com/znz/anyenv-update.git $(anyenv root)/plugins/anyenv-update
anyenv update

# ここまででenv系のセットアップ完了

# nodejsの開発環境の構築


```
TODO: Ubuntuを再インストールした方がいいかも...

他のenv系をインストール済の時に他のenv系を突っ込んではならないけどやってしまった...


まぁ問題起こってからでいいか。



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

