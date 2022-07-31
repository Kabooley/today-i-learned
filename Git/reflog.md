# git reflog

削除された情報を元に戻す方法の模索。

`git reflog`は git プロジェクトで行ったすべての変更が保存されている。

`commit`がその時点のスナップショットで、`reflog`が git での変更すべてを保存する。

https://www.atlassian.com/ja/git/tutorials/rewriting-history/git-reflog

https://git-scm.com/docs/git-reflog

`git reflog`はローカルリポジトリでいつ Git Refs が更新されたのかを記録するコマンドである。

参照ログ (reflog) は、ローカルリポジトリでブランチやその他の参照の先端がいつ更新されたかを記録します。リフロッグは、さまざまな Git コマンドで、ある参照の古い値を指定するのに便利です。たとえば、HEAD@{2} は「2 手前の HEAD があった場所」、master@{one.week.ago} は「このローカルリポジトリで 1 週間前の master が指していた場所」、といったようにです。詳しくは gitrevisions[7] をご覧ください。

このコマンドは、reflog に記録されている情報を管理するものです。

- `git reflog show`

コマンドラインで与えられた参照（デフォルトでは HEAD）のログを表示します。reflog は最近のすべてのアクションをカバーし、さらに HEAD reflog はブランチの切り替えを記録します。git reflog show は git log -g --abbrev-commit --pretty=oneline のエイリアスで、より詳しい情報は git-log[1] をご覧ください。

- `git reflog expire`

古い reflog のエントリを削除します。expire 時間より古いエントリ、あるいは expire-unreachable 時間より古いエントリで現在の tip から到達できないものは、reflog から削除されます。これは通常、エンドユーザーが直接使うことはありません。代わりに git-gc[1] を参照してください。

- `git reflog delete`

reflog から単一のエントリを削除します。引数には正確なエントリを指定しなければなりません (例: "git reflog delete master@{2}")。また、このサブコマンドは通常エンドユーザーが直接使用することはありません。

- `git reflog exists`

reflog が存在するかどうかをチェックします。reflog が存在すればゼロステータスで、存在しなければ非ゼロステータスで終了します。

オプションは公式を見て。

検証１：削除されたコミットを復元する

