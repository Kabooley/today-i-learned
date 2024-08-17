# git-diff

https://stackoverflow.com/questions/9834689/how-do-i-see-the-differences-between-two-branches

https://git-scm.com/docs/git-diff

`git diff <commit>..<commit>`

```bash
$ git branch
*   development
    feat_somefeat
$ git log --oneline
5cbfh4 (HEAD--> XXXX) XXXXX

$ git switch feat_somefeat
$ git log --oneline
6hbfh2 (HEAD--> XXXX) XXXXX

$ git switch development
$ git diff 5cbfh4..6hbfh2
diff --git a/codesnadbox-test-resizable/index.html b/codesnadbox-test-resizable/index.html
deleted file mode 100644
index ef6efdc..0000000
--- a/codesnadbox-test-resizable/index.html
+++ /dev/null
@@ -1,51 +0,0 @@
-<!DOCTYPE html>
-<html lang="en">
-    <head>
-        <meta charset="UTF-8" />
-        <meta http-equiv="X-UA-Compatible" content="IE=edge" />
# brahbrahbrah...


```

## ２つのコミット間の差分の確認

参考：

https://qiita.com/yuya_presto/items/ef199e08021dea777715

シナリオ：`feat/responsive-layout`ブランチには、この後 merge する予定である development ブランチにコミットされた内容より以前のコードを含んでいる可能性があって、そのために development ブランチのコミット内容を消さないように 2 つのコミット間の差分を出力したい

```bash
$ git branch
  development
* feat/responsive-layout
  main
  ...

$ git log --oneline
9e509ba (HEAD -> feat/responsive-layout) YYYYYYYYYYY
fe5abd7 (origin/development, development) XXXXXXXXXXX

# 以下のコマンドはブランチの差分すべてを出力して5000行くらいになったので、ファイル一つずつにする
$ git diff development..feat/responsive-layout


# 差分のあるファイル名を取得する
$ git diff development..feat/responsive-layout --name-only
# 各ファイルのブランチ間差分を出力させる
# 変更内容のコードを分析してコミットするかする

# 特定のファイルにおける2つのブランチ間の差分を出力する
# a: 古いコンテンツ、b: 新しいコンテンツ
$ git diff development..feat/responsive-layout src/App.tsx
```

```bash
# 差分のあるファイル名を取得する
$ git diff development..feat/responsive-layout --name-only
docs/feat-responsive.md
src/App.tsx
src/Layout/EditorSection.tsx
src/Layout/MainContainer.tsx
src/Layout/PaneSection.tsx
src/Layout/desktop/index.tsx
src/Layout/index.tsx
src/Layout/phone/EditorSection.tsx
src/Layout/phone/PaneSection.tsx
src/Layout/phone/index.tsx
src/Layout/tablet/EditorSection.tsx
src/Layout/tablet/PreviewSection.tsx
src/Layout/tablet/index.tsx
src/components/Header/index.tsx
src/components/Modal/index.tsx
src/components/Monaco/MonacoEditor.tsx
src/components/Pane.tsx
src/components/ScrollableElement/index.tsx
src/components/SliderPane.tsx
src/components/TabsAndActions/index.tsx
src/components/VSCodeExplorer/Workspace/index.tsx
src/components/sliderPane.css
src/constants/index.ts
src/index.html
src/sass/abstracts/_mixins.scss
src/sass/base/_base.scss
src/sass/base/_typography.scss
src/sass/components/_editor.scss
src/sass/components/_header.scss
src/sass/components/_tabsAndActions.scss
src/sass/layout/_main.scss
src/sass/main.scss
src/slices/layoutSlice.ts
src/utils/index.ts
src/utils/limitWithinRange.ts
```

あとは VSCode の Git Graph 拡張機能を使って VSCode 上に視覚的に差分を表示して比較するとよい
