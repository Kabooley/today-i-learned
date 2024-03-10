# `git checkout`

## ワーキングツリーの編集内容を取り消す場合

ステージングする前。

別ブランチに移ってから作業に取り掛かりたかったが、うっかりブランチを切る前に`develop`ブランチで編集をしてしまった...

そんなとき。

`git checkout .`と`git clean -f`を実行する。

```bash
$ git branch
* develop
  feat_counter
  main

# ステージングはしていない。ファイルの編集と新規ファイルを追加したところ。
$ git status
On branch development
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   src/XXXXXXXX/YYYYYYY.tsx
        modified:   src/XXXXXXXX/ZZZZZZZ.tsx

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        docs/feat_counter.md

no changes added to commit (use "git add" and/or "git commit -a")
# checkout で編集作業をなかったことに。
$ git checkout .
Updated 2 paths from the index
# checkoutだけだと新規に追加したファイルはそのまま残るので、
$ git status
On branch development
Untracked files:
  (use "git add <file>..." to include in what will be committed)
        docs/feat_conter.md

nothing added to commit but untracked files present (use "git add" to track)
# git cleanでuntracked fileをなかったことにする
$ git clean -f
Removing docs/feat_counter.md
# ワーキングツリーが編集する前の状態に戻った
$ git status
On branch development
nothing to commit, working tree clean

```
