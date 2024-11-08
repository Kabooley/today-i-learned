# Test timers

## Vitestでweb apiのtimer apiを使う関数をテストするには

現実の時間通りにテストを実行するといったいどの位待つことになるかわからない場合もあるときはmockしろとのこと。


例えば～分とか～時間とか待つ場合。

`useFakeTimers()`を使えば指定の時間経過した体で即タイマーを終わらせる。

timerをテストするときは必ず`useFakeTimers()`を使えという話ではないことに注意。

#### 検証コード

```TypeScript
import {
    vi,
    describe,
    test,
    expect,
    assert,
    beforeEach,
    afterEach,
} from 'vitest';

describe('Test repeatPromise.ts', () => {
    let counter = 0;
    let timerId: any;

    beforeEach(() => {
        counter = 0;
        timerId = null;
        vi.useFakeTimers();
    });
    afterEach(() => {
        vi.restoreAllMocks();
    });

    const mockIntervalCallback = vi.fn(() => {
        console.log('counter: ', counter);
        if (counter === 5) {
            clearInterval(timerId);
        }
        counter++;
    });

    test('make sure the way to test timer with useFakeTimers()', () => {
        const timer = (cb: () => void) => {
            return setInterval(cb, 500);
        };
        timerId = timer(mockIntervalCallback);
        vi.runAllTimers();
        expect(mockIntervalCallback).toHaveBeenCalledTimes(6);
    });
});

```

#### 参考

https://vitest.dev/guide/mocking#timers