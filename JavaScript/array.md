# Array

## 2 つの配列を比較して片方に存在しない要素を見つけたいとき

`Array.indexOf(element) === -1`で`element`が`Array`に存在しないので、

その要素からなる配列を`TargetArray.filter()`で生成する。

```TypeScript
/*****************************************************************
 * Compare two array to find the element which is not exist in an array.
 *
 * Ref:
 * https://stackoverflow.com/a/44918208/22007575
 * **************************************************************/

import { describe, expect, test, it } from '@jest/globals';


/***
 * `target`配列には存在するけれど`compareWith`配列に存在しない要素を探して
 * その存在しない要素からなる配列を返す。
 *
 * @param {Array<T>} target - `compareWith`配列に存在しない要素を持つ配列
 * @param {Array<T>} compareWith - 比較対象の配列
 * @return {Array<T>}
 * */
const getElementNotExistInArray = <T>(target: T[], compareWith: T[]): T[] => {
  return target.filter((t) => compareWith.indexOf(t) === -1);
};

const Test = {
  target: [
    'file:///node_modules/@types/prop-types/package.json',
    'file:///node_modules/@types/react-dom/client.d.ts',
    'file:///node_modules/@types/react-dom/experimental.d.ts',
    'file:///node_modules/@types/react-dom/index.d.ts',
    'file:///node_modules/@types/react-dom/next.d.ts',
    'file:///node_modules/@types/react-dom/package.json',
    'file:///node_modules/@types/react-dom/server.d.ts',
    'file:///node_modules/@types/react-dom/test-utils/index.d.ts',
    'file:///node_modules/@types/react/experimental.d.ts',
  ],
  compareWith: [
    'file:///node_modules/@types/prop-types/package.json',
    'file:///node_modules/@types/react-dom/client.d.ts',
    'file:///node_modules/@types/react-dom/experimental.d.ts',
    'file:///node_modules/@types/react-dom/index.d.ts',
    'file:///node_modules/@types/react-dom/next.d.ts',
    'file:///node_modules/@types/react-dom/package.json',
    'file:///node_modules/@types/react-dom/server.d.ts',
    'file:///node_modules/@types/react-dom/test-utils/index.d.ts',
  ],
  shouldBe: ['file:///node_modules/@types/react/experimental.d.ts'],
};

describe('Test getElementNotExistInArray', () => {
  test('Should return `experimental.d.ts`', () => {
    expect(getElementNotExistInArray<string>(Test.target, Test.compareWith)).toEqual(
      Test.shouldBe
    );
  });
});

```
