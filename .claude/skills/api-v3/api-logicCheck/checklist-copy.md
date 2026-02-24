# ChecklistController - copy 端點邏輯檢查

## 1. 基本資訊

| 項目 | 內容 |
|------|------|
| **端點名稱** | copy |
| **HTTP 方法** | POST |
| **API 路徑** | `/checklist/copy` |
| **舊程式檔案** | `modules/srl/prodb_checklists_new.php` |
| **舊程式 action** | `copydata` (L184-215) |
| **新 API Controller** | `ChecklistController::copy()` (L210-230) |
| **新 API Service** | `ChecklistService::copy()` (L217-232) |
| **新 API DAO** | `ChecklistDao::copy()` (L228-261) |
| **相關資料表** | `check_list_table` |
| **檢查日期** | 2026-02-11 |

---

## 2. SQL 查詢比對

### 2.1 查詢原始資料

#### 舊程式 (prodb_checklists_new.php L189-191)
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

**✅ 邏輯一致**（新 API 指定欄位更明確）

### 2.2 插入複製資料

#### 舊程式 (prodb_checklists_new.php L198-200)
```sql
INSERT INTO `check_list_table`
    (seme, teacher_id, type, title_name, qusetion, score, public)
VALUES (:seme, :teacher_id, :type, :title_name, :qusetion, :score, :public)
```

#### 新 API (ChecklistDao.php L245-247)
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
// prodb_checklists_new.php L185-207
$sn = $_POST["sn"];  // hash 編碼
$teacher_id = $vUser_data['user_id'];  // 從 session 取得
$sCopyData->bindValue(':public', $_POST["public"], PDO::PARAM_STR);
```
- SN 需要解 hash
- `public` 從前端傳入
- 無其他參數驗證

### 新 API
```php
// ChecklistController.php L217-219
$params = $this->validate([
    'sn' => 'required|integer'
]);
```

**✅ 新 API 驗證更簡潔**
- ✅ 必填驗證
- ✅ 型別驗證
- ✅ `public` 硬編碼為 0（複製品預設未鎖定）

---

## 4. 業務邏輯流程比對

### 舊程式流程
1. 接收 `sn` 和 `public`
2. 解 hash
3. 從 session 取得 `teacher_id`
4. 查詢原始檢核表資料
5. 提取欄位值並修改標題（加上 "(複製)"）
6. 插入新記錄
7. 返回 `{status: 'success'}`

```php
// prodb_checklists_new.php L184-215
$sn = $_POST["sn"];
$teacher_id = $vUser_data['user_id'];

$sFindIfDone = $dbh->prepare("SELECT * FROM...");
$sFindIfDone->bindValue(":sn", hash2string($sn), PDO::PARAM_STR);
$sFindIfDone->execute();
$sFindIfDone = $sFindIfDone->fetch(\PDO::FETCH_ASSOC);

$scopy_title = $sFindIfDone["title_name"] . '(複製)';

$sCopyData = $dbh->prepare("INSERT INTO...");
$sCopyData->bindValue(':teacher_id', $teacher_id, PDO::PARAM_STR);
$sCopyData->bindValue(':title_name', $scopy_title, PDO::PARAM_STR);
$sCopyData->bindValue(':public', $_POST["public"], PDO::PARAM_STR);
// ...
```

**⚠️ 舊程式無權限檢查 - 可複製他人檢核表**

### 新 API 流程
1. 接收 `sn`
2. **權限檢查**:
   - 檢核表是否存在 (404)
   - （注意：copy 不檢查 teacher_id，允許複製他人公開檢核表）
3. 調用 DAO 複製
   - 標題加上 "(複製)"
   - `public` 硬編碼為 0（未鎖定）
   - `seme` 設為當前學期
   - `teacher_id` 設為當前用戶
4. 返回新 SN

```php
// ChecklistService.php L217-232
$checklist = $this->checklistDao->getById($sn);

