# アーキテクチャ概要

## 技術スタック

| 項目 | 技術 |
|------|------|
| フレームワーク | Flutter |
| 対応プラットフォーム | iOS / Web |
| 状態管理 | Riverpod |
| ローカルDB | Isar |
| パッケージ管理 | pub.dev |

## ディレクトリ構成

```
remembook/
├── apps/
│   └── mobile/               # Flutterアプリ
│       ├── lib/
│       │   ├── main.dart
│       │   ├── features/     # 機能ごとのモジュール
│       │   │   ├── book/     # 本の管理
│       │   │   └── shelf/    # 本棚（一覧表示）
│       │   ├── shared/       # 共通コンポーネント・ユーティリティ
│       │   └── core/         # DI・ルーティング・テーマ設定
│       ├── ios/
│       └── web/
└── docs/                     # ドキュメント
```

## レイヤー構成

```
UI (Widget)
  └── ViewModel (Riverpod Provider)
        └── Repository
              └── DataSource (Isar / API)
```

- **UI層**: Flutter Widget。状態の表示とユーザー操作の受け取りのみ行う
- **ViewModel層**: Riverpod の `AsyncNotifier` / `StateNotifier` で状態を管理
- **Repository層**: データソースを抽象化し、UI側にビジネスロジックを提供
- **DataSource層**: Isar（ローカル）または外部APIとの通信を担当

## プラットフォーム対応方針

- **iOS**: ネイティブのルック＆フィールに近いUI（Cupertino ウィジェット活用）
- **Web**: レスポンシブレイアウト対応、デスクトップブラウザでも快適に使用できる幅設計
