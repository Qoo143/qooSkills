# API-V3 對照表 - missionReport.php

**舊檔案路徑**：`modules/ExamResult/missionReport.php`
**相關後端檔案**：`modules/ExamResult/prodb_missionreport.php`
**相關前端檔案**：`modules/ExamResult/missionReport_detail.php`
**分析日期**：2026-02-03
**主要 Controller**：MissionReportController（現有）
**分析者**：Claude Sonnet Agent

---

## 檔案概述

此模組提供「任務測驗報告」功能，分為學生端和教師端：

### 學生端功能
- 查看自己的任務測驗報告（依學年度分頁）
- 查看歷次測驗成績與比較
- 點擊查看詳細診斷報告

### 教師端功能
- 選擇學期、班級類型、班級、指派老師
- 查看班級學生的任務完成情況
- 查看學生測驗成績與進步退步趨勢
- 查看全班成績（針對任務類型 7）

---

## 端點對照清單

| 選擇 | 舊端點 | Controller 方法 | 狀態 | Controller | 說明 |
|:----:|--------|----------------|:----:|-----------|------|
| [x] | Page Load: 學生報告清單 | list() | 已完成 | MissionReportController（現有） | 學生端已完成 |
| [x] | Action: getSelectOpt | getFilters() | 已完成 | MissionReportController（現有） | 教師端篩選選項 |
| [x] | Action: getMissionList | list() | 已完成 | MissionReportController（現有） | 學生任務報告清單 |
| [x] | Action: getMissionAssigner | getAssigners() | 已完成 | MissionReportController（現有） | 取得指派老師清單 |
| [x] | Action: getMissionReport | classReport() | 已完成 | MissionReportController（現有） | 教師端班級報告 |
| [x] | Action: getMissionData | getMissionItems() | 已完成 | MissionReportController（現有） | 素養任務題目列表 |

---

## 功能詳細分析

### 1. Action: getSelectOpt

**舊代碼位置**：prodb_missionreport.php Line 44-59

**業務邏輯**：
教師端取得篩選選項（學期、班級類型、班級、導師班級）

**參數分析**：
```php
// 舊參數
無參數，直接取得當前教師的班級資訊
```

**對應 Validator 規則**：
```php
[
    // 無參數需求
]
```

**SQL 查詢**：
```sql
-- 1. 取得教師導師班級
SELECT grade, class 
FROM user_info 
WHERE user_id = :user_id 
  AND used = 1

-- 2. 使用 TeacherClasses::getAllClasses() 取得：
-- - option_seme_name: 學期選項
-- - option_class_type: 班級類型（一般、自組、學扶）
-- - option_class_name: 班級名稱
```

**權限要求**：
- access_level: 21, 25（教師）
- 只能查詢自己的班級資料

**回傳格式**：
```json
{
    "class": {
        "option_seme_name": {
            "11501": "115學年度 上學期",
            "11502": "115學年度 下學期"
        },
        "option_class_type": {
            "11501": {
                "normal_class": "一般班級",
                "teacher_class": "自組班級",
                "remedial_class": "學習扶助班級"
            }
        },
        "option_class_name": {
            "11501": {
                "normal_class": [
                    {"class_sn": "hash_value", "class_name": "五年一班"}
                ]
            }
        }
    },
    "menterGradeClass": "hash_value"
}
```

**對應 API 端點**：`POST /api/v3/mission-report/filters`

**實作建議**：
- DAO: MissionReportDao::getTeacherClasses($teacherId)
- Service: MissionReportService::getFilters($teacherId)
- Controller: MissionReportController::getFilters()

---
### 2. Action: getMissionList

**舊代碼位置**：prodb_missionreport.php Line 67-399

**業務邏輯**：
學生端取得自己的任務測驗報告清單，支援學年度篩選、分頁

**參數分析**：
```php
// 舊參數
$seme = $_POST['seme'];     // 學年度，如：115
$page = $_POST['page'];     // 頁碼，預設 1
```

**對應 Validator 規則**：
```php
[
    'year' => 'integer|min:100|max:200|default:' . DateHelper::getYearSeme(),
    'page' => 'integer|min:1|max:9999|default:1'
]
```

**SQL 查詢**：
此查詢為 5 個 UNION 合併多種任務類型的測驗記錄

1. exam_record_indicate（縱貫診斷測驗）
2. exam_record_indicator（單元診斷測驗）
3. eduexam_record（學力測驗、會考等）
4. mission_stud_record（核心素養任務，類型 7）
5. exam_record_literacy_interactive（學科素養互動測驗，類型 8）

