# `Squash` concept

複数のコミットを一つのコミットに圧縮する考え。

たとえば、

`development`ブランチから`feat_coutner`ブランチに分岐して、

その`feat_counter`ブランチで作業を行っているうちに、

`feat_counter`ブランチでのコミットがたくさん生成されてしまったとする。

（たとえば想定の作成時間を超えて完成させることになり、いったん作業を区切る段階でここまでの作業内容をコミットした積み重ねがあったからなど）

一方、development ブランチから見たら`feat_counter`ブランチが完成したコミットだけが必要であって、

その過程のコミットは必要ないし、チームにとっては邪魔になる。

なので`feat_counter`ブランチで付き重ねたコミットを一つのコミットにまとめたいとする。

そのための方法。

## 参考

https://stackoverflow.com/a/5721879/22007575

https://stackoverflow.com/a/5201642/22007575

https://backlog.com/ja/git-tutorial/stepup/34/

https://backlog.com/ja/git-tutorial/stepup/27/

## `git merge --squash BRANCH_NAME`

`git merge --squash BRANCH_NAME`は BRANCH_NAME のすべてのコミットを一つにして現在のブランチにコミットするコマンドである。

以下、`feat_counter`ブランチ上のコミット５つ**すべて**を一つのコミットに圧縮して`development`へコミットする方法。

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
$ git merge --squash feat_counter
Updating xxxx....xxxxxxx
Fast-forward
Squash commit -- not updating HEAD
 counter.ts | 36 ++++++++++++++++++++++++++++--------
 1 file changed, 28 insertions(+), 8 deletions(-)
$ git commit
# エディタが開いてコミットメッセージを入力するよう促される。
```

## `git rebase -i`

```bash
$ git branch
* development
  main

$ git branch feat_form
$ git switch feat_form

# 以下の通り、`feat_form`上で計5つのコミットを重ねてブランチの目標を達成したとする
$ git commit -m "Feat: WIP Implementing form. 1"
$ git commit -m "Feat: WIP Implementing form. 2"
$ git commit -m "Feat: WIP Implementing form. 3"
$ git commit -m "Feat: WIP Implementing form. 4"
$ git commit -m "Feat: Done Implementing form. 5"

# `git rebase -i `に続けるのは圧縮したコミットをコミットする対象のブランチ
$ git rebase -i development
# エディタが開いて変更内容を選択するよう求められる
#
```

> 複数のコミットをまとめて圧縮したい場合は、 git rebase -i を使用できます。
> feature_branch を使用している場合は、 git rebase -i main を実行します。これによりエディター ウィンドウが開き、pick という接頭辞が付いた多数のコミットがリストされます。最初の変更を除くすべての変更をスカッシュすることができます。これにより、Git はそれらの変更をすべて保持し、最初のコミットにスカッシュするように指示されます。それが完了したら、main へチェックアウトし、feature_branch をマージします。

## 以下、コマンドのテストのためにこのファイルを変更した内容

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
