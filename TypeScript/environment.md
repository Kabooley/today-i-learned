# Set up environment of Node.js + yarn + TypeScript

とりあえず基本的に型付きで行いたいからTypeScriptを日常的に使いたい

コードの挙動だけわかればいい学習用の環境なのでバンドリングとかしないからParcelとwebpackは導入しない。

https://www.typescripttutorial.net/typescript-tutorial/nodejs-typescript/

上記の環境開発がてっとりばやい。

この記事でまとめる。

## yarnでパッケージマネジメント

https://yarnpkg.com/getting-started/usage

npm とあんまり使い勝手変わらん

体感としてはちょっと早いくらい

`install`は`add`で。

`--save-dev`は`--dev`で。

## TypeScript

- command options:

https://www.typescriptlang.org/docs/handbook/compiler-options.html

- Compiler Options:

https://www.typescriptlang.org/ja/tsconfig

基本的な流れ: BuiLd & Run

TypeScriptのCLIコマンドで.ts, .tsxファイルをJavaScriptへコンパイルする。

コンパイルされた.js, .jsxファイルをnodeコマンドで実行する。

#### Command Options

https://typescriptbook.jp/reference/tsconfig/tsconfig.json-settings

ややこしいのが、`tsc`と`npx`である。

`tsc`でコンパイルできるし、`npx tsc`でもコンパイルできる。

`tsc`はもっとも低レベルのコマンドで、いろいろオプションをくっつけることでコンパイル内容をカスタマイズできるけれど、通常の開発で使うのは稀では？

たとえば、

- `dist/`へコンパイルしたファイルを出力されて、
- コンパイルオプションはどの`tsconfig.json`を基にするのか指定して、
- 出力ファイルの名前をどうして...


などをすべてオプションで指定しないといけない。

package.jsonで予め指定しておけばらくちんだけどね。

`npx`ってなんやねんの疑問は解決できなかったけど、

`node-ts`を使えばしらなくてもいい感じになるから今のところは無視。

わかっていることは、

`npx tsc`コマンドだけで、tsconfig.jsonにしたがってコンパイルしてくれる。

だからあらかじめoutDirとrootDirとかを指定しておけばコンパイル・出力をやってくれる。


#### Compiler Options

`tsconfig.json --init`で出力されるJSONファイルは予めいくつかのプロパティが、

コメントアウトすればすぐに使えるようになっている。

初期状態ではこのプロパティは二重のオブジェクトになっている。

外側のオブジェクトには以下に示すような任意のオプションを追加することができる。

- `include`オプション：

https://www.typescriptlang.org/ja/tsconfig#include

> プログラムに含めるファイル名またはパターンのリストを指定します。 ファイル名はtsconfig.jsonファイルを含んでいるディレクトリからの相対パスとして解決されます。

とにかくコンパイル対象ファイルはこの`include`に含める。

> includeとexcludeはグロブパターンのためのワイルドカードをサポートしています:

> * ゼロ個以上の文字列にマッチ（ディレクトリセパレータは除く）
> ? 任意の 1 文字にマッチ（ディレクトリセパレータは除く）
> **/ 任意階層の任意ディレクトリにマッチ

> グロブパターンがファイルの拡張子を含まない場合、サポートされる拡張子のみが含まれるようになります（例：.ts、.tsxと.d.tsはデフォルトでインクルードされ、.jsと.jsxはallowJsが設定された場合のみインクルードされます）。

実現したいのは、

**`src/`以下のファイル、ディレクトリのサポート拡張子ファイルをすべてコンパイル対象にして、`dist/`以下へ出力してほしい**

なので

既存のプロパティに
- rootDir: "./src"
- outDir: "./dist"

オプションとして
- include: ["src/**/*"]

を指定すれば、実現できるはず。

```json
{
  "compilerOptions": {
    /* Visit https://aka.ms/tsconfig to read more about this file */
    /* Language and Environment */
    "target": "es2016",                                  /* Set the JavaScript language version for emitted JavaScript and include compatible library declarations. */

    /* Modules */
    "module": "commonjs",                                /* Specify what module code is generated. */
    "rootDir": "./src",                                  /* Specify the root folder within your source files. */
    /* JavaScript Support */
    /* Emit */
    "outDir": "./dist",                                   /* Specify an output folder for all emitted files. */

    /* Interop Constraints */
    // "isolatedModules": true,                          /* Ensure that each file can be safely transpiled without relying on other imports. */
    // "allowSyntheticDefaultImports": true,             /* Allow 'import x from y' when a module doesn't have a default export. */
    "esModuleInterop": true,                             /* Emit additional JavaScript to ease support for importing CommonJS modules. This enables 'allowSyntheticDefaultImports' for type compatibility. */
    // "preserveSymlinks": true,                         /* Disable resolving symlinks to their realpath. This correlates to the same flag in node. */
    "forceConsistentCasingInFileNames": true,            /* Ensure that casing is correct in imports. */

    /* Type Checking */
    "strict": true,                                      /* Enable all strict type-checking options. */

    /* Completeness */
    // "skipDefaultLibCheck": true,                      /* Skip type checking .d.ts files that are included with TypeScript. */
    "skipLibCheck": true                                 /* Skip type checking all .d.ts files. */
  }
  "include": ["src/**/*"]
}

```

