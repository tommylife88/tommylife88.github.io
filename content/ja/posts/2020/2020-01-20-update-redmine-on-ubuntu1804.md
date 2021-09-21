---
title: "Redmineをアップデートする手順（Ubuntu 18.04 LTS Server）"
date: 2020-01-20
draft: false
tags:
- Redmine
- Ubuntu
categories:
- WEB
---

[Redmine 4.0をUbuntu 18.04 LTS Serverにインストールする手順]({{< ref "/posts/2020/2020-01-15-install-redmine-to-ubuntu1804.md" >}})  
で構築したRedmineをアップデート／バージョンアップする。

## アップデート準備

まずは、 http://redmine.jp/redmine_today/ からRedmineの最新情報を確認しておく。

必ず、

* 最新バージョンが動作環境を満たしていることも確認しておく
* アップデート作業に入る前に必ずデータベース、およびアップロードしているファイルのバックアップをとっておく

こと。

アップデートは基本的に、 http://guide.redmine.jp/RedmineUpgrade/  の公式の手順に従うだけ。

## アップデート

RedmineはSVNリポジトリからチェックアウトしてる前提。

```bash
# Redmineのインストールディレクトに移動（自分の環境に合わせて）
cd /var/www/redmine
# SVNチェックアウトのアップグレード
sudo -u www-data svn update
# 必要なgemを更新
sudo -u www-data bundle update
# データベースの更新
sudo -u www-data RAILS_ENV=production bundle exec rake db:migrate
sudo -u www-data RAILS_ENV=production bundle exec rake redmine:plugins:migrate
# キャッシュのクリア
sudo -u www-data RAILS_ENV=production bundle exec rake tmp:cache:clear
# apacheの再起動
sudo systemctl restart apache2
```

最後に、Redmineサイトに管理者権限でログインして「管理」→「ロールと権限」画面を開き、Redmineのバージョンを確認して、一通り動作確認を行う。
