# API-V3 對照表：EduExam (學力測驗/大考專區)

## 舊檔案

| 檔案 | 用途 |
|------|------|
| `modules/exam/useditem_prodb.php` | 前台考古題 API (getEduexamType, getFinishRecord) |
| `modules/eduexam/useditem.php` | 前台考古題選擇介面 |
| `modules/eduexam/answer.php` | 前台作答介面 (answer_prodb 透過 JS 呼叫) |
| `modules/eduexam/classes/EduExam.php` | 考試類型定義 (extends AbstractExam) |
| `modules/eduexam/classes/Paper.php` | 試卷管理 (extends AbstractPaper) |
| `modules/eduexam/classes/Item.php` | 試題管理 (extends AbstractItem) |
| `classes/Exam/AbstractExam.php` | 抽象基底：表名常數、exam_type 常數 |
| `classes/Exam/AbstractPaper.php` | 抽象基底：試卷 CRUD、狀態權限矩陣 |
| `modules/eduexam/backstage/` | 後台管理 (paper/item/itemgroup CRUD) |

## 現有 V3 檔案

| 層級 | 檔案 |
|------|------|
| Controller | `ADLAPI/v3/App/Controller/EduExamController.php` |
| Service | `ADLAPI/v3/App/Service/EduExamService.php` |
| DAO | `ADLAPI/v3/App/Dao/EduExamDao.php` |

## 考試類型與 Mission Type 對應

| 縮寫 | exam_type | mission_type | 名稱 |
|------|-----------|-------------|------|
| BAT | 1 | 6 | 學力測驗 |
| CAP | 2 | 9 | 國中教育會考 |
| GSAT | 3 | 13 | 學測 |
| TVE | 4 | 14 | 統測 |
| AST | 5 | 15 | 指考 |

## 資料表名稱對應

舊系統使用 `getTableName()` 動態組合表名，模式為 `eduexam_` + base name：

| 基底名 | 實際表名 | 用途 |
|--------|---------|------|
| exam_type | exam_type | 考試類型 (共用) |
| paper | eduexam_paper | 試卷 |
| paper_detail | eduexam_paper_detail | 試卷內容 |
| item | eduexam_item | 試題 |
| item_indicator_mapping | eduexam_item_indicator_mapping | 題目與能力指標 |
| item_answer_option | eduexam_item_answer_option | 答案選項 |
| itemgroup | eduexam_itemgroup | 題組 |
| itemgroup_detail | eduexam_itemgroup_detail | 題組內容 |
| record | eduexam_record | 測驗紀錄 |
| record_detail | eduexam_record_detail | 測驗紀錄明細 |

---

## Controller 方法對照

### 1. types() ✅ 已實作

- **路由**: POST /edu-exam/types
- **舊檔案對應**: `useditem_prodb.php` → `getEduexamType()` (部分)
- **參數**: 無
- **業務邏輯**: 查 exam_type 表取得所有考試類型
- **SQL (舊)**:
  ```sql
  SELECT exam_type_code, exam_type_name
  FROM exam_type
  WHERE series = 'eduexam'
  ```
- **SQL (新)**:
  ```sql
  SELECT exam_type_sn, exam_type_name, exam_type_abbr
  FROM exam_type
  WHERE exam_type_sn IN (1,2,3,4,5)
  ```
- **驗證狀態**: ⚠️ 有差異
- **差異說明**:
  1. 舊系統用 `series = 'eduexam'` 篩選，新系統硬編碼 SN 1-5
  2. 舊系統回傳 `exam_type_code`，新系統回傳 `exam_type_abbr` (欄位名可能不同)
  3. 新系統有 fallback 機制使用常數定義，舊系統無

### 2. papers() ⚠️ 不完整

- **路由**: POST /edu-exam/papers
- **舊檔案對應**: `useditem_prodb.php` → `getSubjectType()`
- **參數 (新)**: exam_type, subject_id, year, status, page, per_page
- **參數 (舊)**: chooseEduexamType (考試類型代碼)

- **業務邏輯差異**:

| 項目 | 舊系統 | 新系統 | 狀態 |
|------|--------|--------|------|
| 狀態篩選 | `status = 1` (STATUS_CODE_LOCKED) 只顯示已鎖定 | `status != 4` (排除已刪除) | ❌ 權限漏洞 |
| 未完成次數 | 子查詢計算 unfinishedtimes | 無 | ❌ 缺失 |
| 已完成次數 | 子查詢計算 finishedtimes | 無 | ❌ 缺失 |
| 科目排序 | 自訂中文順序 (國語文→數學→英語...) | ORDER BY subject_id | ⚠️ 順序不同 |
| 科目來源 | subject_groupings (科目群組) | subject 表 | ⚠️ JOIN 不同 |
| map_info display 篩選 | `mi.display = 1` AND `sub.display = 1` | 無 | ❌ 缺失 |
| paper_sn hash | `string2hash()` 加密回傳 | 直接回傳明碼 | ⚠️ 安全性差異 |
| 按科目分組 | 前端按科目分組顯示 | 扁平列表 | ⚠️ 結構不同 |

- **SQL (舊) 關鍵查詢**:
  ```sql
  SELECT ep.paper_sn, ep.paper_name, sg.subject_groupings_name as map_name,
         ep.year, sg.subject_groupings_id as group_id,
         (子查詢 unfinishedtimes),
         (子查詢 finishedtimes)
  FROM eduexam_paper ep
  JOIN map_info mi ON ep.map_sn = mi.map_sn AND mi.display = 1
  JOIN subject sub ON mi.subject_id = sub.subject_id AND sub.display = 1
  JOIN subject_groupings sg ON sub.subject_groupings_id = sg.subject_groupings_id
  JOIN exam_type et ON et.exam_type = ep.exam_type AND et.exam_type_code = :chooseEduexamType
  WHERE ep.status = 1
  ORDER BY map_name ASC, ep.year DESC
  ```

- **特殊邏輯 - 科目自訂排序**:
  ```
  國語文 → 數學 → 英語 → 自然 → 歷史 → 地理 → 公民與社會 → 物理 → 化學 →
  生物 → 地球科學 → 音樂 → 美術 → 藝術生活 → 生命教育 → 生涯規劃 → 家政 →
  生活科技 → 資訊科技 → 健康與護理 → 體育 → 全民國防教育
  ```

### 3. detail() ⚠️ 可改善

- **路由**: POST /edu-exam/detail
- **舊檔案對應**: `AbstractPaper::getPaperInfo()` (後台用)
- **參數**: paper_sn (必填)
- **驗證狀態**: ⚠️ 簡化版
- **差異說明**:
  1. 舊系統 `getPaperInfo()` 有 `map_sn IN (permissionMapSnList)` 權限檢查，新系統無
  2. 舊系統回傳完整的 itembank_list (含題組結構)，新系統只有基本 item 列表
  3. 舊系統使用 JSON_ARRAYAGG 取得嵌套結構，新系統是獨立兩次查詢

### 4. filters() ✅ 基本完整

- **路由**: POST /edu-exam/filters
- **舊檔案對應**: 無直接對應 (新增功能)
- **參數**: exam_type (選填)
- **驗證狀態**: ✅ 完整 (此為新增 API，無舊邏輯需比對)

---

## 尚未遷移的功能

### 高優先：前台學生功能

#### A. getUnfinished() ❌ 未實作

- **舊檔案**: `useditem_prodb.php` → `getIsUnfinished()`
- **建議端點**: `POST /edu-exam/unfinished`
- **功能**: 取得學生未完成的測驗紀錄 (含任務/自主練習)
- **SQL**:
  ```sql
  SELECT er.record_sn, er.paper_sn, er.mission_sn, mi.mission_nm, er.creation_time
  FROM eduexam_record er
  LEFT JOIN mission_info mi ON mi.mission_sn = er.mission_sn
  WHERE er.is_finished = 0 AND er.user_id = :user_id
  ```
- **特殊邏輯**: paper_sn 和 mission_sn 需要 hash 加密回傳

#### B. getFinishRecord() ❌ 未實作

- **舊檔案**: `useditem_prodb.php` → `fnGetFinishRecord()`
- **建議端點**: `POST /edu-exam/records`
- **功能**: 取得試卷的歷史測驗紀錄
- **參數**: paper_sn (必填), is_more (選填, 預設 false 只取 3 筆)
- **SQL**: UNION 查詢，含已刪除試卷的歷史紀錄 (透過 year + map_sn 關聯)
- **特殊邏輯**:
  1. 預設 LIMIT 3，is_more=true 時 LIMIT 12
  2. UNION 第二段：查詢已刪除試卷對應的紀錄 (同年度同科目)
  3. 回傳區分任務紀錄和自主練習紀錄
  4. record_sn、paper_sn、mission_sn 都需要 hash

