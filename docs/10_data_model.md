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
| `unit` | enum | ◯ | 単位(後述) |
| `tax` | enum | ◯ | 税区分(後述) |
| `price` | number | ◯ | 棚卸推奨単価(円)。`0` の場合は単価未設定を表す |

### モックでの例

```javascript
{
  id: 1,
  name: '生ビール（樽）20L',
  code: 'BEV-001',
  route: 'informart',
  supplier: 'カクヤス',
  inventoryOn: true,
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
