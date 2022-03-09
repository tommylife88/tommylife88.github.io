---
title: 【FVM】Flutter SDKの複数バージョン管理
date: 2022-02-15
draft: false
tags:
- Flutter
- FVM
categories:
- Flutter
---

プロジェクト毎にFlutter SDKのバージョンを切り替えたい。

FVM(Flutter Version Management)  
https://fvm.app/

## インストールした手順

Flutterの要件を満たしている前提。

https://fvm.app/docs/getting_started/installation をみる。  
**インストール済みのFlutterがあれば削除しておくのが安心。**

### MacOS

`Homebrew`パッケージから。

```shell
brew tap leoafarias/fvm
brew install fvm
```

### Windows

スタンドアローンがお手軽。

#### スタンドアローン（推奨）

FVM自体のアップデートは手動になるけど、下記`choco`をインストールするのが嫌ならこちらのほうがお手軽。

バイナリを取得する。  
https://github.com/leoafarias/fvm/releases

適当なディレクトリに展開する（以下はユーザーディレクトリの`tools`としている）。  
`C:\Users\<username>\tools\fvm`

環境変数`PATH`に`C:\Users\<username>\tools\fvm`を追加する。

```powershell
# fvm のインストール確認
fvm doctor
```

#### choco

`choco`インストール  
https://chocolatey.org/install  
※fvmが`winget`で取得できればなあ…仕方ない…

```powershell
# choco のインストール確認
choco -v
# choco で fvm インストール
choco install fvm
# fvm のインストール確認
fvm doctor
```

ここまでが`fvm`のインストール。

## 使い方

```shell
# インストール可能なバージョンの一覧表示
fvm releases

# 複数バージョンのインストール
fvm install 2.8.1
fvm install 2.10.1

# インストール済みのバージョンを表示
fvm list
```

## プロジェクトの設定

■Flutterプロジェクトを作成済み場合
```shell
cd <project dir>
fvm use 2.10.1
# fvm use stable #channelの指定もOK
```

■Flutterプロジェクトが未作成の場合
```shell
mkdir fvm_test
cd fvm_test
fvm use 2.10.1 --force
# fvm use stable #channelの指定もOK
fvm flutter create .
```

自動生成される`.fvm\fvm_config.json`をコミットすることで、チームでバージョンを統一できる。

VSCodeや`.gitignore`の設定はこちら。  
https://fvm.app/docs/getting_started/configuration


最後に、`doctor`で正しくSDKを認識しているか確認しておく。
```shell
fvm doctor
fvm flutter doctor
```

## TIPS

### エイリアス

FVM管理のFlutter SDKのコマンドを実行する際に`fvm flutter XXX`や、`fvm dart XXX`のように`fvm`のコマンドが必要になる。  
これはエイリアスで解決できる。  

### global化

プロジェクトとは関係の無いところでFVMで取得したSDKを使用したい場合、`global`コマンドが使える。  
`global`で指定したバージョンが`fvm`無しに実行できる。
```shell
# global には stable チャネルを指定
fvm global stable
# fvm 無しで global に指定したSDKが参照される
flutter doctor
```

### flavor

プロジェクトの中でも複数のSDKを切り替えることができる。  
https://fvm.app/docs/guides/project_flavors

```json:.fvm\fvm_config.json
{
  "flutterSdkVersion": "stable",
  "flavors": {
    "beta": "beta",
    "prod": "2.10.1"
  }
}
```

`beta`として`beta`バージョンを、`prod`として`2.10.1`を指定した例。  
`fvm flavor`で切り替える。

```shell
> fvm flavor
Project flavors configured for "fvm_test":

[1] beta
[2] prod

Select an environment:1 #1を入力
Switching to [beta] flavor, which uses [beta] Flutter sdk.
Now using [beta] flavor. Flutter version [beta].
```