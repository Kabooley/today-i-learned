# `Squash` concept

複数のコミットを一つのコミットに圧縮する考え。

たとえば、

`development`ブランチから`feat_coutner`ブランチに分岐して、

その`feat_counter`ブランチで作業を行っているうちに、

`feat_counter`ブランチでのコミットがたくさん生成されてしまったとする。

（たとえば想定の作成時間を超えて完成させることになり、いったんさぎゅうを区切る段階でここまでの作業内容をコミットした積み重ねがあったからなど）

一方、development ブランチから見たら`feat_counter`ブランチが完成したコミットだけが必要であって、

その過程のコミットは必要ないし、チームにとっては邪魔になる。

なので`feat_counter`ブランチで付き重ねたコミットを一つのコミットにまとめたいとする。

そのための方法。

## 参考

https://stackoverflow.com/a/5721879/22007575

https://stackoverflow.com/a/5201642/22007575

https://backlog.com/ja/git-tutorial/stepup/34/

https://backlog.com/ja/git-tutorial/stepup/27/

## 例

`feat_counter`ブランチ上のコミット５つを一つのコミットに圧縮する方法。

```bash
$ git branch
* development
  main

$ git branch feat_counter
$ git switch feat_counter

# 以下の通り、`feat_counter`上で計5つのコミットを重ねてブランチの目標を達成したとする
$ git commit -m "Feat: WIP Implementing counter. 1"
$ git commit -m "Feat: WIP Implementing counter. 2"
$ git commit -m "Feat: WIP Implementing counter. 3"
$ git commit -m "Feat: WIP Implementing counter. 4"
$ git commit -m "Feat: Done Implementing counter. 5"

$ git switch development
$
$
$
$
$
$
```

## 変更その１

変更その１を加えました。

## 変更その２

変更その２を加えました。

## 変更その３

変更その３を加えました。

## 変更その４

変更その４を加えました。

## 変更その５

変更その５を加えました。
