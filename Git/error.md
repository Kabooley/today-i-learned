# git を扱ううえで遭遇するエラーとその対処法

## `git commit`コマンドで`atom --wait`コマンドが見つからないとか言われたら

git がデフォルトで呼び出すエディタを一致させよう。

`atom`は atom エディタのことで、このエラーメッセージが出た場合は、
git が atom のエディタを呼び出そうとしたけれど atom がなかったもしくは atom エディタを使っていなかった場合である。

VSCode を使っていたら次の通りに設定を変更する。

```bash
# git commitコマンド時に呼び出されるエディタをVSCodeにした
$ git config --global core.editor 'code --wait'
# git rebase -1コマンド時に呼び出されるエディタをVSCodeにした
$ git config --global sequence.editor 'code --wait'
```

参考：

https://qiita.com/ucan-lab/items/9b442e042988e2d7a35d

## `refs`オブジェクトが壊れたといわれたとき

遭遇したエラー： `unable to resolve reference 'refs/remotes/origin/test/setup-test': reference broken`

git push しようとしたらタイトルのようなエラーに遭遇した。

.git/refs のオブジェクトに不具合があるという旨のエラーである。

https://stackoverflow.com/questions/2998832/git-pull-fails-unable-to-resolve-reference-unable-to-update-local-ref

https://git-scm.com/book/ja/v2/Git%E3%81%AE%E5%86%85%E5%81%B4-%E3%83%A1%E3%83%B3%E3%83%86%E3%83%8A%E3%83%B3%E3%82%B9%E3%81%A8%E3%83%87%E3%83%BC%E3%82%BF%E3%83%AA%E3%82%AB%E3%83%90%E3%83%AA

```bash
$ git branch
  development
  feat/explorer--actions
  feat_footer
  feat_footer__linter
  feat_view-improvement--pane-tabs
  main
* test/setup-test
  view/improvement
#
$ git push origin test/setup-test
error: update_ref failed for ref 'refs/remotes/origin/test/setup-test': cannot lock ref 'refs/remotes/origin/test/setup-test': unable to resolve reference 'refs/remotes/origin/test/setup-test': reference broken
Everything up-to-date
$ find .git/refs
.git/refs
.git/refs/tags
.git/refs/remotes
.git/refs/remotes/origin
.git/refs/remotes/origin/fix
.git/refs/remotes/origin/test
.git/refs/remotes/origin/test/setup-test
.git/refs/remotes/origin/feat
.git/refs/remotes/origin/view
.git/refs/heads
.git/refs/heads/test
.git/refs/heads/feat
.git/refs/heads/view
$ cat .git/refs/remotes/origin/test/setup-test
$
```

#### そもそも git ブランチに`/`を含めてもよいのか？

含めてもよい。ただし、制限が発生する。

ブランチを生成すると、`refs`というオブジェクトを生成することになる。

`./git/refs`以下に生成されて、ディレクトリ階層を生成する。

たとえば

`git branch test/setup-test`というブランチを生成したとして

`.git/refs`以下には

`.git/refs/heads/test/setup-test`という refs が生成される。

すると今後

`test`というブランチは作ることはできなくなる。

そういった制限は発生する。

https://stackoverflow.com/questions/2527355/using-the-slash-character-in-git-branch-name

https://stackoverflow.com/a/3651867/22007575

`git check-ref-format --branch <branchname-shorthand>`で
渡された**参照名**が許容できるものかなどを診断してくれるみたい。

https://git-scm.com/docs/git-check-ref-format

#### 有効だった策：refs オブジェクトを/tmp へ移動して`git gc`する

https://stackoverflow.com/a/38192799/22007575

```bash
$ git branch
  main
  development
* test/setup-test
# 何もしないでgit gcすると以下の通りのエラーが発生する
$ git gc
error: bad ref for .git/logs/refs/remotes/origin/test/setup-test
fatal: bad object refs/remotes/origin/test/setup-test
fatal: failed to run repack
# refsオブジェクトを一時フォルダへ移動する
$ mv .git/logs/refs/remotes/origin/test/setup-test /tmp
# 移動したファイルがエラーから消えているのが確認できる
$ git gc
fatal: bad object refs/remotes/origin/test/setup-test
fatal: failed to run repack
# 同様に
$ mv .git/logs/refs/remotes/origin/test/setup-test /tmp
# 実行できた
$ git gc
Enumerating objects: 1064, done.
Counting objects: 100% (1064/1064), done.
Delta compression using up to 4 threads
Compressing objects: 100% (1022/1022), done.
Writing objects: 100% (1064/1064), done.
Total 1064 (delta 583), reused 0 (delta 0), pack-reused 0
# remotesを削除したけど、pushもfetchもできた
$ git push origin test/setup-test
Everything up-to-date
$ git fetch origin test/setup-test
# その後refsオブジェクトを確認するとpushした通りにremotesが追加されていた
$ find .git/refs
.git/refs
.git/refs/tags
.git/refs/remotes
.git/refs/remotes/origin
.git/refs/remotes/origin/fix
.git/refs/remotes/origin/test
.git/refs/remotes/origin/test/setup-test  # <-- こいつ
.git/refs/remotes/origin/feat
.git/refs/remotes/origin/view
.git/refs/heads
.git/refs/heads/test
.git/refs/heads/feat
.git/refs/heads/view
```

