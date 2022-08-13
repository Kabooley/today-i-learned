# 実践と検証と整理： File System stream

dist/in/cat.png をストリームで dist/out/cat.png へコピーするプログラムを作って

stream の挙動を確認、検証する。

stream は`fs.createReadStrem`, `fs.createWriteStream`で生成されるインスタンスを使用する。

## 目次

[実践](#実践)

Readable

[Readable の実装方法](#Readableの実装方法)
[Readable のモード切替方法](#Readableのモード切替方法)
[chunk は直接渡していい](#chunkは直接渡していい)
[on()は EventTarget.addEventListener の Node.js 特化版](<#on()はEventTarget.addEventListenerのNode.js特化版>)

Writable

[Writable ストリームが自動で閉じる条件](#Writableストリームが自動で閉じる条件)
[`drain`と`writable.write()`の仕組み](<#`drain`と`writable.write()`の仕組み>)
[`writable.end()`と`writable.destroy()`の使い分け](<#`writable.end()`と`writable.destroy()`の使い分け>)
[エラーハンドリング](#エラーハンドリング)
[pipe](#pipe)
[実践：改善版](#実践：改善版)

### 実践

画像をコピーするプログラム。

次の仮定（NOTE: 思い込み）で作られている

- `Writable`ストリームは`autoClose:true`で作成されたので読み取るデータがなくなったら勝手に`Writable`は閉じられる
- `Readable`ストリームは読み取るデータがなくなったら勝手に`Readable`を閉じる
- 各ストリームは`highWaterMark`を 1024byte で指定しているので毎度ストリームは 1024byte 読み取って 1024byte 書き込む
- 各ストリームの各イベントでしなくてはならないことを理解していない
- `Readable`の`data`イベントで取得した chunk はそのまま`writable.write()`へわたしていいはず
- `Readable`は flowing モードで運用されるから`readable.readableFlowing`は true になるはず

```TypeScript
import * as stream from 'node:stream';
import * as fs from 'node:fs';
import * as path from 'node:path';
import * as crypto from 'node:crypto';

const outPath = path.join(__dirname, "out");
const inPath = path.join(__dirname, "in");

// ランダムな文字列を生成するやつ
//
// https://qiita.com/fukasawah/items/db7f0405564bdc37820e#node%E3%81%AE%E3%81%BF
const randomString = (upto: number): string => {
    const S="abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";
    return Array.from(crypto.randomFillSync(new Uint8Array(upto))).map((n)=>S[n%S.length]).join('');
}


// 指定のパスにディレクトリは存在するのか確認する関数
/*
https://stackoverflow.com/questions/15630770/node-js-check-if-path-is-file-or-directory

https://nodejs.org/dist/latest-v16.x/docs/api/fs.html#class-fsstats

*/
const isDirExist = (path: string): boolean => {
    return fs.lstatSync(path).isDirectory() && fs.existsSync(path);
}


const createRfs = (): fs.ReadStream => {
    if(!isDirExist(inPath)) throw new Error(`The path: ${inPath} does not exist.`);

    return fs.createReadStream(
        path.join(inPath, "cat.png"),
        {
            encoding: 'binary',     /* default: 'utf8' */
            autoClose: true,
            emitClose: true,
            highWaterMark: 1024     /* default: 64 * 1024 */
        }
    );


}


const createWfs = (): fs.WriteStream => {
    if(!isDirExist(outPath)) throw new Error(`The path: ${outPath} does not exist.`);

    return fs.createWriteStream(
        path.join(outPath, "cat" + randomString(5) + ".png"),
        {
            encoding: 'binary',     /* default: 'utf8' */
            autoClose: true,
            emitClose: true,
            highWaterMark: 1024     /* default: 64 * 1024 */
    });
}



(async function() {
    const rfs: fs.ReadStream = createRfs();
    const wfs: fs.WriteStream = createWfs();

    // --- Readable Event Handlers ------------


    rfs.on('open', () => {
        console.log("readable stream has been opened");
    });

    rfs.on('ready', () => {
        console.log("readable stream is ready");
    });

    rfs.on('close', () => {
        console.log('readable stream has been closed');
    });

    rfs.on('data', (chunk: string | Buffer) => {
        console.log('data read!');
        console.log(`state: ${rfs.readableFlowing}`);
        console.log(`Received ${chunk.length} bytes of data.`);

        wfs.write(chunk, (e: Error | null | undefined) => {
            if(e) console.error(e.message);
            else console.log("Write data has been completed");
        })
    });

    rfs.on('end', () => {
        console.log('End read stream');
        console.log('There is no more data to be consumed from the stream');
    });

    rfs.on('resume', () => {
        console.log('There is no more data to be consumed from the stream');
    });

    rfs.on('error', (e: Error) => {
        console.error(e.message);
        rfs.resume();
    });

    rfs.on('pause', () => {
        console.log("readable paused");
    })

    // --- Writable Event Handlers -----

    wfs.on('close', () => {
        console.log("Writable stream closed");
    });

    wfs.on('drain', () => {
        console.log("Drained");
    });

    wfs.on('finish', () => {
        console.log("Finished");
    });

    wfs.on('pipe', () => {
        console.log("PIPED!");
    });

    wfs.on('unpiped', () => {
        console.log("UNPIPED!!");
    })

    wfs.on('error', (e: Error) => {
        console.error(e.message);
        // 念のために明示的にストリームを破棄させる。
        if(wfs.destroyed) wfs.destroy();
    });

})();
```

ログ

```bash
tream
[start:*run] readable stream has been opened    # `open` Readable
[start:*run] readable stream is ready           # `ready` Readable

# "data" read.
# chunk size is exactly same as highWaterMark threshold.
[start:*run] data read!
[start:*run] state: true
[start:*run] Received 1024 bytes of data.

[start:*run] data read!
[start:*run] state: true
[start:*run] Received 1024 bytes of data.

# `data` has been written.
# `data` callback has been executed.
# なのでここでデータの書き込みが行われた
[start:*run] Write data has been completed


[start:*run] data read!
[start:*run] state: true
[start:*run] Received 1024 bytes of data.
[start:*run] Write data has been completed

[start:*run] data read!
[start:*run] state: true
[start:*run] Received 1024 bytes of data.
[start:*run] Write data has been completed

[start:*run] data read!
[start:*run] state: true
[start:*run] Received 1024 bytes of data.
[start:*run] Write data has been completed

[start:*run] data read!
[start:*run] state: true
[start:*run] Received 1024 bytes of data.
[start:*run] Write data has been completed

[start:*run] data read!
[start:*run] state: true
[start:*run] Received 1024 bytes of data.
[start:*run] Write data has been completed

[start:*run] data read!
[start:*run] state: true
[start:*run] Received 1024 bytes of data.
[start:*run] Write data has been completed

[start:*run] data read!
[start:*run] state: true
[start:*run] Received 1024 bytes of data.
[start:*run] Write data has been completed

[start:*run] data read!
[start:*run] state: true
[start:*run] Received 1024 bytes of data.
[start:*run] Write data has been completed

[start:*run] data read!
[start:*run] state: true
[start:*run] Received 1024 bytes of data.
[start:*run] Write data has been completed

[start:*run] data read!
[start:*run] state: true
[start:*run] Received 1024 bytes of data.
[start:*run] Write data has been completed

[start:*run] data read!
[start:*run] state: true
[start:*run] Received 1024 bytes of data.

[start:*run] data read!
[start:*run] state: true
[start:*run] Received 1024 bytes of data.
[start:*run] Write data has been completed

[start:*run] data read!
[start:*run] state: true
[start:*run] Received 905 bytes of data.

# readable `end` event
[start:*run] End read stream
[start:*run] There is no more data to be consumed from the stream

# writable `data` event callback has been executed
[start:*run] Write data has been completed
[start:*run] Write data has been completed

# readable `close` event
[start:*run] readable stream has been closed

# writable `drain` event
[start:*run] Drained

# writable `data` event callback has been executed
[start:*run] Write data has been completed

# 正常終了
[start:*run] [nodemon] clean exit - waiting for changes before restart
```

画像は正常に取得できたが...プログラムはＡＰＩを正しく使えなかったようだ。

期待通りじゃなかったこと：

- `Writable`ストリームが閉じられていない
- `drain`イベントが一度しか起こらなかった

期待通りだったこと：

- `Readable`は highWaterMark で指定した通りのサイズを読み取ってきた
- `Readable`は読み取るデータがなくなったら閉じられた

ここ以降、実践でうまくいかなかった原因を追究してその学習内容をまとめ、

最終的には API を正しく使用できているプログラムに改善する。

## Readable の実装方法: flowing モード

次の通りに実装するだけ。

問題は起こらない。

```TypeScript
import * as fs from 'node:fs';
import * as path from 'node:path';
import * as crypto from 'node:crypto';

const outPath = path.join(__dirname, "out");
const inPath = path.join(__dirname, "in");

const randomString = (upto: number): string => {
    // ...
}

const isDirExist = (path: string): boolean => {
    // ...
}

const createRfs = (): fs.ReadStream => {
    if(!isDirExist(inPath)) throw new Error(`The path: ${inPath} does not exist.`);

    return fs.createReadStream(
        path.join(inPath, "cat.png"),
        {
            encoding: 'binary',     /* default: 'utf8' */
            autoClose: true,
            emitClose: true,
            highWaterMark: 1024     /* default: 64 * 1024 */
        }
    );
}

(function() {
    const rfs: fs.ReadStream = createRfs();

    // --- Readable Event Handlers ------------


    rfs.on('open', (): void => {
        console.log("readable stream has been opened");
    });

    rfs.on('ready', (): void => {
        console.log("readable stream is ready");
    });

    rfs.on('close', (): void => {
        console.log('readable stream has been closed');
        wfs.end();
    });

    rfs.on('end', (): void => {
        console.log('End read stream');
        console.log('There is no more data to be consumed from the stream');
    });

    rfs.on('resume', (): void => {
        console.log('resume');
        console.log(`readaleFlowing: ${rfs.readaleFlowing}`);
    });

    rfs.on('error', (e: Error): void => {
        console.error(e.message);
        rfs.resume();
    });

    rfs.on('pause', (): void => {
        console.log("readable paused");
    })

    rfs.on('data', (chunk: string | Buffer): void => {
        console.log('data read!');
        console.log(`state: ${rfs.readableFlowing}`);
        console.log(`Received ${chunk.length} bytes of data.`);
    });
})();
```

```bash
5:09:47 AM - Starting compilation in watch mode...
[start:*build]
 [nodemon] restarting due to changes...
 [nodemon] starting `node ./dist/index.js`
 [nodemon] restarting due to changes...
[start:*build]
[start:*build] 5:09:48 AM - Found 0 errors. Watching for file changes.
 [nodemon] starting `node ./dist/index.js`

# resumeイベントはreadableFlowing is not trueの時に発行される
# trueやんけ
# たぶんコールバックが実行される頃にはtrueになるんだと思う
 resume
 readaleFlowing: true

# open
 readable stream has been opened
# ready
 readable stream is ready

# data
 data read!
 state: true
# highWaterMark通りの閾値を取得している
 Received 1024 bytes of data.
 data read!
 state: true
 Received 1024 bytes of data.
 data read!
 state: true
 Received 1024 bytes of data.
 data read!
 state: true
 Received 1024 bytes of data.
 data read!
 state: true
 Received 1024 bytes of data.
 data read!
 state: true
 Received 1024 bytes of data.
 data read!
 state: true
 Received 1024 bytes of data.
 data read!
 state: true
 Received 1024 bytes of data.
 data read!
 state: true
 Received 1024 bytes of data.
 data read!
 state: true
 Received 1024 bytes of data.
 data read!
 state: true
 Received 1024 bytes of data.
 data read!
 state: true
 Received 1024 bytes of data.
 data read!
 state: true
 Received 1024 bytes of data.
 data read!
 state: true
 Received 1024 bytes of data.
 data read!
 state: true
 Received 905 bytes of data.

# end
 End read stream
 There is no more data to be consumed from the stream
# close
 readable stream has been closed
 [nodemon] clean exit - waiting for changes before restart
```

流れ：

readable ストリームインスタンスが生成された

readable ストリームは`paused`モードになっている

このとき`readable.readableFlowing === null`である

`readable.readableFlowing === null`のとき、readable ストリームには何もデータ消費者が提供されていないことを示し、データ取得を行わない状態である。

`readable.readableFlowing !== true`なので`resume`イベントが発行される

`data`イベントハンドラがついているのですぐさま`flowing`モードに変更する

このとき`readable.readableFlowing === true`である

readable ストリームはデータの取得を開始する

`data`イベントハンドラが readable ストリームが取得してきた chunk を取得する

`highWaterMark`で指定した閾値通りの量を毎度取得してくる

readable ストリームの読み取り先から読み取るデータがなくなったら`end`イベントがまず発行される

`end`イベントは「読み取るデータがもうない」ことを示す

その後、createReadStream で autoClose, emitClose で true を渡したので自動的に`close`イベントが発行される

readable ストリームが正常に閉じられる

以上。

> flowing モードの readable ストリームはデータを消費できないとデータは失われる

`data`イベントハンドラを提供すればデータは失われない事が確認できた。

つまり、

> All Readable streams begin in paused mode but can be switched to flowing mode in one of the following ways:

> Adding a 'data' event handler.
> Calling the stream.resume() method.
> Calling the stream.pipe() method to send the data to a Writable.

の説明通りに提供すれば大丈夫

### paused モード

覚えておくべきこと：

- `stream.read()`を必ず明示的に呼び出さなくてはならない

- `data`イベントハンドラを呼び出してはならない

- `stream.resume()`を呼出してはならない

- `readable.pipe()`では paused モードにできない

- ストリームの接続先がないときに、`stream.pause()`を呼び出すと paused モードになる

- pipe されているときに、pipe を除去することで paused モードになる

- `data`イベントハンドラをリムーブしてもただちに paused モードになるわけではない

- パイプされた宛先がある場合、 stream.pause() を呼び出しても、それらの宛先が排出されて追加のデータが要求されると、ストリームが一時停止したままになるとは限りません。

- flowing モードでも`readable`イベントハンドラを追加すると flowing モードが解除される

- paused モードだと`readable.readableFlowing === false`になる

- (共通)一つのストリームからの消費者を複数用意してはならない。

必ず`on('data')`, `on('readble')`, `pipe()`いずれか一つを選ぶこと

- `stream.pause()`は`readable`イベントリスナがあるところでは何の効果もない。

- `readable.read()`は Readable が paused モードの時だけ使うこと

#### 実践：paused モード

`dist/in/text.txt`を読み取るストリーム。

`text.txt`は 40byte.

仮定：

学習した内容から次のコードはこうなるはずという仮定。

- flowing モードになるトリガーが一切なければ`readable.readableFlowing`は常に`false`になるはず

- `readable`イベントリスナがあるから`stream.pause()`を呼出しても意味をなさないはず
- `stream.resume()`を呼出したら`readable`は働かなくなるはず

```TypeScript
import * as fs from 'node:fs';
import * as path from 'node:path';

const isDirExit = () => {
    // Find out the directory is exists.
}

const createRfs = (): fs.ReadStream => {
    if(!isDirExist(inPath)) throw new Error(`The path: ${inPath} does not exist.`);

    return fs.createReadStream(
        path.join(inPath, "text.txt"),
        {
            encoding: 'utf8',     /* default: 'utf8' */
            autoClose: true,
            emitClose: true,
            highWaterMark: 12     /* default: 64 * 1024 */
        }
    );
}

const rfs: fs.ReadStream = createRfs();

rfs.on('error', (e: Error) => {
    console.error(e.message);
    // Destroy stream explicitly
    /***
     * オプショナルで`error`と`close`イベントを発行する
     *
     * destroy() が呼び出されると、それ以降の呼び出しは何も行われず、
     * _destroy() 以外のエラーは「エラー」として出力されることはありません。
     * */
    if(!rfs.destroyed) rfs.destroy(e);
});

rfs.on('close', () => {
    console.log('close');
    if(!rfs.destroyed) rfs.destroy();
});

rfs.on('end', () => {
    console.log("end");
    if(!rfs.destroyed) rfs.destroy();
});


rfs.on('pause', () => {
    console.log("Readable paused");
});

rfs.on('resume', () => {
    console.log('resume');
    console.log(`state: ${rfs.readableFlowing}`);
});



/***
 * `readable` event:
 *
 * `readable`イベントはストリームから読み取るデータがあるときまたは
 *
 * ストリームの「終わり」に到達したら発行されるイベントである。
 *
 * `readable`は効果的にストリームに新しい情報があることを示す
 *
 * もしもデータが得られるのならば、
 *
 * `stream.read()`がデータを返すよ
 *
 * 以下は公式の例と同じ。
 *
 * 場合によっては、
 *
 * `readable`イベントはかなりの量の塊を内部バッファへ保存してなくてはならない場合もある
 *
 * 一般的に、`readable.pipe()`や
 * */
 rfs.on('readable', () => {
    console.log("Readable Event");

    let chunk = rfs.read();
    // なくても大丈夫
    // たぶんストリームのコンストラクタにautoCLoseを渡してあるから
    if(chunk === null) rfs.close();
    else console.log(`Read ${chunk.length} bytes of data and...`);

    // stream.read()はストリームの終了に到達するとnullを返す
    // そして`end`イベントを発行する
    // つまりもう読み取るべきデータはないことを示す
    /***
     * 読み取れるデータがないとnullを返す
     *
     * オプショナルの`size`引数は読み取りサイズを指定する
     *
     * `size`byteが取得できないとき、ストリームが終了されていない限りnullが返される
     *
     * `size`が指定されていないと内部バッファのすべてが返される
     *
     * NOTE: `readable.read()`はReadableストリームがpausedモードであるときだけに使うこと
     *
     * と公式に明示されているよ！
     * */
});

```

```bash
Readable Event
Read 12 bytes of data and...
Readable Event
Read 12 bytes of data and...
Readable Event
Read 12 bytes of data and...
Readable Event
Read 4 bytes of data and...
Readable Event
end
close
```

検証２：途中で flowing モードに切り割るか？

```bash
# pausedモード
 Readable Event
 Read 12 bytes of data and...
 Readable Event
 Read 12 bytes of data and...
 Readable Event
 Read 12 bytes of data and...
 Readable Event
#  flowingモードへ切り替わった
 >> Switched to flowing mode.
 state: false
#  `data`イベントハンドラで「続き」を取得した
 [flowing] Read 4 bytes of data and...
#  `readable`イベントがまだ反応している...
 Read 4 bytes of data and...
#  resumeイベントハンドラの反応遅い
 resume
#  いまだにreadableFlowingはfalseのまま
# よみとるデータがないせいかも？
 state: false
 Readable Event
 >> Switched to flowing mode.
 state: false
 resume
 state: false
 end
 CLOSE
```

```bash
 Readable Event
 Read 12 bytes of data and...
 Readable Event
 Read 12 bytes of data and...
 Readable Event
 Read 12 bytes of data and...
 Readable Event
 Read 4 bytes of data and...
 resume
 state: false
 >> Switched to flowing mode.
 Readable Event
 end
 CLOSE
```

やってみてわかったこと

切り替えはうまくいかない。

#### Readable のモード切替方法

Readable には 2 つのモードがある

- flowing モード：データの取得はシステムが自動的に行ってくれてデータはイベントハンドラで取得できる
- paused モード：`stream.read()`を明示的に呼び出してストリームのチャンクを取得する

この 2 つのモードを切り替える方法

flowing モードに切り替える方法：

- `data`イベントハンドラを追加する
- `stream.resume()`を呼び出す
- `stream.pipe()`で`Writable`へデータを送信する

paused モードに切り替える方法：

- pipe の到達地点がないときに、`stream.pause()`を呼出したとき
- pipe の到達地点があるときに`stream.unpipe()`を呼び出すと起こりうる

ということで、

`stream.resume()`は flowing モードに切り替えて、`stream.pause()`は flowing モードから解除をするので対照的

`stream.pause()`:

> readable.pause() メソッドは、フロー モードのストリームに「データ」イベントの発行を停止させ、フロー モードから切り替えます。利用可能になったデータは内部バッファに残ります。

`stream.resume()`:

> readable.resume() メソッドは、明示的に一時停止された Readable ストリームに「データ」イベントの発行を再開させ、ストリームをフロー モードに切り替えます。

対照的である。

注意：

- `stream.resume()`だけを呼出してもデータが失われる可能性がある。

なので`stream.resume()`で flowing モードに切り替えるときは`data`イベントハンドラなど消費者を用意しておかなくてはならない

## on()は EventTarget.addEventListener の Node.js 特化版

そんなの当たり前じゃんみたいな話をします。

つまり、何度も呼び出すと同じイベントリスナが何度も追加されていってしまうよ、ということです。

かつて私は自身が定義した書き込みストリームだと`drain`イベントが全く発行されない原因はスコープの問題かと思って

次のように`Readable`の`data`イベントのハンドラ内で`on()`を呼び出してしまいました。

```TypeScript

const rfs: fs.Readable = createReadStream(/* params */);
const wfs: fs.Writable = createWriteStream(/* params */);

rfs.on('data', (chunk: string | Buffer) => {
    console.log('data read!');
    console.log(`state: ${rfs.readableFlowing}`);
    console.log(`Received ${chunk.length} bytes of data.`);

    wfs.write(chunk, (e: Error | null | undefined) => {
        if(e) console.error(e.message);
        else console.log("Write data has been completed");
    });

    wfs.on('close', () => {
        console.log("Writable stream closed");
    });

    wfs.on('drain', () => {
        console.log("Drained");
    });

    wfs.on('finish', () => {
        console.log("Finished");
    });
});
```

すると次のように、

- `drain`イベントが読み取りごとに 1 つ増えていく
- メモリリークの警告を受ける

結果になりました

```bash
 There is no more data to be consumed from the stream

#  readableオープン
 readable stream has been opened
 readable stream is ready

 data read!
 state: true
 Received 1024 bytes of data.

 Drained
 Write data has been completed

 data read!
 state: true
 Received 1024 bytes of data.

 Drained
 Drained
 Write data has been completed

 data read!
 state: true
 Received 1024 bytes of data.

 Drained
 Drained
 Drained
 Write data has been completed

 data read!
 state: true
 Received 1024 bytes of data.

 Drained
 Drained
 Drained
 Drained
 Write data has been completed

 data read!
 state: true
 Received 1024 bytes of data.

 Drained
 Drained
 Drained
 Drained
 Drained
 Write data has been completed

 data read!
 state: true
 Received 1024 bytes of data.

 Drained
 Drained
 Drained
 Drained
 Drained
 Drained
 Write data has been completed

 data read!
 state: true
 Received 1024 bytes of data.

 Drained
 Drained
 Drained
 Drained
 Drained
 Drained
 Drained
 Write data has been completed

 data read!
 state: true
 Received 1024 bytes of data.

 Drained
 Drained
 Drained
 Drained
 Drained
 Drained
 Drained
 Drained
 Write data has been completed

 data read!
 state: true
 Received 1024 bytes of data.

 Drained
 Drained
 Drained
 Drained
 Drained
 Drained
 Drained
 Drained
 Drained
 Write data has been completed

 data read!
 state: true
 Received 1024 bytes of data.

 (node:2233) MaxListenersExceededWarning: Possible EventEmitter memory leak detected. 11 close listeners added to [WriteStream]. Use emitter.setMaxListeners() to increase limit
 (Use `node --trace-warnings ...` to show where the warning was created)
 (node:2233) MaxListenersExceededWarning: Possible EventEmitter memory leak detected. 11 drain listeners added to [WriteStream]. Use emitter.setMaxListeners() to increase limit
 (node:2233) MaxListenersExceededWarning: Possible EventEmitter memory leak detected. 11 finish listeners added to [WriteStream]. Use emitter.setMaxListeners() to increase limit
 (node:2233) MaxListenersExceededWarning: Possible EventEmitter memory leak detected. 11 pipe listeners added to [WriteStream]. Use emitter.setMaxListeners() to increase limit
 (node:2233) MaxListenersExceededWarning: Possible EventEmitter memory leak detected. 11 unpiped listeners added to [WriteStream]. Use emitter.setMaxListeners() to increase limit
 (node:2233) MaxListenersExceededWarning: Possible EventEmitter memory leak detected. 11 error listeners added to [WriteStream]. Use emitter.setMaxListeners() to increase limit

# 以下略
```

つまりは`on()`は何度も呼び出していいという謎の思い込みから解き放たれました。

`EventEmitter.on()`:

https://nodejs.org/dist/latest-v16.x/docs/api/events.html#emitteroneventname-listener

> eventName という名前のイベントのリスナー配列の末尾にリスナー関数を追加します。リスナーがすでに追加されているかどうかはチェックされません。 eventName とリスナーの同じ組み合わせを渡す複数の呼び出しにより、リスナーが複数回追加され、呼び出されます。

つまり、同じイベント名で何度も`on()`を呼び出せば呼び出しただけリスナが呼び出される。

余談：一度だけ発火したら解除されるイベントリスナのラッパーはある。

`EventEmitter.once()`：

https://nodejs.org/dist/latest-v16.x/docs/api/events.html#emitteronceeventname-listener

> `eventName`という名前のイベントに対して一度切りのリスナを追加することができます。次回`eventName`がトリガーされたときに、リスナは除去されまた呼び出されます。

`on()`も`once()`もどちらも`Writable`, `Readable`class で定義されています。

## Writable ストリームが自動で閉じる条件

結論：**書き込みストリームの破棄は明示的に実行しなくてはならない**

たとえば次のような使い方の時に、

`Writable`を作る時にコンストラクタに`autoClose: true`, `emitClose`を渡しても自動で`Writable`は自身を閉じてくれない。

```TypeScript
// rfs: fs.Readable
// wfs: fs.Writable
    rfs.on('data', (chunk: string | Buffer): void => {

        wfs.write(chunk, (e: Error | null | undefined): void => {
            if(e) console.error(e.message);
            else console.log("Write data has been completed");
        });
    });

    rfs.on('end', () => {
        console.log("End Readable");
    });

    wfs.on("close", () => {
        console.log("Close Writable");
    });

// When readable end up to read data...
// End Readable
```

理由は、

`error`または`close`または`finishi`イベントが発行されていないからである。

公式を見てみる：

`fs.createWriteStream()`より

> 「error」または「finish」時に autoClose が true (デフォルトの動作) に設定されている場合、ファイル記述子は自動的に閉じられます。

> デフォルトでは、ストリームは破棄された後に「close」イベントを発行します。この動作を変更するには、emitClose オプションを false に設定します。

`fs.Writable::Event:'close'`より

> 「close」イベントは、ストリームとその基になるリソース (ファイル記述子など) が閉じられたときに発行されます。このイベントは、これ以上イベントが発行されず、それ以上の計算が行われないことを示します。

つまり`close`イベントは`Writable`ストリームの仕事が完全に終わったときに発行されるべき。

`fs.Writable::Event'finish'`より

> 「finish」イベントは、stream.end() メソッドが呼び出された後に発行され、すべてのデータが基盤となるシステムにフラッシュされます。

ということは、実は`Writable`は`autoClose: true`にしても勝手にとるのではなくて、閉じるためにイベントを発行させなくてはならないのである。

`autoClose: true`は`error`か`finishi`イベントが発行されたら書き込みストリームを閉じるよという意味で、

`emitClose: true`は「ストリームが破棄されたときに`close`イベントを発行するよ」という意味で、

「書き込みストリームの破棄」は自動で行うよとは一言も言っていないのである。

なので**書き込みストリームの破棄は明示的に実行しなくてはならない**

其の方法がこちら：[`writable.end()`と`writable.destroy()`の使い分け](<#`writable.end()`と`writable.destroy()`の使い分け>)

## `drain`と`writable.write()`の仕組み

重要：

- `writable.write()`が`false`を返したら`drain`イベントが発行されるまでデータ書き込みは停止せよ

参考：

- https://nodejs.org/dist/latest-v16.x/docs/api/stream.html#event-drain

> `stream.write()`呼出が`false`を返したら、**ストリームへのデータの書き込みを再開するのが適切なときに「drain」イベントが発行されます。**

- https://nodejs.org/dist/latest-v16.x/docs/api/stream.html#buffering

> (読み取りストリームは)ひとた内部バッファに保存しているデータ量が`highWaterMark`で指定した閾値に到達したら、読取ストリームはデータが消費されるまで一時的にデータを読み取るのを停止する。

> (書込みストリームは)`writable.write()`が継続的に呼び出されるとデータは`Writable`ストリームへバッファされる。

> (書込みストリームの)内部バッファが`highWaterMark`で指定した閾値へ保存データ量が到達まで、`writable.write()`は true を返し、到達したら`false`を返す。

- https://nodejs.org/dist/latest-v16.x/docs/api/stream.html#writablewritechunk-encoding-callback

> (`writable.write()`は)内部バッファが、チャンクを受け入れた後にストリームが作成されたときに構成された `highWaterMark`よりも小さい場合、戻り値は`true`です。

> **`false`が返された場合、`drain`イベントが発行されるまで、ストリームへのデータ書き込みのさらなる試みは停止する必要があります。**

> ひとたびバッファへ保存された chunk がドレインしたら（内部バッファに保存されたデータが書き込み可能になったら）`drain`イベントが発行されます。

> **`writable.write()`が`false`を返したら、`drain`イベントが発行されるまで chunk の書込みを停止するのが推奨されます。**

...ということで、まとめると...

- `highWaterMark`は内部バッファの「満タン」を（形式的に）定義してストリームを制御する

- `writable.write()`が`false`を返したら`drain`イベントが発行されるまでデータ書き込みは即座に停止しなくてはならない

- `drain`イベントが発行されてから chunk の書き込みを再開せよ

わかった。よくわかった。

では`wrtiable.write()`の戻り値を毎回チェックするとして、`false`が返されたら実際にどう処理すればいいのか？

公式では次を示されている

- https://nodejs.org/dist/latest-v16.x/docs/api/stream.html#writablewritechunk-encoding-callback

> 書き込むデータをオンデマンドで生成またはフェッチできる場合は、ロジックを Readable にカプセル化し、stream.pipe() を使用することをお勧めします。

> ただし、write() の呼び出しが優先される場合は、「drain」イベントを使用してバックプレッシャーを尊重し、メモリの問題を回避することができます。

つまり上でいうところの推奨は`radable.pipe()`を使えである。

```TypeScript
const rfs: fs.Readable = createReadStream(/**/);
const wfs: fs.Writable = createWriteStream(/**/);
rfs.pipe(wfs);
```

なんでかというと、pipe は内部的に`drain`を制御している

参考：

- https://techblog.yahoo.co.jp/advent-calendar-2016/node-stream-highwatermark/

```JavaScript

Readable.prototype.pipe = function(dest) {
  var src = this;
  src.on('data', (chunk) => {
    var ret = dest.write(chunk); // 読み込んだデータを dest に書き込む
    if (false === ret) { // highWaterMark に達していたら
      src.pause(); // 読み込み一時停止
    }
  });

  dest.on('drain', () => { // highWaterMark を下回る
    flow(src); // 読み込み再開
  });
};
```

つまり`pipe`を使えば`drain`イベントに関する制御を丸投げできるのである。

それでも`pipe()`を使えない事情があるとか完全に使用メモリ量を制御したいならば、

`writable.write()`を直接呼出す実装を自分で定義することになる。

参考：

- https://stackoverflow.com/a/45905612/13891684

- https://stackoverflow.com/a/50360972/13891684

課題：

- `writable.write()`が false を返したときにどうやって一時停止すればいいのか？
- `drain`イベントが発行されたらどうやって一時停止を解除すればいいのか？

公式や参考のサイトでは`process.nextTick()`, `flow()`みたいな関数を使われていた

`writable.once('drain', WRITEMETHOD)`みたいなイベントハンドラもあるみたい

しかし、

[Readable のモード切替方法](#Readableのモード切替方法)で学習したように

**`readable.pause()`と`readable.resume()`を使えばいい。**

```TypeScript
// stream/fs_writable_write.ts
// 19kbのファイルのコピーを作るプログラム

const rfs: fs.Readable = fs.createReadStream(/* params */);
const wfs: fs.Writable = fs.createWriteStream(/* params */);

let draining: boolean = true;

rfs.on('data', (chunk) => {
    console.log(`Readable read ${chunk.length} byte of data`)
    draining = wfs.write(chunk, (e: Error | null | undefined) => {
        if(e) {
            // ここでエラーが起こったら`error`イベント前にこの
            // コールバックが実行される
            console.error(e.message);
        }
    });
    // chunkを書き込んだ後のwriteの戻り値がfalseなら
    // 読取ストリームはすぐに停止する
    if(!draining) {
        console.log('Paused Readable because of reaching highWaterMark');
        rfs.pause();
    }
});

// `drain`イベントは書込みが再開できるときに発行される
wfs.on('drain', () => {
    console.log('Drained and resume Readable again.');
    // drainイベントが発行されたら読取ストリームの読取を再開する
    draining = true;
    rfs.resume();
});

wfs.on('end', () => {
    console.log('End Readable');
});

wfs.on('finish', () => {
    console.log('Finished');
});

wfs.on('close', () => {
    console.log('Writable closed');
});

/***
 * `error`イベントが発行されたらストリームは閉じられる
 *
 * `error`以降`close`イベント以外発行されてはならない
 * */
wfs.on('error', (e: Error) => {
    if(!wfs.destroyed) wfs.destroy(e);
});

rfs.on('end', () => {
    console.log('there is no more data to be consumed from Readable');
    // NOTE: writableの破棄は明示的に
    wfs.end();
})

rfs.on('error', (e: Error) => {
    if(!rfs.destroyed) rfs.destroy(e);
});

```

結果：

```bash

# 開始
 Drained and resume Readable again.
#  highWaterMarkで指定した通りのデータサイズを読み取った
 Readable read 1024 byte of data
#  highWaterMarkが閾値ぴったりになったのでwritable.writeがfalseを返しReadableを一時停止した
 Paused Readable because of reaching highWaterMark
#  drainイベント
#  書き込めるようになったので書き込み再開
 Drained and resume Readable again.

 Readable read 1024 byte of data
 Paused Readable because of reaching highWaterMark
 Drained and resume Readable again.

 Readable read 1024 byte of data
 Paused Readable because of reaching highWaterMark
 Drained and resume Readable again.
#  以下しばらく同じ流れが続いたあと...

#  Readableにendイベント
#  Readableが破棄される
#  実装のWritable.end()が実行される
 there is no more data to be consumed from Readable
#  writable.end()ではfinishiイベントが発行される
#  書込みストリームの内部バッファがフラッシュされて...
 Finished
#  そのまま書込みストリームは閉じられた
 Writable closed
#  正常終了
 [nodemon] clean exit - waiting for changes before restart
```

もちろん上記の通りの実装で stream を使うなら pipe()を使った方がわかりやすく話が早い。

しかし pipe()を使わない場合にベースの実装を実現することができた。

## `writable.end()`と`writable.destroy()`の使い分け

1. `Writable`を破棄する前に内部バッファをフラッシュしたいときは`writable.end()`を呼び出そう。

なぜならば、`writable.end()`ならば`close`イベントの前に`finish`イベントを発行させることができるからである。

公式より：

- `close`イベントが発行されるとこれ以降の書き込みは受け付けなくなる。

- `finish`イベントが発行されると内部バッファのデータがすべて書き込まれる（フラッシュされる）。

- もしも`Writable`が`autoClose: true`で作成されてあったら、`finish`イベント時に`Writable`が破棄される(`fs.createWritableStream()`より)

つまり、

書き込みストリームを破棄する前に、内部バッファにあるデータを書き込み先に書き込んじゃいたいときには`finish`イベントを呼び出さない限り書き込む方法は失われるのである。

`finish`イベントを呼び出す前に`close`イベントが発行されると内部バッファにデータがあってもこれ以上の書き込みは受け付けなくなっているので、そのデータは行き先を失ってガベージコレクションに追加される。

たとえば、

内部バッファに残ったデータ量が大きいときに`close`イベントを発行してしまうと大きなメモリリークになりかねない。

イベント発行タイミングと内部バッファがちょうどクリアされているタイミングが一致するのはあんまり期待できない。

なので、

普段使うときは`writable.end()`,`finsih`,`close`のながれで`Writable`を閉じていくのが推奨の流れといえるでしょう。

2. ただちに`Writable`を破棄しなくてはならないなら`writable.destory`を呼び出そう

なぜならば、`writable.destroy()`は呼び出されると直ちに`close`イベントを発行させるからである。

(`Writable`コンストラクタに`emitClose:false`を渡してなければ)

先にも書いた通り、`close`が発行されるとこれ以上の書き込みは受け付けなくなるので内部バッファに玉ってあったデータはストリーム先に書き込まれない。

## エラーハンドリング

## pipe

公式の推奨の使い方である`readable.pipe(writable)`を実装して正しい使い方を学ぶ。

## 実践：改善版

学習した正しい使い方を反映して再度画像ファイルをコピーしてみる。

ちょっとおさらい

- `writable.write()`の戻り値をチェックして`false`が帰っていたら次の`drain`イベントまで`Readable`のデータ取得を停止すること

- `Readable`のモードは、`flowing`モード中に`readable.pause()`呼出しで`paused`モードに、`paused`モード中に`readable.resume()`呼び出しで`flowing`モードに変更される

- `writable`は明示的に閉じる処理を実装すること

以下のプログラムは次の動作をするはずである：

`highWaterMark`が 1024 で指定しているので

毎回読み取りストリームが`highWaterMark`マックスまでデータを読み取って

その都度`writable.write()`が内部バッファへ 1024byte 書き込み

こちらも`highWaterMark`へ到達するので`false`を返すはずである。

そしたら`readable.pause()`で一時停止して`drain`イベントまで読み取りを停止する。

読み取りストリームが`end`イベントを発行したら、

書き込みストリームは明示的に`writable.end()`するので`finish`イベントが発行され、

その時点の書き込みストリームの内部バッファがフラッシュされる（ファイルへ書き込まれる）

そうしたのち書き込みストリームは閉じられて、以降書き込みは許されなくなる。

```TypeScript

import * as fs from 'node:fs';
import * as path from 'node:path';
import * as crypto from 'node:crypto';
import { readFileSync } from 'node:fs';

const outPath = path.join(__dirname, "out");
const inPath = path.join(__dirname, "in");

// -- HELPERS -------------------

// ランダムな文字列を生成するやつ
//
// https://qiita.com/fukasawah/items/db7f0405564bdc37820e#node%E3%81%AE%E3%81%BF
const randomString = (upto: number): string => {
    const S="abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";
    return Array.from(crypto.randomFillSync(new Uint8Array(upto))).map((n)=>S[n%S.length]).join('');
}


// 指定のパスにディレクトリは存在するのか確認する関数
/*
https://stackoverflow.com/questions/15630770/node-js-check-if-path-is-file-or-directory

https://nodejs.org/dist/latest-v16.x/docs/api/fs.html#class-fsstats

*/
const isDirExist = (path: string): boolean => {
    return fs.lstatSync(path).isDirectory() && fs.existsSync(path);
}

const createRfs = (): fs.ReadStream => {
    if(!isDirExist(inPath)) throw new Error(`The path: ${inPath} does not exist.`);

    return fs.createReadStream(
        path.join(inPath, "cat.png"),
        {
            encoding: 'binary',     /* default: 'utf8' */
            autoClose: true,
            emitClose: true,
            highWaterMark: 1024     /* default: 64 * 1024 */
        }
    );
}


const createWfs = (): fs.WriteStream => {
    if(!isDirExist(outPath)) throw new Error(`The path: ${outPath} does not exist.`);

    return fs.createWriteStream(
        path.join(outPath, "cat" + randomString(4) + ".png"),
        {
            encoding: 'binary',     /* default: 'utf8' */
            autoClose: true,
            emitClose: true,
            highWaterMark: 1024     /* default: 64 * 1024 */
    });
}



(function() {
    const rfs: fs.ReadStream = createRfs();
    const wfs: fs.WriteStream = createWfs();

    let draining: boolean = true;

    rfs.on('data', (chunk) => {
        console.log(`Readable read ${chunk.length} byte of data`)
        draining = wfs.write(chunk, (e: Error | null | undefined) => {
            if(e) {
                // ここでエラーが起こったら`error`イベント前にこの
                // コールバックが実行される
                // で、`error`イベントが発行される
                console.error(e.message);
            }
        });
        // chunkを書き込んだ後のwriteの戻り値がfalseなら
        // 読取ストリームはすぐに停止する
        if(!draining) {
            console.log('Paused Readable because of reaching highWaterMark');
            rfs.pause();
        }
    });

    // `drain`イベントは書込みが再開できるときに発行される
    wfs.on('drain', () => {
        console.log('Drained and resume Readable again.');
        // drainイベントが発行されたら読取ストリームの読取を再開する
        draining = true;
        rfs.resume();
    });

    wfs.on('end', () => {
        console.log('End Writable');
    });

    wfs.on('finish', () => {
        console.log('Finished');
    });

    wfs.on('close', () => {
        console.log('Writable closed');
    });

    /***
     * `error`イベントが発行されたらストリームは閉じられる
     *
     * `error`以降`close`イベント以外発行されてはならない
     * */
    wfs.on('error', (e: Error) => {
        if(!wfs.destroyed) wfs.destroy(e);
    });

    rfs.on('end', () => {
        console.log('there is no more data to be consumed from Readable');
        wfs.end();
    })

    rfs.on('error', (e: Error) => {
        if(!rfs.destroyed) rfs.destroy(e);
    });
})();

```

結果

```bash
[start:*run] [nodemon] starting `node ./dist/index.js`
[start:*run] Readable read 1024 byte of data
[start:*run] Paused Readable because of reaching highWaterMark
[start:*run] Drained and resume Readable again.
[start:*run] Readable read 1024 byte of data
[start:*run] Paused Readable because of reaching highWaterMark
[start:*run] Drained and resume Readable again.
[start:*run] Readable read 1024 byte of data
[start:*run] Paused Readable because of reaching highWaterMark
[start:*run] Drained and resume Readable again.
[start:*run] Readable read 1024 byte of data
[start:*run] Paused Readable because of reaching highWaterMark
[start:*run] Drained and resume Readable again.
[start:*run] Readable read 1024 byte of data
[start:*run] Paused Readable because of reaching highWaterMark
[start:*run] Drained and resume Readable again.
[start:*run] Readable read 1024 byte of data
[start:*run] Paused Readable because of reaching highWaterMark
[start:*run] Drained and resume Readable again.
[start:*run] Readable read 1024 byte of data
[start:*run] Paused Readable because of reaching highWaterMark
[start:*run] Drained and resume Readable again.
[start:*run] Readable read 1024 byte of data
[start:*run] Paused Readable because of reaching highWaterMark
[start:*run] Drained and resume Readable again.
[start:*run] Readable read 1024 byte of data
[start:*run] Paused Readable because of reaching highWaterMark
[start:*run] Drained and resume Readable again.
[start:*run] Readable read 1024 byte of data
[start:*run] Paused Readable because of reaching highWaterMark
[start:*run] Drained and resume Readable again.
[start:*run] Readable read 1024 byte of data
[start:*run] Paused Readable because of reaching highWaterMark
[start:*run] Drained and resume Readable again.
[start:*run] Readable read 1024 byte of data
[start:*run] Paused Readable because of reaching highWaterMark
[start:*run] Drained and resume Readable again.
[start:*run] Readable read 1024 byte of data
[start:*run] Paused Readable because of reaching highWaterMark
[start:*run] Drained and resume Readable again.
[start:*run] Readable read 1024 byte of data
[start:*run] Paused Readable because of reaching highWaterMark
[start:*run] Drained and resume Readable again.
[start:*run] Readable read 905 byte of data
[start:*run] there is no more data to be consumed from Readable
[start:*run] Finished
[start:*run] Writable closed
[start:*run] [nodemon] clean exit - waiting for changes before restart
```

元のファイルサイズは 15kb で、取得したのは 15241byte。

書き込みストリームは閉じられている。

`drain`イベントと読み取りストリームの一時停止と再開が正常に動いている。
