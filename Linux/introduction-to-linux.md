# Linux Foundation: Introduction to Linux

参考：

https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/5/html/installation_guide/s1-boot-init-shutdown-process#s2-boot-init-shutdown-loader

## Linux の哲学

Linux 哲学
Linux は、定評のある UNIX オペレーティング システム ファミリーから多くのものを借用しています。Linux は、無料かつオープン ソースの代替として作成されました。当時の UNIX は、PC よりもはるかに高性能なコンピューター向けに設計されており、さらに非常に高価でした。

ファイルは階層的なファイルシステムに保存され、システムの最上位ノードはルートまたは単に「/」です。Linux は、可能な限り、ファイルまたはファイルのようなオブジェクトを介してコンポーネントを利用できるようにします。プロセス、デバイス、およびネットワーク ソケットはすべてファイルのようなオブジェクトで表され、通常のファイルに使用されるのと同じユーティリティを使用して操作できます。Linux は、完全なマルチタスク (つまり、複数の実行スレッドが同時に実行される) のマルチユーザー オペレーティング システムであり、UNIX の世界ではデーモンと呼ばれる組み込みのネットワークおよびサービス プロセスを備えています。

注意: Linux は UNIX にヒントを得ていますが、UNIX ではありません。

Linux がフリーで提供されたおかげでインターネットが普及することができた

Linux は、デーモンと呼ばれるネットワークおよびサービス プロセスが組み込まれた、完全なマルチタスク、マルチユーザー オペレーティング システムです。

## 用語

bootloader: OC を起動するプログラム
Distros: Linux カーネルと組み合わされたプログラムの集合体。前提とする Linux カーネルのバージョンだったり、付随しているプログラムはディストリビューションによって異なったりする
Service: バックグラウンドプロセスとして実行されるプログラム（httpd, nfsd
Filesystem: Linux でファイルを扱うための方法のこと
Command Line: OS 上でコマンドを入力するためのインタフェイス
Shell: コマンドラインの入力を解釈して必要なタスクやコマンドを実行するよう OS に指示するコマンドライン・インタプリタである

## ブートプロセス

```
電源ON
BIOS
MBR or EFI Partition
Boot Loader
Kernel
RAM初期化
/sbin/init (親プロセス)
command shell using getty
GUI
```

#### BIOS

基本入出力システム

コンピュータの電源が ON になると初めに起動するプログラムである。

画面やキーボードなどのハードウェアを初期化し、メイン メモリをテストし、システムを起動するための有効なデバイスを捜索する

次に BIOS は見つけたシステム起動デバイスの、MBR と呼ばれるプログラムをメモリにロードする。

MBR はそのデバイスに記憶されたマシンを起動するためのマシンコード命令が含まれている。

このマシンコード命令にはブートローダが含まれており、RAM へ展開することでブートローダが起動する。

ブートローダを起動すると、BIOS はシステムの制御をブートローダへ移譲する。

BIOS はマザーボード上のメモリチップに保存されている。ブートプロセスのブートローダの起動までを担う

#### MBR EFI パーティション

実は BIOS が起動するデバイスは MBR １種類だけではない

ブートローダは通常ストレージデバイス（SSD や HDD）のブートセクタ（MBR）または EFI パーティションへ保存されている

日付、時刻、周辺機器に関する情報が CMOS 値からロードされる

#### ブートローダの動作

2 つの方式と 2 つの異なる段階

1 段階目：MBR または EFI を展開

MBR 方式:

ハードディスクの最初のセクターに保存されている。

パーティションテーブルを調べる、ブート可能なパーティションを見つける、第２段階のブートローダを検索してそれを RAM へ展開する

EFI 方式:

> UEFI ファームウェアはブートマネージャーデータを読み取り、どの UEFI アプリケーションをどこから起動するか (つまり、どのディスクとパーティションから EFI パーティションを見つけることができるか) を判断します。次に、ファームウェアは、ファームウェアのブートマネージャーのブートエントリで定義されている UEFI アプリケーション (GRUB など) を起動します。この手順は、古い MBR 方式よりも複雑ですが、より多用途です。

２段階目：MBR または EFI に記憶されているブートローダを見つけてメモリに展開すること

ブートローダは`/boot`いかに存在する

