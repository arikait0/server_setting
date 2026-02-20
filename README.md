## 1.	目的と完成条件の定義
　Docker Compose による Web＋DB の 2 コンテナ構成を行う`http://localhost:8080`にアクセスし、`<h1>ok</h1>`と表示されば成功である。

## 2. 前提条件

| 項目 | 内容 |
|------|------|
| 対象OS | Ubuntu 24.04 LTS (on WSL2) |
| 実行環境 | Windows 11 + WSL2 |
| パッケージ管理 | apt, Docker Engine, Docker Compose |
| ネットワーク | http://localhost:8080 |

## 3.	事前準備
```
# WSLのインストール
wsl --install

# パッケージの更新
sudo apt update && sudo apt upgrade -y

# Docker のインストール
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# ユーザーの docker グループへの追加
sudo usermod -aG docker $USER
newgrp docker

# Dockerを起動
sudo service docker start

# 8080ポートの解放（これがないと外部ブラウザから見えません）
sudo ufw allow 8080/tcp
```
## 4.	構築手順
```
# ディレクトリの作成
mkdir -p ~/docker-webserver/htdocs

# docker-webserverへ移動
cd ~/docker-webserver

# Nginxが配信するためのテスト用HTMLファイルの作成
echo '<h1>ok</h1>' > htdocs/index.html

# 設定ファイルの作成
vi compose.yaml
```
### yaml の詳細
| 項目 | 内容 |
|------|------|
| 保存パス | ~/docker-webserver/compose.yaml |
| 所有者 | 実行ユーザー |
| グループ | dockerグループ |
| アクセス権限 | 644（-rw-r--r--） |

```~/docker-webserver/compose.yaml
# 以下の内容を記述して保存する

services:
  # Webサーバ（Nginx）
  web:
    image: nginx:latest
    ports:
      - "8080:80"
    volumes:
      - ./htdocs:/usr/share/nginx/html
    depends_on:
      - db

  # データベース（MariaDB）
  db:
    image: mariadb:latest
    environment:
      MYSQL_ROOT_PASSWORD: password123
      MYSQL_DATABASE: sample_db
    volumes:
      - db_data:/var/lib/mysql

volumes:
  db_data:

```

## 5.	動作確認と検証
```
# コンテナ群の構築とバックグラウンドの起動
docker compose up -d

# データベースのログを表示し、正常に起動したか確認
docker compose logs db

curl  http://localhost:8080
# → <h1>ok</h1>と表示されたら成功

# コンテナ群の削除
docker compose down 
```
## 6.	トラブルシューティング
Docker のインストールでエラーがでる場合 `cd ~` と打ち込んでホームディレクトリに移動してください。また、`WSL DETECTED`と表示される場合は、20秒ほど待てばインストールが始まります。

## 7.	参考文献
木村佑太.「14.Dockerを用いた環境構築」,(大阪公大),2025.https://brawny-slicer-af3.notion.site/14-Docker-2b85e5dbbc3e818291b4d3fbf12b82ff#39e340bc7f434b86802a12bb8facec49