```bash

# まずはディレクトリの内容とこれまでのコミット・ログの確認
$ ls
initial-commit.txt  README.md  stash.txt
$ git log --oneline
12c7f29 (HEAD -> master) Cleaned up git-basics/
7bbf29c Fixed conflict after using git stash
1e47622 (make-sure-git-stash) Prepared to make sure how git stash works
ea096b5 Merge branch 'detached-branch' into master
1e16572 Changes made in detached HEAD
bafb391 Added dummy file to make sure how detached HEAD works
44d5191 Made sure how reset-mixed works
3b6f08b Make sure what tree object is 2
029342a make sure tree object
00203e3 Revert "Revert "Try something crazy""
71e8b44 Revert "Try something crazy"
815499b Try something crazy
3bc319b Make some important changes to initial-commit.txt
4afcc73 Write text to initial-commit.txt
530cfab first commit

# 新規ファイルを追加したコミットをする
$ echo "This file is to make sure how git reflog works" > reflog.txt
$ git add .
$ git commit -m "Added reflog.txt"
[master f6eec01] Added reflog.txt
 2 files changed, 2 insertions(+)
 create mode 100644 reflog.txt
 create mode 100644 stash.txt

# この時点でのログ
$ git log --oneline
f6eec01 (HEAD -> master) Added reflog.txt
12c7f29 Cleaned up git-basics/
7bbf29c Fixed conflict after using git stash
1e47622 (make-sure-git-stash) Prepared to make sure how git stash works
ea096b5 Merge branch 'detached-branch' into master
1e16572 Changes made in detached HEAD
bafb391 Added dummy file to make sure how detached HEAD works
44d5191 Made sure how reset-mixed works
3b6f08b Make sure what tree object is 2
029342a make sure tree object
00203e3 Revert "Revert "Try something crazy""
71e8b44 Revert "Try something crazy"
815499b Try something crazy
3bc319b Make some important changes to initial-commit.txt
4afcc73 Write text to initial-commit.txt
530cfab first commit

# この時点でのreflog
$ git reflog show
f6eec01 (HEAD -> master) HEAD@{0}: commit: Added reflog.txt
12c7f29 HEAD@{1}: reset: moving to HEAD
12c7f29 HEAD@{2}: reset: moving to HEAD
12c7f29 HEAD@{3}: reset: moving to HEAD
12c7f29 HEAD@{4}: commit: Cleaned up git-basics/
7bbf29c HEAD@{5}: commit: Fixed conflict after using git stash
1e47622 (make-sure-git-stash) HEAD@{6}: merge make-sure-git-stash: Fast-forward
ea096b5 HEAD@{7}: checkout: moving from make-sure-git-stash to master
1e47622 (make-sure-git-stash) HEAD@{8}: commit: Prepared to make sure how git stash works
ea096b5 HEAD@{9}: checkout: moving from master to make-sure-git-stash
ea096b5 HEAD@{10}: reset: moving to HEAD
ea096b5 HEAD@{11}: merge detached-branch: Merge made by the 'recursive' strategy.
bafb391 HEAD@{12}: checkout: moving from detached-branch to master
1e16572 HEAD@{13}: checkout: moving from master to detached-branch
bafb391 HEAD@{14}: checkout: moving from 1e165721c1ae7beae181e8df73c2bdc46e9df8b8 to master
1e16572 HEAD@{15}: commit: Changes made in detached HEAD
44d5191 HEAD@{16}: checkout: moving from master to 44d5191
bafb391 HEAD@{17}: commit: Added dummy file to make sure how detached HEAD works
44d5191 HEAD@{18}: reset: moving to 44d5191
be71bd2 HEAD@{19}: commit: Make sure how reset hard works
44d5191 HEAD@{20}: reset: moving to HEAD
44d5191 HEAD@{21}: commit: Made sure how reset-mixed works
3b6f08b HEAD@{22}: reset: moving to 3b6f08b
2b3cef7 HEAD@{23}: commit: Make sure reset--mixed if specified 2 commit ago
3b6f08b HEAD@{24}: reset: moving to HEAD
3b6f08b HEAD@{25}: commit: Make sure what tree object is 2
029342a HEAD@{26}: commit: make sure tree object
00203e3 HEAD@{27}: reset: moving to 00203e3
47e40b8 HEAD@{28}: commit: Try another something more crazy
00203e3 HEAD@{29}: revert: Revert "Revert "Try something crazy""
71e8b44 HEAD@{30}: revert: Revert "Try something crazy"
815499b HEAD@{31}: reset: moving to 815499b
909a60e HEAD@{32}: commit (merge): Merged branch that from detached HEAD
815499b HEAD@{33}: checkout: moving from new-branch-from-detached-head to master


# このコマンドは誤打です。
$ git reset --hard HEAD
HEAD is now at f6eec01 Added reflog.txt

# こっちを実行したかったんや
$ git reset --hard HEAD~1
HEAD is now at 12c7f29 Cleaned up git-basics/

# 新規ファイルを追加したコミットが削除されているのが確認できる
$ git log --oneline
12c7f29 (HEAD -> master) Cleaned up git-basics/
7bbf29c Fixed conflict after using git stash
1e47622 (make-sure-git-stash) Prepared to make sure how git stash works
ea096b5 Merge branch 'detached-branch' into master
1e16572 Changes made in detached HEAD
bafb391 Added dummy file to make sure how detached HEAD works
44d5191 Made sure how reset-mixed works
3b6f08b Make sure what tree object is 2
029342a make sure tree object
00203e3 Revert "Revert "Try something crazy""
71e8b44 Revert "Try something crazy"
815499b Try something crazy
3bc319b Make some important changes to initial-commit.txt
4afcc73 Write text to initial-commit.txt
530cfab first commit

#
# しかし、reflogならrefの更新をすべて記録しているので
# 削除されたコミットのハッシュ値も削除した後も確認できる
#
$ git reflog show
12c7f29 (HEAD -> master) HEAD@{0}: reset: moving to HEAD~1
f6eec01 HEAD@{1}: reset: moving to HEAD
f6eec01 HEAD@{2}: commit: Added reflog.txt     # ここ！
12c7f29 (HEAD -> master) HEAD@{3}: reset: moving to HEAD
12c7f29 (HEAD -> master) HEAD@{4}: reset: moving to HEAD
12c7f29 (HEAD -> master) HEAD@{5}: reset: moving to HEAD
12c7f29 (HEAD -> master) HEAD@{6}: commit: Cleaned up git-basics/
7bbf29c HEAD@{7}: commit: Fixed conflict after using git stash
1e47622 (make-sure-git-stash) HEAD@{8}: merge make-sure-git-stash: Fast-forward
ea096b5 HEAD@{9}: checkout: moving from make-sure-git-stash to master
1e47622 (make-sure-git-stash) HEAD@{10}: commit: Prepared to make sure how git stash works
ea096b5 HEAD@{11}: checkout: moving from master to make-sure-git-stash
ea096b5 HEAD@{12}: reset: moving to HEAD
ea096b5 HEAD@{13}: merge detached-branch: Merge made by the 'recursive' strategy.
bafb391 HEAD@{14}: checkout: moving from detached-branch to master
1e16572 HEAD@{15}: checkout: moving from master to detached-branch
bafb391 HEAD@{16}: checkout: moving from 1e165721c1ae7beae181e8df73c2bdc46e9df8b8 to master
1e16572 HEAD@{17}: commit: Changes made in detached HEAD
44d5191 HEAD@{18}: checkout: moving from master to 44d5191
bafb391 HEAD@{19}: commit: Added dummy file to make sure how detached HEAD works

# 削除されたこみっとのハッシュ値を使ってgit resetする
$ git reset --hard f6eec01
HEAD is now at f6eec01 Added reflog.txt

$ ls
initial-commit.txt  README.md  reflog.txt  stash.txt

$ git log --oneline
f6eec01 (HEAD -> master) Added reflog.txt
12c7f29 Cleaned up git-basics/
7bbf29c Fixed conflict after using git stash
1e47622 (make-sure-git-stash) Prepared to make sure how git stash works
ea096b5 Merge branch 'detached-branch' into master
1e16572 Changes made in detached HEAD
bafb391 Added dummy file to make sure how detached HEAD works
44d5191 Made sure how reset-mixed works
3b6f08b Make sure what tree object is 2
029342a make sure tree object
00203e3 Revert "Revert "Try something crazy""
71e8b44 Revert "Try something crazy"
815499b Try something crazy
3bc319b Make some important changes to initial-commit.txt
4afcc73 Write text to initial-commit.txt
530cfab first commit
```

