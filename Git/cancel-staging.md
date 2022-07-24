# Cancel staging

git add したあとに、インデックスしたファイルをステージングから作業ツリーに戻す方法

## 目次

[checkout](#checkout)
[restore](#restore)

## checkout

#### git checkout HEAD <filename> でステージングを取り消す

git checkout はブランチ（コミット）やファイルを作業ツリーにロードするコマンドである

方法：

-   既存ファイルのステージングを取り消すには git reset
-   既存ファイルのステージングと変更内容を取り消すには git reset してから git checkout
-   ファイルを削除したことのステージングの取り消しは git checkout

```bash
# HEADrefのinitial-commit.txtのステージングを取り消して
#
git checkout HEAD initial-commit.txt
```

検証１：削除してステージングしたファイルを元に戻す

```bash
$ git log --oneline
44d5191 (HEAD -> master) Made sure how reset-mixed works
3b6f08b Make sure what tree object is 2
029342a make sure tree object
00203e3 Revert "Revert "Try something crazy""
71e8b44 Revert "Try something crazy"
815499b Try something crazy
3bc319b Make some important changes to initial-commit.txt
4afcc73 Write text to initial-commit.txt
530cfab first commit

# 間違えてreset.txtしちゃって...
$ rm -rf reset.txt
# そのままgit add してしまって...
$ git add reset.txt

$ git status
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        deleted:    reset.txt
# そのあとにその過ちに気づいたとする。

# この過ちを取り消したいときの従来の方法がこちら...
#
# 講義通りのコマンドだと削除したファイルはうまくいかない模様
$ git checkout reset.txt
error: pathspec 'reset.txt' did not match any file(s) known to git
# ブランチを指定するといいかも
$ git checkout HEAD reset.txt
Updated 1 path from ddef022
# これでreset.txtが作業ツリーに「戻された」はず
#
# ブランチは綺麗です
$ git status
On branch master
nothing to commit, working tree clean

# 削除されたファイルも戻っています。
$ ls
hoge.txt  huga  initial-commit.txt  README.md  reset.txt

```

結果、削除したファイルは元に戻り元通り。

検証２：既存のファイルの変更を反映したステージングから戻して、変更する前の状態まで戻す

```bash
# 既存ファイルに変更を加えた
$ echo "Make sure how git checkout filename works" >> initial-commit.txt

$ cat initial-commit.txt
Write some text at first time
Make some important changes to initial-commit.txt
Add something crazy
make sure what tree object is
make sure how reset works
make sure 2 commit reset
Make sure how git checkout filename works

# ステージングして...
$ git add initial-commit.txt
$ git status
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   initial-commit.txt

# ステージングしてからのblobファイルの確認
$ git ls-files -s
100644 4cf5010e7b78cfd61d33983044299260d21b9189 0       README.md
100644 1904c092b649dc54f3c8fc931acb0ca5bb952c3b 0       hoge.txt
100644 1618b9c3afe455248859eea088e09b571acab392 0       huga/huga.txt
100644 8ee25982417679151ff717f648dc94206b190409 0       initial-commit.txt
100644 13a430dcab6be55ccae190467062dd451a057338 0       reset.txt

# この状態でcheckoutしてみると...
$ git checkout initial-commit.txt
Updated 0 paths from the index

# blobファイルが変化していないのがわかる
$ git ls-files -s
100644 4cf5010e7b78cfd61d33983044299260d21b9189 0       README.md
100644 1904c092b649dc54f3c8fc931acb0ca5bb952c3b 0       hoge.txt
100644 1618b9c3afe455248859eea088e09b571acab392 0       huga/huga.txt
100644 8ee25982417679151ff717f648dc94206b190409 0       initial-commit.txt
100644 13a430dcab6be55ccae190467062dd451a057338 0       reset.txt

# git statusでも変更がないことがわかる
$ git status
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   initial-commit.txt

# reset ファイル　してみると...
$ git reset initial-commit.txt
Unstaged changes after reset:
M       initial-commit.txt

# ステージングだけ取り消しされているのがわかる
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   initial-commit.txt

no changes added to commit (use "git add" and/or "git commit -a")

# blobファイルを確認すると、ステージング直後と異なるのが確認できる
$ git ls-files -s
100644 4cf5010e7b78cfd61d33983044299260d21b9189 0       README.md
100644 1904c092b649dc54f3c8fc931acb0ca5bb952c3b 0       hoge.txt
100644 1618b9c3afe455248859eea088e09b571acab392 0       huga/huga.txt
100644 da4528a61a2f56cf348ec8567cc03a56434c110d 0       initial-commit.txt
100644 13a430dcab6be55ccae190467062dd451a057338 0       reset.txt

# 変更内容は残っているのが確認できる
$ cat initial-commit.txt
Write some text at first time
Make some important changes to initial-commit.txt
Add something crazy
make sure what tree object is
make sure how reset works
make sure 2 commit reset
Make sure how git checkout filename works

# ここでcheckout ファイルすると...
$ git checkout initial-commit.txt
Updated 1 path from the index

# なにもステージングされていないのでblobは変化なしですが
$ git ls-files -s
100644 4cf5010e7b78cfd61d33983044299260d21b9189 0       README.md
100644 1904c092b649dc54f3c8fc931acb0ca5bb952c3b 0       hoge.txt
100644 1618b9c3afe455248859eea088e09b571acab392 0       huga/huga.txt
100644 da4528a61a2f56cf348ec8567cc03a56434c110d 0       initial-commit.txt
100644 13a430dcab6be55ccae190467062dd451a057338 0       reset.txt

# 変更内容も取り消しされているのが確認できる
$ cat initial-commit.txt
Write some text at first time
Make some important changes to initial-commit.txt
Add something crazy
make sure what tree object is
make sure how reset works
make sure 2 commit reset
```

結論：

-   git checkout filename だけでは「既存ファイルの変更を反映したステージング」は戻せない
-   「既存ファイルの変更を反映したステージング」するには git reset filename する必要がある
-   git reset filename の後でステージング内容が作業ツリーに戻る
-   そのあとで git checkout filename すれば変更内容もなかったことになる
-   ファイルの変更は上記の手順で削除したファイルを元に戻すのは別の手順

教訓：

-   `git checkout <commit>`と`git checkout <filename>`は区別しよう
-   git reset も同様

## git restore

https://tracpath.com/docs/git-restore/

> 復元ソースからのコンテンツを使用し、ワークツリー内の指定されたパスを復元します。パスが追跡されている状態で、復元ソースに存在しない場合、ソースと一致するように削除されます。

#### restore コマンドの働き

検証：既存ファイルの変更を反映したステージングからそのファイルのステージングをキャンセルする

```bash
$ echo "Make sure how git restore works" >> initial-commit.txt

$ cat initial-commit.txt
Write some text at first time
Make some important changes to initial-commit.txt
Add something crazy
make sure what tree object is
make sure how reset works
make sure 2 commit reset
Make sure how git restore works

$ git ls-files -s
100644 4cf5010e7b78cfd61d33983044299260d21b9189 0       README.md
100644 1904c092b649dc54f3c8fc931acb0ca5bb952c3b 0       hoge.txt
100644 1618b9c3afe455248859eea088e09b571acab392 0       huga/huga.txt
100644 da4528a61a2f56cf348ec8567cc03a56434c110d 0       initial-commit.txt
100644 13a430dcab6be55ccae190467062dd451a057338 0       reset.txt

# ステージングした
$ git add initial-commit.txt
$ git status
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   initial-commit.txt

$ git ls-files -s
100644 4cf5010e7b78cfd61d33983044299260d21b9189 0       README.md
100644 1904c092b649dc54f3c8fc931acb0ca5bb952c3b 0       hoge.txt
100644 1618b9c3afe455248859eea088e09b571acab392 0       huga/huga.txt
100644 87c812add54ae574bf1077d2a6c6e8892c49d640 0       initial-commit.txt
100644 13a430dcab6be55ccae190467062dd451a057338 0       reset.txt

# 次のコマンドでステージングファイルを作業ツリーへ戻すことができる
$ git restore --staged initial-commit.txt
# 変更はそのままだけどステージングは取り消されているのがわかる
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   initial-commit.txt

no changes added to commit (use "git add" and/or "git commit -a")

# blobも元に戻っている
$ git ls-files -s
100644 4cf5010e7b78cfd61d33983044299260d21b9189 0       README.md
100644 1904c092b649dc54f3c8fc931acb0ca5bb952c3b 0       hoge.txt
100644 1618b9c3afe455248859eea088e09b571acab392 0       huga/huga.txt
100644 da4528a61a2f56cf348ec8567cc03a56434c110d 0       initial-commit.txt
100644 13a430dcab6be55ccae190467062dd451a057338 0       reset.txt

# 変更は残されたまま
$ cat initial-commit.txt
Write some text at first time
Make some important changes to initial-commit.txt
Add something crazy
make sure what tree object is
make sure how reset works
make sure 2 commit reset
Make sure how git restore works

# 先ほど同様、checkoutすると...
$ git checkout initial-commit.txt
Updated 1 path from the index

$ git ls-files -s
100644 4cf5010e7b78cfd61d33983044299260d21b9189 0       README.md
100644 1904c092b649dc54f3c8fc931acb0ca5bb952c3b 0       hoge.txt
100644 1618b9c3afe455248859eea088e09b571acab392 0       huga/huga.txt
100644 da4528a61a2f56cf348ec8567cc03a56434c110d 0       initial-commit.txt
100644 13a430dcab6be55ccae190467062dd451a057338 0       reset.txt

# 変更もなかったことになっている
$ cat initial-commit.txt
Write some text at first time
Make some important changes to initial-commit.txt
Add something crazy
make sure what tree object is
make sure how reset works
make sure 2 commit reset
```

結果、git reset とおなじ挙動をすることが分かった。
