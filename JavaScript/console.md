# console あれこれ

## `console.dir`

https://developer.mozilla.org/en-US/docs/Web/API/console/dir_static

ネストされたオブジェクトのプロパティが省略されずに出力してくれる。

> In other words, console.dir() is the way to see all the properties of a specified JavaScript object in console by which the developer can easily get the properties of the object.

```JavaScript
const menu = {
  main: {
    currey: 500,
    udon: 450,
    rice: 150
  },
  side: {
    soup: {
      misoshiru: {
        wakame: 250,
        asari: 250
      },
      potage: {
        corn: 250,
        potato: 250
      }
    },
    dessert: {
      icecream: {
        vanilla: 100,
        chocolate: 100
      }
    }
  }
}

console.log(menu)
/*
{
  main: { currey: 500, udon: 450, rice: 150 },
  side: {
    soup: { misoshiru: [Object], potage: [Object] },
    dessert: { icecream: [Object] }
  }
}
*/

console.dir(menu, {depth: null})
/*
{
  main: { currey: 500, udon: 450, rice: 150 },
  side: {
    soup: {
      misoshiru: { wakame: 250, asari: 250 },
      potage: { corn: 250, potato: 250 }
    },
    dessert: { icecream: { vanilla: 100, chocolate: 100 } }
  }
}
*/

```

node.js だと`utils.inspect`という出力方法がある模様。
