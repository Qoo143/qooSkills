# API-V3 對照表：Literacy Controller

## Controller 資訊
- **檔案**: `ADLAPI/v3/App/Controller/LiteracyController.php`
- **Service**: `ADLAPI/v3/App/Service/LiteracyService.php`
- **DAO**: `ADLAPI/v3/App/Dao/LiteracyDao.php`

---

## 端點對照總覽

| # | V3 端點 | HTTP | 舊檔案 | 舊動作 (actType) | 狀態 |
|---|---------|------|--------|-----------------|------|
| 1 | `/literacy/list` | POST | `modules/LiteracyMaterial/Literacylist_sql.php` | `getLiteracyItem` | ⚠️ 有差異 |
| 2 | `/literacy/detail` | POST | `modules/LiteracyMaterial/newLiteracyFunction.php` | `getItemContent` / interactive detail | ✅ 基本對應 |
| 3 | `/literacy/filters` | POST | `modules/LiteracyMaterial/Literacylist_sql.php` | `getPublisher` + `getGrade` + `getTheme` | ✅ 基本對應 |

---

## 1. 素養題目列表 `/literacy/list`

### 參數對照
| V3 參數 | 舊參數 | 說明 |
|---------|--------|------|
| `subject_id` (required) | `$_POST['subject_get']` | 科目 ID (1=國語, 2=數學, 3=自然, 77=運算思維) |
| `grade` (optional) | `$_POST['gradeOption']` | 年級 |
| `theme` (optional) | `$_POST['themeOption']` | 表現類型 |
| `publisher_id` (optional) | `$_POST['publisherOption']` | 版本 ID |

### 邏輯對照

#### 科目→map_sn 對照
| V3 (`LiteracyDao::getMapSnCondition`) | 舊 (`Literacylist_sql.php`) | 一致 |
|----------------------------------------|---------------------------|------|
| subject 2,28 → `IN ('3','32')` | 同上 | ✅ |
| subject 3,29 → `IN ('4','33')` | 同上 | ✅ |
| subject 1,27 → `IN ('10','31')` | 同上 | ✅ |
| subject 77 → `IN ('81')` | 同上 | ✅ |

#### 三種查詢
| 類型 | V3 | 舊 | 差異 |
|------|------|------|------|
| 素養單題 | `LiteracyDao::getSingleItems()` | `literacy_single` SQL | ✅ SQL 基本一致 |
| 素養題組 | `LiteracyDao::getGroupItems()` | `literacy_group` SQL | ✅ SQL 基本一致 |
| 互動試題 | `LiteracyDao::getInteractiveItems()` | `literacy_interactive` SQL | ✅ SQL 基本一致 |

#### API 參數組裝（互動試題 src）
- ✅ V3 `appendApiParams()` 和舊版邏輯一致：`base64_encode(md5(date('s')) . '-' . $userId . '-' . $itemsn . '-' . $behavior . '-' . $sMd5)`

### 缺失邏輯
- ⚠️ **學年過濾 (seme_sql)**: 舊版當有 grade 或 publisher 篩選時，會加上 `$seme_sql = " AND p.seme IN ({$semeYear}1, {$semeYear}2)"` 限制當前學年；V3 **無此過濾**，可能回傳歷史學年資料
- ⚠️ **一般試題類型 (interactive_non)**: 舊版有 `literacy_interactive_non` 查詢 (`type = 0`)，取得互動試題資料表中的一般試題；V3 **只查 `type = 1`**，遺漏 type=0 的試題
- ⚠️ **單元列表 (unit)**: 舊版回傳中包含 `literacy_Info['unit']`（從所有題目資料中彙整 seme/grade/unit/unit_name）；V3 **未回傳單元列表**
- ⚠️ **版本時的單元查詢**: 舊版選擇版本後另外查 `literacy_publisher_mapping` 取得單元列表；V3 無此功能

---

## 2. 素養題目詳情 `/literacy/detail`

### 參數對照
| V3 參數 | 舊參數 | 說明 |
|---------|--------|------|
| `item_sn` (required, string) | `$_POST['item_sn']` | 格式: `"li-123"` 互動, `"s-123"` 單題, `"g-123"` 題組 |

### 邏輯對照

#### item_sn 解析
- V3 使用 `parseItemSn()` 正規表達式 `/^(li|s|g)-(\d+)$/` 解析
- 舊版無此格式，直接傳數字 item_sn
- V3 兼容純數字（預設為互動試題）

#### 三種 detail 查詢
| 類型 | V3 | 舊 | 差異 |
|------|------|------|------|
| 互動試題 (li) | `getInteractiveItemDetail()` → 基本欄位 | `getLiteracyInteractiveItem` (Literacylist_sql) | ⚠️ V3 詳情較完整（含 questions_word/img, solution_word/img 解析） |
| 單題 (s) | `getSingleItemDetail()` → item_sn, item_name, theme, sub_subject | `getItemContent` (newLiteracyFunction) | ⚠️ **V3 回傳較少欄位**：舊版回傳含 indicator, situation, item_type, filename, options, answer, core, crossSubject 等完整資訊 |
| 題組 (g) | `getGroupItemDetail()` → group_sn, item_name, sub_subject | `getPaperContent` (newLiteracyFunction) | ⚠️ **V3 回傳較少**：舊版回傳題組內所有題目 |

