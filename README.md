# notion-widgets

GH（屋号）の Notion 埋め込みウィジェット置き場。GitHub Pages で公開し、Notion に `/embed` で貼り付けて使う。

各ウィジェットは 1機能 = 1フォルダ で管理する。フォルダ内は `index.html`（本体）と `README.md`（技術メモ）。公開URLは末尾のファイル名を省略できる（例: `.../triathlon-weekly/`）。

## ウィジェット一覧

| ウィジェット | 説明 | 公開URL |
|---|---|---|
| [triathlon-weekly](./triathlon-weekly/) | Strava の今週の Swim/Bike/Run 距離を目標と対比してドーナツリング表示。目標はウィジェット内で変更可 | https://ghmaster.github.io/notion-widgets/triathlon-weekly/ |
| [condition](./condition/) | intervals.icu（Garmin）のコンディション（TSB/HRV/安静時心拍/睡眠）を追い込み判断用に表示 | https://ghmaster.github.io/notion-widgets/condition/ |

各ウィジェットの技術詳細・依存リソース・更新手順は、それぞれのフォルダ内 `README.md` を参照。

## 共通の約束ごと

- APIキー・トークン類は一切このリポジトリに置かない（Cloudflare の Secret に格納）
- Notion 埋め込みはページを開いた時に一度読み込むだけなので、各ウィジェットは定期 fetch と `visibilitychange` による再取得で自動更新している
- Notion 側でウィジェットを最新化したい時は、埋め込みURL末尾の `?v=<数字>` を上げる
- バックエンドはいずれも Cloudflare Workers（+必要に応じて KV）。データ源は各ウィジェットで異なる（Strava/Notion、intervals.icu など）
