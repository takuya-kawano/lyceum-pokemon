# lyceum-pokemon

ポケモン API を使った Nuxt+Express アプリ/サーバの開発演習

## 準備

- AWS 認証情報のアクセスキー ID とシークレットアクセスキーを生成してください
- AWS S3 バケットを作成してください

## 実行環境

- 本リポジトリをクローンして実行してください
- 環境変数は https://ja.vitejs.dev/guide/env-and-mode.html#env-files にしたがって `.env` などに設定してください

## 使い方

- `npm install`: npm パッケージのインストール
- `npm run dev`: 開発サーバーの起動
- `npm run build`: アプリケーションのプロダクションビルド
- `npm start`: プロダクションビルドを使ったローカルサーバーの起動
- `npm run lint`: コードリント
- `npm run format`: コード整形

## 環境変数

| 変数名                  | 説明                                                     | 初期値                    |
| :---------------------- | :------------------------------------------------------- | :------------------------ |
| `AWS_ACCESS_KEY_ID`     | AWS 認証情報のアクセスキー ID                            | なし                      |
| `AWS_SECRET_ACCESS_KEY` | AWS 認証情報のシークレットアクセスキー                   | なし                      |
| `REGION`                | AWS のリージョン                                         | `"ap-northeast-1"`        |
| `BUCKET_NAME`           | 本アプリケーションのデータ永続化に用いる AWS S3 バケット | `""`                      |
| `VITE_SERVER_ORIGIN`    | 用意したサーバー側 の API リクエストに用いるオリジン     | `"http://localhost:3000"` |

## クライアント画面構成

| 画面名               | 機能                                                                                                                   |
| :------------------- | :--------------------------------------------------------------------------------------------------------------------- |
| スタート             | 「つづきからはじめる」と「あたらしくはじめる」に遷移できる                                                             |
| あたらしくはじめる   | トレーナー名を入力してトレーナーが追加できる                                                                           |
| つづきからはじめる   | どのトレーナーを表示するか選択できる                                                                                   |
| トレーナー情報       | 「ポケモンをつかまえる」に遷移できる、ポケモンにニックネームをつけられる、ポケモンを削除できる、トレーナーを削除できる |
| ポケモンをつかまえる | ポケモンを追加できる                                                                                                   |

## ER 図

![トレーナー{名前（主キー）、手持ちポケモン}<-一（必須）対多（任意）->ポケモン{手持ちポケモン識別子（主キー）、ニックネーム、ポケモン図鑑番号、名前、スプライト（画像）}](https://github.com/webdino/lyceum-pokemon/raw/main/docs/pokemon.drawio.png)

## サーバー API と AWS S3 の対応関係

| サーバー API         | AWS S3                       |
| :------------------- | :--------------------------- |
| トレーナー一覧の取得 | オブジェクト一覧の取得       |
| トレーナーの追加     | オブジェクトの追加または更新 |
| トレーナーの取得     | オブジェクトの取得           |
| トレーナーの更新     | オブジェクトの追加または更新 |
| トレーナーの削除     | オブジェクトの削除           |
| ポケモンの追加       | オブジェクトの追加または更新 |
| ポケモンの削除       | オブジェクトの追加または更新 |

## S3 バケットに作成するオブジェクトのサンプル

トレーナー名が `レッド` の場合

- `レッド.json`: S3 オブジェクトキー（ファイル名）
- 次のコードブロック: S3 オブジェクト値（ファイル内容）

```json:レッド.json
{
  "name": "レッド",
  "pokemons": [
    {
      "id": 1,
      "nickname": "",
      "order": 35,
      "name": "pikachu",
      "sprites": {
        "front_default": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/25.png"
      }
    },
    {
      "id": 2,
      "nickname": "",
      "order": 220,
      "name": "espeon",
      "sprites": {
        "front_default": "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/196.png"
      }
    }
  ]
}
```

## API エンドポイント

### `/api/pokeapi`

https://pokeapi.co/api/v2/ へのプロキシー

#### パラメーター

なし

#### レスポンス

https://pokeapi.co/docs/v2 に準じる

### GET `/api/trainers`

トレーナーの一覧の取得

#### パラメーター

なし

#### レスポンス

##### 200

```json
["コジロウ", "サトシ", "ムサシ", "レッド"]
```

### POST `/api/trainer`

トレーナーの追加

#### パラメーター

なし

#### リクエストボディ

- `name`: トレーナー名（必須）
- `pokemons`: 手持ちポケモン（任意）

```json
{ "name": "satoshi" }
```

#### レスポンス

##### 200

[PutObjectCommandOutput](https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/clients/client-s3/interfaces/putobjectcommandoutput.html)

##### 400

空（リクエストボディに必要なプロパティが含まれていない場合に返される）

##### 409

空（すでにトレーナーが存在する場合に返される）

### GET `/api/trainer/:trainerName`

トレーナーの取得

#### パラメーター

- `trainerName`: トレーナー名

#### レスポンス

##### 200

```json
{ "name": "satoshi", "pokemons": [] }
```

### POST `/api/trainer/:trainerName`

トレーナーの更新

#### パラメーター

- `trainerName`: トレーナー名

#### リクエストボディ

- `name`: トレーナー名（必須）
- `pokemons`: 手持ちポケモン（任意）

```json
{ "name": "satoshi" }
```

#### レスポンス

##### 200

[PutObjectCommandOutput](https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/clients/client-s3/interfaces/putobjectcommandoutput.html)

##### 404

空（トレーナーが存在しない場合に返される）

### DELETE `/api/trainer/:trainerName`

トレーナーの削除

#### パラメーター

- `trainerName`: トレーナー名

#### レスポンス

##### 204

[DeleteObjectCommandOutput](https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/clients/client-s3/interfaces/deleteobjectcommandoutput.html)

### PUT `/api/trainer/:trainerName/pokemon/:pokemonName`

ポケモンの追加

#### パラメーター

- `trainerName`: トレーナー名
- `pokemonName`: ポケモン名

#### レスポンス

##### 200

[PutObjectCommandOutput](https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/clients/client-s3/interfaces/putobjectcommandoutput.html)

### DELETE `/api/trainer/:trainerName/pokemon/:pokemonId`

ポケモンの削除

#### パラメーター

- `trainerName`: トレーナー名
- `pokemonId`: 手持ちポケモン識別子

#### レスポンス

##### 200

[DeleteObjectCommandOutput](https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/clients/client-s3/interfaces/deleteobjectcommandoutput.html)
