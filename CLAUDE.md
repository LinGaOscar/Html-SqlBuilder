# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 專案概述

**Prism** 是一個離線 SQL Query Builder，核心設計原則：

- 單一 HTML 目錄，零安裝、零建置步驟（vendor/ 與 tailwind.css 已 commit）
- 匯入 `.sql` DDL 檔 → 視覺化選欄 → 自動產生 SQL
- 所有運算在瀏覽器本地執行，資料不離開本機
- **完整功能需 Chrome / Edge**（File System Access API）；其他瀏覽器降級為手動匯入/匯出

## 開發環境

直接用 Chrome / Edge 開啟 `index.html`（需 HTTP 伺服器以啟用 FSA 安全上下文）：

```
# VS Code Live Server（推薦）
# 或
npx serve .
```

**無需建置步驟。** `vendor/vue.global.js` 與 `storage/tailwind.css` 已 commit 進版控，`index.html` 直接從本地相對路徑載入，完全離線可用。

### 新增 Tailwind class 時

若加入新的 Tailwind utility class，需重新產生 `storage/tailwind.css`（下載執行檔存於專案根目錄，已加入 .gitignore）：

```bash
# macOS（Apple Silicon）
curl -sL https://github.com/tailwindlabs/tailwindcss/releases/latest/download/tailwindcss-macos-arm64 -o tailwindcss && chmod +x tailwindcss
./tailwindcss -i ./storage/input.css -o ./storage/tailwind.css --minify
```

產生後 commit `storage/tailwind.css` 即可，不需其他步驟。

### 部署

push 至 main branch，GitHub Actions 自動將整個目錄部署至 GitHub Pages。

### DDL Parser 測試

`src/parser/testcases.js` 維護完整測試案例，在瀏覽器 console 執行即可驗證：

```js
// 開啟 index.html 後於 console 執行
window.parseDDL(`CREATE TABLE users (id INT PRIMARY KEY, name VARCHAR(100))`)
```

## 技術棧

| 用途 | 技術 |
|------|------|
| 響應式 UI | Vue 3 Global Build（`vendor/vue.global.js`，IIFE，`window.Vue`） |
| 樣式 | Tailwind CSS v4（`storage/tailwind.css`，standalone CLI 產生） |
| ERD 圖表 | 自製 SVG 渲染器（`ErdPanel.js`，含 MutationObserver 監聽主題切換） |
| DDL 解析 | 自製 Parser（Regex + 狀態機，IIFE） |
| 目錄持久化 | IndexedDB（`idb-handles.js`） |

所有模組皆為 IIFE，透過 `window.*` 全域暴露，無 ESM import。

> **ERD 自製原因**：Mermaid.js 打包體積約 3 MB，違背「單一自含 HTML」原則，故改用純 SVG 自繪（`ErdPanel.js`）。點擊 ERD 方塊會 emit `select-table` 事件，由 `app.js` 切換至查詢 tab。

## 架構設計

### 模組載入順序（index.html 需保持此順序）

```
Vue
  → parser/ddl-parser.js          (window.parseDDL, window.detectDialect)
  → storage/idb-handles.js        (window.idbHandles)
  → storage/file-system.js        (window.fsStorage)
  → storage/md-format.js          (window.mdFormat)
  → builder/select-builder.js     (window.buildSelect)
  → builder/join-builder.js       (window.joinBuilder)
  → builder/dml-builder.js        (window.dmlBuilder)
  → components/JoinBuilder.js     (window.JoinBuilderComponent)
  → components/DmlPanel.js        (window.DmlPanelComponent)
  → components/TablePanel.js      (window.TablePanelComponent)
  → components/SqlPreview.js      (window.SqlPreviewComponent)
  → components/ConditionBuilder.js (window.ConditionBuilderComponent)
  → components/SortLimitPanel.js  (window.SortLimitPanelComponent)
  → components/ErdPanel.js        (window.ErdPanelComponent)
  → app.js                        (Vue app.mount)
```

### 核心資料流

```
.sql 檔案匯入（追加模式，支援多檔）
    ↓ parseDDL() → TableSchema[]
Vue 3 app.js（所有 reactive state 集中管理）
    ↓ 使用者操作（欄位勾選 / 條件設定 / JOIN / 方言切換）
buildSelect() / joinBuilder.buildJoinSql() / dmlBuilder.*()
    ↓ computed sqlOutput
SQL 預覽區即時更新
```

### 狀態持久化策略

| 資料 | 儲存位置 | 說明 |
|------|---------|------|
| DDL + savedQueries | `.md` 檔（FSA 或手動匯出） | 主要業務資料 |
| 目錄 handle | IndexedDB（idb-handles.js） | 跨 session 記憶儲存目錄 |
| Query History | localStorage（`prism_query_history`） | 暫態，最多 50 筆，複製 SQL 時自動寫入 |
| 主題偏好 | localStorage（`prism_theme`） | 非業務資料，允許 localStorage |

**自動儲存**：rawDdl 或 savedQueries 變更後 1.5 秒寫入（需先設定儲存目錄）。

**savedQueries 每筆結構**：`{ name, selectedTable, selectedColumns, where, orderBy, limit, offset, joins, joinMode }`

### DDL Parser 設計

