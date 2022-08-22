# Prototype & Inheritance

継承とプロトタイプについて

## 目次

[継承とプロトタイプ](#継承とプロトタイプ)
[コンストラクタ関数と`F.prototype`](#コンストラクタ関数と`F.prototype`)
[`constructor`プロパティ](#`constructor`プロパティ)
[](#)

TODO: prototypeとconstructorについてこんがらがってきたので整理

## 継承とプロトタイプ

https://ja.javascript.info/prototypes

基本：

JavaScript においてプロトタイプチェーンを実現できるのはすべてのオブジェクトがプロトタイプという特別なプロパティを持つ。

ここでいうプロトタイプとは`[[Prototype]]`を指す。

`[[Prototype]]`とは、JavaScript オブジェクトが必ずもつ特別なプロパティである。

`[[Prototype]]`はプロトタイプチェーンを実現しているプロパティで、自身のプロトタイプとなるオブジェクトを指している。

`[[Prototype]]`は JavaScript がちょせくアクセスできるプロパティではなく、「隠されている」。

しかし、実は`[[Prototype]]`は`__proto__`プロパティでアクセスできる。

さらに言えば`__proto__`は`[[Prototype]]`という隠されたプロパティを「暴露」するアクセサ（getter/setter）なので非推奨（MDN 曰く）である。

ということで、

-   `[[Prototype]]`プロパティはすべての JavaScript オブジェクトが持つ特別なプロパティでこれのおかげでプロトタイプチェーンが成立する。

-   `__proto__`は`[[Prototype]]`へアクセスする手段で本来使用すべきでない。

確認：

```JavaScript
{
    let animal = {
        eats: true,
    };
    let rabbit = {
        jumps: true,
    };

    //通常のオブジェクトは初期生成時にObject.prototypeが自身のプロトタイプになる
    console.log(animal.__proto__ === Object.prototype); // true
    console.log(rabbit.__proto__ === Object.prototype); // true

    // __proto__はプロトタイプを変更させることができてしまう
    rabbit.__proto__ = animal; // (*)

    // 変更されたのがわかる
    console.log(rabbit.__proto__ === Object.prototype); // false
    console.log(rabbit.__proto__); // animal{}

    // rabbitのプロトタイプにanimalが割り当てられたので
    // ﾌﾟﾛﾄﾀｲﾌﾟﾁｪｰﾝでanimalのプロパティにアクセスできるようになった
    console.log(rabbit.eats); // true (**)
    console.log(rabbit.jumps); // true

    // `prototype`というプロパティは通常のオブジェクトはアクセスできない
    console.log(animal.prototype);  // undefined
    console.log(rabbit.prototype);  // undefined
}

```

#### まとめ:継承とプロパティ

-   `[[Prototype]]`は JavaScript オブジェクトすべてが持つ特別なプロパティである
-   `[[Prototype]]`は通常隠されているプロパティでアクセスする手段がないが、`__proto__`はそれを暴露するので通常、使用は非推奨である。
-   `__proto__`は`[[Prototype]]`の getter/setter である
-   `__proto__`に代入することでオブジェクトの`[[Prototype]]`を変更することは可能なので非常に危険である
-   `prototype`というプロパティはコンストラクタ関数のみがアクセスできるプロパティで、通常のオブジェクトもインスタンスもアクセスできない
-   `F.prototype`は`new F()`されるときに、生成されるインスタンスの`[[Prototype]]`へ`F.prototype`の値を割り当てるのに使用されるだけである
-   インスタンスを作ってから`F.prototype`を変更しても既存のインスタンスの`[[Prototype]]`はそれに伴って変更されることはなく固定されたままである
-   プロトタイプチェーンは`[[Prototype]]`をたどることで親オブジェクトのプロパティにアクセスできる


## コンストラクタ関数と`F.prototype`

-   `F.prototype`というプロパティは、コンストラクタ関数のみがアクセスできるプロパティである

-   通常のオブジェクト、インスタンスオブジェクトも`prototype`というプロパティはアクセスできない

-   `F.prototype`は`new F()`するときしか使われない

`f.__proto__`は`new F()`するときに固定されて後から`F.prototype`を後から変更しても既存のインスタンス`f.__proto__`は変更されないという意味

-   `F.prototype`は`[[Prototype]]`と同じではない。`new F()`されたときのみ`[[Prototype]]`へ新しいオブジェクトを割り当てる

とにかく`new F()`するときだけしか出番がないのである

-   `F.prototype`の値は null かオブジェクトでなくてはならない

`F.prototype`の挙動確認:

```JavaScript

{
    // F.prototype
    //
    // F.prototypeというプロパティはコンストラクタ関数だけがアクセスできるプロパティ
    //
    // Rabbit.prototype --> animal
    // rabbit.__proto__ --> animal

    let animal = {
        eats: true
    };

    const Rabbit = function(name) {
        this.name = name;
    };

    // 通常のオブジェクトだとprototypeというプロパティへアクセスできないが
    // コンストラクタ関数だとprototypeへアクセスできる
    console.log(animal.prototype);  // undefined
    console.log(Rabbit.prototype);  // {constructor: f()}
    console.log(Rabbit.prototype.constructor === Rabbit);   // true

    Rabbit.prototype = animal;

    // prototypeへ直接オブジェクトを割り当てると
    // prototype経由で割り当てたオブジェクトのプロパティへアクセスできる
    console.log(Rabbit.prototype);  // {eats: true}
    console.log(Rabbit.prototype.eats); // true


    const rabbit = new Rabbit("inaba");

    // インスタンスからもF.prototypeにはアクセスできないことの確認
    console.log(rabbit.prototype);  // undefined
    console.log(rabbit.__proto__);  // Object: { eats: true }

    // F.prototypeから割り当てられたオブジェクトを参照できていることの確認
    console.log(rabbit.eats);   // true
    Rabbit.prototype.eats = "carrot";
    console.log(animal.eats);   // carrot

}


{
    // F.prototypeはnew F()するときしか使われないことの確認
    //
    // つまり、インスタンス.__proto__はnewするときに固定されて
    // 後から変更はされないということ

    let animal = {
        eats: true
    };

    let robot = {
        eats: false
    };

    const Rabbit = function(name) {
        this.name = name;
    };

    Rabbit.prototype = animal;

    const rabbit = new Rabbit("inaba");

    // 後出しでRabbitのprototypeを変更しても
    // rabbitのprototypeはanimalを指す
    //
    // つまり、インスタンスにとっては
    // F.prototypeはnew F()するときしか使われないことがわかる
    Rabbit.prototype = robot;
    console.log(rabbit.__proto__ === animal);   // true

    // Rabbit.prototypeにrobotoを割り当てた後にnewしたインスタンスでは
    // robotを指しているので
    // まちがいなく
    // F.prototypeはnew F()するときしか使われないということがわかる
    const blackRabbit = new Rabbit("black rabbit");
    console.log(blackRabbit.__proto__ === robot);   // true
}
```

インスタンス自体はただのオブジェクトなので、当然 `f.__proto__`で直接変更できるけど、

その影響はそのオブジェクトだけであり、すべての同一コンストラクタからなる既存インスタンスは影響を受けない。

-   `F.prototype`は、`F`が継承していないコンストラクタなら自身を指す

```JavaScript

{
    // F.prototypeは継承していないコンストラクタなら自身を指す
    // 派生クラスなら基底クラスを指す

    const Rabbit = function(name) {
        this.name = name;
    };

    class NewRabbit extends Rabbit {
        constructor(name) {
            super(name);
        }
    }

    console.log(Rabbit.prototype);      // Rabbit
    console.log(NewRabbit.prototype);   // Rabbit
}
```


## `constructor`プロパティ

https://ja.javascript.info/function-prototype

https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Object/constructor

> constructor プロパティは、インスタンスオブジェクトを生成した Object のコンストラクター関数への参照を返します。

(Object.create(null) で生成されたオブジェクトを除いて) すべてのJavaScriptは`prototype`プロパティを持つ。

**デフォルトでは`prototype`は`constructor`というプロパティだけを持つオブジェクトである。**

つまり`F.prototype`は`F.prototype.constructor`である。

だから上記の[コンストラクタ関数と`F.prototype`](#コンストラクタ関数と`F.prototype`)での話はそのまま`constructor`プロパティの話である。


```JavaScript
    // 
    // Object.prototype.constructorが何を指すのかの確認
    // 

    function Rabbit() {};

    // デフォルトでprototypeプロパティはconstructorというプロパティのみを持つ
    console.log(Rabbit.prototype.constructor === Rabbit);   // true

    // `new`はインスタンスの__proto__に、続くコンストラクタ関数の`prototype`のオブジェクトを割り当てる
    const rabbit = new Rabbit();

    // インスタンス.constructorはnewされたときのコンストラクタ関数を指すのではなく、
    // newされたときのコンストラクタ関数のprototypeを指すことの確認
    // その１
    console.log(rabbit.constructor === Rabbit);     // true
    console.log(rabbit.constructor === Rabbit.prototype.constructor);     // true

    // インスタンス.constructorはnewされたときのコンストラクタ関数を指すのではなく、
    // newされたときのコンストラクタ関数のprototypeを指すことの確認
    // その２
    const Robot = function Robot(){
        this.name = "Robot";
    };

    // Rabbitのprototypeを変更した
    Rabbit.prototype = new Robot();
    // コンストラクタのconstructorはprototypeへの代入で割り当てられたオブジェクトと同じになる。
    console.log(Rabbit.prototype.constructor === Robot);    // true

    const robot = new Rabbit();

    // あとからprototypeを変更しても既存のインスタンスに影響しない
    console.log(rabbit.constructor === Rabbit); // true 
    // Rabbit.prototypeに割り当てられたオブジェクトと同じものを指しているのが確認できるので
    // newされたときのコンストラクタ関数のprototypeを指すことが確かに確認できた
    console.log(robot.constructor !== Rabbit && robot.constructor === Robot);  // true
```

#### `constructor`プロパティの取り扱い

先の通り、`constructor`はnewしたときのコンストラクタ関数ではなくてそのコンストラクタ関数のprototypeが指すオブジェクトと同じオブジェクトを参照する。

インスタンスの`constructor`を常にnewに続くコンストラクタ関数に固定させたいときは？



## `__proto__`は制御しなくていいのか？

プロトタイプは、ご存じの通り`[[Prototype]]`は`__proto__`で後出しでなんぼでも変更ができてしまう。

それを利用してプロトタイプを変更することをプロトタイプ汚染という。

これを防ぐ方法...調べてみたけどなんかない。

## JavaScriptにおける型判定


