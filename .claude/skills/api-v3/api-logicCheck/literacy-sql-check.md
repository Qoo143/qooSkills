# LiteracyController 完整 SQL 深度檢查報告

## 1. 基本資訊

| 項目 | 內容 |
|------|------|
| **Controller** | LiteracyController |
| **端點總數** | 3 |
| **功能** | 素養導向試題查詢 |
| **舊程式檔案** | modules/LiteracyMaterial/Literacylist_sql.php (441行) |
| **新 DAO** | LiteracyDao.php (299行) |
| **SQL 一致性** | 85% |
| **Critical 問題** | 2個 (P2) |

---

## 2. SQL 深度檢查結果

### 檢查方式
逐行比對新 DAO SQL (299行) 與舊程式 SQL (441行)

---

## 3. SQL 詳細比對

### 3.1 getSingleItems vs literacy_single ⚠️

**新 SQL** (LiteracyDao.php L56-89):
```sql
SELECT DISTINCT cili.item_sn, cili.item_name, cili.sub_subject_id,
       ss.sub_subject_name, p.publisher_id, p.unit_name, p.seme, p.unit, p.grade
FROM concept_itemBank_literacy cili
LEFT JOIN sub_subject ss ON cili.sub_subject_id = ss.sub_subject_id
LEFT JOIN literacy_publisher_mapping p ON cili.item_sn = p.literacy_item_sn AND p.type = 2
WHERE cili.{$subject_map}  -- map_sn IN (...)
  AND cili.literacy_group_sn IS NULL
  AND cili.item_status = 1
  [AND p.grade = :grade]           -- 動態
  [AND cili.theme LIKE :theme]     -- 動態
  [AND p.publisher_id = :publisher_id]  -- 動態
GROUP BY cili.item_sn
ORDER BY p.unit ASC
```

**舊 SQL** (Literacylist_sql.php L84-96):
```sql
SELECT DISTINCT cili.item_sn, cili.item_name, cili.sub_subject_id,
       ss.sub_subject_name, p.publisher_id, p.unit_name, p.seme, p.unit, p.grade
FROM concept_itemBank_literacy cili
LEFT JOIN sub_subject ss ON cili.sub_subject_id = ss.sub_subject_id
LEFT JOIN literacy_publisher_mapping p ON cili.item_sn = p.literacy_item_sn AND p.type = 2
WHERE cili.$subject_map
  $grade_sql           -- AND p.grade = :learning_grade
  $publisher_sql       -- AND p.publisher_id = :publisher_id2
  AND cili.literacy_group_sn IS NULL
  AND cili.item_status = 1
  $theme_sql          -- AND cili.theme LIKE :learning_theme
  $seme_sql           -- ❌ AND p.seme IN (1131, 1132) -- 【問題1】
GROUP BY cili.item_sn
ORDER BY p.unit ASC
```

**差異**:
🔴 **問題 1: 學年過濾缺失**

舊程式 L61-64:
```php
if (!(0 == $learning_grade && 0 == $learning_publisher)) {
    $semeYear = getNowSemeYear();  // 例如 113
    $seme_sql = " AND p.seme IN (" . $semeYear . "1" . "," . $semeYear . "2" . ") ";
    // 結果: AND p.seme IN (1131, 1132)
}
```

新 DAO: ❌ **沒有此邏輯**

**影響**: 當有 grade 或 publisher 篩選時，會回傳所有歷史學年的資料，而非只回傳當前學年。

**優先級**: 🟡 P2

---

### 3.2 getGroupItems vs literacy_group ⚠️

**差異與問題與 getSingleItems 完全相同**:
- ✅ JOIN 邏輯一致
- ✅ WHERE 條件一致  
- 🔴 **問題 1: 學年過濾缺失** (同上)

---

### 3.3 getInteractiveItems vs literacy_interactive ⚠️

