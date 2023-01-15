# CORS

## 参考

- https://developer.mozilla.org/ja/docs/Web/HTTP/CORS
- https://www.tohoho-web.com/ex/cors.html
- https://qiita.com/att55/items/2154a8aad8bf1409db2b

## 前提

#### オリジンとドメインの違い。

オリジン：`https://yahoo.co.jp:443`

ドメイン：`yahoo.co.jp`

つまり、オリジンはプロトコルとドメインとポート番号からなる URL である。

#### 同一生成元ポリシー

ブラウザはその仕様で「同一生成元ポリシー」が実装されている。

これは異なるオリジンへのリソースのアクセスを制限する仕組みである。

どういうことかというと、

今 web アプリケーションが`https://www.example.com:4000`で動作しているとする。

このアプリケーションが画像リソース取得のために`https://www.example.com:5000`へ GET リクエストを送信した場合、

このリクエストは無効にさせられる。

理由はオリジンが異なるからである。

## CORS

MDN より。

> `あるオリジンで動作しているウェブアプリケーションに、異なるオリジンにある選択されたリソースへのアクセス権を与えるようブラウザーに指示するための仕組みです。`

つまり異なるオリジンへのアクセス権を許可する仕組みである。

具体的にどういう仕組みなのか。

## 具体的な仕組み

#### 単純リクエスト

https://developer.mozilla.org/ja/docs/Web/HTTP/CORS#%E5%8D%98%E7%B4%94%E3%83%AA%E3%82%AF%E3%82%A8%E3%82%B9%E3%83%88

クライアント側：

http リクエストヘッダに`Origin: (自分側のオリジンURL)`をつけてリクエストを送信する。
これは CORS してほしいから付与されているヘッダプロパティではなくて、
いつもこの通りなので実際クライアント側はすることがない。

この`Origin`は、このリクエストがどこから来たものなのかを示す。

サーバ側：

レスポンスヘッダに`Access-Control-Allow-Origin`プロパティを追加して、
許可する対象をその値に指定する。

この値がそのリソースへのアクセスを許可されたオリジンである。

つまり、サーバがそのリクエストを送ってきたオリジンを評価して許可するかどうかを判断するのである。

値が...

ワイルドカード：すべてのリクエストを許可する
null: **使うな**
<origin>: 指定のオリジンのみ許可する。

この`Access-Control-Allow-Origin`を含んだレスポンスを返すことで
クライアントに許可されたかどうかを知らせる。

#### preflight request

要はいまからこんなリクエストを送信したいのですがいいですか？
と伺ってからリクエストを送信するようなやり取り。

一番初めのリクエストが確認のためのリクエストとなり、サーバからの返事次第で本来のリクエストを送信する。

クライアント側：

`OPTIONS`メソッドで確認リクエストを送信する。
次のようなヘッダを含める。

```http
Access-Control-Request-Method: POST
Access-Control-Request-Headers: X-PINGOTHER, Content-Type
```

実際のリクエストが`POST`になること、`Content-Type`が含まれることになることを送信する。

サーバ側：

許可されたら次のようなヘッダを含むレスポンスが返される。

```
Access-Control-Allow-Origin: https://foo.example
Access-Control-Allow-Methods: POST, GET, OPTIONS
Access-Control-Allow-Headers: X-PINGOTHER, Content-Type
Access-Control-Max-Age: 86400
```

`~Allow~`で許可対象を回答している。

クライアント側：

プリフライトリクエストが許可されたことを確認したら、
クライアントは実際のリクエストを送信する。

#### preflight request and redirect

プリフライトリクエストを送信したときにリダイレクトが発生すると、
場合によってはエラーが発生する。

回避方法：

プリフライトをしないようにするか、リダイレクトをしないようにする、
単純リクエストの実にするなど。

#### 資格情報を含むリクエスト

**既定ではことなるオリジンに対して Cookie は送信されない**

なので Cookie を送受信可能にするには、クライアント・サーバ両方に受け付けるよう特別に実装しなくてはならない。

クライアント側：

```JavaScript
// XHR を使う場合

const xhr = new XMLHttpRequest();
xhr.withCredentials = true; // ここを追加。

// Fetch

fetch('https://trusted-api.co.jp', {
  mode: 'cors',
  credentials: 'include' // ここを追加。
});

// axios

axios.get('https://trusted-api.co.jp', {
  withCredentials: true
});

axios.defaults.withCredentials = true; // global に設定してしまう場合
```

サーバ側：

許可するオリジンを明示的に指定しなくてはならない。

ワイルドカードは使えない。

> サーバーは Access-Control-Allow-Origin, Access-Control-Allow-Headers, Access-Control-Allow-Methods ヘッダーで "\*" ワイルドカードを指定してはならず、
>
> Access-Control-Allow-Origin: https://example.com のように、明示的にオリジンを指定しなければなりません。

```
Access-Control-Allow-Origin: https://trusted-one.co.jp
// CORS を許可する Origin を明示的にする
Access-Control-Allow-Credentials: **true**
```

`Access-Control-Allow-Credentials: true`を含めなければレスポンスは無視されてクッキーは利用できない。

> **資格情報を含むリクエストを異なるドメインに行う場合、サードパーティクッキーポリシーも適用されます。このポリシーは、この章で説明しているように、サーバーやクライアントでの設定とは無関係に常に実施されます。**

ブラウザは既定では`XMLHttpRequest`または`Fetch`において資格情報を送信しない。

資格情報を送信するには`XMLHttpRequest`オブジェクトまたは`Request`コンストラクタにおいて特定のフラグを設定する必要がある。

Cookie を送信するとき、`withCrednetials = true`を設定する。
