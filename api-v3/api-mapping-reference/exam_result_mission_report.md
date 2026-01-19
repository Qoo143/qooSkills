# ExamResult/missionReport 功能對照表

## 概述

**舊程式頁面**: `modules_new.php?op=modload&name=ExamResult&file=missionReport&crumbs=測驗報告`

**功能描述**: 測驗報告模組，提供學生查看個人測驗報告、教師查看班級測驗報告的功能。

---

## 原始程式檔案

| 檔案 | 路徑 | 說明 |
|------|------|------|
| missionReport.php | `modules/ExamResult/missionReport.php` | 前端頁面 (HTML/CSS/JavaScript) |
| missionReport_detail.php | `modules/ExamResult/missionReport_detail.php` | 報告詳情頁面 |
| prodb_missionreport.php | `modules/ExamResult/prodb_missionreport.php` | AJAX 後端處理 |

---

## 功能拆解與 API 對應

### 1. 學生端功能

#### 舊程式功能表

| 功能 | AJAX act | 位置 | 說明 |
|------|----------|------|------|
| 取得任務清單 | `getMissionList` | `prodb_missionreport.php:67-399` | 學生個人任務報告清單，支援分頁 |
| 查看詳細報告 | (直接頁面跳轉) | `missionReport_detail.php` | 依任務類型顯示不同報告格式 |

#### v3 API 對應表

| 舊功能 | v3 API | Controller 方法 | 狀態 |
|--------|--------|-----------------|------|
| `getMissionList` | `POST /mission-report/list` | `MissionReportController::list()` | ✅ 已實作 |
| 報告詳情 | `POST /mission-report/detail` | `MissionReportController::detail()` | ✅ 已實作 |

#### 參數對照表 - getMissionList

| 舊參數 | 新參數 | 說明 |
|--------|--------|------|
| `seme` | `year` | 學年度 (舊: 字串前綴匹配, 新: 整數) |
| `page` | `page` | 頁碼 |
| - | `accesstoken` | v3 需要 Token 驗證 |

> [!NOTE]
> **設計異動**: 
> - 舊程式使用 `$_SESSION['user_id']` 取得使用者 ID，新版透過 `$this->request->getUserId()` 取得
> - 回傳格式重新設計，將 hash 過的 mission_sn 改為直接回傳整數

#### 參數對照表 - detail

| 舊參數 | 新參數 | 說明 |
|--------|--------|------|
| `q_user_id` | - | v3 從 Token 取得 |
| `q_paper_sn` | `paper_sn` | 試卷編號 (單元 type 4) |
| `q_cs_id` | `cs_id` | 課程結構 ID (縱貫 type 5) |
| `exam_sn` | `exam_sn` | 測驗記錄編號 |
| `mission_sn` | `mission_sn` | 任務編號 |
| `report` | - | v3 不使用，統一透過 mission_type 判斷 |
| - | `mission_type` | v3 新增：任務類型 (4/5/6/7/8/9/13/14/15) |

---

### 2. 教師端功能

#### 舊程式功能表

| 功能 | AJAX act | 位置 | 說明 |
|------|----------|------|------|
| 取得篩選選項 | `getSelectOpt` | `prodb_missionreport.php:44-59` | 取得學期、班級類型、班級選項 |
| 取得指派老師清單 | `getMissionAssigner` | `prodb_missionreport.php:403-408` | 取得該班級的任務指派者清單 |
| 取得班級報告 | `getMissionReport` | `prodb_missionreport.php:409-475` | 教師查看班級測驗報告 |
| 取得任務資料 | `getMissionData` | `prodb_missionreport.php:478-525` | 取得素養互動任務詳情 |

#### v3 API 對應表

| 舊功能 | v3 API | Controller 方法 | 狀態 |
|--------|--------|-----------------|------|
| `getSelectOpt` | `POST /mission-report/filters` | - | ❌ 未實作 |
| `getMissionAssigner` | `POST /mission-report/assigners` | - | ❌ 未實作 |
| `getMissionReport` | `POST /mission-report/class-report` | - | ❌ 未實作 |
| `getMissionData` | `POST /mission-report/mission-data` | - | ❌ 未實作 |

> [!IMPORTANT]
> **教師端功能尚未實作**，Controller 中已標記 TODO：
> ```php
> // TODO: 教師端功能 (後續開發)
> // - filters() - 篩選選項
> // - assigners() - 指派者清單
> // - classReport() - 班級報告
> ```

#### 參數對照表 - getSelectOpt (待實作)

