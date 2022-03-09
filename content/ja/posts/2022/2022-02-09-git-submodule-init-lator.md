---
title: Git cloneで--recursive忘れて、後からsubmoduleをcloneしたいときの対処
date: 2022-02-09
draft: false
tags:
- Git
categories:
- Git
---

いつも忘れるんだよなあ。

`submodule`ありのリポジトリをcloneする際、
```shell
git clone --recursive <repo url>
```
で`submodule`をcloneできるところを、`--recursive`をつけ忘れてしまう。

## 対処法

cloneしたディレクトリに移動して、
```shell
git submodule update --init --recursive
```
で`submodule`を後からcloneできる。