ブートローダが起動するとどの OS、どのカーネルを起動するかをスプラッシュスクリーン上に表示して選択させる

（選択されなかった場合、デフォルトの選択を選ぶ）

ブートローダは選択された OS のカーネルを RAM に展開する。

#### RAM の初期化

カーネルを起動し終えたら、ブートローダはルートファイルシステムをマウントするために`Initramfs`というファイルシステムイメージを呼び出す。

ブートローダはカーネルと initranfs イメージをロードすると、ブートプロセスの制御をカーネルへ委譲する。

カーネルの目標はルートファイルシステムをマウントすること。

カーネルはルートファイルシステムをマウントするために必要なサポートしているすべてのデバイスドライバを持っている必要がある。

ドライバすべてをカーネルに組み込むとカーネルは肥大化するし新しいデバイスのサポートはできないなどでメリットばかり。

そのため必要なドライバを必要に応じてロードする仕組みとしてカーネルモジュールがある。

しかしカーネルモジュールをルートファイルシステムに保存しておくことはできない（今呼び出すために必要なのにそこに保存しては永遠に呼び出せないから）

そこでルートファイルシステムの代わりにミニルートと呼ばれるファイルシステムにカーネルモジュールを保存することにした。

`Initramfs`はそのミニルートファイルシステムのことである。

つまり、

カーネルを起動して次にやるべきことはルートファイルシステムをマウントすることである。

ルートファイルシステムをマウントするには必要なデバイスドライバが必要で、そのデバイスドライバをロードするためにカーネルモジュールを呼び出さなくてはならない。

カーネルモジュールの保存場所がミニルートファイルシステムであり、ミニルートファイルシステムとは`Initramfs`なのである。

> initramfs ファイルシステム イメージには、適切なルート ファイルシステムをマウントするために必要なすべてのアクションを実行するプログラムとバイナリ ファイルが含まれています。これには、使用される特定のファイルシステムに必要なカーネル機能の提供、および udev システム (ユーザー デバイス用) を利用して大容量ストレージ コントローラのデバイス ドライバーのロードが含まれます。udev システムは、存在するデバイスを特定し、デバイスが適切に動作するために必要なデバイス ドライバーを見つけてロードする役割を担います。ルート ファイルシステムが見つかったら、エラーがチェックされ、マウントされます。

> マウントプログラムは、ファイル システムが使用可能であることをオペレーティング システムに指示し、ファイル システムの全体的な階層内の特定のポイント (マウント ポイント) に関連付けます。これが正常に実行されると、initramfs が RAM からクリアされ、ルート ファイル システム ( /sbin/init ) 上の init プログラムが実行されます。

> init は最終的な実際のルート ファイルシステムへのマウントとピボットを処理します。大容量ストレージにアクセスする前に特別なハードウェア ドライバーが必要な場合は、それらを initramfs イメージに配置する必要があります。

カーネルモジュールのロード、デバイスドライバのロードが完了したら、ルートファイルシステムがマウントされ（initramfs が RAM からクリアされ）、

ルートファイルシステム上の`/sbin/init`が呼び出される

カーネルは制御を`init`に渡す

プログラム/sbin/init はすべてのサービスとユーザー空間ツールをロードし、 /etc/fstab にリストされているすべてのパーティションをマウントします。

`init`はすべてのプログラムの親プログラムになる。

#### Text Mode login

ブートプロセスの終わり近くに、init はテキストモードのログインプロンプトをいくつか開始する。これらにより、ユーザー名を入力し、パスワードを入力し、最終的にコマンドシェルを取得することができます。

通常、デフォルトのコマンドシェルは bash（GNU Bourne Again Shell）ですが、他にも多くの高度なコマンドシェルがあります。シェルはテキストプロンプトを表示し、コマンドを受け付ける準備ができていることを示します。ユーザーがコマンドを入力して Enter キーを押すと、コマンドが実行され、コマンドが終了すると別のプロンプトが表示されます。

#### Linux kernel

カーネルは Initramfs と（ほぼ）一緒に呼び出される

カーネルはひとたび呼び出されるとただちにハードウェアの構成の初期化、メモリの初期化を実行する

#### /sbin/init and services

## Graphical Interface

#### X Window system

