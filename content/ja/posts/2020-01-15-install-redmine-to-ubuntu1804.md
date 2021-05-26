---
title: "Redmine 4.0をUbuntu 18.04 LTS Serverにインストールする手順"
date: 2020-01-15
draft: false
tags:
- Redmine
- Ubuntu
categories:
- WEB
---

最小構成でインストールしたUbuntu 18.04.1 LTS ServerにRedmine 4.0をインストールする手順を残しておく。

{{< alert theme="info" >}}
UbuntuのLTSの最新は20.04だが、Redmineの依存ライブラリ（Ruby等）がUbuntuの標準パッケージのものは新しすぎて環境整備が大変。
現時点では18.04が楽と思われる。（Redmineのバージョンアップに期待）
{{< /alert >}}

要件は公式を確認しておこう。  
http://guide.redmine.jp/RedmineInstall/

## サーバー環境

Redmine以外は基本的にOS標準のパッケージで足りていることを想定しています。

* OS：Ubuntu 18.04.1 LTS Server
* Redmine：Redmine 4.0 stable
* データベース：MySQL 5.7.26
* Webサーバー：Apache 2.4.18 (PassengerでRails実行)
* Ruby：2.5.1

## OSの設定

OSインストール直後、パッケージの最新化、日本語設定。

```bash
sudo locale-gen ja_JP.UTF-8
sudo update-locale LANG=ja_JP.UTF-8
sudo dpkg-reconfigure tzdata
sudo apt update
sudo apt upgrade -y
sudo apt-get dist-upgrade
```

## 必要なパッケージをインストール

一応、Redmine 4.0に必要なRubyがOS標準で入手可能か確認する。公式によるとRedmine 4.0の場合、Ruby 2.2.2 以降、Rails 5.2が必須。

```bash
sudo apt update
apt-cache madison ruby
#ruby |    1:2.5.1 | http://archive.ubuntu.com/ubuntu bionic/main amd64 Packages
sudo apt install ruby-dev ruby bundler apache2 libapache2-mod-passenger imagemagick libmagick++-dev subversion mysql-server libmysqlclient-dev
#ruby -v
ruby 2.5.1p57 (2018-03-29 revision 63029) [x86_64-linux-gnu]
```

## MySQLの設定

文字コードの設定、ユーザーとデータベースの設定。

```bash
sudo vi /etc/mysql/conf.d/redmine.cnf
sudo chmod 644 /etc/mysql/conf.d/redmine.cnf
```
redmine.cnf
```ini
[mysqld]
innodb_file_format = Barracuda
innodb_file_per_table = 1
innodb_large_prefix = 1
character-set-server = utf8mb4
skip-character-set-client-handshake
collation-server = utf8mb4_general_ci
init-connect = SET NAMES utf8mb4

[mysql]
default-character-set = utf8mb4

[client]
default-character-set = utf8mb4

[mysqldump]
default-character-set = utf8mb4
```

`mysql_secure_installation`で`root`ユーザーのパスワード等、MySQLをインストールする。

```bash
sudo mysql_secure_installation
sudo service mysql restart
sudo mysql -uroot -p
```

ここからMySQLのコマンドプロンプト上で。  
Redmine用のデータベースを`redmine`という名前で作成し、`localhost`の`redmine`というユーザー(パスワード`my_password`)にredmineデータベースへのすべての権限を与えている。

```bash
CREATE DATABASE redmine CHARACTER SET utf8mb4;
CREATE USER 'redmine'@'localhost' IDENTIFIED BY 'my_password';
GRANT ALL PRIVILEGES ON redmine.* TO 'redmine'@'localhost';
```

## Redmineの設定

Redmineアプリケーション実行用のフォルダを作成し、オーナーとグループを`www-data`に設定する。(apacheプロセスがRedmine環境のユーザーになるため、apacheプロセスへのアクセス権を与える)

```bash
sudo mkdir -p /var/www/redmine
sudo chown www-data:www-data /var/www/redmine
sudo -u www-data svn co http://svn.redmine.org/redmine/branches/4.0-stable /var/www/redmine
sudo -u www-data vi /var/www/redmine/config/database.yml
```

データベースの設定ファイル`config/database.yml`を以下のように設定する。

```yml:config/database.yml
production:
 adapter: mysql2
 database: redmine
 host: localhost
 username: redmine
 password: "my_redmine"
 encoding: utf8mb4
 charset: utf8mb4
 collation: utf8mb4_general_ci
 pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
 socket: /var/run/mysqld/mysqld.sock
```

utf8mb4の有効化。

```bash
sudo -u www-data vi /var/www/redmine/config/initializers/utf8mb4.rb
```

```rb:config/initializers/utf8mb4.rb
module Utf8mb4
 def create_table(table_name, options = {})
   table_options = options.merge(options: 'ENGINE=InnoDB ROW_FORMAT=DYNAMIC')
   super(table_name, table_options) do |td|
     yield td if block_given?
   end
 end
end
ActiveSupport.on_load :active_record do
 module ActiveRecord::ConnectionAdapters
   class AbstractMysqlAdapter
     prepend Utf8mb4
   end
 end
end
```

## Redmineのインストール

RubyGemの依存解決と、データベース構築など。

```bash
sudo chown -R www-data:www-data /var/www/redmine
cd /var/www/redmine/
sudo gem update bundler
sudo apt-get remove bundler # apt の bundler は要らなかった
sudo -u www-data bundle install --without development test postgresql sqlite --path vendor/bundle
sudo -u www-data bundle exec rake generate_secret_token
sudo -u www-data RAILS_ENV=production bundle exec rake db:migrate
sudo -u www-data RAILS_ENV=production REDMINE_LANG=ja bundle exec rake redmine:load_default_data
```

## Apache連携

RedmineとApcheの連携。PassengerはApache上でRailsアプリを動かすもの。

### Passenger設定

```bash
sudo vi /etc/apache2/mods-available/passenger.conf
```

```ini:/etc/apache2/mods-available/passenger.conf
<IfModule mod_passenger.c>
 PassengerRoot /usr/lib/ruby/vendor_ruby/phusion_passenger/locations.ini
 PassengerDefaultRuby /usr/bin/ruby
 PassengerPreStart http://127.0.0.1:80/redmine/
</IfModule>
```

IPアドレスやポートは適宜変更する。

### Redmine設定

```bash
sudo vi /etc/apache2/sites-available/redmine.conf
```

/etc/apache2/sites-available/redmine.conf
```ini
listen 80
<VirtualHost *:80>
   ServerName redmine.com

   ServerAdmin redmine@redmine.com
   DocumentRoot /var/www/html

   RackBaseURI /redmine
   PassengerHighPerformance on
   <Directory /var/www/redmine/public>
#        Require ip ::1 127. 192.168.
       AllowOverride None
       Options None
   </Directory>

   ErrorLog ${APACHE_LOG_DIR}/error.log
   CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
RailsEnv production
RailsBaseURI /redmine
```

IPアドレスやポートは適宜変更する。

```bash
sudo ln -s /var/www/redmine/public /var/www/html/redmine
sudo a2ensite redmine
```

## 起動！

Apacheを再起動して運用開始！！

```bash
sudo systemctl reload apache2
```

ブラウザから`http://127.0.0.1:80/redmine/`にアクセスしてRedmineのページが表示されれば作業完了です！
