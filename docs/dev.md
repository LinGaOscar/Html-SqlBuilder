# 開發文件（dev.md）
> 本檔由 /sync-docs 自動生成，請勿手動編輯——手改內容下次會被覆蓋。

## 環境需求
- Chrome / Edge（完整功能需 File System Access API；其他瀏覽器降級為手動匯入/匯出 `.md`）
- 任一靜態 HTTP 伺服器（Node.js 的 `npx serve`，或 VS Code Live Server）
- 無建置步驟，無資料庫，無需 `.env`

## 快速啟動
```bash
git clone https://github.com/LinGaOscar/Html-SqlBuilder.git
cd Html-SqlBuilder
npx serve .
```
以 Chrome / Edge 開啟 `http://localhost:3000`。`vendor/vue.global.js` 與 `storage/tailwind.css` 已 commit，離線可用。

> 以 `file://` 直接開啟 `index.html` 可看到畫面，但無法使用 File System Access API。

## 常用指令
| 用途 | 指令 |
|------|------|
| 啟動本機伺服器 | `npx serve .` |
| 下載 Tailwind standalone CLI（macOS Apple Silicon，已被 `.gitignore`） | `curl -sL https://github.com/tailwindlabs/tailwindcss/releases/latest/download/tailwindcss-macos-arm64 -o tailwindcss && chmod +x tailwindcss` |
| 重新產生樣式（新增 Tailwind class 後） | `./tailwindcss -i ./storage/input.css -o ./storage/tailwind.css --minify` |
| DDL Parser 手動測試（瀏覽器 console） | `window.parseDDL("CREATE TABLE users (id INT PRIMARY KEY, name VARCHAR(100))")` |

測試案例集中於 `src/parser/testcases.js`；本專案無自動化測試框架。

## 部署
push 至 `main` 後，`.github/workflows/deploy.yml` 自動將整個目錄部署至 GitHub Pages。
