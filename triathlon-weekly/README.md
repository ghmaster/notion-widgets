# notion-widgets

GH（屋号）の Notion 埋め込みウィジェット置き場。GitHub Pages で公開し、Notion に `/embed` で貼り付けて使う。

各ウィジェットはフォルダまたは単体HTMLで管理する。公開URLは `https://ghmaster.github.io/notion-widgets/<ファイルまたはパス>`。

---

## ウィジェット一覧

### triathlon-weekly.html — Triathlon Weekly Dashboard

Strava の今週（月〜日, JST）の Swim / Bike / Run 合計距離を、目標との達成度として白黒ミニマルなドーナツリングで表示する。ウィジェットをダブルクリック（PC）/ ダブルタップ（スマホ）すると設定モーダルが開き、週間目標を変更できる。

- 公開URL: https://ghmaster.github.io/notion-widgets/triathlon-weekly.html
- Notion 埋め込み時は末尾に `?v=<数字>` を付けてキャッシュ回避。更新のたびに数字を上げる。

#### 構成（データの流れ）

```
Strava
  → Zapier
  → Notion DB「Exported Strava」
  → Cloudflare Worker「triathlon-api」（今週分を集計）
  → ウィジェット（このリポジトリ / GitHub Pages）が取得して表示

目標値: ウィジェット → Worker（POST）→ Cloudflare KV に保存
```

#### 依存する外部リソース

| 項目 | 値 |
|---|---|
| Cloudflare Worker 名 | `triathlon-api` |
| Worker URL | `https://triathlon-api.keisuke322.workers.dev` |
| Worker 環境変数 | `NOTION_TOKEN`（Secret。値は Cloudflare 内のみ。**コード・READMEに書かない**） |
| Cloudflare KV namespace | `triathlon-goals` |
| KV binding 変数名 | `GOALS` |
| KV キー | `goals`（値は JSON `{swim, bike, run}`） |
| 参照 Notion DB | 「Exported Strava」 database_id = `8443a75c-246a-40bd-b306-088afa860237` |

#### Notion DB で使うプロパティ

- `Category`（formula）: `Swim` / `Bike` / `Run`（Ride・VirtualRide は Bike に統合済み）
- `Distance in K`（formula）: km 単位の距離
- `Start Date GMT from Export and UT from Zap`（date）: JST（+09:00）で格納。日付境界は JST 基準

#### Worker API 仕様

- エンドポイント形式: `POST https://api.notion.com/v1/databases/{database_id}/query`（`data_sources` ではなく `databases`。`Notion-Version: 2022-06-28`）
- `GET`  … 今週の実績 `{swim, bike, run, weekStart}` ＋ 保存済み目標 `{goals}` を返す
- `POST` … body `{swim, bike, run}` を KV に保存
- 週の定義: 月曜始まり・JST

#### 現在の目標デフォルト値

Swim 5.6 / Bike 150 / Run 40（km/週）。KV に保存があればそちらが優先される。

#### 更新手順

- ウィジェットの見た目・挙動を変える → このリポジトリの `triathlon-weekly.html` を上書き → Notion 埋め込みURLの `?v=` を上げる
- 集計ロジック・目標保存を変える → Cloudflare の Worker `triathlon-api` を Edit code → Deploy

---

## 共通メモ

- APIキー・トークン類は一切このリポジトリに置かない（Cloudflare の Secret に格納）。
- Notion 埋め込みはページを開いた時に一度読み込むだけなので、ウィジェット側で定期 fetch と `visibilitychange` による再取得を実装して自動更新している。
