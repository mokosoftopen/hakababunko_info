# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a static website for "墓場文庫｜最新情報ポータル" (Hakaba Bunko Latest Information Portal) - a Japanese information portal that displays categorized X/Twitter posts as content cards. The site loads `data.json` from the same directory and displays it in a filterable grid layout.

## Architecture

### Core Files
- `index.html` - トップページ（一覧・カレンダー）。HTML/CSS/JS を1ファイルに持つ
- `share.html` - 「予定を画像にする」ページ（訪問者向け）。予定を選んで X 投稿用の 16:9 画像を作る
- `data.json` - 掲載データ
- `CNAME` - GitHub Pages custom domain configuration
- `logo.png` - 仮の画像（「Logo」と書かれているだけ。使っていない）
- `ogp.jpg` - Open Graph Protocol image for social sharing

### Key Features
- **Data Source**: `data.json`（取得に失敗したときだけ `index.html` 内の `FALLBACK`）
- **Content Types**: 6 categories (goods, books, events, web articles, news, sale info)
- **Twitter Integration**: Embedded tweets using Twitter Widget API
- **トップの操作**: 期間タブ（いま開催中／これから／おわったもの）、カテゴリのチップ、並び順の select。
  URL クエリ `?status=current|upcoming|ended` `?category=...` `?sort=desc|asc` で初期状態を指定できる
- **右下のボタン**: 「予定を画像にする」（share.html へ）とカレンダー。カレンダー画面の
  「この月を画像にする」は `share.html?kind=calendar&month=YYYY-MM` を開く
- **Responsive Design**: Mobile-optimized layout with CSS custom properties

### JavaScript Architecture（index.html）
- **Data Management**: `fetchData()` で `data.json` を読む
- **Rendering**: `card()` function generates content cards with embedded tweets
- **State Management**: Global variables for active filters (`active`, `sortOrder`, `statusFilter`)
- **Event Handling**: タブ・チップ・select の操作で `render()` が描き直す

### 予定を画像にする（share.html）
- 画像は 1200×675 の DOM（`#stage`）を html-to-image@1.11.11 で 2400×1350 の PNG にする
- 型は リスト（サムネイル付き・最大5件）／カレンダー（1か月）／画像タイル（4〜8件）
- 初期選択は「期間内に start_date が始まる行」。候補一覧は新しい順（`date` 降順）で、
  並び替え・キーワード検索・「えらんだ予定だけ表示する」・「チェックをすべて外す」がある
- 文字（見出し・ラベル）の編集は `?edit=1` のときだけ出す（公式風の偽画像を簡単に作らせないため）
- URL パラメータ: `kind=list|calendar|tile`、`month=YYYY-MM`、`edit=1`
- フォントは画像に使う文字だけを Google Fonts の `text=` で取り寄せて埋め込む（約36KB）
- iOS / Safari では書き出し前に空打ちする（html-to-image の WebKit の既知の問題対策）
- サムネイルは CORS 許可のある画像だけ使える（pbs.twimg.com は可）。読めないものは種別色のブロックになる

### CSS Architecture
- **Design System**: CSS custom properties for colors and theming
- **Layout**: CSS Grid for responsive card layout
- **Components**: Modular CSS classes for cards, chips, badges, and controls

## Development Workflow

### No Build Process
This is a static HTML site with no build pipeline, package.json, or dependencies. All development is done directly in `index.html` / `share.html`.

### Local Development
- Serve the site using any static file server (e.g., `python -m http.server`, `live-server`)
- No compilation or build steps required

### Content Management
- 掲載データは **`data.json`**（このリポジトリ内）。`index.html` が同じ場所から読み込む
- 取得に失敗した場合のみ `index.html` 内の `FALLBACK` 配列が使われる
- 項目: `url`, `status`, `type`, `thumbnail`, `media_type`, `date`, `start_date`, `end_date`, `title`
- 旧データ源の Google スプレッドシート（`1oFtEcREkdS2MiUpZw0CmIioTZ0lUcXQ4PIEKOpetjs4`）は
  2026-08-09 に `data.json` へ移行済み。参照はしていないが記録として残す

### 掲載データの更新手順（Claude Code に依頼する）

運用者が「ポータル更新して」と言ったら、以下を行う。常時動く自動化はしない。

1. `data.json` を読み、登録済みの投稿 ID を把握する
   （`url` の `status/<数字>` 部分が投稿 ID）
2. @hakababunko の X タイムラインを取得する。**リポストも含める**
   （他社の告知はリポスト経由で拾うため。リプライは除外）
   - 転載ボット側の資格情報が `/Volumes/SSD/GitHub/hakaba-mirror-nab/secrets/.env` にある
     （`X_BEARER_TOKEN` と `X_USER_ID`）
   - `GET /2/users/:id/tweets` に `expansions=referenced_tweets.id,referenced_tweets.id.author_id`
     を付けると、リポスト元の本文と投稿者が取れる
   - **必ず `since_id`（下記「前回の到達点」）を付ける。全期間の取り直しは禁止。**
     X API の課金は「返ってきた投稿の件数」で決まる（自アカウントの読み取りは
     $0.001/件、それ以外は $0.005/件。リクエスト回数では課金されない）。
     expansions で返る引用元・リポスト元も1件として数えられる。
     2026-08-11 に 8 か月ぶんを取り直して約 $1.3 を無駄にした。差分だけ取ること
   - `tweet.fields=created_at,author_id,referenced_tweets,attachments,note_tweet`、
     `expansions` に `attachments.media_keys,referenced_tweets.id.attachments.media_keys` も足し、
     `media.fields=type,url,preview_image_url`、`user.fields=username`、`exclude=replies`、`max_results=100`。
     リポスト元・引用元の画像と、長文（note_tweet）の全文が取れる
   - 取得したレスポンスは加工前にそのままファイルに保存し、判定はそのファイルから行う
     （判定をやり直すときに取り直して課金されないように）
