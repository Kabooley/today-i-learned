# git show

## 特定のコミットについて情報を得たいとき

https://stackoverflow.com/a/11007594/22007575

```bash
$ git log --oneline
0bd6efe (HEAD -> development, origin/development) Fix: Editor operation and file linkage
903e4dd Fix XXXXXXXXXXXXXXXXXXXXXXXXXX
ba61531 Feat: XXXXXXXXXXXXXXXXXXXXXXXXXXXXX
9b14fe0 Feat: ZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZ
f549b8d Fix: YYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY
# ....
$ git show ba61531
# commitした人、日時、差分などを出力してくれる

# 特定のファイルのみ出力するなどできる
$ git show ba61531:src/components/Counter.tsx
```