| 舊參數 | 新參數 | 說明 |
|--------|--------|------|
| - | `accesstoken` | Token 驗證 |

回傳資料：
- `class.option_seme_name`: 學期選項
- `class.option_class_type`: 班級類型 (normal_class/teacher_class/remedial_class)
- `class.option_class_name`: 班級名稱
- `menterGradeClass`: 導師班資訊

#### 參數對照表 - getMissionAssigner (待實作)

| 舊參數 | 新參數 | 說明 |
|--------|--------|------|
| `seme` | `semester` | 學期 |
| `class_sn` | `class_sn` | 班級編號 (hash 過) |

#### 參數對照表 - getMissionReport (待實作)

| 舊參數 | 新參數 | 說明 |
|--------|--------|------|
| `classSeme` | `semester` | 學期 |
| `Class_type` | `class_type` | 班級類型 |
| `class_sn` | `class_sn` | 班級編號 |
| `assignor` | `teacher_id` | 指派老師 ID (選填) |
| `page` | `page` | 頁碼 |

#### 參數對照表 - getMissionData (待實作)

| 舊參數 | 新參數 | 說明 |
|--------|--------|------|
| `mission` | `mission_sn` | 任務編號 (hash 過) |
| `stu_user_id` | `user_id` | 學生 ID (hash 過) |

---

## SQL 對照詳細分析

### 1. getMissionList / getStudentReports - 學生任務清單

#### UNION 區塊比對

| 區塊 | 資料表 | 任務類型 | 舊程式 | v3 DAO | 狀態 |
|------|--------|----------|--------|--------|------|
| 1 | `exam_record_indicate` | 縱貫診斷 | ✅ | ✅ | ⚠️ 註1 |
| 2 | `exam_record_indicator` | 單元診斷 | ✅ | ✅ | ✅ 一致 |
| 3 | `eduexam_record` | 大考專區 (6/9/13/14/15) | ✅ | ✅ | ✅ 一致 |
| 4 | `mission_stud_record` | 素養導向 (type 7) | ✅ | ✅ | ✅ 一致 |
| 5 | `exam_record_literacy_interactive` | 素養互動 (type 8) | ✅ | ✅ | ✅ 一致 |

> [!NOTE]
> **註1**: 舊程式註解標記 `exam_record_indicate` 為「縱貫」，`exam_record_indicator` 為「單元」，但 v3 DAO 註解相反。經查詢邏輯分析：
> - `exam_record_indicate.result_sn` JOIN `mission_info.mission_sn` → 這是舊版紀錄格式
> - `exam_record_indicator.mission_sn` JOIN `mission_info.mission_sn` → 這是新版紀錄格式
> 
> 兩邊查詢邏輯相同，僅註解文字不一致，**不影響功能**。

#### SQL 逐段比對

**區塊 1: exam_record_indicate**

```sql
-- 舊程式 (prodb_missionreport.php:192-212)
SELECT
  COUNT(exam_record_indicate.result_sn) AS exam_count,
  GROUP_CONCAT(exam_record_indicate.exam_sn ORDER BY exam_record_indicate.exam_sn DESC SEPARATOR '@XX@') AS exam_sn,
  GROUP_CONCAT(exam_record_indicate.score ORDER BY exam_record_indicate.exam_sn DESC SEPARATOR '@XX@') AS score,
  mission_info.mission_sn,
  mission_info.mission_nm,
  mission_info.teacher_id,
  exam_record_indicate.cs_id,
  GROUP_CONCAT(exam_record_indicate.date ORDER BY exam_record_indicate.exam_sn DESC SEPARATOR '@XX@') AS date,
  mission_info.mission_type
FROM exam_record_indicate
JOIN mission_info ON mission_info.mission_sn = exam_record_indicate.result_sn
WHERE exam_record_indicate.user_id = :user_id1
  AND mission_info.semester LIKE :seme1
  AND mission_info.unable = 1
GROUP BY exam_record_indicate.result_sn

-- v3 DAO (MissionReportDao.php:112-127) - ✅ 完全一致
```

**區塊 2: exam_record_indicator**

