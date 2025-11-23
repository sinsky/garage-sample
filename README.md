# Garage Sample

Garageを使用したS3互換オブジェクトストレージのサンプル環境です。TraefikをリバースプロキシとしてGarageのS3 API、Web、管理UIにアクセスできます。

> [!WARNING]
> このリポジトリは学習・検証目的で利用してください。Garageの推奨は3ノード以上のクラスタ構成です。

## 構成

このプロジェクトには以下のサービスが含まれています:

- **Garage** (v2.1.0): S3互換オブジェクトストレージ
- **Traefik** (v3.6): リバースプロキシ
- **Docker Socket Proxy**: Dockerソケットへの読み取り専用アクセス
- **Garage WebUI**: Garage管理用Webインターフェース

## 前提条件

- Docker
- Docker Compose

## 使用方法

### 1. 環境変数の設定

`.env.sample`をコピーして`.env`ファイルを作成し環境変数を設定、あるいは次のスクリプトを実行して`.env`を生成してください

手動で作成する場合

```bash
cp .env.sample .env
# 必要に応じて.envファイルを編集してください
```

自動で作成する場合

```bash
cat > .env << EOF
GARAGE_RPC_SECRET="$(openssl rand -hex 32)"
GARAGE_ADMIN_TOKEN="$(openssl rand -base64 32)"
GARAGE_METRICS_TOKEN="\$(openssl rand -base64 32)"
EOF
```

### 2. 起動

```bash
docker compose up -d
```

### 3. アクセス

以下のURLでサービスにアクセスできます:

- **Garage WebUI**: <http://webui.127.0.0.1.traefik.me>
- **S3 API**: <http://s3.127.0.0.1.traefik.me>
- **Web公開**: `*.web.127.0.0.1.traefik.me` (バケット名をサブドメインとして使用)

### 4. Garageの初期設定

初回起動時は、Garageクラスタの初期設定が必要です:

```bash
# Garageコンテナに入る
docker exec -it garage sh

# ノードIDを確認
garage status

# レイアウトを適用
garage layout assign -z dc1 -c 1G <ノードID>
garage layout apply --version 1

# Garageキーペアを作成
garage key new my-key

# バケットを作成してキーに紐づける
garage bucket create my-bucket
garage bucket allow --read --write my-bucket --key my-key
```

### 5. S3アクセスの設定

AWS CLIやS3互換クライアントで以下の設定を使用します:

- **Endpoint**: <http://s3.127.0.0.1.traefik.me>
- **Access Key**: Garageで作成したキーのAccess Key ID
- **Secret Key**: Garageで作成したキーのSecret Access Key
- **Region**: garage (任意の値)

### 6. 停止

```bash
docker compose down
```

### 7. データの永続化

以下のディレクトリにデータが保存されます:

- `./meta`: Garageのメタデータ
- `./data`: オブジェクトデータ

完全にリセットする場合は、これらのディレクトリを削除してください。

## ファイル構成

- `compose.yml`: Docker Compose設定
- `garage.toml`: Garage設定ファイル
- `traefik.yml`: Traefik設定ファイル

## ポート

- `80`: Traefik (HTTPエントリーポイント)

内部的に以下のポートが使用されます:

- `3900`: Garage S3 API
- `3902`: Garage Web公開
- `3903`: Garage Admin API
- `3909`: Garage WebUI

## トラブルシューティング

### サービスの状態確認

```bash
docker compose ps
docker compose logs
```

### Garageのステータス確認

```bash
docker exec garage garage status
```

## 参考リンク

- [Garage Documentation](https://garagehq.deuxfleurs.fr/)
- [Traefik Documentation](https://doc.traefik.io/traefik/)
