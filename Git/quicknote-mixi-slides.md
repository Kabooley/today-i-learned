# mixi 新卒研修スライドのノート

## Git の内部

## add するとは

git add が行っていることとは、index の更新、blob オブジェクトの生成である

```bash
$ git status
On branch modify_git_error_note
Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        deleted:    "Git/\343\202\250\343\203\251\343\203\274.md"

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        Git/error.md
        Git/quicknote-mixi-slides.md
        Git/references.md
$ git add .
# indexは.git/indexに存在する
$ cat .git/index
# これをする内容によっては大変重たいデータが出力される

# ls-filesはgitのアクティブなコピーを調べ結果を表示してくれるコマンドで
# catするよりこちらでindexについて表示させる
$ git ls-files --stage
# 先に`git add .`したのでそのディレクトリのすべてのファイルがindexに登録されているのが確認できる
# ファイルの種類+パーミッション blobハッシュ コンフリクトフラグ ステージングしたファイル
100644 f409fb1d55405384b9835491907e0e719a486270 0       Git/error.md

```

ちなみにディレクトリが追加されてもtreeオブジェクト（これもgitオブジェクトの一つ）は生成されず、blobオブジェクトしか生成されていないことがわかる。


-   git add した対象のファイルが git/index にもれなく追加されていることがわかる
-   blob オブジェクトが生成されていることがわかる

#### blob ハッシュ

git が管理するオブジェクトのうちの一つで、ファイルの情報が入っている

git ls-files の出力内容を見ての通り、各ファイルに対して一つ生成されているのがわかる

このハッシュ値は JavaScript の Map オブジェクトのように key-value 関係の key の方であり
このハッシュとそのファイルの関連情報がペアになっている

`git cat-file -p <object>`は git オブジェクトの中身や種類について教えてくれるコマンドである

```bash
$ git ls-files --stage
100644 f409fb1d55405384b9835491907e0e719a486270 0       Git/error.md
...
$ git cat-file -p f409fb1d55405384b9835491907e0e719a486270
# Git/error.mdの中身が表示された
```

参考:

https://stackoverflow.com/questions/56235287/what-does-git-ls-files-do-exactly-and-how-do-we-remove-a-file-from-it


## commitするとは

1. index オブジェクトから tree オブジェクトを生成
2. commit オブジェクトを生成
3. HEADを新しい commit ハッシュに書き換える

```bash
# 先の続きからそのままコミットしたとして
$ git commit
$ git log --oneline
$ git cat-file -p 
$ 
$ 
$ 
```