## Git:別ブランチの特定のファイルだけマージしたいとき

https://stackoverflow.com/questions/449541/how-can-i-selectively-merge-or-pick-changes-from-another-branch-in-git

案１：

https://jasonrudolph.com/blog/2009/02/25/git-tip-how-to-merge-specific-files-from-another-branch/

```bash
$ git checkout source_branch -- path/to/file
# Resolve conflicts if any
$ git commit -am "..."
```

要約すると、

特定のファイルのみ git checkout できるから、

その切ったブランチ(ファイル)をマージするだけ、みたいな。

```bash
# 今、その特定のファイルが存在するブランチにいるとして
$ git branch
  development
  feat_imple-monaco-class-component
  feat_multi-file
  feat_resizable-container
  feat_sandboxing-preview
  main
* temp_makesure_worker

# マージしたい方のブランチへ移動する
$ git switch feat_sandboxing-preview
k$ git branch
  development
  feat_imple-monaco-class-component
  feat_multi-file
  feat_resizable-container
* feat_sandboxing-preview
  main
  temp_makesure_worker
# 次の呪文を唱える
$ git checkout temp_makesure_worker -- experiments-worker-messaging.md
# するとその特定のファイルのみこのブランチに追加されているのが確認できる（add済っぽい）
$ git status
On branch feat_sandboxing-preview
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   experiments-worker-messaging.md
# 一応add
$ git add .
# コミット
$ git commit -m "Docs: Found how exchange messaging with worker in strictmode"
[feat_sandboxing-preview 3a7f53f] Docs: Found how exchange messaging with worker in strictmode
 1 file changed, 293 insertions(+)
 create mode 100644 experiments-worker-messaging.md

```
