---
title: "「Linuxに触れる」パッケージのソースを落としてビルド、デバッグする方法"
date: 2018-10-11
draft: false
tags:
- Linux
- gdb
categories:
- Linux
---

こんな人に読んでほしい。

* Linux開発に触れてみたい
* Linux開発始めたいけど、いまいち何から手をつけたらいいか分からない

Linuxのコマンドをデバッグしてみよう。  
コマンドのパッケージを特定してソースコードを落としてビルドしてデバックするところまでを実践的に触れることができると思う。

## lsコマンドをデバッグしてみる

本記事ではLinuxでよく使うコマンド`ls`をソースコートを落としてビルド、デバッグまでやってみる。

## 環境

```bash
$ cat /etc/lsb-release
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=18.04
DISTRIB_CODENAME=bionic
DISTRIB_DESCRIPTION="Ubuntu 18.04.1 LTS"
```

## コマンドのパッケージを特定

まずは、`ls`コマンドのパッケージを特定します。

```bash
$ which ls
/bin/ls
$ dpkg -S /bin/ls
coreutils: /bin/ls
```

`ls`は `oreutils`っていうパッケージに含まれていることが分かりました。

## パッケージのソースをダウンロード

ubuntu の公式パッケージであれば`apt source`でソースコードをダウンロードできます。  
リポジトリからソースをダウンロードするため、`sources.list`の`deb-src`を有効にします。

```bash
$ sudo su -c "grep '^deb ' /etc/apt/sources.list | \
sed 's/^deb/deb-src/g' > /etc/apt/sources.list.d/deb-src.list"
```

リポジトリを更新しパッケージをダウンロードします。

```bash
$ sudo apt update
$ apt source coreutils
```

## lsをビルドする環境を整える

`apt build-dep`でソースパッケージをビルドするのに必要なソフトウェアをインストールしてくれます。便利。

```bash
$ sudo apt install build-essential
$ sudo apt build-dep coreutils
```

## ビルドする

デバッグしたいので、デバッグ情報付加、最適化しないようにします。

```bash
$ cd coreutils-8.28
$ ./configure CFLAGS="-g3 -O0"
$ make
```

srcディレクトリ配下に実行ファイルが生成されます。

## デバッグする

GNUデバッガ`gdb`を使います。$ gdb (実行ファイル名)で起動します。

```bash
$ cd src
$ gdb ./ls
(gdb) b main #main()関数にブレークポイントを貼る
Breakpoint 1 at 0x4fc5: file src/ls.c, line 1445.
(gdb) r #実行する
Starting program: /home/coreutils-8.28/src/ls
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".

Breakpoint 1, main (argc=1, argv=0x7fffffffe388) at src/ls.c:1445
1445    {
(gdb) n #ステップ実行する
1451      set_program_name (argv[0]);
(gdb)
1452      setlocale (LC_ALL, "");
(gdb)
1453      bindtextdomain (PACKAGE, LOCALEDIR);
```

main()関数にブレークポイントを貼ってステップ実行している（デバッグしている）例です。gdbの使い方に関してはマニュアルを参照しましょう。

どうでしょう？たったこれだけですが、Linux開発の基本的なところである、「パッケージのソースを落としてビルド環境整えて、ビルド、デバッグする」という部分に触れることができましたね。