https://qiita.com/masato_makino/items/bf640a253d56b708fe0b

注意として、

tscコマンドに対しては、rootDirで指定した範囲を検索対象に制限しないそうで、

includeを使わないと明確にコンパイル対象を制限できないそうです。

#### ts-node

https://github.com/TypeStrong/ts-node#installation

TypeScript CLIの、

- `tsc index.ts`でコンパイル
- 出力ファイルの確認
- `node index.js`で実行

の少なくとも3つの工程を１コマンドで済ませてくれるパッケージ・モジュール

`ts-node index.ts`ですぐに結果がわかる

ということは、

あとはtsconfig.jsonの内容に従ってts-nodeに仕事してもらえばまともに開発が始められそうだね

というこことでts-nodeにtsconfigを認識してもらう。

https://typestrong.org/ts-node/docs/configuration/

ts-nodeはいくつかの、tsconfig.jsonで指定したオプションをサポートする。

指定方法は、CLIのフラグとして、環境変数として、プログラマティックに。

推奨は`tsconfig.json`。

どういうことかというと、ts-nodeはどうやら自動でtsconfig.jsonをさがしてそれに従ってくれるらしい。

> ts-nodeは、tsconfig.jsonを自動的に検索してロードします。ほとんどのts-nodeオプションは、プログラムによるキャメルケース名を使用して「ts-node」オブジェクトで指定できます。 node --require ts-node / registerなどのCLIフラグを渡すことができない場合や、シバンを使用する場合でも機能するため、これをお勧めします。

CLIでわざわざ`--project`オプションを追加すると任意の`tsconfig.json`を指定できる。

`--skipProject`でスキップさせることも可能。

...ということで次のコマンドだけで十分なのかも...


`ts-node`

##### 余談：ts-nodeの便利な使い方

その場でTypeScriptコードを実行できる

```bash
# コマンドライン上でTypeScriptコードが打てる
# Ctrl + Dで離脱
ts-node

# Execute code with TypeScript.
ts-node -e 'console.log("Hello, world!")'

# Execute, and print, code with TypeScript.
ts-node -p -e '"Hello, world!"'

# ファイルを指定することでファイルがビルド・実行される
ts-node <file-path>
```

##### ts-nodeはコンパイルするだけ

なのでbuildコマンドですな。

実行は別で`node`コマンドは知らせないといかん。

#### TypeScript build & run を一度にしてほしいとき



https://www.npmjs.com/package/concurrently

https://www.typescripttutorial.net/typescript-tutorial/nodejs-typescript/

`nodemon`と`concurrently`を使う。

`nodemon`: 指定のディレクトリかファイルを監視して変化があったら実行する

`concurrently`: 2つのコマンドを一度に実行する

最終的には次の`script`が出来上がるはず

```JSON
{
    "script": {
        "start:build": "npx tsc -w",
        "start:run": "nodemon ./dist/index.js",
        "start": "concurrently npm:start:*",
        // "build:run": "ts-node"  // ts-nodeはもはや指定のファイルがどうなるかて鳥羽役確認するためのコマンドになっているのでいらないかも
    },
}
```

- `npx tsc -w`: rootDirでの変化を監視しながら自動的にコンパイルを実行してくれる
- `nodemon ./dist/index.js`: ビルドされたファイルがtsconfig.jsonのoutDirで指定した出力場所へ出力されるはずだから、ビルドのたびにnodemonが反応してくれるはず
- `concurrenlty npm:start:*`: `npm:start:`から始まるすべてのコマンドを一度に実行してくれる

NOTE: `npx`がないといけないのかどうかは未確認。

`tsc -w`だけでもいいのかも。

ということでインストールする。

```bash
$ yarn add nodemon concurrently --dev 
```

```bash
$ npm start
# 問題なく実行できていればOK
```

ts-nodeは特定のファイルだけコンパイルしたいときに使えばいい。