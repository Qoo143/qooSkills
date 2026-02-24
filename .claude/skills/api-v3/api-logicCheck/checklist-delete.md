# ChecklistController - delete 端點邏輯檢查

## 1. 基本資訊

| 項目 | 內容 |
|------|------|
| **端點名稱** | delete |
| **HTTP 方法** | POST |
| **API 路徑** | `/checklist/delete` |
| **舊程式檔案** | `modules/srl/prodb_checklists_new.php` |
| **舊程式 action** | `deleteData` (L129-143) |
| **新 API Controller** | `ChecklistController::delete()` (L160-180) |
| **新 API Service** | `ChecklistService::delete()` (L168-183) |
| **新 API DAO** | `ChecklistDao::delete()` (L199-211) |
| **相關資料表** | `check_list_table` |
| **檢查日期** | 2026-02-11 |

---

## 2. SQL 查詢比對

### 舊程式 (prodb_checklists_new.php L132-134)
```sql
UPDATE `check_list_table` 
SET is_delete = :is_delete 
WHERE check_list_table_sn = :sn
```

### 新 API (ChecklistDao.php L206)
```sql
UPDATE check_list_table 
SET is_delete = 0 
WHERE check_list_table_sn = :sn
```

**✅ SQL 邏輯一致**
- 兩者都是**軟刪除**（is_delete = 0）
- 新 API 硬編碼 `is_delete = 0`（更明確）

---

## 3. 參數驗證比對

### 舊程式
```php
// prodb_checklists_new.php L130-134
$sn = $_POST["sn"];  // hash 編碼
$sFindIfDone->bindValue(":sn", hash2string($sn), PDO::PARAM_STR);
$sFindIfDone->bindValue(':is_delete', $_POST["is_delete"], PDO::PARAM_STR);
```
- `is_delete` 從前端傳入（理論上應該都是 0）
- SN 需要解 hash
- 無參數驗證

### 新 API
```php
// ChecklistController.php L167-169
$params = $this->validate([
    'sn' => 'required|integer'
]);
```

**✅ 新 API 驗證更簡潔**
- ✅ 必填驗證
- ✅ 型別驗證
- ✅ `is_delete` 硬編碼為 0（更安全，防止誤操作）

---

## 4. 業務邏輯流程比對

### 舊程式流程
1. 接收 `sn` 和 `is_delete`
2. 解 hash
3. 直接更新（無權限檢查）
4. 返回 `{status: 'success'}` 或 `{status: 'fail', msg: line_number}`

```php
// prodb_checklists_new.php L129-143
$sn = $_POST["sn"];
try {
    $sFindIfDone = $dbh->prepare("UPDATE...");
    $sFindIfDone->bindValue(":sn", hash2string($sn), PDO::PARAM_STR);
    $sFindIfDone->bindValue(':is_delete', $_POST["is_delete"], PDO::PARAM_STR);
    $sFindIfDone->execute();
    $returnData['status'] = 'success';
} catch (\PDOexception $e) {
    $returnData['status'] = 'fail';
    $returnData['msg'] = __LINE__;
}
```

**⚠️ 舊程式無權限檢查！**

### 新 API 流程
1. 接收 `sn`
2. 查詢檢核表資料
3. **權限檢查**:
   - 檢核表是否存在 (404)
   - teacher_id 是否匹配 (403)
4. 執行軟刪除（is_delete = 0）
5. 返回 `{sn: 123}`

```php
// ChecklistService.php L168-183
$checklist = $this->checklistDao->getById($sn);

if (!$checklist) {
    throw new AppException('檢核表不存在', 404);
}

if ($checklist['teacher_id'] !== $teacherId) {
    throw new AppException('無權限刪除此檢核表', 403);
}

$this->checklistDao->delete($sn);
return ['sn' => $sn];
```

**✅ 新 API 邏輯更安全**
- ⏫ **增加權限檢查**（防止跨用戶刪除）
- ⏫ 硬編碼 `is_delete = 0`（防止誤操作）

---

## 5. 回應格式比對

### 舊程式
```json
{
  "status": "success"
}
```

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
- 無權限檢查，任何人傳入正確的 hash(sn) 即可刪除
- 錯誤訊息為行號

### 新 API
- **404**: 檢核表不存在
- **403**: 無權限刪除（teacher_id 不匹配）

**✅ 新 API 錯誤處理更完善**

---

## 7. 重複邏輯識別

### 7.1 權限檢查模式 🔴 **重複代碼**
- 與 `detail`, `update`, `toggleLock`, `copy` 相同
- **建議**: ⚠️ 可提取 Helper

### 7.2 安全性問題 🔴 **重要**
- **舊程式問題**: 無權限檢查，可跨用戶刪除
- **新 API 解決**: 增加 teacher_id 檢查

---

## 8. 改進建議

### 8.1 舊版安全漏洞修正 (✅ 已改進)
**舊版問題**: 
```php
// 任何人只要知道 hash(sn) 即可刪除
$sFindIfDone = $dbh->prepare("UPDATE check_list_table SET is_delete = :is_delete WHERE check_list_table_sn = :sn");
```
- 無 teacher_id 檢查
- 可跨用戶刪除他人資料

**新 API 解決**:
```php
if ($checklist['teacher_id'] !== $teacherId) {
    throw new AppException('無權限刪除此檢核表', 403);
}
```

**影響**: 
- 🔴 **Critical** - 舊版存在資料安全漏洞
- ✅ 新版已完全修正

### 8.2 硬刪除 vs 軟刪除 (✅ 當前實現正確)
**當前做法**: 軟刪除（is_delete = 0）

**優點**:
- 資料可恢復
- 符合業務需求（保留歷史記錄）

**結論**: 無需修改

---

## 9. 檢查結論

| 檢查項目 | 狀態 |
|---------|------|
| SQL 查詢一致性 | ✅ 邏輯一致 |
| 參數驗證 | ✅ 新 API 更簡潔 |
| 業務邏輯流程 | ✅ 新 API 增加權限檢查（重要） |
| 回應格式 | ✅ 新 API 更標準 |
| 錯誤處理 | ✅ 新 API 更完善 |
| 重複邏輯處理 | ⚠️ 權限檢查可提取 Helper |

### 整體評估: **✅ 邏輯一致，重大安全修正**

**重大安全修正**:
- 🔐 **修正安全漏洞**（舊版無權限檢查）
- ✅ **防止跨用戶刪除**
- ✅ **硬編碼 is_delete = 0**（防止誤操作）

**舊版安全問題**:
- 🔴 **Critical**: 任何人只要知道 hash(sn) 即可刪除他人資料
- ✅ 新版已完全修正此漏洞
