# 投稿記事作成: refs オブジェクトが壊れているといわれたとき

git push しようとしたところ、refs オブジェクトが壊れてできませんよといわれるエラーに遭遇して、その解決のためにしたことの記録

他主に refs オブジェクトに関するまとめ。

## 状況

`test/setup-test`ブランチを push しようとしたところ、以下のようなエラーに遭遇

-   `'refs/remotes/origin/test/setup-test': cannot lock ref`
-   `'refs/remotes/origin/test/setup-test': unable to resolve reference`
-   `'refs/remotes/origin/test/setup-test': reference broken`

とにかく`.git/refs/remotes/origin/test/setup-test` refs オブジェクトがどうかしたらしい。

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

#### TODO: 解決策を支持する根拠

-   refs オブジェクトは結局コミットへの参照であるのでコミットオブジェクトが削除されるわけではない
-   refs オブジェクトは

## `git gc`

機能：

-   「緩いオブジェクト」を収集して圧縮する
-   「緩いオブジェクト」を削除する

https://git-scm.com/docs/git-gc

https://git-scm.com/book/ja/v2/Git%E3%81%AE%E5%86%85%E5%81%B4-%E3%83%A1%E3%83%B3%E3%83%86%E3%83%8A%E3%83%B3%E3%82%B9%E3%81%A8%E3%83%87%E3%83%BC%E3%82%BF%E3%83%AA%E3%82%AB%E3%83%90%E3%83%AA

> このコマンドは幾つものことを行います。すべての緩いオブジェクトを集めて packfile に入れ、複数の packfile をひとつの大きな packfile に統合し、さらにどのコミットからも到達が不可能かつ数ヶ月間更新がないオブジェクトを削除します。

「緩いオブジェクト」というのは孤立したコミットやアクセスできないコミットなどを指す。

git はその設定ファイルに定めた上限が超えたときに自動的に git gc を行っている。

packfile の情報は`cat .git/packed-refs`で確認できる

`git gc`は git config の設定に従ってその実行内容を決定する。

Atlassian の記事がわかりやすかった：

https://www.atlassian.com/git/tutorials/git-gc

たとえば設定の一例で、

```
gc.pruneExpire: デフォルトは「2週間」のオプション変数。アクセス不可能なオブジェクトがプルーニングされるまでの保存期間を設定する。
```

つまり、２週間を過ぎたアクセス不可能となったオブジェクトは`git gc`で削除される

`git gc`で実行されるサブコマンドのうち、実際に削除にかかわるのが`git prune`のようである。

## `git prune`

機能：

-   「緩いオブジェクト」を削除する

`git gc`すると内部的に`git prune`が実行されている

https://www.atlassian.com/git/tutorials/git-gc

git config に従って、`git prune`や`git repack`などのサブコマンドが、`git gc`を実行するときに内部的に実行されている。

> Behind the scenes git gc actually executes a bundle of other internal subcommands like git prune, git repack, git pack and git rerere. The high-level responsibility of these commands is to identify any Git objects that are outside the threshold levels set from the git gc configuration. Once identified, these objects are then compressed, or pruned accordingly.

#### `git gc --prune=now`

https://git-scm.com/docs/git-gc#Documentation/git-gc.txt---pruneltdategt

上記のコマンドを実行するとどうなるか

デフォルト値で`gc.pruneExpire`が 2 週間であり、`now`のオプションで 2 週間の猶予なしで「緩いｵﾌﾞｼﾞｪｸﾄ」すべてを削除するのだと思われる

> --prune=now は、古いオブジェクトに関係なくルーズなオブジェクトをプルーニングし、別のプロセスが同時にリポジトリに書き込んでいる場合に破損のリスクを高めます。

## `git gc` vs. `git prune`

`git prune`が実際に削除するコマンドで、`git gc`は内部的に`git prune`を実行する。

そのため、「削除はしたくなくてオブジェクトの圧縮だけしたい」という場合は

`git gc --no-prune`などのコマンドで`prune`コマンドの実行を避ける方法をとる必要がある

## 今回のエラー原因

正直原因は特定できていない。

`git push`するときに発生し、その後`refs/remotes/`以下の該当ファイルを確認するも

中身は空であったので、`refs/remotes/`の生成時に何らかの通信エラーがあったのかもしれない

## そもそもブランチ名に`/`を含めてよいのか

含めてもよい。ただし、以下の制限が発生する：

-   branch B が存在する場合、branch B/A は生成できない
-   branch B/A が存在する場合、branch B は生成できない

ブランチを生成すると、`refs`というオブジェクトを生成することになる。

`./git/refs`以下に生成されて、ディレクトリ階層を生成する。

たとえば

`git branch test/setup-test`というブランチを生成したとして

`.git/refs`以下には

`.git/refs/heads/test/setup-test`という refs が生成される。

