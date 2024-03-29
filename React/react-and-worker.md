# React + WebWorker

React + WebWorker を実現するために試行錯誤した記録とたどり着いた解決策をここに記します。

## Summary

-   [環境](#環境)
-   [React は値を保持しない](#Reactは値を保持しない)
-   [よく見かける提案](#よく見かける提案)
-   [検証](#検証)
-   [message がやり取りできない原因](#message-がやり取りできない原因)
-   [worker インスタンスを維持するために useMemo()が使えない理由](<#workerインスタンスを維持するためにuseMemo()が使えない理由>)
-   [解決策: `useRef`を使う](#解決策-userefを使う)
-   [class コンポーネントはまだ現役](#classコンポーネントはまだ現役)
-   [React+Webpack での worker の扱い方](#react+webpack-での-worker-の扱い方)
-   [webpack で webworker を扱ううえでの注意](#webpackでwebworkerを扱ううえでの注意)
-   [1. バンドルする関係上予期せぬライブラリが依存関係に含まれる](#1-バンドルする関係上予期せぬライブラリが依存関係に含まれる)
-   [その他注意点: web event では React は更新されない](#その他注意点-web-event-では-react-は更新されない)
-   [これまでの話をまとめて webworker のカスタムフックを作る](#これまでの話をまとめて-webworker-のカスタムフックを作る)
-   [まとめ](#まとめ)

## 環境

webpack5 + React 17 (React 18 でも同じことですが)

## React は値を保持しない

関数コンポーネントは純粋関数でなくてはなりません。

そのため関数は再レンダリングにまたがって値を保持する方法として提供しているのが

`useState`, `useRef`, `useMemo`, `useCallback`等であり、

関数コンポーネント内で宣言された JavaScript 変数は再レンダリングをまたがって保持しません。

毎度レンダリング時に関数コンポーネントは再実行され、以前の変数は消え、今回のレンダリングで同じ変数は初期値を与えられます。

そのため worker を使うためには

どうにかして webworker のインスタンスを再レンダリング時にまたがって保持する必要があります。

## よく見かける提案

ネットを検索するとわりかし次の記事のような提案をよく見かけます。

関数コンポーネントで webworker を使うためには、`useMemo()`を使うことをよく提案されています。

https://blog.logrocket.com/web-workers-react-typescript/

https://stackoverflow.com/questions/70794189/using-web-workers-with-react-and-webpack-5/72081241#72081241

```TypeScript
// 関数コンポーネント内にて
const worker: Worker = useMemo(
    () =>
        new Worker(
            new URL('/src/worker/Counter.worker.ts', import.meta.url)
        ),
    []
);
```

`useMemo()`の触れ込み通りならば、

依存関係ぬきで`useMemo()`で worker インスタンスを生成すれば、

マウント時にこれを実行すれば関数コンポーネントがアンマウントされない限り worker インスタンスは毎レンダリングをまたがって保持されるはずです。

検証してみましょう。

## 検証

#### 検証コード

worker に負荷の高い計算を任せて、

その計算が済んだら返してもらって結果を表示するプログラムを用意しました。

とにかく React で worker が使えるのかどうかだけ確かめるだけです。

ディレクトリ構成：

```bash
public/
    index.html
src/
    index.tsx
    App.tsx
    components/
        FunctionalCase.tsx
    worker/
        Coutner.worker.ts
tsconfig.json
webpack.config.js
```

```TypeScript
// FunctionalCase.tsx
import React, { useEffect, useMemo, useState } from 'react';
import type { iRequest, iResponse } from './worker/Counter.worker';

const FunctionalCase = () => {
    const [counter, setCounter] = useState<number>(0);

    const worker: Worker = useMemo(
        () =>
            new Worker(
                new URL('/src/worker/Counter.worker.ts', import.meta.url)
            ),
        []
    );

    useEffect(() => {
        console.log('did mount');

        if (window.Worker) {
            console.log('set message listener');

            worker.addEventListener('message', handleWorkerMessage);
        }

        return () => {
            if (window.Worker) {
                console.log('termnate worker');

                worker.removeEventListener('message', handleWorkerMessage);
                worker.terminate();
            }
        };
    }, []);

    // 依存関係にworkerを含めることに意味はあるか？

    const handleWorkerMessage = (e: MessageEvent<iResponse>) => {
        const { count } = e.data;
        console.log(`Got message from worker: ${count}`);
        setCounter(count);
    };

    const handleClick = () => {
        console.log('click');

        worker.postMessage({
            count: counter,
            order: 'calculate',
        } as iRequest);
    };

    return (
        <div className="functional-case">
            <h2>In case functional component with Worker</h2>
            <button onClick={handleClick}>count up</button>
            <h2>{counter}</h2>
        </div>
    );
};

export default FunctionalCase;
```

```TypeScript
// Counter.worker.ts
export interface iRequest {
    order: 'calculate';
    count: number;
}

export interface iResponse {
    count: number;
}

const expensiveCalculator = (val: number) => {
    let result = val;
    for (let i = 0; i < 10000; i++) {
        result += 1;
    }
    return result;
};

self.addEventListener('message', (e: MessageEvent<iRequest>) => {
    const { order, count } = e.data;
    // To ignore other posted message.
    if (order !== 'calculate') return;

    const result = expensiveCalculator(count);
    self.postMessage({
        count: result,
    });
});
```

[webpack.config.js, tsconfig.json の設定について](#設定)

以下のような処理の流れが期待されます。

-   `<button onClick={handleClick}>count up</button>`のボタンをクリック
-   `Counter.worker`へメッセージを送信し
-   `Counter.worker`は 1 万ループする無駄計算を実行し 1 万増えた値を`FunctionaleCase`へ返す
-   `FunctionaleCase`の`coutner`が更新される

#### 実行結果

ではアプリケーションを実行して、counter のボタンを押してみましょう。

```bash
$ npm run start
# ....
did mount
set message listener
termnate worker
# StrictModeによる再実行
did mount
set message listener
[Counter.worker] running...
[Counter.worker] running...
# ボタンをクリックしてworkerへメッセージを送信したが...
click

# ...なにも起こらない
```

`click`の表示以降何も変化しません。

なぜでしょう？

原因を追究します。

メッセージを取得したらコンソールに表示されるようにしているので、メッセージ自体が届いていない可能性があります。

## message がやり取りできない原因

#### origin が違うから？

origin は同じでした。

アプリケーションはローカルで実行しており、

メイン環境、ワーカー環境どちらも`http://localhost:8080`であることを確認済です。

#### `React.StrictMode`で二度実行されていることが関係しているから？

結論を言うとこれが原因です。

事実、`StrictMode`を無くすと期待通りに動きます。

そのため`StrictMode`が何かしら関係していることが考えられます。

## worker インスタンスを維持する手段として useMemo()が使えない原因

原因ははっきりと特定できていません。

しかし以下のようではないかなと思っています。

原因の所在は`React.StrictMode`によりコンポーネントが二度実行されている点にあることははっきりしています。

https://react.dev/reference/react/useMemo#caveats

`useMemo`が`StrictMode`により 2 度実行され、その`useMemo`呼出のうちのいずれかは破棄されます。
公式は、`useMemo`の使い方としては、計算関数は純粋関数を使っているはずだから、何度実行しても同じ値が返されるはずだからどちらを破棄しても問題ないよねというスタンスです。

なので、可能性としては

React は初期生成ワーカーを残すつもりで再実行で生成されるワーカーを破棄したが、

実はクリーンアップ関数で初期生成ワーカーが`terminate()`されていた...

というシナリオはあり得ます。

React が破棄する方の値が常に最初に生成した方であった場合、

`useMemo()`はクリーンアップ関数で処理が必要な値を扱ってはならないということになり、そうであった場合関数コンポーネントは worker を terminate()出来ないことが明らかになります。

実際`StrcitMode`なしで実行すれば発生しない問題なので worker が二度実行されている点、`useMemo()`を使っている点から推測するとなくはないと思っています。

## 解決策: `useRef`を使う

そもそもネットに載っているような方法を使っているのが原因では？というのが根本原因なので、

基本を思い出し、worker は React の理の外である、つまり`useRef`を使うべきと気づくべきでした。

#### 検証

```TypeScript
import React, { useEffect, useMemo, useState, useRef } from 'react';
import type { iRequest, iResponse } from './worker/Counter.worker';

const FunctionalCase = () => {
    const [counter, setCounter] = useState<number>(0);
    const refWorker = useRef<Worker>();

    // const worker: Worker = useMemo(
    //     () =>
    //         new Worker(
    //             new URL('/src/worker/Counter.worker.ts', import.meta.url)
    //         ),
    //     []
    // );

    useEffect(() => {
        console.log('did mount');

        // if (window.Worker) {
        //     console.log('set message listener');

        //     worker.addEventListener('message', handleWorkerMessage);
        // }

        if (window.Worker && refWorker.current === undefined) {
            console.log('Generate worker and set message listener');

            refWorker.current = new Worker(
                new URL('/src/worker/Counter.worker.ts', import.meta.url)
            );
            refWorker.current.addEventListener('message', handleWorkerMessage);
        }

        return () => {
            if (window.Worker && refWorker.current) {
                console.log('terminate worker');

                // worker.removeEventListener('message', handleWorkerMessage);
                // worker.terminate();

                refWorker.current.removeEventListener(
                    'message',
                    handleWorkerMessage
                );
                refWorker.current.terminate();
                refWorker.current = undefined;
            }
        };
    }, []);

    const handleWorkerMessage = (e: MessageEvent<iResponse>) => {
        const { count } = e.data;
        console.log(`Got message from worker: ${count}`);
        setCounter(count);
    };

    const handleClick = () => {
        console.log('click');

        if (refWorker.current === undefined) return;

        refWorker.current.postMessage({
            count: counter,
            order: 'calculate',
        } as iRequest);
    };

    return (
        <div className="functional-case">
            <h2>In case functional component with Worker</h2>
            <button onClick={handleClick}>count up</button>
            <h2>{counter}</h2>
        </div>
    );
};

export default FunctionalCase;
```

結果：

```bash
# workerコンテキストが生成された
[Counter.worker] running...
# 初期マウント
did mount
Generate worker and set message listener
terminate worker
# StrictModeによる再実行
did mount
Generate worker and set message listener
[Counter.worker] running...

# リクエストボタンをクリックしてみたら...
click
[Counter.worker] got request
[Counter.worker] send result
Got message from worker: 10000

click
[Counter.worker] got request
[Counter.worker] send result
Got message from worker: 20000

# このとおりworkerと通信出来た。
```

上記のコードのとおり、

`terminate()`した後に`ref.current`へ undefined を渡さないとなりません。

これをしなかった場合、`useEffect`の再実行時に`refWorker.current`はまだオブジェクトを保持しています。

オブジェクトが残っているのならば worker と通信できるのでは？と思っても通信はできません。

試してみたのですが worker のコンテキストは多分消えています。

なので`terminate()`の役割は、worker コンテキストの始末（メモリの解放とか？）であって、

当然ですが worker インスタンスである値が undefined やら null になるわけではありません。

ここら辺は先の`useMemo()`が使えなかった原因に結びつくかもしれません。

## class コンポーネントはまだ現役

先の通りの方法でほぼ解決かと思いますが、class コンポーネントを使うという手段もあります。

つまり、class の field として worker インスタンスを保持するという方法です。

#### 検証

```TypeScript
import React from 'react';
import type { iResponse, iRequest } from './worker/Counter.worker';

interface iProps {}
interface iState {
    counter: number;
}

class ClassCase extends React.Component<iProps, iState> {
    state = {
        counter: 0,
    };
    _worker: Worker | undefined;

    constructor(props: iProps) {
        super(props);
        this.handleMessage = this.handleMessage.bind(this);
        this.handleClick = this.handleClick.bind(this);
    }

    componentDidMount() {
        console.log('did mount');
        if (window.Worker && this._worker === undefined) {
            console.log('generate and setup worker');

            this._worker = new Worker(
                new URL('/src/worker/Counter.worker.ts', import.meta.url)
            );
            this._worker.addEventListener('message', this.handleMessage);
        }
    }

    componentDidUpdate() {
        console.log('did update');
    }

    componentWillUnmount(): void {
        if (window.Worker && this._worker !== undefined) {
            console.log('terminate worker');

            this._worker.removeEventListener('message', this.handleMessage);
            this._worker.terminate();
            this._worker = undefined;
        }
    }

    handleMessage(e: MessageEvent<iResponse>) {
        const { count } = e.data;
        console.log(`Got message from worker: ${count}`);
        this.setState({ counter: count });
    }

    handleClick() {
        console.log('click');

        if (this._worker === undefined) return;

        this._worker.postMessage({
            count: this.state.counter,
            order: 'calculate',
        } as iRequest);
    }

    render() {
        return (
            <div className="functional-case">
                <h2>In case functional component with Worker</h2>
                <button onClick={this.handleClick}>count up</button>
                <h2>{this.state.counter}</h2>
            </div>
        );
    }
}

export default ClassCase;
```

結果：

```bash
Counter.worker.ts:18 [Counter.worker] running...
03:26:46.701 ClassCase.tsx:20 did mount
03:26:46.704 ClassCase.tsx:22 generate and setup worker
03:26:46.707 ClassCase.tsx:37 terminate worker
03:26:46.708 ClassCase.tsx:20 did mount
03:26:46.709 ClassCase.tsx:22 generate and setup worker
03:26:50.209 Counter.worker.ts:18 [Counter.worker] running...

# リクエストボタンを押してみたら
03:26:55.065 ClassCase.tsx:52 click
03:26:55.066 Counter.worker.ts:25 [Counter.worker] got request
03:26:55.067 Counter.worker.ts:29 [Counter.worker] send result
03:26:55.071 ClassCase.tsx:47 Got message from worker: 10000
03:26:55.082 ClassCase.tsx:32 did update
03:26:59.521 ClassCase.tsx:52 click
03:26:59.522 Counter.worker.ts:25 [Counter.worker] got request
03:26:59.522 Counter.worker.ts:29 [Counter.worker] send result
03:26:59.525 ClassCase.tsx:47 Got message from worker: 20000
03:26:59.532 ClassCase.tsx:32 did update

# 先の例と同じ結果になった
```

## React+Webpack での worker の扱い方

## 設定

webpack 5 でのお話です。

基本公式の通りでいいと思います。

https://webpack.js.org/guides/web-workers/

リンクのページでは webpack.config.js の具体的な設定は載っていませんが、以下の通りで問題なく動いています。

```JavaScript
const path = require('path');
const HtmlWebPackPlugin = require('html-webpack-plugin');
const ReactRefreshWebpackPlugin = require('@pmmmwh/react-refresh-webpack-plugin');

const isDevelopment = process.env.NODE_ENV !== 'production';

module.exports = {
    mode: 'development',
    entry: {
        index: './src/index.tsx',
        'your.worker': './src/worker/your.worker.ts',
        'another.worker': './src/worker/another.worker.ts',
    },
    resolve: {
        extensions: ['.js', '.jsx', '.tsx', '.ts'],
    },
    output: {
        globalObject: 'self',
        filename: '[name].bundle.js',
        path: path.resolve(__dirname, 'dist'),
    },
    module: {
        rules: [
            {
                test: /\.(js|jsx|tsx|ts)$/,
                exclude: /node_modules/,
                use: [
                    {
                        loader: require.resolve('babel-loader'),
                        options: {
                            presets: [
                                '@babel/preset-env',
                                '@babel/preset-typescript',
                                '@babel/preset-react',
                            ],
                            plugins: [
                                isDevelopment &&
                                    require.resolve('react-refresh/babel'),
                            ].filter(Boolean),
                        },
                    },
                ],
            }
        ],
    },
    plugins: [
        new HtmlWebPackPlugin({
            template: 'src/index.html',
        }),
        isDevelopment && new ReactRefreshWebpackPlugin(),
    ].filter(Boolean),
};
```

重要なポイントは

-   `globalObject: 'self'`
-   `entry`に worker ファイルを含める

`target`プロパティがデフォルト(`web`)のままだけど問題なくworkerは生成されるみたいです。

#### webpack5 での worker の使い方について

https://webpack.js.org/guides/web-workers/#root

プラグインが不要になった。

## webpack で webworker を扱ううえでの注意

## バンドルする関係上予期せぬライブラリが依存関係に含まれる

webpack は worker ファイルのの依存関係をすべてひとつにバンドルします。

すると以下のようなエラーに遭遇することがあります。

```bash
browser.js:131 Uncaught (in promise) ReferenceError: window is not defined
    at eval (browser.js:131:1)
    at ./node_modules/monaco-editor/esm/vs/base/browser/browser.js (vendors-node_modules_monaco-editor_esm_vs_editor_editor_main_js-node_modules_idb-keyval_dist_-7d7b36.bundle.js:928:1)
    at options.factory (src_worker_fetchLibs_worker_ts.bundle.js:790:31)
    at __webpack_require__ (src_worker_fetchLibs_worker_ts.bundle.js:208:33)
    at fn (src_worker_fetchLibs_worker_ts.bundle.js:429:21)
    at eval (fontMeasurements.js:6:82)
    at ./node_modules/monaco-editor/esm/vs/editor/browser/config/fontMeasurements.js (vendors-node_modules_monaco-editor_esm_vs_editor_editor_main_js-node_modules_idb-keyval_dist_-7d7b36.bundle.js:3238:1)
    at options.factory (src_worker_fetchLibs_worker_ts.bundle.js:790:31)
    at __webpack_require__ (src_worker_fetchLibs_worker_ts.bundle.js:208:33)
    at fn (src_worker_fetchLibs_worker_ts.bundle.js:429:21)
```

`ReferenceError: window is not defined`というエラー。

本来、webworker のグローバル変数は`DedicatedWorkerGlobalScope`というものになるはずで、

`./node_modules/monaco-editor/esm/vs/base/browser/browser.js`という知らん奴がどういうわけかワーカー環境の中で`window`オブジェクトを参照しようとしているというエラーです。

しかしワーカーは`./node_modules/monaco-editor/esm/vs/base/browser/browser.js`を一切 import していない。

そんなとき。

#### 原因

こうなる原因は webpack が全ての依存関係をワーカーのために一つにバンドルするためである。

つまり、バンドルした依存関係の中に`window`を必要とする依存関係が存在したのである。

ワーカーのコード：

```TypeScript
// awesome.worker.ts
import { logger } from '../utils';

// ...以下略
```

一見一切`monaco-editor`のモジュールは import していない。

しかし実は`utils/index.ts`は`monaco-editor`を import しているモジュールを import していた。

それは`src/utils/index.tsx`である。

```TypeScript
// utils/index.ts

// こいつ。
// `getModelByPath`はmonacoをインポートしている。
export { getModelByPath } from './getModelByPath';

// ...

// `awesome.worker`が取り込もうとしていた対象
export { logger } from "./logger";
```

`src/utils/index.tsx`はその内に`./getModelByPath`という monaco-editor を内に import しているモジュールを取りこんでいた。

webpack は、ワーカーがこの`src/utils/index.tsx`を import すると、ワーカーファイルが必要とする依存関係をすべてワーカーのためにひとつにバンドルする。

そのためにワーカー単体では全く関係ない`monaco-editor`モジュールを取り込むことになり、結果`window`前提のワーカーとなってしまった。

worker を webpack で扱う際には依存関係に気をつけなくてはならない。

webpack 詳しい人ならば簡単に避けられる話かもしれない。

#### ワーカーが変な依存関係をしていたら調べるといい場所

Chrome のデベロッパーツールの`source`より。

左側のペインの`page`内容が...

```explorer
top
    localhost:8080
    React DevelopperTool
    ....
your-worker-awesome-name....ts
fetchLibs_worker_ts_....ts
```

みたいに並んでいる。

調べたい worker が`fetchLibs_worker_ts...`だとしたらそいつをクリックして

```explorer

fetchLibs_worker_ts_....ts
    localhost:8080
    your-app-project-name
        node_modules
        src/worker
```

みたいにひらく。

ここの内容がそのワーカー`fetchLIbs_worker_ts..`の依存関係一覧である。

依存関係がおかしいと思ったらここの内容を調べておかしな依存関係が含まれていないかみてみよう。

## その他注意点: web event では React は更新されない

worker を使うにあたって`postMessage`でやり取りする以上`message`イベントでワーカーからのメッセージを受信することになります。

例えば以下のようにコールバック関数内で state の値を読み取っても最新の state の値を取得してくれません。

例：あるデータをリクエストして worker に取得してもらい、その結果を受信したら state を更新するコンポーネント。

リクエスト段階ではそのリクエストしたモジュールの state プロパティを`loading`に、

正常に受信出来たら`loaded`に更新する。

```TypeScript
// A component that manages worker.
import React, {
    createContext,
    useState,
    useEffect,
    useRef,
    useContext,
} from 'react';

interface iDataState {
    dataId: string;
    state: 'loading' | 'loaded';
};

const ManageWorkercomponent = ({ children }) => {
    const [data, setData] = useState<iDataState[]>([]);
    const agent = useRef<Worker>();

    // Attach message event on mount.
    useEffect(() => {
        if (window.Worker && agent.current === undefined) {
            agent.current = new Worker(
                new URL('/src/worker/your.worker.ts', import.meta.url),
                { type: 'module' }
            );
            agent.current.addEventListener('message', handleWorkerMessage);
        }

        return () => {
            if (window.Worker && agent.current) {
                agent.current.removeEventListener(
                    'message',
                    handleWorkerMessage
                );
                agent.current.terminate();
                agent.current = undefined;
            }
        };
    }, []);

    // make sure data is updated correctly.
    useEffect(() => {
        console.log('did update');
        console.log(data);
    });

    // This callback function cannot access latest reactives...
    const handleWorkerMessage = (e: MessageEvent<yourMessageType>) => {
        const { dataId, dataMap } = e.data.payload;

        console.log(`Got response of ${dataId}`);
        // THIS `data` is always empty!!!!
        console.log(data);

        // ...
    };

    const requestFetchData = (dataId: string) => {

        const updatedDeps: iDataState[] = [
            ...data,
            {
                dataId,
                state: 'loading',
            },
        ];
        setData(updatedDeps);

        if (agent.current !== undefined) {
            agent.current!.postMessage({
                dataId
            });
        }
    };

    return (
        // ...
    );
};

```

簡単な流れ：

-   `requestFetchData()`を呼び出して取得したいデータをリクエストする
-   `requestFetchData()`はリクエストされたデータをひとまず data に`state: "loading"`で登録する
-   worker にリクエストする
-   `handleWorkerMessage`が worker からのレスポンスを受信する。
-   `data`の該当要素のプロパティ`state:"loaded"`に更新する。

このとき`handleWorkerMessage`から当然最新の`data`が取得できることが期待されます。

しかし`data`は空の配列を取得します。

なぜか？

理由は、React は web event で更新されないからと、`message`イベントのコールバック関数は、worker に`addEventListener('message')`をアタッチした時点の state の値しか読み取れなくなるからです。

つまり、

```TypeScript
const [data, setData] = useState<iDataState[]>([]);
```

マウント時にイベントリスナを Worker へアタッチしたので、その時点では state `data`の値は初期値の空配列です。

そしてイベントリスナをアタッチするとその時点の state `data`にしかアクセスできなくなるため、いくら他で`data`を更新しようとも常にアタッチ時の`data`を取得することになるのです。

厄介なのが、setState 関数はアタッチ時の state と最新の state 両方に影響できるみたいで両方更新されます。

そのため、この場合であれば、別の場所で`data`が更新されていても`message`イベントのコールバックから setState すると空配列に対する更新が最新の state へ上書きされてしまうのです。

ということでイベントリスナをつけるとその時点の state だけを参照してしまうということがわかりました。

#### 解決策

次の選択肢から選び取ることになります。

1. useState を使うのをやめて useRef で管理する（参照を持たせる）。
2. 毎レンダリングで`addEventListener`を付け替える。

1 の方法を採用するか否かは、値の更新が再レンダリングを起こすか起こさないかを天秤にかけて決めることになります。

`useRef`の利用ならば ref の参照を参照するだけになるので常にその最新の値をコールバック関数からでも追跡できます。ただし`useRef`の値の更新は再レンダリングを起こしません。

```diff

const ManageWorkercomponent = ({ children }) => {
-   const [data, setData] = useState<iDataState[]>([]);
+   const data = useRef<iDataState[]>([]);
    // ...
}
```

２の方法は再レンダリングをやたら引き起こしたくない場合

```TypeScript
// 先のコードに追加する。
    useEffect(() => {
        if (window.Worker && agent.current !== undefined) {
            agent.current.addEventListener('message', handleWorkerMessage);
        }
        return () => {
            if (window.Worker && agent.current !== undefined) {
                agent.current.removeEventListener(
                    'message',
                    handleWorkerMessage
                );
            }
        };
    }, [data]);
```

state `data`の更新のたびにイベントリスナを再度アタッチします。こうすることで常に最新の値にアクセスできるようになります。

ということで React の理を web イベントのコールバックに含める場合は上記の工夫が必須となります。

## これまでの話をまとめて webworker のカスタムフックを作る

汎用的なものではなく先の話に特化したフックです。

データがロード済かどうかを管理する state `data`と、
データの中身を保存しておく`dataMap`を扱い、

worker とやり取りして２つの値を更新しその値を返します。

```TypeScript
import React, { useState, useEffect, useRef } from "react"


export const useFetchDataWorker = (): [
    iData, Map<string, string> | undefined, (moduleName: string, version: string) => void
] => {
    const worker = useRef<Worker>();
    const [data, setData] = useState<iData[]>([]);
    const dataMap = useRef<Map<string, string>>(new Map<string, string>());

    useEffect(() => {
        if (window.Worker && agent.current === undefined) {
            agent.current = new Worker(
                new URL('/src/worker/your.worker.ts', import.meta.url),
                { type: 'module' }
            );
            agent.current.addEventListener('message', handleWorkerMessage);
        }

        return () => {
            if (window.Worker && agent.current) {
                agent.current.removeEventListener(
                    'message',
                    handleWorkerMessage
                );
                agent.current.terminate();
                agent.current = undefined;
            }
        };
    }, []);

    // `data`の毎度更新のたびにイベントリスナを付け替える
    useEffect(() => {
        if (window.Worker && agent.current === undefined) {
            agent.current.addEventListener('message', handleWorkerMessage);
        }
        return () => {
            if (window.Worker && agent.current) {
                agent.current.removeEventListener(
                    'message',
                    handleWorkerMessage
                );
            }
        };
    }, [data]);

    // 毎度イベントリスナを付け替えするのでそのままsetStateして問題ない
    const handleWorkerMessage = (event: MessageEvent<yourMessageType>) => {
        const { dataId, dataMap } = e.data;

        // Update dependencies, dependencyMap...
        // 配列stateの更新方法は公式を見てください...
    }

    const requestFetchLibrary = (dataId: string) => {
        setData([
            ...data, { dataId, state: "loading" }
        ]);
        if(agent.current !== undefined) {
            agent.current.postMessage({
                dataId
            });
        }
    }

    return [data, dataMap.current, requestFetchLibrary];
}
```

useRef で参照している値（dataMap）を返しており、更新されていない値を渡す可能性があるように見えますが、
`dataMap`の更新は常に`data`の更新と同時に実施するため、再レンダリングをトリガーし両値の更新が担保されます。

## まとめ

-   関数コンポーネントで worker インスタンスは`useRef`を使って保持すること。
-   もしくは class コンポーネントの field として保持すること
-   webpack でバンドルされるライブラリのグローバルスコープが`DedicatedWorkerGlobalScope`以外にならないか確認すること
-   Reactive な値を web イベントハンドラの中で使う場合、例えば`useState`の代わりに`useRef`を使うか、イベントリスナの付け替えが必須である

## 参考

https://stackoverflow.com/questions/60540985/react-usestate-doesnt-update-in-window-events