詳細 SQL 請見舊檔案 Line 69-305

**權限要求**：
- access_level: 1, 4, 9（學生）
- 只能查詢自己的測驗記錄

**回傳格式**：
```json
{
    "now_Page": 1,
    "page_num": 3,
    "TotalData": 24,
    "maxCount": 5,
    "mission": {
        "hash_mission_sn_1": {
            "title": "【單元診斷測驗】數學複習任務",
            "count": 3,
            "mission_type": "4",
            "teacher_name": "王小明 老師",
            "detail": [
                {
                    "exam_sn": 12345,
                    "score": 85,
                    "date": "2026-01-15 14:30:00",
                    "url": "&report=1&q_user_id=hash&..."
                }
            ]
        }
    }
}
```

**對應 API 端點**：`POST /api/v3/mission-report/list`

**實作建議**：
- DAO: MissionReportDao::getStudentReports($userId, $year, $page, $pageSize)
- Service: MissionReportService::getStudentReports($userId, $year, $page)
- Controller: MissionReportController::list() ✅ 已實作

**特殊處理邏輯**：
1. 任務類型 7（核心素養）不顯示分數，僅顯示完成狀態
2. 任務類型 8（學科素養）固定顯示 1 次，避免表格過長
3. 指派人顯示邏輯：
   - 自己指派 → "自己指派"
   - 教師指派 → "xxx 老師"
   - 家長指派 → "xxx 家長"

---

### 3. Action: getMissionAssigner

**舊代碼位置**：prodb_missionreport.php Line 403-408, Function Line 529-566

**業務邏輯**：
教師端選擇班級後，取得曾指派任務給該班的所有老師清單

**參數分析**：
```php
// 舊參數
$class_sn = hash2string($_POST['class_sn'], false);  // 班級 SN (hash)
$seme = $_POST['seme'];                               // 學期，如：11501
```

**對應 Validator 規則**：
```php
[
    'class_sn' => 'required|string',
    'semester' => 'required|string|regex:/^\d{5}$/'
]
```

**SQL 查詢**：
```sql
SELECT MI.teacher_id,
       UI.uname
FROM mission_info MI
JOIN user_info UI ON UI.user_id = MI.teacher_id
WHERE MI.target_id = :target_id
  AND MI.semester = :semester
GROUP BY MI.teacher_id
```

**權限要求**：
- access_level: 21, 25（教師）
- 只能查詢自己有權限的班級

**回傳格式**：
```json
{
    "status": true,
    "result": [
        {
            "teacher_id": "hash_value",
            "uname": "王小明"
        }
    ]
}
```

**對應 API 端點**：`POST /api/v3/mission-report/assigners`

**實作建議**：
- DAO: MissionReportDao::getMissionAssigners($classSn, $semester)
- Service: MissionReportService::getAssigners($classSn, $semester)
- Controller: MissionReportController::getAssigners()

---
### 4. Action: getMissionReport

**舊代碼位置**：prodb_missionreport.php Line 409-475

**業務邏輯**：
教師端查看班級學生的任務完成報告，顯示每個學生的最新測驗成績與進步退步趨勢

**參數分析**：
```php
// 舊參數
$classSeme = $_POST['classSeme'];      // 學期，如：11501
$Class_type = $_POST['Class_type'];    // 班級類型：normal_class, teacher_class, remedial_class
$class_sn = $_POST['class_sn'];        // 班級 SN (hash)
$assignor = $_POST['assignor'];        // 指派老師 ID (hash)，空值表示全部老師
$page = $_POST['page'];                // 頁碼
```

**對應 Validator 規則**：
```php
[
    'semester' => 'required|string|regex:/^\d{5}$/',
    'class_type' => 'required|in:normal_class,teacher_class,remedial_class',
    'class_sn' => 'required|string',
    'assignor' => 'string|default:',
    'page' => 'integer|min:1|default:1'
]
```

**SQL 查詢**：
```sql
-- 1. 取得總筆數（使用 getMissionInfoCount 函數）
SELECT count(TMP.mission_sn) AS RowCount 
FROM (
    SELECT MI.mission_sn
    FROM mission_info MI
    LEFT JOIN mission_stud_record MSR ON MSR.mission_sn = MI.mission_sn
    WHERE MSR.user_id IN (:student_ids)
      AND MI.mission_type IN (4, 5, 6, 7, 8, 9, 13, 14, 15)
      AND MI.semester = :semester
      AND MI.unable = 1
      AND MI.node NOT LIKE '%clCT%'
      AND MSR.finish_node_count > 0
      [AND MI.teacher_id = :teacher_id]
    GROUP BY MI.mission_sn
) as TMP

-- 2. 取得任務資料（使用 getMissionStudRecord 函數）
-- 詳細 SQL 請見 Line 626-682
```

