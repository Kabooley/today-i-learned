# Browser

How it works.

## sources

https://www.chromium.org/Home/

## Chromium Docs

#### build on lunux.

https://chromium.googlesource.com/chromium/src.git/+/HEAD/docs/linux/build_instructions.md

要件がうちの PC 上回っていて悲しい。

## とある開発者のブログ記事より

https://web.dev/howbrowserswork/

記事は 2011 年のもの

以下、ポイントをメモする。

#### browser's main functionality

ブラウザが HTML を傍受して表示する方法は HTML と CSS の仕様に則る。

この仕様は W3C という web の標準仕様を定める素子区が定めている。

#### browser's main component

-   UI
-   browser engine
-   rendering engine
-   networking
-   UI backend
-   JavaScript interpreter
-   Data storage

#### Rendering Engine

IE: Trident
Firefox: Gecko
Safari: webkit
Chrome: Blink

webkit はオープンソースらしい。

Blink は

https://www.chromium.org/blink/#learning-about-blink-development

#### Rendering main flow

rendering エンジンはネットワーキング層からリクエストしたドキュメントの中身を取得しはじめる。

これが完了すると、以下の流れでレンダリングする。

-   Parsing HTML: DOM ツリー作成のため
-   render ツリーを構成する
-   render ツリーの座標（配置）を定める
-   ツリーを描く

parsing 段階とレンダリング段階では

content tree: 内容に関するツリー。HTML ドキュメントから HTML 要素を DOM node へ変換してツリー構造化させる
render tree: 見た目に関するツリー。パースされた CSS などの装飾要素情報と、content tree を組み合わせたツリーのこと

NOTE: 記事の図がわかりやすい

content tree と render tree の生成が完了したら、階層プロセスへ移行する。

この工程では node をスクリーン上のどこに配置するのか座標を定める

描画段階では、レンダー ツリーが走査され、UI バックエンド レイヤーを使用して各ノードがペイントされます。

これらの処理はたとえばすべての HTML ファイルがネットワークからダウンロード完了してから実行されるのではなくて、届き次第すぐに可能な限り実行されていく。

#### Parsing 一般的な

`Parsing`とは文字列を使用可能なコードへ変換する作業である。

パース結果は通常、読み取ったドキュメントの構造を表す node のツリーとなる。

これがシンタックスツリーまたはパースツリーと呼ばれる。

> 構文解析は、文書が従う構文規則 (文書が記述された言語または形式) に基づいています。構文解析できるすべての形式には、語彙と構文規則で構成される決定論的文法が必要です。文脈自由文法といいます。自然言語はそのような言語ではないため、従来の解析手法では解析できません。

parsing は 2 つのプロセスに分けることができる。

-   lexicial analyse
-   syntax analyse

lexicial analyse:

入力されたものを token に砕く。token は言語の単語という粒度に砕かれる。

syntax analyse:

文法を適用する。

parser は 2 つに機能を分割する。

-   lexer: 入力されたものを有効なトークンにまで砕く作業を担う
-   parser: 言語ルールに従ったドキュメント構造を分析してツリーを構成する役割を担う

なので、

ネットワークから取得したドキュメントは...

Document --> lexical analyses(lexer does) --> syntax analyses(parser does)
--> Parsed tree

と処理されてくる。

#### translation

## 他

#### ブラウザとは何かを学習するには

-   C++のネットワーク関係を知る
-   IPC (Inter Process Communication)プロセス間通信
-   chromium をクローンしていじってみれば？
-   assembly

つらい。
