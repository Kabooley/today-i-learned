# Usage of Chrome Devtools

1 次情報：

https://developer.chrome.com/docs/devtools/

参考：

https://murashun.jp/article/performance/chrome-devtools.html#chapter-31

## 目次

[Performance](#Performance)
[Memory](#Memory)
[](#)
[](#)
[](#)

## Performance

https://developer.chrome.com/docs/devtools/overview/#performance

Web ページのアクティビティを記録して分析するのに使う。

## Memory

https://developer.chrome.com/docs/devtools/overview/#memory

https://developer.chrome.com/docs/devtools/memory-problems/

ページのパフォーマンスに影響するメモリに関する問題を調査するためにどうやって devtools を使うのかを理解する。

メモリリーク、メモリの膨張、頻度の高い GC についても。

#### タスクマネージャから

chrome menu > その他のツール > タスクマネージャ

or

shift + Esc

右クリックで開くコンテキストメニューから JavaScript のメモリ監視も含める

-   メモリ行はネイティブ・メモリを表す。DOM ノードはこのネイティブメモリに含まれる。

もしもここが増加しているなら DOM ノードが増えていると思っていい。

-   JS メモリ行は JS ヒープを表す。2 つの値を含む。

貴方が関心を寄せる値はこのライブ値です。（それとカッコ内の値です）

`4700k (3620k ライブ)`と表示されている。

ライブ値はページ上の到達可能なオブジェクトが使用しているメモリの量を表します。この数が増加している場合は、新しいオブジェクトが作成されているか、既存のオブジェクトが増加しています。

#### Timeline 記録でメモリリークを視覚化

NOTE: `Timeline`という名称は廃止されて今は`Performance`という名称である。
