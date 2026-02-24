# 學生端 API 範圍清單 (USER_STUDENT)

**建立日期**: 2026-02-10
**最後更新**: 2026-02-11
**來源**: modules/menu/acl.php L776-862
**狀態**: Phase 5 完成

---

## 執行摘要

| 項目 | 數量 |
|------|------|
| 主選單功能 | 7 個 |
| 已完成 Controller | 19 個 |
| 已完成端點 | 約 85+ 個 |
| 待處理功能 | 0 個 |
| 不需處理 | 3 個 (排行榜、討論區、網站資源) |

---

## 1. 功能狀態總覽

### 1.1 已完成 (✅)

| 功能 | Controller | Mapping | 說明 |
|------|-----------|---------|------|
| 我的任務 | MissionController | modules_student.md | 任務 CRUD 完整 |
| 待辦事項/行事曆 | TodoController | modules_student.md | srl_calendar CRUD 完整 |
| 公告 | AnnouncementController | modules_student.md | 靜態資料 |
| 個人設定 | PersonalConfigController | modules_student.md | 設定讀寫 |
| 測驗報告 | MissionReportController | missionReport.md | 學生端完整 |
| 學習紀錄 | LearningRecordController | video_personal_file.md | 完成 |
| 知識結構 | KnowledgeController | knowledge.md | 完成 |
| 學習模組 | KsLearningController | ks-learning.md | 練習/評量/影片 |
| 素養題 | LiteracyController | literacy.md | 完成 |
| 問卷 | QuestionnaireController | questionnaire.md | 外包部分 |
| **大考專區** | EduExamController | edu-exam.md | ✅ 完整 (9端點) |
| **學習筆記** | VideoNoteController | video-note.md | ✅ 完整 (11端點) |
| **學習提問** | VideoAskController | video-ask.md | ✅ 完整 (4端點) |
| **檢核表** | ChecklistController | checklist.md | ✅ 教師端完整 (9端點) |
| **科技化評量+縣市學力檢測** | RemedyTestController | remedy-test.md | ✅ 完整 (10端點) |

### 1.2 已解決的 GAP ✅

| 功能 | 原問題 | 解決方案 |
|------|--------|----------|
| SRL 筆記 | 資料表錯誤 (srl_learning_note) | ✅ 建立 video-note.md，使用 video_note + 4 關聯表 |
| SRL 提問 | 資料表錯誤 (srl_learning_ask) | ✅ 建立 video-ask.md，使用 video_noteask + video_noteask_plus |
| SRL 檢核表 | 資料表和角色錯誤 | ✅ 建立 checklist.md + ChecklistController，使用 check_list_table |
| SRL 行事曆 | 只有讀取，缺 CRUD | ✅ TodoController 已有完整 CRUD (srl_calendar) |
| 大考專區 | 缺 submit/result | ✅ EduExamController 補完 start/submit/finish/result/history |

### 1.3 待處理 (❌)

**無** - 所有功能已完成

### 1.4 不需處理 (⏭️)

| 功能 | 原因 |
|------|------|
| 排行榜 | 用戶決定不納入 API-V3 |
| 討論區 (eZDiscus) | 外部系統，不納入 |
| 討論詳情 | 依附於外部討論區系統 |

### 1.5 外包/獨立管理

| 功能 | 說明 |
|------|------|
| 任務類型 2,7,8,12,17,18,19 | 外包廠商負責 |
| 對話機器人 (conversational_robot) | 外包 |
| 高階能力 (high_level_capability) | 外包 |
| 語音辨識 (voice_recognition) | 獨立 API |
| 課程平台 (inclass_platform) | 獨立模組 |

---

## 2. 主選單功能詳細

### 2.1 我的任務
**路由**: `modules_new.php?op=modload&name=dashboard&file=modules_student`
**狀態**: ✅ 完成

| 端點 | 方法 | 狀態 |
|------|------|------|
| POST /mission/types | types() | ✅ |
| POST /mission/list | list() | ✅ |
| POST /mission/detail | detail() | ✅ |
| POST /mission/group-report | groupReport() | ✅ |
| POST /todo/list | list() | ✅ |
| POST /todo/create | save() | ✅ |
| POST /todo/update | save() | ✅ |
| POST /todo/delete | delete() | ✅ |
| POST /announcement/list | list() | ✅ |
| POST /personal-config/get | get() | ✅ |
| PUT /personal-config/update | update() | ✅ |

### 2.2 知識結構星空圖
**路由**: `modules_new.php?op=modload&name=D3&file=chart`
**狀態**: ⚠️ 待確認

說明: D3.js 前端渲染，可能直接調用 Knowledge API 而非獨立 API

### 2.3 排行榜
**路由**: `modules_new.php?op=modload&name=rankingList&file=rankingList`
**狀態**: ⏭️ 不需處理

> 用戶決定：排行榜功能不納入 API-V3 遷移範圍

---

## 3. 報表功能群組 (s_report)

