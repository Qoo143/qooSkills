# ChecklistController - list 端點邏輯檢查

## 1. 基本資訊

| 項目 | 內容 |
|------|------|
| **端點名稱** | list |
| **HTTP 方法** | POST |
| **API 路徑** | `/checklist/list` |
| **舊程式檔案** | `modules/srl/prodb_checklists_new.php` |
| **舊程式 action** | `getPages` (L38-57) + `loadData` (L59-101) |
| **新 API Controller** | `ChecklistController::list()` (L54-78) |
| **新 API Service** | `ChecklistService::getList()` (L57-74) |
| **新 API DAO** | `ChecklistDao::getPageCount()` (L46-77) + `ChecklistDao::getList()` (L79-121) |
| **相關資料表** | `check_list_table` |
| **檢查日期** | 2026-02-11 |

---

## 2. SQL 查詢比對

### 2.1 分頁數查詢

#### 舊程式 (prodb_checklists_new.php L43-47)
```sql
SELECT CEIL(COUNT(*) / 10) pages
FROM check_list_table
WHERE teacher_id = :user_id
AND is_delete = 1
$sWhereSql  -- 動態 WHERE (type, seme)
```

#### 新 API (ChecklistDao.php L51-55)
```sql
SELECT CEIL(COUNT(*) / 10) as pages
FROM check_list_table
WHERE teacher_id = :teacher_id
AND is_delete = 1
-- 動態 WHERE (type, seme)
```

**✅ SQL 完全一致**

### 2.2 列表數據查詢

#### 舊程式 (prodb_checklists_new.php L68-80)
```sql
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
$sWhereSql  -- 動態 WHERE (type, seme)
ORDER BY check_list_table_sn DESC
$sLimitSql  -- LIMIT offset, 10
```

#### 新 API (ChecklistDao.php L85-108)
```sql
SELECT check_list_table_sn as sn,
       seme,
       type,
       title_name,
       qusetion,
       score,
       public
FROM check_list_table
WHERE teacher_id = :teacher_id
AND is_delete = 1
-- 動態 WHERE (type, seme)
ORDER BY check_list_table_sn DESC
LIMIT {offset}, 10
```

**✅ SQL 完全一致**

---

## 3. 參數驗證比對

### 舊程式
- `$_POST['type']` - 直接使用，無驗證
- `$_POST['seme']` - 直接使用，無驗證
- `$_POST['page']` - 通過 `intval()` 轉型
- 無型別驗證
- 無預設值處理

### 新 API
```php
// ChecklistController.php L66-70
$params = $this->validate([
    'type' => 'string',          // 字串型別
    'seme' => 'string',          // 字串型別
    'page' => 'integer|min:1|default:1'  // 整數，最小值 1，預設 1
]);
```

**✅ 新 API 驗證更完善**
- ✅ 增加型別驗證
- ✅ 增加範圍限制（page >= 1）
- ✅ 增加預設值（page = 1）
- ✅ HTTP 方法限制 (POST only)
- ✅ Token 驗證
- ✅ 教師身分驗證

---

## 4. 業務邏輯流程比對

### 舊程式流程
1. 構建動態 WHERE 條件語句
   - `type === 'all'` → 不加條件
   - `seme === 'all'` → 不加條件
2. **分兩次請求完成**:
   - 第一次請求: `act=getPages` 取得總頁數
   - 第二次請求: `act=loadData` 取得列表數據
3. 計算分頁 offset: `(page - 1) * 10`
4. 查詢資料庫
5. 通過 `explode()` 將字串拆分為陣列
6. 使用 `string2hash()` 混淆 SN 作為 key
7. 返回 JSON

```php
// prodb_checklists_new.php L59-101
$sWhereSql = '';
$sWhereSql .= $_POST['type'] === 'all' ? '' : " AND type = :type ";
$sWhereSql .= $_POST['seme'] === 'all' ? '' : " AND seme = :seme ";
$iLimitStart = (intval($_POST['page']) - 1) * 10;

// 查詢...
foreach ($vData as $iSn => $vValue) {
    $vValue['qusetion'] = array_filter(explode(_SPLIT_SYMBOL, $vValue['qusetion']));
    $vValue['score'] = array_filter(explode(_SPLIT_SYMBOL, $vValue['score']));
    $vReturn[string2hash($iSn)] = $vValue;
}
```

### 新 API 流程
1. 構建 filters 陣列
   - `type` 和 `seme` 直接傳入（預設 'all'）
2. **單次請求完成**:
   - 同時取得列表數據和總頁數
3. 在 DAO 層動態構建 WHERE 條件
4. 計算分頁 offset: `(page - 1) * 10`
5. 查詢資料庫
6. 通過 `formatChecklist()` 格式化每筆資料
   - `stringToArray()` 將字串拆分為陣列
   - 增加 `type_name`、`is_locked` 等欄位
   - 不混淆 SN，直接返回
7. 返回結構化數據（包含 pagination 信息）

```php
// ChecklistService.php L57-74
$filters = [
    'type' => $params['type'] ?? 'all',
    'seme' => $params['seme'] ?? 'all'
];
$page = $params['page'] ?? 1;

$checklists = $this->checklistDao->getList($teacherId, $filters, $page);
$totalPages = $this->checklistDao->getPageCount($teacherId, $filters);

return [
    'checklists' => array_map([$this, 'formatChecklist'], $checklists),
    'page' => (int)$page,
    'total_pages' => $totalPages,
    'count' => count($checklists)
];
```

