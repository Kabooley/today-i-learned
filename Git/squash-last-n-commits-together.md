# Squash last N commits together

現在のコミットの前のコミット複数を squash（もしくは同じこと）したいとき

## git reset --soft

https://stackoverflow.com/a/5201642/22007575

#### 実際に試してみた

HEAD コミットと、その前の WIP コミット 2 つを一つのコミットにまとめる

以下の log だと、sha は 3f8a5c4, ba0c12c, a2854ac の３つ。

```bash
$ git log --oneline
3f8a5c4 (HEAD -> feat/loading, origin/feat/loading) Feat: Added skeleton animation to explorer & editor while initial mounting.
ba0c12c WIP: Implementing skeleton animation to explorer. 2
a2854ac WIP: Implementing skeleton animation to explorer. 1.
3cbdeaf Docs: note trivial things
eb46651 Merge branch 'feat/redux' into feat/loading
e9008d8 (origin/feat/redux, feat/redux) Feat: Added packageJsonSlice.ts to reflect latest dependencies to package.json file.
aa050bb Feat: Replaced TypingLibsContext with typingLibsSlice.
8c5a78e Feat: WIP Replacing TypingLibsContext to typingLibsSlice. 1
3776876 Feat: Replaced FilesContext to filesSlice.
2f6cf8c
...

# まず、どこまでのコミットの指定が間違わないようにHEAD~の指定方法が正しいか確認する
$ git rev-parse HEAD~2
a2854ac374479663e8f583d85eba4ff9c0bf1eef
# `HEAD~2`の指定で問題ないので

$ git reset --soft HEAD~2
$ git commit
```

できた

#### ローカルで git reset した後に push 済のリモートブランチへ reset 済のコミットを push するには

問題はこれ。

つまり、git reset する前の以下のコミットはすべてリモートブランチにコミット済である

```bash
$ git log --oneline
3f8a5c4 (HEAD -> feat/loading, origin/feat/loading) Feat: Added skeleton animation to explorer & editor while initial mounting.
ba0c12c WIP: Implementing skeleton animation to explorer. 2
a2854ac WIP: Implementing skeleton animation to explorer. 1.
3cbdeaf Docs: note trivial things
eb46651 Merge branch 'feat/redux' into feat/loading
e9008d8 (origin/feat/redux, feat/redux) Feat: Added packageJsonSlice.ts to reflect latest dependencies to package.json file.
aa050bb Feat: Replaced TypingLibsContext with typingLibsSlice.
8c5a78e Feat: WIP Replacing TypingLibsContext to typingLibsSlice. 1
3776876 Feat: Replaced FilesContext to filesSlice.
2f6cf8c
```

そのあとで git reset -> git commit したら、リモートブランチと矛盾が生じる。

```bash

$ git reset --soft HEAD~2
$ git commit
$ git push origin feat/laoding
To github.com:Kabooley/sandbox-editor.git
 ! [rejected]        feat/loading -> feat/loading (non-fast-forward)
error: failed to push some refs to 'github.com:Kabooley/sandbox-editor.git'
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. Integrate the remote changes (e.g.
hint: 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.

```

解決方法は 2 つ。

1. git pull origin feat/loading してコンフリクトを解決する
2. git push --force を使って強制 push する

pull する方法：

うまくいかなかった。結局 pull もできない

--force の方法：

個人開発ならいいかも。チームで開発している場合他人の push 内容も消してしまうので使ってはならない。

では解決方法がないのでは？

git revert を使うといいのかな？わからなかった
