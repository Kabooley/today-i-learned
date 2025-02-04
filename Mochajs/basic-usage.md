# Mocha.js Basic usage


## Browser Test

#### 基本的な構成

```TypeScript
import 'mocha/mocha';
import * as chai from 'chai';
import { reportBrowserTest } from './utils/reportBrowserTest';

const { assert } = chai;

mocha.setup({
  ui: 'tdd',
  rootHooks: {
    afterAll() {
      // return promise可能
    },
    async beforeAll() {
        // async/await可能
    },
  },
    // 各testは通常5秒経過でtimeout
  timeout: 30000,
});

(function() {
    suite('test suite', () => {
        test('test', async function() {
            // ...
            assert.strictEqual(value, true);
        })
    })

    const runner = mocha.run();
    runner.on('suite', function(e: Mocha.Suite) {
        // ...
    });
    runner.on('test', function(e: Mocha.Test) {
        // ...
    });
})()
```

- アロー関数は使わないよう公式が指摘している
- `const runner = mocha.run()`のrunnerは`runner.on('suite',)`のようにテストのイベントを取得できるのでテスト状況をモニタ出来る
- rootHooks（beforeAll, beforeEach, afterEach, afterAll）を非同期処理するなら、return Promiseにするか、async beforeEach()のようにする

- mochaのtestが記述された上記のTypeScriptコードをJavaScriptへトランスパイルして、htmlファイルがそのJavaScriptコードをscriptタグで呼び出すなり囲うなりすることでブラウザテストが実行できる