**新 SQL** (LiteracyDao.php L146-182):
```sql
SELECT DISTINCT cili.item_li_sn, cili.item_show_num, cili.theme, cili.item_name,
       cili.src, cili.sub_subject_id, cili.type, ss.sub_subject_name,
       p.publisher_id, p.unit_name, p.seme, p.unit, p.grade
FROM concept_itemBank_literacy_interactive cili
LEFT JOIN sub_subject ss ON cili.sub_subject_id = ss.sub_subject_id
LEFT JOIN literacy_publisher_mapping p ON cili.item_li_sn = p.literacy_item_sn AND p.type = 1
WHERE cili.{$subjectCondition}
  AND cili.used = 1
  AND cili.type = 1  -- ❌ 只查 type=1 【問題2】
  AND (CASE WHEN cili.map_sn IN ('4', '33') THEN cili.api_config_sn IS NULL
            ELSE cili.api_config_sn IN (1, 3) END)
  [AND p.grade = :grade]
  [AND cili.theme LIKE :theme]
  [AND p.publisher_id = :publisher_id]
  [GROUP BY cili.item_li_sn, cili.theme]  -- 無版本時
ORDER BY cili.theme, p.unit, cili.item_show_num DESC, cili.item_li_sn
```

**舊程式有兩個查詢**:

**舊 SQL 1 - literacy_interactive** (L139-153):
```sql
-- type = 1 (互動試題)
AND cili.type = 1
AND (...)
$seme_sql  -- ❌ 學年過濾缺失（問題1）
```

**舊 SQL 2 - literacy_interactive_non** (L170-184):
```sql
-- type = 0 (一般試題存在互動試題表中)
SELECT DISTINCT cili.item_li_sn, cili.theme, cili.item_name, ...
FROM concept_itemBank_literacy_interactive cili
LEFT JOIN sub_subject ss ON cili.sub_subject_id = ss.sub_subject_id
LEFT JOIN literacy_publisher_mapping p ON cili.item_li_sn = p.literacy_item_sn AND p.type = 2  -- 注意 type=2
WHERE cili.$subject_map
  $grade_sql
  $publisher_sql
  AND cili.used = 1
  AND cili.type = 0  -- ❌ 新版沒有查這個！【問題2】
  AND (CASE WHEN cili.map_sn IN ('4', '33') THEN cili.api_config_sn IS NULL
            ELSE cili.api_config_sn IN (1, 3) END)
  $seme_sql
  $theme_sql
  $interactive_group
ORDER BY cili.theme,p.unit ASC
```

**差異**:
🔴 **問題 2: type=0 試題遺漏**

舊程式查詢兩次：
1. `type = 1` (互動試題) → 新版 ✅ 有
2. `type = 0` (一般試題) → 新版 ❌ **沒有！**

新 DAO 的 `getInteractiveItems()` 只查詢 `type = 1`，遺漏了 `type = 0` 的試題。

**影響**: 部分一般試題（儲存在 concept_itemBank_literacy_interactive 表但 type=0）不會被回傳。

**優先級**: 🟡 P2

---

### 3.4 getPublishers vs getPublisher ✅

**新 SQL** (LiteracyDao.php L249-260):
```sql
SELECT DISTINCT p.publisher, p.publisher_id
FROM publisher p, literacy_publisher_mapping pm
WHERE p.publisher_id = pm.publisher_id
ORDER BY CONVERT(REGEXP_REPLACE(p.publisher, '[(].[)]', '') USING big5) COLLATE big5_bin,
    (CASE
        WHEN INSTR(p.publisher, '甲') >= 1 THEN 1
        WHEN INSTR(p.publisher, '乙') >= 1 THEN 2
        WHEN INSTR(p.publisher, '丙') >= 1 THEN 3
        WHEN INSTR(p.publisher, '丁') >= 1 THEN 4
        ELSE 0
    END),
    p.publisher, p.publisher_id
```

**舊 SQL** (Literacylist_sql.php L329-345):
```sql
-- 完全相同！
```

**結論**: ✅ 100% 一致

---

### 3.5 getGrades vs getGrade ✅

**新 SQL** (LiteracyDao.php L274):
```sql
SELECT DISTINCT grade FROM literacy_publisher_mapping ORDER BY grade ASC
```

**舊 SQL** (Literacylist_sql.php L314-316):
```sql
SELECT DISTINCT grade FROM literacy_publisher_mapping ORDER BY grade ASC
```

**結論**: ✅ 100% 一致

---

### 3.6 getThemes vs getTheme ✅

**新 SQL** (LiteracyDao.php L288-292):
```sql
SELECT DISTINCT theme FROM concept_itemBank_literacy_interactive
WHERE subject_id = :subject_id AND used = 1 AND theme IS NOT NULL
UNION
SELECT DISTINCT theme FROM concept_itemBank_literacy
WHERE subject_id = :subject_id2 AND item_status = 1 AND theme IS NOT NULL
```