#### 互動試題格式化
- V3 `formatInteractiveDetail()` 解析 `;;;` 分隔的欄位 (`questions_word`, `questions_img`, `solution_word`, `solution_img`)
- V3 組合 `core_code` + `learning_content` 為 `core` 欄位
- 舊版在 `getLiteracyInteractiveItem` 的 SQL 中直接用 CONCAT/CASE 處理

### 缺失邏輯
- ⚠️ **單題詳情不完整**: V3 只回傳 5 個欄位；舊版 `getItemContent` 回傳含 indicator（含 indicate_name 對照）、filename（題目圖片）、op_filename（選項圖片）、answer、situation、thoughtType、learningPerformance、core、crossSubject 等完整欄位
- ⚠️ **題組詳情不完整**: V3 只回傳題組基本資訊；舊版 `getPaperContent` 回傳題組包含的所有題目列表

---

## 3. 篩選選項 `/literacy/filters`

### 參數對照
| V3 參數 | 舊參數 | 說明 |
|---------|--------|------|
| `subject_id` (optional, default 2) | `$_POST['subject_get']` (各 action 各自帶) | 科目 ID |

### 邏輯對照
| 篩選項 | V3 | 舊 | 差異 |
|--------|------|------|------|
| 版本 | `LiteracyDao::getPublishers()` | `getPublisher` action | ✅ SQL 一致（含 big5 排序） |
| 年級 | `LiteracyDao::getGrades()` | `getGrade` action | ⚠️ 舊版 `getGrade` 根據 categoryOption 有不同查詢（categoryOption==3 查 publisher_mapping）；V3 只查 literacy_publisher_mapping |
| 表現類型 | `LiteracyDao::getThemes()` | `getTheme` action | ⚠️ 舊版 `getTheme` 固定 `subject_id = 2`（數學）；V3 接受 `$subjectId` 參數，更靈活 |

---

## 關鍵差異總結

### 🔴 嚴重問題

1. **學年過濾缺失**: V3 無 `seme_sql` 過濾，可能回傳歷史學年資料導致題目列表過大或包含過期內容
2. **一般試題 (type=0) 遺漏**: `getInteractiveItems()` 只查 `type = 1`，遺漏舊版 `literacy_interactive_non` (type=0) 的試題。這些題目在舊版中也會顯示在列表中

### 🟡 中度問題

3. **單題 detail 欄位不足**: 缺少 indicator, filename, op_filename, answer, situation, thoughtType, learningPerformance, core, crossSubject 等欄位
4. **題組 detail 不含子題**: 未回傳題組內的題目列表
5. **單元列表未回傳**: 前端可能需要 unit 資訊做分組顯示
6. **年級查詢差異**: 舊版對 categoryOption=3（資訊安全）有不同查詢路徑

### 🟢 設計改進

7. **item_sn 格式化**: V3 使用 `"li-123"`, `"s-123"`, `"g-123"` 前綴區分類型，比舊版更清晰
8. **filters 統一端點**: 舊版 publisher/grade/theme 分三個 action；V3 合併為一個 `/filters` 端點
9. **themes 支援動態科目**: V3 接受 subject_id 參數，不像舊版固定 subject_id=2

### 📋 舊版功能未實作（管理端）

以下舊版功能在 V3 中未實作，主要為題目管理端（老師/管理員）功能：

| 舊版功能 | 舊版位置 | 說明 |
|---------|---------|------|
| 新增題目 | `newLiteracyFunction::addItem` | 含檔案上傳 |
| 更新題目 | `newLiteracyFunction::updateItem` | 含檔案更新 |
| 刪除題目 | `newLiteracyFunction::deleteItem` | 軟刪除 (status=2) |
| 新增題組 | `newLiteracyFunction::addPaper` | |
| 刪除題組 | `newLiteracyFunction::deletePaper` | 軟刪除 |
| 上鎖/解鎖 | `newLiteracyFunction::lockItems` / `unlockItems` | |
| 移入/移出題組 | `newLiteracyFunction::editItemsGroup` / `removeItems` | |
| 搜尋題目 | `newLiteracyFunction::searchItemUnlock` | |
| 競賽題目管理 | 多個 `Contest*` actions | contest_itemBank_literacy 相關 |
| 運算思維單元 | `getComputationalUnit` | publisher_mapping 查詢 |
| 資訊安全 | `getTech` | subject_id=54 特殊查詢 |
| 學習階段/情境/思考類型等 | 多個 get* actions | ref_LiteracyMaterial.php 參考資料 |

> 注意：這些管理端功能屬於不同的使用場景，如需實作建議另建 `LiteracyManageController`
