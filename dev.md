# 開發指南

給貢獻者的簡短說明。完整架構細節見 `CLAUDE.md`，使用者操作說明見 `README.md`。

## 本機啟動

無建置步驟。需透過 HTTP 伺服器開啟（`file://` 協定無法使用 File System Access API）：

```bash
npx serve .
# 或使用 VS Code Live Server 擴充功能
```

以 Chrome / Edge 開啟對應網址，可使用完整功能（含目錄存取）；其他瀏覽器會降級為手動匯入/匯出 `.md`。

## 專案結構

```
index.html          # 主入口，載入順序見檔案內註解
vendor/vue.global.js # Vue 3 Global Build（已 commit）
storage/tailwind.css # 已產生的 Tailwind 輸出（已 commit）
src/
  parser/           # DDL Parser（Regex + 狀態機），window.parseDDL / window.detectDialect
  builder/          # SQL Builder（SELECT / JOIN / DML）
  storage/          # File System API / IndexedDB / .md 序列化
  components/       # Vue 3 元件（IIFE，window.*Component）
  app.js            # Vue app 進入點，集中管理 reactive state
```

所有模組為 IIFE、透過 `window.*` 暴露，無 ESM import／無 bundler。新增模組時記得同步在 `index.html` 的 `<script>` 標籤加入，並注意載入順序（依賴需先載入）。

## 修改樣式（Tailwind）

若新增 Tailwind utility class，需重新產生 `storage/tailwind.css`（standalone CLI 執行檔已加入 `.gitignore`，需自行下載）：

```bash
# macOS（Apple Silicon）
curl -sL https://github.com/tailwindlabs/tailwindcss/releases/latest/download/tailwindcss-macos-arm64 -o tailwindcss && chmod +x tailwindcss
./tailwindcss -i ./storage/input.css -o ./storage/tailwind.css --minify
```

產生後 commit `storage/tailwind.css` 即可。

## 測試

沒有自動化測試框架。DDL Parser 的測試案例集中在 `src/parser/testcases.js`，於瀏覽器開啟 `index.html` 後在 console 手動執行驗證：

```js
window.parseDDL(`CREATE TABLE users (id INT PRIMARY KEY, name VARCHAR(100))`)
```

功能性變更建議手動走一次：匯入 DDL → 選欄 → WHERE/JOIN/ORDER BY → 複製 SQL，確認各方言（MySQL / PostgreSQL / MSSQL / Oracle）輸出正確。

## 部署

push 到 `main` branch 後，`.github/workflows/deploy.yml` 會自動將整個目錄部署至 GitHub Pages，無建置步驟。線上版網址由 `window.location.hostname === 'lingaoscar.github.io'` 判斷並停用儲存相關功能（見 `src/app.js` 的 `isOnlineDemo`）。

## 注意事項

- `.gitignore` 對 `*.md` 採白名單制（僅 `README.md` / `CLAUDE.md` / `PLAN.md` / `doc/*.md` 不被忽略），新增其他 Markdown 檔（如本檔）需以 `git add -f` 強制加入。
- 未發現任何硬編碼的 API key / token / password；本專案不需要任何後端憑證。
