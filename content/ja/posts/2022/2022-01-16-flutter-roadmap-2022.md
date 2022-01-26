---
title: 【Flutter】2022年のロードマップ
date: 2022-01-16
draft: false
tags:
- flutter
- Dart
categories:
- flutter
---

[GitHubのWiki-Roadmap](https://github.com/flutter/flutter/wiki/Roadmap)が更新されていました。

## フォーカスする領域

### 開発者体験

開発者が愛するSDKにする。

* 一般的な問題を解決するWidgetやPluginの作成
* 既存のAPIの整理
* 頻繁に見られるパターンを簡素化するための新しいAPIの導入
* エラーメッセージの改善
* 開発者ツールとIDEプラグインの改善
* 新しいLintの作成
* フレームワークとエンジンのバグの修正
* APIドキュメントの改善
* より有用なサンプルの作成
* Web上でのホットリロード
* Dart-to-JSシナリオでのスタックトレースを改善

### Desktop

デスクトップサポートを`stable`チャネルにあげる。

Windowsに始まり、Linux、macOSと一つづつプラットフォームをテストし、リリースする予定。この取り組みの重要な部分として、リグレッションテストスイートを拡張して、既存のコードを壊すことなく拡大すること。

### Web

以下の改善、向上に取り組む。

* パフォーマンス
* プラグインの品質
* アクセシビリティ
* アクセシビリティ

また、FlutterアプリをFlutter以外のHTMLに埋め込みやすくする予定。

### フレームワーク、エンジン

Materialライブラリを更新して、[Material 3](https://m3.material.io/)をサポート予定（[issue](https://github.com/flutter/flutter/issues/91605)）。これは主に、Androidに忠実に向上させるためだが、プラットフォームに限定されない。  
`cross-widget text selection`なるWidgetを実装したい。これは、ウェブプラットフォームにより忠実であるという目標を達成するためだが、ウェブに限定されるわけではない。

デスクトップのテキスト編集やiPadOSの手書き文字認識など、全てのプラットフォームでテキスト編集を改善させたい。

デスクトップとWebに対して実装されるOSに沿ったコンテキストメニューやメニューバーをリリースする予定。  
また、ホストOS（特にmacOS）に沿ったメニュー（コンテキストメニューとメニューバー）をリリースする。

プラットフォームに限らず、単一の[Isolate](https://api.dart.dev/stable/2.10.4/dart-isolate/dart-isolate-library.html)から、複数のウィンドウへのレンダリングをサポートすることを試みるつもり。

### Dart

静的メタプログラミングのような大きな機能と、パッケージのインポート構文の改善を含むいくつかの小さな改良を行う。

また、`WasmGC`のタイムリーな標準化に応じて`Wasm`へコンパイルするために、Dartのコンパイルツールチェーンを拡張する予定。

### Jank

{{< notice info "Jank" >}} 
ユーザー体験の低下を引き起こすこと
{{< /notice >}}
2021年にはjankに関する多くのissueを解決してきたが、シェーダーをどう使うか再考する必要があると結論付けた。その結果、グラフィックスのバックエンドを再構築していった。  
2022年には、iOSのFlutterを新しいアーキテクチャに移行し、その経験に基づいて、他のプラットフォームに移植する作業を開始していく予定。また、新しいDisplayListシステムが実現した機能など、パフォーマンスの向上やパフォーマンスの自己観察要素を実装していく。

## 非推奨になること

32ビットiOSのサポートを終了予定としている。

## Infrastructure

サプライチェーンのセキュリティへの投資を拡大し、インフラを[SLSA Level 4](https://slsa.dev/spec/v0.1/)に記載された要件に一致させる予定。

{{< notice info "SLSA(Supply-chain Levels for Software Artifacts)" >}} 
Googleが提案しているソフトウェアサプライチェーン攻撃の脅威拡大に対処する手段
{{< /notice >}}
