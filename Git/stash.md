# git stash

`git stash`すると、作業ツリー上の変更またはステージング内容が「退避」されて作業ツリーがクリーンになる

https://www.atlassian.com/ja/git/tutorials/saving-changes/git-stash

https://git-scm.com/docs/git-stash

> 作業ディレクトリやインデックスの現在の状態を記録しておきたいけれども、きれいな作業ディレクトリに戻したいという場合に git stash を使用します。このコマンドは、ローカルで行った変更を退避させ、作業ディレクトリを HEAD のコミットにあわせたものに戻します。

> このコマンドで保存された変更は、git stash list で一覧表示し、git stash show で確認し、git stash apply で (別のコミットの上に) 復元することができます。引数なしで git stash を呼び出すと、git stash push と同じ意味になります。stash はデフォルトでは "WIP on branchname ..." と表示されますが、作成時にコマンドラインでもっとわかりやすいメッセージを指定することができます。

> 作成した最新のスタッシュは refs/stash に格納されます。古いスタッシュはこのリファレンスの reflog にあり、通常の reflog 構文で名前を付けることができます (例: stash@{0} は最近作成したスタッシュ、stash@{1} はその前のもの、stash@{2.hourset} も可能です). また，スタッシュはスタッシュインデックスだけを指定して参照することもできます（例えば，整数 n は stash@{n}と同等です）．

つまり、`git stash`すると、stash する直前までの変更はすべて「一旦どこかへ退避される」。

そして作業ツリーはクリーンな状態に戻る。

`git stash list`: 退避された(push された)変更一覧を表示する

`git stash push`: 変更を退避させる（リストに追加する）

`.git/refs/stash`: 退避された変更が保存される先

`git stash apply`:で退避一覧から作業ツリーへ変更を適用させる。

`git stash pop`: 退避一覧から変更を作業ツリーへ適用させて、退避一覧からその変更を消去させる。

`git stash drop <number>`: 退避一覧から番号を指定して一覧から stash を削除する。番号は振りなおされる。

`git stash clear`: 退避一覧からすべて削除する。

apply と pop の違いは、

apply しても退避した変更内容は退避一覧に維持されるけど

pop のほうは文字通り一覧からポップアウトされてしまう。

push 毎「ひとつ」退避リストへ追加したことになる。

挙動確認：

```bash
# 新規ファイルを作成した
$ echo "This file is for making sure how git stash works" > stash.txt

$ git status
On branch master
Untracked files:
  (use "git add <file>..." to include in what will be committed)
        stash.txt

nothing added to commit but untracked files present (use "git add" to track)

# NOTE: untracked fileはstashできない
$ git stash push
No local changes to save

# なのでgit add . してから...
$ git add .

$ git status
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   stash.txt

# stashできた
$ git stash
Saved working directory and index state WIP on master: 12c7f29 Cleaned up git-basics/

# stash listで確認できた
$ git stash list
stash@{0}: WIP on master: 12c7f29 Cleaned up git-basics/

# stashされた後なのでstash.txtはuntracked fileのままで、別の変更を加えてみた
$ echo "Second change" >> stash.txt

# 既存ファイルを変更した
$ echo "make sure how git stash works" >> initial-commit.txt

# 既存ファイルの変更&untracked fileの変更
$ git stash
Saved working directory and index state WIP on master: 12c7f29 Cleaned up git-basics/

$ git stash list
stash@{0}: WIP on master: 12c7f29 Cleaned up git-basics/
stash@{1}: WIP on master: 12c7f29 Cleaned up git-basics/

$ cat stash.txt
Second change

# stash apply が untracked file の変更をもたらす場合、stash applyは拒否される
$ git stash apply 1
error: The following untracked working tree files would be overwritten by merge:
        stash.txt
Please move or remove them before you merge.
Aborting

# untracked fileを削除してみる
$ rm stash.txt

$ git status
On branch master
nothing to commit, working tree clean

# 再度 apply 1 してみるとstash@{1}では存在していたのでstash.txtが復活する
$ git stash apply 1
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   stash.txt

# 反映されているのが確認できる
$ cat stash.txt
This file is for making sure how git stash works

$ git stash apply 0
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   stash.txt

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   initial-commit.txt


$ git stash list
stash@{0}: WIP on master: 12c7f29 Cleaned up git-basics/
stash@{1}: WIP on master: 12c7f29 Cleaned up git-basics/

$ echo "Make sure how git stash pop works" >> initial-commit.txt

$ git stash
Saved working directory and index state WIP on master: 12c7f29 Cleaned up git-basics/

$ git stash list
stash@{0}: WIP on master: 12c7f29 Cleaned up git-basics/
stash@{1}: WIP on master: 12c7f29 Cleaned up git-basics/
stash@{2}: WIP on master: 12c7f29 Cleaned up git-basics/

# stash popすると...
$ git stash pop 2
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   stash.txt

Dropped refs/stash@{2} (4bdd86a8855215a7e834aa4b0c486233d05e32c9)

# stash listからstash popした内容が削除されているのが確認できる
$ git stash list
stash@{0}: WIP on master: 12c7f29 Cleaned up git-basics/
stash@{1}: WIP on master: 12c7f29 Cleaned up git-basics/
```

