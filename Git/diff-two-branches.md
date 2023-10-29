# 2 つのブランチの比較

https://stackoverflow.com/questions/9834689/how-do-i-see-the-differences-between-two-branches

https://git-scm.com/docs/git-diff

`git diff <commit>..<commit>`

```bash
$ git branch
*   development
    feat_somefeat
$ git log --oneline
5cbfh4 (HEAD--> XXXX) XXXXX

$ git switch feat_somefeat
$ git log --oneline
6hbfh2 (HEAD--> XXXX) XXXXX

$ git switch development
$ git diff 5cbfh4..6hbfh2
diff --git a/codesnadbox-test-resizable/index.html b/codesnadbox-test-resizable/index.html
deleted file mode 100644
index ef6efdc..0000000
--- a/codesnadbox-test-resizable/index.html
+++ /dev/null
@@ -1,51 +0,0 @@
-<!DOCTYPE html>
-<html lang="en">
-    <head>
-        <meta charset="UTF-8" />
-        <meta http-equiv="X-UA-Compatible" content="IE=edge" />
# brahbrahbrah...


```
