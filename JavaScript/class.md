# Note about class

https://ja.javascript.info/class

https://ja.javascript.info/class-inheritance

コンストラクタ関数と比較などもする

ワークスペース： `~/workspaces/JavaScript/`

## 目的

class について理解して正しい使い方を身に着ける

## 目次

[class サマリ](#classサマリ)
[クラスフィールドでバインドされたメソッドを作成する](#クラスフィールドでバインドされたメソッドを作成する)
[super](#super)
[extends](#extends)
[オーバーライド](#オーバーライド)
[静的プロパティとメソッド](#静的プロパティとメソッド)
[継承とプロパティ](#継承とプロパティ)
[new](#new)
[delete](#delete)
[](#)
[正しい使い方の模索](#正しい使い方の模索)
[高度なトピック](#高度なトピック)

## class サマリ

```JavaScript
class MyClass {
  prop = value; // プロパティ

  constructor(...) { // コンストラクタ
    // ...
    // メソッドはclassのthisと結びついていないので
    this.method = this.method.bind(this);
  }

  method(...) {} // メソッド

  get something(...) {} // getter
  set something(...) {} // setter

  [Symbol.iterator]() {} // 計算された名前のメソッド (ここではシンボル)
  // ...
}

class ExtendedMyClass extedns MyClass {
  constructor(...) {
    super(...); // `this`を呼び出す前に必ず呼び出すこと
                // 基底クラスのコンストラクタが必要とする引数を渡すこと
  }
}
```

## class syntax と ES5 以前への変換

class は

```JavaScript
class User {
  constructor(name) { this.name = name; }
  sayHi() { alert(this.name); }
}

alert(typeof User); // function
```

constructor はまんまコンストラクタ関数であり、つまり以下と同じである

```JavaScript
const User = function(name) {
    this.name = name;
}
```

クラスメソッドも、内部的には prototype で User に追加されています。

```JavaScript
// 先のsayHi()は以下と同じ
const User = function(name) {
    this.name = name;
}

User.prototype.sayHi = function() {
    alert(this.name);
}
```

あとはインスタンスを作る方法も出来上がる結果も同じになります

では、

class とコンストラクタ関数とそのプロトタイプメソッドは全く同じで、

ただのシンタックスシュガーなのでしょうか？

いいえ。重要な違いがあります。

1. class で生成された関数は特別な内部プロパティ [[IsClassConstructor]]: true でラベル付けされています。

そのため手動で作成するのと全く同じではない

class 関数はそれ自体で呼び出すことはできない。

2. class は常に`use strict`である

3. class では巻き上げが起こらない

#### クラスフィールドでバインドされたメソッドを作成する

クラスメソッドは明示的に this を指定しないと、this はメソッドの所属インスタンスを指さない

```JavaScript
class Button {
  constructor(value) {
    this.value = value;
  }

  click() {
    alert(this.value);
  }
}

let button = new Button("hello");

setTimeout(button.click, 1000); // undefined
```

上記は button.click の this が setTimeout のコンテキストになった。

通常関数の this は呼出時に this を指定しないとグローバルか undefined になる。

これを解決する方法は２つ

1. コンストラクタで bind する

```JavaScript

class Button {
    constructor(value) {
        this.value = value;
        this.click = this.click.bind(this);
    }

    click() {
        console.log(this.value);
    }
}
```

2. アロー関数で書く

```JavaScript
class Button {
    constructor(value) {
        this.value = value;
    }

    click = () => {
        console.log(this.value);
    }
}
```

## super

super()は親コンストラクタを呼び出す特別な関数である。

継承 class が親コンストラクタや親メソッドを呼び出すときに用いる。

継承 class が親コンストラクタに必要な引数を渡すためにも用いる。

super()を呼び出す前に this を呼出してはならない。

アロー関数の中に super()は存在しない。もしもアクセスすると外部の関数から取得する。

```JavaScript
class Animal {
  // 基底classは引数nameが必要である
    constructor(name) {
        this.speed = 0;
        this.name = name;
    }

    run(speed) {
        this.speed += speed;
        console.log(`${this.name} runs with speed ${this.speed}.`);
    }
}

class Cat extends Animal {
  // 明示的にconstructorを呼び出すときは
  // super()を必ず呼び出すこと
  contructor(name, meow) {
    // thisを呼び出す前にsuper()を呼び出すこと
    super(name);
    this.meow = meow;
  }
}

class Rabbit extends Animal {
    // constructor自体を省略すると
    // 裏で自動的にsuper()を呼び出してくれている。

    running() {
      // 親クラスのメソッドをsuper経由で呼び出すことができる
      super.run(10);
    }

    stop() {
      // 親メソッドのstop()を呼出す
      super.stop();
      // setTimeoutのコールバックコンテキストでアロー関数を使っているので
      // その外部のstop()からsuper()を取得する
      // なので結果親クラスのstop()を呼び出す
      setTimeout(() => super.stop(), 1000);
      // 予期しない結果をもたらす
      setTimeout(function(){super.stop()}, 1000);
    }
}
```

上記の例でのポイントは super()はコンテキストに気を付けることである。

アロー関数と通常の関数の違いは this のコンテキストが異なるという点なので

super()の振る舞いが異なるのである。

## extends

extends を ES5 以前までの文法で書くにはどうしたらいいか？

なんでそんなことを知りたいのかというと、その仕組みを知っておいた方がよさそうだから（その程度）

ちょっと調べた限りだと、相当 JavaScript に精通していないとわかんなそう...

割愛。

#### extends で派生したクラスと親クラスの関係

鉄則：**派生クラスは自身のコンストラクタ内部で`this`を呼び出す前に`super()`を必ず呼び出さなくてはならない**

`extends`で派生したクラスは、コンストラクタを定義しないと一見うまくいく。

**仕様によると、クラスが別のクラスを拡張し、`constructor`を持たない場合、拡張クラスの方に暗黙にコンストラクタが生成される。**

でその暗黙コンストラクタの内部で super が呼び出されるから、`constructor`を定義しなくても正常に稼働するのである。

```JavaScript

  class Button {
      constructor(value) {
          this.value = value;
          this.click = this.click.bind(this);
      }

      click() {
          console.log(this.value);
      }
  }

  class closeButton extends Button {
    // constructorが定義されていない！
      push() {
          console.log(this);
          console.log(`${this.value} pushed!`);
      }
  }

  const closebtn = new closeButton('awesome close button');
  closebtn.click();  // awesoem close button
  closebtn.push();    // awesome close button pushed!
```

上記のように派生クラスに、親クラスで必要な文字列を渡しているけど、派生クラスで特にコンストラクタを定義しなくても

期待通りに動いている。

コンストラクタを定義するなら、this を呼び出す前に super()を呼び出すように気を付けよう。

## オーバーライド

#### コンストラクタ・オーバーライド

暗黙的なコンストラクタ:

明示的な継承 class 定義で constructor が記述されていなかった場合、

constructor は暗黙的にそのクラスに対して次のコンストラクタを定義する

```JavaScript
// 暗黙体なコンストラクタ
class Rabbit extends Animal {
  // 独自のコンストラクタを持たないクラスを拡張するために生成されます
  constructor(...args) {
    super(...args);
  }
}
```

明示的なコンストラクタ：

**明示的にコンストラクタを定義するときは、継承クラスは必ず`super()`を呼び出し、`this`を使う前にそれを行わなくてはならない。**

```JavaScript
class Animal {
  constructor(name) {
    this.speed = 0;
    this.name = name;
  }
  // ...
}

class Rabbit extends Animal {

  constructor(name, earLength) {
    // super()が呼び出されていないと...
    this.speed = 0;
    this.name = name;
    this.earLength = earLength;
  }

  // ...
}

// 動作しません!
let rabbit = new Rabbit("White Rabbit", 10); // Error: this は定義されていません

// なのでsuper()を、thisを使う前に呼び出そう
class Rabbit extends Animal {

  constructor(name, earLength) {
    // NOTE: 必ず呼び出そう
    super(name);
    this.speed = 0;
    this.name = name;
    this.earLength = earLength;
  }
  // ...
}
```

なぜ`super()`を呼ばないといけないのか？

派生コンストラクタ関数は特別な内部プロパティ`[[ConstructorKind]]: derived`が疲れられる。

このラベルは特別なもので、`new`の振る舞いに影響を与える。

-   派生コンストラクタが`new`で呼び出されるとき、親コンストラクタが空のオブジェクトを生成してそこに this を割り当てることを期待する。

だから親コンストラクタが、派生コンストラクタを new するときに必要になるので、super()で呼び出すのである。

そうしないと、this が生成できないからである。

#### メソッド・オーバーライド

https://ja.javascript.info/class-inheritance#ref-453

**`extends`したクラスは基底クラスのメソッドをオーバーライドできる**

```JavaScript

    class Animal {
        constructor(name) {
            this.speed = 0;
            this.name = name;
        }

        run(speed) {
            this.speed += speed;
            console.log(`${this.name} runs with speed ${this.speed}.`);
        }

        stop() {
            this.speed = 0;
            console.log(`${this.name} stopped.`);
        }
    }

    class Rabbit extends Animal {
        hide() {
            console.log(`${this.name} hides!`);
        }

        // Overrided
        stop() {
            super.stop(); // 親の stop 呼び出し
            this.hide(); // その後隠す
        }
    }

    let rabbit = new Rabbit('White Rabbit');

    rabbit.run(5); // White Rabbit runs with speed 5.
    rabbit.hide(); // White Rabbit hides!
```

## 静的プロパティとメソッド

結論：

-   静的メソッド、静的プロパティはそれが属するクラス（コンストラクタ）でのみアクセス可能で、そのインスタンスからはアクセスできない
-   静的メソッドの this はそのメソッドが属するクラス（コンストラクタ）である
-   静的プロパティ、静的メソッドは継承できる

使い方：

```JavaScript

    class User {
        static staticMethod() {
            alert(this === User);
        }
    }

    User.staticMethod(); // true

    // 以下は上記と同じこと

    class User {}

    User.staticMethod = function () {
        alert(this === User);
    };

    User.staticMethod(); // true
```

-   静的メソッドの`this`はそのメソッドが所属するコンストラクタを指す。

-   静的メソッドは、クラスには属するけれど、特定のオブジェクトには属さない関数を実装するのに使用される。

つまりインスタンスからはアクセスできないメソッドである。

使いどころ：

1. インスタンスの比較
2. ファクトリ・メソッド

```JavaScript
class Article {
  constructor(title, date) {
    this.title = title;
    this.date = date;
  }

  // Instanceの比較
  static compare(articleA, articleB) {
    return articleA.date - articleB.date;
  }
  // Factory Method
  static createTodays() {
    // this equals Article
    return new this("Today's digest", new Date());
  }
}

let article = Article.createTodays();

alert( article.title ); // Todays digest

// usage
let articles = [
  new Article("Mind", new Date(2019, 1, 1)),
  new Article("Body", new Date(2019, 0, 1)),
  new Article("JavaScript", new Date(2019, 11, 1))
];

articles.sort(Article.compare);

alert( articles[0].title ); // Body
```

createTodays()も compare()も Article インスタンスのメソッドではなくてクラスにのみ属する関数である。

#### 静的プロパティ

```JavaScript

    class Article {
        static publisher = 'Ilya Kantor';
    }

    console.log(Article.publisher); // Ilya Kantor

    // 直接コンストラクタへ代入するのと同じこと
    Article.publisher = "Ilya Kantor";

    const article = new Article();
    // インスタンスを作って中身を確認しても
    // 静的プロパティなのでpublisherは存在していない
    console.log(article);   // Article{}


    // 当然全く同じ名前のプロパティを追加できる
    article.publisher = "Mr. Donovan";
    console.log(article);   // Aritcle{ publisher: "Mr. Donovan"}

    // 静的メソッドと上記の通り追加したメソッドは別物なので
    console.log(Article.publisher);  // Ilya Kantor のままである
```

#### 静的メソッドとプロパティの継承

https://ja.javascript.info/static-properties-methods#statics-and-inheritance

**静的メソッドとプロパティは、派生クラスに継承される**

確認：

````JavaScript
class Animal {
  static planet = "Earth";

  constructor(name, speed) {
    this.speed = speed;
    this.name = name;
  }

  run(speed = 0) {
    this.speed += speed;
    alert(`${this.name} runs with speed ${this.speed}.`);
  }

  static compare(animalA, animalB) {
    return animalA.speed - animalB.speed;
  }

}

// Inherit from Animal
class Rabbit extends Animal {
  hide() {
    alert(`${this.name} hides!`);
  }
}

let rabbits = [
  new Rabbit("White Rabbit", 10),
  new Rabbit("Black Rabbit", 5)
];

rabbits.sort(Rabbit.compare);

rabbits[0].run(); // Black Rabbit runs with speed 5.

alert(Rabbit.planet); // Earth```JavaScript
````

派生クラス Rabbit は、親クラス Animal の静的メソッド`compare()`とプロパティ`planet`を継承しているのが確認できる。

これの仕組みを考える。

`rabbits[0]`の`[[Prototype]]`は`Rabbit`である。
new Rabbit()で生成されたから、Rabbbit.prototype の値が割り当てられるから。

`Rabbit`の`[[Prototype]]`は`Animal`である。
Rabbit は Animal から継承しているからである。

だから Animal と Rabbit の両方の参照を持つ。

だから両方の静的メソッドを参照できる。

## new

https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/new

https://ja.javascript.info/constructor-new

`new`演算子は次のことを行います

1. 空のプレーンな JavaScript オブジェクトを生成します
2. このオブジェクトに`__proto__`を追加し、コンストラクタ関数の`prototype`オブジェクトを結びつける
3. 新しく生成されたオブジェクトインスタンスを`this`コンテキストとして結びつける
4. 関数がオブジェクトを返さない場合は`this`を返す

つまり、

```JavaScript
const Animal = function() {};

const animal = new Animal();

// --- this `new` process is doing like this  ---

function _new(_prototype) {
  let _animal = {};
  _animal.__proto__ = _prototype.prototype;
  _animal.bind(_prototype);  // これ正しいのかはさておき
  return animal;
}
```

コンストラクタ関数でないといけないというわけではない。

コンストラクタ関数のイニシャルが大文字なのは慣習である。

```JavaScript

    const asshole = function() {
        let color = "red";
        this.sensitive = true;
    }

    const ass = new asshole();

    console.log(ass); // {sensitive: true}
    console.log(ass.color); // undefined
```

ただし`this`で指定していない変数は new されても無視される。

理由は、new は**proto**に prototype を割り当てるだけでコピーを作るわけでないから。

this に登録されていないのならば無視されるのである。

つまり単純にスコープの問題になるので color なんて存在しないのである。

## delete

インスタンスのプロパティを削除する。

**インスタンス側のメンバーの追加や削除はプロトタイプのオブジェクトに影響を与えない**

```JavaScript
const Member = function(){};

Member.prototype.sex = "Male";

const M1 = new Member();
const M2 = new Member();

console.log(M1.sex + ',' + M2.sex); // Male,Male ..1

M2.sex = "Female";

console.log(M1.sex + ',' + M2.sex); // Male,Female ..2

delete M1.sex;
delete M2.sex;

console.log(M1.sex + ',' + M2.sex); // Male,Male ..3


```

これの説明は簡単である。

1. プロトタイプチェーンで Member のフィールドを参照しているだけ

2. M2.sex は M2 インスタンスだけのフィールドでそれを参照しているだけ

3. delete はプロトタイプに割り当てられているフィールドを削除できないだけ

つまり 3 の時は 1 の時同様ﾌﾟﾛﾄﾀｲﾌﾟﾁｪｰﾝで参照しているだけ

また、

`M2.sex = "Female`としてもプロトタイプのフィールドが変化するわけではないこともわかる。

## お作法

#### オブジェクトリテラルを使って prototype 追加分を節約

以下のような一つずつ prototype を追加していくやり方はデメリットが多い。

-   prototype 定義がたくさんできて可読性下がる

-   コンストラクタ関数名が変更になったら変更箇所多い

とか

```JavaScript
const Member = function(name, age) {
  this.name = name;
  this.age = age;
}

Member.prototype.sayName = function() {
    return this.name;
}

Member.prototype.sayAge = function() {
  return this.age;
}
```

上記は`prototype.`が増えて可読性下がるとのことなので、オブジェクトリテラルを使う

```JavaScript
const Member = function(name, age) {
  this.name = name;
  this.age = age;
}

Member.prototype = {
  sayName: function() {
    return this.name;
  };

  sayAge: function() {
    return this.age;
  }
}
```

これなら一つのブロックに prototype 追加されたのが集約されているのでわかりやすい。

#### 静的プロパティは読み取り専用の値にすること

静的プロパティのコンテキストはその定義されているコンストラクタ関数である。

インスタンスではないので、

静的プロパティが後から変更されるとすべてのコンストラクタ関数の継承クラスが影響を受ける。

## 正しい使い方の模索

## 高度なトピック

各割愛。忘れるから。

#### クラス・フィールドのオーバーライド

#### Super: Internals, [[HomeObject]]
