# LiteracyController 完整邏輯檢查總結

## 1. 基本資訊

| 項目 | 內容 |
|------|------|
| **Controller** | LiteracyController |
| **端點總數** | 3 |
| **功能** | 素養導向試題查詢 (mission_type = 8) |
| **舊程式檔案** | `Literacylist_sql.php`, `newLiteracyFunction.php`, `Literacyskill.php` |
| **新 API Service** | `LiteracyService.php` |
| **新 API DAO** | `LiteracyDao.php` |
| **相關資料表** | concept_itemBank_literacy (單題), concept_itemBank_literacy_group (題組), concept_itemBank_literacy_interactive (互動), literacy_publisher_mapping (版本對應) |
| **檢查日期** | 2026-02-11 |
| **檢查依據** | API mapping 文檔 + 實際代碼掃描 |

---

## 2. 端點檢查總覽

| 端點 | 舊程式對應 | 邏輯一致性 | 主要問題 | 優先級 |
|------|-----------|-----------|---------|--------|
| list | Literacylist_sql.php::getLiteracyItem | ⚠️ 有差異 | 學年過濾缺失、type=0遺漏 | 🔴 高 |
| detail | newLiteracyFunction.php::getItemContent | ⚠️ 有差異 | 欄位不完整 | 🟡 中 |
| filters | Literacylist_sql.php::getPublisher/Grade/Theme | ✅ 基本對應 | 年級查詢差異 | 🟢 低 |

---

## 3. 🔴 嚴重問題（Critical）

### 3.1 學年過濾缺失 🔴
```php
// 舊版 (Literacylist_sql.php)
if ($_POST['gradeOption'] != 'all' || $_POST['publisherOption'] != 'all') {
    $semeYear = substr(getCurrentSemesterPart(), 0, 3);
    $seme_sql = " AND p.seme IN ({$semeYear}1, {$semeYear}2)";
    // 限制當前學年，避免顯示歷史資料
}

// 新版 (LiteracyDao::getSingleItems())
// ❌ 無 seme 過濾
```

**影響**: 🔴 **High** - 可能回傳大量歷史學年資料，導致：
- 題目列表過大
- 包含過期內容
- 影響使用者體驗

### 3.2 一般試題 (type=0) 遺漏 🔴
```php
// 舊版 (Literacylist_sql.php)
// 查詢 1: type = 1 (互動試題)
$literacy_interactive = "... WHERE type = 1 ...";

// 查詢 2: type = 0 (一般試題，但在 interactive 表中)
$literacy_interactive_non = "... WHERE type = 0 ...";

// 新版 (LiteracyDao::getInteractiveItems())
// ❌ 只查 type = 1，遺漏 type = 0
WHERE type = 1
```

**影響**: 🔴 **High** - 遺漏舊版中 type=0 的試題

### 3.3 單元列表未回傳 ⚠️
```php
// 舊版
$literacy_Info['unit'] = array_unique($unit_list);  // 回傳單元列表

// 新版
// ❌ 未回傳 unit 資訊
```

**影響**: 🟡 **Medium** - 前端可能需要單元資訊做分組顯示

---

## 4. 🟢 外包系統特性（低優先級）

> **💡 重要說明**: 核心素養評量與素養導向式題為**外包系統**，主要需求是**製作跳轉參數**，而非在平台內完整顯示題目。因此 detail 欄位不足不算問題。

### 4.1 單題 detail 欄位簡化（符合需求）
```php
// 新版 (LiteracyService::getDetail, type='s')
return [
    'item_sn',       // 跳轉用
    'item_name',     // 顯示標題
    'theme',         // 顯示類型
    'sub_subject',   // 顯示科目
    'item_type'      // 識別類型
    // ✅ 只回傳必要欄位，符合「製作跳轉參數」的需求
];

// 舊版回傳更多欄位（indicator, filename, answer等）
// 但在外包系統中，這些欄位由外包系統自行處理
```

**評估**: ✅ **符合需求** - 提供跳轉所需的基本資訊即可

### 4.2 題組 detail 簡化（符合需求）
```php
// 新版
return [
    'group_sn',    // 跳轉用
    'item_name',   // 顯示標題
    'sub_subject'  // 顯示科目
    // ✅ 跳轉到外包系統後，由外包系統顯示題組內容
];
```

**評估**: ✅ **符合需求** - 外包系統會自行處理題組內題目

### 4.3 年級查詢差異（低影響）
```php
// 舊版 (Literacylist_sql.php::getGrade)
if ($_POST['categoryOption'] == 3) {  // 資訊安全
    // 查詢 literacy_publisher_mapping
} else {
    // 一般查詢
}

// 新版 (LiteracyDao::getGrades())
// 只查 literacy_publisher_mapping（通用做法）
```

**影響**: 🟢 **Low** - 特定情境下可能有差異

---

## 5. ✅ 設計改進

### 5.1 item_sn 格式化 ✅
```php
// 新版使用前綴區分類型
"li-123"  // 互動試題
"s-123"   // 單題
"g-123"   // 題組

// 舊版：純數字，無法從 SN 判斷類型
```

