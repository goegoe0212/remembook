# アーキテクチャ概要

## 技術スタック

| 層 | 技術 |
|----|------|
| フロントエンド | Flutter (iOS / Web) |
| バックエンド | C# / ASP.NET Core (REST API) |
| データベース | PostgreSQL |
| インフラ | Kubernetes (コンテナ) |

## システム構成

```
┌─────────────────────────────────────┐
│  クライアント                         │
│  ┌─────────────┐  ┌──────────────┐  │
│  │ Flutter iOS │  │ Flutter Web  │  │
│  └──────┬──────┘  └──────┬───────┘  │
└─────────┼────────────────┼──────────┘
          │  HTTP/REST      │
          ▼                 ▼
┌─────────────────────────────────────┐
│  Kubernetes Cluster                 │
│  ┌──────────────────────────────┐   │
│  │  ASP.NET Core API (Pod)      │   │
│  └──────────────┬───────────────┘   │
│                 │                   │
│  ┌──────────────▼───────────────┐   │
│  │  PostgreSQL (Pod)            │   │
│  └──────────────────────────────┘   │
└─────────────────────────────────────┘
```

## iOS / Web でデータ同期する理由

iOS と Web の両プラットフォームでデータを共有するため、ローカル DB ではなく ASP.NET Core API 経由で PostgreSQL にアクセスする設計とする。これにより、どのデバイスからでも同じデータを参照・更新できる。

## ディレクトリ構成

```
remembook/
├── apps/
│   ├── frontend/         # Flutter アプリ (iOS / Web)
│   │   ├── lib/
│   │   │   ├── main.dart
│   │   │   ├── features/ # 機能ごとのモジュール
│   │   │   │   ├── book/
│   │   │   │   └── shelf/
│   │   │   ├── shared/   # 共通コンポーネント・ユーティリティ
│   │   │   └── core/     # DI・ルーティング・テーマ設定
│   │   ├── ios/
│   │   └── web/
│   └── backend/          # ASP.NET Core API
│       ├── src/
│       │   ├── Controllers/
│       │   ├── Services/
│       │   ├── Repositories/
│       │   └── Models/
│       ├── Dockerfile
│       └── k8s/          # Kubernetes マニフェスト
└── docs/
```

## フロントエンド レイヤー構成

```
UI (Widget)
  └── ViewModel (Riverpod Provider)
        └── Repository
              └── API Client (HTTP / ASP.NET Core API)
```

| 層 | 役割 |
|----|------|
| UI | Flutter Widget。状態の表示とユーザー操作の受け取り |
| ViewModel | Riverpod の `AsyncNotifier` で状態を管理 |
| Repository | API 呼び出しを抽象化し、ビジネスロジックを提供 |
| API Client | ASP.NET Core API への HTTP リクエストを担当 |

## バックエンド レイヤー構成

```
Controller (エンドポイント)
  └── Service (ビジネスロジック)
        └── Repository (DB アクセス)
              └── PostgreSQL
```

## インフラ構成（Kubernetes）

| リソース | 内容 |
|----------|------|
| Deployment | ASP.NET Core API Pod |
| Deployment | PostgreSQL Pod |
| Service | API への内部・外部ルーティング |
| PersistentVolumeClaim | PostgreSQL データの永続化 |
| ConfigMap / Secret | 環境変数・接続文字列の管理 |
