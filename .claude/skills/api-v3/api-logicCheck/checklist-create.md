# ChecklistController - create 端點邏輯檢查

## 1. 基本資訊

| 項目 | 內容 |
|------|------|
| **端點名稱** | create |
| **HTTP 方法** | POST |
| **API 路徑** | `/checklist/create` |
| **舊程式檔案** | `modules/srl/prodb_checklists_new.php` |
| **舊程式 action** | `inputdata` (L104-126) |
| **新 API Controller** | `ChecklistController::create()` (L102-130) |
| **新 API Service** | `ChecklistService::create()` (L105-119) |
| **新 API DAO** | `ChecklistDao::create()` (L148-173) |
| **相關資料表** | `check_list_table` |
| **檢查日期** | 2026-02-11 |

---

## 2. SQL 查詢比對

### 舊程式 (prodb_checklists_new.php L107-109)
```sql
INSERT INTO `check_list_table`
    (seme, teacher_id, type, title_name, qusetion, score, public)
VALUES (:seme, :teacher_id, :type, :title_name, :qusetion, :score, :public)
```

### 新 API (ChecklistDao.php L156-158)
```sql
INSERT INTO check_list_table
    (seme, teacher_id, type, title_name, qusetion, score, public)
VALUES (:seme, :teacher_id, :type, :title_name, :qusetion, :score, :public)
```

**✅ SQL 完全一致**

---

## 3. 參數驗證比對

### 舊程式
```php
// prodb_checklists_new.php L105-116
$iTable_type = hash2string($_POST["table_type"]);  // 需要解 hash

$result->bindValue(':seme', getYearSeme(), PDO::PARAM_STR);
$result->bindValue(':teacher_id', $_POST["teacher_id"], PDO::PARAM_STR);
$result->bindValue(':type', $iTable_type, PDO::PARAM_STR);
$result->bindValue(':title_name', $_POST["title_name"], PDO::PARAM_STR);
$result->bindValue(':qusetion', $_POST["qusetion"], PDO::PARAM_STR);
$result->bindValue(':score', $_POST["score"], PDO::PARAM_STR);
$result->bindValue(':public', $_POST["public"], PDO::PARAM_STR);
```
- 無參數型別驗證
- `teacher_id` 從前端傳入（不安全）
- `type` 需要先解 hash

### 新 API
```php
// ChecklistController.php L115-121
$params = $this->validate([
    'type'       => 'required|integer|in:1,2,3,4',
    'title_name' => 'required|string',
    'questions'  => 'array',
    'scores'     => 'array',
    'public'     => 'integer|in:0,1|default:0'
]);
```

**✅ 新 API 驗證更嚴格**
- ✅ 必填欄位驗證（type, title_name）
- ✅ 型別驗證（integer, string, array）
- ✅ 值範圍限制（type: 1-4, public: 0-1）
- ✅ 預設值處理（public = 0）
- ✅ `teacher_id` 從 session 取得（更安全）

---

## 4. 業務邏輯流程比對

### 舊程式流程
1. 接收前端參數
2. `hash2string()` 解 hash 取得 type
3. 使用 `getYearSeme()` 取得當前學期
4. 直接從前端接收 `teacher_id`（不安全）
5. 插入資料庫
6. 返回 `{status: 'success'}` 或 `{status: 'fail', msg: line_number}`

```php
// prodb_checklists_new.php L104-126
$iTable_type = hash2string($_POST["table_type"]);
try {
    $result = $dbh->prepare("INSERT INTO...");
    $result->bindValue(':teacher_id', $_POST["teacher_id"], PDO::PARAM_STR);
    // ...
    $returnData['status'] = 'success';
} catch (\PDOexception $e) {
    $returnData['status'] = 'fail';
    $returnData['msg'] = __LINE__;
}
```

### 新 API 流程
1. Guard 驗證 + 教師身分驗證
2. 參數驗證（型別、範圍）
3. `getCurrentSeme()` 取得當前學期
4. 從 `request->getUserId()` 取得 teacher_id（安全）
5. **陣列轉字串**: `questions` / `scores` → `|||` 分隔的字串
6. 插入資料庫
7. 返回新增的 SN

```php
// ChecklistService.php L105-119
$data = [
    'seme' => $this->getCurrentSeme(),
    'type' => $params['type'],
    'title_name' => $params['title_name'],
    'qusetion' => $this->arrayToString($params['questions'] ?? []),
    'score' => $this->arrayToString($params['scores'] ?? []),
    'public' => $params['public'] ?? 0
];

$sn = $this->checklistDao->create($teacherId, $data);
return ['sn' => $sn];
```

