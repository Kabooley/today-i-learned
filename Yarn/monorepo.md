# yarn workspaces

yarn で monorepo を管理するためのあれこれ。

yarn version は 1.22.22

## 公式

https://classic.yarnpkg.com/blog/2017/08/02/introducing-workspaces/

https://classic.yarnpkg.com/en/docs/cli/workspace

https://classic.yarnpkg.com/en/docs/cli/workspaces

## Summary

- [Set up flow](#set-up-flow)
- [hoisting](#hoisting)
- [プロジェクトの yarn バージョンを固定する](#プロジェクトのyarnバージョンを固定する)
- [ワークスペースに個別に依存関係をインストールする](#ワークスペースに個別に依存関係をインストールする)
- [ワークスペースを個別に script を実行する](#ワークスペースを個別にscriptを実行する)
- [気になる記事](#気になる記事)
- [Node.js バージョン管理について](#Node.jsバージョン管理について)

## Set up flow

```bash
$ yarn config set workspaces-experimental true

$ mkdir monorepo-jbook
$ cd monorepo-jbook
$ yarn init -yp
```

```diff json
{
    "name": "monorepo-jbook",
    "version": "1.0.0",
    "main": "index.js",
    "license": "MIT",
    "private": true,
+   "workspaces": [
+       "packages/cli",
+       "packages/local-api",
+       "packages/local-client"
+   ]
}

```

```bash
# Add package.json for each packages manually...
$ touch workspaces/cli/package.json
$ touch workspaces/local-api/package.json
$ touch workspaces/local-client/package.json
```

NOTE: （未検証ですが）各 workspace を`yarn init -y`するか、手動で package.json で追加するのかでモノレポとしてのちの管理が異なるかもしれないかも？

ここで各 package.json を各ワークスペースに応じて編集する...

```bash
$ cd monorepo-jbook
$ yarn install
```

プロジェクトで使う yarn バージョンを固定する

```bash
monorepo-jbook$ yarn policies set-version 1.22.22
Resolving 1.22.22 to a url...
Downloading https://github.com/yarnpkg/yarn/releases/download/v1.22.22/yarn-1.22.22.js...
Saving it into /home/teddy/udemy-stephan-portfolio-course/monorepo-jbook/.yarn/releases/yarn-1.22.22.cjs...
Updating /home/teddy/udemy-stephan-portfolio-course/monorepo-jbook/.yarnrc...
Done!
```

`yarn policies set-version`すると実行したディレクトリの階層に`.yarn`ディレクトリと`.yarnrc`が生成される

現状のディレクトリの状態：

```bash
.
├── node_modules
│   └── # packages/以下の依存関係はほぼここにインストールされた
├── package.json
├── packages
│   ├── cli
│   │   ├── package.json
│   │   └── node_modules
│   ├── local-api
│   │   ├── package.json
│   │   └── node_modules
│   └── local-client
│       ├── package.json
│       └── node_modules
│               # 以下のように実際の依存関係はpackages以下にはインストールされていない
│               ├── bin
│               └── @types
├── yarn-workspaces2.md
├── .yarn
└── .yarnrc

4 directories, 6 files
```

ここで各 packages に共通しているような依存関係は「巻上げ」される。

[巻き上げについて](#hoisting)

ちなみにこの通りルートディレクトリの node_modules にほとんどすべての依存関係がインストールされているが、ルートディレクトリの package.json の dependencies などは更新されていない。

## hoisting

複数のパッケージ間で重複する依存関係を、重複しないで済むように最適化してくれる機能。

これは yarn install に備わっている機能で、先の set up flow で yarn install されたときに自動的に実施される。

https://classic.yarnpkg.com/blog/2018/02/15/nohoist/

たとえば複数のパッケージを管理する monorepo で、package-a と package-b で全く同じ依存関係を持つとする。

例えばどちらも`react@17.0.2`を依存関係に持つとする。

プロジェクト全体から見たら`react@17.0.2`は重複していることになる。

これは無駄になるので、こうした依存関係の重複をなくすためにあるのが`hoisting`機能である。

`hoisting`を使うと、

-   重複する依存関係は root ディレクトリの node_modules で管理される
-   重複している依存関係を必要とするパッケージはルートディレクトリの依存関係を参照する

という最適化をもたらすと。

Node は依存関係を捜索する際に、パッケージが見つかるまで親ディレクトリの node_modules を再帰的に操作する仕組みである。

つまりそのプロジェクトの直近の node_modules から見つからなかったらその上の階層の node_modules を捜索し始め、それでもみつからなかったらさらに上の階層を...以下略

この Node の性質を利用して複数のパッケージ間で重複している依存関係をいっそのことすべて上位のディレクトリに配置するのが hoisting の機能である

`hoisting`は yarn workspaces の標準の機能である（たぶｎ）

https://classic.yarnpkg.com/blog/2018/02/15/nohoist/

https://tars0x9752.com/posts/yarn-hoisting

#### `nohoist`について

前提として`nohoist`機能は yarn の version 1.42 だかからである

なんか使わない方がいいみたい。

## ワークスペースに個別に依存関係をインストールする

https://classic.yarnpkg.com/blog/2017/08/02/introducing-workspaces/

`Managing dependencies of Workspaces`より。

ワークスペースに個別に依存関係をインストールしたい場合は、シングルリポジトリ同様そのワークスペースで適切な yarn コマンドを使えとのこと。

つまり、

```bash
package-a$ yarn add left-pad --dev
$ git status
# これは`package-a/package.json`
modified: package.json
# これは`project-root/yarn.lock`
modified: ../../yarn.lock
```

各 workspace は個別の yarn.lock ファイルを持たない。

代わりにルートディレクトリの yarn.lock ファイルが更新される。

package.json はワークスペース個別の者が更新される。

#### `yarn workspace package-a add left-padd`との違い

TODO: 個別の workspace に移動して`yarn add`するのと`yarn workspace package-a add`するのでは何が異なるのだ？

## ワークスペースを個別に script を実行する

## 気になる記事

https://github.com/yarnpkg/yarn/issues/4513

## Node.js バージョン管理について

複数の Node.js バージョンを使いたい場合、バージョンごとに Node.js のファイルサイズが要求される。

なので Node.js が例えば 10gb だとしたら、Node.js のバージョンを 2 つ以上使いたかったら PC のストレージは少なくとも 20gb 以上必要になる。