### 3.1 測驗報告
**路由**: `modules_new.php?op=modload&name=ExamResult&file=missionReport`
**狀態**: ✅ 完成

| 端點 | 方法 | 狀態 |
|------|------|------|
| POST /mission-report/list | list() | ✅ |
| POST /mission-report/detail | detail() | ✅ |
| POST /mission-report/filters | getFilters() | ✅ |
| POST /mission-report/class-report | classReport() | ✅ |

### 3.2 學習紀錄
**路由**: `modules_new.php?op=modload&name=learn_video&file=video_personal_file`
**狀態**: ✅ 完成 (98%)

| 端點 | 方法 | 狀態 |
|------|------|------|
| POST /learning-record/personal | personal() | ✅ |

---

## 4. 討論功能群組 (s_discuss)

### 4.1 學習筆記
**路由**: `modules_new.php?op=modload&name=srl&file=learning_note`
**狀態**: ✅ Mapping 完成

**Mapping 文件**: [api-mapping-video-note.md](api-mapping-video-note.md)

| 端點 | 方法 | 說明 |
|------|------|------|
| POST /video-note/list | list() | 學生/教師取筆記列表 |
| POST /video-note/detail | detail() | 筆記詳情 |
| POST /video-note/create | create() | 建立筆記 |
| POST /video-note/update | update() | 更新筆記 |
| POST /video-note/delete | delete() | 刪除筆記 |
| POST /video-note/toggle-like | toggleLike() | 按讚/取消 |
| POST /video-note/toggle-favorite | toggleFavorite() | 收藏/取消 |
| POST /video-note/toggle-display | toggleDisplay() | 公開/隱藏 |
| POST /video-note/feedback/list | feedbackList() | 回饋列表 |
| POST /video-note/feedback/create | feedbackCreate() | 寫入回饋 |
| POST /video-note/feedback/delete | feedbackDelete() | 刪除回饋 |
| POST /video-note/recommend | recommend() | 教師推薦筆記 |
| POST /video-note/give-coins | giveCoins() | 教師給代幣 |
| POST /video-note/recommended-list | recommendedList() | 取得推薦筆記 |

### 4.2 提問
**路由**: `modules_new.php?op=modload&name=srl&file=learning_ask_student`
**狀態**: ✅ Mapping 完成

**Mapping 文件**: [api-mapping-video-ask.md](api-mapping-video-ask.md)

| 端點 | 方法 | 說明 |
|------|------|------|
| POST /video-ask/list | list() | 學生/教師取提問列表 |
| POST /video-ask/detail | detail() | 提問詳情 (含回覆) |
| POST /video-ask/create | create() | 建立提問 |
| POST /video-ask/update | update() | 更新提問 |
| POST /video-ask/delete | delete() | 刪除提問 |
| POST /video-ask/reply | reply() | 回覆提問 |
| POST /video-ask/reply/update | replyUpdate() | 更新回覆 |
| POST /video-ask/reply/delete | replyDelete() | 刪除回覆 |
| POST /video-ask/toggle-solved | toggleSolved() | 標記已解決 |

### 4.3 討論區
**路由**: `eZDiscus/index.php`
**狀態**: ⏭️ 外部系統，不納入 API v3

---

## 5. 學習扶助群組 (s_remedial)

### 5.1 科技化評量 + 縣市學力檢測 (整合)
**路由**: `modules_new.php?op=modload&name=remedyTest&file=query_prioriData`
**狀態**: ✅ 完成

**Mapping 文件**: [api-mapping-remedy-test.md](api-mapping-remedy-test.md)

> 科技化評量與縣市學力檢測整合為同一 Controller，透過 exam_type 參數區分

| 端點 | 方法 | 角色 | 說明 |
|------|------|------|------|
| POST /remedy-test/filters | filters() | 學生 | 取得篩選選項 |
| POST /remedy-test/records | records() | 學生 | 取得測驗紀錄 |
| POST /remedy-test/result | result() | 學生 | 取得測驗結果 |
| POST /remedy-test/self-mission | selfMission() | 學生 | 自派診斷任務 |
| POST /remedy-test/students | students() | 教師 | 查詢補救學生 |
| POST /remedy-test/grades | grades() | 教師 | 取得年級列表 |
| POST /remedy-test/classes | classes() | 教師 | 取得班級列表 |
| POST /remedy-test/teachers | teachers() | 教師 | 取得教師列表 |
| POST /remedy-test/subjects | subjects() | 教師 | 取得科目列表 |
| POST /remedy-test/dates | dates() | 教師 | 取得測驗日期 |

**exam_type 參數**:
- `''` (空字串): 科技化評量
- `'A'`: 縣市學力檢測
- `'B'`: 學力檢測考古題

### 5.3 大考專區
**路由**: `modules_new.php?op=modload&name=remedyTest&file=query_prioriData_new`
**狀態**: ✅ 完成