3. 未登録のものを拾い、次を判断する
   - **載せるもの**: 告知（新情報）を含む投稿。リポストは**リポスト元の投稿**の URL で登録する
     （墓場文庫の短い反応つき引用は、引用元の本文で判定する）
   - **載せないもの**: 雑談・お礼・制作過程・誕生日イラスト・外国語版・在庫や品切れの案内・
     同じ告知の繰り返し（カウントダウン、「本日まで」の再掲）
   - **同じ話題は1件にまとめる**: 同じ告知を複数のアカウントが伝えている場合（公式・メディア記事・
     商品紹介の連投など）は代表1件だけ載せる。代表は、期間と画像がそろった公式（墓場文庫や
     主催者）の投稿を優先する。既に `data.json` にある話題なら追加しない。
     ただし、日付や行動が違う別の告知（例: イベント本体と事後通販）は別の行にする
   - `type`: goods / books / events / web / news / sale のどれか
   - `start_date` / `end_date`: 次のルールで入れる（2026-08-11 のコミット aa78c60・a3c4dfd で決めたもの）
     - 本文やリンク先に期間の記載があれば、それを入れる
     - 雑誌・連載の号: 終了日＝発売日（投稿日）の1か月後
     - 小説・コミックの発売: 開始日＝発売日、終了日は空（期間限定ではないため）
     - グッズで終了日が分からない: 開始日（なければ投稿日）の3か月後。投稿から1か月以上
       経っている場合は1か月後
     - イベント・お知らせ・Web記事・セールで期間の記載がない: 終了日＝投稿日
     - 開始日だけ分かる: 終了日＝開始日の1か月後
     - 書式は `YYYY/MM/DD`（ゼロ埋め）。`date` は日本時間の `YYYY/MM/DD HH:MM`
   - `thumbnail`: 投稿の画像 URL（動画は preview_image_url）。リポスト元・引用元の画像でもよい。
     画像がない場合は下の「画像の補い方」
   - `title`: 分かりやすい見出し。画像ページでは訪問者が文字を直せないので、推測の「？」などを残さない
   - `status`: `公開`
4. **追加する一覧を運用者に見せて確認を取る**。判断を含むので勝手に確定しない
5. `data.json` の末尾に追記し（既存行の並びは変えない）、下の since_id も更新して commit・push する

### 画像の補い方（thumbnail が空の行）

- **AI で生成した画像は使わない**
- X の埋め込み用エンドポイント `https://cdn.syndication.twimg.com/tweet-result?id=<投稿ID>&token=<…>`
  （X API ではないので課金されない）の `mediaDetails`（投稿の写真）か、`quoted_tweet.mediaDetails`
  （引用元の写真）を使う。token は埋め込みウィジェットと同じ計算（id/1e15×π を36進数にして 0 と . を除く）
- リンクのプレビュー画像（`pbs.twimg.com/card_img/...`）は**使わない**。時間がたつと 404 になる
- 写真がなくリンク先がある場合は、リンク先ページの `og:image`（公式の共有用画像）を使ってよい。
  Another Record（となりのヤングジャンプ）は作品の画像になる
- それでもなければ空のままにする（トップでは画像なしのカード、画像ページでは種別色のブロックになる）

### 前回の到達点（次回はここから差分を取る）

```
since_id  2111021477734854761   # 2026-10-16T09:08:35Z
```

更新するたびに、その回で取得した最新の投稿 ID をここに書き換えること。

### 注意

- `status` が `公開` 以外の行はサイトに表示されない。様子を見たい行や、重複をまとめるときに
  代表以外の行を隠すのに使う（`非公開`）。行を消すと次回の更新で同じ投稿を再登録しかねないので、
  まとめるときは削除せず `非公開` にする
- 日付が両方空の行は常に「開催中」扱いになる。終了したら手で `end_date` を入れる
- 同じ投稿 ID を二重に登録しない（`url` で照合する。`非公開` の行も登録済みとして扱う）

### Deployment
- Hosted on GitHub Pages
- Domain: configured via `CNAME` file
- Auto-deploys from main branch
- `data.json` を push すれば数分でサイトに反映される（ビルド不要）

## Key Configuration

### Google Analytics
- Measurement ID: `G-RRYLCYYG74` (configured in `index.html:26`。share.html には入れていない)

### Twitter Integration
- Uses Twitter Widget API for embedded tweets
- Theme: dark mode
- Configuration: `cards: 'hidden'` for thumbnail mode, `cards: 'visible'` for full embeds

### Content Categories
- `goods` - グッズ (yellow)
- `books` - 書籍 (blue) 
- `events` - イベント (green)
- `web` - Web記事 (red)
- `news` - お知らせ (teal)
- `sale` - セール情報 (amber)