検証１：stash 前後で同じファイルを編集してから stash apply したらどうなるか

結果：conflict が起こる


```bash
$ echo "Make sure how git stash works" >> initial-commit.txt

$ cat initial-commit.txt
Write some text at first time
Make some important changes to initial-commit.txt
Add something crazy
make sure what tree object is
make sure how reset works
make sure 2 commit reset
Detached HEAD to fix forgotten things
Make sure how git stash works

$ git ls-files -s
100644 4cf5010e7b78cfd61d33983044299260d21b9189 0       README.md
100644 e69de29bb2d1d6434b8b29ae775ad8c2e48c5391 0       detached.txt
100644 e69de29bb2d1d6434b8b29ae775ad8c2e48c5391 0       dummy.txt
100644 1904c092b649dc54f3c8fc931acb0ca5bb952c3b 0       hoge.txt
100644 1618b9c3afe455248859eea088e09b571acab392 0       huga/huga.txt
100644 40b214e1cc1201a39bbc3e158f069638c858af5f 0       initial-commit.txt
100644 13a430dcab6be55ccae190467062dd451a057338 0       reset.txt

$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   initial-commit.txt

no changes added to commit (use "git add" and/or "git commit -a")

$ git stash
Saved working directory and index state WIP on master: ea096b5 Merge branch 'detached-branch' into master

$ git status
On branch master
nothing to commit, working tree clean

$ git branch make-sure-git-stash

$ git switch make-sure-git-stash
Switched to branch 'make-sure-git-stash'

$ echo "This string might be over written" >> initial-commit.txt

$ cat initial-commit.txt
Write some text at first time
Make some important changes to initial-commit.txt
Add something crazy
make sure what tree object is
make sure how reset works
make sure 2 commit reset
Detached HEAD to fix forgotten things
This string might be over written

$ git ls-files -s
100644 4cf5010e7b78cfd61d33983044299260d21b9189 0       README.md
100644 e69de29bb2d1d6434b8b29ae775ad8c2e48c5391 0       detached.txt
100644 e69de29bb2d1d6434b8b29ae775ad8c2e48c5391 0       dummy.txt
100644 1904c092b649dc54f3c8fc931acb0ca5bb952c3b 0       hoge.txt
100644 1618b9c3afe455248859eea088e09b571acab392 0       huga/huga.txt
100644 40b214e1cc1201a39bbc3e158f069638c858af5f 0       initial-commit.txt
100644 13a430dcab6be55ccae190467062dd451a057338 0       reset.txt

$ git add .

$ git commit -m "Prepared to make sure how git stash works"
[make-sure-git-stash 1e47622] Prepared to make sure how git stash works
 1 file changed, 1 insertion(+)

$ git switch master
Switched to branch 'master'

$ git merge make-sure-git-stash
Updating ea096b5..1e47622
Fast-forward
 initial-commit.txt | 1 +
 1 file changed, 1 insertion(+)

$ git ls-files -s
100644 4cf5010e7b78cfd61d33983044299260d21b9189 0       README.md
100644 e69de29bb2d1d6434b8b29ae775ad8c2e48c5391 0       detached.txt
100644 e69de29bb2d1d6434b8b29ae775ad8c2e48c5391 0       dummy.txt
100644 1904c092b649dc54f3c8fc931acb0ca5bb952c3b 0       hoge.txt
100644 1618b9c3afe455248859eea088e09b571acab392 0       huga/huga.txt
100644 9ded6781475f7503fbcb3961c9da5e151770512b 0       initial-commit.txt
100644 13a430dcab6be55ccae190467062dd451a057338 0       reset.txt

$ git status
On branch master
nothing to commit, working tree clean

$ git stash apply
Auto-merging initial-commit.txt
CONFLICT (content): Merge conflict in initial-commit.txt

$ cat initial-commit.txt
Write some text at first time
Make some important changes to initial-commit.txt
Add something crazy
make sure what tree object is
make sure how reset works
make sure 2 commit reset
Detached HEAD to fix forgotten things
<<<<<<< Updated upstream
This string might be over written
=======
Make sure how git stash works
>>>>>>> Stashed changes

$ git add .

$ git commit -m "Fixed conflict after using git stash"
[master 7bbf29c] Fixed conflict after using git stash
 1 file changed, 1 insertion(+)

$ git ls-files -s
100644 4cf5010e7b78cfd61d33983044299260d21b9189 0       README.md
100644 e69de29bb2d1d6434b8b29ae775ad8c2e48c5391 0       detached.txt
100644 e69de29bb2d1d6434b8b29ae775ad8c2e48c5391 0       dummy.txt
100644 1904c092b649dc54f3c8fc931acb0ca5bb952c3b 0       hoge.txt
100644 1618b9c3afe455248859eea088e09b571acab392 0       huga/huga.txt
100644 06faadf552e04b14c010557ee59b5026e3433bd6 0       initial-commit.txt
100644 13a430dcab6be55ccae190467062dd451a057338 0       reset.txt
```
