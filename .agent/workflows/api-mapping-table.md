---
description: 建立或補全 API-V3 對照表
---

# Workflow: 建立或補全 API-V3 對照表

本工作流程用於分析舊 PHP 檔案，建立詳細的 API 端點對照表。

## 目的

為每個舊 PHP 檔案建立獨立的對照表，記錄：
1. 舊檔案中所有的功能端點（若是程序式程式碼，則為功能區塊）
2. 每個功能是否要轉換為 API-V3
3. 對應的 Controller 方法名稱
4. 實作狀態（待處理、進行中、已完成）
5. 是寫入現有 Controller 還是新建 Controller

## 前置條件

- 已了解 API-V3 架構（可參考 `.agent/SKILL/api-v3/SKILL.md`）
- 已識別需要轉換的舊 PHP 檔案

## 執行步驟

### 1. 選擇舊檔案

識別需要轉換的舊 PHP 檔案，例如：
- `modules_student.php`
- `modules_teacher.php`
- `mission_api.php`

### 2. 建立對照表檔案

在 `.agent/SKILL/api-v3/api-mapping/` 目錄下建立對照表：

**命名格式**：`api-mapping-{舊檔案名稱}.md`

**範例**：
- `api-mapping-modules_student.md`
- `api-mapping-modules_teacher.md`
- `api-mapping-mission_api.md`

### 3. 對照表結構

使用以下結構建立對照表：

```markdown
# API-V3 對照表 - {舊檔案名稱}

**舊檔案路徑**：`path/to/modules_student.php`  
**最後更新**：YYYY-MM-DD  
**Controller**：TodoController / MissionController / 新建Controller

---

## 端點對照清單

| 選擇 | 舊端點（功能區塊/邏輯） | Controller 方法 | 狀態 | Controller 類型 | 說明 |
|------|-------------------|----------------|------|----------------|------|
| [ ] | Page Load (Get Mission List) | `list()` | ⏳ 待處理 | TodoController（現有） | 頁面載入時取得任務清單 |
| [x] | Action: getDetail | `detail()` | ✅ 已完成 | TodoController（現有） | 取得任務詳情 |
| [ ] | Action: create | `create()` | ⏳ 待處理 | StudentController（新建） | 新增學生資料 |
| [/] | Action: update | `update()` | 🚧 進行中 | TodoController（現有） | 更新資料 |

---

## 狀態說明

- `⏳ 待處理`：尚未開始
- `🚧 進行中`：正在開發
- `✅ 已完成`：已完成並測試

## Controller 類型說明

- **現有**：寫入已存在的 Controller（如 TodoController、MissionController）
- **新建**：需要建立新的 Controller

---

## 開發記錄

### 2026-01-30
- [x] 建立對照表
- [ ] 完成第一個端點轉換

### YYYY-MM-DD
- [ ] 記錄開發進度
```

### 4. 分析舊檔案端點

開啟舊 PHP 檔案，列出所有功能端點或邏輯區塊：

**情況 A：具名函式**
```php
function getStudentList() { ... }          // 列入對照表
```

**情況 B：程序式程式碼 (Procedural Code)**
如果檔案主要是 script 執行，請識別主要的邏輯區塊：
- **頁面載入 (Page Load)**: `SELECT * FROM ...`
- **Actions**: `if ($_POST['act'] == 'update') { ... }`
- **AJAX Handlers**: 處理 AJAX 請求的區塊

**記錄要點**：
- 功能名稱 (如果是函式寫函式名，如果是邏輯區塊寫描述，如 "Filter Logic")
- 功能說明
- 接收的參數
- 回傳的資料

### 5. 規劃 Controller 方法

根據功能決定：

#### a. 使用現有 Controller

如果功能與現有 Controller 相關：
- TodoController：待辦事項相關
- MissionController：任務相關
- AnnouncementController：公告相關

記錄為：`Controller 類型 = TodoController（現有）`

#### b. 新建 Controller

如果功能需要新的 Controller：
- StudentController：學生管理
- TeacherController：教師管理
- CourseController：課程管理

記錄為：`Controller 類型 = StudentController（新建）`

### 6. 勾選要轉換的端點

在對照表的「選擇」欄位中：
- `[ ]`：不轉換或待確認
- `[x]`：要轉換為 API

### 7. 執行轉換

根據勾選的端點，執行 `/api-write` workflow 進行轉換。
**注意**：對於程序式代碼，目標是將其邏輯提取並重構為 API 方法，而不是直接複製代碼。

### 8. 更新對照表狀態

完成轉換後，更新對照表：

**轉換前**：
```markdown
| [ ] | Page Load (Get Mission List) | `list()` | ⏳ 待處理 | TodoController（現有） | 頁面載入時取得任務清單 |
```

**轉換後**：
```markdown
| [x] | Page Load (Get Mission List) | `list()` | ✅ 已完成 | TodoController（現有） | 已完成並測試 |
```

## 完整範例

### api-mapping-modules_student.md

```markdown
# API-V3 對照表 - modules_student.php

**舊檔案路徑**：`modules/dashboard/modules_student.php`  
**最後更新**：2026-01-30  
**主要 Controller**：TodoController

---

## 端點對照清單

| 選擇 | 舊端點（功能區塊/邏輯） | Controller 方法 | 狀態 | Controller 類型 | 說明 |
|------|-------------------|----------------|------|----------------|------|
| [x] | Page Load (Student Mission List) | `list()` | ✅ 已完成 | TodoController（現有） | 學生任務清單邏輯 |
| [x] | Filter (POST parameters) | `list()` (w/ params) | ✅ 已完成 | TodoController（現有） | 支援篩選功能 |
| [x] | AJAX: getmissiondetail | `detail()` | ✅ 已完成 | TodoController（現有） | 取得任務詳情 |
| [ ] | Logic: Personal Config | `profile()` | ⏳ 待處理 | StudentController（新建） | 取得個人設定 |

---

## 開發記錄

### 2026-01-30
- [x] 建立對照表
- [x] 完成 list/detail/create 三個端點
- [/] update 端點開發中
```

## 維護規則

### 開始開發時
1. 將「選擇」欄位勾選：`[ ]` → `[x]`
2. 將「狀態」改為：`⏳ 待處理` → `🚧 進行中`

### 完成開發時
1. 將「狀態」改為：`🚧 進行中` → `✅ 已完成`
2. 在「開發記錄」中記錄完成日期

### 新增端點時
1. 在清單底部新增一行
2. 標記為 `⏳ 待處理`

## 檢查清單

- [ ] 已列出所有舊端點/邏輯區塊
- [ ] 已規劃 Controller 方法名稱
- [ ] 已標註是現有或新建 Controller
- [ ] 已勾選需要轉換的端點
- [ ] 定期更新開發狀態
- [ ] 完成後更新「開發記錄」

## 後續步驟

1. 使用 `/api-write` workflow 執行轉換
2. 使用 `/api-review` workflow 進行 code review
3. 定期回來更新此對照表

