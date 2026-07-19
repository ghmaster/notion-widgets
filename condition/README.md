# condition — Condition Dashboard

intervals.icu（Garmin由来）のコンディション指標を、追い込み判断用に表示するNotion埋め込みウィジェット。TSB（調子）/ HRV（回復）/ 安静時心拍 / 睡眠 の4指標を、ステータス＋バーで一目で読める。

- 公開URL: https://ghmaster.github.io/notion-widgets/condition/
- Notion 埋め込み時は末尾に `?v=<数字>` を付けてキャッシュ回避。更新のたびに数字を上げる。

## 表示仕様

- 4指標を横並び（PC）/ 画面幅400px以下で2×2に自動折り返し（スマホ）
- 基本は白黒ミニマル。status が caution の指標だけ琥珀色（#EF9F27）で強調
- 各指標にアイコン: TSB=gauge / HRV=activity / 安静時心拍=heart / 睡眠=moon（Lucide）
- 各指標の下に小さなバー:
  - TSB・睡眠 = 絶対スケール（TSBは0基準線、睡眠は7h目標線）
  - HRV・安静時心拍 = 7日平均を中央基準にした偏差。**「右=良好 / 左=注意」に統一**（安静時心拍は「低い=良い」なので軸を反転）

## 指標の読み方

- TSB（調子）: CTL−ATL。+5〜25=回復して追い込みOK / 大きくマイナス=疲労蓄積。高すぎもフィットネス低下なので万能に高ければ良いわけではない
- HRV: 高いほど回復良好。絶対値は個人差大なので7日平均との比較で判断
- 安静時心拍: 低いほど良い。普段より高い日は疲労・体調不良のサイン
- 睡眠: 7時間以上が目安。スコアは0-100（80+良好 / 60-79普通 / 60未満不良）

## 構成（データの流れ）

```
Garmin
  → intervals.icu（睡眠・HRV・安静時心拍・CTL/ATL を保持）
  → Cloudflare Worker「condition-api」（直近を取得・7日平均を計算）
  → ウィジェット（このリポジトリ / GitHub Pages）が取得して表示
```

## 依存する外部リソース

| 項目 | 値 |
|---|---|
| Cloudflare Worker 名 | `condition-api` |
| Worker URL | `https://condition-api.keisuke322.workers.dev` |
| Worker 環境変数 | `ICU_API_KEY`（Secret。intervals.icu のAPIキー。値は Cloudflare 内のみ。**コード・READMEに書かない**） |
| intervals.icu Athlete ID | `i569366`（コードに直接埋め込み） |
| intervals.icu API | Basic認証（ユーザー名 `API_KEY` / パスワードにAPIキー）。エンドポイント `GET /api/v1/athlete/{id}/wellness?oldest=&newest=` |

## Worker API 仕様

- `GET` のみ。直近10日の wellness を取得し、各指標の最新値＋HRV/安静時心拍の7日平均を返す
- 当日データが未同期でも、値のある最新日を採用するので欠損しない
- レスポンス例:
```json
{"tsb":{"value":10.2,"status":"good"},
 "hrv":{"value":44,"avg":41,"status":"good"},
 "restingHr":{"value":47,"avg":44,"status":"good"},
 "sleep":{"hours":5.5,"score":68,"status":"caution"},
 "weight":68,"updated":"2026-07-19"}
```

## ステータス判定ロジック

- TSB: >=5 good / >=-10 ok / それ未満 caution
- HRV: 7日平均以上 good / 平均の90%以上 ok / それ未満 caution
- 安静時心拍: 7日平均以下 good / 平均の105%以内 ok / それ超 caution
- 睡眠: 7h以上 good / 6h以上 ok / それ未満 caution

## 更新手順

- 見た目・挙動を変える → このフォルダの `index.html` を上書き → Notion 埋め込みURLの `?v=` を上げる
- 取得ロジック・判定閾値を変える → Cloudflare の Worker `condition-api` を Edit code → Deploy
