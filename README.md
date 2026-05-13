# inventory-product-management

ラクミー経営管理 棚卸商品管理 UIモック

> **制作:** [@suguru789987](https://github.com/suguru789987) (PdM)
> **担当範囲:** 要件整理 / UIモック制作 / 棚卸商品マスタ管理画面のインタラクション設計
> **ツール:** Claude Code / HTML / CSS / JavaScript
> **🌐 Live Demo:** https://suguru789987.github.io/inventory-product-management/

---

## 概要

ラクミー経営管理における**棚卸商品管理**機能のUIモック。
棚卸対象の商品マスタを管理する画面一式を単独HTMLで実装し、イテレーションを高速化。

## 関連リポジトリ

- [rakumy-inventory-mapping](https://github.com/suguru789987/rakumy-inventory-mapping) — インフォマート → ラクミー 棚卸データのマッピング仕様

## ファイル

| ファイル | 内容 |
|---|---|
| `index.html` | 棚卸商品管理 UIモック本体(ヘッダー/店舗セレクタ/商品一覧/編集) |
| `docs/` | 開発者向け仕様書一式 |

## 仕様書 (docs/)

| ファイル | 内容 |
|---|---|
| [00_overview.md](./docs/00_overview.md) | 概要・想定ユーザー・画面構成・画面遷移・用語 |
| [01_screen_list.md](./docs/01_screen_list.md) | 一覧画面の仕様 |
| [02_screen_data.md](./docs/02_screen_data.md) | データ画面(閲覧)の仕様 |
| [03_screen_edit.md](./docs/03_screen_edit.md) | 編集画面の仕様 |
| [04_screen_add.md](./docs/04_screen_add.md) | 新規追加画面の仕様 |
| [10_data_model.md](./docs/10_data_model.md) | データモデル(Product/履歴)・enum定義 |
| [20_business_rules.md](./docs/20_business_rules.md) | 業務ルール(棚卸ステータス/登録経路/単価等) |
| [90_open_questions.md](./docs/90_open_questions.md) | 未決事項・実装時の論点 |

---

*本リポジトリは在職中の学習/共有目的で公開しており、顧客情報・本番データは含みません。*
