# git log

## `git log --graph --oneline --all`

https://stackoverflow.com/a/4991675/22007575

これまでの git の推移をグラフでターミナルで確認できるコマンド

```bash
#
$ git log --graph --oneline --all
* 0bd6efe (HEAD -> development) Fix: Editor operation and file linkage
*   903e4dd Fix conflict by accepting ba61531 commit
|\
| *   332f637 (origin/development) Merge branch 'view/improvement' into development Reflect view improvement so far.
| |\
| | | * d4cdaa9 (origin/fix/file-management, fix/file-management) Added docs/monaco-editor.md and renamed docs/fix-file-management.md to file-management.md
| | | * bcf1020 Fix: Make monaco extraLibs to be updated when a file deleted
| | | * d81a4b2 Fix: Files selectable by clicking file on Explorer/workspace
| | | * 557d4d2 Fix: New file on workspace is now displayed on editor
| | | * 30b778e Fix: WIP fixing managing monaco-editor's ITextModel. 2
| | | * 2e03fb1 Fix: WIP fixing managing monaco-editor's ITextModel. 1
| | | * e82f3ba Fix: Fixed unchanging model's value on change select tab
| |_|/
|/| |
* | | ba61531 Feat: Enabled updating dependency on modifying package.json.
| | | * 1a64303 (origin/improve-dependency-management, improve-dependency-management) Feat: Done adding auto acquiring type files when package.json modified.
| | | * 8225a6c Feat: WIP Adding auto acquiring type files when package.json modified. 5
| | | * 4104da6 Feat: WIP Adding auto acquiring type files when package.json modified. 4
| | | * 0c3aaad Feat: WIP Adding auto acquiring type files when package.json modified. 3
| | | * ed8d92e Feat: WIP Adding auto acquiring type files when package.json modified. 2
| | | * 58c762b Feat: WIP Adding auto acquiring type files when package.json modified. 1
| | | * 5247168 Feat; WIP Implementing deletion of dependency. 1
| | | * 381e05d Feat; WIP Implementing deletion of dependency
| | | * c6f7229 Fix: Fixed handleWorkerMessage to be able to access updated React state
| | | | * 3901db3 (origin/view/improvement, view/improvement) Feat: WIP adding action to TabsAndActions. 1
| | | |/
| | |/|
```
