# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a static website for "墓場文庫｜最新情報ポータル" (Hakaba Bunko Latest Information Portal) - a Japanese information portal that displays categorized X/Twitter posts as content cards. The site fetches data from Google Sheets and displays it in a filterable grid layout.

## Architecture

### Core Files
- `index.html` - Single-page application containing all HTML, CSS, and JavaScript
- `CNAME` - GitHub Pages custom domain configuration
- `logo.png` - Site logo image
- `ogp.jpg` - Open Graph Protocol image for social sharing

### Key Features
- **Data Source**: Google Sheets integration via CSV export API
- **Content Types**: 6 categories (goods, books, events, web articles, news, sale info)
- **Twitter Integration**: Embedded tweets using Twitter Widget API
- **Filtering**: Category-based filtering, date sorting, and status-based filtering
- **Responsive Design**: Mobile-optimized layout with CSS custom properties

### JavaScript Architecture
- **Data Management**: `fetchGoogleSheet()` function handles Google Sheets API calls
- **Rendering**: `card()` function generates content cards with embedded tweets
- **State Management**: Global variables for active filters (`active`, `sortOrder`, `statusFilter`)
- **Event Handling**: Filter buttons and sorting controls update display in real-time

### CSS Architecture
- **Design System**: CSS custom properties for colors and theming
- **Layout**: CSS Grid for responsive card layout
- **Components**: Modular CSS classes for cards, chips, badges, and controls

## Development Workflow

### No Build Process
This is a static HTML site with no build pipeline, package.json, or dependencies. All development is done directly in the `index.html` file.

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
   - 転載ボット側の資格情報が `/Volumes/SSD2/github/hakababunko_mirror/.env` にある
   - `GET /2/users/:id/tweets` に `expansions=referenced_tweets.id,referenced_tweets.id.author_id`
     を付けると、リポスト元の本文と投稿者が取れる
   - **必ず `since_id`（下記「前回の到達点」）を付ける。全期間の取り直しは禁止。**
     X API の課金は「返ってきた投稿の件数」で決まる（自アカウントの読み取りは
     $0.001/件、それ以外は $0.005/件。リクエスト回数では課金されない）。
     expansions で返る引用元・リポスト元も1件として数えられる。
     2026-08-11 に 8 か月ぶんを取り直して約 $1.3 を無駄にした。差分だけ取ること
3. 未登録のものを拾い、次を判断する
   - `type`: goods / books / events / web / news / sale のどれか
   - `start_date` / `end_date`: 本文に明記があれば入れる。**分からなければ空**にする
     （空の行は「開催中」として表示され続ける。誤った日付で早く消えるより安全）
   - `thumbnail`: 投稿の画像 URL（動画は preview_image_url）
   - `title`: 任意。分かりやすい見出しがあれば
   - `status`: `公開`
4. **追加する一覧を運用者に見せて確認を取る**。判断を含むので勝手に確定しない
5. `data.json` に追記して commit・push する

### 前回の到達点（次回はここから差分を取る）

```
since_id  2086418400579137609   # 2026-08-09T11:44:45Z
```

更新するたびに、その回で取得した最新の投稿 ID をここに書き換えること。

### 注意

- `status` が `公開` 以外の行はサイトに表示されない。様子を見たい行に使える
- 日付が両方空の行は常に「開催中」扱いになる。終了したら手で `end_date` を入れる
- 同じ投稿 ID を二重に登録しない（`url` で照合する）

### Deployment
- Hosted on GitHub Pages
- Domain: configured via `CNAME` file
- Auto-deploys from main branch
- `data.json` を push すれば数分でサイトに反映される（ビルド不要）

## Key Configuration

### Google Analytics
- Measurement ID: `G-RRYLCYYG74` (configured in `index.html:10`)

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