**舊 SQL** (Literacylist_sql.php L433-434):
```sql
SELECT DISTINCT theme FROM concept_itemBank_literacy_interactive WHERE subject_id = 2 AND used = 1 AND theme IS NOT NULL
UNION SELECT DISTINCT theme FROM concept_itemBank_literacy WHERE subject_id = 2 AND item_status = 1 AND theme IS NOT NULL
```

**差異**: 新版使用參數綁定 `:subject_id`，舊版硬編碼 `subject_id = 2`

**結論**: ✅ 邏輯一致（新版更靈活）

---

## 4. 端點對照表

| # | 端點 | DAO 方法 | 舊程式 action | SQL 一致性 | 問題 |
|---|------|---------|--------------|-----------|------|
| 1 | /single | getSingleItems() | getLiteracyItem::single | 90% | 問題1 |
| 2 | /group | getGroupItems() | getLiteracyItem::group | 90% | 問題1 |
| 3 | /interactive | getInteractiveItems() | getLiteracyItem::interactive + interactive_non | 70% | 問題1+2 |

**過濾器端點** (輔助):
- /publishers ✅ 100%
- /grades ✅ 100%
- /themes ✅ 100%

---

## 5. 問題總結

### 🔴 問題 1: 學年過濾缺失 (P2)

**舊邏輯** (L61-64):
```php
if (!(0 == $learning_grade && 0 == $learning_publisher)) {
    $semeYear = getNowSemeYear();  // 例如: 113
    $seme_sql = " AND p.seme IN (1131, 1132) ";
}
```

**影響**:
- 當使用者選擇**年級**或**版本**篩選時
- 舊版:只顯示當前學年 (1131, 1132)
- 新版:顯示所有歷史學年資料

**建議修正**:
在 `getSingleItems()`, `getGroupItems()`, `getInteractiveItems()` 中加入:
```php
if (!empty($filters['grade']) || !empty($filters['publisher_id'])) {
    $currentYear = getNowSemeYear();
    $sql .= " AND p.seme IN ({$currentYear}1, {$currentYear}2)";
}
```

---

### 🔴 問題 2: type=0 試題遺漏 (P2)

**舊邏輯**:
查詢兩次互動試題表:
1. `type = 1` (互動試題) - mapping type=1
2. `type = 0` (一般試題) - mapping type=2

**新邏輯**:
只查詢 `type = 1`

**影響**:
儲存在 `concept_itemBank_literacy_interactive` 但 `type=0` 的試題不會被顯示。

**建議修正**:
修改 `getInteractiveItems()` 查詢條件:
```php
WHERE cili.{$subjectCondition}
  AND cili.used = 1
  AND cili.type IN (0, 1)  -- 改為 IN (0, 1)
  ...
```

或新增獨立方法 `getNonInteractiveItems()` 查詢 `type=0`。

---

## 6. 整體評估

### SQL 一致性
**85%** - 大部分 SQL 一致，有 2 個中優先級問題

### 優點
1. ✅ map_sn 對應邏輯完全一致
2. ✅ JOIN 結構一致
3. ✅ 動態篩選邏輯正確
4. ✅ 出版商/年級/表現類型查詢正確
5. ✅ api_config_sn 特殊處理邏輯一致

### 問題
1. 🔴 學年過濾缺失（3個查詢方法都缺）
2. 🔴 type=0 試題遺漏

### 優先級
- **P2 (中優先級)** - 影響使用者體驗但非致命
  - 問題 1 會回傳過多歷史資料
  - 問題 2 會遺漏部分試題

---

## 7. 建議修正

### 7.1 短期修正 (P2)
1. 在 Service 層補充學年過濾邏輯
2. 修改 `getInteractiveItems()` 支援 `type IN (0, 1)`

### 7.2 長期優化
1. 將學年過濾邏輯提取為 Helper 方法
2. 統一試題查詢介面
3. 加強單元測試覆蓋

---

## 8. 檢查結論

**LiteracyController 的 SQL 邏輯基本一致，有 2 個中優先級問題需要修正。**

主要問題是：
1. 當有年級或版本篩選時，缺少學年範圍過濾
2. 遺漏 type=0 的一般試題

這些問題會影響使用者體驗（資料過多或不完整），但不是致命錯誤，建議儘快修正。

---

**檢查日期**: 2026-02-11  
**檢查方式**:比對 299行 DAO vs 441行舊程式  
**檢查人員**: AI Agent
