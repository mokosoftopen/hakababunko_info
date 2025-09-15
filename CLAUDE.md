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
- Content is managed via Google Sheets (spreadsheet ID: `1oFtEcREkdS2MiUpZw0CmIioTZ0lUcXQ4PIEKOpetjs4`)
- Fallback data is hardcoded in the `FALLBACK` array within `index.html:135-142`
- Content fields: `type`, `url`, `status`, `thumbnail`, `title`, `media_type`, `date`, `start_date`, `end_date`

### Deployment
- Hosted on GitHub Pages
- Domain: configured via `CNAME` file
- Auto-deploys from main branch

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