---
title: "パフォーマンスモニタ カウンターを手動で復元する方法"
date: 2021-07-01
draft: false
tags:
- Windows
- パフォーマンスカウンター
---

Windowsのパフォーマンスモニタツールの一部カウンターが選択できなくなった。

[サーバー 2008 64 ビットまたは Windows Server 2008 R2 システムのパフォーマンス カウンター Windows手動で再構築する](https://docs.microsoft.com/ja-jp/troubleshoot/windows-server/performance/manually-rebuild-performance-counters)には、
データベースが破損し再構築が必要になる場合がある、らしい。

そこで、パフォーマンスモニタツールのカウンターを手動で復元する。  
以下のコマンドを実行すれば、ライブラリのDBを再構築して修復されるらしい。

*以下の実行は、自己責任でお願いします。*
```
cd C:\Windows\System32
lodctr /R
cd C:\Windows\SysWOW64
lodctr /R
```