**✅ 新 API 邏輯更安全**
- ⏫ `teacher_id` 從 session 取得（不從前端接收）
- ⏫ 返回新增的 SN（方便前端後續操作）
- ⏫ 支援陣列格式（前端更友好）

---

## 5. 回應格式比對

### 舊程式
```json
{
  "status": "success"
}
```
或
```json
{
  "status": "fail",
  "msg": 123
}
```
- 成功時無返回 SN
- 失敗時返回行號（不友好）

### 新 API
```json
{
  "status": "success",
  "data": {
    "sn": 123
  }
}
```
- 返回新增的 SN
- 標準化錯誤訊息

**✅ 新 API 回應更實用**

---

## 6. 錯誤處理比對

### 舊程式
```php
catch (\PDOexception $e) {
    $returnData['status'] = 'fail';
    $returnData['msg'] = __LINE__;  // 返回行號
}
```
- 錯誤訊息是行號（不直觀）
- 無法區分錯誤類型

### 新 API
- **400**: 參數錯誤（型別、範圍）
- **403**: 非教師身分
- **500**: 資料庫錯誤（統一處理）

**✅ 新 API 錯誤處理更明確**

---

## 7. 重複邏輯識別

### 7.1 陣列 ↔ 字串轉換 ✅ **已提取**
- **位置**:
  - `ChecklistService::arrayToString()` (L340-348)
  - `ChecklistService::stringToArray()` (L356-364)
- **建議**: ✅ 已提取為 Helper 方法

### 7.2 當前學期取得 ✅ **已提取**
- **位置**:
  - `ChecklistService::getCurrentSeme()` (L301-311)
- **建議**: ✅ 已提取，有 fallback

### 7.3 安全性問題識別 🔴 **重要**
- **舊程式問題**: `teacher_id` 從前端傳入
  ```php
  $result->bindValue(':teacher_id', $_POST["teacher_id"], PDO::PARAM_STR);
  ```
- **新 API 解決**: 從 session 取得
  ```php
  $teacherId = $this->request->getUserId();
  ```
- **建議**: ✅ 新 API 已修正此安全漏洞

---

## 8. 改進建議

### 8.1 安全性修正 (✅ 已改進)
**舊版問題**: `teacher_id` 從前端傳入，可被偽造

**新 API 解決**:
```php
// Controller 層
$teacherId = $this->request->getUserId();  // 從 session 取得
$result = $this->checklistService->create($teacherId, $params);
```

**優點**:
- 防止偽造他人身分新增資料
- 符合安全最佳實踐

### 8.2 返回新增 SN (✅ 已改進)
**舊版問題**: 新增成功後無返回 SN，前端無法立即獲取新資料

**新 API 解決**:
```php
$sn = $this->checklistDao->create($teacherId, $data);
return ['sn' => $sn];
```

**優點**:
- 前端可立即導航到詳情頁
- 減少一次查詢請求

### 8.3 陣列格式支援 (✅ 已改進)
**舊版問題**: 前端需自行組裝 `|||` 分隔字串

**新 API 解決**:
```php
// 前端傳入陣列格式
{
  "questions": ["問題1", "問題2"],
  "scores": ["5", "3"]
}

// Service 層轉換
'qusetion' => $this->arrayToString($params['questions'] ?? [])
```

**優點**:
- 前端代碼更簡潔
- 格式轉換由後端統一處理

---

## 9. 檢查結論

| 檢查項目 | 狀態 |
|---------|------|
| SQL 查詢一致性 | ✅ 完全一致 |
| 參數驗證 | ✅ 新 API 更嚴格（型別、範圍、預設值） |
| 業務邏輯流程 | ✅ 新 API 更安全（teacher_id 防偽造） |
| 回應格式 | ✅ 新 API 返回 SN（更實用） |
| 錯誤處理 | ✅ 新 API 更明確 |
| 重複邏輯處理 | ✅ 已提取 Helper 方法 |

### 整體評估: **✅ 邏輯一致，重大安全改進**

**重大改進**:
- 🔐 **修正安全漏洞**（teacher_id 防偽造）
- ✅ **返回新增 SN**（前端更便利）
- ✅ **支援陣列格式**（前端更友好）
- ✅ **嚴格參數驗證**（型別、範圍）

**安全性修正**:
- 🔴 舊版允許前端傳入 `teacher_id`，可能被偽造
- ✅ 新版從 session 取得，完全杜絕偽造風險
