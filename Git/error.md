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


## unable to resolve reference 'refs/remotes/origin/test/setup-test': reference broken

https://stackoverflow.com/questions/2998832/git-pull-fails-unable-to-resolve-reference-unable-to-update-local-ref

https://git-scm.com/book/ja/v2/Git%E3%81%AE%E5%86%85%E5%81%B4-%E3%83%A1%E3%83%B3%E3%83%86%E3%83%8A%E3%83%B3%E3%82%B9%E3%81%A8%E3%83%87%E3%83%BC%E3%82%BF%E3%83%AA%E3%82%AB%E3%83%90%E3%83%AA

```bash
$ find .git/refs
.git/refs
.git/refs/tags
.git/refs/remotes
.git/refs/remotes/origin
.git/refs/remotes/origin/fix
.git/refs/remotes/origin/test
# あるやんけ
.git/refs/remotes/origin/test/setup-test
.git/refs/remotes/origin/feat
.git/refs/remotes/origin/view
.git/refs/heads
.git/refs/heads/test
.git/refs/heads/feat
.git/refs/heads/view
```

#### そもそもgitブランチに`/`を含めてもよいのか？

https://stackoverflow.com/questions/2527355/using-the-slash-character-in-git-branch-name

https://stackoverflow.com/a/3651867/22007575

`git check-ref-format --branch <branchname-shorthand>`で
渡された**参照名**が許容できるものかなどを診断してくれるみたい。

https://git-scm.com/docs/git-check-ref-format

```bash
```

曰く、

> gitでは参照の名前付けに関して次のルールが適用される：

> 1. 階層 (ディレクトリ) グループ化のためにスラッシュ `/` を含めることができますが、スラッシュで区切られたコンポーネントを`.`で始めることはできません。または`.lock`で終わるようなシーケンス。


ということで、


参照に対しては`/`を含めることは可としているが、参照とはそもそも何ぞ？
 

 ## git `ref`について

https://git-scm.com/book/en/v2/Git-Internals-Git-References

