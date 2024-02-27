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
$ git commit -m "Feat: WIP Implementing form.ts 1"
$ git commit -m "Feat: WIP Implementing form.ts 2"
$ git commit -m "Feat: WIP Implementing form.ts 3"
$ git commit -m "Feat: WIP Implementing form.ts 4"
$ git commit -m "Feat: Done Implementing form.ts 5"

# `git rebase -i `に続けるのは圧縮したコミットをコミットする対象のブランチ
$ git rebase -i development
# エディタが開いて変更内容を選択するよう求められる
# 統合したいコミットの接頭辞をsquashにする
# コミット内容の確認が促される
$ git switch development
$ git merge feat_form
# これでfeat_form上のコミット5つがひとつのコミットとしてdevelopmentへマージされた。
```

以下のようなテキストがエディタに現れる。

接頭辞`pick`を後半に書かれている通りの任意の接頭辞に変更することでその通りの変更が施されるという感じ。

e.g. `pick e8131e6 Feat: WIP Implementing form.ts 2` -> `squash e8131e6 Feat: WIP Implementing form.ts 2`

pick のコミットに squash のコミットが統合される。

```bash
pick acbce48 Feat: WIP Implementing form.ts 1
pick e8131e6 Feat: WIP Implementing form.ts 2
pick 9bce99b Feat: WIP Implementing form.ts 3
pick 924091f Feat: WIP Implementing form.ts 4
pick d72f328 Feat: WIP Implementing form.ts 5

# Rebase 3c57e96..d72f328 onto 924091f (5 commands)
#
# Commands:
# p, pick <commit> = use commit
# r, reword <commit> = use commit, but edit the commit message
# e, edit <commit> = use commit, but stop for amending
# s, squash <commit> = use commit, but meld into previous commit
# f, fixup <commit> = like "squash", but discard this commit's log message
# x, exec <command> = run command (the rest of the line) using shell
# b, break = stop here (continue rebase later with 'git rebase --continue')
# d, drop <commit> = remove commit
# l, label <label> = label current HEAD with a name
# t, reset <label> = reset HEAD to a label
# m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
# .       create a merge commit using the original merge commit's
# .       message (or the oneline, if no original merge commit was
# .       specified). Use -c <commit> to reword the commit message.
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out

```

> 複数のコミットをまとめて圧縮したい場合は、 git rebase -i を使用できます。
> feature_branch を使用している場合は、 git rebase -i main を実行します。これによりエディター ウィンドウが開き、pick という接頭辞が付いた多数のコミットがリストされます。最初の変更を除くすべての変更をスカッシュすることができます。これにより、Git はそれらの変更をすべて保持し、最初のコミットにスカッシュするように指示されます。それが完了したら、main へチェックアウトし、feature_branch をマージします。

rebase から merge したあとの git log

```bash
git log --oneline
08710a6 (HEAD -> main, test_git_rebase_on_squashmd) Feat: WIP Implementing form.ts 1
# feat_formブランチを切る前のdevelopmentブランチの最後のコミット
3c57e96 Docs: WIP squash.md 1
```

## 以下、コマンドのテストのためにこのファイルを変更した内容

#### 変更その１

変更その１を加えました。

#### 変更その２

変更その２を加えました。

#### 変更その３

変更その３を加えました。

#### 変更その４

変更その４を加えました。

#### 変更その５

変更その５を加えました。

## git rebase -i 前の変更その１

## git rebase -i 前の変更その 2

## git rebase -i 前の変更その 3

## git rebase -i 前の変更その 4

## git rebase -i 前の変更その 5
