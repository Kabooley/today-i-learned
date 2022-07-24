# Inside of Git

# 参考

https://mixi-developers.mixi.co.jp/22-technical-training-5fc362a9dc41

https://git-scm.com/book/ja/v2/Git%E3%81%AE%E5%86%85%E5%81%B4-Git%E3%82%AA%E3%83%96%E3%82%B8%E3%82%A7%E3%82%AF%E3%83%88

## 目次

[まとめ](#まとめ) Git の内部構造のまとめ
[Git の内部構造](#Gitの内部構造)

## まとめ

#### 3つの領域

- 作業ツリー
- インデックス（ステージ）: `.git/index/`
- コミット履歴: 


#### Git を add したら起こっていること

Git は add, commit 時に独自のオブジェクトを生成する。

Git が生成するオブジェクトは主に以下の４つである。

-   commit: コミット情報が入っている
-   tree: ディレクトリの情報が入っている
-   blod: ファイルの情報が入っている
-   tag: annotating tag の情報が入っている

`.git/objects/`以下に zlid 圧縮形式で保存される。

NOTE: `git add`されたら、以下の 2 つのみを行っている。

-   Git は add したファイルを`.git/index`に登録している
-   add したファイルごとに blob オブジェクトが生成される

blob ファイルは add したときのファイルの情報を保存している。

blob ファイルは`git/objects/blobハッシュ値の上位2桁/blobハッシュ値の上位2桁以下の値`に保存される

blob ファイルはステージングごとに生成されいつも異なるハッシュ値になるので各ステージングはすべて区別される。

`git ls-files`はインデックスと作業ツリーのファイルの情報を表示するコマンドである。

`git ls-files --stage`は**ステージングされた**コンテンツの mode とオブジェクトの名前とステージ番号とハッシュ値を表示させるので、

add したファイルのステージング状態を知るにはちょうどいいコマンド。

#### git commit すると起こること

1. index から tree オブジェクトを生成する
2. commit オブジェクトを生成する
3. HEAD を新しい commit ハッシュへ書き換える

コミットをすると、

ステージング時の blob オブジェクトは生成済として、

tree オブジェクトはそのディレクトリの blob オブジェクトと下層の tree オブジェクトへの参照を保存し、

commit オブジェクトは、tree オブジェクトの参照とそのコミット時のタイムスタンプやコミットメッセージを保存する。

tree はどの blob と tree なのかを知っていて、commit オブジェクトはどの tree なのかといつどこの誰の情報を知っているので、

コミットをするとその時点のスナップショットを保存できるのである。

参照構造：

commit --> tree --> blob --> contents

各矢印は矢印の先の対象のハッシュ値を矢印の元が持っているという意味である。

## Git の内部構造

#### git add したときに何が起こっているか

「コミットさせたいファイルを index に登録している」

index とは.git/index である

index の中身を確認するコードがある

```bash
# README.mdをaddしたとする
$ git ls-files --stage

# ファイルの種類＋パーミッション blobハッシュ コンフリクトフラグ ファイル名
# という並び
10064 e23fs43232fsr432fse232 0 README.md
```

Git は様々なデータをオブジェクトと呼ばれる概念で表現しており

オブジェクトには次の 4 つがある

-   commit: コミット情報が入っている
-   tree: ディレクトリの情報が入っている
-   blod: ファイルの情報が入っている
-   tag: annotating tag の情報が入っている

オブジェクトの実態は次の場所に zlib 圧縮形式で保存されている

`.git/objects/`

先のやつなら blob オブジェクトが

`.git/objects/e2/3fs43232fsr432fse232`に保存されている

各オブジェクトの中身を確認するためのコマンドがある

`git cat-file -p <object-hash>`

あたらしくファイルを作ってオブジェクトを確認してみると

```bash
$ echo "This is README" > README.md
$ git add README.md
$ git ls-files --stage
100644 171cd12d63717a3f594589b2313e9470fcd1003c 0       README.md
100644 d158b023ecbf9e0a69daa8536f04273587396977 0       initial-commit.txt

# 新たにファイルを保存する
$ echo hoge > hoge.txt
$ git add hoge.txt
$ git ls-files --stage
100644 171cd12d63717a3f594589b2313e9470fcd1003c 0       README.md
100644 2262de0c121f22df8e78f5a37d6e114fd322c0b0 0       hoge.txt
100644 d158b023ecbf9e0a69daa8536f04273587396977 0       initial-commit.txt
```

**git add は基本的に index の更新と blob オブジェクトの生成しかしていない**

ディレクトリを作成してみる

```bash
# huga/huga.txtを作ってaddした
$ git ls-files --stage
100644 171cd12d63717a3f594589b2313e9470fcd1003c 0       README.md
100644 2262de0c121f22df8e78f5a37d6e114fd322c0b0 0       hoge.txt
100644 e69de29bb2d1d6434b8b29ae775ad8c2e48c5391 0       huga/huga.txt
100644 d158b023ecbf9e0a69daa8536f04273587396977 0       initial-commit.txt
```

ディレクトリの追加が起こったけれど、tree オブジェクトは生成していない

#### git commit したときに何が起こっているか

##### tree object

主にこの 3 つを行っているらしい

1. index から tree オブジェクトを生成する
2. commit オブジェクトを生成する
3. HEAD を新しい commit ハッシュへ書き換える

tree オブジェクトは Git が管理するあらゆるコンテンツへの多くのポインタを含む

コミットをすると、差分だけでなくて、リポジトリのルートディレクトリを含むすべてのディレクトリの tree オブジェクトを自動で生成する。

先の add してあったものをコミットして、tree オブジェクトをのぞいてみる

```bash
# master^{tree} のシンタックスは、
# master ブランチ上での最後のコミットが指しているツリーオブジェクトを示します。
# つまり最後のコミットのtreeオブジェクトをcat-fileしてねみたいなコマンド
$ git cat-file -p master^{tree}
100644 blob 171cd12d63717a3f594589b2313e9470fcd1003c    README.md
100644 blob 2262de0c121f22df8e78f5a37d6e114fd322c0b0    hoge.txt
040000 tree b5a33ea70f1cb1d361014f0f88a2f724fb9522ca    huga
100644 blob d158b023ecbf9e0a69daa8536f04273587396977    initial-commit.txt
```

つまり、

コミットするとそのディレクトリのファイル情報は blob オブジェクト

ディレクトリの情報は tree オブジェクトに保存する

blob オブジェクトはファイルの情報、tree オブジェクトは blob オブジェクトと下層のディレクトリの tree オブジェクトを保存する

なのでコミットするとその時点のスナップショットを保存できるのである

ファイルの状態、ディレクトリの状態をすべて記録するから。

つまり tree オブジェクトはある時点のスナップショット情報を参照できるということにもなる

上記の各ファイルをそれぞれ変更してまたコミットしてみる

```bash
# 次のコミットしたあとのルートディレクトリのtreeオブジェクトの中身
$ git cat-file -p master^{tree}
100644 blob 4cf5010e7b78cfd61d33983044299260d21b9189    README.md
100644 blob 1904c092b649dc54f3c8fc931acb0ca5bb952c3b    hoge.txt
040000 tree 8406deefc043c8f56777cd1ca55610536442d1f3    huga
100644 blob 5c6af654a3b833764a26e8baf849312555134997    initial-commit.txt
```

先のコミットと最新のコミットで各オブジェクトの SHA-1 がまったく異なるのがわかる

ということは、たとえばコミットを以前の状態に戻したいというときには

その時の tree オブジェクトが必ず必要になるということで

どの tree オブジェクトなの？の情報も必要になってくる

それを収めるのが commit オブジェクトである

##### commit object

commit オブジェクトが収めている情報

-   ルートディレクトリの tree オブジェクトの SHA-1
-   committer と author のタイムスタンプ、名前、メアド
-   親コミット・ノードの SHA-1
-   コミット・メッセージ

確認方法解らん

改善されているかどうかの確認：親コミットを commit オブジェクトに含めることで改ざんされていないことが保証される

commit オブジェクトの一部を変更（改ざん）すると、別の commit ハッシュに変わってしまうから

ブロックチェーンと同じ仕組みみたい

ということでここまでの簡単なまとめ：

-   commit をすると作られるオブジェクトは４つ
    commit, tree, blob
-   blob オブジェクトはファイルの情報を保存するオブジェクトである。ファイルを復元するのにこの情報が必要である。
-   tree オブジェクトはその時点のスナップショットを保存するオブジェクトである。
    ルートディレクトリのファイル情報を保存した blob ファイルへの SHA-1、ルートディレクトリ以下のディレクトリを収めた tree オブジェクトへの SHA-1 を保存してあるので、その時点の状態を後から復元できる。
    tree オブジェクトはディレクトリごとに作成される。
-   commit オブジェクトはスナップショットを取ったときの tree オブジェクトの SHA-1 とタイムスタンプ、コミットメッセージ、誰がコミットしたのかの情報
-   commit オブジェクトが tree オブジェクトを参照しており、tree オブジェクトは blob オブジェクトを参照する

#### HEAD

コミットしたときに内部的に起こること仕上げ。

commit オブジェクトを作ったら、Git は最後に HEAD を書き換える。

HEAD とは？その前に refs を理解しないといかん。

refs は特定の commit を指すポインタのようなもの。

HEAD は現在の commit を指す refs の一つである。

checkout すると HEAD は書き変わっている。（別のコミットを指すことになるから）

.git/HEAD に保存されている

```bash
$ cat .git/HEAD
ref: refs/heads/master
```

commit すると、HEAD を書き換えるという話だけど

-   HEAD が直接 commit ハッシュを参照している場合：HEAD の commit ハッシュを書き換える

    これって git checkout commit-hash しているときに commit した場合のことかしら？

HEAD が branch を参照している場合：HEAD が参照している branch の commit ハッシュを書き換える

    ほとんどこの通りなんじゃないの？

## ちょっとコマンドまとめ

確認用コマンドまとめ：

`git ls-files -s`:

staging-area（ステージングエリア）でインデックスされているファイルを確認したい場合に利用するコマンド

`-s`は blob の hash 値とファイル名の一覧を表示する

blob オブジェクトはステージングされたときに生成されるファイル情報を保存した git オブジェクトである。

`git cat-file -p master^{tree}`:

> cat-file コマンドを使うと、コンテンツを Git から取り出すことができます。 このコマンドは、Git オブジェクトを調べるための万能ナイフのようなものです。 -p オプションを付けると、cat-file コマンドはコンテンツのタイプを判別し、わかりやすく表示してくれます。

`master^{tree}`は、master ブランチの最新のコミットにおけるルートディレクトリの tree オブジェクトを指定している

#### git hooks

.git/hooks/にスクリプトを書いておくと、イベントが来たときに自動的にスクリプトが発火してくれるらしい

Git フックは、Git でコマンドを実行する直前もしくは実行後に特定のスクリプトを実行するための仕組みです。

たとえば、pre-commit という Git フックを使用すると、git commit を実行する直前に任意のスクリプトを実行することができます。これを利用することで、コミットメッセージにイシューやチケットの番号が含まれていなかったらコミットを中止する、などの処理を行うことができるようになります。

ということでミスを減らすための仕組みのようです

#### Packfile

https://git-scm.com/book/ja/v2/Git%E3%81%AE%E5%86%85%E5%81%B4-Packfile

Git は基本的に毎回 blob オブジェクトを生成するときは差分じゃなくてファイルすべてをバックアップする

なので容量の大きいファイルを毎回 blob させていたらさぁ大変

そんな時に差分だけを記録する仕組みがある

詳しくは上記のリンクを...

#### どこからも参照されていないオブジェクトを探すとき

git fsck を使うらしい
