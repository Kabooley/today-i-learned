# Chrome Devtools: IndexedDB

## IndexedDB の中身をささっと知りたいとき

https://developer.chrome.com/docs/devtools/storage/indexeddb#snippets

Chrome Devtools `Source` --> `Snippets`から新規の snippet を追加して、
現在の IndexedDB へアクセスするための JavaScript コードを書くことができる

```bash
# DB
sandbox-editor--set-of-dependency--cachde-v1-db
# store
sandbox-editor--set-of-dependency--cachde-v1-store
# このストアは、オンラインエディタの依存関係が保存されてあるとする
```

上記の store から登録されてあるデータを読み取りたいとき。

例）保存されてあるデータのうち、package.json ファイルを読み取りたいとき

```JavaScript
// IndexedDBの特定のDBを開く
const storeOfSetOfDeps = indexedDB.open('sandbox-editor--set-of-dependency--cachde-v1-db');

// DBが開けたのかどうかは非同期なので
// 開けたらonsuccesイベントリスナで
storeOfSetOfDeps.onsuccess = (e) => {
    const db = e.target.result;

    const transactions = db.transaction(['sandbox-editor--set-of-dependency--cachde-v1-store']);
    const setOfDepsStoreTransaction = transactions.objectStore('sandbox-editor--set-of-dependency--cachde-v1-store');

    setOfDepsStoreTransaction.openCursor().onsuccess = (event) => {
      const cursor = event.target.result;
      if (cursor) {
        // ここでやっと、storeに保存されたkey-valueペアにアクセスできる
        for(const [key, value] of cursor.value.entries()) {
            if(key.includes('package.json')) {
                console.log(value);
            }
        }
        cursor.continue();
      } else {
        console.log("No more entries!");
      }
    };
}
``
```