**權限要求**：
- access_level: 21, 25（教師）
- 只能查詢自己教授的班級

**回傳格式**：
```json
{
    "now_Page": 1,
    "TotalData": 15,
    "page_num": 2,
    "arrAllStudentList": [
        {
            "user_id": "hash_value",
            "uname": "王小明",
            "seme_num": 1
        }
    ],
    "mission": {
        "hash_mission_sn": {
            "title": "【單元診斷測驗】數學複習",
            "mission_type": "4",
            "teacher_name": "張老師",
            "start_time": "2026-01-01 00:00:00",
            "end_time": "2026-01-31 23:59:59",
            "mission_target": "五年一班",
            "detail": {
                "hash_student_id": [
                    {
                        "exam_sn": 12345,
                        "score": 85,
                        "date": "2026-01-15",
                        "url": "modules_new.php?op=modload&...",
                        "student_id": "hash",
                        "mission_sn": "hash"
                    }
                ]
            }
        }
    }
}
```

**對應 API 端點**：`POST /api/v3/mission-report/class-report`

**實作建議**：
- DAO: MissionReportDao::getClassReport($classSn, $classType, $semester, $assignor, $page)
- Service: MissionReportService::getClassReport($params)
- Controller: MissionReportController::classReport()

**特殊處理邏輯**：
1. 根據 class_type 取得學生清單（使用 TeacherClasses::getStudentList）
2. 每頁顯示 8 個任務（PAGE_DISPLAY_MISSION_QUANTITY = 8）
3. 任務類型 7 需特殊處理（核心素養），顯示「查看」按鈕而非分數
4. 任務類型 8（學科素養）顯示「查看」按鈕連結到互動測驗

---

### 5. Action: getMissionData

**舊代碼位置**：prodb_missionreport.php Line 478-526

**業務邏輯**：
取得素養任務（類型 8）的題目清單，顯示學生完成狀態

**參數分析**：
```php
// 舊參數
$mission = hash2string($_POST['mission']);       // 任務 SN
$stu_user_id = hash2string($_POST['stu_user_id']); // 學生 user_id
```

**對應 Validator 規則**：
```php
[
    'mission_sn' => 'required|integer',
    'user_id' => 'required|string'
]
```

**SQL 查詢**：
```sql
-- 1. 取得任務的題目節點
SELECT mi.mission_sn, mi.node, erli.user_id, erli.exam_sn, erli.item_li_sn
FROM mission_info AS mi
LEFT JOIN exam_record_literacy_interactive AS erli
  ON erli.mission_sn = mi.mission_sn
  AND erli.exam_sn IN (
      SELECT MAX(exam_sn)
      FROM exam_record_literacy_interactive
      GROUP BY item_li_sn
  )
WHERE mi.mission_sn = :mission_sn

-- 2. 取得題目名稱與學生完成狀態
SELECT cili.item_name, cili.item_li_sn, erli.item_li_sn, erli.user_id, erli.exam_sn
FROM concept_itemBank_literacy_interactive AS cili
LEFT JOIN exam_record_literacy_interactive AS erli
  ON erli.exam_sn IN (
      SELECT MAX(exam_record_literacy_interactive.exam_sn)
      FROM exam_record_literacy_interactive
      WHERE mission_sn = :mission_sn AND user_id = :stu_user_id
      GROUP BY item_li_sn
  )
  AND erli.item_li_sn = cili.item_li_sn
WHERE cili.item_li_sn IN (:item_list)
```

**權限要求**：
- access_level: 1, 4, 9（學生）或 21, 25（教師）
- 學生只能查詢自己的資料
- 教師可查詢班上學生的資料

**回傳格式**：
```json
[
    {
        "item_name": "閱讀理解測驗一",
        "item_li_sn": "12345",
        "exam_sn": "base64_encoded_value"
    },
    {
        "item_name": "閱讀理解測驗二",
        "item_li_sn": "12346",
        "exam_sn": null
    }
]
```

**對應 API 端點**：`POST /api/v3/mission-report/mission-items`

