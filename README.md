# Prism SQL Builder

離線 SQL Query Builder。用 Chrome / Edge 開啟 `index.html`，匯入 DDL，視覺化選欄並即時產生 SQL。

**[線上試用 →](https://lingaoscar.github.io/Prism-SQL-Builder/)**

[![Deploy](https://github.com/LinGaOscar/Prism-SQL-Builder/actions/workflows/deploy.yml/badge.svg)](https://github.com/LinGaOscar/Prism-SQL-Builder/actions/workflows/deploy.yml)

---

## 特色

- **零安裝、零建置**：clone 後直接開啟 `index.html`，無需任何編譯步驟
- **完全離線**：所有邏輯在瀏覽器本地執行，資料不外傳
- **多方言支援**：MySQL / PostgreSQL / MSSQL / Oracle
- **檔案式儲存**：以 `.md` 保存 DDL 與查詢設定，適合搬移與版控
- **深淺色模式**：可切換深色 / 淺色

---

## 功能

| 功能 | 說明 |
|------|------|
| DDL 解析 | 支援 MySQL / PostgreSQL / MSSQL / Oracle `CREATE TABLE`，含 `[bracket]` 與 `schema.table` |
| SELECT 查詢 | 欄位勾選、WHERE 條件、ORDER BY、LIMIT / OFFSET |
| JOIN 查詢 | FK 自動推薦、INNER / LEFT / RIGHT JOIN、欄位衝突自動加 `table.column` 前綴 |
| DML 模板 | INSERT / UPDATE / DELETE，支援 named（`:col`）與 positional（`?`）佔位符 |
| ERD 關聯圖 | 自製 SVG 渲染器，點擊節點跳至查詢設定 |
| 查詢管理 | 同一 Schema 下可儲存多組查詢，命名後一鍵還原 |
| 查詢歷史 | 複製 SQL 時自動記錄，最多 50 筆，可重新載入 |

---

## 使用方式

### 支援環境

建議使用 **Chrome 或 Edge**（File System Access API 所需）。其他瀏覽器可正常使用，但儲存功能降級為手動匯入/匯出 `.md` 檔。

### 開啟方式

```bash
# clone 後，用 VS Code Live Server 或任意 HTTP 伺服器開啟
npx serve .
# 接著用 Chrome / Edge 開啟 http://localhost:3000
```

> 直接雙擊 `index.html`（`file://` 協定）無法使用 File System Access API。需透過 HTTP 伺服器開啟。

---

## 操作說明

### 歡迎畫面

啟動後顯示歡迎畫面，提供四種進入方式：

| 選項 | 說明 |
|------|------|
| 開啟 Schema 檔案 | 載入現有 `.md` 檔，後續直接覆寫儲存（Chrome/Edge） |
| 選擇儲存位置 | 指定資料夾，新建 Schema，往後自動寫入（Chrome/Edge） |
| 電商系統 / 部落格系統 | 內建範例，免匯入直接試用 |
| 空白開始，不儲存 | 不設定儲存，手動匯出 |

> 使用 Firefox 等其他瀏覽器時，「開啟 Schema 檔案」會自動降級為 file input 匯入；「選擇儲存位置」選項不顯示。

---

### 匯入 DDL

Header 右側提供三個入口：

- **開啟**：從 `.md` 檔還原完整工作狀態（含已儲存查詢）
- **儲存 / 另存新檔**：將目前 DDL + 查詢設定寫入 `.md` 檔
- **匯入 .sql / 追加 .sql**：載入原始 DDL 檔案；已有資料表時改為追加模式（依資料表名去重）

匯入 `.sql` 後系統自動偵測方言（MySQL / PostgreSQL / MSSQL / Oracle）並同步選擇器。

---

### SELECT 查詢

1. 左側列表選擇資料表
2. 右側欄位清單勾選欄位（支援全選 / 清除）
3. SQL 即時顯示於右側預覽區
4. 點擊「**格式化**」將 SELECT / WHERE / ORDER BY 展開為多行
5. 點擊「**複製**」複製 SQL 並記錄至 History

**方言選擇**：SQL 預覽工具列提供 MySQL / PG / MSSQL / Oracle 切換，影響 LIMIT / OFFSET 分頁語法。

---

### WHERE 條件

- 點擊「**+ 新增條件**」加入一列
- 支援運算子：`=`、`!=`、`>`、`>=`、`<`、`<=`、`LIKE`、`IN`、`IS NULL`、`IS NOT NULL`
- `IS NULL` / `IS NOT NULL` 不需輸入值；`IN` 的格式為 `('a','b')`
- 多條件以 `AND` 串接

---

### ORDER BY / LIMIT / OFFSET

- 點擊「**+ 新增排序**」可加入多欄排序，每欄可選 ASC / DESC
- LIMIT 輸入 `0` 表示不限；OFFSET 配合 LIMIT 使用

> **MSSQL 注意**：OFFSET/FETCH 語法必須有 ORDER BY，請確保至少設定一個排序欄位。

---

### JOIN 多表查詢

1. 勾選「**啟用 JOIN 多表查詢**」
2. 欄位區出現提示框，引導至右側「JOIN 設定」
3. FK 自動推薦依解析出的外鍵顯示建議，點「+ 加入」一鍵套用
4. 亦可從「手動加入」下拉選擇無 FK 的資料表，手動填寫 ON 條件
5. 已加入的 JOIN 可修改類型（INNER / LEFT / RIGHT）或刪除
6. 欄位區切換為分組顯示，標示 FK badge；可全選 / 清除各表或一鍵清除全部

---

### DML 模板

切換至 **DML** 頁籤後（需先在 SELECT 頁籤選擇資料表）：

- 選擇模式：**INSERT** / **UPDATE** / **DELETE**
- 選擇佔位符風格：
  - `:name`（named）— 適用 PDO、SQLAlchemy
  - `?`（positional）— 適用 JDBC、mysql2
- 模板即時產生，點「**複製**」複製至剪貼簿

---

### ERD 關聯圖

切換至 **ERD** 頁籤：

- 顯示所有資料表方塊與 FK 關係連線
- **點擊資料表方塊**可跳至該資料表的查詢介面
- 深淺色主題切換時自動重繪

---

### 儲存查詢

在 SELECT 頁籤上方輸入查詢名稱後點「**+ 儲存查詢**」：

- 儲存目前的欄位選取、WHERE、JOIN、ORDER BY、LIMIT 等完整狀態
- 已儲存的查詢以 pill 顯示，點擊名稱即還原，點 ✕ 刪除
- 名稱重複時會提示警告，不允許覆蓋

---

### Query History

切換至 **History** 頁籤：

- 每次複製 SQL 後自動記錄，最多保留 50 筆
- 每筆顯示方言、資料表名、前 3 行 SQL 預覽、時間戳
- 可「載入」還原設定或「複製 SQL」再次取用
- 「清除全部」需二次確認

> 線上試用版（GitHub Pages）不支援此功能，History 頁籤顯示提示。

---

### 重設

Header 右側「**重設**」按鈕（載入 Schema 後才顯示）：

- 清除目前所有 Schema、查詢設定與已儲存查詢
- 返回歡迎畫面，可重新匯入不同的 Schema

---

## 資料儲存

| 資料 | 儲存位置 | 說明 |
|------|---------|------|
| DDL + savedQueries | `.md` 檔（FSA 或手動匯出） | 主要業務資料 |
| 目錄 handle | IndexedDB | 跨 session 記憶儲存目錄 |
| Query History | localStorage | 暫態，最多 50 筆 |
| 主題偏好 | localStorage | 允許 localStorage |

**自動儲存**：DDL 或查詢清單變更後 1.5 秒寫入（需先設定儲存目錄）。

`.md` 格式人類可讀，適合版控，換電腦複製檔案即可還原。

---

## DDL 支援重點

支援常見 `CREATE TABLE` 寫法：

```sql
CREATE TABLE users (
  id INT PRIMARY KEY,
  name VARCHAR(100) NOT NULL
);

CREATE TABLE [dbo].[orders] (
  id INT PRIMARY KEY,
  user_id INT,
  FOREIGN KEY (user_id) REFERENCES [dbo].[users](id)
);
```

已涵蓋：

- 反引號、雙引號、MSSQL 方括號識別符
- `CREATE TABLE IF NOT EXISTS`
- `schema.table` / `[schema].[table]`
- 行內與表級 `PRIMARY KEY`、`FOREIGN KEY`
- `ALTER TABLE ... ADD CONSTRAINT ... PRIMARY KEY / FOREIGN KEY`
- `AUTO_INCREMENT`、`SERIAL`
- `DEFAULT (...)`
- 跳過 `UNIQUE`、`INDEX`、`KEY` 表級定義

---

## 專案結構

```
index.html          # 主入口，直接開啟即可用
vendor/
  vue.global.js     # Vue 3（已 commit，無需下載）
storage/
  input.css         # Tailwind 原始輸入
  tailwind.css      # 已產生並 commit，無需重新產生
src/
  parser/           # DDL Parser（Regex + 狀態機）
  builder/          # SQL Builder（SELECT / JOIN / DML）
  storage/          # File System API / IndexedDB / .md 格式
  components/       # Vue 3 元件
  app.js            # 應用程式入口
```

---

## CI / CD

push 至 `main` branch，GitHub Actions 直接部署整個目錄至 GitHub Pages，無建置步驟。

---

## 技術棧

| 用途 | 技術 |
|------|------|
| UI 框架 | Vue 3（`vendor/vue.global.js`，已 commit） |
| 樣式 | Tailwind CSS v4（`storage/tailwind.css`，已 commit） |
| ERD 圖表 | 自製輕量 SVG 渲染器（`src/components/ErdPanel.js`） |
| DDL 解析 | 自製 Parser（Regex + 狀態機） |

---

## 已知限制

- File System Access API 需要 Chrome / Edge；其他瀏覽器降級為手動匯入/匯出。
- DDL parser 以常見 `CREATE TABLE` 為主，非標準或高度方言化語法可能需要先簡化後匯入。
