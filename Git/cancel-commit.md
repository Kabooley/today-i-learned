# Cancel Commit

## 参考

## 目次

[checkout:古いコミットを確認する](#古いコミットを確認する)
[rever:公開リポジトリのコミットを基に戻す](#rever:公開リポジトリのコミットを基に戻す)
[reset:コミットを戻す最適な方法](#reset:コミットを戻す最適な方法)

## 古いコミットを確認する

`git checkout <commit-hash>`で過去のコミットを作業ツリーへ参照することができます。

このときHEADは`Detached HEAD`になります。

detached HEAD状態でのあらゆる変更はブランチrefへ戻るとなかったことになります。

このなかったことは、ファイルの編集、追加、削除、コミットも含みます。

なぜならdetached HEADでのコミットは孤立するからです。

Gitでコミットの孤立は、どこからも参照されていないコミットを意味し、

通常コミットオブジェクトは親コミットオブジェクトへの参照値（ハッシュ値）を所持し

それによって誰が親かが判別できる仕組みになっています。

この仕組みによってどのコミットから参照されているのかわかるのだけれど、

孤立したコミットはどこからも参照されていません。

そのうちガベージコレクションに削除されます。

ということでgit checkoutで過去のコミットを作業ツリーへロードして

いろいろいじって問題のある個所を探したりするのに便利です。

検証：

```bash
$ cat initial-commit.txt
Write some text at first time
Make some important changes to initial-commit.txt
# Try something crazyで行った変更がないので指定したコミットへ戻ったことが確認できる

# detached HEADでコミットした
# ブランチの指すコミットがない。これはブランチ上で作業していないから。
$ git log --oneline
e9ee83c (HEAD) Commited on detahced HEAD
3bc319b Make some important changes to initial-commit.txt
4afcc73 Write text to initial-commit.txt
530cfab first commit

# これでブランチに戻ってみる
# 
# すると、
# 「どのブランチにも参照されていない1つのコミットを残しています」という警告が出る
$ git checkout -
Warning: you are leaving 1 commit behind, not connected to
any of your branches:

  e9ee83c Commited on detahced HEAD

If you want to keep it by creating a new branch, this may be a good time
to do so with:

 git branch <new-branch-name> e9ee83c

Switched to branch 'master'

# detached HEADでのコミットが含まれていないことがわかる
# なので過去のコミットへchekcoutで戻って変更してもブランチに戻ればなかったことになる
# というのは確認できた
$ git log --oneline
815499b (HEAD -> master) Try something crazy
3bc319b Make some important changes to initial-commit.txt
4afcc73 Write text to initial-commit.txt
530cfab first commit

# 先の孤立したコミットへ移動はできる
$ git checkout e9ee83c
$ git switch -c new-branch-from-detached-head
Switched to a new branch 'new-branch-from-detached-head'
# それまでのコミットは参照できる
$ git log --oneline
e9ee83c (HEAD -> new-branch-from-detached-head) Commited on detahced HEAD
3bc319b Make some important changes to initial-commit.txt
4afcc73 Write text to initial-commit.txt
530cfab first commit

# 戻ってみると...
# detached HEADへ戻り、masterではない
git checkout -
Note: switching to 'e9ee83cb92bdefb984657d53ec6146dcc9934af0'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by switching back to a branch.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -c with the switch command. Example:

  git switch -c <new-branch-name>

Or undo this operation with:

  git switch -

Turn off this advice by setting config variable advice.detachedHead to false

HEAD is now at e9ee83c Commited on detahced HEAD

# masterにも戻れる
$ git checkout master
Switched to branch 'master'

# detached HEADから確立したブランチをmasterへmergeしようとすると、conflictが起こる
$ git merge new-branch-from-detached-head
Auto-merging initial-commit.txt
CONFLICT (content): Merge conflict in initial-commit.txt
Automatic merge failed; fix conflicts and then commit the result.

# コンフリクトを解消する前だと、解消前の状態のまま
$ git log --oneline
815499b (HEAD -> master) Try something crazy
3bc319b Make some important changes to initial-commit.txt
4afcc73 Write text to initial-commit.txt
530cfab first commit

# ためしにマージしたブランチの内容を受け付けてみる
# 
# conflictが起こる
$ git status
On branch master
You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)

Unmerged paths:
  (use "git add <file>..." to mark resolution)
        both modified:   initial-commit.txt

no changes added to commit (use "git add" and/or "git commit -a")

# 結果
$ git log --oneline
909a60e (HEAD -> master) Merged branch that from detached HEAD
e9ee83c (new-branch-from-detached-head) Commited on detahced HEAD
815499b Try something crazy
3bc319b Make some important changes to initial-commit.txt
4afcc73 Write text to initial-commit.txt
530cfab first commit
```
git checkoutでdetached HEADになってもあらたにそこからブランチを作成して、

元のブランチへmergeすることで過去のコミットに変更を施してそれを反映させる方法もあります。

```bash
git checkout <past-commit>
# edit and commited
git switch -c <new-branch-name-from-past-commit>
git switch master
git merge <new-branch-name-from-past-commit>
```

ただしコンフリクトが起こる可能性が高いです。

## rever:公開リポジトリのコミットを基に戻す

`git revert`は、指定のコミットを「打ち消す」様なコミットを行います。

つまり、`git revert <commit-id>`で指定したコミットで行ったコミット内容すべてを

元に戻す処理を変更をコミットするのである。

特徴的なのは、

他の多くの取り消すコマンドが過去のコマンドに戻るのに対して

`git revert`は戻らずにコミットを追加するのである。

なので一切過去のコミットを消すことがなく、すべて履歴に残すことができる。

なので共有リポジトリで作業するときの理想的な「元に戻す」ための方法です。

以下の検証を見てみると、

直前のコミットよりも前のコミットを指定して、コンフリクトが起こるかどうかは

どこを修正するかに依るのかしら。

```bash
# 今、またTry something crazyという取り消したいコミットがあるとする
$ git log --oneline
815499b (HEAD -> master) Try something crazy
3bc319b Make some important changes to initial-commit.txt
4afcc73 Write text to initial-commit.txt
530cfab first commit

# HEADを「打ち消し」た
# 
$ git revert HEAD
hint: Waiting for your editor to close the file... 
[master 71e8b44] Revert "Try something crazy"
 1 file changed, 1 deletion(-)

# このコミットは"Try something crazxy"をコミット履歴から消すのではなく
# のこしたまま、
# revertによるコミットがそのコミットを打ち消すような変更でコミットされるのである
$ git log --oneline
71e8b44 (HEAD -> master) Revert "Try something crazy"
815499b Try something crazy
3bc319b Make some important changes to initial-commit.txt
4afcc73 Write text to initial-commit.txt
530cfab first commit

# 次でそれが確認できる
# 
# "Try something crazxy"の書き込みがなくなっている
$ cat initial-commit.txt
Write some text at first time
Make some important changes to initial-commit.txt
```

- 同じブランチを使い続けることができる
- 重要なコミットを消す必要がなくなる
- 取り消したいコミットもコミット履歴に残すことができる

検証：

- 2つ以上前のコミットをrevertすると1つ前のコミットはどうなるのか？
- revertしたコミットをrevertしたらどうなるのか？

```bash
$ git log --oneline
71e8b44 (HEAD -> master) Revert "Try something crazy"
815499b Try something crazy
3bc319b Make some important changes to initial-commit.txt
4afcc73 Write text to initial-commit.txt
530cfab first commit

# さらにrevert
$ git revert  HEAD
hint: Waiting for your editor to close the file...
[master 00203e3] Revert "Revert "Try something crazy""
 1 file changed, 1 insertion(+)

# revertしたコミットをrevertすると元に戻るのが確認できる
$ cat initial-commit.txt
Write some text at first time
Make some important changes to initial-commit.txt
Add something crazy

```

```bash
$ git log --oneline
47e40b8 (HEAD -> master) Try another something more crazy
00203e3 Revert "Revert "Try something crazy""
71e8b44 Revert "Try something crazy"
815499b Try something crazy
3bc319b Make some important changes to initial-commit.txt
4afcc73 Write text to initial-commit.txt
530cfab first commit

# 2つ前のコミットをrevertしてみる
# 
# revertはできなくてconflictが起こる
$ git revert 815499b
Auto-merging initial-commit.txt
CONFLICT (content): Merge conflict in initial-commit.txt
error: could not revert 815499b... Try something crazy
hint: after resolving the conflicts, mark the corrected paths
hint: with 'git add <paths>' or 'git rm <paths>'
hint: and commit the result with 'git commit'
```

結果：

- git revertは１つのコミットだけ打ち消すことができる

> git revert は、1 つのコミットのみを元に戻すコマンドであることをしっかりと理解してください。

つまり結局直前のコミットだけになるね

先の2つまえのコミットを指定してrevertしたらrevertはできなくてコンフリクトが起こった

## reset:コミットを戻す最適な方法

`./reset.md`に書いた。