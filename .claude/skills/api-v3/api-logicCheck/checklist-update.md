# ChecklistController - update 端點邏輯檢查

## 1. 基本資訊

| 項目 | 內容 |
|------|------|
| **端點名稱** | update |
| **HTTP 方法** | POST |
| **API 路徑** | `/checklist/update` |
| **舊程式檔案** | `modules/srl/prodb_checklists_new.php` |
| **舊程式 action** | `updateeditdata` (L161-182) |
| **新 API Controller** | `ChecklistController::update()` (L132-158) |
| **新 API Service** | `ChecklistService::update()` (L129-159) |
| **新 API DAO** | `ChecklistDao::update()` (L175-197) |
| **相關資料表** | `check_list_table` |
| **檢查日期** | 2026-02-11 |

---

## 2. SQL 查詢比對

### 2.1 權限檢查查詢

#### 舊程式 (prodb_checklists_new.php L163-166)
```sql
SELECT * 
FROM `check_list_table` 
WHERE check_list_table_sn = :sn
```

#### 新 API (ChecklistDao.php L129-142)
```sql
SELECT check_list_table_sn as sn,
       seme,
       teacher_id,
       type,
       title_name,
       qusetion,
       score,
       public,
       is_delete
FROM check_list_table
WHERE check_list_table_sn = :sn
```

**✅ 新 API 查詢更明確**（指定欄位 vs SELECT *）

### 2.2 更新查詢

#### 舊程式 (prodb_checklists_new.php L169-173)
```sql
UPDATE `check_list_table` 
SET title_name = :title_name, 
    qusetion = :qusetion, 
    score = :score
WHERE check_list_table_sn = :sn
```

#### 新 API (ChecklistDao.php L181-186)
```sql
UPDATE check_list_table
SET title_name = :title_name,
    qusetion = :qusetion,
    score = :score
WHERE check_list_table_sn = :sn
```

**✅ SQL 完全一致**

---

## 3. 參數驗證比對

### 舊程式
```php
// prodb_checklists_new.php L161-173
$sn = $_POST["sn"];  // hash 編碼，需解碼
// 檢查鎖定狀態和擁有者
if ($isLockList['public'] == '0' && $isLockList['teacher_id'] == $vUser_data['user_id']) {
    // 執行更新
}
```
- 無參數型別驗證
- 僅當**未鎖定 AND 是擁有者**時才更新
- SN 需要解 hash

### 新 API
```php
// ChecklistController.php L143-148
$params = $this->validate([
    'sn'         => 'required|integer',
    'title_name' => 'string',
    'questions'  => 'array',
    'scores'     => 'array'
]);
```

**✅ 新 API 驗證更完善**
- ✅ 必填驗證（sn）
- ✅ 型別驗證
- ✅ 所有欄位皆為選填（可部分更新）

---

## 4. 業務邏輯流程比對

### 舊程式流程
1. 接收 `sn`（hash 編碼）
2. `hash2string($sn)` 解碼
3. 查詢檢核表資料
4. **雙重檢查**:
   - `public == '0'` (未鎖定)
   - `teacher_id == $user_id` (是擁有者)
5. 若通過，執行更新
6. 返回 `{status: 'success'}` 或 `{status: 'fail'}`

```php
// prodb_checklists_new.php L161-182
$sn = $_POST["sn"];
$isLockList = $dbh->prepare("SELECT * FROM...");
$isLockList->bindValue(":sn", hash2string($sn), PDO::PARAM_STR);
$isLockList->execute();
$isLockList = $isLockList->fetch(\PDO::FETCH_ASSOC);

if ($isLockList['public'] == '0' && $isLockList['teacher_id'] == $vUser_data['user_id']) {
    // 執行更新
}
```

### 新 API 流程
1. 接收參數（sn 為整數）
2. 通過 DAO 查詢檢核表
3. **三重檢查**:
   - 檢核表是否存在 (404)
   - teacher_id 是否匹配 (403)
   - public 是否為 0 (400 - 已鎖定不可編輯)
