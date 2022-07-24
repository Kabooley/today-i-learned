# Note about class

https://ja.javascript.info/class

https://ja.javascript.info/class-inheritance

コンストラクタ関数と比較などもする

#### class サマリ

```JavaScript
class MyClass {
  prop = value; // プロパティ

  constructor(...) { // コンストラクタ
    // ...
  }

  method(...) {} // メソッド

  get something(...) {} // getter
  set something(...) {} // setter

  [Symbol.iterator]() {} // 計算された名前のメソッド (ここではシンボル)
  // ...
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
