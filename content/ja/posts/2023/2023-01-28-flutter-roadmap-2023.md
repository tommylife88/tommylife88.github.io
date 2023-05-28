---
title: 【Flutter】2023年のロードマップ
date: 2023-01-28
draft: false
tags:
- flutter
- Dart
categories:
- flutter
---

[GitHubのWiki-Roadmap](https://github.com/flutter/flutter/wiki/Roadmap)が更新されていました。

## 技術的な負債とチームの速度

プロジェクトの全体的な速度を上げる。

技術的負債の削減を行い、新しい人々がチームに参加し、生産性を上げるために、プロセスを改善する。

問題のバックログに時間を費やし、時代遅れまたは実行不可能な問題を修正し、残りの問題に優先順位を付ける予定。

## パフォーマンス

パフォーマンスの改善は最優先事項である。

シェーダーコンパイラジャンクの問題を、まずは iOS で改善し、次に Android に取り組む。

Web では `Wasm` をサポートする。カスタムシェーダーのパフォーマンスを改善する。

## 品質

アクセシビリティはFlutterアプリケーションにとって重要であり、すべてのプラットフォームでのアクセシビリティサポートの品質を向上させる。  
同様に、ドキュメントを改善し続けることが重要。

また、各プラットフォーム、特に動きの速い Android と iOS で最新の機能に対応し続けていく。  
（例えば iOS の `Cupertino` ウィジェット、 Android カメラプラグインの `CameraX API` への移行）

## セキュリティ

`SLSA-4` を継続することを視野に、今年、主要なリポジトリの `SLSA-3` に到達することを目標に、 SLSA コンプライアンスに引き続き取り組む。

## 新機能

* [Custom asset transformers](https://github.com/flutter/flutter/issues/101077)
* [Efficient 2D scrolling widgets](https://github.com/orgs/flutter/projects/32)
* [Multiple windows](https://github.com/flutter/flutter/issues/30701)
* [Drag and drop](https://github.com/flutter/flutter/issues/30719)
* Wireless debugging on iOS
* [Custom "flutter create" templates](https://github.com/flutter/flutter/issues/77104)

## 実装予定無し

* Web 上でのホットリロード
* ウェアラブル端末対応
* Apple CarPlay 対応
* Android Auto 対応
* SEO サポート
* homebrew によるインストール