**實作建議**：
- DAO: MissionReportDao::getMissionItems($missionSn, $userId)
- Service: MissionReportService::getMissionItems($missionSn, $userId)
- Controller: MissionReportController::getMissionItems()

---
## 資料表關聯

**主要資料表**：
- mission_info - 任務資訊
- exam_record_indicate - 縱貫診斷測驗記錄
- exam_record_indicator - 單元診斷測驗記錄
- eduexam_record - 學力測驗記錄
- mission_stud_record - 學生任務執行記錄
- exam_record_literacy_interactive - 素養互動測驗記錄
- concept_itemBank_literacy_interactive - 素養題庫

**外鍵關係**：
```
mission_info.mission_sn → exam_record_indicate.result_sn
mission_info.mission_sn → exam_record_indicator.mission_sn
mission_info.mission_sn → eduexam_record.mission_sn
mission_info.mission_sn → mission_stud_record.mission_sn
mission_info.mission_sn → exam_record_literacy_interactive.mission_sn
mission_info.teacher_id → user_info.user_id
```

**重要欄位說明**：
- mission_info.mission_type: 任務類型（4=單元, 5=縱貫, 6/9/13/14/15=學力測驗, 7=核心素養, 8=學科素養）
- mission_info.unable: 任務啟用狀態（1=啟用, 0=停用）
- mission_info.semester: 學期（如：11501）
- mission_info.target_type: 指派對象類型（I=個人, C=班級）
- mission_info.class_type: 班級類型（1=自組, 2=學扶, NULL=一般）
- mission_stud_record.finish_node_count: 完成節點數

---

## 安全性檢查

- [x] SQL 使用 prepared statements
- [x] 輸出使用 preventXss() / htmlspecialchars()
- [x] 權限檢查：學生只能查詢自己的資料，教師只能查詢自己班級的資料
- [x] CSRF token 驗證（Line 19-24）
- [x] XSS 過濾：$_POST, $_REQUEST 全域過濾（Line 26-31）
- [ ] Hash 參數驗證：使用 hash2string() 解碼，需驗證解碼後的值

**潛在安全問題**：
1. Line 492: 使用 split() 函數（PHP 7+ 已棄用），應改用 explode()
2. Line 498: SQL IN clause 使用 array2sqlIN()，需確認此函數有正確的 SQL injection 防護
3. Line 509: SQL 動態拼接 IN clause，建議改用參數綁定

---

## 開發優先級

### 1. 高優先級（核心功能）
- [x] list() - 學生任務報告清單（✅ 已完成）
- [x] detail() - 報告詳情（✅ 已完成）
- [ ] classReport() - 教師班級報告（教師端核心功能）

### 2. 中優先級（常用功能）
- [ ] getFilters() - 教師端篩選選項
- [ ] getAssigners() - 指派老師清單

### 3. 低優先級（輔助功能）
- [ ] getMissionItems() - 素養任務題目清單（僅任務類型 8 使用）

---

## 任務類型對照表

| mission_type | 名稱 | 資料表 | 特殊處理 |
|--------------|------|--------|---------|
| 4 | 單元診斷測驗 | exam_record_indicator | 有 paper_sn |
| 5 | 縱貫診斷測驗 | exam_record_indicate | 有 cs_id |
| 6 | 學力測驗 | eduexam_record | 使用 record_sn |
| 7 | 核心素養 | mission_stud_record | 無分數，顯示「查看」 |
| 8 | 學科素養 | exam_record_literacy_interactive | 無分數，顯示「查看」 |
| 9 | 會考 | eduexam_record | 使用 record_sn |
| 13 | 學測 | eduexam_record | 使用 record_sn |
| 14 | 統測 | eduexam_record | 使用 record_sn |
| 15 | 指考 | eduexam_record | 使用 record_sn |

---

## 前後端對應關係

### JavaScript 變數對應
```javascript
// 前端 JavaScript (missionReport.php)
var router = '<?php echo encodeAjaxUrlBase64('ExamResult', 'prodb_missionreport'); ?>';
var isTeacher = <?= $sIsTeacher ?>;
var nowSeme = <?= getNowSemeYear(); ?>;

// 對應 API 參數
router → 不需要（直接呼叫 API endpoint）
isTeacher → 從 JWT token 取得 access_level
nowSeme → 從 DateHelper::getYearSeme() 取得
```

