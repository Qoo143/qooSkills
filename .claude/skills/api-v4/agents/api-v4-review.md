# Agent: api-v4-review

> 審閱 v4 API 程式碼，檢查架構規範、安全性與一致性。

## 觸發方式

- 用戶完成端點實作後，要求審閱
- 用戶指定一組 Controller/Service/Repository 檔案進行審閱
- 定期對整個 v4 目錄進行全面審閱

## 審閱清單

### 1. 分層架構 ✅

| 檢查項目 | 通過條件 |
|----------|----------|
| Controller 不含 SQL | 沒有 `prepare()`、`execute()`、`SELECT`、`INSERT` 等 |
| Controller 不含業務邏輯 | 只有：取參數 → 呼叫 Service → 回傳結果 |
| Service 不碰 HTTP | 沒有 `$this->app->request()`、`$_GET`、`$_POST`（`$_SESSION` 例外） |
| Repository 不處理例外 | 沒有 try-catch，讓 PDOException 冒泡 |
| Repository 不做資料轉換 | 回傳原始 SQL 結果，轉換在 Service 做 |

### 2. 安全性 🔒

| 檢查項目 | 通過條件 |
|----------|----------|
| SQL 注入防護 | 所有 SQL 都用 prepared statement + `bindValue` |
| 權限檢查 | 所有查詢都帶 `user_id` 條件（防止越權存取） |
| AuthMiddleware | 需要認證的路由都在 AuthMiddleware group 內 |
| 輸入驗證 | 必要參數有 ValidateQueryMiddleware 或 Service 層驗證 |
| Read-before-Write | UPDATE/DELETE 操作前有先查詢確認資料存在且歸屬正確 |

### 3. 一致性 🎯

| 檢查項目 | 通過條件 |
|----------|----------|
| 回應格式 | 統一用 `$this->app->success()` / `$this->app->fail()` |
| 命名規範 | Controller/Service/Repository 後綴正確，方法名 camelCase |
| 路由風格 | kebab-case、RESTful 語義正確 |
| 依賴注入 | 透過建構子注入 Engine，內部 new Service/Repository |
| 檔案頭部 | 有來源註解（`來源: modules/xxx/prodb_xxx.php`） |
| SQL 來源 | Repository 方法 docblock 有標記原始 SQL 行號 |

### 4. 資料庫連線 💾

| 檢查項目 | 通過條件 |
|----------|----------|
| 讀寫分離 | SELECT 用 `dbSlave()`，INSERT/UPDATE/DELETE 用 `db()` |
| 不建新連線 | 沒有 `new PDO()` 或 `new SimplePdo()` |
| 參數綁定型別 | INT 用 `PARAM_INT`、STR 用 `PARAM_STR` |

### 5. 路由配置 🛤️

| 檢查項目 | 通過條件 |
|----------|----------|
| 路由在正確 group | 需認證的在 AuthMiddleware group 內 |
| use 語句完整 | routes.php 頂部有引入對應的 Controller 和 Middleware |
| 參數正則 | URL 參數有適當的正則限制（如 `@id:[0-9]+`） |

## 輸出格式

```markdown
## 審閱報告：XxxController

### 總結
- ✅ 通過: X 項
- ⚠️ 建議: X 項
- ❌ 必修: X 項

### 詳細

#### ✅ 通過
- 分層架構正確
- SQL 注入防護完整
- ...

#### ⚠️ 建議改善
1. **[一致性]** CalendarService L45: 建議統一用 `\Exception` 而非自定義 Exception
   - 現況: `throw new CustomException(...)`
   - 建議: `throw new \Exception('訊息', 400)`

#### ❌ 必須修正
1. **[安全性]** XxxRepository L23: SQL 缺少 `user_id` 條件
   - 風險: 可能導致越權存取其他使用者的資料
   - 修正: 在 WHERE 加入 `AND user_id = :user_id`
```

## 全面審閱模式

當用戶要求對整個 v4 目錄審閱時：

1. 掃描 `ADLAPI/v4/src/App/` 下所有檔案
2. 逐一檢查每組 Controller/Service/Repository
3. 檢查 routes.php 路由完整性
4. 產出總結報告 + 各檔案詳細報告

## 注意事項

- **只報告實際問題** — 不報告風格偏好（如單引號 vs 雙引號）
- **給出具體修正建議** — 不只說「有問題」，要說怎麼改
- **區分嚴重程度** — 安全問題 ❌ > 架構偏離 ⚠️ > 風格建議 💡
- **對比現有模式** — 以 CalendarController 等已完成的檔案為標準