#### 今回のエラーの原因

正直、原因は特定できていない。

```bash
$ git push origin test/setup-test
error: update_ref failed for ref 'refs/remotes/origin/test/setup-test': cannot lock ref 'refs/remotes/origin/test/setup-test': unable to resolve reference 'refs/remotes/origin/test/setup-test': reference broken
Everything up-to-date
$ find .git/refs
.git/refs
.git/refs/tags
.git/refs/remotes
.git/refs/remotes/origin
.git/refs/remotes/origin/fix
.git/refs/remotes/origin/test
.git/refs/remotes/origin/test/setup-test
.git/refs/remotes/origin/feat
.git/refs/remotes/origin/view
.git/refs/heads
.git/refs/heads/test
.git/refs/heads/feat
.git/refs/heads/view
# 以下の通り２つのrefsオブジェクトがおかしかった
$ git gc
error: bad ref for .git/logs/refs/remotes/origin/test/setup-test
fatal: bad object refs/remotes/origin/test/setup-test
fatal: failed to run repack
```

https://git-scm.com/book/en/v2/Git-Internals-Git-References

refs が生成される瞬間:

コミットしたとき。

refs は特定のコミットオブジェクトを参照するポインタのようなものである。

そして branch は refs の一種である。

ということは、`.git/refs/remots/origin/test/setup-test`という refs オブジェクトは

`git push origin test/setup-test`したときに生成されるはずである。

`remotes` refs: remote を追加して push したときのそのプッシュした値（コミット）をブランチごとに`refs/remots`へ格納する

> The third type of reference that you’ll see is a remote reference. If you add a remote and push to it, Git stores the value you last pushed to that remote for each branch in the refs/remotes directory.

`heads` refs: `heads` ディレクトリの各ファイルはローカルリポジトリのすべてのブランチを参照する

つまり、ブランチを生成すると、`.git/refs/heads`にそのブランチのrefsが追加されるはずということ

https://www.atlassian.com/git/tutorials/refs-and-the-reflog

例 `.git/refs/heads`に新規ブランチが追加されることの確認:

```bash
$ find .git/refs
.git/refs
.git/refs/heads
.git/refs/heads/main
.git/refs/heads/modify_git_error_note
.git/refs/remotes
.git/refs/remotes/origin
.git/refs/remotes/origin/HEAD
.git/refs/remotes/origin/main
.git/refs/remotes/origin/modify_git_error_note
.git/refs/tags
$ git branch
  main
* modify_git_error_note
$ git branch temporary-to-make-sure-git-refs
$ find .git/refs
.git/refs
.git/refs/heads
.git/refs/heads/main
.git/refs/heads/modify_git_error_note
# 追加されている
.git/refs/heads/temporary-to-make-sure-git-refs
.git/refs/remotes
.git/refs/remotes/origin
.git/refs/remotes/origin/HEAD
.git/refs/remotes/origin/main
.git/refs/remotes/origin/modify_git_error_note
.git/refs/tags
```

#### 他の提案 1: `git remote set-head origin --auto`

https://stackoverflow.com/a/49944297/22007575

#### 他の提案 2：`git update-ref -d refs/remotes/origin/test/setup-test`

https://stackoverflow.com/questions/32881670/git-update-ref-appears-to-do-nothing

https://stackoverflow.com/questions/58126421/cannot-lock-ref-refs-remotes-origin-master?noredirect=1&lq=1

#### 他の提案 3：`git remote prune origin`

でもこれは状況が異なるみたい

https://stackoverflow.com/questions/6656619/git-and-nasty-error-cannot-lock-existing-info-refs-fatal

## git `ref`について

https://git-scm.com/book/en/v2/Git-Internals-Git-References
