---
title: "Dockerコンテナをエクスポート／インポートする"
date: 2019-11-22
draft: false
tags:
- Docker
categories:
- Docker
---

Dockerレジストリを経由せず、Dockerイメージをファイルとして配布したい。

## コンテナをtarファイル化

`docker export`使う。

```bash
$ docker export <container id> > docker_archive.tar
```

## tarファイルからDockerイメージを作成

`docker import`使う。
```bash
$ cat docker_archive.tar | docker import - hoge:latest
```

## コンテナ起動

```bash
$ docker run -it hoge:latest
```
