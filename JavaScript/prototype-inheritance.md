# Prototype & Inheritance

継承とプロトタイプについて

## まとめ

- `[[Prototype]]`はJavaScriptオブジェクトすべてが持つ特別なプロパティである
- `[[Prototype]]`は通常隠されているプロパティでアクセスする手段がないが、`__proto__`はそれを暴露するので通常、使用は非推奨である。
- `__proto__`は`[[Prototype]]`のgetter/setterである
- `__proto__`に代入することでオブジェクトの`[[Prototype]]`を変更することは可能なので非常に危険である
- `prototype`というプロパティはコンストラクタ関数のみがアクセスできるプロパティで、通常のオブジェクトもインスタンスもアクセスできない
- `F.prototype`は`new F()`されるときに、生成されるインスタンスの`[[Prototype]]`へ`F.prototype`の値を割り当てるのに使用されるだけである
- インスタンスを作ってから`F.prototype`を変更しても既存のインスタンスの`[[Prototype]]`はそれに伴って変更されることはなく固定されたままである
- プロトタイプチェーンは`[[Prototype]]`をたどることで親オブジェクトのプロパティにアクセスできる

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

コンストラクタと`F.prototype`：

-   `F.prototype`というプロパティは、コンストラクタ関数のみがアクセスできるプロパティである

-   通常のオブジェクト、インスタンスオブジェクトも`prototype`というプロパティはアクセスできない

-   `F.prototype`は`new F()`するときしか使われない

`f.__proto__`は`new F()`するときに固定されて後から`F.prototype`を変更しても既存のインスタンス`f.__proto__`は変更されないという意味

-   `F.prototype`は`[[Prototype]]`と同じではない。`new F()`されたときのみ`[[Prototype]]`へ新しいオブジェクトを割り当てる

とにかく`new F()`するときだけしか出番がないのである

-   `F.prototype`の値は null かオブジェクトでなくてはならない

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

インスタンス自体はただのオブジェクトなので、当然 f.**proto**で直接変更できるけど、

その影響はそのオブジェクトだけであり、すべての同一コンストラクタからなる既存インスタンスは影響を受けない。

- `F.prototype`は、`F`が継承していないコンストラクタなら自身を指す

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
