# Linux Foundation: Introduction to Linux course

## 走り書き

昨日のおさらい

how computer boot up:

1. bios
2. boot loader does boot up process which has two phase
3. phase 1. MBF
4. before kernel

mini-root filesystem を ROM から呼び出してカーネルモジュールを mrfs から呼出て、

デバイスドライバをロードする

デバイスドライバをロードしたら kernel を呼び出してルートファイルシステムをマウントする

## おさらい部分

どうやってコンピュータは起動しているのか

電源 ON
BIOS
MBR or EFI
Boot loader
Kernel
Initial RAM Disk and initramfs
/sbin/init - grand parent of all process
command shell using getty

電源が ON にされたらまず BIOS が起動する/BIOS はマザーボード上の ROM に保存されている/BIOS はハードウェアの初期化、メインメモリやシステムのテストを行う/システム起動ができるデバイスの捜索を行う/起動できるデバイスとは MBR や EFI のことである/BIOS はそのデバイスを見つけるとそれをメモリに展開する

MBS や EFI はブートローダと呼ばれるマシンを起動するためのマシンコード命令やパーティションテーブルを記憶している/BIOS がそれら MBR をメモリに展開することはブートローダを見つけてメモリにロードするための手順である/つまりブートローディングは 2 段階あって、初めにブートローダを見つけるために MBR をメモリに展開してからブートローダを展開するまでの段階、そしてブートローダが実際にシステム（カーネル）を起動する段階である/2 段階目のブートローダは`/boot`以下に保存されている/ブートローダはどの kernel と OS を起動するのか選択する/ブートローダは選択した OS と kernel をメモリに展開する/それが済んだらシステムとハードウェアのチェックを行う

kernel を起動したら、ルートファイルシステムをマウントすることが目標となる/ルートファイルシステムをマウントするにはあらゆるデバイスドライバをロードする必要がある/このロードには kernel module が必要になる/kernel moudle はルートファイルシステムではなく、身にルートファイルシステムに保存されている/この身にルートファイルのフォーマットに一つが initramfs である/ミニルートファイルシステムにはルートファイルシステムをマウントするために必要なカーネルモジュールやスクリプトが保存されている/身にルートの展開、kernel module やスクリプトの実行によってルートファイルシステム（/sbin/init）が実行（マウント）される

ルートファイルシステムがマウントされると、

## Linux basics and system startup

#### The linux kernel

boot loader は kernel と filesystem を RAM に展開する

ひとたび kernel が RAM に展開されると、kernel はシステムに接続されたハードウェアやメモリの構成を初期化する

ルートファイルシステムも RAM に展開する

それ等が完了すると、kernel は/sbin/init を呼び出す。

/sbin/init は kernel が直接呼び出すプログラムで init はすべてのプロセスの親プロセスになる

init はシステムの制御と安全にシステムをシャットダウンすることである。

そして、init は kernel なしでプログラムの制御を行う

init は kernel が直接呼び出すプログラムで、以降システムの制御の責務を負う

他の起動代替手段：

    歴史：SysVinit -> Systemd

Systemd の機能

Systemd の起動時間は初期の init よりも速かった

速くできた理由は積極的に起動段階を並列化させたことにある

#### Linux filesystem basics

書籍やメディア媒体を複数のセクションに分別して整理することはファイルシステムのコンセプトに似ている

partitions and filesystems

パーティションとはストレージメディアの物理的な割り当て領域である

ハードディスクでは、割り当て領域は連続した領域を割り当てられていたが、今日のストレージはより複雑になっている

filesystem とはファイルのアクセスと保存するためのメソッドである

filesystem hierarchy(FHS):

FHS とは Linux 財団の採用する標準的な filesystem の階層構造のことである

path が`/`で始まるファイル構造

##### Distro を選択する際のヒント

Questions to Ask When Choosing a Distribution
Some questions worth thinking about before deciding on a distribution include:

What is the main function of the system (server or desktop)?
What types of packages are important to the organization? For example, web server, word processing, etc.
How much storage space is required, and how much is available? For example, when installing Linux on an embedded device, space is usually constrained.
How often are packages updated?
How long is the support cycle for each release? For example, LTS releases have long-term support.
Do you need kernel customization from the vendor or a third party?
What hardware are you running on? For example, it might be X86, RISC-V, ARM, PPC, etc.
Do you need long-term stability? Or can you accept (or need) a more volatile cutting-edge system running the latest software versions?
