# React `ref`の教訓

## Summary

-   [ref を参照していい（いけない）タイミング](#ref-を参照していい（いけない）タイミング)
-   [ref を使う上で知っておくこと](#ref-を使う上で知っておくこと)
-   [`useEffect`と`ref`の正しい使い方](#`useEffect`と`ref`の正しい使い方)
-   [`ref`活用場面](#`ref`活用場面)
-   [[編集中]監視する DOM を変更したいとき](#監視する-DOM-を変更したいとき)
-   [[この記事が必要か要検討]DOM のリサイズを監視したいときは？](#DOM-のリサイズを監視したいときは？)
-   [`useEffect`の依存関係に`ref.current`だけ渡すのは危険](#`useEffect`の依存関係に`ref.current`だけ渡すのは危険)
-   [[編集中]各 ref の特徴を箇条書き](#各-ref-の特徴を箇条書き)
-   [使用例](#使用例)
-   [子または孫以下のコンポーネントへ ref を渡す方法あれこれ](#子または孫以下のコンポーネントへ-ref-を渡す方法あれこれ)
-   [親コンポーネントからの ref 、かつ自コンポーネントの ref 両方を一つの DOM に渡したいとき](#親コンポーネントからの-ref-、かつ自コンポーネントの-ref-両方を一つのDOMに渡したいとき)
-   [動的配列の各要素にひとつの ref を渡したいとき](#動的配列の各要素にひとつのrefを渡したいとき)

## ref を参照していい（いけない）タイミング

参考: https://react.dev/learn/manipulating-the-dom-with-refs#when-react-attaches-the-refs

**`ref`は DOM を参照している場合 rendering 後でのみ参照すること。**

レンダリング中は`ref`を参照してはならない。

以下説明：

React のレンダリングの流れは以下のとおりである。

1. レンダリングのトリガーの発生
2. コンポーネントのレンダリング
3. レンダリング内容の DOM への反映

(https://react.dev/learn/render-and-commit)

ここで、「2. コンポーネントのレンダリング」のレンダリングの意味は、React コンポーネントの再呼出のことであると書かれている。

> After you trigger a render, React calls your components to figure out what to display on screen. “Rendering” is React calling your components.

なので、コンポーネントの再呼出（再計算）によって導き出された戻り値を、

DOM に反映させるのが「3. レンダリング内容の DOM への反映」である。

このとき変更の反映対象となる DOM は前回と違いが発生する DOM のみである。

ということで（当然であるが）DOM は毎レンダリング変化する可能性がある。

そして、この毎度のレンダリングにまたがって値を保持してくれるのが`ref`である。

DOM は毎レンダリングで変更される可能性があるが、一方で`ref`は毎レンダリングにまたがって値を保持する機能である。

`ref`を DOM を参照するように使っているとき、`ref`は変更がコミットされた DOM を参照している。

`ref`は本来変更される前の値を保持し続けるはずであるが、いつ更新された DOM を参照しなおすのか？

DOM にアタッチされている`ref`がレンダリングプロセスの間どうなっているのかの状況が次の通り：

1.  初期レンダリング時：DOM はまだ作成されていないから ref は null。
2.  毎レンダリング時: ref は更新されない。
3.  コミット時（変更を DOM に適用するとき）: ref は DOM にアタッチされる(ref.current へ DOM が渡される)

(https://react.dev/learn/manipulating-the-dom-with-refs#when-react-attaches-the-refs)

ということで、`ref`は DOM の参照として渡されると、レンダリング中はいったん null にされて、コミット段階の DOM への変更の提要が終わってから対象の DOM が改めて渡されるようである。

つまり、

**レンダリング中に`ref`を読み取ったり書き込んだりしてはならない。**

上述の通りレンダリング中は変更がまだ適用されていないからである。DOM を参照する使い方に限らず、`ref`はレンダリング中に参照してはならない。

`ref`は React の理の外にあるものを指すためのエスケープ・ハッチである。

ということで、`ref`は副作用かイベントハンドラのタイミングでのみ使うべきである。

https://react.dev/reference/react/useRef#caveats

https://react.dev/learn/synchronizing-with-effects#step-1-declare-an-effect

## ref を使う上で知っておくこと

https://react.dev/reference/react/useRef#caveats

https://react.dev/learn/referencing-values-with-refs#differences-between-refs-and-state

-   ref は、明示的に変更しない限りレンダリングにまたがって値を記憶する。
-   ref.current は、state と異なり変更されても再レンダリングをトリガーしない（React は関知しない）
-   `ref.current`の値をレンダリング中に読み取ってはならない。レンダリング中に ref はまだ更新されていない。
-   `useEffect(, [ref.current])`はご法度である。理由は ref.current の変更は再レンダリングをトリガーせず、予期しないタイミングで`useEffect(, [ref.current])`が実行される可能性があるからである。

`useEffect()`はそのうちに使う変数などは依存関係に含めないといけないが、`ref.current`は例外といえるようで、`ref.current`を依存関係に含めなくても使うことができる。
(https://react.dev/learn/synchronizing-with-effects#why-was-the-ref-omitted-from-the-dependency-array)

## `useEffect`の依存関係に`ref.current`だけ渡すのは危険

`ref`参照対象の変化を監視したいときに考え付く方法とは以下のものであると思う。

`React.useRef`を使って ある対象を参照しておき、

`ref.current`に変更があったら`useEffect(, [ref.current])`で変更を検知しようという考えである。

しかしそれは失敗する。

理由は`ref.current`の変更は再レンダリングを引き起こさないから。

もし`useEffect`の依存関係に含めたとしても、`ref.current`の変更によって`useEffect`が呼び出されず更新のトリガーになりえないからである。

これをやってしまうと、別のトリガーで引き起こされた再レンダリング後のタイミングで`useEffect(,[ref.current])`が実行されてしまう。

#### 例

任意の button をクリックするとその button の番号を ref に渡し、その変更を取得しようとして失敗しているコード

```TypeScript
import React, { useRef, useEffect, useState } from "react";

const DontPassUnstaticRefAsDependency = () => {
  const refButtonId = useRef<number>(0);

  useEffect(
    () => {
      console.log(`ref.current has been changed to ${refButtonId.current}`);
    },
    [refButtonId.current]
  );

  const handleClick = (num: number) => {
    console.log(`Clicked ${num}`);
    refButtonId.current = num;
  };

  console.log("rendering");

  return (
    <div style={{ position: "relative" }}>
      <button onClick={() => handleClick(1)}>Pass 1 to ref</button>
      <button onClick={() => handleClick(2)}>Pass 2 to ref</button>
      <button onClick={() => handleClick(3)}>Pass 3 to ref</button>
      <button onClick={() => handleClick(4)}>Pass 4 to ref</button>
    </div>
  );
};
```

`refButtonId.current`が`handleClick`で変更されても`useEffect(,[refButtonId.current])`は実行されない。

`ref`の参照対象の変更は再レンダリングをトリガーしないからである。

そりゃそうじゃんという話ですが、

このコードの困る点は、ほかのトリガーによる再レンダリング発生時に`refButtonId.current`の値のみを取得することになる点である。

常に button クリックによる変更を知らなきゃならないという状況の時は、そのトリガーが発生するまでに行ったすべての`refButtonId.current`の変更は失われるのである。

## `useEffect`と`ref`の効果的な使い方

先の問題の解決策を探る。

https://react.dev/learn/synchronizing-with-effects#why-was-the-ref-omitted-from-the-dependency-array

`useEffect`の使い方として、その内部で参照する値はすべて依存関係に含めなくてはならないというルールがあるが、

依存関係に含めるべき変数が「常に安定した値」を返す場合は省略していいようだ。

> This is because the ref object has a stable identity: React guarantees you’ll always get the same object from the same useRef call on every render. It never changes, so it will never by itself cause the Effect to re-run. Therefore, it does not matter whether you include it or not. Including it is fine too:

となると、

`ref.current`がずっと変化しなければ依存関係に含める必要がなく、変化するようなことがあれば含めなくてはならない。

解決策を 2 つ考えた。

#### 解決策１：`ref.current`の変更時に再レンダリングをトリガーさせる

`ref.current`の参照対象を変更するようなことがある場合は`useEffect`の依存関係に`ref.current`に依存関係を含めなくてはならないというのがルールとのことなので、`useEffect`の扱いはそうしないといけないとして、

とはいえ`ref.current`の変更は再レンダリングをトリガーしないのは事実である。

そこで、`ref.current`の変更対象が変更されたら再レンダリングをトリガーするようにすればよいのである。

ref.current の変更と setState の実行を必ずセットにするのである。

```TypeScript

const DontPassUnstaticRefAsDependency = () => {
  const refButtonId = useRef<number>(0);
  const [currentNum, setCurrentNum] = useState<number>(0);

  useEffect(() => console.log("did uodate"));

  useEffect(
    () => {
      console.log(
        `refButtonId.current has been updated: ${refButtonId.current}`
      );
      console.log(`currentNum: ${currentNum}`);
    },
    // ref.currentの変更とsetStateをセットにすることを前提にすれば
    // ref.current単体を依存関係に含めても問題ない
    [refButtonId.current]
  );

  const handleClick = (num: number) => {
    console.log(`Clicked ${num}`);
    // ref.currentの変更とsetStateをセットにする
    refButtonId.current = num;
    setCurrentNum(num);
  };

  console.log("rendering");

  return (
    <div style={{ position: "relative" }}>
      <button onClick={() => handleClick(1)}>Pass 1 to ref</button>
      <button onClick={() => handleClick(2)}>Pass 2 to ref</button>
      <button onClick={() => handleClick(3)}>Pass 3 to ref</button>
      <button onClick={() => handleClick(4)}>Pass 4 to ref</button>
        force rerender
      </button>
    </div>
  );
};

```

結果：

```bash
# StrictModeです
# mount時
rendering
did update
refButtonId.current has been updated: 0
currentNum: 0
did update
refButtonId.current has been updated: 0
currentNum: 0
# button 2をクリックした
Clicked 2     # setCurrentNum(2)して
rendering     # 再レンダリングが発生し
did update    # useEffect(,[refButtonId.current])が実行される
refButtonId.current has been updated: 2
currentNum: 2
# button 3をクリックした
# 以下同様
Clicked 3
rendering
did update
refButtonId.current has been updated: 3
currentNum: 3
```

今回の例は`button id`を state 管理すればいいだけでは？という拙い例であるが、

`ref`の参照が変化したことを検知したい状況に対しては使える例と考える。

#### 解決策 2：`Callback ref`を使う

https://legacy.reactjs.org/docs/refs-and-the-dom.html#callback-refs

> `ref`は`componentDidMount`または`compoenntDidUpdate`が発火する前に更新されることが保証されます。

なので再レンダリングが発生したら必ず`Callback ref`が実行されるので、`useEffect`が必要ない。

一方`useEffect`を使う場合と異なり、必ずレンダリング時毎回`Callback ref`が呼び出され制御できないというデメリットがある。

解決策１と２どちらをとるか参照という手段をあきらめるかは状況によると思うので

天秤にかけて選択する。

## `ref`活用場面

#### 例：親コンポーネントから ref を取得し、なおかつ自コンポーネントも useRef()で ref を使いたいとき

親コンポーネントは、常に最新の自コンポーネントの`DOMRect`と`Element.scrollWidth`情報を必要としているとする。

一方自コンポーネントは`useRef`を使っている。

親コンポーネントも自コンポーネントもどちらも同じ DOM を参照させなくてはならない。

そんなとき。

```TypeScript
interface iProps {
  // 親コンポーネントからのref
  _ref: React.RefObject<HTMLDivElement>;
  // div.tabの数
  numberOfTabs: number;
}

const Tabs = ({_ref, numberOfTabs}: iProps) => {
  const [selected, setSelected] = useState<number>(1);
  // 自コンポーネントが使っているref
  const _refTabArea = useRef<HTMLDivElement>(null);
  // div.tabの各要素を参照するrefの配列
  const _refTabs = useRef(
    Array.from({ length: numberOfTabs }, (_, i) => i + 1).map(() =>
      React.createRef<HTMLDivElement>()
    )
  );

  // _refTabAreaはどのタブがクリックされたのか調べるときに使う
  const changeTab = (selectedTabNode: HTMLDivElement) => {
    // 一旦すべてのtabのclassNameを'tab'にする
    for (var i = 0; i < _refTabArea.current!.childNodes.length; i++) {
      var child: iJSXNode = _refTabArea.current!.childNodes[i];
      if (/tab/.test(child.className!)) {
        child.className = "tab";
      }
    }
    // 選択されたtabのみclassName='tab active'にする
    selectedTabNode.className = "tab active";
  };

  return (
    <div
      className="tabs-area"
      ref={_refTabArea}
      // 両方渡す方法は...
      // ref={_ref}
      style={stylesOfTabsArea}
    >
      {Array.from({ length: numberOfTabs }, (_, i) => i + 1).map((i, index) => (
        <div
          className={index === selected ? "tab active" : "tab"}
          key={index}
          style={stylesOfTab}
          ref={_refTabs.current[index]}
          onClick={() =>
              changeTab(_refTabs.current[index].current!)
          }
        >
          <span>tab {i}</span>
        </div>
      ))}
    </div>
  );
};
```

## アプローチ１：`useImperativeHandle`フックを使う方法

https://react.dev/reference/react/useImperativeHandle

`useImperativeHandle`は、親コンポーネントから渡された ref に対して、DOM を渡す代わりに自コンポーネントのスコープを持つ関数を渡す代物である。

メリット：

-   親コンポーネントからの ref を DOM に渡す必要がないので子コンポーネントは自分の ref を使うことができる
-   ref は props 経由で渡された ref でもいいので、孫コンポーネント以下へ渡すことも可能
-   親コンポーネントは ref の呼び出しを任意のタイミングにできる

デメリット：

-   子コンポーネントは親コンポーネントからの要求を知らなくてはならない

```TypeScript

interface iProps {
  // useImperativeHandleのコールバックの型に合わせる
  _ref: React.RefObject<{
    getTabsAreaRect: () => DOMRect | undefined;
    getScrollWidth: () => number;
  }>;
  numberOfTabs: number;
}

const Tabs = ({ _ref, numberOfTabs }: iProps) => {
  const [selected, setSelected] = useState<number>(1);
  const _refTabArea = useRef<HTMLDivElement>(null);
  const _refTabs = useRef(
    Array.from({ length: numberOfTabs }, (_, i) => i + 1).map(() =>
      React.createRef<HTMLDivElement>()
    )
  );

  // 親コンポーネントから受け取ったrefはここに渡す
  useImperativeHandle(
    _ref,
    () => {
      return {
        getTabsAreaRect() {
          if (_refTabArea.current) {
            return _refTabArea.current.getBoundingClientRect();
          } else return undefined;
        },
        getScrollWidth() {
          if (_refTabArea.current) {
            return _refTabArea.current.scrollWidth;
          } else return undefined;
        }
      };
    },
    []
  );

  const changeTab = (selectedTabNode: HTMLDivElement, index: number) => {
    // ...
  };

  return (
    <div className="tabs-area"
      // 自コンポーネントのrefを維持できる
      ref={_refTabArea}
    >
      {Array.from({ length: numberOfTabs }, (_, i) => i + 1).map((i, index) => (
        // ...
      ))}
    </div>
  );
};
```

今回のコードはリアクティブな値である`refTabArea`を依存関係に含めていない。

理由は先の方で述べた通り、ref が安定して同じ値を指し続ける場合は省略可能であるため。

## アプローチ２：`Callback ref`を使う方法

https://stackoverflow.com/a/72519722/22007575

https://legacy.reactjs.org/docs/refs-and-the-dom.html#callback-refs

つまり、親からの ref と自身の ref を callback ref のコールバック関数内で呼び出すことで両方に DOM を渡すのである

やること：

-   ref を渡したい対象の ref には`Callback ref`を渡す
-   渡せる ref の型は undefined を受け入れるようにする。`React.MutableRefObject<HTMLDivElement | undefined>`

`Callback ref`は毎レンダリング時に必ず呼び出されるので、`ref.current`は毎レンダリング時に更新されることになるが

結局ずっと同じ対象を参照するので`useEffect(,[ref.current])`しなければ無駄な処理は起こらない。

```TypeScript

interface iProps {
  _ref: React.MutableRefObject<HTMLDivElement | undefined>;
  numberOfTabs: number;
}

const Tabs = ({ _ref, numberOfTabs }: iProps) => {
  const [selected, setSelected] = useState<number>(1);
  // undefinedを受け入れさせる　且つ nullを渡さない
  const _refTabArea = useRef<HTMLDivElement | undefined>();
  const _refTabs = useRef<HTMLDivElement[]>([]);

  // ...

  return (
    <div
      className="tabs-area"
      // Callback refを渡す
      ref={(node: HTMLDivElement) => {
        // callback内でnodeを渡す
        _refTabArea.current = node;
        _ref.current = node;
      }}
      style={stylesOfTabsArea}
    >
      {Array.from({ length: numberOfTabs }, (_, i) => i + 1).map((i, index) => (
        <div
          className={index === selected ? "tab active" : "tab"}
          key={index}
          style={stylesOfTab}
          ref={(node: HTMLDivElement) => (_refTabs.current[index] = node)}
          onClick={() => changeTab(_refTabs.current[index], index)}
        >
          <span>tab {i}</span>
        </div>
      ))}
    </div>
  );
};

export default Tabs;

```

`React.RefObject<HTMLDivElement>`にすると`ref.current`は読み取り専用だからできませんエラーが出る。

## 子または孫以下のコンポーネントへ ref を渡す方法

https://react.dev/reference/react/forwardRef#forwarding-a-ref-through-multiple-components

公式曰く、

-   直接の子コンポーネントなら`forwardRef`
-   孫以下なら別に ref を props 経由でバケツリレーしてかまわない


## 一つのコンポーネントに複数 ref を渡したいとき

こんなとき（そんなときがあるのかはさておき）：

-   親コンポーネントから ref を取得し、なおかつ自コンポーネントも ref を使いたいとき。
-   DOM を操作する関数ではなくて DOM 自身を参照する ref だけ、親コンポーネントが欲しているときとか）

ここでは`useImpertativeHandle`をあえて無視する。

解決策：

-   `Callback ref`を使う。

https://stackoverflow.com/a/72519722

-   複数 ref からなるオブジェクトを指す ref を渡す

https://stackoverflow.com/a/53818443

## 動的配列で生成される各コンポーネント全てに ref を渡したいとき

https://stackoverflow.com/a/56063129

-   `ref`配列の長さは`useEffect`で更新する
-   動的配列から生成される各要素には`ref={_refTabs.current[index]}`という方法で ref を渡す

```TypeScript
interface iProps {
  // div.tabsの数。TabsはnumberOfTabsを元に表示div.tab数を決定する
  numberOfTabs: number;
}

const Tabs = ({ numberOfTabs }: iProps) => {
  const _refTabArea = useRef<HTMLDivElement>(null);
  const _refTabs = useRef<HTMLDivElement[]>([]);

  // _refTabs.currentのref配列の数をnumberOfTabsに一致するように再計算するための更新
  useEffect(() => {
    if (_refTabs.current) {
      _refTabs.current = _refTabs.current.slice(0, numberOfTabs);
    }
  }, []);

  // _refTabs.currentのref配列の数をnumberOfTabsに一致するように再計算するための更新
  useEffect(() => {
    if (_refTabs.current) {
      _refTabs.current = _refTabs.current.slice(0, numberOfTabs);
    }
  }, [numberOfTabs]);

  //...

  // DOM情報を使ってdiv.tabのclassNameを変更する関数
  const changeTab = (selectedTabNode: HTMLDivElement, index: number) => {
    for (var i = 0; i < _refTabArea.current!.childNodes.length; i++) {
      var child: iJSXNode = _refTabArea.current!.childNodes[i];
      if (/tab/.test(child.className!)) {
        child.className = "tab";
      }
    }
    selectedTabNode.className = "tab active";
  };

  return (
    <div className="tabs-area" ref={_refTabArea} >
      {Array.from({ length: numberOfTabs }, (_, i) => i + 1).map((i, index) => (
        <div
          className={index === selected ? "tab active" : "tab"}
          key={index}
          style={stylesOfTab}
          ref={(node: HTMLDivElement) => (_refTabs.current[index] = node)}
          onClick={() => changeTab(_refTabs.current[index], index)}
        >
          <span>tab {i}</span>
        </div>
      ))}
    </div>
  );
};
```



## [記事化しない情報] 監視する DOM を変更したいとき

TODO: 検証を完了させて。

動的に参照対象の DOM を切り替える可能性がある機能を作っているとする。

明示的に何らかのイベント時に`ref.current`の参照対象を変更されることがあるとして、そのことを検知するにはどうすべきか。

公式より：

> (`callback ref`を使えば)ref を別の DOM にアタッチしたときにコールバックを呼び出します。

> この例で useRef を選択しなかったのは、オブジェクト ref では現在の ref 値の変更が通知されないからです。コールバック ref を使用することで、子コンポーネントが後で（例えばクリックに反応して）測定したノードを表示した場合でも、親コンポーネントでその変更が通知され、測定値を更新することができます。

> useCallback に依存配列として[]を渡していることに注意してください。これにより、再レンダリング時に ref コールバックが変更されず、React が不必要にコールしないようになります。

```JavaScript
function MeasureExample() {
  const [rect, ref] = useClientRect();
  return (
    <>
      <h1 ref={ref}>Hello, world</h1>
      {rect !== null &&
        <h2>The above header is {Math.round(rect.height)}px tall</h2>
      }
    </>
  );
}

function useClientRect() {
  const [rect, setRect] = useState(null);
  // `ref`は関数で、こいつを`ref`属性に渡せばコールバックrefになる。
  const ref = useCallback(node => {
    if (node !== null) {
      setRect(node.getBoundingClientRect());
    }
  }, []);
  return [rect, ref];
}

```

このカスタムフックは、

マウント時、ref の対象変更時、アンマウント時に呼び出される。

毎レンダリングは呼び出されない。

## [記事化しない情報] DOM のリサイズを監視したいときは？

TODO: 検証を完了させて。

公式が`ResizeObserver`かサードパーティ製の Hook を使えと言っている。

が、やれないこともない。

検証コード：`./caseWatchResize`.

以下のように、callback ref + useCallback の組み合わせで

ref の参照対象の DOM の最新情報を取得するタイミングを、

useCallback の依存関係に何を含めるかで制御できる。

```TypeScript
import React, { useState, useEffect, useRef, useCallback } from "react";
import { useClientRect } from "./hooks/useClientRect";

const styleOfItem: React.CSSProperties = {
  minWidth: "100px",
  height: "80px",
  backgroundColor: "#565879",
  border: "1px solid white"
};

/****
 * `increment`, `decrement`のボタンクリックに応じてitemが増減するので
 * div.itemsのwidthが併せて増減する。
 *
 * - case useCllientRect(): マウント時に取得した値を保持し続けるだけで、対象DOMのサイズが変更されても更新したりしない
 * - case callback ref: 毎レンダリング後、useEffectの呼び出し前に呼び出される。ただし特に必要がなくても必ず呼び出されるので無駄な処理が毎レンダリング発生することになる。
 * - case useCallback + callback ref: callback refを更新するタイミングをuseCallbackの依存関係に含める値が変化したときにできる。これなら無駄な処理が発生しない。
 *
 * */
export const TestUseClientRectHook = () => {
  const [items, setItems] = useState<number>(10);
  const [bgColor, setBgColor] = useState<string>("#565879");
  // ボツ案１：
  // const [rect, ref] = useClientRect<HTMLDivElement>();

  // ボツ案２：
  // const refContainer = (node: HTMLDivElement) => {
  //   if (!node) return;
  //   const rect = node.getBoundingClientRect();
  //   console.log(`rect.width: ${rect.width}`);
  // };

  const cbRefContainer = useCallback(
    (node: HTMLDivElement) => {
      if (!node) return;
      const rect = node.getBoundingClientRect();
      console.log(`rect.width: ${rect.width}`);
    },
    [bgColor, items]
  );

  useEffect(() => {
    console.log("compoent did update");
  });

  const handleCounterIncrement = (e: React.MouseEvent<HTMLButtonElement>) => {
    e.preventDefault();
    e.stopPropagation();

    console.log("increment");

    setItems(items + 1);
  };

  const handleCounterDecrement = (e: React.MouseEvent<HTMLButtonElement>) => {
    e.preventDefault();
    e.stopPropagation();

    console.log("decrement");

    setItems(items - 1 === 0 ? items : items - 1);
  };

  const handleChangeBackground = (e: React.MouseEvent<HTMLButtonElement>) => {
    e.preventDefault();
    e.stopPropagation();

    console.log("change background color");

    if (bgColor === "#C6D3D3") {
      setBgColor("#565879");
    } else {
      setBgColor("#C6D3D3");
    }
  };

  const width = items * 100;

  console.log("rendering...");
  console.log(`width: ${width}`);

  return (
    <div>
      <div
        className="items"
        style={{
          width: width + "px",
          height: "80px",
          backgroundColor: "#a7a9c1",
          display: "flex"
        }}
        // ref={ref}
        // ref={refContainer}
        ref={cbRefContainer}
      >
        {Array.from({ length: items }, (_, i) => i + 1).map((i, index) => (
          <div
            className="tab"
            key={index}
            style={{
              minWidth: "100px",
              height: "80px",
              backgroundColor: bgColor,
              border: "1px solid white"
            }}
          >
            tab {i}
          </div>
        ))}
      </div>
      <div>
        <button onClick={handleCounterIncrement}>increment</button>
        <button onClick={handleCounterDecrement}>decrement</button>
        <button onClick={handleChangeBackground}>change background</button>
      </div>
    </div>
  );
};

```

上記であれば、`bgColor`だけにしたら`bgColor`の変更時のみ callback ref が呼び出されて、`items`抜きなら`items`更新時に callback ref は呼び出されない。

依存関係が変化したときだけ呼び出される callback ref になったので、毎レンダリング無駄に呼び出されることもなくなった。