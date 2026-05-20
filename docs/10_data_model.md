# 10. データモデル

棚卸商品管理機能で扱うデータ構造を定義する。
モックでは `index.html` の `products` 配列にインメモリで保持しているが、本実装ではバックエンドAPIから取得・更新される想定。

## Product（棚卸商品マスタ）

棚卸し対象となる商品1件を表す。

| フィールド | 型 | 必須 | 説明 |
|---|---|---|---|
| `id` | number | ◯ | 商品マスタID(自動採番) |
| `name` | string | ◯ | 商品名 |
| `code` | string | - | 商品コード。インフォマート連携時に取得、書き起こし/自社加工品は空文字列の場合あり |
| `route` | enum | ◯ | 登録経路(後述) |
| `supplier` | string | ◯ | 発注先名。自社加工品は `(自社加工品)` または `-` |
| `inventoryOn` | boolean | ◯ | 棚卸ステータス。`true`=表示(棚卸対象) / `false`=非表示(対象外) |
| `orderOn` | boolean | ◯ | 発注対象フラグ。`true`=発注アプリで発注する商品 / `false`=発注対象外。**棚卸属性とは独立**(後述)。本画面では参照のみ、操作不可 |
| `category` | enum | ◯ | 費目(後述)。既存商品はAI推定値、棚卸商品の手動登録時はユーザーが選択(AI推定フォールバックなし) |
| `unit` | enum | ◯ | 単位(後述) |
| `tax` | enum | ◯ | 税区分(後述) |
| `price` | number | ◯ | 棚卸推奨単価(円)。`0` の場合は単価未設定を表す |

### 棚卸属性 / 発注属性の独立（重要）

商品は「**棚卸対象か**(`inventoryOn`)」と「**発注対象か**(`orderOn`)」の **2つの独立した属性** を持つ。
従来は発注メニュー(「発注先＆仕入先商品」)配下に棚卸関連機能を相乗りさせていたが、納品書書き起こし商品は発注先を持たず表示できない等の不整合があり、棚卸を独立した本画面に切り出した経緯がある。

| route | 既定 `orderOn` | 既定 `inventoryOn` | 備考 |
|---|---|---|---|
| `order`(発注アプリ連携) | `true` | 任意 | 唯一の発注対象 |
| `informart` / `delivery` | `false` | 既存分は現状維持(後述の移行方針) | 仕入由来。発注先を持たない場合あり |
| `in-house`(自社加工品) | `false` | `true` | 顧客が棚卸商品として作成 |

### モックでの例

```javascript
{
  id: 1,
  name: '生ビール（樽）20L',
  code: 'BEV-001',
  route: 'informart',
  supplier: 'カクヤス',
  inventoryOn: true,
  orderOn: false,
  category: '酒類仕入高',
  unit: '樽',
  tax: '外税（標準税率）',
  price: 12000
}
```

## enum: route（登録経路）

| 値 | ラベル | 表示色(`route-badge` クラス) | 説明 |
|---|---|---|---|
| `informart` | インフォマート連携 | teal (#e0f2f1 / #00695c) | インフォマート受発注からのマスタ自動同期 |
| `delivery` | 納品書書き起こし | orange (#fff3e0 / #e65100) | 納品書(FAX含む)からの書き起こし登録 |
| `order` | 発注アプリ連携 | green (#e8f5e9 / #2e7d32) | アスクル等の発注アプリ連携によるマスタ取込 |
| `in-house` | 自社加工品 | gray (#eceff1 / #455a64) | 新規追加画面から手動で登録した自社加工品 |

※モックの `routeLabels` 定数で値→ラベルを変換している(`index.html:1208-1213`)。

設計上は **5分類** を想定しているが、現状の運用ではFAX発注は「納品書書き起こし」に統合され、表向き4分類となっている(過去のFAX発注バッジは廃止)。

## enum: unit（単位）

下記から選択(編集画面・新規追加画面共通)。

```
個 / パック / 本 / 箱 / 樽 / kg / g / L / ml
```

## enum: tax（税区分）

下記から選択(編集画面・新規追加画面共通)。

```
外税（標準税率） / 外税（軽減税率） / 内税（標準税率） / 内税（軽減税率） / 非課税
```

## category（費目）— 費用設定の変動費目を参照

棚卸商品の費目。**選択肢は固定enumではなく、店舗が費用設定で設定した「変動費目」を引いてくる**。
仕入・食材は会計上の変動費であり、変動費予算へ実績を積むため、変動費目を単一マスタとして参照する。

- **選択肢のソース:** `monthly_cost_items`（月次費目マスタ＝変動費予算の費目）。本実装ではAPIで店舗ごとに動的取得。
- **保存先:** `shop_item_expenditure_types.expenditure_type`（商品×費目の割り当て）。`is_ai_generated` でAI付与/手動を区別。
  - 連携・書き起こし商品 → AI付与（`is_ai_generated=true`）
  - 自社加工品 → 新規追加画面で手動選択（`is_ai_generated=false`）
- **モック:** `categoryOptions` 定数で変動費目の例（食材費 / 酒類費 / ドリンク費 / 消耗品費 / その他変動費）を保持。実値は店舗の費用設定により異なる。

> ⚠️ **要確認（OQ-11）:** `expenditure_type`(int) が `monthly_cost_items`(変動費目) を参照するのか、別の固定enum（FD分類等）かは未確定。前者ならこの設計のまま実装可能。

## InventoryHistory（棚卸し履歴）

データ画面で表示する過去の棚卸記録。Productに従属する。

| フィールド | 型 | 説明 |
|---|---|---|
| `date` | string (YYYY-MM-DD) | 実施日 |
| `quantity` | number | 在庫数量(小数1桁) |
| `unitPrice` | number | 商品単価(円) |
| `subtotal` | number | 小計(円)。`quantity * unitPrice` |

## DeliveryHistory（納品書書き起こし履歴）

データ画面で表示する過去の納品書記録。Productに従属する。

| フィールド | 型 | 説明 |
|---|---|---|
| `date` | string (YYYY-MM-DD) | 納品日 |
| `quantity` | number | 数量(整数) |
| `unitPrice` | number | 納品単価(円) |
| `subtotalExTax` | number | 小計（税別）(円) |

## モック実装の制約

- `products` は単一配列で、店舗の概念は持たない(モックでは1店舗固定)
- 履歴データ(`InventoryHistory` / `DeliveryHistory`)はテーブルにハードコードされており、商品IDとの紐付けはない
- 本実装では店舗ID(`storeId`)を商品に付与し、店舗セレクタとの連動が必要(→ [90. 未決事項](./90_open_questions.md))

## 関連
- [01. 一覧画面](./01_screen_list.md) — フィールドの表示マッピング
- [20. 業務ルール](./20_business_rules.md) — enum値の意味と運用