NOTE: git reset は過去のコミットに戻るコマンドではなくて指定のコミットに HEAD もブランチも移動させるコマンド

...であるということを認識しておくこと。

なので削除されたコミットを復元するには HEAD とブランチ両方の ref をそのコミットに移動させなくてはならない。

だから git reset を使って復元するのである。

検証 2：削除されたブランチを復元する

参考：https://stackoverflow.com/questions/3640764/can-i-recover-a-branch-after-its-deletion-in-git

削除されたブランチを再度復元するには、

reflog でハッシュ値を確認して、

checkout でそのハッシュ値のコミットをロードして、

そこから新たなブランチを作成するだけ。

```bash
$ git branch feature-

$ git switch feature-
Switched to branch 'feature-'

$ touch reflog-branch.txt

$ ls
initial-commit.txt  README.md  reflog.txt  reflog-branch.txt  stash.txt

$ git add .

$ git commit -m "reflog-branch.txt added"
[feature- 62bbb07] reflog-branch.txt added
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 reflog-branch.txt

$ git switch master
Switched to branch 'master'

$ git branch -d feature-
error: The branch 'feature-' is not fully merged.
If you are sure you want to delete it, run 'git branch -D feature-'.

$ git branch -D feature-
Deleted branch feature- (was 62bbb07).

$ git branch
* master

$ git reflog show
f6eec01 (HEAD -> master) HEAD@{0}: checkout: moving from feature- to master
62bbb07 HEAD@{1}: commit: reflog-branch.txt added
f6eec01 (HEAD -> master) HEAD@{2}: checkout: moving from master to feature-
f6eec01 (HEAD -> master) HEAD@{3}: reset: moving to f6eec01
12c7f29 HEAD@{4}: reset: moving to HEAD~1
f6eec01 (HEAD -> master) HEAD@{5}: reset: moving to HEAD
f6eec01 (HEAD -> master) HEAD@{6}: commit: Added reflog.txt
12c7f29 HEAD@{7}: reset: moving to HEAD
12c7f29 HEAD@{8}: reset: moving to HEAD
12c7f29 HEAD@{9}: reset: moving to HEAD
12c7f29 HEAD@{10}: commit: Cleaned up git-basics/
7bbf29c HEAD@{11}: commit: Fixed conflict after using git stash
1e47622 HEAD@{12}: merge make-sure-git-stash: Fast-forward
ea096b5 HEAD@{13}: checkout: moving from make-sure-git-stash to master
1e47622 HEAD@{14}: commit: Prepared to make sure how git stash works
ea096b5 HEAD@{15}: checkout: moving from master to make-sure-git-stash
ea096b5 HEAD@{16}: reset: moving to HEAD
ea096b5 HEAD@{17}: merge detached-branch: Merge made by the 'recursive' strategy.
bafb391 HEAD@{18}: checkout: moving from detached-branch to master
1e16572 HEAD@{19}: checkout: moving from master to detached-branch
bafb391 HEAD@{20}: checkout: moving from 1e165721c1ae7beae181e8df73c2bdc46e9df8b8 to master
1e16572 HEAD@{21}: commit: Changes made in detached HEAD
44d5191 HEAD@{22}: checkout: moving from master to 44d5191
bafb391 HEAD@{23}: commit: Added dummy file to make sure how detached HEAD works
44d5191 HEAD@{24}: reset: moving to 44d5191
be71bd2 HEAD@{25}: commit: Make sure how reset hard works
44d5191 HEAD@{26}: reset: moving to HEAD
44d5191 HEAD@{27}: commit: Made sure how reset-mixed works
3b6f08b HEAD@{28}: reset: moving to 3b6f08b
2b3cef7 HEAD@{29}: commit: Make sure reset--mixed if specified 2 commit ago
3b6f08b HEAD@{30}: reset: moving to HEAD
3b6f08b HEAD@{31}: commit: Make sure what tree object is 2
029342a HEAD@{32}: commit: make sure tree object
00203e3 HEAD@{33}: reset: moving to 00203e3
47e40b8 HEAD@{34}: commit: Try another something more crazy
00203e3 HEAD@{35}: revert: Revert "Revert "Try something crazy""
71e8b44 HEAD@{36}: revert: Revert "Try something crazy"
815499b HEAD@{37}: reset: moving to 815499b
909a60e HEAD@{38}: commit (merge): Merged branch that from detached HEAD
815499b HEAD@{39}: checkout: moving from new-branch-from-detached-head to master
e9ee83c HEAD@{40}: checkout: moving from e9ee83cb92bdefb984657d53ec6146dcc9934af0 to new-branch-from-detached-head
e9ee83c HEAD@{41}: checkout: moving from new-branch-from-detached-head to e9ee83cb92bdefb984657d53ec6146dcc9934af0
e9ee83c HEAD@{42}: checkout: moving from e9ee83cb92bdefb984657d53ec6146dcc9934af0 to new-branch-from-detached-head
e9ee83c HEAD@{43}: checkout: moving from master to e9ee83c
815499b HEAD@{44}: checkout: moving from 3bc319b79acf14d6f9a3aaffc9b1af9f916afd63 to master
3bc319b HEAD@{45}: checkout: moving from master to 3bc319b
815499b HEAD@{46}: checkout: moving from e9ee83cb92bdefb984657d53ec6146dcc9934af0 to master
e9ee83c HEAD@{47}: commit: Commited on detahced HEAD
3bc319b HEAD@{48}: checkout: moving from master to 3bc319b
815499b HEAD@{49}: commit: Try something crazy
3bc319b HEAD@{50}: commit: Make some important changes to initial-commit.txt
4afcc73 HEAD@{51}: commit: Write text to initial-commit.txt
530cfab HEAD@{52}: commit (initial): first commit

$ git checkout 62bbb07
Note: switching to '62bbb07'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by switching back to a branch.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -c with the switch command. Example:

  git switch -c <new-branch-name>

Or undo this operation with:

  git switch -

Turn off this advice by setting config variable advice.detachedHead to false

HEAD is now at 62bbb07 reflog-branch.txt added

$ git branch
* (HEAD detached at 62bbb07)
  master

$ git switch -c feature-
Switched to a new branch 'feature-'

$ git log --oneline
62bbb07 (HEAD -> feature-) reflog-branch.txt added
f6eec01 (master) Added reflog.txt
12c7f29 Cleaned up git-basics/
7bbf29c Fixed conflict after using git stash
1e47622 Prepared to make sure how git stash works
ea096b5 Merge branch 'detached-branch' into master
1e16572 Changes made in detached HEAD
bafb391 Added dummy file to make sure how detached HEAD works
44d5191 Made sure how reset-mixed works
3b6f08b Make sure what tree object is 2
029342a make sure tree object
00203e3 Revert "Revert "Try something crazy""
71e8b44 Revert "Try something crazy"
815499b Try something crazy
3bc319b Make some important changes to initial-commit.txt
4afcc73 Write text to initial-commit.txt
530cfab first commit

$ git switch master
Switched to branch 'master'

$ git log --oneline
f6eec01 (HEAD -> master) Added reflog.txt
12c7f29 Cleaned up git-basics/
7bbf29c Fixed conflict after using git stash
1e47622 Prepared to make sure how git stash works
ea096b5 Merge branch 'detached-branch' into master
1e16572 Changes made in detached HEAD
bafb391 Added dummy file to make sure how detached HEAD works
44d5191 Made sure how reset-mixed works
3b6f08b Make sure what tree object is 2
029342a make sure tree object
00203e3 Revert "Revert "Try something crazy""
71e8b44 Revert "Try something crazy"
815499b Try something crazy
3bc319b Make some important changes to initial-commit.txt
4afcc73 Write text to initial-commit.txt
530cfab first commit
```

これで feature-ブランチが復活した。
