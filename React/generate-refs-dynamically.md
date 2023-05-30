# Generate refs dynamically

動的に ref を増やしたり減らして dom へ渡したいとき。

## 参考

https://stackoverflow.com/questions/55995760/how-to-add-refs-dynamically-with-react-hooks

## 実際に使ってみた

`changeTab`はクリックされた span 要素の className を変更する機能を持つ。
`changeTab`は動的に生成される span 要素の各 DOM 参照を必要とする。

```TypeScript
const Tabs = () => {
    const [activeTab, setActiveTab] = useState<string>(files['react-typescript'].path);
    // div.tabAreaを参照する
    const _refTabArea = useRef<HTMLDivElement>(null);
    // filesのプロパティの増減に応じてrefオブジェクトを生成するref
    const _refTabs = useRef(
        Object.keys(files).map(() => React.createRef()as LegacyRef<HTMLSpanElement>)
    );

    const changeTab = (selectedTabNode: HTMLSpanElement, desiredModelId: string) => {
        // 一旦すべてのtabのclassNameを'tab'にする
        for (var i = 0; i < _refTabArea.current!.childNodes.length; i++) {
            var child: iJSXNode = _refTabArea.current!.childNodes[i];
            if (/tab/.test(child.className!)) {
                child.className = 'tab';
            }
        }
        // 選択されたtabのみclassName='tab active'にする
        selectedTabNode.className = 'tab active';
    };


    return (
        <div className="tabArea" ref={_refTabArea}>
            {
                Object.keys(files).map((key, index) => {
                    const file: iFile = files[key];
                        return (
                            <span
                                className={file.path === activeTab ? "tab active": "tab"}
                                ref={_refTabs.current[index]}
                                onClick={() => changeTab(
                                    // 各refオブジェクトを参照するとき
                                    _refTabs.current[index].current!,
                                    file.path
                                )}
                            >
                                {file.path}
                            </span>
                        );
                })
            }
        </div>
    );
};
```

-   `_refTabs`は`React.RefObject<HTMLSpanElement>[]`を参照する
-   `_refTabs.current`配列の各要素は ref オブジェクトである。

ref オブジェクトが指すものにアクセスするのにも current が必要なので
`_refTabs.current[index].current`というアクセス方法になる。

#### test @codesandbox

```TypeScript
import React, { useEffect, useRef, useState } from "react";

export interface iFiles {
  [path: string]: iFile;
}

export interface iFile {
  path: string;
  language: string;
  value: string;
}

export const files: iFiles = {
  javascript: {
    path: "/main.js",
    language: "javascript",
    value: ``
  },
  typescript: {
    path: "/main.ts",
    language: "typescript",
    value: ``
  },
  react: {
    path: "/main.jsx",
    language: "javascript",
    value: ``
  },
  "react-typescript": {
    path: "/main.tsx",
    language: "typescript",
    value: `import { createRoot } from 'react-dom/client';\r\nimport React from 'react';\r\nimport 'bulma/css/bulma.css';\r\n\r\nconst App = () => {\r\n    return (\r\n        <div className=\"container\">\r\n          <span>REACT</span>\r\n        </div>\r\n    );\r\n};\r\n\r\nconst root = createRoot(document.getElementById('root'));\r\nroot.render(<App />);`
  }
};

interface iJSXNode extends Node {
  className?: string;
}


const Tabs = () => {
  const _refTabArea = useRef<HTMLDivElement>(null);
  const [activeTab, setActiveTab] = useState<string>(
    files["react-typescript"].path
  );
  const _refTabs = useRef(
    Object.keys(files).map(() => React.createRef<HTMLSpanElement>())
  );

  useEffect(() => {
    console.log(`activeTab: ${activeTab}`);
    console.log(_refTabs.current);
  });

  const changeTab = (
    selectedTabNode: HTMLSpanElement,
    desiredFilePath: string
  ) => {
    // DEBUG:
    console.log("[changeTab]");
    console.log("selectedTabNode");
    console.log(selectedTabNode);
    console.log(`desiredFilePath: ${desiredFilePath}`);
    // 一旦すべてのtabのclassNameを'tab'にする
    for (var i = 0; i < _refTabArea.current!.childNodes.length; i++) {
      var child: iJSXNode = _refTabArea.current!.childNodes[i];
      if (/tab/.test(child.className!)) {
        child.className = "tab";
      }
    }
    // 選択されたtabのみclassName='tab active'にする
    selectedTabNode.className = "tab active";
    setActiveTab(desiredFilePath);
  };

  return (
    <div className="tab-area" ref={_refTabArea}>
      {Object.keys(files).map((key, index) => {
        const file: iFile = files[key];
        return (
          <span
            className={file.path === activeTab ? "tab active" : "tab"}
            ref={_refTabs.current[index]}
            onClick={() =>
              changeTab(_refTabs.current[index].current!, file.path)
            }
          >
            {file.path}
          </span>
        );
      })}
    </div>
  );
};

export default Tabs;
```

```css
.App {
    font-family: sans-serif;
    text-align: center;
}

.tab-area {
    position: absolute;
    box-sizing: border-box;
    top: 0;
    left: 0;
    height: 20px;
    box-sizing: border-box;
    border-bottom: 1px solid #999;
}

.tab {
    height: 20px;
    line-height: 20px;
    box-sizing: border-box;
    color: #999;
    padding: 0 8px;
    border: 1px solid #999;
    border-bottom: 0;
    cursor: pointer;
    float: left;
}

.tab.active {
    color: black;
    border-bottom: 1px solid white;
    background-color: violet;
}
```
