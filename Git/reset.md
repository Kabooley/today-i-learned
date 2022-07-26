# Note about git reset

## 参考

https://git-scm.com/book/ja/v2/Git-%E3%81%AE%E3%81%95%E3%81%BE%E3%81%96%E3%81%BE%E3%81%AA%E3%83%84%E3%83%BC%E3%83%AB-%E3%83%AA%E3%82%BB%E3%83%83%E3%83%88%E3%82%B3%E3%83%9E%E3%83%B3%E3%83%89%E8%A9%B3%E8%AA%AC

https://www.atlassian.com/ja/git/tutorials/undoing-changes/git-reset

## 目次

[前提の話](#前提の話)
[コミットに対する reset](#コミットに対するreset)
[ファイルに対する reset](#ファイルに対するreset)

## 前提の話

https://git-scm.com/book/ja/v2/Git-%E3%81%AE%E3%81%95%E3%81%BE%E3%81%96%E3%81%BE%E3%81%AA%E3%83%84%E3%83%BC%E3%83%AB-%E3%83%AA%E3%82%BB%E3%83%83%E3%83%88%E3%82%B3%E3%83%9E%E3%83%B3%E3%83%89%E8%A9%B3%E8%AA%AC

#### 3 つのツリー

リセットコマンドといえばということで`reset`と`checkout`を知らなくてはならない。

これらのコマンドの動きを理解するには前提として 3 つのツリーを理解しておかなくてはならない。

> ここでいう「ツリー」とはあくまで「ファイルの集まり」であって、データ構造は含みません。
> 厳密にツリー構造であるわけでもない

-   HEAD: 現在（最新）のブランチ ref を参照する ref。
-   インデックス：add したら`.git/index/`へ保存されるスナップショットで次のコミットの候補
-   作業ツリー：サンドボックス(保護された自由に使える空間のこと)

HEAD:

> 現在のブランチを指し示すポインタは HEAD と呼ばれています。HEAD は、そのブランチの最新コミットを指し示すポインタでもあります。 ということは、HEAD が指し示すコミットは新たに追加されていくコミットの親になる、ということです。 **HEAD のことを 最新のコミット のスナップショットと捉えておくとわかりやすいでしょう。**

`git checkout other-branch`したら HEAD は`other-branch`の最新コミット（ブランチ ref が指しているもの）を指し示す

HEAD はブランチの最新 ref を指させないこともできる。それは detached HEAD といわれる。

作業ツリー：

作業ツリー以外の 2 つのツリーは、主に`.git/`以下へデータを保存するのに対して、

作業ツリーではデータはユーザが通常ファイルを編集するのと同様に実際のファイルが展開される。

インデックス：

> インデックスの中身は、前回のチェックアウトで作業ディレクトリに保存されたファイルの一覧になっています。
> 保存時のファイルの状態も記録されています。 ファイルに変更を加え、git commit コマンドを実行すると、ツリーが作成され新たなコミットとなります。

つまり、インデックスには`git add`されたファイルのデータだけじゃなくて、前回のコミットした結果作業ツリーへ保存されたファイルの一覧もある

`.git/index`へ保存されているのは blob ファイルである。

#### 通常のコミットまでに起こること

前回のコミットの直後という状況を想定する。

HEAD は現在のブランチの最新のコミットを指し示している。

作業ツリーは実際のファイルが展開されているので、

ファイルの編集、新規追加、削除はそのまま反映される。

index, HEAD には現時点で何も変化はない。

`git add .`する。

すると index に add で指定したファイル毎の blob ファイルが生成される。

blob ファイルは add 毎に一意のハッシュ値を生成する。

HEAD に変化はない。

コミットする前に同じファイルをまた編集してまた add したら、ことなるハッシュ値の blob ファイルが生成されるので、完全に add 毎のステージングは異なるものとして扱われる

`git commit`する。

すると`.git/objects/`以下に tree オブジェクトと commit オブジェクトが生成される。

詳しくは`inside-of-git.md`を見ること。

この時点でスナップショットが保存される。こうして生まれるのがコミットオブジェクト。

HEAD はその最新のコミットオブジェクトを指すのである

## コミットに対する reset

`git reset`は HEADref もブランチ ref も指定のコミットへ移動する。

#### git reset --mixed HEAD

-   HEADref、ブランチ ref は指定のコミットを指す

-   ステージングの状態は、指定のコミットの直後の状態である

    `.git/index/`は指定のコミットをしたときの状態になる

-   作業ツリーには、ファイルの編集状態が反映されている

たとえば 2 つ前のコミットを指定すると、1 つ前以降のコミットは孤立す。

-   なので add と commit をしたことだけを取り消して、編集内容などはそのまま残したいときは--mixed HEAD を使うといい

確認：

```bash
# 一つのファイルを変更して、一つのファイルを新規追加してgit addした
$ git add new-file.txt
$ git status
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   new-file.txt

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   initial-commit.txt
# インデックスの状態を確認
$ git ls-files -s
100644 36db3e74125eff086804e547bcb87afb096e50e2 0       initial-commit.txt
100644 2e35eb989a37ee7f7df45aa9f472b71e9b492b16 0       new-file.txt
#
# reset --mixed HEADしてみる
#
$ git reset --mixed HEAD
Unstaged changes after reset:
M       initial-commit.txt

$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   initial-commit.txt

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        new-file.txt

no changes added to commit (use "git add" and/or "git commit -a")

$ git ls-files -s
100644 36db3e74125eff086804e547bcb87afb096e50e2 0       initial-commit.txt
```

つまり、

-   index されたことはなかったことになる

    `git ls-files -s`で新しく追加したファイルのことがなくなっているし、
    `git status`で untracked ファイル扱いになっている
    更新したファイル(initial-commit.txt)もステージングされていない扱いになっている

-   作業ディレクトリへの変更はそのままだけど、ステージングはキャンセルされて、コミットは指定のコミットの状態にリセットされる。

```bash
# ファイルの更新と新規ファイルの追加をしてgit addした
$ git ls-files -s
100644 ca6137f59ff6211ec6473d9efc725e5570bc702a 0       initial-commit.txt
100644 2e35eb989a37ee7f7df45aa9f472b71e9b492b16 0       new-file.txt

$ git reset --mixed HEAD
Unstaged changes after reset:
M       initial-commit.txt
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   initial-commit.txt

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        new-file.txt

no changes added to commit (use "git add" and/or "git commit -a")
# 更新した方のファイルはインデックスされる前のSHA-1になっている
# つまり以前のコミット時のSHA-1値である
$ git ls-files -s
100644 36db3e74125eff086804e547bcb87afb096e50e2 0       initial-commit.txt
```

2 つ以上まえのコミットを指定するとどうなるか

```bash
# 一度コミットをして、
$ git log --oneline
2b3cef7 (HEAD -> master) Make sure reset--mixed if specified 2 commit ago
3b6f08b Make sure what tree object is 2
029342a make sure tree object
00203e3 Revert "Revert "Try something crazy""
71e8b44 Revert "Try something crazy"
815499b Try something crazy
3bc319b Make some important changes to initial-commit.txt
4afcc73 Write text to initial-commit.txt
530cfab first commit
# そのコミットの一つ前のコミットでgit reset --mixedを行った
$ git reset --mixed 3b6f08b
Unstaged changes after reset:
M       initial-commit.txt
# HEADもブランチも指定のコミットへ戻った
$ git log --oneline
3b6f08b (HEAD -> master) Make sure what tree object is 2
029342a make sure tree object
00203e3 Revert "Revert "Try something crazy""
71e8b44 Revert "Try something crazy"
815499b Try something crazy
3bc319b Make some important changes to initial-commit.txt
4afcc73 Write text to initial-commit.txt
530cfab first commit
# 検証１：2b3cef7のコミット内容は作業ツリーに残っているのか？
#
# 結果：各ファイルの最後の行がコミットしたときに追加した文章で、これが残っている
$ cat reset.txt
make sure how reset works
make sure 2 commit reset

$ cat initial-commit.txt
Write some text at first time
Make some important changes to initial-commit.txt
Add something crazy
make sure what tree object is
make sure how reset works
make sure 2 commit reset
# なのでこのままadd, commitすればresetする前のコミットの通りになる
```

2 つ以上前のコミットを指定して`git reset --mixed`したら、その間のコミット内容はすべて作業ツリーへ戻されるので、

そのままコミットすれば reset するまえのコミットの通りになる

#### git reset --hard HEAD

-   HEADref, ブランチ ref は指定のコミットを指す

-   ステージングは完全にまっさらな状態になる

-   作業ツリーも完全にクリーンである。

つまり、--hard にするとステージングだけじゃなくて、編集したファイルや追加、削除したファイルもなかったことになるので

完全に指定したコミットの直後の状態に戻される

検証１：

既存ファイル更新: initial-commit.txt、
既存ファイル１つ削除: reset.txt、
新規ファイル追加：reset-hard.txt
git add .し、reset --hard HEAD した

```bash
# addしたあと
$ git ls-files -s
100644 4cf5010e7b78cfd61d33983044299260d21b9189 0       README.md
100644 1904c092b649dc54f3c8fc931acb0ca5bb952c3b 0       hoge.txt
100644 1618b9c3afe455248859eea088e09b571acab392 0       huga/huga.txt
100644 a2fbecb07e63156bcf56ece45564f6d1c55a65e2 0       initial-commit.txt
100644 920678635e2bf1e679087a9b14bf342fd02cd15b 0       reset-hard.txt

$ git reset --hard HEAD
HEAD is now at 44d5191 Made sure how reset-mixed works

# 作業ツリーもクリーンである
# 新規に追加したファイルがどこにも存在しない
$ git status
On branch master
nothing to commit, working tree clean


$ git ls-files -s
100644 4cf5010e7b78cfd61d33983044299260d21b9189 0       README.md
100644 1904c092b649dc54f3c8fc931acb0ca5bb952c3b 0       hoge.txt
100644 1618b9c3afe455248859eea088e09b571acab392 0       huga/huga.txt
# 元に戻っているのがわかる
100644 da4528a61a2f56cf348ec8567cc03a56434c110d 0       initial-commit.txt
# 削除したファイルも戻っている
100644 13a430dcab6be55ccae190467062dd451a057338 0       reset.txt
```

間違いなく指定のコミットの直後の状態に戻っている

検証２：

2 つ以上前のコミットを指定してみる

--mixed ではすべて作業ツリーに戻されたが、--hard ではどうなるか

って書くまでもないけど（確認はしたけど）、

指定したコミットの直後の状態に戻るだけだから作業ツリーもインデックスも空である。

--mixed はリセット前の状態を作業ツリーに残しておけるけど
--hard は完全に指定のコミットの直後のクリーンな状態に作業ツリーもインデックスも戻される

#### git reset --soft

ちょっと割愛