- Regex + 狀態機解析 `CREATE TABLE`，支援 MySQL / PostgreSQL / MSSQL / Oracle
- `window.parseDDL(sql)` 回傳 `TableSchema[]`；型別定義見 `src/parser/types.js`（`TableSchema`、`ColumnDef`、`ForeignKey`）
- `window.detectDialect(sql)` 分析 SQL 關鍵字並回傳方言字串，由 `handleParse` / `handleAppend` 呼叫以自動切換 `dialect` ref
- MSSQL schema-qualified 表名（`[dbo].[table]` 或 `dbo.table`）解析為 `schema.table` 格式；ALTER TABLE 語句同樣支援此格式
- `ALTER TABLE ... ADD CONSTRAINT ... PRIMARY KEY / FOREIGN KEY` 語句在主 CREATE TABLE 解析完成後後處理，補回主鍵與外鍵關係
- 追加模式：多個 `.sql` 檔可累加（依 tableName 去重，已存在的不覆蓋）
- 無 textarea 輸入，**僅支援檔案匯入**

### Toast 通知

`showToast(msg, type)` 顯示居中淡出浮動通知，2.2 秒後自動消失。`type` 可為 `'success'`（預設）、`'error'`、`'warn'`。新增任何用戶操作回饋時請呼叫此函式，勿直接操作 DOM。

### 歡迎畫面

`started` ref 控制歡迎畫面顯示（`showStart = !started && !rawDdl`）。`skipMode` ref 在使用者選擇「空白開始」時為 `true`，用於阻止 `onMounted` 的非同步 `saveDir` 還原覆蓋使用者選擇。

歡迎畫面的「開啟 Schema 檔案」和「選擇儲存位置」按鈕皆以 `v-if="fsSupported"` 控制，非 FSA 瀏覽器改顯示 file input 匯入方案。

### app.js 主要函式

| 函式 | 說明 |
|------|------|
| `handleParse()` | 解析 rawDdl，自動偵測方言，更新 tables |
| `handleAppend(ddl)` | 追加解析，依 tableName 去重 |
| `importSqlFile(event)` | 從 .sql 檔觸發解析或追加 |
| `clearAll()` | 清除所有資料並返回歡迎畫面（需 confirm） |
| `saveCurrentQuery()` | 儲存查詢至 savedQueries，重複名稱時 warn |
| `loadQuery(q)` | 還原查詢設定（含 dialect） |
| `addToHistory()` | 複製 SQL 時自動記錄 |
| `loadDemo(demo)` | 載入內建範例 Schema |
| `goToTable(name)` | ERD 點擊後切換 table 並跳回 query tab |

### SQL 方言支援

`dialect` ref 控制分頁語法差異：`mysql` / `postgresql` / `mssql` / `oracle`

方言選擇器顯示於 **SqlPreview 工具列**（與 SQL 輸出並排），並同時作為 prop 傳入所有 builder。

### SQL 格式化（SqlPreview.js）

`formatSql()` 展開多欄 SELECT、多條件 WHERE（AND）、多排序 ORDER BY，每項獨立一行。由 `pretty` ref 切換，複製內容跟隨格式化狀態。

`SqlPreview` props：`sql`、`dialect`；emits：`copy`、`update-dialect`。

### JOIN 模式（TablePanel.js）

- `joinMode && displayTables.length <= 1`：顯示虛線引導框，提示使用者至右側設定 JOIN
- `joinMode && displayTables.length > 1`：切換為垂直分組列表，顯示 FK badge（`.join-fk-badge`），顯示已選欄位數
- 一般模式：單表垂直列表

### 線上試用版限制

`isOnlineDemo = window.location.hostname === 'lingaoscar.github.io'`

- 開啟 / 儲存 / 追加按鈕顯示為 disabled
- History 頁籤顯示「試用版不支援」提示

### 儲存格式（.md）

DDL 與 savedQueries 以 code block 結構序列化，人類可讀且適合版控。`mdFormat.serialize()` / `mdFormat.deserialize()` 負責轉換。

## 關鍵邊界條件（DDL Parser）

- 複合主鍵 `PRIMARY KEY (col1, col2)`
- 行內 vs 表級約束混用
- 欄位預設值含括號，例如 `DEFAULT (CURRENT_TIMESTAMP)`
- 帶引號的識別符，例如 `` `table_name` `` 或 `"table_name"`
- ALTER TABLE 中的 schema-qualified 表名（如 `[dbo].[OrderDetail]`）：ALTER TABLE 後處理 regex 必須能匹配含空白的複雜表名
- MSSQL 的 `OFFSET...FETCH` 必須搭配 `ORDER BY`，`buildSelect` 不自動補上；呼叫端負責確保有排序欄位

## UI 慣例

- 可捲動面板（TablePanel 表格清單、欄位清單）統一使用 `max-height: max(360px, calc(100vh - 480px))` + `custom-scrollbar` class
- SqlPreview 使用 `max-height: max(300px, calc(100vh - 580px))`
- 以上皆套用於最外層容器 div，**不在內層 ul 另加捲動**
- 所有使用者操作回饋一律用 `showToast()`，不使用 `alert()` 或直接操作 DOM
- 方言選擇器（segmented buttons）固定放在 SqlPreview 工具列，不放在 Header