**Mapping 文件**: [api-mapping-edu-exam.md](api-mapping-edu-exam.md)

| 端點 | 方法 | 狀態 |
|------|------|------|
| POST /eduexam/types | types() | ✅ |
| POST /eduexam/papers | papers() | ✅ |
| POST /eduexam/detail | detail() | ✅ |
| POST /eduexam/filters | filters() | ✅ |
| POST /eduexam/start | start() | ✅ |
| POST /eduexam/submit | submit() | ✅ |
| POST /eduexam/finish | finish() | ✅ |
| POST /eduexam/result | result() | ✅ |
| POST /eduexam/history | history() | ✅ |

---

## 6. 開發階段進度

### Phase 4: 學生端 API 完善 ✅ 已完成

| 任務 | 狀態 | 說明 |
|------|------|------|
| SRL 筆記 Mapping | ✅ | api-mapping-video-note.md |
| SRL 提問 Mapping | ✅ | api-mapping-video-ask.md |
| SRL 檢核表 Mapping + Controller | ✅ | api-mapping-checklist.md + ChecklistController |
| SRL 行事曆 CRUD | ✅ | TodoController 已涵蓋 |
| 大考專區完整流程 | ✅ | EduExamController (9 端點) |
| 排行榜 | ⏭️ | 用戶決定不納入 |

### Phase 5: 已完成 ✅

| 任務 | 狀態 | 說明 |
|------|------|------|
| 科技化評量 + 縣市學力檢測 | ✅ | RemedyTestController (10 端點) |
| VideoNoteController 實作 | ✅ | 11 個端點 |
| VideoAskController 實作 | ✅ | 4 個端點 |

**開發重點**:
- 科技化評量與縣市學力檢測整合為 RemedyTestController，透過 exam_type 參數區分
- VideoNote 與 VideoAsk 保持獨立，分別處理筆記與提問功能

---

## 7. 資料表對照

### 已修正的資料表引用 ✅

| 功能 | 舊 V3 錯誤 | 正確資料表 | Mapping |
|------|-----------|-----------|---------|
| 筆記 | srl_learning_note | video_note + 4 關聯表 | video-note.md |
| 提問 | srl_learning_ask | video_noteask, video_noteask_plus | video-ask.md |
| 檢核表 | srl_checklist | check_list_table | checklist.md |

### 正確使用的資料表

| 功能 | 資料表 | Controller |
|------|--------|------------|
| 行事曆 | srl_calendar | TodoController |
| 任務 | mission_info, mission_stud_record | MissionController |
| 知識結構 | map_node, publisher_mapping | KnowledgeController |
| 學習紀錄 | video_user, practice_record, dynex_student_exam | LearningRecordController |
| 大考專區 | eduexam_paper, eduexam_record, eduexam_record_detail | EduExamController |
| 檢核表 | check_list_table | ChecklistController |

---

## 8. 完成總結

### Phase 5 已完成 ✅
- [x] 建立 `api-mapping-remedy-test.md` (科技化評量 + 縣市學力檢測)
- [x] 建立 RemedyTestController / Service / Dao (10 端點)
- [x] 建立 VideoNoteController / Service / Dao (11 端點)
- [x] 建立 VideoAskController / Service / Dao (4 端點)

### 全部已完成 ✅
- [x] 分析 `prodb_learning_note.php`，建立 api-mapping-video-note.md
- [x] 分析 `prodb_learning_ask.php`，建立 api-mapping-video-ask.md
- [x] 分析 `prodb_checklists_new.php`，建立 api-mapping-checklist.md
- [x] 建立 ChecklistController / Service / Dao
- [x] 補完 EduExamController (start/submit/finish/result/history)
- [x] 確認 TodoController 已涵蓋行事曆 CRUD
- [x] 分析 `prodb_get_priori_data.php`，建立 RemedyTestController

---

## 參考文件

### 主要 Mapping 文件
- [api-mapping-modules_student.md](api-mapping-modules_student.md) - 任務/待辦/公告/設定
- [api-mapping-missionReport.md](api-mapping-missionReport.md) - 測驗報告
- [api-mapping-video_personal_file.md](api-mapping-video_personal_file.md) - 學習紀錄
- [api-mapping-edu-exam.md](api-mapping-edu-exam.md) - 大考專區 ✅ 完整
- [api-mapping-video-note.md](api-mapping-video-note.md) - 學習筆記 ✅ 完整
- [api-mapping-video-ask.md](api-mapping-video-ask.md) - 學習提問 ✅ 完整
- [api-mapping-checklist.md](api-mapping-checklist.md) - 檢核表 ✅ 完整
- [api-mapping-srl-calendar.md](api-mapping-srl-calendar.md) - 行事曆 ✅ 完整
- [api-mapping-remedy-test.md](api-mapping-remedy-test.md) - 科技化評量/縣市學力檢測 ✅ 完整

### 原始碼參考
- [acl.php](../../modules/menu/acl.php) - 學生選單定義 (L776-862)