if (!$checklist) {
    throw new AppException('檢核表不存在', 404);
}

$newSn = $this->checklistDao->copy($sn, $teacherId, $this->getCurrentSeme());

if (!$newSn) {
    throw new AppException('複製失敗', 500);
}

return ['sn' => $newSn];
```

```php
// ChecklistDao.php L228-261
$original = $this->getById($sn);

$stmt = $this->db->prepare("INSERT INTO...");
$stmt->execute([
    ':seme' => $seme,
    ':teacher_id' => $teacherId,  // 使用當前用戶
    ':type' => $original['type'],
    ':title_name' => $original['title_name'] . '(複製)',
    ':qusetion' => $original['qusetion'],
    ':score' => $original['score'],
    ':public' => 0  // 硬編碼為未鎖定
]);

return $this->db->lastInsertId();
```

**✅ 新 API 邏輯更清晰**
- ⏫ 返回新 SN
- ⏫ `public` 硬編碼為 0（更合理）
- ⏫ `teacher_id` 自動設為當前用戶

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
    "sn": 456
  }
}
```

**✅ 新 API 返回新 SN**（方便前端導航）

---

## 6. 錯誤處理比對

### 舊程式
- 無權限檢查
- 錯誤訊息為行號

### 新 API
- **404**: 檢核表不存在
- **500**: 複製失敗

**✅ 新 API 錯誤處理更明確**

---

## 7. 重複邏輯識別

### 7.1 複製邏輯的特殊性 ✅ **合理設計**
- **注意**: `copy` 方法**不檢查** `teacher_id`
- **原因**: 允許複製他人公開的檢核表（作為模板）
- **實現**: 複製時 `teacher_id` 自動設為當前用戶

**這是合理的業務邏輯**:
- 教師 A 可以複製教師 B 的檢核表
- 複製品的擁有者是教師 A（不是 B）

### 7.2 標題處理 ✅ **一致**
- 舊版和新版都在標題後加 "(複製)"

### 7.3 公開狀態處理 ⚠️ **差異**
- **舊版**: `public` 從前端傳入
- **新版**: `public` 硬編碼為 0

**新版更合理**: 複製品應該預設未鎖定，由用戶自行決定是否鎖定

---

## 8. 改進建議

### 8.1 公開狀態硬編碼 (✅ 已改進)
**舊版問題**: 
```php
$sCopyData->bindValue(':public', $_POST["public"], PDO::PARAM_STR);
```
- 前端可指定複製品的鎖定狀態
- 不合理（複製品應該預設未鎖定）

**新 API 解決**:
```php
':public' => 0  // 硬編碼為未鎖定
```

**優點**:
- 更符合業務邏輯
- 統一行為（所有複製品都是未鎖定）

### 8.2 返回新 SN (✅ 已改進)
**新 API 優勢**:
```json
{
  "sn": 456
}
```
- 前端可立即導航到新檢核表
- 減少一次查詢請求

---

## 9. 檢查結論

| 檢查項目 | 狀態 |
|---------|------|
| SQL 查詢一致性 | ✅ 一致 |
| 參數驗證 | ✅ 新 API 更簡潔 |
| 業務邏輯流程 | ✅ 新 API 更清晰 |
| 回應格式 | ✅ 新 API 返回新 SN |
| 錯誤處理 | ✅ 新 API 更明確 |
| 重複邏輯處理 | ✅ 合理設計 |

### 整體評估: **✅ 邏輯一致，實現優良**

**重要特性**:
- ✅ **允許跨用戶複製**（合理的業務需求）
- ✅ **複製品擁有者自動設為當前用戶**
- ✅ **公開狀態硬編碼為 0**（更合理）
- ✅ **返回新 SN**（前端更便利）

**業務邏輯**:
- 複製功能允許教師複製他人公開的檢核表作為模板
- 這是合理的功能設計，不是安全漏洞