4. 構建更新資料（保留原值 or 使用新值）
5. 執行更新
6. 返回 `{sn: 123}`

```php
// ChecklistService.php L129-159
$checklist = $this->checklistDao->getById($sn);

if (!$checklist) {
    throw new AppException('檢核表不存在', 404);
}

if ($checklist['teacher_id'] !== $teacherId) {
    throw new AppException('無權限編輯此檢核表', 403);
}

if ($checklist['public'] == 1) {
    throw new AppException('已鎖定的檢核表不可編輯', 400);
}

$data = [
    'title_name' => $params['title_name'] ?? $checklist['title_name'],
    'qusetion' => isset($params['questions'])
        ? $this->arrayToString($params['questions'])
        : $checklist['qusetion'],
    'score' => isset($params['scores'])
        ? $this->arrayToString($params['scores'])
        : $checklist['score']
];

$this->checklistDao->update($sn, $data);
return ['sn' => $sn];
```

**✅ 新 API 邏輯更完善**
- ⏫ 明確的錯誤訊息（404, 403, 400）
- ⏫ 支援部分更新（未傳入的欄位保留原值）
- ⏫ 陣列格式支援

---

## 5. 回應格式比對

### 舊程式
```json
{
  "status": "success"
}
```
或無回應（未通過檢查時）

### 新 API
```json
{
  "status": "success",
  "data": {
    "sn": 123
  }
}
```

**✅ 新 API 回應更標準**

---

## 6. 錯誤處理比對

### 舊程式
- 未通過 `public == 0 && teacher_id == user_id` 檢查時，無任何回應
- 前端無法區分失敗原因

### 新 API
- **404**: 檢核表不存在
- **403**: 無權限編輯（teacher_id 不匹配）
- **400**: 已鎖定的檢核表不可編輯

**✅ 新 API 錯誤處理更明確**

---

## 7. 重複邏輯識別

### 7.1 權限檢查模式 🔴 **重複代碼**
- **位置**: 與 `detail`, `delete`, `toggleLock`, `copy` 相同
- **建議**: ⚠️ 可提取 Helper（見 detail.md § 8.1）

### 7.2 部分更新模式 ✅ **良好實踐**
```php
$data = [
    'title_name' => $params['title_name'] ?? $checklist['title_name'],
    // ...
];
```
- 保留原值 or 使用新值
- 支援部分更新（PATCH 語意）

---

## 8. 改進建議

### 8.1 鎖定檢查位置 (⚪ 低優先級)
**當前做法**: 在 Service 層檢查鎖定狀態

**替代方案**: 可合併到權限檢查 Helper
```php
private function verifyEditPermission($sn, $teacherId)
{
    $checklist = $this->verifyOwnership($sn, $teacherId, '編輯');
    
    if ($checklist['public'] == 1) {
        throw new AppException('已鎖定的檢核表不可編輯', 400);
    }
    
    return $checklist;
}
```

**優點**: 
- 權限檢查更集中
- 代碼更簡潔

**缺點**:
- 可能過度抽象（其他方法不需要檢查鎖定狀態）

**結論**: 當前實現已經很清晰，無需修改

---

## 9. 檢查結論

| 檢查項目 | 狀態 |
|---------|------|
| SQL 查詢一致性 | ✅ 一致，新 API 更明確 |
| 參數驗證 | ✅ 新 API 更完善 |
| 業務邏輯流程 | ✅ 新 API 增加明確錯誤處理 |
| 回應格式 | ✅ 新 API 更標準 |
| 錯誤處理 | ✅ 新 API 提供明確錯誤訊息 |
| 重複邏輯處理 | ⚠️ 權限檢查可提取 Helper |

### 整體評估: **✅ 邏輯一致，實現優良**

**重大改進**:
- ✅ **明確的錯誤訊息**（404, 403, 400）
- ✅ **支援部分更新**（未傳欄位保留原值）
- ✅ **陣列格式支援**
- ✅ **鎖定狀態檢查**（防止誤操作）

**邏輯完整，無重大問題**
