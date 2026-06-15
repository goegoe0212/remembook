# テーブル定義

## ER図（概要）

```
users
  └── book_titles（タイトルマスタ）
        ├── book_tags ── tags
        └── books（版ごとの書籍）
              ├── purchase_records ── shops
              ├── book_disposals
              └── reading_records
```

---

## テーブル一覧

| テーブル名 | 概要 |
|------------|------|
| users | ユーザー（認証情報含む） |
| book_titles | 書籍タイトルマスタ（作品単位） |
| books | 版ごとの書籍情報（物理的な本の単位） |
| shops | ショップマスタ |
| purchase_records | 購入記録 |
| book_disposals | 売却・譲渡記録 |
| reading_records | 読書記録 |
| tags | タグ |
| book_tags | タイトルとタグの中間テーブル |

---

## users

ユーザー情報。Google SSO など外部認証プロバイダに対応できる構造にする。

| カラム名 | 型 | NOT NULL | 説明 |
|----------|----|----------|------|
| id | UUID | ✓ | PK |
| email | VARCHAR(255) | ✓ | メールアドレス（UNIQUE） |
| name | VARCHAR(100) | ✓ | 表示名 |
| auth_provider | VARCHAR(50) | ✓ | 認証プロバイダ（`local` / `google` など） |
| provider_id | VARCHAR(255) | | 外部プロバイダのユーザーID（SSO用） |
| created_at | TIMESTAMPTZ | ✓ | 作成日時 |
| updated_at | TIMESTAMPTZ | ✓ | 更新日時 |

- `(auth_provider, provider_id)` にユニーク制約

---

## book_titles

書籍のタイトルマスタ。「作品」単位で管理する。複数の版はすべてここに紐づく。

| カラム名 | 型 | NOT NULL | 説明 |
|----------|----|----------|------|
| id | UUID | ✓ | PK |
| user_id | UUID | ✓ | FK → users.id |
| title | VARCHAR(255) | ✓ | タイトル |
| author | VARCHAR(255) | | 著者名 |
| genre | VARCHAR(100) | | ジャンル |
| created_at | TIMESTAMPTZ | ✓ | 作成日時 |
| updated_at | TIMESTAMPTZ | ✓ | 更新日時 |

---

## books

版ごとの書籍情報。`book_titles` の1タイトルに対して複数の版を持てる。購入・売却・読書記録はこのテーブルに紐づく。

| カラム名 | 型 | NOT NULL | 説明 |
|----------|----|----------|------|
| id | UUID | ✓ | PK |
| book_title_id | UUID | ✓ | FK → book_titles.id |
| edition_number | INTEGER | | 版番号（1, 2, 3 ...） |
| edition_label | VARCHAR(50) | | 版の表示名（「改訂版」「増補改訂版」など） |
| isbn | VARCHAR(20) | | ISBN |
| published_at | DATE | | 発行日 |
| cover_image_url | VARCHAR(500) | | 表紙画像URL |
| created_at | TIMESTAMPTZ | ✓ | 作成日時 |
| updated_at | TIMESTAMPTZ | ✓ | 更新日時 |

---

## shops

購入先ショップのマスタ。Amazon・メルカリなどをあらかじめ登録しておく。ユーザーが独自に追加することも可能。

| カラム名 | 型 | NOT NULL | 説明 |
|----------|----|----------|------|
| id | UUID | ✓ | PK |
| name | VARCHAR(100) | ✓ | ショップ名（例：Amazon, メルカリ） |
| url | VARCHAR(255) | | ショップのURL（参考用） |
| is_system | BOOLEAN | ✓ | システム定義のショップかどうか（デフォルト: false） |
| created_at | TIMESTAMPTZ | ✓ | 作成日時 |

---

## purchase_records

書籍（版単位）の購入記録。1冊の本に対して複数回の購入記録を持てる（再購入などを考慮）。

| カラム名 | 型 | NOT NULL | 説明 |
|----------|----|----------|------|
| id | UUID | ✓ | PK |
| book_id | UUID | ✓ | FK → books.id |
| shop_id | UUID | | FK → shops.id（マスタにないショップはNULL） |
| shop_name_free | VARCHAR(100) | | ショップ名の自由入力（shop_idがない場合に使用） |
| purchase_date | DATE | | 購入日 |
| price | INTEGER | | 購入価格（円） |
| memo | TEXT | | メモ |
| created_at | TIMESTAMPTZ | ✓ | 作成日時 |
| updated_at | TIMESTAMPTZ | ✓ | 更新日時 |

---

## book_disposals

書籍（版単位）の売却・譲渡記録。

| カラム名 | 型 | NOT NULL | 説明 |
|----------|----|----------|------|
| id | UUID | ✓ | PK |
| book_id | UUID | ✓ | FK → books.id |
| disposal_type | VARCHAR(20) | ✓ | 種別（`sold` / `transferred`） |
| disposal_date | DATE | | 売却・譲渡日 |
| sale_price | INTEGER | | 売却価格（円）。`sold` の場合のみ使用 |
| memo | TEXT | | メモ |
| created_at | TIMESTAMPTZ | ✓ | 作成日時 |
| updated_at | TIMESTAMPTZ | ✓ | 更新日時 |

---

## reading_records

書籍（版単位）の読書記録。

| カラム名 | 型 | NOT NULL | 説明 |
|----------|----|----------|------|
| id | UUID | ✓ | PK |
| book_id | UUID | ✓ | FK → books.id |
| status | VARCHAR(20) | ✓ | 読書ステータス（`unread` / `reading` / `finished`） |
| started_at | DATE | | 読み始め日 |
| finished_at | DATE | | 読了日 |
| memo | TEXT | | 感想・メモ |
| created_at | TIMESTAMPTZ | ✓ | 作成日時 |
| updated_at | TIMESTAMPTZ | ✓ | 更新日時 |

---

## tags

ユーザーが作成したタグ。

| カラム名 | 型 | NOT NULL | 説明 |
|----------|----|----------|------|
| id | UUID | ✓ | PK |
| user_id | UUID | ✓ | FK → users.id |
| name | VARCHAR(50) | ✓ | タグ名 |
| created_at | TIMESTAMPTZ | ✓ | 作成日時 |

- `(user_id, name)` にユニーク制約

---

## book_tags

タイトルとタグの中間テーブル。タグはタイトル単位で管理する。

| カラム名 | 型 | NOT NULL | 説明 |
|----------|----|----------|------|
| book_title_id | UUID | ✓ | FK → book_titles.id |
| tag_id | UUID | ✓ | FK → tags.id |

- `(book_title_id, tag_id)` を複合 PK とする
