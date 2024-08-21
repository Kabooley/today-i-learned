# git-rebase

## `You are currently editing a commit while rebasing branch 'test/setup-test' on '1d1c9bc'`

おそらくだけど、`git rebase`を完了しないままなんだが？ということを言っているらしい。

`test/setup-test`という現在のブランチではないブランチで git rebase したけれど、`git rebase --continue`し忘れてそのまま現在ブランチに来ているよという状態。

実際は、`test/setup-test`で`git rebase`したところ修正しきれないコンフリクトが発生したため、このトピックブランチを継続するのは不可能と判断し、ブランチを削除したいから一旦トピックブランチ上の git rebase する前の状態に`git reser --hard`でもどして、
ブランチを切り替えし`test/setup-test`を削除した

しかるのち新規のブランチを作成して編集し、git status を確認したところ表題のメッセージが現れたという次第。

てっきりブランチの削除迄したら git rebase の事実は一緒に消え去るものかと思ったけれどそうはならないというのが今回の教訓。

解決の流れ：

```bash
# 現状の状態
$ git branch
  *test/setup
   test/setup-test
   develop
   main

$ git status
On branch test/setup
Last command done (1 command done):
   pick d26312b Test: WIP Setting up test environment with jest and RTL. 1
Next commands to do (6 remaining commands):
   pick 56b4462 Test: WIP Setting up test environment with jest and RTL. 2
   pick 954f93f Test: WIP Enabled to test .ts .js files but .tsx .jsx. 3
  (use "git rebase --edit-todo" to view and edit)
You are currently editing a commit while rebasing branch 'test/setup-test' on '1d1c9bc'.
  (use "git commit --amend" to amend the current commit)
  (use "git rebase --continue" once you are satisfied with your changes)

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   package.json
        modified:   src/utils/moveInArray.ts
        modified:   yarn.lock

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        .babelrc.json
        __tests__/
        docs/test-setup.md
        jest.config.js
        tsconfig.jest.json

no changes added to commit (use "git add" and/or "git commit -a")

# 現在のトピックブランチをいったん保留する
$ git stash push -u -m "leave message you need"

# するとrebaseに関するメッセージの未確認できる
$ git status
On branch test/setup
Last command done (1 command done):
   pick d26312b Test: WIP Setting up test environment with jest and RTL. 1
Next commands to do (6 remaining commands):
   pick 56b4462 Test: WIP Setting up test environment with jest and RTL. 2
   pick 954f93f Test: WIP Enabled to test .ts .js files but .tsx .jsx. 3
  (use "git rebase --edit-todo" to view and edit)
You are currently editing a commit while rebasing branch 'test/setup-test' on '1d1c9bc'.
  (use "git commit --amend" to amend the current commit)
  (use "git rebase --continue" once you are satisfied with your changes)

nothing to commit, working tree clean

# 問題のブランチはすでに削除済であり、且つgit rebase --abortはどのブランチでも関係ないので
# このままこのブランチでabortする
$ git rebase --abort

# するとなぜかかつてのブランチに勝手に移動した...
$ git branch
   test/setup
  *test/setup-test
   develop
   main

# 念のためまだrebaseに関するメッセージが表示されるのか確認
$ git status
On branch test/setup-test
nothing to commit, working tree clean

# rebaseに関するメッセージが表示されなくなったので
# もともと削除予定のこのブランチを削除する
$ git checkout test/setup
$ git branch -D test/setup-test

# 念のため状況を確認
$ git status
# 問題なければ退避しておいた編集内容を戻す
$ git stash list
$ git stash apply stash@{0}

# 最終確認
$ git status
```

git rebase --abort するといきなりブランチが勝手に移動しているなどあるので
作業は慎重に進めるべき。

#### git rebase したあとはどうするべきであったのか

通常トピックブランチがベースブランチの現在の進行状況を取り込むために配下の手順を踏むらしい

つまり、add -> --continue -> commit である。

```bash
$ git rebase <base-branch>
# コンフリクトがあるなら解消
$ git add .
$ git rebase --continue
$ git commit
```

先の問題は git rebase の後 git reset をした。

どうやら git rebase をしたらなんであれ git rebase --continue するまで rebase は完了した扱いにならないらしい