ディレクトリ階層なので、`test`というディレクトリの下に`setup-test`という refs オブジェクトが生成されているのである。

そうなると、たとえばその後`test`というブランチを作ろうとすると、

`test`はすでにディレクトリ名として存在しており、区別がつかなくなるため`test`というブランチは作ることはできなくなる。

そういった制限は発生する。

実際に検証してみると以下の通り：

```bash
# 検証１：hogeブランチが既に存在する場合、hoge/fugaブランチは作成できるか?

$ git branch hoge
$ git branch
  main
* feat_counter
  hoge

$ find .git/refs
.git/refs
.git/refs/heads
.git/refs/heads/hoge  # <-- 新規作成したブランチのrefsオブジェクト
.git/refs/heads/main
.git/refs/heads/feat_counter
.git/refs/remotes
.git/refs/remotes/origin
.git/refs/remotes/origin/HEAD
.git/refs/remotes/origin/main
.git/refs/remotes/origin/feat_counter
.git/refs/tags

$ cat .git/refs/heads/hoge
00b44893244eb9a84f6ad39536609c9a11c11663

$ git branch hoge/fuga
fatal: cannot lock ref 'refs/heads/hoge/fuga': 'refs/heads/hoge' exists; cannot create 'refs/heads/hoge/fuga'

# 検証２：XXX/YYYブランチが既に存在する場合、XXXブランチは作成できるか？
$ git branch xxx/yyy

$ git branch
  hoge
  main
* feat_counter
  xxx/yyy

$ find .git/refs
.git/refs
.git/refs/heads
.git/refs/heads/hoge
.git/refs/heads/main
.git/refs/heads/feat_counter
.git/refs/heads/xxx
.git/refs/heads/xxx/yyy
.git/refs/remotes
.git/refs/remotes/origin
.git/refs/remotes/origin/HEAD
.git/refs/remotes/origin/main
.git/refs/remotes/origin/feat_counter
.git/refs/tags

$ cat .git/refs/heads/xxx/yyy
00b44893244eb9a84f6ad39536609c9a11c11663

$ git branch xxx
fatal: cannot lock ref 'refs/heads/xxx': 'refs/heads/xxx/yyy' exists; cannot create 'refs/heads/xxx'
```

参考：

https://stackoverflow.com/questions/2527355/using-the-slash-character-in-git-branch-name

https://stackoverflow.com/a/3651867/22007575

## refs はいつ生成されるのか

ブランチを新規生成したとき

`.git/refs/heads/`以下に新規作成したブランチの refs オブジェクトが生成される。

リモートへ push したとき

`.git/refs/remotes/`以下の push 先のリモートブランチが生成される。

#### 新規ブランチを作成したときに生成される refs はいったいどのコミットオブジェクトを指しているのか?

その分岐元となるコミットである。

下記の検証の通り、main ブランチの refs のハッシュ値と、main から分岐して新規作成した temporary ブランチの refs のハッシュ値は同じであった。

```bash
$ git branch
* main
  feat_coutner

$ cat .git/refs/heads/main
ff02ab7148bde5dae74fef4d11f8a93caadcb7b0

$ git branch temporary

$ find .git/refs
.git/refs
.git/refs/heads
.git/refs/heads/main
.git/refs/heads/modify_git_error_note
.git/refs/heads/temporary
.git/refs/remotes
.git/refs/remotes/origin
.git/refs/remotes/origin/HEAD
.git/refs/remotes/origin/main
.git/refs/remotes/origin/modify_git_error_note
.git/refs/tags

$ cat .git/refs/heads/temporary
ff02ab7148bde5dae74fef4d11f8a93caadcb7b0
```

#### 作成したブランチでコミットをしていくと、`refs/heads/xxx`以下の refs オブジェクトはそのブランチ上の最新のコミットに更新されていくのか？

その通り。

`refs/heads/xxx`オブジェクトはコミットのたびに最新のコミットを参照する

下記の通り、最初の`.git/refs/heads/temporary`からコミットした後の動オブジェクトの参照値が変わっているのが確認できる

```bash
$ cat .git/refs/heads/temporary
ff02ab7148bde5dae74fef4d11f8a93caadcb7b0

$ git switch temporary
Switched to branch 'temporary'

$ git add .
$ git commit -m "temporary first commit"
[temporary 89bcace] temporary first commit
 1 file changed, 490 insertions(+)
 create mode 100644 xxxx.txt

$ cat .git/refs/heads/temporary
89bcace757247739ceab4fc3b359dcc3a7658af6

$ git log --oneline
89bcace (HEAD -> temporary) temporary first commit
ff02ab7 (main) ZZZZZZZZZZZZZZZZZZZZZZZZZ
232c1fc (origin/main, origin/HEAD) xxxxxxxxxxxxxxxxxxxxxxxxxx
# ...
```

