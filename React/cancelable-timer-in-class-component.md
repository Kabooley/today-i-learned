# Class component with cancelable timer

キャンセル可能な遅延関数を class component でつかえるようにする

## setTimeout と clearTimeout でキャンセル可能な遅延関数を実装したコード

```TypeScript
import React from 'react';

interface iProps {}
interface iState {
  value: string;
  readOnlyValue: string;
}

const $FiveSec = 5000;
const $ThreeSec = 3000;
const $OneSec = 1000;

class TestCalcelableTimer extends React.Component<iProps, iState> {
  _timer: any | null;
  state = {
    value: 'asshole',
    readOnlyValue: '',
  };
  constructor(props: iProps) {
    super(props);
    this.state = {
      value: 'asshole',
      readOnlyValue: '',
    };
    this._timer = null;
    this._handleChangeEvent = this._handleChangeEvent.bind(this);
    this._handleDebouncedChange = this._handleDebouncedChange.bind(this);
    this._handleDebouncedChange = this._handleDebouncedChange.bind(this);
  }

  _handleChangeEvent(e: React.ChangeEvent<HTMLInputElement>) {
    e.stopPropagation();
    e.preventDefault();
    this.setState({ value: e.currentTarget.value });

    console.log(`change event: ${e.currentTarget.value}`);

    const temporary = e.currentTarget.value;

    clearTimeout(this._timer);
    this._timer = setTimeout(() => {
      // timerのコールバックの中では既にイベントオブジェクトは失われている
      // なのでe.currentTarget.valueはundefinedである
      // console.log(`timer: ${e.currentTarget.value}`);

      // 現在の最新のstateにはアクセスできるみたい
      console.log(`timer: ${this.state.value}`);
      // 先のイベントの値は変数に渡しておくとundefinedにならずに記憶できる模様
      console.log(`timer: ${temporary}`);
      this._handleDebouncedChange(temporary);
    }, $ThreeSec);
  }

  _handleDebouncedChange(value: string) {
    console.log(`debounced: ${value}`);
    console.log(`curent state value: ${this.state.value}`);

    this.setState({ readOnlyValue: value });
  }

  render() {
    return (
      <div>
        <div>
          <input
            type="text"
            value={this.state.value}
            onChange={this._handleChangeEvent}
          />
        </div>
        <div>
          <span>{this.state.readOnlyValue}</span>
        </div>
      </div>
    );
  }
}

export default TestCalcelableTimer;
```

イベントオブジェクトはタイマーのコールバックに渡しても実行時にはすでに失われているので変数に渡しておくなどする。

class コンポーネントでは WEBAPI のタイマー関数のコールバク関数から state を呼び出しても最新の値にアクセスできる。

## lodash.debounce を使ったキャンセル可能な遅延関数を実装したコード

```TypeScript
import React from 'react';
import debounce from 'lodash.debounce';
import type { DebouncedFunc } from 'lodash-es';

interface iProps {}
interface iState {
  value: string;
  readOnlyValue: string;
}

const $FiveSec = 5000;
const $ThreeSec = 3000;
const $OneSec = 1000;

class TestCalcelableDebounce extends React.Component<iProps, iState> {
  _timer: any | null;
  state = {
    value: 'asshole',
    readOnlyValue: '',
  };
  _debounceChangeEvent: DebouncedFunc<(v: string) => void>;
  constructor(props: iProps) {
    super(props);
    this.state = {
      value: 'asshole',
      readOnlyValue: '',
    };
    this._timer = null;
    this._handleChangeEvent = this._handleChangeEvent.bind(this);
    this._handleDebouncedChange = this._handleDebouncedChange.bind(this);
    this._debounceChangeEvent = debounce(
      this._handleDebouncedChange,
      $ThreeSec
    );
  }

  _handleChangeEvent(e: React.ChangeEvent<HTMLInputElement>) {
    e.stopPropagation();
    e.preventDefault();
    this.setState({ value: e.currentTarget.value });

    console.log(`change event: ${e.currentTarget.value}`);

    // なんかキャンセルを呼び出さなくてもかってにキャンセルしてくれているみたい...
    this._debounceChangeEvent.cancel();
    this._debounceChangeEvent(e.currentTarget.value);
  }

  _handleDebouncedChange(value: string) {
    console.log(`debounced: ${value}`);

    this.setState({ readOnlyValue: value });
  }

  render() {
    console.log('rendering');

    return (
      <div>
        <div>
          <input
            type="text"
            value={this.state.value}
            onChange={this._handleChangeEvent}
          />
        </div>
        <div>
          <span>{this.state.readOnlyValue}</span>
        </div>
      </div>
    );
  }
}

export default TestCalcelableDebounce;
```

なんか明示的に`DebounceFunc.cancel()`を呼び出さなくても、タイマーが切れた最後の呼び出しのみコールバックを実行してくれているみたい。

## Installed library

```bash
npm i lodash.debounce lodash-es
npm i --save-dev @types/lodash.debounce @types/lodash.debounce
```

lodash-es を install しているのは、その型情報を import する必要があるから。

@types/lodash.debounce を import しても型情報は import できない。

https://github.com/DefinitelyTyped/DefinitelyTyped/issues/50200

## lodash-es は lodash とどう違うのか

バンドラが Tree Shaking の効果を発揮できるかどうか。

lodash は CommonJS で書かれている

lodash-es は ESM で書かれている

そのため、バンドらを使うときに tree-shaking 機能が ESM 記法で書かれている方のファイルには発揮される

そのため

lodash は tree-shaking できない、一方 lodash-es は tree-shaking ができるので、lodash 全体がバンドルされることはない
