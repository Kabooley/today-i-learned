# 投稿記事作成: refs オブジェクトが壊れているといわれたとき

git push しようとしたところ、refs オブジェクトが壊れてできませんよといわれるエラーに遭遇して、その解決のためにしたことの記録

## 状況

`test/setup-test`ブランチを push しようとしたところ、以下のようなエラーに遭遇

-   `'refs/remotes/origin/test/setup-test': cannot lock ref`
-   `'refs/remotes/origin/test/setup-test': unable to resolve reference`
-   `'refs/remotes/origin/test/setup-test': reference broken`

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
# 空
```

## 解決した方法

**refs オブジェクトをいったん/tmp へ移動して`git gc`する**

```bash
$ git branch
  main
  development
* test/setup-test
  # ...
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
.git/refs/remotes/origin/test/setup-test  # <-- これ
.git/refs/remotes/origin/feat
.git/refs/remotes/origin/view
.git/refs/heads
.git/refs/heads/test
.git/refs/heads/feat
.git/refs/heads/view
```

参考：

https://stackoverflow.com/a/38192799/22007575

https://stackoverflow.com/a/7348934/22007575

## `git gc`

https://git-scm.com/docs/git-gc

https://git-scm.com/book/ja/v2/Git%E3%81%AE%E5%86%85%E5%81%B4-%E3%83%A1%E3%83%B3%E3%83%86%E3%83%8A%E3%83%B3%E3%82%B9%E3%81%A8%E3%83%87%E3%83%BC%E3%82%BF%E3%83%AA%E3%82%AB%E3%83%90%E3%83%AA

> このコマンドは幾つものことを行います。すべての緩いオブジェクトを集めて packfile に入れ、複数の packfile をひとつの大きな packfile に統合し、さらにどのコミットからも到達が不可能かつ数ヶ月間更新がないオブジェクトを削除します。

git はその設定ファイルに定めた上限が超えたときに自動的に git gc を行っている。

packfile の情報は`cat .git/packed-refs`で確認できる

## 今回のエラー原因

## そもそもブランチ名に`/`を含めてよいのか

## refs はいつどうやって生成されるのか

## `git reflog`
