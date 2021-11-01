---
title: M1 MacにNode.jsとYarnをインストール
date: 2021-10-30
draft: false
tags:
- Mac
- MacM1
- Node
- Nodenv
- Yarn
categories:
- Node
---

M1 Macに`Node.js`インストール。

ネットにはいろいろやり方があるけど自分用に整理した。  
（MacはAppleシリコンやらで情報が錯綜しがち、数ヶ月の情報も古かったり信用できなかったり・・）

## 環境

Mac
```zsh
% sw_vers
ProductName:    macOS
ProductVersion: 12.0.1
BuildVersion:   21A559
```

Homebrew
```zsh
% brew --version
Homebrew 3.3.1-47-gdae9a34
Homebrew/homebrew-core (git revision 229487f07e1; last commit 2021-10-30)
Homebrew/homebrew-cask (git revision 66bab33b26; last commit 2021-10-30)
```

## Node.js

`rbenv`に慣れているので、`Node.js`のバージョン管理に`nodenv`を使う。  
https://github.com/nodenv/nodenv

公式の手順そのまま。

### インストール

```zsh
% brew install nodenv
% echo 'eval "$(nodenv init -)"' >> ~/.zshrc
```

### バージョン確認

```zsh
% nodenv -v
nodenv 1.4.0
```

### Node.js本体をインストール

とりあえず最新バージョン

```zsh
% nodenv install 16.13.0
% nodenv global 16.13.0
% node -v
v16.13.0
% npm -v
8.1.0
```

### インストールチェック

本当はインストール後にやるものだけど。

```zsh
% curl -fsSL https://github.com/nodenv/nodenv-installer/raw/master/bin/nodenv-doctor | bash
Checking for `nodenv' in PATH: /opt/homebrew/bin/nodenv
Checking for nodenv shims in PATH: OK
Checking `nodenv install' support: /opt/homebrew/bin/nodenv-install (node-build 4.9.61)
Counting installed Node versions: 1 versions
Auditing installed plugins: OK
```

## Yarn

Homebrewからインストールした（意図的にnodeからインストールしていない）。

### インストール

`--ignore-dependencies`を付与して、nodeの依存関係を無視している。Warningでたけど気にしない。。。

```zsh
% brew install yarn --ignore-dependencies
% yarn --version
1.22.17
```