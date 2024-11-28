## デプロイ手順 (ubuntu)

```bash
# EC2 インスタンス作成
# マネジメントコンソールから EC2 インスタンスを作成する。

# nginx インストール
sudo apt-get update && sudo apt-get install nginx

# nginx インストール確認
nginx -v

# nginx 設定ファイル確認
ls /etc/nginx/

# nginx 起動
sudo systemctl start nginx

# nginx 起動確認
sudo systemctl status nginx

# リソース配置
# デプロイコマンド実施など。テスト用にはgit pullをインスタンス内から実施する
sudo apt-get update && sudo apt-get install git -y && git clone {リポジトリURL}

# ウェブサイトディレクトリ作成
sudo mkdir -p /var/www/html/resume/public

# ディレクトリ権限確認
ls -l /var/www/html/resume

# ディレクトリ権限変更
sudo chown -R $USER:$USER /var/www/html/resume/public/

# シンボリックリンク作成
sudo ln -s ~/resume/ /var/www/html/resume/public/

# シンボリックリンク確認
ls -la /var/www/html/resume/public/
ls -la /var/www/html/resume/public/resume

# ウェブサイト実行ユーザーに閲覧権限を与える
chmod 755 ~

# nginx 設定ファイル編集
# root /var/www/html; を root /var/www/html/resume/public/resume; に変更
# index index.html index.htm; を index index.html; に変更
sudo vi /etc/nginx/sites-available/default


# 設定を反映
sudo systemctl restart nginx

# シンボリックリンク作成
ln -s /etc/nginx/sites-available/default /etc/nginx/sites-enabled

# インスタンスのインバウンドルールのセキュリティグループを編集して、HTTP 80 ポート0.0.0.0/0に開放
# なかなかアクセスできずタイムアウトになる場合上記のSG設定などが間違っている可能性が高い

# http://<Public IP>/resume でアクセスできることを確認(httpsではない)

# うまくいかなかったが、以下実行で解決。どれかは不明
chown $USER:$USER /home/ubuntu/resume
sudo chown $USER:$USER /var/www/html/resume/public/resume
sudo chown $USER:$USER /home/ubuntu/resume/
sudo chown -h $USER:$USER /home/ubuntu/resume
sudo chown -h ubuntu:ubuntu /home/ubuntu/resume
sudo chown -h ubuntu:ubuntu /var/www/html/resume/public/resume
sudo chown -R $USER:$USER /var/www/
chmod 755 /home/ubuntu

# DNSの設定
# ドメインを以下のように設定
# Aレコード、mainドメイン(wwwなどなし)をec2のパブリックIPに設定
# CNAMEレコード、wwwサブドメインをmainドメインに設定
# CNAMEレコード、resumeサブドメインをmainドメインに設定
# resume.ドメイン でアクセスできることを確認

# 一旦nginxの設定を元に戻す
sudo vi /etc/nginx/sites-available/default

# 以下を残す
# root /var/www/html;
# index index.html index.htm index.nginx-debian.html;

# 設定の反映
sudo systemctl restart nginx

# サブドメインの設定を分ける
sudo vi /etc/nginx/sites-available/resume.ドメイン
```

以下を設定ファイルに記述

```nginx
server {
    listen 80;
    listen [::]:80;

    root /var/www/html/resume/public/resume;
    index index.html;

    server_name resume.ドメイン;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

```bash
# シンボリックリンク作成
sudo ln -s /etc/nginx/sites-available/resume.ドメイン /etc/nginx/sites-enabled

# シンボリックリンク確認
ls -la /etc/nginx/sites-enabled

# 設定の反映
sudo systemctl restart nginx

# 以上でresume.ドメイン でアクセスできることを確認
# ドメインへのアクセスでnginxのデフォルトページが表示されることを確認

# certbot インストール
sudo snap install core && sudo snap refresh core && sudo snap install --classic certbot && sudo ln -s /snap/bin/certbot /usr/bin/certbot

# certbot 確認
certbot --version

# certbot でtsl設定
sudo certbot --nginx -d resume.ドメイン

# インバウンドルールのセキュリティグループを編集して、HTTPS 443 ポート、0.0.0.0/0に開放
```

## TODO: コンテンツ

コンテンツは現在サンプル。後ほど更新予定。

## ファイル閲覧で fetch cors エラー

```bash
npm install -g http-server
http-server
```
