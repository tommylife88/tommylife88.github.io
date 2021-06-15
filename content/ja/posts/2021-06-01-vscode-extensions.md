---
title: "VSCodeでお勧めの拡張機能を共有する"
date: 2021-06-01
draft: false
tags:
- VSCode
categories:
- VSCode
---

チームでVSCodeで開発するときに、「この拡張機能は入れておけ」ってあると思う。  
そんな時、VSCodeでお勧めの拡張機能に出す出さないを`.vscode/extensions.json`で管理できる。  
リポジトリで管理できるので便利。

ただし、拡張機能でも信頼できないものもあるので、チーム以外のリポジトリ（よくわからないもの）の`.vscode/extensions.json`はちゃんと拡張機能の評価を見てからにしましょう。  
この機能は勝手にローカルにインストールするものでは無いのでそこは最終的に自己責任になる。

https://qiita.com/Glavis/items/c3dac07e4bcf5c50db0a

```json
{
  // See https://go.microsoft.com/fwlink/?LinkId=827846 to learn about workspace recommendations.
  // Extension identifier format: ${publisher}.${name}. Example: vscode.csharp

  // List of extensions which should be recommended for users of this workspace.
  "recommendations": [
    "EditorConfig.EditorConfig",
    "dbaeumer.vscode-eslint",
    "esbenp.prettier-vscode",
  ],
  // List of extensions recommended by VS Code that should not be recommended for users of this workspace.
  "unwantedRecommendations": ["ms-vscode.vscode-typescript-tslint-plugin"]
}
```
