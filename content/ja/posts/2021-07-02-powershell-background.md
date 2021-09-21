---
title: "PowerShellで簡単バックグランド実行"
date: 2021-07-02
draft: false
tags:
- PowerShell
categories:
- PowerShell
---

PowerShellのジョブのバックグラウンド実行したい。Start-Jobめんどくさい。Linuxみたく、演算子"&"みたいにできないのか。

インストール済みのPowerShellバージョン。
```powershell
> $PSVersionTable

Name                           Value
----                           -----
PSVersion                      5.1.19041.1023
PSEdition                      Desktop
PSCompatibleVersions           {1.0, 2.0, 3.0, 4.0...}
BuildVersion                   10.0.19041.1023
CLRVersion                     4.0.30319.42000
WSManStackVersion              3.0
PSRemotingProtocolVersion      2.3
SerializationVersion           1.1.0.1
```

どうも、PowerShellバージョン6.0からジョブの実行にBackground演算子`&`が使える様になったみたい。
（[Background演算子 &](https://docs.microsoft.com/ja-jp/powershell/module/microsoft.powershell.core/about/about_operators?view=powershell-7.1#background-operator-)）

https://github.com/PowerShell/PowerShell/releases  
から最新の安定版をインストール。

複数バージョンインストール可能みたい。

バージョンアップしたPowerShellを起動。
```powershell
> $PSVersionTable

Name                           Value
----                           -----
PSVersion                      7.1.3
PSEdition                      Core
GitCommitId                    7.1.3
OS                             Microsoft Windows 10.0.19042
Platform                       Win32NT
PSCompatibleVersions           {1.0, 2.0, 3.0, 4.0…}
PSRemotingProtocolVersion      2.3
SerializationVersion           1.1.0.1
WSManStackVersion              3.0
```
やってみた。
```powershell
> Get-Process -Name pwsh &

Id     Name            PSJobTypeName   State         HasMoreData     Location             Command
--     ----            -------------   -----         -----------     --------             -------
1      Job1            BackgroundJob   Running       True            localhost            Microsoft.PowerShell.Man…
```
これは、
```powershell
> Start-Job -ScriptBlock {Get-Process -Name pwsh}
```
と等価。

めっちゃ楽になった。
