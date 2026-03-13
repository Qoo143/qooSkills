# Agent: api-v4-analyze

> 分析舊 PHP 檔案，產出 v4 端點遷移規劃。

## 觸發方式

用戶提供一個舊 PHP 檔案路徑（通常是 `modules/*/prodb_*.php`），本 Agent 進行分析。

## 執行流程

### 1. 讀取舊檔案

- 讀取用戶指定的 `prodb_*.php`
- 找出所有 `$_POST['act']` / `$action` 的 switch-case 分支
- 記錄每個 action 的功能、參數、SQL 查詢位置

### 2. 產出端點對照表

```markdown
## 端點分析：prodb_xxx.php

| # | 舊 act 值 | 功能說明 | HTTP 方法 | v4 路由建議 | Controller 方法 |
|---|-----------|----------|-----------|-------------|-----------------|
| 1 | get_list  | 取得清單 | GET       | /xxx        | index()         |
| 2 | get_detail| 取得詳情 | GET       | /xxx/@id    | show()          |
| 3 | save      | 新增     | POST      | /xxx        | create()        |
| 4 | update    | 更新     | PUT       | /xxx/@id    | update()        |
| 5 | delete    | 刪除     | DELETE    | /xxx/@id    | destroy()       |
```

### 3. 逐端點分析（用戶選擇後展開）

對每個端點產出：

```markdown
### 端點 #1: get_list → GET /xxx

**參數分析**

| 參數 | 來源 | 型別 | 必填 | 說明 |
|------|------|------|------|------|
| user_id | SESSION | string | 自動 | 登入者 ID |
| page | query | int | 否 | 頁碼，預設 1 |
| per_page | query | int | 否 | 每頁筆數，預設 15 |

**SQL 位置**（不複製 SQL，只標行號）

| 用途 | 舊檔案行號 | 目標 Repository 方法 |
|------|------------|---------------------|
| 查詢清單 | L45-78 | findList() |
| 計算總數 | L80-85 | countList() |

**依賴分析**

| 外部依賴 | 類型 | 說明 |
|----------|------|------|
| classes/Mission.php | require_once | 可直接引入使用 |
| $_SESSION['user_data'] | Session | 已由 AuthMiddleware 驗證 |

**命名建議**

| 項目 | 建議 |
|------|------|
| Controller | XxxController |
| Service | XxxService |
| Repository | XxxRepository |
```

### 4. 確認 RESTful 設計

檢查並建議：
- URL 命名是否為 kebab-case（`/mission-types` 而非 `/missionTypes`）
- 是否使用正確的 HTTP 方法（查詢用 GET、建立用 POST、更新用 PUT/PATCH、刪除用 DELETE）
- 路由參數用 `@` 前綴（`@id`、`@sn`）
- 是否需要 ValidateQueryMiddleware / ValidateBodyMiddleware

## 輸出規範

1. 先輸出完整的端點對照表
2. 等用戶選擇要實作的端點
3. 展開該端點的詳細分析
4. 用戶確認後，建議接續使用 `/api-v4-write` 進行實作

## 注意事項

- **不複製 SQL** — 只標行號，讓用戶自行確認
- **不預設架構** — 如果舊檔案的邏輯很簡單（例如只有一個查詢），不需要強行拆三層
- **標註複雜度** — 如果某個 action 邏輯很複雜（多次 DB 查詢、外部 API 呼叫），要特別標註
- **識別共用邏輯** — 如果多個 action 共用相同的查詢或驗證，建議抽取為共用方法
