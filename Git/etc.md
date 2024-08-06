# Git あれこれ

## HEAD~や HEAD~~や HEAD~3 の意味

https://git-scm.com/docs/git-rev-parse#Documentation/git-rev-parse.txt-emltrevgtltngtemegemHEADmaster3em

`HEAD~`: HEADref が指すコミットの親コミットのこと

`HEAD~~`:

> リビジョン パラメーターの接尾辞 ~<n> は、最初の親のみに続く、名前付きコミット オブジェクトの <n> 番目の世代の祖先であるコミット オブジェクトを意味します。つまり、 <rev>~3 は <rev>^^^ と同等であり、これは <rev>^1^1^1 と同等です

つまり、

コミットオブジェクトは親コミットの情報を持っているので親コミットの親コミットの親コミットの...と現在のコミットから祖先をたどることを指定できるのである

次を試すとわかりやすいかも

```bash
# rev-parseは指定したコミットのshaを返す
$ git rev-parse HEAD
3f8a5c4c813a879253dd60f07ab0e068066a6ec0
```

なのであらかじめ git log でコミット履歴を出しておいて rev-parse に続けて HEAD~~や HEAD~3 が実際にどのコミットを指すのか確認できる

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
2f6cf8c Feat: WIP. Replacing FilesContext to filesSlice. 3
ee1b826 Feat: WIP. Replacing FilesContext to filesSlice. 2
1284bec Feat: WIP. Replacing FilesContext to filesSlice. 1
0711b58 Feat: WIP replacing LayoutContext to layoutSlice

$ git rev-parse HEAD~
ba0c12c74e519139cb36d230fca919c978f20ac6
$ git rev-parse HEAD~3
3cbdeafe61c7770c82d9141396b0466a2023c640
```

つまり、

`HEAD~`: HEAD が指すコミットの親コミット

`HEAD~3`: 3 つ上の親コミット
