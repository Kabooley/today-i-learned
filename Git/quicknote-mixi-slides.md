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

ちなみにディレクトリが追加されても tree オブジェクト（これも git オブジェクトの一つ）は生成されず、blob オブジェクトしか生成されていないことがわかる。

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

## commit するとは

1. index オブジェクトから tree オブジェクトを生成
2. commit オブジェクトを生成
3. HEAD を新しい commit ハッシュに書き換える

```bash
# 先の続きからそのままコミットしたとして
$ git commit -m "Docs: WIP note how to resolve reference error. 1"
$ git log --oneline
8d53fbf (HEAD -> modify_git_error_note) Docs: WIP note how to resolve reference
error. 1
...
$ git cat-file -p 8d53fbf
tree 4d9ca95d60bad0f6bc52acc65d9dd5ab98e67529
parent 232c1fc6f24e95cb7602bed70bdca2b5c81468ad
author XXXXXX <0000000+XXXXXX@users.noreply.github.com> 1710761722 +0900
committer XXXXXX <0000000+XXXXXX@users.noreply.github.com> 1710761722 +0900

Docs: WIP note how to resolve reference error. 1
# treeオブジェクトを見てみると、ルートディレクトリ直下のアイテム全てに対してblobまたはtreeオブジェクトが生成されているのがわかる
$ git cat-file -p 4d9ca95d60bad0f6bc52acc65d9dd5ab98e67529
100644 blob 3c4248f0d0c15edcb5ab41afe4b8ba65a86294c0    .gitignore
040000 tree b5ca534e5bd2680016b4a304d8a07e21b3cfe4fc    Books
040000 tree 66708307c8a65ffc340e95d889456125b7745e00    Git
040000 tree b7e86d190fab4b2b030fc156e3f20544e1b6ca13    JavaScript
040000 tree 858f5489d31bfb733b13b112367233a97959f434    Linux
040000 tree 3eb752348920a44ee6803095e627df55b7eef31e    Nodejs
100644 blob 0a45e9f47516132a1cab2ebe308e3290d64e7183    README.md
040000 tree e19d06a4e7de3acb0110523c5fc2b6c0ab8256c0    React
040000 tree 7cdb8b456b5cff359a8a5e08e1074902e1cd6613    SSH
040000 tree 9e39947655bf5c6aefa64f393e766e8f9f53b919    TypeScript
040000 tree 9d144515c3ca2a292f2984c164bf030c8b3846f9    Web
040000 tree 1c99bf458340bd2fb0ec56e3a972a2f4c43ccad7    contribute
040000 tree af6131321dd56323ac47529b4709e911f3d516eb    devtools
100644 blob 13d428792f8ce808c102984279368c8c31b8440b    schedule.md
040000 tree 59bedadb081da21e39f0e1c2c304a08f61f39085    security
040000 tree d4ff4d4ad813b347e606fa7d3021791e75753d02    temporary
$
$
```

コミットすると、

1. tree オブジェクトの生成：

リポジトリのルートディレクトリを含むすべてのディレクトリ分の tree オブジェクトを生成する

その際、変更があった部分だけ、新しい blob オブジェクトと tree オブジェクトが生成され、
変更がなかった部分は前回の tree オブジェクトがコピーされる

上記の通り、

blob オブジェクトはファイル内容を、tree オブジェクトはディレクトリ内容を記録する。

例えば Books ディレクトリ以下の git オブジェクトを調べたかったら Books の tree オブジェクトのハッシュ値を改めて cat-file すれば確認できる

こうして**blob オブジェクトと tree オブジェクトでコミットした瞬間のリポジトリの状態を再現できるようになっている**

2. commit オブジェクトの生成：

上記のように tree オブジェクト、blob オブジェクトの生成が完了したらコミットオブジェクトを生成して、

上記の通り、ルートオブジェクトの tree オブジェクト、parent オブジェクト（前回のコミット・オブジェクト）、作成者情報などが生成される

こうして、コミットオブジェクトは前回のコミットオブジェクトを参照しているのでコミットのつながりが把握できる

3. HEAD を新しい commit ハッシュへ書き換え：

その前に refs について

#### git references

refs は特定の commit を指すポインタのようなもの

そして HEAD は ref の一種である

refs には

-   light weight tag: 特定の commit オブジェクトを指すだけ
-   annotated tag: コメントがつけられるタグのこと
-   branch: light weight tag と変わらないが、保存場所が.git/refs/heads 以下にある

ブランチについて：

```bash
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
.git/refs/heads/main
$ cat .git/refs/heads/main
232c1fc6f24e95cb7602bed70bdca2b5c81468ad
```

曰く、

ブランチを生成すると、refs オブジェクトが生成される

でこの refs オブジェクトは`./git/refs/`へ保存される。

そのため、ブランチ名に`/`を含めるとブランチ名の生成に制限がかかることになる。

どういうことかというと、

`test/setup-test`というブランチを作ったとすると、

`.git/refs/test/setup`という refs が生成されることになるため、

`.git/refs/test`という refs は新規に作れなくなる

結果`test`というブランチが作れなくなるということになる。

HEAD について：

HEAD は現在の commit オブジェクトを指す

なので checkout すると HEAD は書き換わる

```bash
$ git branch
    main
    develop
    *modify_git_error_note
# HEAD refsには現在のコミットが書き込まれている
$ cat .git/HEAD
    ref: refs/heads/modify_git_error_note
# なのでたとえばブランチを移動すると
$ git switch main
$ cat .git/HEAD
    ref: refs/heads/main
# コミットハッシュ名でgit switchするとHEAD refsにはそのコミットハッシュ名が書き込まれる
```

ということで、コミット時に git は HEAD ref オブジェクトを書き換えるが、