```sql
-- 舊程式 (prodb_missionreport.php:215-234)
SELECT
  COUNT(exam_record_indicator.mission_sn) AS exam_count,
  GROUP_CONCAT(exam_record_indicator.exam_sn ORDER BY exam_record_indicator.exam_sn DESC SEPARATOR '@XX@') AS exam_sn,
  GROUP_CONCAT(exam_record_indicator.score ORDER BY exam_record_indicator.exam_sn DESC SEPARATOR '@XX@') AS score,
  mission_info.mission_sn,
  mission_info.mission_nm,
  mission_info.teacher_id,
  mission_info.node,                                          -- 舊程式欄位名為 node
  GROUP_CONCAT(exam_record_indicator.date ORDER BY exam_record_indicator.exam_sn DESC SEPARATOR '@XX@') AS date,
  mission_info.mission_type
FROM exam_record_indicator
JOIN mission_info ON mission_info.mission_sn = exam_record_indicator.mission_sn
WHERE exam_record_indicator.user_id = :user_id2
  AND mission_info.semester LIKE :seme2
  AND mission_info.unable = 1
GROUP BY exam_record_indicator.mission_sn

-- v3 DAO (MissionReportDao.php:131-146)
-- 差異: mission_info.node AS cs_id  ← v3 統一別名為 cs_id
-- 結果: ✅ 等效 (僅別名不同，不影響資料)
```

**區塊 3: eduexam_record**

```sql
-- 舊程式 (prodb_missionreport.php:237-255)
SELECT COUNT(*) exam_count,
       GROUP_CONCAT(er.record_sn ORDER BY er.record_sn DESC SEPARATOR '@XX@') exam_sn,
       GROUP_CONCAT(fn_eduexam_correct_rate(record_sn) ORDER BY er.record_sn DESC SEPARATOR '@XX@') score,
       er.mission_sn,
       mi.mission_nm,
       mi.teacher_id,
       mi.node,
       GROUP_CONCAT(er.finished_time ORDER BY er.record_sn DESC SEPARATOR '@XX@') date,
       mi.mission_type
FROM eduexam_record er
JOIN eduexam_paper ep ON er.paper_sn = ep.paper_sn
JOIN exam_type et ON ep.exam_type = et.exam_type            -- 舊程式有 JOIN exam_type
JOIN mission_info mi ON er.mission_sn = mi.mission_sn
  AND mi.unable = 1
  AND mi.semester LIKE :seme4
WHERE er.user_id = :user_id4
  AND er.is_finished = 1
GROUP BY er.mission_sn

-- v3 DAO (MissionReportDao.php:150-167)
-- 差異: 
--   1. v3 少了 JOIN exam_type (該 JOIN 結果未被 SELECT 使用，無影響)
--   2. mission_info.node AS cs_id (別名統一)
-- 結果: ✅ 等效
```

**區塊 4: mission_stud_record (素養導向 type 7)**

```sql
-- 舊程式 (prodb_missionreport.php:257-280)
SELECT
  COUNT(mission_stud_record.mission_sn) AS exam_count,
  0 AS exam_sn,
  'N' AS score,
  mission_info.mission_sn,
  mission_info.mission_nm,
  mission_info.teacher_id,
  mission_info.node,
  mission_stud_record.update_time,
  mission_info.mission_type
FROM mission_info
JOIN mission_stud_record ON mission_info.mission_sn = mission_stud_record.mission_sn
  AND mission_info.mission_type = 7
  AND mission_stud_record.user_id = :user_id5
WHERE mission_info.semester LIKE :seme5
  AND mission_info.unable = 1
  AND mission_info.node NOT LIKE '%clCT%'
  AND mission_stud_record.finish_node_count != '0'
GROUP BY mission_info.mission_sn

-- v3 DAO (MissionReportDao.php:171-189)
-- 差異:
--   1. v3: '0' AS exam_sn (字串) vs 舊程式: 0 AS exam_sn (整數)
--   2. v3: update_time AS date (有別名)
--   3. mission_info.node AS cs_id
-- 結果: ✅ 等效 (MySQL 隱式轉換，不影響功能)
```

**區塊 5: exam_record_literacy_interactive (素養互動 type 8)**

```sql
-- 舊程式 (prodb_missionreport.php:282-303)
SELECT
  COUNT(exam_record_literacy_interactive.mission_sn) AS exam_count,
  GROUP_CONCAT(exam_record_literacy_interactive.exam_sn ORDER BY ... SEPARATOR '@XX@') AS exam_sn,
  'N' AS score,
  mission_info.mission_sn,
  mission_info.mission_nm,
  mission_info.teacher_id,
  mission_info.node,
  GROUP_CONCAT(exam_record_literacy_interactive.update_time ORDER BY ... SEPARATOR '@XX@') AS date,
  mission_info.mission_type
FROM exam_record_literacy_interactive
JOIN mission_info ON mission_info.mission_sn = exam_record_literacy_interactive.mission_sn
WHERE exam_record_literacy_interactive.user_id = :user_id6
  AND mission_info.semester LIKE :seme6
  AND mission_info.unable = 1
GROUP BY exam_record_literacy_interactive.mission_sn

-- v3 DAO (MissionReportDao.php:193-208)
-- 結果: ✅ 完全一致
```