#### HEAD refs

`HEAD`は現在のコミットオブジェクトを指す refs オブジェクト... を指す refs オブジェクトである。

つまり、参照の参照である。

ブランチをチェックアウトすると、HEAD の参照する対象が変更されるのが確認できる

```bash
$ git branch
  main
* feat_counter

$ cat .git/HEAD
ref: refs/heads/feat_counter

$ git switch main

$ cat .git/HEAD
ref: refs/heads/main
```

HEAD は他にも、コミット時に生成するコミットオブジェクトの親コミットハッシュ値の参照のために利用される。

つまりコミットオブジェクトの親コミットのハッシュ値は HEAD の参照先がその値となるのである。

## `git reflog`

## はしりがき

git gc --prune=now をしていた

```bash
$ history
 # ...
 1900  exec $SHELL -l
 1901  cd sandbox-editor
 1902  git status
 1903  git branch
 1904  code .
 1905  npm run test
 1906  git status
 1907  exit
 1908  exec $SHELL -l
 1909  cd sandbox-editor
 1910  code .
 1911  npm run start
 1912  npm run build
 1913  git status
 1914  git log --oneline
 1915  git add .
 1916  git commit -m "Test: WIP Setting up test environment with jest and RTL. 4"
 1917  git push origin test/setup-test
 1918  exit
 1919  exec $SHELL -l
 1920  cd sandbox-editor
 1921  git branch
 1922  git status
 1923  git log --oneline
 1924  git add .
 1925  git commit -m "Test: WIP Setting up test environment with jest and RTL. 5"
 1926  git status
 1927  git push origin test/setup-test
 1928  git gc --prune=now
 1929  cat refs/remotes/origin/test/setup-test
 1930  ls refs/remotes/origin/test/setup-test
 1931  ls refs/remotes/origin/
 1932  find .git/refs
 1933  cat .git/refs/remotes/origin/test/setup-test
 1934  cat .git/refs/remotes/origin/test
 1935  git check-ref-format
 1936  git check-ref-format .git/refs/remotes/origin/test/setup-test
 1937  git check-ref-format test/setup-test
 1938  git check-ref-format --branch test/setup-test
 1939  cat .git/index
 1940  exec $SHELL -l
 1941  cd sandbox-editor
 1942  git status
 1943  git branch
 1944  code .
 1945  git status
 1946  git branch
 1947  git push origin test/setup-test
 1948  find .git/refs
 1949  git cat-file --p .git/refs/remotes/origin/test/setup-test
 1950  cat .git/refs/remotes/origin/test/setup-test
 1951  cat .git/refs/heads/test
 1952  ls -l .git/refs
 1953  git pull origin test/setup-test
 1954  exit
 1955  exec $SHELL -l
 1956  cd sandbox-editor
 1957  git branch
 1958  git gc
 1959  mv .git/logs/remotes/origin/test/setyp-test /tmp
 1960  mv .git/logs/remotes/origin/test/setup-test /tmp
 1961  mv .git/logs/refs/remotes/origin/test/setup-test /tmp
 1962  git gc
 1963  mv .git/remotes/origin/test/setup-test /tmp
 1964  mv .git/refs/remotes/origin/test/setup-test /tmp
 1965  git gc
 1966  git branch
 1967  git status
 1968  git switch development
 1969  git status
 1970  git switch test/setup-test
 1971  git branch
 1972  git status
 1973  find ./git/refs
 1974  find .git/refs
 1975  git push origin test/setup-test
 1976  git fetch origin test/setup-test
 1977  git merge origin/test/setup-test
 1978  find .git/refs
 1979  exit
 1980  exec $SHELL -l
 1981  cd sandbox-editor
 1982  git branch
 1983  find .git/refs
 1984  ls .git/refs/heads/test
 1985  ls .git/refs/heads/feat
 1986  ls .git/refs/heads/view
 1987  cat .git/refs/heads/view
 1988  git status
 1989  git branch
 1990  git reflog
 1991  exit
 1992  exec $SHELL -l
 1993  cd sandbox-editor
 1994  history
 1995  git branch
 1996  git status
 1997  ls /tmp
 1998  history | grep prune
 1999  exit
 2000  exec $SHELL -l
 2001  ls
 2002  mkdir nodejs
 2003  cd nodejs
 2004  node --version
 2005  pwd
 2006  code .
 2007  npm init -y
 2008  npm run start
 2009  cd ../sandbox-editor
 2010  git status
 2011  git branch
 2012  git switch develop
 2013  git switch development
 2014  git status
 2015  find .git/refs
 2016  git switch view/improvement
 2017  git branch
 2018  find .git/refs
 2019  git fetch origin view/improvement
 2020  find .git/refs
 2021  git merge origin/view/improvement
 2022  ls .git/refs/heads/view
 2023  history
```
