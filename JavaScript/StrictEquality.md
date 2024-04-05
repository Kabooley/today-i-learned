# 厳密な等価比較

厳密な比較は、オブジェクトや配列同士の比較の場合、値同士の比較ではなく参照の比較を行う

詳しくは以下のテストの最後の方に。

## テスト

```TypeScript
import { describe, expect, test, it } from '@jest/globals';

interface File {
  _path: string;
  _value: string;
  _language: string;
  _isFolder: boolean;
  _selected: boolean;
  _opening: boolean;
  _tabIndex: number | null;
}

const operand1: File[] = [
  {
    _path: 'package.json',
    _value:
      '{\n  "name": "react-typescript",\n  "version": "1.0.0",\n  "description": "React and TypeScript example starter project",\n  "keywords": [\n    "typescript",\n    "react",\n    "starter"\n  ],\n  "main": "src/index.tsx",\n  "dependencies": {\n    "@types/react": "18.0.25",\n    "@types/react-dom": "18.0.9",\n    "react": "18.2.0",\n    "react-dom": "18.2.0",\n    "react-scripts": "5.0.1",\n    "typescript": "4.4.2"\n  },\n  "devDependencies": {},\n  "scripts": {\n    "start": "react-scripts start",\n    "build": "react-scripts build",\n    "test": "react-scripts test --env=jsdom",\n    "eject": "react-scripts eject"\n  },\n  "browserslist": [\n    ">0.2%",\n    "not dead",\n    "not ie <= 11",\n    "not op_mini all"\n  ]\n}',
    _language: 'json',
    _isFolder: false,
    _selected: false,
    _opening: false,
    _tabIndex: null,
  },
  {
    _path: 'public/index.html',
    _value:
      '\n<!DOCTYPE html>\n<html>\n  <head>\n    <meta charset="utf-8" />\n    <title>React TypeScript</title>\n  </head>\n  <body>\n    <div id="root"></div>\n  </body>\n</html>',
    _language: 'html',
    _isFolder: false,
    _selected: false,
    _opening: false,
    _tabIndex: null,
  },
  {
    _path: 'src/App.tsx',
    _value:
      '\nimport React from \'react\';\nimport "./styles.css";\n\nexport default function App(): React.JSX.Element {\n  return (\n    <div className="App">\n      <h1>Hello CodeSandbox</h1>\n      <h2>Start editing to see some magic happen!</h2>\n    </div>\n  );\n};\n      ',
    _language: 'typescript',
    _isFolder: false,
    _selected: true,
    _opening: true,
    _tabIndex: null,
  },
  {
    _path: 'src/index.tsx',
    _value:
      '\nimport React from "react";\nimport ReactDOM from "react-dom/client";\nimport App from "./App";\n\nconst rootElement = document.getElementById("root");\nif(rootElement) {\n  const root = ReactDOM.createRoot(rootElement);\n\n  root.render(\n    <React.StrictMode>\n      <App />\n    </React.StrictMode>\n  );   \n}',
    _language: 'typescript',
    _isFolder: false,
    _selected: false,
    _opening: false,
    _tabIndex: null,
  },
  {
    _path: 'src/styles.css',
    _value:
      '.App {\n        font-family: sans-serif;\n        text-align: center;\n      }\n      ',
    _language: 'css',
    _isFolder: false,
    _selected: false,
    _opening: false,
    _tabIndex: null,
  },
  {
    _path: 'tsconfig.json',
    _value:
      '{\n      "include": [\n          "./src/**/*"\n      ],\n      "compilerOptions": {\n          "strict": true,\n          "esModuleInterop": true,\n          "lib": [\n              "dom",\n              "es2015"\n          ],\n          "jsx": "react-jsx"\n      }\n  }',
    _language: 'json',
    _isFolder: false,
    _selected: false,
    _opening: false,
    _tabIndex: null,
  },
];

// operand1と全く同じ内容の配列
const operandSameAs1: File[] = [
  {
    _path: 'package.json',
    _value:
      '{\n  "name": "react-typescript",\n  "version": "1.0.0",\n  "description": "React and TypeScript example starter project",\n  "keywords": [\n    "typescript",\n    "react",\n    "starter"\n  ],\n  "main": "src/index.tsx",\n  "dependencies": {\n    "@types/react": "18.0.25",\n    "@types/react-dom": "18.0.9",\n    "react": "18.2.0",\n    "react-dom": "18.2.0",\n    "react-scripts": "5.0.1",\n    "typescript": "4.4.2"\n  },\n  "devDependencies": {},\n  "scripts": {\n    "start": "react-scripts start",\n    "build": "react-scripts build",\n    "test": "react-scripts test --env=jsdom",\n    "eject": "react-scripts eject"\n  },\n  "browserslist": [\n    ">0.2%",\n    "not dead",\n    "not ie <= 11",\n    "not op_mini all"\n  ]\n}',
    _language: 'json',
    _isFolder: false,
    _selected: false,
    _opening: false,
    _tabIndex: null,
  },
  {
    _path: 'public/index.html',
    _value:
      '\n<!DOCTYPE html>\n<html>\n  <head>\n    <meta charset="utf-8" />\n    <title>React TypeScript</title>\n  </head>\n  <body>\n    <div id="root"></div>\n  </body>\n</html>',
    _language: 'html',
    _isFolder: false,
    _selected: false,
    _opening: false,
    _tabIndex: null,
  },
  {
    _path: 'src/App.tsx',
    _value:
      '\nimport React from \'react\';\nimport "./styles.css";\n\nexport default function App(): React.JSX.Element {\n  return (\n    <div className="App">\n      <h1>Hello CodeSandbox</h1>\n      <h2>Start editing to see some magic happen!</h2>\n    </div>\n  );\n};\n      ',
    _language: 'typescript',
    _isFolder: false,
    _selected: true,
    _opening: true,
    _tabIndex: null,
  },
  {
    _path: 'src/index.tsx',
    _value:
      '\nimport React from "react";\nimport ReactDOM from "react-dom/client";\nimport App from "./App";\n\nconst rootElement = document.getElementById("root");\nif(rootElement) {\n  const root = ReactDOM.createRoot(rootElement);\n\n  root.render(\n    <React.StrictMode>\n      <App />\n    </React.StrictMode>\n  );   \n}',
    _language: 'typescript',
    _isFolder: false,
    _selected: false,
    _opening: false,
    _tabIndex: null,
  },
  {
    _path: 'src/styles.css',
    _value:
      '.App {\n        font-family: sans-serif;\n        text-align: center;\n      }\n      ',
    _language: 'css',
    _isFolder: false,
    _selected: false,
    _opening: false,
    _tabIndex: null,
  },
  {
    _path: 'tsconfig.json',
    _value:
      '{\n      "include": [\n          "./src/**/*"\n      ],\n      "compilerOptions": {\n          "strict": true,\n          "esModuleInterop": true,\n          "lib": [\n              "dom",\n              "es2015"\n          ],\n          "jsx": "react-jsx"\n      }\n  }',
    _language: 'json',
    _isFolder: false,
    _selected: false,
    _opening: false,
    _tabIndex: null,
  },
];

/**
 * 厳密な比較 `a === b`：
 *
 * オブジェクト同士の比較の時、厳密な比較は値同士の比較ではなく、
 * 同じ参照を指しているかを比較する
 *
 * 例：オブジェクト配列同士の比較を行うとして、それが真となるのはどんなときか？
 * 答え：まったく同じ参照であったときに真となる
 *
 * つまり、配列の変数を宣言した場合、その変数にはメモリ番号が与えられる
 * つまり配列変数はそのメモリを参照しているポインタである
 * 配列同士を比較した場合、この参照しているメモリ番号が同じであったときに真となるのである。
 * その配列の各要素が一致しているかどうかの比較ではない
 * **/
describe('Test Strict equality', () => {
  // 1. 内容が同じでも、参照しているオブジェクトが異なる場合、オブジェクト同士の比較は偽となる
  test('should return false unless each refer to same objects.', () => {
    const john1 = { name: 'John', age: 30 };
    const john2 = { name: 'John', age: 30 };
    expect(john1 === john2).toBe(false);
  });
  // 厳密な比較でなくても参照が同じであるかどうか比較される
  test('should return false unless each refer to same objects.', () => {
    const john1 = { name: 'John', age: 30 };
    const john2 = { name: 'John', age: 30 };
    expect(john1 == john2).toBe(false);
  });
  // 2. 配列の内容は同じだけど、要素の並びが異なる場合、偽となる
  test('Should be false if same contents but order is different.', () => {
    const order1 = ['a', 'b', 'c'];
    const order2 = ['a', 'c', 'b'];
    expect(order1 === order2).toBe(false);
  });
  // 3. オブジェクト配列同士の比較はオブジェクト同士の比較と同様で、同じオブジェクトを参照していないと偽となる
  test('should return false even though two array contents same objects', () => {
    expect(operand1 === operandSameAs1).toBe(false);
  });
  // では同じ参照を持ったオブジェクトからなる配列同士の比較なら真になるかというとそうではない
  // 以下はarr1の参照とarr2の参照が異なるためである
  // 要はポインタ同士の比較みたいなもので、どのメモリ番号を指しているかという比較を行っている
  test('should return false even though two array contents same objects', () => {
    const a = {};
    const b = {};
    const arr1 = [a, b];
    const arr2 = [a, b];
    expect(arr1 === arr2).toBe(false);
  });
});
```