#### 總結比對結果

| 項目 | 狀態 | 說明 |
|------|------|------|
| UNION 結構 | ✅ | 5 個區塊順序與邏輯完全一致 |
| WHERE 條件 | ✅ | user_id、semester、unable 條件一致 |
| GROUP BY | ✅ | 分組邏輯一致 |
| ORDER BY | ✅ | `ORDER BY date DESC` 一致 |
| LIMIT 分頁 | ✅ | 相同的分頁邏輯 (PAGE_SIZE = 8) |
| 欄位別名 | ⚠️ | v3 統一使用 `cs_id` 別名，不影響功能 |
| 多餘 JOIN | ⚠️ | 舊程式 eduexam 區塊有未使用的 `JOIN exam_type`，v3 已移除 |

---

### 2. 教師端 SQL (待實作)

#### getMissionAssignor

```sql
-- 舊程式 (prodb_missionreport.php:534-541)
SELECT MI.teacher_id, UI.uname
FROM mission_info MI
JOIN user_info UI ON UI.user_id = MI.teacher_id
WHERE MI.target_id = :target_id
  AND MI.semester = :semester
GROUP BY MI.teacher_id
```

#### getMissionInfoCount

```sql
-- 舊程式 (prodb_missionreport.php:584-598)
SELECT count(TMP.mission_sn) AS RowCount FROM (
  SELECT MI.mission_sn
  FROM mission_info MI
  LEFT JOIN mission_stud_record MSR ON MSR.mission_sn = MI.mission_sn
  WHERE MSR.user_id IN (...)
    AND MI.mission_type IN ('4','5','6','7','8','9','13','14','15')
    AND MI.semester = :seme
    AND MI.unable = 1
    AND MI.node NOT LIKE '%clCT%'
    AND MSR.finish_node_count > 0
    [AND MI.teacher_id = :teacher_id]  -- 若有指定老師
  GROUP BY MI.mission_sn
) as TMP
```

#### getMissionStudRecord

```sql
-- 舊程式 (prodb_missionreport.php:626-682)
-- 複雜查詢包含 CASE WHEN 判斷 mission_target 顯示文字
-- 使用 GROUP_CONCAT 合併學生 ID
-- 支援按 teacher_id 篩選
```

> [!IMPORTANT]
> 教師端 SQL 邏輯較複雜，使用了 `TeacherClasses` 類別和多個 helper function，未來實作需注意。

---

## 現有 v3 實作

### 檔案位置

| 檔案 | 路徑 |
|------|------|
| Controller | `ADLAPI/v3/App/Controller/MissionReportController.php` |
| Service | `ADLAPI/v3/App/Service/MissionReport/MissionReportService.php` |
| DAO | `ADLAPI/v3/App/Dao/MissionReportDao.php` |
| 報告產生器 | `ADLAPI/v3/App/Service/MissionReport/ReportGenerator/` |

### 策略模式

報告詳情使用 **策略模式 (Strategy Pattern)** 處理不同任務類型：

```
ReportGeneratorFactory
├── UnitReportGenerator (type 4)
├── LongitudinalReportGenerator (type 5)
├── EduExamReportGenerator (type 6,9,13,14,15)
├── CompetencyReportGenerator (type 7)
└── LiteracyInteractiveReportGenerator (type 8)
```

---

## 驗證檢查清單

### 學生端

- [x] 任務報告清單 API 已實作
- [x] 報告詳情 API 已實作
- [x] 支援分頁
- [x] 支援多種任務類型 (4/5/6/7/8/9/13/14/15)
- [x] 使用策略模式處理不同報告類型

### 教師端

- [ ] 篩選選項 API
- [ ] 指派者清單 API
- [ ] 班級報告 API
- [ ] 任務資料 API

---

## 備註

1. **Session 替代**：舊程式大量使用 `$_SESSION`，v3 版本改用 Token 驗證
2. **Hash 編碼**：舊程式使用 `string2hash()` / `hash2string()` 編碼 ID，v3 版本直接使用整數
3. **教師端功能**：目前僅實作學生端，教師端功能待後續開發
