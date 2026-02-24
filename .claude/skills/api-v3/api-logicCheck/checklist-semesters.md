# ChecklistController - semesters 端點邏輯檢查

## 1. 基本資訊

| 項目 | 內容 |
|------|------|
| **端點名稱** | semesters |
| **HTTP 方法** | POST |
| **API 路徑** | `/checklist/semesters` |
| **舊程式檔案** | `modules/srl/prodb_checklists_new.php` |
| **舊程式 action** | `getSheetSemes` (L24-36) |
| **新 API Controller** | `ChecklistController::semesters()` (L38-52) |
| **新 API Service** | `ChecklistService::getSemesters()` (L39-48) |
| **新 API DAO** | `ChecklistDao::getSemesters()` (L27-44) |
| **相關資料表** | `check_list_table` |
| **檢查日期** | 2026-02-11 |

---

## 2. SQL 查詢比對

### 舊程式 (prodb_checklists_new.php L25-29)
```sql
SELECT DISTINCT seme
FROM check_list_table
WHERE teacher_id = :teacher_id
AND is_delete = 1
ORDER BY seme DESC
```

### 新 API (ChecklistDao.php L33-39)
```sql
SELECT DISTINCT seme
FROM check_list_table
WHERE teacher_id = :teacher_id
AND is_delete = 1
ORDER BY seme DESC
```

**✅ SQL 完全一致**

---

## 3. 參數驗證比對

### 舊程式
- 通過 `$_SESSION['USERDATA']` 取得 `user_id`
- 無額外參數需求
- 無參數型別驗證

### 新 API
- 通過 Guard 驗證 POST + Token
- 通過 `UserHelper::guardTeacher()` 驗證教師身分
- `teacher_id` 從 `$request->getUserId()` 自動取得
- 無額外參數需求

**✅ 新 API 驗證更完善**
- ✅ 增加了 CSRF Token 驗證
- ✅ 增加了教師身分驗證
- ✅ HTTP 方法限制 (POST only)

---

## 4. 業務邏輯流程比對

### 舊程式流程
1. 檢查 CSRF Token (全域)
2. Session 取得 `user_id`
3. 查詢資料庫取得學期列表
4. 通過 `getYearSeme()` 取得當前學期
5. `array_unshift()` 加入當前學期到最前面
6. `array_unique()` 去除重複
7. JSON 編碼輸出

```php
// prodb_checklists_new.php L24-36
$query = $dbh->prepare("SELECT DISTINCT seme...");
$query->bindValue(":teacher_id", $user_id, PDO::PARAM_STR);
$query->execute();
$vSemes = $query->fetchAll(PDO::FETCH_COLUMN);
array_unshift($vSemes, getYearSeme());
$vSemes = array_unique($vSemes);
echo json_encode($vSemes);
```

### 新 API 流程
1. Guard 驗證 POST + Token (Controller L44)
2. 教師身分驗證 (Controller L45)
3. `request->getUserId()` 取得教師 ID (Controller L47)
4. 查詢資料庫取得學期列表 (Dao L33-43)
5. 通過 `getCurrentSeme()` 取得當前學期 (Service L44)
6. `array_unshift()` 加入當前學期到最前面 (Service L45)
7. `array_unique()` 去除重複 (Service L47)
8. `array_values()` 重建索引 (Service L47)
9. 標準化 JSON 回應 (Controller L49)

```php
// ChecklistService.php L39-48
$semesters = $this->checklistDao->getSemesters($teacherId);
$currentSeme = $this->getCurrentSeme();
array_unshift($semesters, $currentSeme);
return array_values(array_unique($semesters));
```

**✅ 邏輯完全一致，新 API 增加額外處理**
- ⏫ 增加 `array_values()` 重建陣列索引（避免索引不連續）
- ⏫ 增加權限驗證層

---

## 5. 回應格式比對

### 舊程式
```json
["11302", "11301", "11202"]
```
- 直接返回學期陣列
- 純 JSON 格式

### 新 API
```json
{
  "status": "success",
  "data": {
    "semesters": ["11302", "11301", "11202"]
  }
}
```
- 包裝在標準化 API 回應格式中
- `status` + `data` 結構

**📝 差異說明**: 
- 舊版直接返回陣列
- 新版使用統一的 API 回應格式（符合 API v3 規範）

---

## 6. 錯誤處理比對

### 舊程式
- 無明確錯誤處理
- PDO 異常會直接中斷執行

### 新 API
- Guard 失敗會拋出異常（401/403）
- 教師身分驗證失敗會拋出異常（403）
- PDO 異常會被 BaseController 捕獲並返回標準錯誤格式

**✅ 新 API 錯誤處理更完善**

---

## 7. 重複邏輯識別

### 7.1 當前學期取得邏輯
- **位置**:
  - 舊程式: `getYearSeme()` 函數 (全域函數)
  - 新 API: `ChecklistService::getCurrentSeme()` (L301-311)
- **差異**: 新 API 自行實現了邏輯，包含 fallback 機制
- **建議**: ✅ 已獨立實現，有 fallback，更穩健

### 7.2 學期列表處理模式
- **位置**:
  - `ChecklistService::getSemesters()` L39-48
- **模式**: 
  - 查詢 DB → 加入當前學期 → 去重
- **建議**: 此模式已提取為獨立方法 ✅

---

## 8. 改進建議

### 8.1 當前學期的重複性 (⚪ 低優先級)
**問題**: 當前學期已在資料庫中存在時，會出現兩次，雖然 `array_unique()` 可以去重，但多一次操作

**建議方案**:
```php
// Option 1: 先去重再加入
$semesters = array_unique($this->checklistDao->getSemesters($teacherId));
if (!in_array($currentSeme, $semesters)) {
    array_unshift($semesters, $currentSeme);
}
return array_values($semesters);
```

**優點**: 減少不必要的陣列操作

**影響**: 極小，當前實現已經正確

---

## 9. 檢查結論

| 檢查項目 | 狀態 |
|---------|------|
| SQL 查詢一致性 | ✅ 完全一致 |
| 參數驗證 | ✅ 新 API 更完善（增加身分驗證） |
| 業務邏輯流程 | ✅ 完全一致，新 API 增加索引重建 |
| 回應格式 | 📝 新 API 使用標準化格式 (符合預期) |
| 錯誤處理 | ✅ 新 API 更完善 |
| 重複邏輯處理 | ✅ 已獨立提取 |

### 整體評估: **✅ 邏輯一致，實現優於舊版**

**亮點**:
- ✅ 增加了完整的權限驗證
- ✅ 使用標準化 API 回應格式  
- ✅ 錯誤處理更完善
- ✅ 代碼結構清晰（Controller → Service → DAO）

**無重大問題**
