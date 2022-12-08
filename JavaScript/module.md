# Modules and JavaScript

## メモ

Bundling: モジュールをすべて一つのファイルに記述すること

## JavaScript のモジュールの概要

モジュールとスクリプトの違いは何だろう？

モジュールは ES6 で話を進める。

```html
<!-- HTMLから見て -->

<script type="text/javascript" src="./script1.js">
    // 通常のsscriptファイルのインポート
</script>

<script type="text/javascript" src="./script2.js">
    // 通常のsscriptファイルのインポート
</script>

<script type="module" src="./module1.js">
    // moduleファイルのインポート
</script>
```

-   module は単独のスクリプトのスコープにインポートされる。

    つまり、HTML ファイルにインポートされるわけじゃないよ。
    では`<script type="module" src="./module1.js">`とは何者なのか？

    まず`import/export`を使うファイルはすべて module ファイルとして扱われる。
    なので何かしら import または export するファイルを HTML へ組み込むならば
    それは`<script type="module" src="./module1.js">`として組み込むということ。

    `moduel1.js`が import したファイルは module1.js へスコープされている。

-   module はインポートされたファイルの中からしかスコープできないが、text/javascript は例外なくグローバルになる

-   module は基本的に厳格モードであるが script はそうでない

-   module ファイルの中身は外部からアクセスできない。export での未公開でき、import でのみ export 内容をスコープできる。

-   module はトップレベルの this は undefined であるが、script では`window`である。

-   module は同期的にインポートされる

-   module のインポートは実行前に行われる

    JavaScript エンジンはファイル内容をパースしているときに、
    インポートを同期的に実行する。

    ただしインポートする内容のダウンロードは非同期で行われる。
    なのでインポート操作のみが同期的に実行される。

-   export された値は import でコピーされるわけではなく、その参照を持つだけである。

-   import ファイルのダウンロードが完了するとファイル内容を実行してから、import 先のファイルが実行される

    なので`main.js`が`Math.js`というファイルをインポートしていたとしたら、
    パーサが main.js で import 分を見つけたら Math.js をダウンロードして、
    Math.js を実行する。
    その実行が完了してから、main.js が実行される。

-   module は import 時の初回にのみ評価される。

    https://ja.javascript.info/modules-intro#ref-416

> もし同じモジュールが複数の他の場所でインポートされる場合、そのコードは初回のみ実行されます。その後エクスポートしたものはすべてのインポートしているモジュールで利用されます。

> 1 度限りの評価は重要な結果をもたらすため、注意が必要です。

```JavaScript
// 📁 alert.js
alert("Module is evaluated!");
// 別のファイルから同じモジュールをインポート

// 📁 1.js
import `./alert.js`; // Module is evaluated!

// 📁 2.js
import `./alert.js`; // (nothing)
```

つまり、

はじめて import された時点でのみ評価されて、あとは別のファイルで import されても既に評価済という扱いになるということ。

例でいえば、

alert.js は 1.js で import された時点で評価されるけど、2.js で import したときにはすでに評価済となる、ということである。

aler.js には特に関数でラップされているわけではない alert()があるけど

この alert()が実行されるのは、alert.js が評価されたときのみ（1.js で import されたときのみ）である。
