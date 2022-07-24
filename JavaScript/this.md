# Note about `this`

`this`が何者か説明できるようになる。

## 目次

[グローバル・コンテキスト](#グローバル・コンテキスト)
[関数コンテキスト](#関数コンテキスト)
[class コンテキスト](#classコンテキスト)
[object コンテキスト](#objectコンテキスト)

## 参考

https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/this#%E9%96%A2%E9%80%A3%E6%83%85%E5%A0%B1

https://dmitripavlutin.com/gentle-explanation-of-this-in-javascript/

## `this`基礎

> ほとんどの場合、this の値はどのように関数が呼ばれたかによって決定されます (実行時結合)。これは実行時に代入によって設定することはできず、関数が呼び出されるたびに異なる可能性があります。ES5 では bind() メソッドが導入され、関数がどのように呼ばれたかに関係なく `this` の値を設定するすることができるようになり、ES2015 では、自身では this の結び付けを行わないアロー関数が導入されました (これは包含する構文上のコンテキストの this の値を保持します)。

要は、いろんな場面で this が指すものは異なるのであるので、this の振舞は場面場面で覚える必要がある。

具体的にはコンテキストと呼び出され方によって異なる。

## グローバル・コンテキスト

#### strict モードでないとき

グローバルコンテキスト：グローバル・オブジェクトを指す。

関数コンテキスト： 関数の呼び出し方によって異なる。

-   this を指定しないと this はグローバルオブジェクトを指す

```JavaScript

// Window
console.log(this);
function hoge() {
    // Window
    console.log(this);
}
hoge();
```

#### strict モードの時

グローバルで呼び出されたならばグローバルオブジェクトを。

関数コンテキスト：this を指定しないと`undefined`になる

```JavaScript

    'use strict';
    // Window
    console.log(this);
    function hoge() {
        'use strict';
        // undefined
        console.log(this);
    }

    hoge();
```

## 関数コンテキスト

呼び出し時に this を指定しないと基本的に以下のようになる。

-   関数内部が strict モードでないとき：グローバルオブジェクトを指す
-   関数内部が strict モーでのとき；undefined となる

関数呼び出し時に this を指定する方法は以下。

-   Funciton.prototype.call()
-   Funciton.prototype.apply()
-   Funciton.prototype.bind()

#### Function.call()

https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Function/call

> call() はあるオブジェクトに所属する関数やメソッドを、別なオブジェクトに割り当てて呼び出すことができます。

> call() は関数やメソッドに this の新しい値を提供します。 call() によって、いったんメソッドを書いてから、新しいオブジェクトへメソッドを描き直さずに他のオブジェクトへと継承することができます。

ということで、

`call()`は関数の呼び出し時に call()で別のオブジェクトを指定すると、

その呼び出された関数の this がそのオブジェクトを指すようにできる代物である。

つまり、

自分のものでないものをあたかも自分のもののように呼び出すことができる。

```TypeScript

interface Function {
    /**
     * Calls a method of an object, substituting another object for the current object.
     * @param thisArg The object to be used as the current object.
     * @param argArray A list of arguments to be passed to the method.
     */
    call(this: Function, thisArg: any, ...argArray: any[]): any;
}
// NOTE: 第一引数のthisはFunctionオブジェクトのことで、通常使用する場合無視する
// 例：
// 呼び出したいオブジェクト.call(呼び出したい側のthis)
// 呼び出したい関数.call(その関数のthisとして指定したいオブジェクト)
```

ということで、

呼び出したい関数.call(その関数における this として指定したいオブジェクト)

.call 付きで関数を呼び出すときに、その関数に渡したい引数は

.call の第二引数以降にその引数を渡す。

```JavaScript

    'use strict';
    function Product(name, price) {
        this.name = name;
        this.price = price;
    }

    function Food(name, price) {
        // ProductをFoodに所属させた
        Product.call(this, name, price);
        this.category = 'food';
    }

    // Foodには存在しない`name`だけど、ProductをFoodのプロパティにしたからアクセスできるようになった
    console.log(new Food('cheese', 5).name); // "cheese"
    console.log(new Food('cheese', 5));

    // callを使用した、所属のない関数と、メソッドを持たないオブジェクトを組み合わせて呼び出す方法
    function greet() {
        const reply = [
            this.animal,
            'typically sleep between',
            this.sleepDuration,
        ].join(' ');
        console.log(reply);
    }

    const obj = {
        animal: 'cats',
        sleepDuration: '12 and 16 hours',
    };

    greet.call(obj); // cats typically sleep between 12 and 16 hours
```

#### Function.apply()

```TypeScript
interface Function {
    /**
     * Calls the function, substituting the specified object for the this value of the function, and the specified array for the arguments of the function.
     * @param thisArg The object to be used as the this object.
     * @param argArray A set of arguments to be passed to the function.
     */
    apply(this: Function, thisArg: any, argArray?: any): any;
}
```

Function.call()との違い:

-   call()は連続した引数のリストを受け取るのに対して、apply()は引数の配列を一つだけ受け取れるという点

ということで、Function.call()との違いは、

呼び出したい関数へ渡す引数は配列型の変数一つだけということ。

ではこの配列がどのように呼び出したい関数へ渡されるのか。

1. 配列を別の配列に追加する

```JavaScript
    let arr = [1, 2, 3];
    let numbers = [4, 5, 6];

    arr.push.apply(arr, numbers);

    console.log(arr); // 1, 2, 3, 4, 5, 6
```

通常、配列 1.push(配列 2)だと、配列 2 の中身すべてが配列 1 の一つの要素として末尾に追加される。

そうではなくて配列１のつづきとして配列２の中身を追加したいときに、

上記のようにできる。

NOTE: 既存の配列に追加する。

新規の配列を返してほしいときは concat を使う。

他、いろいろ利用例があるけれどぶっちゃけ覚えるのは大変。

コードレビューするときに仕様と意図を読み取るようにした方がいいかも。

#### まとめ

-   **基本的に「関数を呼び出したときに、this を『設定』しないと this はグローバル変数か undefined になる」**

関数の this は関数呼出時に指定すればその通りに、指定しなければ、strict モードでなければグローバルコンテキストの this と同じになり、strict モードなら undefined になる

## class コンテキスト

https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/this#%E3%82%AF%E3%83%A9%E3%82%B9%E3%82%B3%E3%83%B3%E3%83%86%E3%82%AD%E3%82%B9%E3%83%88

コンストラクタ関数含む。

関数コンテキストとの違い...

-   class における this はオブジェクトである

関数ではグローバルオブジェクトか undefined か call, apply, bind で指定した対象である

-   class のメソッドは this のプロトタイプに追加される

静的メソッドは this に追加されない。クラス自身のプロパティとなるからクラスメソッドとして使えないのである。

-   class メソッドだからと言って、メソッドの this は常に所属のインスタンスを指すわけではない

基本的にコンストラクタで bind するか、アロー関数で定義すること

```JavaScript

    const jonas = {
        name: 'Jonas',
        year: 1989,
        calcAge: function () {
            return 2021 - this.year;
        },
        whoIsThis: function () {
            return this;
        },
    };

    console.log(jonas.calcAge()); // 32
    console.log(jonas.whoIsThis()); // オブジェクトjonasを指した

    // 試しに後出しでf1()をjonasのメソッドとしたらどうなるか
    function f1() {
        return this;
    }
    jonas.whoIAm = f1;
    console.log(jonas.whoIAm()); // オブジェクトjonasを指した

    // クラス
    // thisはPersonオブジェクトを指す
    //
    var Person = function (name, age) {
        this.name = name;
        this.age = age;
        this.self = this;
    };

    Person.prototype.whoIsThis = function () {
        return this;
    };

    var Mike = new Person('Mike', 28);
    console.log(Mike.whoIsThis());  // Mikeのインスタンス
    console.log(Mike.self); // Mikeのインスタンス
```

#### クラスメソッドは this を指定しなくてはならない

クラスメソッドは明示的に this を指定しないと、this はメソッドの所属インスタンスを指さない

なので、クラスメソッドは必ずアロー関数で書くか、コンストラクタで bind しろ。

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

## Object コンテキスト