**優點**: 更清晰、更易於理解

### 5.2 filters 統一端點 ✅
```php
// 舊版：3 個 action
Literacylist_sql.php?actType=getPublisher
Literacylist_sql.php?actType=getGrade
Literacylist_sql.php?actType=getTheme

// 新版：統一端點
POST /literacy/filters  // 一次回傳 publishers, grades, themes
```

**優點**: 減少 API 請求次數

### 5.3 themes 支援動態科目 ✅
```php
// 舊版：固定 subject_id = 2 (數學)
$themeGroup = "... WHERE subject_id = 2 ...";

// 新版：接受 subject_id 參數
public function getThemes($subjectId) {
    // 支援不同科目的主題查詢
}
```

**優點**: 更靈活

---

## 6. 科目→map_sn 對照

| subject_id | map_sn | 說明 |
|------------|--------|------|
| 2, 28 | IN ('3','32') | 數學 (97課綱+108課綱) |
| 3, 29 | IN ('4','33') | 自然 (97課綱+108課綱) |
| 1, 27 | IN ('10','31') | 國語 (97課綱+108課綱) |
| 77 | IN ('81') | 運算思維 |

**狀態**: ✅ 新舊版一致

---

## 7. API 參數組裝（互動試題 src）

```php
// 新版 (LiteracyService::appendApiParams)
$sMd5 = strtoupper(md5(date('Y-m-d H:i:s')));
$behavior = 'showItem';
$apiSn = base64_encode(
    md5(date('s')) . '-' . $userId . '-' . $itemSn . '-' . $behavior . '-' . $sMd5
) . '=' . $sMd5 . '=';

$item['src'] = $item['src'] . '?data=' . $apiSn . '&mission_sn=0';

// 舊版：相同邏輯
```

**狀態**: ✅ 完全一致

---

## 8. 改進建議總結

### 🔴 Critical - 必須立即修正

| 項目 | 影響 | 建議做法 |
|------|------|----------|
| 學年過濾缺失 | 回傳歷史資料過多 | 在 DAO 中加入 `seme IN ($currentYear.1, $currentYear.2)` 條件 |
| type=0 試題遺漏 | 遺漏部分試題 | `getInteractiveItems()` 改為 `WHERE type IN (0, 1)` 或分開查詢 |

### 🟡 Medium - 視需求改善

| 項目 | 影響 | 建議做法 |
|------|------|----------|
| 單元列表未回傳 | 前端分組顯示困難 | 在 `getList()` 中彙整並回傳 unit 列表（若前端需要） |

---

## 9. 舊版功能未實作（管理端）

以下舊版功能在 V3 中未實作，主要為**題目管理端**（老師/管理員）功能：

| 功能 | 舊版位置 | 說明 |
|------|---------|------|
| 新增題目 | `newLiteracyFunction::addItem` | 含檔案上傳 |
| 更新題目 | `newLiteracyFunction::updateItem` | 含檔案更新 |
| 刪除題目 | `newLiteracyFunction::deleteItem` | 軟刪除 (status=2) |
| 新增題組 | `newLiteracyFunction::addPaper` | |
| 刪除題組 | `newLiteracyFunction::deletePaper` | 軟刪除 |
| 上鎖/解鎖 | `newLiteracyFunction::lockItems` | |
| 移入/移出題組 | `newLiteracyFunction::editItemsGroup` | |
| 搜尋題目 | `newLiteracyFunction::searchItemUnlock` | |
| 競賽題目管理 | 多個 `Contest*` actions | contest_itemBank_literacy |

> 💡 **建議**: 這些管理端功能屬於不同的使用場景，如需實作建議另建 `LiteracyManageController`

---

## 10. 總體檢查結論

| 檢查項目 | 評估結果 |
|---------|----------|
| **邏輯一致性** | ⚠️ **70% 一致**（有多項缺失） |
| **安全性** | ✅ 無安全問題 |
| **功能完整性** | 🔴 **缺少關鍵功能**（學年過濾、type=0試題） |
| **優先級評估** | 🔴 **高優先級 - 建議立即修正** |

### 整體評估
**LiteracyController 主要功能是製作跳轉參數給外包系統！**

> **💡 背景**: 核心素養評量與素養導向式題為**外包系統**，平台主要提供題目列表和跳轉參數，實際題目內容由外包系統處理。

🔴 **Critical 問題**（影響跳轉功能）:
- **學年過濾缺失** - 列表可能包含過多歷史資料
- **type=0 試題遺漏** - 部分試題未出現在列表中

✅ **優點**:
- item_sn 格式化（li-/s-/g- 前綴）✅
- filters 統一端點 ✅
- themes 支援動態科目 ✅
- API 參數組裝邏輯正確 ✅
- detail 簡化符合外包系統需求 ✅

🟡 **視需求改善**:
- 單元列表未回傳（若前端需要分組）

### 結論
**符合外包系統需求，但建議修正學年過濾和type=0試題問題以優化列表顯示**