デスクトップ環境は、グラフィックの起動とセッションを管理するセッションマネージャで構成されている

session manager + window manager + set of utilities

しばらく省略

## GUI setup

やはり GUI を入れるつもりがないので省略

## Command line operations

#### Basic Operations

##### shutdown:

シャットダウンの時間をスケジュール設定できる

これは複数ユーザを管理する上で重要である。
ユーザにシャットダウンするまでの猶予を与えることが可能になるからだ

たとえばシャットダウン時間を設定して、いざその時間を迎えるとどうなるか。

> After the delay, shutdown tells init to change to runlevel 0 (halt) or 6 (reboot). (Note that omitting -h or -r will cause the system to go into single-user mode (runlevel 1), which kills most system processes but does not actually halt the system; it still allows the administrator to remain logged in as root.)

`--halt`オプション付きで shutdown すると、設定時間を迎えたときに admin 以外のユーザプロセスはキルされ、admin だけ残る状態になる。なので実際にはシステムはシャットダウンしていない。admin 以外のユーザからはシャットダウンしている(ように見える）

```bash
# 22:00にシャットダウンするよう設定する
$ sudo shutdown --halt 22:00 "シャットダウン時間に他のユーザ向けに表示されるメッセージ"
# default
$ sudo shutdown --poweroff
# 再起動させる
$ sudo shutdown --reboot
```

参考：

https://unix.stackexchange.com/a/8694

##### Locating Applications

アプリケーションのインストール場所は distro やアプリケーションの種類などによってことなるかも

```
/bin
/sbin
/usr/sbin/
/usr/bin/
/usr/local/bin
```

なので実際にどこにあるのか探してくれるコマンド

```bash
# そのアプリケーションはfilesystemのどこに位置しているのか探してくれる
$ which diff
/usr/bin/diff

# whichで見つけられない場合、より広範なシステム中を捜索してくれる
$ whereis diff
```

#### absolute/relative path

-   absolute path: `/`から始まる
-   relative path: `/`から始まらない

ディレクトリとファイルの間に複数のスラッシュ ( / ) を使用できますが、パス名の要素間のスラッシュは 1 つを除いてすべてシステムによって無視されます。////usr//bin は有効ですが、システムでは/usr/bin として認識されます。

#### `tree -d`でファイルシステム・ツリーを俯瞰する

`-d`でディレクトリのみを表示する

#### hardlinks

参考：

https://www.redhat.com/sysadmin/linking-linux-explained

そもそも Linux のファイルシステム上のファイルはすべてハードリンクから始まっている

ユーザが作成するハードリンクはこの場合と少々異なる

```bash
# file-2がfile-1のショートカット
$ ln file-1 file-2

# ファイルの作成
$ echo "hello world" > /home/User/demo/link_test
```

この時点で`link_test`は`hello workd`というデータと単一のハードリンクが生成されたことになる

```bash
# `link_test`のハードリンクを生成した
$ ln link_test /tmp/link_new

$ ls -l link_test /tmp/link_new
-rw-rw-r--. 2 User User 12 Aug 29 14:27 link_test
-rw-rw-r--. 2 User User 12 Aug 29 14:27 /tmp/link_new
```

自分の環境で検証：

```bash
$ ls
note.md
$ ls -l note.md
-rw-r--r--   1 teddy teddy  25763 Jul 25  2023 note.md
$ ln note.md link_note.md
$ ls -l note.md /tmp/link_note.md
-rw-r--r-- 2 teddy teddy 25763 Jul 25  2023 /tmp/link_note.md
-rw-r--r-- 2 teddy teddy 25763 Jul 25  2023 note.md
# 生成したリンクをcatするともとのファイルの内容を表示する

```

ハードリンクを生成する前後で異なるのは

-   リンクカウントが１から 2 へ増えている
-   リンクファイルの詳細な内容はオリジナルと同じになる

オリジナルのファイルが削除されてもデータ自体は消えない。なのでハードリンクはオリジナルが削除されてもオリジナルのコンテンツを参照できる。

コンテンツデータが削除されるのはそのデータがどこからも参照されなくなったら

もしも２つのファイルのうちどちらかが片方のハードリンクであるかどうかを確認したいときは、`ls -i`オプションをつけてみる。

ls の-i オプションは、最初の列に inode 番号を出力します。これは、各ファイル オブジェクトに固有の数値で、ハードリンクとオリジナルは inode 番号が同じになる

```bash
$ ls -li note.md /tmp/link_note.md
109436 -rw-r--r-- 2 teddy teddy 25763 Jul 25  2023 /tmp/link_note.md
109436 -rw-r--r-- 2 teddy teddy 25763 Jul 25  2023 note.md
```

注意

-   ハードリンクは異なるファイルシステムをまたがって存在することはできない。つまり、ハードリンクが可能な対象は同一のファイルシステム上に存在していなくてはならない

-   オリジナルかハードリンクのいずれかを削除すると inode オブジェクトが残る。このときまた同一の名称のファイルを作成するとエラーが発生する可能性があるとのこと

-   レギュラーファイルに対してのみ利用できる。

#### Soft link

aka Symbolic link

参考：

https://www.redhat.com/sysadmin/linking-linux-explained

```bash
$ ln -s note.md /tmp/link_node.md
```

ソフトリンクはファイルシステム上のスペースを占有しない

ハードリンクと異なる部分：

-   レギュラーファイルだけでなくすべてのファイルに対してリンクを生成可能
-   ソフトリンクはレギュラーファイルではなくオリジナルとはことなる inode 番号が与えられる
-   ことなるファイルシステム/ディスクをまたがって参照することが可能
-   ソフトリンクは現在オリジナルが存在しない場合に、「ぶら下がりリンク」になる

注意：

オリジナルファイルが削除されるとソフトリンクは壊れる。この壊れたリンクを「ぶら下がったリンク（dangling soft link）」と呼ぶ。

もしもぶら下がったリンクのオリジナルと同じ名前の新規のファイルを偶然作成した場合、ぶら下がったリンクはその新しいファイルを差し始める

#### pushd popd

頻繁に異なるディレクトリを移動するようなとき、元の場所がどこであったのか記憶する必要なしに戻りたいときに使えるのが`pushd`や`popd`コマンドであるらしい

```bash
$ pwd
/home/User/projects/myProject/
$ cd /tmp/
$ cd /home/User/repos
...
# cdで元の場所/home/User/projects/myProject/に戻るには
# 絶対パスを与えなくてはならない
$ cd /home/User/projects/myProject/

# もしもこの場合にpushdを使うと
$ pwd
/home/User/projects/myProject/
$ pushd /tmp/
# /home/User/projects/myProject/がディレクトリスタックに追加される
$ pushd
# ディレクトリスタックの一覧が表示される

# 今ディレクトリスタックが次の通りの状況であるとする
$ pushd
# すたっくなので一番上が新しくて下に行くごとに古くなっている
/home/User/repos
/tmp/
/home/User/projects/myProject/
$ pwd
/usr/bin

$ popd
/home/User/repos
$ popd
/tmp/
$ popd
/home/User/repos

```

## Working with files

-   tac: cat の反対でつまり閲覧したいファイル内容の末尾行から先頭行に向かって出力する

hoge.txt

```
abc
gihyo
tec
```

という内容のファイルがあったとすると

```bash
$ cat hoge.txt
abc
gihyo
tec
$ tac hoge.txt
tect
gihyo
abc
```

-   less: 長いファイルを１画面に収まるように、収まらない部分はスクロールで表示できるようにしてくれるコマンド

たとえばものすごく長いファイルを cat で出力すると、全部出力するので、画面に収まらずに結果そのファイルの初めのほうはもはや消えている場合がある

less を使うと１画面に収まる範囲を出力しスクロール操作で残りを閲覧可能にしてくれる

# FUCKYOU STACKBLITZ!!!!!!!!!!!!!!!!!!!!!!! stackblitz が編集内容を保存しなかった!!!!

死ね

ただただ死ね

## Installing software

Linux のディストロやほとんどの add-ons はパッケージマネージャシステムによってインストールされる

おもに２つある

debian ベースのものと、RPM という低レベルのパッケージマネージャ

ハイレベル：apt や zypper

低レベル：dpgk、rpm

apt: debian failysystem
zypper: SUSE familysystem
dnf: redhat familysystem

ディストロによってパッケージマネージャが異なるのでつかっているディストロのパッケージマネージャと正しいコマンドを使わんといかん。Ubuntu 使っているのに zypper のコマンドを打っても仕方ない

#### dpkg

Debian 系のディストロの中間レベルのパッケージマネージャ。

```bash
# インストールされているシステムのパッケージとどんなアプリケーション七日の説明など
$ dpkg --list
| Status=Not/Inst/Conf-files/Unpacked/halF-conf/Half-inst/trig-aWait/Trig-pend
|/ Err?=(none)/Reinst-required (Status,Err: uppercase=bad)
||/ Name                           Version                           Architecture Description
+++-==============================-=================================-============-=====================================>
ii  accountsservice                0.6.55-0ubuntu12~20.04.7          amd64        query and manipulate user account inf>
ii  adduser                        3.118ubuntu2                      all          add and remove users and groups
ii  adwaita-icon-theme             3.36.1-2ubuntu0.20.04.2           all          default icon theme of GNOME (small su>
ii  alsa-topology-conf             1.2.2-1                           all          ALSA topology configuration files
ii  alsa-ucm-conf                  1.2.2-1ubuntu0.13                 all          ALSA Use Case Manager configuration f>
ii  apparmor                       2.13.3-7ubuntu5.3                 amd64        user-space parser utility for AppArmor
ii  apport
# パッケージ(今回はbzip2)を指定してその中身が何かを見たいとき
$ dpkg --listfiles bzip2 | less
/.
/bin
/bin/bunzip2
/bin/bzcat
/bin/bzdiff
/bin/bzexe
/bin/bzgrep
/bin/bzip2
/bin/bzip2recover
/bin/bzmore
/usr
/usr/share
/usr/share/doc
/usr/share/doc/bzip2
/usr/share/doc/bzip2/copyright
/usr/share/man
/usr/share/man/man1
/usr/share/man/man1/bzdiff.1.gz
/usr/share/man/man1/bzexe.1.gz
/usr/share/man/man1/bzgrep.1.gz
/usr/share/man/man1/bzip2.1.gz
/usr/share/man/man1/bzmore.1.gz
/bin/bzcmp
/bin/bzegrep
/bin/bzfgrep
/bin/bzless
/usr/share/doc/bzip2/changelog.Debian.gz
/usr/share/man/man1/bunzip2.1.gz
/usr/share/man/man1/bzcat.1.gz
/usr/share/man/man1/bzcmp.1.gz
/usr/share/man/man1/bzegrep.1.gz
/usr/share/man/man1/bzfgrep.1.gz
/usr/share/man/man1/bzip2recover.1.gz
/usr/share/man/man1/bzless.1.gz
# 削除しようとすると、他のパッケージの削除が必要であるといわれる
$ sudo dpkg --remove bzip2
[sudo] password for teddy:
dpkg: dependency problems prevent removal of bzip2:
 ubuntu-minimal depends on bzip2.
 dpkg-dev depends on bzip2.

dpkg: error processing package bzip2 (--remove):
 dependency problems - not removing
Errors were encountered while processing:
 bzip2

```

#### apt, apt-get

https://aws.amazon.com/jp/compare/the-difference-between-apt-and-apt-get/#:~:text=apt%2Dget%20update-,Key%20differences%3A%20apt%20vs.,management%20tools%20for%20user%20convenience.

apt は apt-get より、よりユーザフレンドリなコマンドを提供するがどちらも Debian 系の高レベルパッケージマネージャである

#### dpkg, apt, apt-get

What's the difference?

https://askubuntu.com/a/309121

dpkg は指定されたパッケージをインストールするだけで、依存関係のインストールは行わないし、`packageName.deb`を構成しない。

apt-get はパッケージマネージャシステムで依存関係を管理してくれる

## Chap.9 Finding Linux Documentation

操作方法やコマンドの詳細など知りたいときにどうすればいいのかを分かれ

-   man
-   GUN info
-   command help
-   other doc courses

#### man

online にもあるよ

https://man7.org/linux/man-pages/

```bash
$ man <調べたいコマンド>
# manコマンドの詳細
$ man man
# マニュアルページのトピックをそれぞれ1行で表示する
$ man -f man
#
```

オプション：
