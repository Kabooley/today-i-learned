# props に基づいて子コンポーネントの state を更新させたいとき

props の値に基づいて、**現在の state と比較して**新しい state を計算して set したい。

その更新は可能か？違法か？を調べる。

## 結論

親コンポーネント state に元づいて決定される子コンポーネント state は持つべきでない。

-   useEffect を使って state を更新すると無限ループになるので副作用は使えない
-   useState の代わりに useMemo を使えばレンダリングにまたがって値を保持してくれるので props の値を依存関係に含めれば更新可能だが、前回の useMemo の値との比較は、useMemo はできない

回避策：

-   推奨）親 state に基づく state 管理はやめて親 state に基づいて計算可能な値にするべき
-   次善の解的な）前回の props の値を state 管理する
-   state をリフトアップして親コンポーネントが管理するべき

https://react.dev/learn/you-might-not-need-an-effect#adjusting-some-state-when-a-prop-changes

## 例

-   files の各要素のどれが`isOpening: true`であるかは、毎度親コンポーネントが決定するので子コンポーネントはわからない。
-   子コンポーネントは順番(tabOrder)を記憶し更新する
-

```JavaScript
interface iFile {
    isOpening: boolean;
};

const Tabs = ({
    files
}: {
    files: iFile[];
}) => {
    const [tabOrder, setTabOrder] = useState<string[]>([]);

    return (
        tab
    )
}
```

NOTE: 書きかけ。

## 副作用時に setState すると無限ループ

https://react.dev/learn/synchronizing-with-effects

-   副作用はレンダリング結果として実行される
-   setState はレンダリングをトリガーする

結果無限ループに陥る。

他の state に基づいて state を決定したい場合、副作用は必要ない可能性が高い。

componentDidMount でならいいかも。

componentDidUpdate ではやっちゃだめ。

## useMemo が現在の useMemo の値を参照できるか？

結論：できない。

理由は無限ループになるから。

```JavaScript
// fileのpathの順番を収める変数filesOrderを生成する...
  const filesOrder: string[] = useMemo(() => {
    const openingFiles: string[] = files
    .filter((f) => f.isOpening())
    .map((f) => f.getPath());
    if(filesOrder === undefined) return openingFiles;

    // 前回のfilesOrderの順番を反映させた順番を生成するが
    // このままだとエラーになり...
    const reordered = openingFiles.sort(funciton(a, b) {
      return filesOrder.indexOf(a) - filesOrder.indexOf(b);
    });
    return d;
  }, [files])
```

エラーを解決するには

多分 filesOrder を依存関係に含めないといけないのだけれど、

それをすると無限ループに陥る。
