# 所持品管理API

高級品やコレクションアイテムを管理するREST APIサーバーです。

## 📋 API仕様

### ベースURL
```
http://localhost:8080
```

### エンドポイント一覧

| メソッド | パス | 説明 | ステータスコード |
|---------|------|------|-----------------|
| GET | `/health` | ヘルスチェック | 200 |
| GET | `/items` | 全アイテム取得 | 200 |
| POST | `/items` | アイテム登録 | 201, 400 |
| GET | `/items/{id}` | 特定アイテム取得 | 200, 404 |
| PATCH | `/items/{id}` | アイテム部分更新 | 200, 400, 404 |
| DELETE | `/items/{id}` | アイテム削除 | 204, 404 |
| GET | `/items/summary` | カテゴリー別集計 | 200 |

### データ形式

#### アイテム (Item)
```json
{
  "id": 1,
  "name": "ロレックス デイトナ",
  "category": "時計",
  "brand": "ROLEX",
  "purchase_price": 1500000,
  "purchase_date": "2023-01-15",
  "created_at": "2023-01-15T10:00:00Z",
  "updated_at": "2023-01-15T10:00:00Z"
}
```

#### 有効なカテゴリー
- `時計`
- `バッグ`
- `ジュエリー`
- `靴`
- `その他`

### バリデーションルール

#### アイテム登録 (POST)
| フィールド | 必須 | 制限 |
|-----------|------|------|
| name | ✓ | 100文字以内 |
| category | ✓ | 有効なカテゴリーのみ |
| brand | ✓ | 100文字以内 |
| purchase_price | ✓ | 0以上の整数 |
| purchase_date | ✓ | YYYY-MM-DD形式 |

#### アイテム更新 (PATCH)
| フィールド | 必須 | 制限 | 備考 |
|-----------|------|------|------|
| name | - | 100文字以内 | 部分更新可能 |
| brand | - | 100文字以内 | 部分更新可能 |
| purchase_price | - | 0以上の整数 | 部分更新可能 |

**注意**: PATCHでは少なくとも1つのフィールドを指定する必要があります。  
**不変フィールド**: `id`, `category`, `purchase_date`, `created_at` は更新できません。

### API使用例

#### 1. 全アイテム取得
```bash
curl -X GET http://localhost:8080/items
```

**レスポンス:**
```json
[
  {
    "id": 1,
    "name": "ロレックス デイトナ",
    "category": "時計",
    "brand": "ROLEX",
    "purchase_price": 1500000,
    "purchase_date": "2023-01-15",
    "created_at": "2023-01-15T10:00:00Z",
    "updated_at": "2023-01-15T10:00:00Z"
  }
]
```

#### 2. アイテム登録
```bash
curl -X POST http://localhost:8080/items \
  -H "Content-Type: application/json" \
  -d '{
    "name": "エルメス バーキン",
    "category": "バッグ",
    "brand": "HERMÈS",
    "purchase_price": 2000000,
    "purchase_date": "2023-02-20"
  }'
```

#### 3. 特定アイテム取得
```bash
curl -X GET http://localhost:8080/items/1
```

#### 4. アイテム部分更新
```bash
# 名前のみ更新
curl -X PATCH http://localhost:8080/items/1 \
  -H "Content-Type: application/json" \
  -d '{
    "name": "更新されたアイテム名"
  }'

# 複数フィールドを同時更新
curl -X PATCH http://localhost:8080/items/1 \
  -H "Content-Type: application/json" \
  -d '{
    "name": "新しい名前",
    "brand": "新しいブランド",
    "purchase_price": 1800000
  }'
```

**レスポンス:**
```json
{
  "id": 1,
  "name": "更新されたアイテム名",
  "category": "時計",
  "brand": "ROLEX",
  "purchase_price": 1800000,
  "purchase_date": "2023-01-15",
  "created_at": "2023-01-15T10:00:00Z",
  "updated_at": "2023-01-20T15:30:00Z"
}
```

#### 5. アイテム削除
```bash
curl -X DELETE http://localhost:8080/items/1
```

#### 6. カテゴリー別集計
```bash
curl -X GET http://localhost:8080/items/summary
```