#### C. 作答流程 ❌ 未實作

- **舊檔案**: `answer_prodb.php` (透過 JS answer.js 呼叫)
- **建議端點**:
  - `POST /edu-exam/start` - 開始/繼續作答 (建立 record)
  - `POST /edu-exam/save-answer` - 儲存單題答案
  - `POST /edu-exam/submit` - 交卷
  - `POST /edu-exam/history` - 查看歷史作答詳情
- **功能**: 完整作答生命週期
- **模式**: SRL (自主練習) / MISSION (任務) / HISTORY (查看歷史)

### 低優先：後台管理功能

#### D. 試卷管理 ❌ 未實作

- **舊檔案**: `AbstractPaper` + `backstage/`
- **功能**: createPaper, updatePaper, lockPaper, deletePaper
- **權限**: 需要 `getManagingMapSN()` 權限檢查
- **狀態矩陣**: free → can_edit/can_delete/can_lock, locked → 全部不可

#### E. 試題管理 ❌ 未實作

- **舊檔案**: `AbstractItem` + `backstage/item.php`
- **功能**: CRUD、狀態檢查、題組關聯

#### F. 題組管理 ❌ 未實作

- **舊檔案**: `AbstractItemGroup` + `backstage/itemgroup.php`

#### G. 任務試卷列表 ❌ 未實作

- **舊檔案**: `AbstractPaper::getMissionPaperList()`
- **功能**: 教師指派任務時取得可用試卷列表 (status=1)

---

## 安全性檢查

| 項目 | 舊系統 | 新系統 | 風險 |
|------|--------|--------|------|
| CSRF 驗證 | `csrf_token` 比對 | Token-based auth | ✅ OK |
| XSS 防護 | `preventXss()` | SanitizeMiddleware | ✅ OK |
| 登入檢查 | `$_SESSION['user_id']` | accesstoken 驗證 | ✅ OK |
| papers status 篩選 | `status = 1` (已鎖定) | `status != 4` | ❌ 學生可能看到未發布試卷 |
| SN 加密 | `string2hash()` | 明碼 | ⚠️ 資安考量 |
| map_info display | 檢查 display=1 | 未檢查 | ❌ 可能顯示停用科目 |

---

## 特殊邏輯清單

1. **ExamSeries 架構**: 舊系統使用 ExamSeries 工廠模式，`eduexam` 和 `contestexam` 共用抽象基底，表名透過 `getModuleName()` 動態組合
2. **科目排序**: 中文自訂順序排列 (非 subject_id)，使用 `mb_str_split` 取前兩字元比對
3. **已刪除試卷紀錄**: UNION 查詢透過 year + map_sn 找到已刪除版本的作答紀錄
4. **paper_sn hash**: 分為短 hash (`string2hash($sn, false)`) 和長 hash (`string2hash($sn)`)，前端使用不同場景
5. **STATUS_CODE 定義不同**: 舊系統 Paper status: 0=未上鎖, 1=已上鎖, 2=已刪除, 3=已上鎖(未公開)；新系統 DAO 用: 0=未發布, 1=待組卷, 2=可使用, 3=已鎖定, 4=已刪除 → 定義完全不同，需確認實際 DB 欄位值

---

## 遷移建議

### papers() 修正重點
1. 加入 `status = 1` 篩選 (只顯示已鎖定/可用試卷給學生)
2. JOIN `map_info` 加上 `display = 1` 篩選
3. JOIN `subject_groupings` 取得科目群組名
4. 加入 unfinishedtimes / finishedtimes 子查詢
5. 實作科目自訂中文排序邏輯

### 新增端點優先序
1. `POST /edu-exam/unfinished` - 未完成測驗
2. `POST /edu-exam/records` - 歷史紀錄
3. `POST /edu-exam/start` - 開始作答
4. `POST /edu-exam/save-answer` - 儲存答案
5. `POST /edu-exam/submit` - 交卷
