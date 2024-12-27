# Project References

一つのプロジェクトに対して複数 tsconfig.json ファイルが必要になった時の適切な tsconfig.json の運用方法について

どのように設定、運用するべきなのかを記す

## Summary

-   具体的なプロジェクトと解決するべき問題
-   単純に複数 tsconfig.json ファイルを生成すればいいというわけでない理由
-   `include`/`exclude`は上書きされない
-   `reference`オプションで複数 tsconfig.json ファイルの管理

## 情報収集

https://stackoverflow.com/a/56542874

https://echobind.com/post/deep-dive-into-extending-tsconfig-json

[how-to-use-references-in-typescript](https://stackoverflow.com/a/55000497)

[Playwright tutorial](https://playwright.dev/docs/test-typescript)

## [TypeScript] 複数 tsconfig.json ファイルの管理

開発環境用、テスト環境用など使い分けたいときに直面する extends する際の問題

```bash
|
+-- browser-test/
|   +-- tsconfig.mocha-browser.json
|
+-- src/
+-- package.json
+-- tsconfig.json
```

上記の通り、プロジェクト直下の開発用の tsconfig.json と

テスト環境（browser-test）要の tsconfig.mocha-browser.json の２つの tsconfig ファイルがある。

`tsconfig.json`:

```JSON
{
    "compilerOptions": {
        "jsx": "react-jsx",
        "module": "es6",
        "target": "es6",
        "moduleResolution": "bundler",
        "resolveJsonModule": true,
        "experimentalDecorators": true,
        "esModuleInterop": true,
        "lib": ["dom", "es6"]
    },
    "include": ["src/**/*.ts", "src/**/*.tsx"],
    "exclude": ["node_modules"],
    "compileOnSave": false
}
```

`tsconfig.mocha-browser.json`:

```JSON
{
    "extends": "../tsconfig.json",
    "compilerOptions": {
        "lib": ["esnext", "dom"]
    },
    "include": ["../src/**/*.ts", "../src/**/*.tsx", "./**/*.ts"]
}
```

何が問題かというと、extends する側の` include``exclude `は

現状、`browser-test/`以下で適用される tsconfig 設定は以下の通り（`--showConfig`オプションで確認）

TODO:　とはいえ可能性はある

```bash
$ cd browser-test/
# TODO: `tsconfig.mocha-browser.json`をコンフィグファイルとして認識していない
# そのため結局tsconfig.jsonの結果しか表示していない
# TODO: configファイルを指定する
$ npx tsc --showConfig
```

#### [tsc] `tsc --showConfig` オプションで tsconfig ファイルの最終形態を確認

`--showConfig`オプションをつけて tsc コマンドを実行すると最終的に適用されるコンフィグが JSON 形式で表示される

#### 試していること

tsconfig.json:

```diff
{
    "compilerOptions": {
        "jsx": "react-jsx",
        "module": "es6",
        "target": "es6",
        "moduleResolution": "bundler",
        "resolveJsonModule": true,
        "experimentalDecorators": true,
        "esModuleInterop": true,
        "lib": ["dom", "es6"]
    },
-   "include": ["./src/**/*.ts", "./src/**/*.tsx"],
+   "include": ["src/**/*.ts", "src/**/*.tsx"],
    "exclude": ["node_modules"],
    "compileOnSave": false
}
```

ひとまず上記に変更しても問題なくバンドルできる

browser-test/tsconfig.mocha-browser.json:

```JSON
{
    "extends": "../tsconfig.json",
    "compilerOptions": {
        "lib": ["esnext", "dom"]
    },
    "include": ["src/**/*.ts", "src/**/*.tsx", "browser-test/**/*.ts"]
}

```

#### extends

> It’s worth noting that "files", "include", and "exclude" from the inheriting config file overwrite those from the base config file, and that circularity between configuration files is not allowed.

> 「ファイル」「インクルード」「エクスクルード」は継承する設定ファイルからのもので、ベースの設定ファイルからのものを上書きすること、そして設定ファイル間での循環は許可されていないことに注意が必要です。

#### rootDir

default: tsconfig.json ファイルが存在しているディレクトリ

## [TypeScript 公式] Project References

https://www.typescriptlang.org/docs/handbook/project-references.html

たとえば開発環境（src/）とテスト環境（test/）をひとつのプロジェクトにまとめているとき

単一の tsconfig.json ファイルしか使えなかったとき

-   実装ファイルをテストファイルが import することは可能であった
-   出力フォルダ名称に`src`が表示されずに test と src を同時にビルドすることは不可能である
-   実装ファイルを変更したらテストして再度の型チェックをする必要があるがこれによってあらたなエラーは発生しない
-   テストファイルの変更は実装が変更されていなくても再度の型チェックが必要であった

複数 tsconfig.json ファイルはこのうちいくつかを解決したけど

-   ファイルの更新によって都度実行されるような組み込みの型チェック機能はないからつねに貴方は tsc を二度（実装とテスト）実行する必要がある
-   tsc を二度実行することはオーバーヘッドを増やす
-   tsc -w 　は複数コンフィグファイルを一度に実行できない

プロジェクトの参照とは？

`"reference"`オプションの`"path"`は`tsconfig.json`ファイルを持つディレクトリを参照することができる（もしくは tsconfig.json ファイルそのものを参照できる）

`composite`

参照される方のプロジェクト（ディレクトリ）は`composite`オプションを有効にしなくてはならない

`composite`オプションの有効化はいくつかの変化をもたらす

-   `rootDir`設定は明示的に設定されていない場合、`tsconfig.json`ファイルが存在するディレクトリを`rootDir`とする

-   すべての実装ファイルは、インクルードパターンに一致するか、ファイル配列にリストされている必要があります。この制約に違反した場合、tsc は指定されていないファイルを通知します。

-   `declaration`を有効にしなくてはならない

Project References の注意点：

#### [Project References] 聞きかじり

https://stackoverflow.com/a/55000497

ことなるフォルダ（src/, test/）ごとに tsconfig.json ファイルを用意して、共通の設定を定義している tsconfig-base.json をルートプロジェクトに置く

src/tsconfig.json と test/tsconfig.json はそれぞれ tsconfig-base.json を extends する

test/は src/に依存するため、この「関係」を TypeScript へ伝える必要がある

その明文化するための機能が`reference`であると

## [走り書き] `rootDir`

https://www.typescriptlang.org/tsconfig/#rootDir

Default:

> デフォルト: すべての非宣言入力ファイルの最長共通パス。composite が設定されている場合、デフォルトは tsconfig.json ファイルを含むディレクトリになります。

## [走り書き] `baseUrl`

https://www.typescriptlang.org/tsconfig/#baseUrl

NOTE: `AMD`モジュールのために追加されたオプションであり、`paths`オプションを使うならば最早必要ないオプションである

## [走り書き] `paths`

モジュール解決する際のファイル検索場所を指定する

`baseUrl`が指定されていればそこからの相対パス

または`tsconfig.json`ファイルが
