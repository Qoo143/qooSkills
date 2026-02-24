# ChecklistController - template 端點邏輯檢查

## 1. 基本資訊

| 項目 | 內容 |
|------|------|
| **端點名稱** | template |
| **HTTP 方法** | POST |
| **API 路徑** | `/checklist/template` |
| **舊程式檔案** | `modules/srl/prodb_checklists_new.php` |
| **舊程式 action** | `loadTableContent` (L218-281) |
| **新 API Controller** | `ChecklistController::template()` (L232-251) |
| **新 API Service** | `ChecklistService::getTemplate()` (L240-292) |
| **新 API DAO** | 無（純業務邏輯，無資料庫查詢） |
| **相關資料表** | 無 |
| **檢查日期** | 2026-02-11 |

---

## 2. SQL 查詢比對

### 舊程式
- 無資料庫查詢
- 純硬編碼模板資料

### 新 API
- 無資料庫查詢
- 純硬編碼模板資料

**✅ 一致（兩者都無 SQL）**

---

## 3. 參數驗證比對

### 舊程式
```php
// prodb_checklists_new.php L219
$iTable_type = hash2string($_POST["table_type"]);
```
- `type` 需要解 hash
- 無型別驗證
- 無範圍驗證

### 新 API
```php
// ChecklistController.php L239-241
$params = $this->validate([
    'type' => 'required|integer|in:1,2,3,4'
]);
```

**✅ 新 API 驗證更嚴格**
- ✅ 必填驗證
- ✅ 整數型別驗證
- ✅ 值範圍限制（1-4）

---

## 4. 業務邏輯流程比對

### 舊程式流程
1. 接收 `table_type`（hash 編碼）
2. `hash2string()` 解碼
3. 硬編碼定義四種模板：
   - 類型 1: 檢核單
   - 類型 2: 同儕評分表
   - 類型 3: 組間評分表
   - 類型 4: 組內評分表
4. 使用 `switch` 選擇模板
5. 返回 `{arraTableName, arraTableContent}`

```php
// prodb_checklists_new.php L218-281
$iTable_type = hash2string($_POST["table_type"]);

$arraChecklistName = '檢核單';
$arraChecklistContent = ["學習單有呈現領域...", ...];

$arraPersonalScoresheetName = '同儕評分表';
$arraPersonalScoresheetContent = ["能先介紹自己的姓名", ...];

// ...

switch ($iTable_type) {
    case '1':
        $arraTableName = $arraChecklistName;
        $arraTableContent = $arraChecklistContent;
        break;
    case '2':
        $arraTableName = $arraPersonalScoresheetName;
        $arraTableContent = $arraGroupScoresheetContent;  // ⚠️ Bug!
        break;
    // ...
}

echo json_encode(array('arraTableName'=>$arraTableName,'arraTableContent'=>$arraTableContent));
```

**🔴 發現 Bug**: 類型 2 使用了錯誤的內容陣列！
- 應該用 `$arraPersonalScoresheetContent`
- 實際用了 `$arraGroupScoresheetContent`

### 新 API 流程
1. 接收 `type`（整數 1-4）
2. 從預定義陣列選擇模板
3. 驗證類型有效性
4. 返回 `{name, questions}`

```php
// ChecklistService.php L240-292
$templates = [
    1 => [
        'name' => '檢核單',
        'questions' => [...]
    ],
    2 => [
        'name' => '同儕評分表',
        'questions' => [...]
    ],
    3 => [
        'name' => '組間評分表',
        'questions' => [...]
    ],
    4 => [
        'name' => '組內評分表',
        'questions' => [...]
    ]
];

if (!isset($templates[$type])) {
    throw new AppException('無效的類型', 400);
}

return $templates[$type];
```

**✅ 新 API 修正了舊版 Bug**

---

## 5. 模板內容比對

### 類型 1: 檢核單
- ✅ **內容完全一致**

### 類型 2: 同儕評分表
- 🔴 **舊版使用錯誤內容**（使用了類型 3 的內容）
- ✅ **新版內容正確**

### 類型 3: 組間評分表
- ✅ **內容完全一致**

### 類型 4: 組內評分表
- ✅ **內容完全一致**

---

## 6. 回應格式比對

### 舊程式
```json
{
  "arraTableName": "檢核單",
  "arraTableContent": ["問題1", "問題2", ...]
}
```
- 不一致的命名（`arraTable...`）
- 無標準化結構

