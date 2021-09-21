---
title: "Dockerコンテナのログのクリア方法とローテーション設定"
date: 2019-11-20
draft: false
tags:
- Docker
categories:
- Docker
---

`docker`で起動しっぱなしのコンテナのログがディスクを圧迫していたので、ログを消去する方法と、ついでにログのローテーション設定を行った。  
`docker-compose`でも同じ。

## ログのクリア方法

まずは手動でログをクリアする方法から。

### デフォルトのロギングドライバは「json-file」

```bash
$ docker info | grep Logging
Logging Driver: json-file
```

json-fileはコンテナの標準出力と標準エラーを`/var/lib/docker/containers/[container-id]/[container-id]-json.log`（環境に依存するかも）に出力するため起動しっぱなしだとディスクを圧迫する。

### ログを手動で消去する

{{< alert theme="info" >}}
名前は適宜置き換えて。
{{< /alert >}}

まず起動しているコンテナの一覧を表示する。

```bash
$ docker-compose ps
       Name                       Command               State           Ports
---------------------------------------------------------------------------------------
XXXX_app_1             /docker-entrypoint.sh dock ...   Up      0.0.0.0:3000->3000/tcp
…
```

ログを吐いてみる。

```
$ docker logs XXXX_app_1
#大量のログが・・
```

コンテナのログの保存場所を調べる。

```bash
$ docker inspect XXXX_app_1 | grep -i log
"LogPath": "/var/lib/docker/containers/bef1e7aa9c237a34a9ea08dfd5253c3621915bed2cfd65f7985580998607beb1/bef1e7aa9c237a34a9ea08dfd5253c3621915bed2cfd65f7985580998607beb1-json.log",
"LogConfig": {
```

（消去して問題ないかを確認した上で）ログを消去する。
root権限が必要かも。

```bash
$ truncate -s 0 /var/lib/docker/containers/bef1e7aa9c237a34a9ea08dfd5253c3621915bed2cfd65f7985580998607beb1/bef1e7aa9c237a34a9ea08dfd5253c3621915bed2cfd65f7985580998607beb1-json.log
```

というか、`docker logs clean`てきなコマンドはないのかね。

## ログのローテート設定

次にローテーションさせる。

### グローバル設定で行う場合

dockerデーモンの設定ファイル`/etc/docker/daemon.json`（デフォルトの場所）にロギングドライバのオプションを書く。ファイルが無い場合は新規作成する。

http://docs.docker.jp/engine/reference/commandline/dockerd.html#daemon-configuration-file

```bash
$ cat /etc/docker/daemon.json
{
   "log-driver": "json-file",
   "log-opts": {"max-size": "10m", "max-file": "3"}
}
```

dockerデーモンを再起動する。

```bash
$ docker-compose stop
$ systemctl restart docker
$ docker-compose start
```

設定は新しく作成したコンテナから適用される。 （構築済みのコンテナには反映されません）

### docker run オプションで行う場合

`docker run`コマンドのオプションとして指定できる。

http://docs.docker.jp/engine/reference/logging/overview.html#json

```bash
--log-opt max-size=[0-9+][k|m|g]
--log-opt max-file=[0-9]
```

```bash
$ docker run -d --log-opt max-size=1k --log-opt max-file=10
$ docker start --log-opt max-size=100m --log-opt max-file=10
```

###  docker-compose.yml で設定する場合

`docker-compose`では

```yml:docker-compose.yml
  mongo:
   image: mongo:3.4
   volumes:
     - mongo_configdb:/data/configdb
     - mongo_db:/data/db
# add ↓↓↓
   logging:
     driver: "json-file" # defaults if not specified
     options:
       max-size: "10m"
       max-file: "3"
# add ↑↑↑
```
と書くと設定される。

構築済みのコンテナの場合は、`docker-compose up`でコンテナの再構築が必要。
