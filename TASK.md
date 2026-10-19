# モデルとデータベース

## 背景

会員向けアプリに「商品レビュー」を載せることになりました。まずはレビューを保存するモデルと、会員がレビューを書くAPIを作ります（アプリの画面は別のチームが作ります）。

## やること

### 1. Review モデル

レビューのテーブル（`reviews`）とモデル（`Review`）を作る。

| 列 | 内容 |
| --- | --- |
| `product_id` | 商品（必須、外部キー） |
| `member_id` | 書いた会員（必須、外部キー） |
| `rating` | 評価。1〜5の整数（必須） |
| `body` | 本文（なくてもよい。1000文字まで） |
| `created_at` / `updated_at` | |

ルール:

- 同じ会員は、同じ商品に1つだけレビューを書ける（モデルのバリデーションと、DBの一意の制約の両方で守る）
- **購入した商品だけ** レビューを書ける（キャンセル以外の注文にその商品がある）。満たさないときは `errors[:base]` に `購入した商品だけレビューできます`
- `Product` と `Member` から `reviews` で関連をたどれる。商品や会員を削除したら、レビューも削除する
- `Product#average_rating`: 評価の平均を小数第1位で四捨五入した数値。レビューがなければ `nil`

### 2. 会員向けAPI

- `POST /api/v1/products/:product_id/reviews`（`rating`、`body`）でレビューを書ける
  - 書けたら 201 で `{ "review": { "id", "product_id", "rating", "body", "created_at" } }`
  - ルールに合わなければ 422 で `{ "errors": { "rating": ["…"], "base": ["…"] } }`（項目ごとのメッセージの配列）
  - トークンがなければ 401、公開していない商品は 404
- 商品の詳細（`GET /api/v1/products/:id`）に、`average_rating` と `reviews_count` を追加する

## 受け入れ条件

- [ ] マイグレーションでテーブルを作り、`db/schema.rb` も commit している
- [ ] 上のルールをバリデーションとDBの制約で守っている
- [ ] 会員向けAPIでレビューを書け、商品の詳細に平均と件数が出る
- [ ] 既存のテストがすべて通る

## 参考

- テスト: `spec/tasks/reviews_task_spec.rb`
- Rails ガイド: Active Record バリデーション https://railsguides.jp/active_record_validations.html 、関連付け https://railsguides.jp/association_basics.html

---

課題で用意したテストと `.github/` は変更しないでください。変更したPRは判定の対象外になります。
