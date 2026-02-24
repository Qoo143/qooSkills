# ChecklistController - detail 端點邏輯檢查

## 1. 基本資訊

| 項目 | 內容 |
|------|------|
| **端點名稱** | detail |
| **HTTP 方法** | POST |
| **API 路徑** | `/checklist/detail` |
| **舊程式檔案** | `modules/srl/prodb_checklists_new.php` |
| **舊程式 action** | `loadData` (L59-101) - 通過 SN 查詢單筆 |
| **新 API Controller** | `ChecklistController::detail()` (L80-100) |
| **新 API Service** | `ChecklistService::getDetail()` (L83-96) |
| **新 API DAO** | `ChecklistDao::getById()` (L123-146) |
| **相關資料表** | `check_list_table` |
| **檢查日期** | 2026-02-11 |

---

## 2. SQL 查詢比對

### 舊程式 (prodb_checklists_new.php L68-80)
```sql
-- 使用 list 查詢，但前端通過 hash(sn) 定位單筆
SELECT check_list_table_sn sn,
       seme,
       type,
       title_name,
       qusetion,
       score,
       public
FROM check_list_table
WHERE teacher_id = :user_id
AND is_delete = 1
-- 可能還有 type/seme 篩選
ORDER BY check_list_table_sn DESC
```

**注意**: 舊程式沒有獨立的「查單筆」action，而是從 list 結果中通過 hash(sn) 定位

### 新 API (ChecklistDao.php L129-142)
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

**✅ 新 API 更精確**
- 直接通過 SN 查詢單筆
- 增加 `teacher_id` 和 `is_delete` 欄位（用於權限驗證）

---

## 3. 參數驗證比對

### 舊程式
- 無獨立的 detail action
- 依賴前端從 list 結果中選擇

### 新 API
```php
// ChecklistController.php L87-89
$params = $this->validate([
    'sn' => 'required|integer'
]);
```

**✅ 新 API 提供獨立端點**
- ✅ 必填驗證
- ✅ 整數型別驗證
- ✅ HTTP 方法限制 (POST)
- ✅ Token 驗證
- ✅ 教師身分驗證

---

## 4. 業務邏輯流程比對

### 舊程式流程
1. 前端調用 `loadData` 取得列表
2. 前端通過 hash(sn) 定位單筆資料
3. 無後端權限驗證（只依賴 teacher_id 查詢條件）

### 新 API 流程
1. 接收 `sn` 參數
2. 通過 DAO 查詢單筆資料
3. **權限驗證**:
   - 檢查檢核表是否存在 (404)
   - 檢查 teacher_id 是否匹配 (403)
4. 格式化並返回

```php
// ChecklistService.php L83-96
$checklist = $this->checklistDao->getById($sn);

if (!$checklist) {
    throw new AppException('檢核表不存在', 404);
}

if ($checklist['teacher_id'] !== $teacherId) {
    throw new AppException('無權限檢視此檢核表', 403);
}

return $this->formatChecklist($checklist);
```

**✅ 新 API 邏輯更完善**
- ⏫ 獨立端點，不依賴 list
- ⏫ 明確的權限檢查
- ⏫ 精確的錯誤訊息

---

## 5. 回應格式比對

### 舊程式
```json
{
  "abc123xyz": {
    "sn": 45,
    "seme": "11301",
    "type": "1",
    "title_name": "檢核單測試",
    "qusetion": ["問題1", "問題2"],
    "score": ["5", "3"],
    "public": "0"
  }
}
```
- Hash key
- 包含列表中的所有資料

### 新 API
```json
{
  "status": "success",
  "data": {
    "sn": 45,
    "seme": "11301",
    "type": 1,
    "type_name": "檢核單",
    "title_name": "檢核單測試",
    "questions": ["問題1", "問題2"],
    "scores": ["5", "3"],
    "public": 0,
    "is_locked": false
  }
}
```

**✅ 新 API 格式更標準**
- 直接返回單筆物件（不是陣列）
- 增加 `type_name`、`is_locked` 欄位
- 正確的型別

---

## 6. 錯誤處理比對

### 舊程式
- 無獨立端點，無法單獨處理錯誤
- 若 SN 不存在於列表中，前端可能無法處理

### 新 API
- **404**: 檢核表不存在
- **403**: 無權限檢視（teacher_id 不匹配）
- **400**: 參數錯誤（sn 非整數）

**✅ 新 API 錯誤處理更明確**

---

## 7. 重複邏輯識別

### 7.1 權限檢查模式 🔴 **高頻出現**
- **位置**:
  - `ChecklistService::getDetail()` L85-93
  - `ChecklistService::update()` L131-139
  - `ChecklistService::delete()` L170-178
  - `ChecklistService::toggleLock()` L195-203
  - `ChecklistService::copy()` L219-223
- **模式**:
  ```php
  $checklist = $this->checklistDao->getById($sn);
  if (!$checklist) {
      throw new AppException('檢核表不存在', 404);
  }
  if ($checklist['teacher_id'] !== $teacherId) {
      throw new AppException('無權限操作', 403);
  }
  ```
- **建議**: ⚠️ **可提取為 Helper 方法**
  ```php
  private function checkOwnership($sn, $teacherId, $action = '操作')
  {
      $checklist = $this->checklistDao->getById($sn);
      if (!$checklist) {
          throw new AppException('檢核表不存在', 404);
      }
      if ($checklist['teacher_id'] !== $teacherId) {
          throw new AppException("無權限{$action}此檢核表", 403);
      }
      return $checklist;
  }
  ```

---

## 8. 改進建議

### 8.1 提取權限檢查 Helper (🔴 高優先級)
**問題**: 權限檢查代碼在 5 個方法中重複

**建議方案**:
```php
// ChecklistService.php
private function verifyOwnership($sn, $teacherId, $action = '操作')
{
    $checklist = $this->checklistDao->getById($sn);
    
    if (!$checklist) {
        throw new AppException('檢核表不存在', 404);
    }
    
    if ($checklist['teacher_id'] !== $teacherId) {
        throw new AppException("無權限{$action}此檢核表", 403);
    }
    
    return $checklist;
}

// 使用示例
public function getDetail($teacherId, $sn)
{
    $checklist = $this->verifyOwnership($sn, $teacherId, '檢視');
    return $this->formatChecklist($checklist);
}
```

**優點**:
- 減少重複代碼
- 統一錯誤訊息
- 易於維護

---

## 9. 檢查結論

| 檢查項目 | 狀態 |
|---------|------|
| SQL 查詢一致性 | ✅ 新 API 更精確（直接查單筆） |
| 參數驗證 | ✅ 新 API 提供獨立端點 |
| 業務邏輯流程 | ✅ 新 API 增加權限驗證 |
| 回應格式 | ✅ 新 API 格式更標準 |
| 錯誤處理 | ✅ 新 API 錯誤處理更明確 |
| 重複邏輯處理 | ⚠️ 權限檢查可提取 Helper |

### 整體評估: **✅ 邏輯完整，有優化空間**

**重大改進**:
- ✅ **提供獨立端點**（舊版依賴 list）
- ✅ **增加權限驗證**（teacher_id 檢查）
- ✅ **明確的錯誤處理**（404, 403）

**優化建議**:
- 🔴 提取權限檢查 Helper（減少重複代碼）