**✅ 邏輯一致，新 API 改進明顯**
- ⏫ 單次請求完成（舊版需兩次）
- ⏫ 返回更豐富的分頁信息
- ⏫ 不使用 hash 混淆 SN（更直觀）
- ⏫ 增加欄位（type_name, is_locked）

---

## 5. 回應格式比對

### 舊程式 (getPages)
```json
{
  "pages": 3
}
```

### 舊程式 (loadData)
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
- Hash key 作為索引
- 字串型別的數字（"1", "0"）
- 需要兩次請求分別取得

### 新 API
```json
{
  "status": "success",
  "data": {
    "checklists": [
      {
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
    ],
    "page": 1,
    "total_pages": 3,
    "count": 10
  }
}
```
- 陣列索引（更標準）
- 數字型別正確（1, 0, 45）
- 單次請求包含所有資訊
- 增加欄位: `type_name`, `is_locked`
- 標準化 API 回應格式

**📝 差異說明**: 
- 舊版使用 hash key，新版使用標準陣列
- 舊版需兩次請求，新版單次完成
- 新版提供更豐富的分頁信息
- 新版符合 REST API 最佳實踐

---

## 6. 錯誤處理比對

### 舊程式
- 無參數驗證，傳入非法值可能導致 SQL 錯誤
- 無權限檢查（只檢查 teacher_id）
- PDO 異常直接中斷

### 新 API
- 參數驗證失敗返回 400（Bad Request）
- Guard 失敗返回 401/403
- 教師身分驗證失敗返回 403
- PDO 異常被統一處理

**✅ 新 API 錯誤處理更完善**

---

## 7. 重複邏輯識別

### 7.1 動態 WHERE 條件構建 🔴 **高頻出現**
- **位置**:
  - `prodb_checklists_new.php` L39-41 (getPages)
  - `prodb_checklists_new.php` L60-62 (loadData)
  - `ChecklistDao::getPageCount()` L59-68
  - `ChecklistDao::getList()` L95-106
- **模式**: 
  ```php
  if ($filter !== 'all') {
      $sql .= " AND field = :field";
      $params[':field'] = $filter;
  }
  ```
- **建議**: ✅ 已在 DAO 層統一處理，避免在 Controller/Service 重複

### 7.2 字串 ↔ 陣列轉換邏輯
- **位置**:
  - 舊程式: `explode(_SPLIT_SYMBOL, $value)` + `array_filter()`
  - 新 API: `ChecklistService::stringToArray()` (L356-364) + `arrayToString()` (L340-348)
- **符號**: `|||` (SPLIT_SYMBOL)
- **建議**: ✅ 已提取為 Helper 方法，可重用

### 7.3 分頁計算邏輯
- **位置**:
  - 舊程式: `(intval($_POST['page']) - 1) * 10`
  - 新 API: `($page - 1) * 10`
- **建議**: ⚪ 固定公式，不需提取。若未來支持「每頁數量可變」，可考慮提取

### 7.4 Hash SN 處理
- **位置**:
  - 舊程式: `string2hash($sn)` (L96)
- **問題**: 舊版使用 hash 混淆 SN
- **建議**: ✅ 新 API 不再使用 hash，更直觀

---

## 8. 改進建議

### 8.1 兩次請求 → 單次請求 (✅ 已改進)
**舊版問題**: 需要兩次 AJAX 請求
- 第一次: `act=getPages` 取得總頁數
- 第二次: `act=loadData` 取得列表

**新 API 解決**: 單次請求返回所有資訊
```php
return [
    'checklists' => [...],
    'page' => 1,
    'total_pages' => 3,
    'count' => 10
];
```

**優點**: 
- 減少網路請求
- 降低前端複雜度
- 提升效能

### 8.2 Hash SN 移除 (✅ 已改進)
**舊版問題**: 使用 `string2hash()` 混淆 SN 作為 key，增加複雜度

**新 API 解決**: 直接使用 SN，配合權限檢查確保安全

**優點**:
- 代碼更簡潔
- 前端更容易處理
- 權限檢查已在 Service 層實現

### 8.3 回應格式標準化 (✅ 已改進)
**舊版問題**: 回應格式不統一
- 字串型別的數字
- Hash key 作為索引

**新 API 解決**: 標準化 JSON 結構
- 正確的數字型別
- 陣列索引
- 增加語意化欄位（type_name, is_locked）

---

## 9. 檢查結論

| 檢查項目 | 狀態 |
|---------|------|
| SQL 查詢一致性 | ✅ 完全一致 |
| 參數驗證 | ✅ 新 API 更完善（型別、範圍、預設值） |
| 業務邏輯流程 | ✅ 一致，新 API 優化（單次請求） |
| 回應格式 | ✅ 新 API 標準化，更豐富 |
| 錯誤處理 | ✅ 新 API 更完善 |
| 重複邏輯處理 | ✅ 已提取 Helper 方法 |

### 整體評估: **✅ 邏輯一致，重大改進**

**重大改進**:
- ✅ **單次請求完成**（舊版需兩次）
- ✅ **移除 Hash SN 混淆**（更直觀）
- ✅ **標準化回應格式**（類型正確、結構清晰）
- ✅ **增加分頁元數據**（page, total_pages, count）
- ✅ **增加語意化欄位**（type_name, is_locked）

**性能提升**:
- 網路請求減少 50%（2次 → 1次）

**無重大問題，建議直接採用**
