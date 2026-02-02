---
description: 根據對照表執行 API-V3 撰寫任務
---

# Workflow: 根據對照表執行 API-V3 撰寫任務

本工作流程用於根據 API 對照表，系統化地撰寫 API-V3 程式碼。

## 前置條件

- 已完成 API 對照表（參考 `/api-mapping-table` workflow）
- 已閱讀 API-V3 開發技能（`.agent/SKILL/api-v3/SKILL.md`）
- 已了解資料庫結構與 Schema（`.agent/SKILL/database/SKILL.md`）
- 已分析舊功能的業務邏輯

## 執行步驟

### 1. 選擇開發項目

從對照表中選擇一個待處理（⏳）或進行中（🚧）的項目。

**建議順序**：
1. 先完成簡單的查詢 API（list, detail）
2. 再完成 CRUD 操作（create, update, delete）
3. 最後完成複雜的業務邏輯

### 2. 分析舊功能

仔細閱讀舊 PHP 檔案，記錄：
- 參數名稱、類型、是否必填、預設值
- 權限要求、資料驗證規則
- SQL 查詢語句、回傳資料格式
- 錯誤情境與訊息

### 3. 設計 API 結構

#### 3.1 確定 HTTP 方法
本專案所有 API 統一使用 **POST** 方法

#### 3.2 設計參數驗證

| 舊參數邏輯 | Validator 規則 |
|-----------|----------------|
| `if (empty($id))` | `'id' => 'required'` |
| `$page = intval($_POST['page'] ?? 1)` | `'page' => 'integer\|min:1\|default:1'` |
| `in_array($status, ['ongoing', 'expired'])` | `'status' => 'in:ongoing,expired'` |

### 4. 撰寫程式碼

按照 **DAO → Service → Controller** 的順序撰寫。

#### 4.1 撰寫 DAO

**參考**：
- 程式碼範例：`.agent/SKILL/api-v3/SKILL.md`
- 資料表結構：`.agent/SKILL/database/SKILL.md` (可查詢 Schema、Foreign Keys)

重點：
- 使用 Prepared Statements
- 讀取使用 `$dbSlave`，寫入使用 `$db`
- 不包含業務邏輯、不處理異常

#### 4.2 撰寫 Service

**參考**：`.agent/SKILL/api-v3/SKILL.md` 中的 Service 範例

重點：
- 實作業務邏輯
- Update/Delete 使用 **Read-before-Write** 模式
- 使用 `AppException` 處理業務錯誤

#### 4.3 撰寫 Controller

**參考**：`.agent/SKILL/api-v3/SKILL.md` 中的 Controller 範例

重點：
- 繼承 `BaseController`
- 使用 `guard()` 驗證 HTTP 方法和 Token
- 使用 `validate()` 驗證參數
- 使用 `success()` 回傳結果

### 5. 更新對照表

完成程式碼後，更新 `ADLAPI/v3/api-mapping.md`：

```markdown
| 舊功能 | 新 API | 狀態 | Controller | Service | DAO | 說明 |
|--------|--------|------|------------|---------|-----|------|
| `/xxx/list.php` | `POST /xxx/list` | ✅ | XxxController | XxxService | XxxDao | 已完成 |
```

### 6. 驗證與測試

使用 Postman 或 curl 測試 API：

// turbo
```powershell
# 測試 API
curl -X POST "http://localhost/v3/xxx/list" \
  -H "Content-Type: application/json" \
  -d '{"accesstoken": "your_token", "page": 1}'
```

**檢查項目**：
- [ ] HTTP 方法驗證正確
- [ ] Token 驗證正確
- [ ] 參數驗證正確
- [ ] 業務邏輯正確
- [ ] 錯誤處理正確
- [ ] 回傳格式正確

### 7. Code Review

完成後使用 `/api-review` workflow 進行 code review。

## 開發檢查清單

### DAO 層
- [ ] 使用 Prepared Statements
- [ ] 讀取使用 `$dbSlave`，寫入使用 `$db`
- [ ] 不包含業務邏輯
- [ ] 不處理異常

### Service 層
- [ ] 包含明確的業務邏輯
- [ ] Update/Delete 使用 Read-before-Write 模式
- [ ] 使用 `AppException` 處理業務錯誤
- [ ] 不使用 try-catch 捕捉 PDOException

### Controller 層
- [ ] 繼承 `BaseController`
- [ ] 使用 `guard()` 驗證
- [ ] 使用 `validate()` 驗證參數
- [ ] 不包含業務邏輯
- [ ] 使用 `success()` 回傳結果

### 命名與路由
- [ ] URL 使用 kebab-case
- [ ] Controller 名稱符合慣例
- [ ] Namespace 與檔案路徑對應

## 常見問題

### Q1: 舊功能太複雜，如何拆分？
**A**: 使用策略模式或提取為 Helper 類別，參考 `MissionTaskCounter` 的設計

### Q2: 需要呼叫多個 DAO？
**A**: 在 Service 層整合多個 DAO 的資料

## 完成後

- [ ] 更新對照表狀態為 ✅
- [ ] 提交程式碼
- [ ] 執行 `/api-review` workflow