**レスポンス:**
```json
{
  "categories": {
    "時計": 2,
    "バッグ": 1,
    "ジュエリー": 3,
    "靴": 0,
    "その他": 1
  },
  "total": 7
}
```

### エラーレスポンス形式

```json
{
  "error": "validation failed",
  "details": [
    "name is required",
    "purchase_price must be 0 or greater"
  ]
}
```

## 🛠️ 技術スタック

- **言語**: Go 1.23
- **フレームワーク**: Echo v4
- **データベース**: MySQL 8.0
- **コンテナ**: Docker & Docker Compose
- **テスト**: testify/mock, testify/assert

## ✨ 実装の特徴

### PATCH /items/{id} エンドポイントの実装
- **部分更新サポート**: 必要なフィールドのみ更新可能
- **ポインタ型の活用**: nil判定で更新対象フィールドを識別
- **不変フィールド保護**: id、category、purchase_date、created_atは変更不可
- **包括的なバリデーション**: 
  - 少なくとも1つのフィールドが必須
  - 各フィールドの値の検証（空文字列、負の値など）
  - エンティティレベルでの整合性チェック
- **自動タイムスタンプ更新**: updated_atは自動的に現在時刻に更新

### テスト戦略
- **モックを使用した単体テスト**: 依存関係を分離
- **テーブル駆動テスト**: 保守性の高いテストコード
- **100%カバレッジ**: 全てのユースケースをテスト
- **正常系・異常系の網羅**: エッジケースも含めた完全なテスト

## 📁 プロジェクト構成

```
.
├── cmd/
│   └── main.go                 # エントリーポイント
├── internal/
│   ├── domain/
│   │   ├── entity/            # ドメインエンティティ
│   │   └── errors/            # ドメインエラー
│   ├── infrastructure/
│   │   ├── config/            # 設定管理
│   │   ├── database/          # データベース接続
│   │   └── server/            # HTTPサーバー
│   ├── interfaces/
│   │   ├── controller/        # HTTPハンドラー
│   │   └── database/          # リポジトリ
│   └── usecase/              # ビジネスロジック
├── sql/
│   └── init.sql              # データベース初期化
├── docker-compose.yml
├── Dockerfile
├── .env.example
└── README.md
```

## 🚀 クイックスタート

### 前提条件
- Docker
- Docker Compose

### サーバーの起動
```bash
# コンテナをビルド・起動
docker-compose up -d --build

# ログを確認
docker-compose logs -f app

# サーバーの停止
docker-compose down
```

サーバーは `http://localhost:8080` で起動します。

### ヘルスチェック
```bash
curl http://localhost:8080/health
```

## 🧪 テスト

### ユニットテストの実行
```bash
# 全テストを実行
docker run --rm -v ${PWD}:/app -w /app golang:1.23-alpine go test -v ./internal/usecase

# カバレッジを表示
docker run --rm -v ${PWD}:/app -w /app golang:1.23-alpine go test -cover ./internal/usecase

# 特定のテストのみ実行
docker run --rm -v ${PWD}:/app -w /app golang:1.23-alpine go test -v ./internal/usecase -run TestItemUsecase_UpdateItem
```

**テストカバレッジ**: 100%

### テストケース一覧
- ✅ GetAllItems - 全アイテム取得
- ✅ GetItemByID - 特定アイテム取得
- ✅ CreateItem - アイテム作成
- ✅ UpdateItem - アイテム部分更新（新規実装）
- ✅ DeleteItem - アイテム削除
- ✅ GetCategorySummary - カテゴリー別集計

## 🔧 ローカル開発

### 前提条件
- Go 1.23以上
- Docker
- Docker Compose

### ローカル開発環境のセットアップ
```bash
# 依存関係をインストール
go mod download

# ローカルでMySQLを起動（docker-compose経由）
docker-compose up -d mysql

# 環境変数を設定（ローカル開発用）
export DB_HOST=localhost
export DB_PORT=3306
export DB_USER=root
export DB_PASSWORD=password
export DB_NAME=items_db

# アプリケーションを起動
go run cmd/main.go
```

### テストデータ

初期データとして以下のアイテムが登録されています：

1. ロレックス デイトナ (時計)
2. エルメス バーキン (バッグ)
3. ティファニー ネックレス (ジュエリー)
4. ルブタン パンプス (靴)
5. アップルウォッチ (その他)