### AJAX 請求對應
```javascript
// 舊版 AJAX
$.post('../../turnpage.php', {
    router: router,
    act: 'getMissionList',
    seme: semes,
    page: pages
}, function(result) { ... });

// 新版 API
POST /api/v3/mission-report/list
Content-Type: application/json
Authorization: Bearer {token}

{
    "year": 115,
    "page": 1
}
```

---

## 開發記錄

### 2026-02-03 (初始分析)
- [x] 完成舊檔案深度分析
- [x] 識別 6 個主要端點
- [x] 建立完整 SQL 查詢文檔
- [x] 分析 5 種任務類型的資料表關聯
- [x] 標註已完成的功能（學生端 list, detail）
- [x] 規劃待開發功能（教師端功能）

### 2026-02-03 (教師端開發完成)
- [x] 完成 getFilters() - 教師端篩選選項
  - DAO: getTeacherMentorClass()
  - Service: getFilters() 使用 TeacherClasses::getAllClasses()
  - Controller: getFilters()

- [x] 完成 getAssigners() - 取得指派老師清單
  - DAO: getMissionAssigners()
  - Service: getAssigners() 含 hash 編解碼
  - Controller: getAssigners()

- [x] 完成 classReport() - 教師端班級報告
  - DAO: getClassMissionCount(), getClassMissionReports(), getStudentExamRecords()
  - Service: getClassReport() 使用 TeacherClasses::getStudentList()
  - Controller: classReport()
  - 支援 3 種班級類型：normal_class, teacher_class, remedial_class

- [x] 完成 getMissionItems() - 素養任務題目列表
  - DAO: getMissionNodes(), getLiteracyMissionItems()
  - Service: getMissionItems() 含節點解析
  - Controller: getMissionItems()

### 測試建議
1. 學生端測試：
   ```bash
   # 取得學生任務報告清單
   POST /api/v3/mission-report/list
   {
     "year": 115,
     "page": 1
   }

   # 取得報告詳情
   POST /api/v3/mission-report/detail
   {
     "mission_type": 4,
     "mission_sn": 12345,
     "exam_sn": 67890,
     "paper_sn": 111
   }
   ```

2. 教師端測試：
   ```bash
   # 取得篩選選項
   POST /api/v3/mission-report/filters
   {}

   # 取得指派老師清單
   POST /api/v3/mission-report/assigners
   {
     "class_sn": "hash_value",
     "semester": "11501"
   }

   # 取得班級報告
   POST /api/v3/mission-report/class-report
   {
     "semester": "11501",
     "class_type": "normal_class",
     "class_sn": "hash_value",
     "assignor": "",
     "page": 1
   }

   # 取得素養任務題目
   POST /api/v3/mission-report/mission-items
   {
     "mission_sn": 12345,
     "user_id": "hash_value"
   }
   ```

### 資料庫優化建議
- 考慮將 5 個 UNION 查詢改為 VIEW
- 為 mission_info.semester 建立索引
- 為 exam_record_*.user_id 建立索引

---

## 參考資料

- 現有 Controller：`ADLAPI/v3/App/Controller/MissionReportController.php` ✅
- 現有 Service：`ADLAPI/v3/App/Service/MissionReport/MissionReportService.php` ✅
- BaseController 方法：`ADLAPI/v3/App/Controller/BaseController.php`
- 舊版函數庫：
  - `include/adp_core_function.php` - string2hash(), hash2string(), id2uname()
  - `classes/TeacherClasses.php` - 教師班級相關功能

---

## 常見問題處理

**Q: 為什麼學生端已完成但教師端待處理？**
A: MissionReportController 目前只實作了學生端的 list() 和 detail()，教師端功能（篩選、班級報告）尚未開發。

**Q: 任務類型 7 和 8 為什麼沒有分數？**
A: 任務類型 7（核心素養）和 8（學科素養）是互動式學習，沒有傳統的分數概念，只顯示完成狀態。

**Q: 為什麼使用 5 個 UNION 查詢？**
A: 因為不同任務類型的測驗記錄存放在不同的資料表，需要合併查詢才能顯示完整的任務報告。

**Q: 如何處理 hash 參數？**
A: 舊版使用 string2hash() 和 hash2string() 函數，新版 API 應在 middleware 統一處理 hash 編解碼，或改用 JWT token 傳遞 user_id。

**Q: SESSION 變數如何處理？**
A: 舊版使用 $_SESSION 存放 search_exam_sn 和 search_mission_sn，新版 API 應改為在請求參數中直接傳遞，避免 stateful 設計。
