# corepack

疑問: 結論

- corepack をインストールすればもはや nodenv は要らないのか: NO.
- 現環境に corepack を通じてモダン yarn をインストールできるか: YES.
- corepack を使った yarn や node のパッケージ管理:　下記参照

## 公式より

corepack はいまだ実験的な代物である

corepack は Node.js のパッケージマネージャ（yarn など）のバージョン管理を支援する実験的なツール

corepack は Node.js のインストール時に一緒に含まれるが、corepack が管理することになるパッケージマネージャは Node.js の配布に含まれないので、初めて使う場合は、corepack はそれらのパッケージマネージャの最新バージョンをネットワークからダウンロードする。

(解釈： yarn は Node.js のパッケージに含まれないから corepack は yarn を使う段階になって yarn をネットワークからダウンロードするよということ)

（パッケージマネージャの）アップデートは自己責任で。

corepack はカレントディレクトリから最も近い package.json ファイルを見つけ出して、package.json ファイルの`packageManager`プロパティを取り出して、

値がサポートされているパッケージ マネージャーに対応する場合、Corepack は関連するバイナリへのすべての呼び出しが要求されたバージョンに対して実行されることを確認し、必要に応じてオンデマンドでダウンロードし、正常に取得できない場合は中止します。

つまり、package.json に記載のある指定のパッケージマネージャ@バージョンをダウンロード、インストールし呼び出し時にはそいつを使うということ。

`corepack use yarn@X.XX.X`で使いたい yarn を指定すると、package.json の packageManager を自動的に上書きする。

実験的な代物と言いつつ、yarn の公式では installation に corepack を使うのが前提になっている。

#### 注意: corepack と nodenv

**corepack は Node.js のバージョン管理を行う代物ではない。Node.js が使うパッケージマネージャのマネージャである。つまり yarn のためにある。**

そのため corepack を使うからと言って nodenv を消す必要はない。

nodenv は Node.js のバージョン管理マネージャであって Node.js が使うパッケージマネージャのマネージャではないから。

Node v16 　と v20 を切り替えられるようにしておきたいなら nodenv はいまだ必要である。

corepack はyarn@1.22.19をyarn@4.3.1にプロジェクトごとに切り替え可能にしてくれる代物である。

両者は別物である。

## 導入から使うところまで

```bash
# corepackを使い始める前にyarnをアンインストールしておく
$ npm uninstall -g yarn
# move to your project
$ cd my-project
```

package.json に以下を追記

```diff JSON
{
+  "packageManager": "yarn@4.3.1"
}
```

```bash
# 念のため
$ yarn
yarn: command not found
$ corepack enable     # これやらなくていいのかも
# yarnを使う前に実施
$ corepack enable yarn
# バージョン通りのyarnになったか確認
$ yarn
4.3.1
```

これで問題なければ OK
