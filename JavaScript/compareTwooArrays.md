# 2 つの配列を比較する

最強の回答：

https://stackoverflow.com/a/33034768/

今 2 つの配列 A, B が存在するとする。

## A 且つ B

A と B 両方に共通して存在する要素を取り出したいとき。

```JavaScript
const aAndB = arrA.filter(a => arrB.includes(a));
```

## A にのみ存在する

B には存在しない。

```JavaScript
const onlyA = arrA.filter(a => !arrB.includes(a));
```

## A と B に共通していない値

XOR

```JavaScript
let symDifference = arr1.filter(x => !arr2.includes(x))
                        .concat(arr2.filter(x => !arr1.includes(x)));
```

## ある配列の要素のいずれかが、他方の配列に存在した場合その要素を取り出したいとき

```TypeScript
const compareWith = ["apple","grape"];

const target1 = ["apple","banana","pineapple"];
const target2 = ["grape", "pineapple"];
const target3 = ["pineapple"];

const result1 = target1.some(cw => compareWith.includes(cw));
const result2 = target2.some(cw => compareWith.includes(cw));
const result3 = target3.some(cw => compareWith.includes(cw));
console.log(result1);   // true
console.log(result2);   // true
console.log(result3);   // false
```

`Array.prototype.some`:

少なくとも一つの要素がテスト関数をパスする場合 true を返す。
なので、`compareWith`配列の要素のうち、少なくとも一つはテスト関数を合格したことになる。

`Array.prototype.includes`:

`A === B`のような比較を行っている。

参考：

https://stackoverflow.com/a/39893636/22007575

https://stackoverflow.com/a/51603480/22007575
