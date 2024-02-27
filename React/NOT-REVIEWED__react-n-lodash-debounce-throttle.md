# React で `lodash.throttle` や `setTimeout` など遅延関数を使う方法の模索

## この記事は何？

React で`lodash`の`debounce`や`throttle`を使う方法はないかと探している人向けの記事。

安直に`useCallback`等を使えば行けるのでは？と考えがちになってしまう自分が今後同じ轍を踏まないために経験を記す。

## 結論

- `lodash`は React 向けのライブラリではないのでラッピングが大変面倒でかつ使用状況に制限がある
- `setTimeout`と`useEffect`の組み合わせがシンプルで分かりやすい
- `setTimeout`と`useEffect`からなるカスタム・フックは、汎用的にするよりも使用場面に特化させれば便利である
- 一番手っ取り早いのは`react-use`の各タイマー関数を使うべき

## summary

- [Lodash `throttle`は React で使えるか？](#Lodash-`throttle`は-React-で使えるか？)
- [`lodash.throttle` + `useCallback`](#`lodash.throttle`-+-`useCallback`)
- [`useRef` + `useEffect`](#`useRef`-+-`useEffect`)
- [遅延関数を作るのに lodash を使わない例](#遅延関数を作るのにlodashを使わない例)
- [遅延関数のカスタムフックを作ってみる](#遅延関数のカスタムフックを作ってみる)
- [余談：`lodash`ライブラリ全部インストールするな](#余談：`lodash`ライブラリ全部インストールするな)

## Lodash `throttle`は React で使えるか？

※class コンポーネントで使う場合を扱いません

かねてより、React では純粋関数が求められている。

そのため、

```JavaScript
const delayedFunc = throttle(() => onMouseMoveHandler(), 1000);
```

を React 関数内で定義しても、毎レンダリング時に`delayedFunc`は再作成される。

再作成されるために、遅延効果が発生しない。

再レンダリング前の呼び出しによるタイマーは、再レンダリングによって以前の関数が消え、タイマーがリセットされるためである。

React でこうした遅延関数を使う方法がないか模索する。

## `lodash.throttle` + `useCallback`

`useCallbacK`は、依存関係が変化するたびに再作成されるので、再作成前の関数と別物になることから遅延を発生させる機能を持った関数に使うべきでない。

ただし`useCallbacK`の依存関係に何も含めないならば、つまり一度`throttled`を生成して以降変更させないならば使うことができる。

```JavaScript
// GOOD: 依存関係がなく再作成されないから。
const throttled = useCallback(throttle(newValue => console.log(newValue), 1000), []);

// BAD：依存関係の変化のたびに`throttled`は「新しくなる」。
const throttled = useCallback(throttle(() => console.log(value), 1000), [value]);
```

後者は value の値が変化するたびに遅延を発生させずに即時実行されてしまう。

#### 検証: `lodash.throttle` + `useCallback`において`useCallback`の依存関係有無によって結果が変わるか

`useCallback`に依存関係を含めると、毎レンダリング時に再生成される例：

input フォームへ連続で入力を続けたら、１秒ごとにコンソールへログを出力するはずが毎入力ごとに反応している。

```TypeScript
import React, { useCallback, useState } from "react";
import throttle from "lodash.throttle";

const ReactalizeLodash = () => {
  const [value, setValue] = useState<string>("initial");

  const throttled = useCallback(
    throttle(() => console.log(value), 1000),
    [value]
  );

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    setValue(e.currentTarget.value);
    throttled();
  };
  return (
    <div className="reactalize-lodash">
      <input onChange={handleChange} type="text" />
    </div>
  );
};
```

依存関係を含めないとそれは起こらない。

```TypeScript
import React, { useCallback, useState } from "react";
import throttle from "lodash.throttle";

const ReactalizeLodash = () => {
  const [value, setValue] = useState<string>("initial");

  const throttled = useCallback(
    throttle((val) => console.log(val), 1000),
    []
  );

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    setValue(e.currentTarget.value);
    throttled(e.currentTarget.value);
  };
  return (
    <div className="reactalize-lodash">
      <input onChange={handleChange} type="text" />
    </div>
  );
};
```

ということで、

依存関係なし`useCallback`で throttle を囲う方法はアリ。

依存関係を含めなくてはならない場合は`lodash.throttle`の使用を見直す必要がある。

#### `useRef` + `useEffect`

そもそも lodash は React 向けに作られているわけじゃないので`useRef` + `useEffect`の組み合わせで React の理の外に出すべき。

```TypeScript
import React, { useRef, useState, useEffect } from "react";
import throttle from "lodash.throttle";

const ThrottleNRef = () => {
  const [value, setValue] = useState<string>("initial");
  const refThrottle = useRef(
    throttle((newValue) => console.log(newValue), 1000)
  );

  useEffect(() => {
    if (refThrottle.current) {
      refThrottle.current(value);
    }
  }, [value]);

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    setValue(e.currentTarget.value);
  };

  return (
    <div className="reactalize-lodash">
      <input onChange={handleChange} type="text" />
    </div>
  );
};

export default ThrottleNRef;
```

ここまでして`lodash.throttle`を使う価値があるのかは使う人の事情によるとして、

シンプルな使い方を求めるならやはり後で述べるように`setTimeout`を使うのがいいと思う。

## 遅延関数を作るのに lodash を使わない例

結局`setTimeout`, `clearTimeout`の組み合わせてカスタム HOOK にするのが使いやすい。

どう便利なのかは以下の例をみればわかるかも。

参考：

https://stackoverflow.com/questions/36270422/reactjs-settimeout-not-working?rq=3

https://stackoverflow.com/questions/29526739/stopping-a-timeout-in-reactjs?rq=3

#### `setTimeout` + `useEffect`

要素の連続で発生する onresize イベントに対して一番最後のイベントにのみハンドラが反応するようにタイマーを使う。

サンプルコードの説明：

リサイズ可能な要素をリサイズすると、リサイズしてから遅れてリサイズ後の幅の値をログに出力する。

処理の流れ：

- onresize イベントが発生する
- `sectionWidth`が更新される
- `useEffect(,[sectionWidth])`が実行される
- タイマーがセットされる
- 500 ミリ秒経過する間に`useEffect(,[sectionWidth])`が再度呼び出されなければ`setTimeout`のコールバックが実行されて、経過前に再度呼び出されれば`clearTimeout`される。

```TypeScript
const Timer = () => {
  const [sectionWidth, setSectionWidth] = useState<number>(300);

  useEffect(() => {
    const timer = setTimeout(() => {
      console.log(sectionWidth);
    }, 500);

    return () => clearTimeout(timer);
  }, [sectionWidth]);

  const onEditorSecResize: (
    e: React.SyntheticEvent,
    data: ResizeCallbackData
  ) => any = (event, { node, size, handle }) => {
    setSectionWidth(size.width);
  };

  return (
    <>
      <ResizableBox
        width={sectionWidth}
        height={Infinity}
        minConstraints={[100, Infinity]}
        maxConstraints={[600 * 0.8, Infinity]}
        onResize={onEditorSecResize}
        resizeHandles={["e"]}
        handle={(h, ref) => (
          <span className={`custom-handle custom-handle-${h}`} ref={ref} />
        )}
      >
        <div className="resizable-section" style={{ width: sectionWidth }}>
          <div
            className="box"
            style={{
              width: sectionWidth,
              height: "400px",
              backgroundColor: "#47487a"
            }}
          ></div>
        </div>
      </ResizableBox>
    </>
  );
};

```

こちらのほうが心理的に安心。

lodash という JavaScript 向けライブラリを React でどうにか使う場合はどこか見落としがないか常に不安なので。

タイマーを発生させるかどうかはどの値を`useEffect`の依存関係に渡すかで決められて便利。

#### よく見かけるカスタム・フック化について

よくみるやつ。

以下の例はネットで検索するとしょっちゅう出てくるよくない例。

```TypeScript
import React, { useEffect, useRef } from "react";

/**
 * cb: タイマーで実行させたいコールバック関数
 * delay: setTimeoutのdelay引数
 * trigger: `useEffect()`に渡す依存関係で同時に実行のトリガー
 * refCb: 渡されたcb引数を参照する。
 * */
const useTimer = (cb: () => void, delay: number, trigger: any) => {
  const refCb = useRef<() => void>();

  useEffect(() => {
    // DEBUG:
    console.log("[useTimer] This side effect runs every call useTimer.");

    if (refCb.current === undefined) {
      refCb.current = cb;
    }
  }, [cb]);

  useEffect(() => {
    const timer = setTimeout(() => {
      if (refCb.current) {
        refCb.current();
      }
    }, delay);

    return () => clearTimeout(timer);
  }, [trigger, delay]);
};

export default useTimer;


// 呼び出し側
  useTimer(() => console.log(sectionWidth), 1000, sectionWidth);
```

これの問題点は`useEffect(,[cb])`が毎度呼び出されることである。

引数`cb`（`() => console.log(sectionWidth)`）は毎呼び出し新しく生成され

`cb`引数は常に前回と異なるものだから必ず毎度`useEffect(,[cb])`が実行される。

そのため`useRef`の存在意義が失われている。

こうなった場合、`cb`は引数で受け取ることをやめてハードコードしたほうがいいかも。

汎用性が下がるけど、ある使用場面に特化させたほうが余計な処理もなくわかりやすい。

#### 最後の`onresize`イベントにのみ反応して DOMRect を返すカスタム・フック

例：`onresize`イベントハンドラの発火にたいしてタイマーを設け、タイマーが切れたらある DOM の DOMRect を取得して返す

```TypeScript
import React, { useState, useEffect, useRef } from "react";

/***
 * 呼び出し側で発生したonresizeイベントに応じてタイマーをセットし、
 * 次のタイマーの呼び出しがなかった時に
 * `refDom`の指すDOMからDOMRect情報を取得し返す
 *
 * @param {number} delay - Delayed time of setTimeout. Also time from call to completion.
 * @param {number} width - Width of element which fired resize event on caller.
 *
 * ある要素のリサイズの後にDOMRect情報が欲しいときに使うと便利。
 * */
const useDelayedRect = <T extends HTMLElement = HTMLDivElement>(
  delay: number,
  width: number
) => {
  const [rect, setRect] = useState<DOMRect>();
  const refDom = useRef<T>(null);

  useEffect(() => {
    if (refDom.current === undefined || !refDom.current) return;
    const timer = setTimeout(() => {
      const _rect = refDom.current!.getBoundingClientRect();
      setRect(_rect);
    }, delay);

    return () => clearTimeout(timer);
  }, [width, delay]);

  return [rect, refDom];
};

export default useDelayedRect;
```

使い方：

```TypeScript
import React, { useEffect, useState } from "react";
import { ResizableBox } from "react-resizable";
import type { ResizeCallbackData } from "react-resizable";
import useDelayedRect from "./useDelayedRect";

const ResizableContainer = () => {
  const [sectionWidth, setSectionWidth] = useState<number>(300);
  // タイマー1000ミリ秒
  // 実行トリガー：`sectionWidth`が更新したら
  const [rect, refDom] = useDelayedRect<HTMLDivElement>(1000, sectionWidth);

  useEffect(() => {
    console.log(rect);
  }, [rect]);

  const onEditorSecResize: (
    e: React.SyntheticEvent,
    data: ResizeCallbackData
  ) => any = (event, { node, size, handle }) => {
    setSectionWidth(size.width);
  };

  return (
    <>
      <ResizableBox
        width={sectionWidth}
        height={Infinity}
        minConstraints={[100, Infinity]}
        maxConstraints={[600 * 0.8, Infinity]}
        onResize={onEditorSecResize}
        resizeHandles={["e"]}
        handle={(h, ref) => (
          <span className={`custom-handle custom-handle-${h}`} ref={ref} />
        )}
      >
        <div
          className="resizable-container"
          style={{ width: sectionWidth }}
          // DOMRectを取得する要素のrefへrefDomを渡す
          ref={refDom}
        >
          <div
            className="box"
            style={{
              width: sectionWidth,
              height: "400px",
              backgroundColor: "#47487a"
            }}
          ></div>
        </div>
      </ResizableBox>
    </>
  );
};

export default Timer;
```

`ResizableBox`を水平方向にリサイズすると、

リサイズ中は`useDelayedRect`（の setTimeout コールバック）は実行されず、

リサイズを終えたら実行され、`rect`が更新される。

## 余談：`lodash`ライブラリ全部インストールしてはならない

lodash ライブラリは巨大である。

```bash
$ npm install lodash
```

とそのまますべてインストールしてしまうと、

webpack 環境などだと lodash ライブラリすべてをバンドルするので

lodash のうち使っていないライブラリコードがアプリケーションに大量に含まれることになる。

そのため使うライブラリだけインストールするようにする。

`debounce`だけ使いたいならば

```bash
$ npm install --save lodash.debounce
$ npm install --dev @types/lodash.debounce
```

```TypeScript
import debounce from 'lodash.debounce';
```

## 参考

https://stackoverflow.com/a/54666498/22007575

https://stackoverflow.com/a/43479515

`react-use`では`useDebounce`と名付けつつ中身は`setTimeout`を使っていた。

https://github.com/streamich/react-use/blob/master/src/useDebounce.ts
