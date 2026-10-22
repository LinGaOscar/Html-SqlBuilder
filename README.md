<!-- 本檔由 /sync-docs 自動生成，請勿手動編輯——手改內容下次會被覆蓋。 -->
# Prism SQL Builder

離線 SQL Query Builder：匯入 DDL，視覺化選欄並即時產生 SQL，資料不離開本機。
![系統畫面](docs/screenshot.png)

線上試用：https://lingaoscar.github.io/Html-SqlBuilder/

## 技術棧
| 用途 | 技術 |
|------|------|
| UI 框架 | Vue 3 Global Build（`vendor/vue.global.js`） |
| 樣式 | Tailwind CSS v4（`storage/tailwind.css`） |
| ERD 圖表 | 自製 SVG 渲染器（`src/components/ErdPanel.js`） |
| DDL 解析 | 自製 Parser（Regex + 狀態機），支援 MySQL / PostgreSQL / MSSQL / Oracle |
| 持久化 | `.md` 檔（File System Access API）、IndexedDB、localStorage |

## 快速開始
```bash
npx serve .
```
以 Chrome / Edge 開啟 `http://localhost:3000`。完整開發說明見 [docs/dev.md](docs/dev.md)。

## 目錄結構
| 路徑 | 用途 |
|------|------|
| `index.html` | 主入口，依序載入各 script |
| `src/parser/` | DDL Parser |
| `src/builder/` | SELECT / JOIN / DML SQL 產生器 |
| `src/storage/` | File System API、IndexedDB、`.md` 序列化 |
| `src/components/` | Vue 3 元件 |
| `src/app.js` | Vue app 進入點 |
| `vendor/` | Vue 3（已 commit） |
| `storage/` | Tailwind 輸入與產生的 CSS |
| `doc/` | GitHub Actions 部署說明 |
| `docs/` | dev.md 與系統截圖 |
