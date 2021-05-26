---
title: "「超簡単」仮想環境でWiki「GROWI」を導入する（Markdown対応）"
date: 2018-12-13
draft: false
tags:
- Markdown
- Wiki
- vagrant
- docker
- docker-compose
categories:
- WEB
---

仕事でいまだに業務のノウハウとかをExcelファイルで管理してませんか？  
いちいちくそ重いExcelを起動して、セルを方眼紙みたくして、さらには手書きで変更履歴を書いて・・・考えるだけで鬱です。

もうそんな運用はやめて、超快適なWikiプラットフォームを導入してWEBベースでノウハウを蓄積しよう。  
今回インストールするのは [Growi](https://growi.org/) になります。こちらはオープンソースソフトウェアでブラウザベースで動作します。

Markdownに対応しており、リアルタイムプレビュー機能付きなのでライティング、リライティングは爆速です。もちろん複数人OK、変更履歴も管理してくれます。

開発はとても賑わっているようです。（→ [Growi GitHub](https://github.com/weseek/growi)）ありがたや。

ただインストールするだけなら面白くないので、今回は仮想環境上で作っちゃいます。仮想環境はVagrant＋VirtuaBox。Vagrantとかの詳しい話は割愛します。  
うまくいけば、ホストPCが変更になってもWikiの移設が簡単簡単。

## 構築したときの環境

筆者の環境
* ホストPC：Windows 10 Home（64bit）
  * CPU：Intel Core i7-7700HQ
  * RAM：16GB
  * Vagrant 2.2.3
  * VirtualBox 6.0.4
* ゲストPC（仮想PC）：Ubuntu 18.04.2 LTS（bento/ubuntu-18.04）
  * Docker 18.09.2
  * docker-compose 1.24.0-rc1

※ホストPCだが、サーバー用途ならWindowsじゃなくてLinux系OSのほうが絶対良い。

## 仮想環境を整える

[公式](https://www.vagrantup.com/downloads)に従い、ホストPCに`Vagrant`インストール  
https://www.vagrantup.com/downloads

[公式](https://www.virtualbox.org/wiki/Downloads)に従い、ホストPCに`VirtualBox`インストール  
https://www.virtualbox.org/wiki/Downloads

## Vagrant＋VirtualBoxでゲストPC（仮想PC）を作る

ホストPCでコマンドプロンプトを起動し、vagrant boxイメージを生成します。  
ここでは作業ディレクトリとしてDドライブのvagrantフォルダにgrowiフォルダを作ることにする。

```bash
D:\vagrant> mkdir growi
D:\vagrant> cd growi
D:\vagrant\growi> vagrant init bento/ubuntu-18.04
D:\vagrant\growi> vagrant up
D:\vagrant\growi> vagrant ssh
```

自動生成されたVagrantファイルは後で弄る。  
ゲストPCにSSHでログインできればOK。

## ゲストPCの環境設定

{{< alert theme="warning" >}}
ここからはゲストPC側の設定です。  
パッケージの最新化と日本語環境、タイムゾーン設定を行う。
{{< /alert >}}

```bash
vagrant@vagrant:~$ sudo apt update && sudo apt upgrade -y
vagrant@vagrant:~$ sudo locale-gen ja_JP.UTF-8
vagrant@vagrant:~$ sudo localectl set-locale LANG=ja_JP.UTF-8 LANGUAGE="ja_JP:ja"
vagrant@vagrant:~$ exit #いったんゲストPCから抜ける
D:\vagrant\growi> vagrant ssh
vagrant@vagrant:~$ echo $LANG
ja_JP.UTF-8
vagrant@vagrant:~$ sudo dpkg-reconfigure tzdata
#[アジア]-[東京]を選択しましょう
vagrant@vagrant:~$ exit #いったんゲストPCから抜ける
D:\vagrant\growi> vagrant reload #ゲストPCを再起動
D:\vagrant\growi> vagrant ssh
```

## ゲストPCに Docker インストール

[公式](https://docs.docker.com/engine/install/)に従い、`Docker`をインストールする。  
https://docs.docker.com/engine/install/

```bash
vagrant@vagrant:~$ curl -fsSL https://get.docker.com/ | sudo sh
vagrant@vagrant:~$ docker -v
Docker version 18.06.2-ce, build 6d37f41
vagrant@vagrant:~$ sudo usermod -aG docker $USER #root権限なしで実行する
vagrant@vagrant:~$ exit #いったんゲストPCから抜ける
D:\vagrant\growi> vagrant ssh
vagrant@vagrant:~$ groups
#dockerに所属していることを確認する
```

## ゲストPCに Docker Compose インストール

[公式](http://docs.docker.jp/compose/install.html)に従い、Docker Composeをインストールする。  
http://docs.docker.jp/compose/install.html

```bash
vagrant@vagrant:~$ sudo curl -L https://github.com/docker/compose/releases/download/1.24.0-rc1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
vagrant@vagrant:~$ sudo chmod +x /usr/local/bin/docker-compose
vagrant@vagrant:~$ docker-compose -v
docker-compose version 1.24.0-rc1, build 0f3d4dda
```

## ゲストPCに Growi インストール

お持たせしました！ようやく「Growi」をインストールします。
といってもコマンド一発で`Docker`さんがイイ感じにしれくれます。

[GitHubにある手順](https://github.com/weseek/growi-docker-compose)に従い、Growiをインストールする。  
https://github.com/weseek/growi-docker-compose

ホストPCからゲストPCにアクセスできるよう、`docker-compose.yml`の`ports`を変更しておきます。

```bash
vagrant@vagrant:~$ git clone https://github.com/weseek/growi-docker-compose.git growi
vagrant@vagrant:~$ cd growi
vagrant@vagrant:~$ vi docker-compose.yml
```

弄るのは１行だけ。

```yml:docker-compose.yml
ports:
#- 127.0.0.1:3000:3000    # localhost only by default
 - 3000:3000 # ← modified
```

アプリをビルドします。

```bash
vagrant@vagrant:~$ docker-compose up
#ひたすらログが流れて
app_1 ・・・ : [production] Express server is listening on port 3000
#↑が表示されれば完了した
```

完了後、`http://localhost:3000`にアクセスするとGrowiのセットアップ画面が表示されるはず・・。がアクセスできません。

Vagrantのポートフォワードの設定が忘れてましたね。  
**このアプリはデフォルトで3000番ポートで待ち受けているようですので、ホストPCの12345番ポート（ここは任意）をゲストPCの3000番ポートにポートフォワード設定させます。**

Ctrl+Cでアプリを止め、ゲストPCを抜けVagrantfileを弄ります。
Vagrantfileはこうなりました。こちらは超最小限の設定です。

```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :
 
# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
 # The most common configuration options are documented and commented below.
 # For a complete reference, please see the online documentation at
 # https://docs.vagrantup.com.
 
 # Every Vagrant development environment requires a box. You can search for
 # boxes at https://vagrantcloud.com/search.
 config.vm.box = "bento/ubuntu-18.04"
 
 # Disable automatic box update checking. If you disable this, then
 # boxes will only be checked for updates when the user runs
 # `vagrant box outdated`. This is not recommended.
 # config.vm.box_check_update = false
 
 # Create a forwarded port mapping which allows access to a specific port
 # within the machine from a port on the host machine. In the example below,
 # accessing "localhost:8080" will access port 80 on the guest machine.
 # NOTE: This will enable public access to the opened port
 # config.vm.network "forwarded_port", guest: 80, host: 8080
 config.vm.network "forwarded_port", guest: 3000, host: 12345
 
 # Create a forwarded port mapping which allows access to a specific port
 # within the machine from a port on the host machine and only allow access
 # via 127.0.0.1 to disable public access
 # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"
 
 # Create a private network, which allows host-only access to the machine
 # using a specific IP.
 # config.vm.network "private_network", ip: "192.168.33.10"
 
 # Create a public network, which generally matched to bridged network.
 # Bridged networks make the machine appear as another physical device on
 # your network.
 # config.vm.network "public_network"
 
 # Share an additional folder to the guest VM. The first argument is
 # the path on the host to the actual folder. The second argument is
 # the path on the guest to mount the folder. And the optional third
 # argument is a set of non-required options.
 # config.vm.synced_folder "../data", "/vagrant_data"
 
 # Provider-specific configuration so you can fine-tune various
 # backing providers for Vagrant. These expose provider-specific options.
 # Example for VirtualBox:
 #
 # config.vm.provider "virtualbox" do |vb|
 #   # Display the VirtualBox GUI when booting the machine
 #   vb.gui = true
 #
 #   # Customize the amount of memory on the VM:
 #   vb.memory = "1024"
 # end
 #
 # View the documentation for the provider you are using for more
 # information on available options.
 
 # Enable provisioning with a shell script. Additional provisioners such as
 # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
 # documentation for more information about their specific syntax and use.
 # config.vm.provision "shell", inline: <<-SHELL
 #   apt-get update
 #   apt-get install -y apache2
 # SHELL
end
```

ゲストPCを再起動して、再度SSHではいりアプリを起動します。  
dockerイメージはビルド済みなのでstartコマンドでOK。

```bash
D:\vagrant\growi> vagrant reload
D:\vagrant\growi> vagrant ssh
vagrant@vagrant:~$ cd growi
vagrant@vagrant:~$ docker-compose start
Starting mongo         ... done
Starting elasticsearch ... done
Starting app           ... done
```

`http://localhost:12345`でセットアップ画面が表示されましたね。

LAN内の他のPCからも、`http://[ホストPCのIPアドレス]:12345`でアクセス可能になっているはずです。  
localhostではOKで、外部からのIP指定でダメならホストPCのファイアウォールとかの設定が怪しいかも、ここでは割愛します。  
そもそもサーバー用途ならWindowsはおすすめしないです・・。


ではでは、良いGrowiライフを！
