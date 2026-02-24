# ChecklistController - toggleLock 端點邏輯檢查

## 1. 基本資訊

| 項目 | 內容 |
|------|------|
| **端點名稱** | toggleLock |
| **HTTP 方法** | POST |
| **API 路徑** | `/checklist/toggle-lock` |
| **舊程式檔案** | `modules/srl/prodb_checklists_new.php` |
| **舊程式 action** | `updatelockdata` (L145-158) |
| **新 API Controller** | `ChecklistController::toggleLock()` (L182-208) |
| **新 API Service** | `ChecklistService::toggleLock()` (L193-208) |
| **新 API DAO** | `ChecklistDao::updatePublicStatus()` (L213-226) |
| **相關資料表** | `check_list_table` |
| **檢查日期** | 2026-02-11 |

---

## 2. SQL 查詢比對

### 舊程式 (prodb_checklists_new.php L148-150)
```sql
UPDATE `check_list_table` 
SET public = :public
WHERE check_list_table_sn = :sn
```

### 新 API (ChecklistDao.php L220)
```sql
UPDATE check_list_table 
SET public = :public 
WHERE check_list_table_sn = :sn
```

**✅ SQL 完全一致**

---

## 3. 參數驗證比對

### 舊程式
```php
// prodb_checklists_new.php L146-150
$sn = $_POST["sn"];  // hash 編碼
$sFindIfDone->bindValue(":sn", hash2string($sn), PDO::PARAM_STR);
$sFindIfDone->bindValue(':public', $_POST["public"], PDO::PARAM_STR);
```
- SN 需要解 hash
- `public` 直接使用，無驗證
- 無權限檢查

### 新 API
```php
// ChecklistController.php L189-192
$params = $this->validate([
    'sn'     => 'required|integer',
    'public' => 'required|integer|in:0,1'
]);
```

**✅ 新 API 驗證更嚴格**
- ✅ 必填驗證
- ✅ 型別驗證（integer）
- ✅ 值範圍限制（0 或 1）

---

## 4. 業務邏輯流程比對

### 舊程式流程
1. 接收 `sn` 和 `public`
2. 解 hash
3. **直接更新**（無權限檢查）
4. 返回 `{status: 'success'}` 或 `{status: 'fail'}`

```php
// prodb_checklists_new.php L145-158
$sn = $_POST["sn"];
try {
    $sFindIfDone = $dbh->prepare("UPDATE...");
    $sFindIfDone->bindValue(":sn", hash2string($sn), PDO::PARAM_STR);
    $sFindIfDone->bindValue(':public', $_POST["public"], PDO::PARAM_STR);
    $sFindIfDone->execute();
    $returnData['status'] = 'success';
} catch (\PDOexception $e) {
    $returnData['status'] = 'fail';
    $returnData['msg'] = __LINE__;
}
```

**⚠️ 舊程式無權限檢查！**

### 新 API 流程
1. 接收 `sn` 和 `public`
2. 查詢檢核表資料
3. **權限檢查**:
   - 檢核表是否存在 (404)
   - teacher_id 是否匹配 (403)
4. 更新 public 狀態
5. 返回 `{sn: 123, public: 1}`

```php
// ChecklistService.php L193-208
$checklist = $this->checklistDao->getById($sn);

if (!$checklist) {
    throw new AppException('檢核表不存在', 404);
}

if ($checklist['teacher_id'] !== $teacherId) {
    throw new AppException('無權限操作此檢核表', 403);
}

$this->checklistDao->updatePublicStatus($sn, $public);
return ['sn' => $sn, 'public' => $public];
```

**✅ 新 API 邏輯更安全**
- ⏫ **增加權限檢查**（防止跨用戶操作）

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
    "sn": 123,
    "public": 1
  }
}
```

**✅ 新 API 回應更豐富**
- 返回更新後的狀態（方便前端同步）

---

## 6. 錯誤處理比對

### 舊程式
- 無權限檢查，任何人可操作
- 錯誤訊息為行號

### 新 API
- **404**: 檢核表不存在
- **403**: 無權限操作（teacher_id 不匹配）
- **400**: 參數錯誤（public 不是 0 或 1）

**✅ 新 API 錯誤處理更完善**

---

## 7. 重複邏輯識別

### 7.1 權限檢查模式 🔴 **重複代碼**
- 與 `detail`, `update`, `delete`, `copy` 完全相同
- **建議**: ⚠️ **強烈建議提取 Helper**

### 7.2 安全性問題 🔴 **重要**
- **舊程式問題**: 無權限檢查，可跨用戶鎖定/解鎖他人檢核表
- **新 API 解決**: 增加 teacher_id 檢查

---

## 8. 改進建議

### 8.1 舊版安全漏洞修正 (✅ 已改進)
**舊版問題**: 
```php
// 任何人只要知道 hash(sn) 即可鎖定/解鎖他人檢核表
UPDATE check_list_table SET public = :public WHERE check_list_table_sn = :sn
```
- 無 teacher_id 檢查
- 可跨用戶修改他人檢核表狀態
- 可能導致：
  - 惡意鎖定他人檢核表（無法編輯）
  - 惡意解鎖他人已鎖定的檢核表

**新 API 解決**:
```php
if ($checklist['teacher_id'] !== $teacherId) {
    throw new AppException('無權限操作此檢核表', 403);
}
```

**影響**: 
- 🔴 **Critical** - 舊版存在嚴重安全漏洞
- ✅ 新版已完全修正

### 8.2 返回更新後狀態 (✅ 已改進)
**新 API 優勢**:
```json
{
  "sn": 123,
  "public": 1
}
```
- 前端可立即同步狀態
- 無需額外查詢

---

## 9. 檢查結論

| 檢查項目 | 狀態 |
|---------|------|
| SQL 查詢一致性 | ✅ 完全一致 |
| 參數驗證 | ✅ 新 API 更嚴格 |
| 業務邏輯流程 | ✅ 新 API 增加權限檢查（重要） |
| 回應格式 | ✅ 新 API 更豐富 |
| 錯誤處理 | ✅ 新 API 更完善 |
| 重複邏輯處理 | ⚠️ 權限檢查強烈建議提取 Helper |

### 整體評估: **✅ 邏輯一致，重大安全修正**

**重大安全修正**:
- 🔐 **修正嚴重安全漏洞**（舊版無權限檢查）
- ✅ **防止跨用戶惡意鎖定/解鎖**
- ✅ **參數範圍限制**（public 只能是 0 或 1）

**舊版安全問題**:
- 🔴 **Critical**: 任何人可鎖定/解鎖他人檢核表
  - 可能導致惡意鎖定（他人無法編輯）
  - 可能導致惡意解鎖（破壞資料完整性）
- ✅ 新版已完全修正此漏洞

**強烈建議**:
- 🔴 提取權限檢查 Helper（已在 5 個方法中重複）