### 新 API
```json
{
  "status": "success",
  "data": {
    "name": "檢核單",
    "questions": ["問題1", "問題2", ...]
  }
}
```

**✅ 新 API 命名更一致**
- `questions` vs `arraTableContent`
- 標準化 API 回應結構

---

## 7. 錯誤處理比對

### 舊程式
- 若傳入不存在的類型，`$arraTableContent` 可能為空
- 無明確錯誤訊息

### 新 API
- **400**: 無效的類型（不是 1-4）

**✅ 新 API 錯誤處理更明確**

---

## 8. 重複邏輯識別

### 8.1 模板資料維護 🔴 **重要**
- **位置**:
  - 舊程式: `prodb_checklists_new.php` L220-277
  - 新 API: `ChecklistService::getTemplate()` L242-285
- **問題**: 模板資料硬編碼在程式中
- **建議**: ⚠️ 若模板內容需要經常修改，可考慮移到資料庫或設定檔

**當前建議**: 
- 若模板內容相對固定 → ✅ 硬編碼可接受
- 若需要管理員動態調整 → 🔴 建議移到資料庫

### 8.2 舊版 Bug 🔴 **Critical**
- **位置**: `prodb_checklists_new.php` L267
- **Bug**: 
  ```php
  case '2':
      $arraTableName = $arraPersonalScoresheetName;
      $arraTableContent = $arraGroupScoresheetContent;  // 錯誤！
      break;
  ```
- **影響**: 類型 2（同儕評分表）返回錯誤的模板內容
- **新 API**: ✅ 已修正

---

## 9. 改進建議

### 9.1 修正舊版 Bug (✅ 已改進)
**舊版 Bug**:
```php
case '2':
    $arraTableName = $arraPersonalScoresheetName;  // 正確
    $arraTableContent = $arraGroupScoresheetContent;  // 錯誤！應該是 $arraPersonalScoresheetContent
    break;
```

**新 API 修正**:
```php
2 => [
    'name' => '同儕評分表',
    'questions' => [
        '能先介紹自己的姓名',
        '分享時聲音大小、時間控制是否合宜 (4分鐘)?',
        // ...
    ]
],
```

**影響**:
- 🔴 舊版返回錯誤的模板內容（類型 2 和類型 3 內容相同）
- ✅ 新版完全修正

### 9.2 模板資料管理 (⚪ 可選優化)
**當前做法**: 硬編碼在程式中

**替代方案**: 移到資料庫
```sql
CREATE TABLE checklist_templates (
    id INT PRIMARY KEY AUTO_INCREMENT,
    type INT,
    type_name VARCHAR(50),
    question TEXT,
    sort_order INT
);
```

**優點**:
- 管理員可動態新增/修改模板
- 不需要修改代碼

**缺點**:
- 增加資料庫查詢
- 增加複雜度

**結論**: 若模板內容相對固定，當前硬編碼實現已足夠

---

## 10. 檢查結論

| 檢查項目 | 狀態 |
|---------|------|
| SQL 查詢一致性 | ✅ 一致（兩者都無 SQL） |
| 參數驗證 | ✅ 新 API 更嚴格 |
| 業務邏輯流程 | ✅ 新 API 修正舊版 Bug |
| 回應格式 | ✅ 新 API 命名更一致 |
| 錯誤處理 | ✅ 新 API 更明確 |
| 重複邏輯處理 | ⚪ 模板資料硬編碼可接受 |

### 整體評估: **✅ 邏輯一致，修正重大 Bug**

**重大 Bug 修正**:
- 🔴 **舊版 Bug**: 類型 2（同儕評分表）返回錯誤的模板內容
  ```php
  // 舊版 L267
  $arraTableContent = $araGroupScoresheetContent;  // 應該是 $arraPersonalScoresheetContent
  ```
- ✅ **新版已修正**: 每個類型都返回正確的模板內容

**改進**:
- ✅ **參數驗證更嚴格**（型別、範圍）
- ✅ **命名更一致**（questions vs arraTableContent）
- ✅ **錯誤處理更明確**（無效類型返回 400）

**舊版嚴重問題**:
- 🔴 類型 2 模板內容錯誤，用戶會得到錯誤的模板
- ✅ 新版已完全修正
