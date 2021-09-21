---
title: "Docker上のJenkinsからホストのDockerを使う（DooD）"
date: 2020-08-29
draft: false
tags:
- Jenkins
- Docker
categories:
- Docker
---

Jenkinsサーバーのビルド環境をクリーンに保ちたいのでDockerコンテナでCIを回したい。
その環境作成メモ。

っていうか、これやるなら、GitLab CI/CD、GitHub Actionsの方が簡単だけど。（Jesnkinsおじさんとはもうおさらば）

## 要件など

Docker outside of Docker（DooD）したい。

* Jenkins本体はDockerコンテナ上で実行される
* Jenkinsのビルドジョブ時のDocker操作はホストに接続され、ホスト環境にコンテナを作成する（DooD）
* セキュリティの観点から、Dockerを操作するユーザーをroot以外（`jenkins`）で作成する
  * `docker`グループをホスト側のDockerグループIDで作成する
  * `jenkins`ユーザーを`docker`グループに所属させ、ホストのDockerへの操作権限を与える

## ホスト環境

JenkinsをDockerするホストの環境。

```bash
$ uname -srv
Linux 5.8.0-40-generic #45~20.04.1-Ubuntu SMP Fri Jan 15 11:35:04 UTC 2021

$ cat /etc/lsb-release 
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=20.04
DISTRIB_CODENAME=focal
DISTRIB_DESCRIPTION="Ubuntu 20.04.2 LTS"
```

dockerコンテナのイメージ一覧。  
後でJenkins上のビルドジョブから`docker images`して同じ結果が表示されればOK。

```bash
$ docker --version
Docker version 19.03.8, build afacb8b7f0
$ docker images
REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
gcc                      latest              e4aa98dc41ec        5 weeks ago         1.19GB
docker/getting-started   latest              3c156928aeec        10 months ago       24.8MB
```

## 構築してみた

構築した環境はGitHubにあげています。

https://github.com/tommylife88/jenkins-dood

### 簡単な解説

ホストのDockerグループIDをCOMPOSE内で使用するための環境変数を記載した設定ファイル。

```ini:.env
DOCKER_GROUP_ID=1001
```

docker-composeの設定ファイル。  
ポートフォワードや`JAVA_OPTS`は好きなようにカスタマイズして良いが、肝は`volumes`で`/var/run/docker.sock:/var/run/docker.sock`しているところ。このマウントを行うことでJenkinsからホストのDockerに接続できる。

docker-composeの設定まわりは、[公式](https://docs.docker.jp/compose/toc.html)見よう。

```yml:docker-compose.yml
version: '2'
services:
  jenkins:
    container_name: "jenkins-dood"
    build:
      context: ./
      args:
        - DOCKER_GROUP_ID_HOST=${DOCKER_GROUP_ID}
    environment:
      - JAVA_OPTS=-Duser.timezone=Asia/Tokyo -Dfile.encoding=UTF-8 -Dsun.jnu.encoding=UTF-8
    ports:
      - 8080:8080
    volumes:
      - ./jenkins-data:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock # mount docker.sock on host
```

Jenkins DooD環境構築用の`Dockerfile`。特に難しいことはしていないはず。

```dockerfile
FROM jenkins/jenkins:lts

USER root

# add jenkins user
RUN mkdir /home/jenkins && chown jenkins:jenkins /home/jenkins && usermod -d /home/jenkins jenkins

# add docker group
ARG DOCKER_GROUP_ID_HOST
RUN groupadd -g ${DOCKER_GROUP_ID_HOST} docker && usermod -aG docker jenkins

# install docker, docker-compose
ENV DOCKER_VERSION 20.10.0
RUN curl -fL -o docker.tgz "https://download.docker.com/linux/static/test/x86_64/docker-$DOCKER_VERSION.tgz" && \
    tar --strip-component=1 -xvaf docker.tgz -C /usr/bin

ENV DOCKER_COMPOSE_VERSION 1.28.4
RUN curl -L https://github.com/docker/compose/releases/download/${DOCKER_COMPOSE_VERSION}/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose && chmod +x /usr/local/bin/docker-compose

USER jenkins
```

## 動作確認

Jenkinsのビルドジョブを作成、シェルスクリプトで`docker images`してみた結果。

最後の`docker images`でホストのDocker環境に接続されている様子が分かる。

```bash
Running as SYSTEM
Building in workspace /var/jenkins_home/workspace/dood
[dood] $ /bin/sh -xe /tmp/jenkins3072304799133685752.sh
+ docker --version
Docker version 20.10.0, build 7287ab3
+ docker run hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

+ docker images
REPOSITORY               TAG       IMAGE ID       CREATED             SIZE
jenkins-dood_jenkins     latest    4b69f7a5d92f   About an hour ago   877MB
jenkins/jenkins          lts       3f6389c017cc   13 days ago         566MB
gcc                      latest    e4aa98dc41ec   6 weeks ago         1.19GB
docker/getting-started   latest    3c156928aeec   10 months ago       24.8MB
hello-world              latest    bf756fb1ae65   13 months ago       13.3kB
Finished: SUCCESS
```

## Docker Pipeline プラグイン

`Docker Pipeline`プラグインを使っても同じ。

Jenkinsのパイプラインジョブで以下のような構成のパイプラインを設定する。  
（ホストにある`gcc`イメージからコンテナを作成しバージョンを表示するだけ）
```
pipeline {
    agent {
        docker { image 'gcc' }
    }
    stages {
        stage('Test') {
            steps {
                sh 'gcc --version'
            }
        }
    }
}
```

実行した結果。

```bash
Running in Durability level: MAX_SURVIVABILITY
[Pipeline] Start of Pipeline
[Pipeline] node
Running on Jenkins in /var/jenkins_home/workspace/dood-on-pipeline
[Pipeline] {
[Pipeline] isUnix
[Pipeline] sh
+ docker inspect -f . gcc
.
[Pipeline] withDockerContainer
Jenkins seems to be running inside container 088af532f126104ee3d55483f61105735e282b73383c8d1040a97a9b6300f723
$ docker run -t -d -u 1000:1000 -w /var/jenkins_home/workspace/dood-on-pipeline --volumes-from 088af532f126104ee3d55483f61105735e282b73383c8d1040a97a9b6300f723 -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** gcc cat
$ docker top aa54c31546bb4b3a7b5876ce9145c1ee80eef828c3ec0659cb1a95410568ef28 -eo pid,comm
[Pipeline] {
[Pipeline] stage
[Pipeline] { (Test)
[Pipeline] sh
+ gcc --version
gcc (GCC) 10.2.0
Copyright (C) 2020 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

[Pipeline] }
[Pipeline] // stage
[Pipeline] }
$ docker stop --time=1 aa54c31546bb4b3a7b5876ce9145c1ee80eef828c3ec0659cb1a95410568ef28
$ docker rm -f aa54c31546bb4b3a7b5876ce9145c1ee80eef828c3ec0659cb1a95410568ef28
[Pipeline] // withDockerContainer
[Pipeline] }
[Pipeline] // node
[Pipeline] End of Pipeline
Finished: SUCCESS
```

問題なさそう。

ただ、GitLab CI/CDとかに慣れると、Jenkinsのパイプライン設定はとっつきにくい。
