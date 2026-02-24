# API-V3 GAP 分析報告

**建立日期**: 2026-02-10
**分析範圍**: 學生端 API (基於 api-scope-student.md)

---

## 執行摘要

| 項目 | 數量 |
|------|------|
| 現有 Controller | 13 個 (不含 BaseController) |
| Mapping 檔案 | 10 個 |
| 嚴重 GAP (資料表錯誤) | 3 個功能 |
| 功能缺失 (僅讀取無寫入) | 2 個 Controller |
| 端點缺失 | 約 30+ 個 |

---

## Controller 現況總覽

| Controller | 方法數 | Mapping | 狀態 |
|------------|--------|---------|------|
| AnnouncementController | 1 | modules_student.md | ✅ 完成 |
| EduExamController | 4 | edu-exam.md | ⚠️ 缺 submit/result |
| KnowledgeController | 4 | knowledge.md | ✅ 完成 |
| KsLearningController | 8 | ks-learning.md | ✅ 完成 |
| LearningRecordController | 2 | video_personal_file.md | ✅ 完成 |
| LiteracyController | 3 | literacy.md | ✅ 完成 |
| MissionController | 4 | modules_student.md | ✅ 完成 |
| MissionReportController | 6 | missionReport.md | ✅ 完成 |
| PersonalConfigController | 2 | modules_student.md | ✅ 完成 |
| QuestionnaireController | 4 | questionnaire.md | ✅ 完成 |
| SrlController | 5 | srl.md | 🔴 嚴重問題 |
| TodoController | 5 | modules_student.md | ✅ 完成 |
| UserController | 2 | (無專用) | ✅ 基礎功能 |

---

## 🔴 嚴重 GAP：SrlController

### 問題 1：學習筆記 - 資料表完全錯誤

**現況**: 查詢 `srl_learning_note`
**正確**: 應查詢 `video_note` + 4 關聯表

| 缺失功能 | 舊版 action | 優先級 |
|----------|------------|--------|
| 按讚/取消 | updateLike | P1 |
| 收藏/取消 | updateFavorite | P1 |
| 查看回饋 | searchFeedback | P1 |
| 寫入回饋 | sentFeedback | P1 |
| 刪除回饋 | deleteFeedback | P2 |
| 公開/隱藏 | displayNote | P2 |
| 刪除筆記 | deleteNotes | P2 |
| 教師給代幣 | giveCoins | P2 |
| 推薦筆記 | recommendNotes | P2 |
| 取推薦數量 | getTeacherRecommend | P3 |
| 影片推薦筆記 | getRecommendNotes | P3 |

**所需資料表**:
- video_note
- video_note_favorite
- video_note_feedback
- video_note_recommend
- role_coin_total
- role_coin_history

---

### 問題 2：學生提問 - 資料表完全錯誤

**現況**: 查詢 `srl_learning_ask`
**正確**: 應查詢 `video_noteask` + `video_noteask_plus`

| 缺失功能 | 說明 | 優先級 |
|----------|------|--------|
| 教師取提問 | 4 個 UNION 查詢 | P1 |
| 學生取提問 | 4 個 UNION 查詢 | P1 |
| 回覆功能 | video_noteask_plus | P1 |
| 小組提問 | user_group + seme_group | P2 |

---

### 問題 3：檢核表 - 資料表 + 角色錯誤

**現況**: 查詢 `srl_checklist` (user_id)
**正確**: 應查詢 `check_list_table` (teacher_id)

| 缺失功能 | 舊版 action | 優先級 |
|----------|------------|--------|
| 新增檢核表 | inputdata | P2 |
| 編輯檢核表 | updateeditdata | P2 |
| 刪除檢核表 | deleteData | P2 |
| 鎖定/解鎖 | updatelockdata | P2 |
| 複製檢核表 | copydata | P3 |
| 預設模板 | loadTableContent | P3 |
| 學期篩選 | getSheetSemes | P3 |
| 分頁功能 | getPages | P3 |

---

### 問題 4：行事曆 - 僅讀取無寫入

**現況**: 只有 `calendar()` 讀取方法
**缺失**: 全部 CRUD 操作

| 缺失功能 | 舊版 action | 優先級 |
|----------|------------|--------|
| 新增事件 | add_event | P1 |
| 編輯事件 | edit_event | P1 |
| 刪除事件 | delete_event | P1 |
| 標記完成 | edit_event_isDone | P2 |
| 到期提醒 | reminder_events | P2 |
| 特定日期查詢 | now_allevent | P3 |

---

## ⚠️ 中度 GAP

### EduExamController - 缺少提交/結果

| 缺失功能 | 說明 | 優先級 |
|----------|------|--------|
| 提交答案 | submit | P1 |
| 查看結果 | result | P1 |

---

## ✅ 已完成 Controller

以下 Controller 與 Mapping 對照完整，無明顯 GAP：

1. **AnnouncementController** - 公告列表
2. **KnowledgeController** - 知識結構 (node, unit, list, relatedNodes)
3. **KsLearningController** - 學習模組 (練習、評量、影片、檢查點)
4. **LearningRecordController** - 學習紀錄
5. **LiteracyController** - 素養題
6. **MissionController** - 任務管理
7. **MissionReportController** - 測驗報告
8. **PersonalConfigController** - 個人設定
9. **QuestionnaireController** - 問卷
10. **TodoController** - 待辦事項

---

## 開發優先順序建議

### P0 - 立即修復 (資料表錯誤)

1. **VideoNoteController** (新建，取代 SrlController 筆記功能)
   - 修正資料表為 video_note 系統
   - 實作 13 個 action

2. **VideoAskController** (新建，取代 SrlController 提問功能)
   - 修正資料表為 video_noteask 系統
   - 實作教師/學生兩套查詢

### P1 - 高優先級

3. **SrlController** 行事曆 CRUD
   - add_event, edit_event, delete_event
   - 保持使用 srl_calendar (正確)

4. **EduExamController** 補完
   - submit(), result()

### P2 - 中優先級

5. **ChecklistController** (新建，教師功能)
   - 修正為 check_list_table
   - 實作 8 個 action

### P3 - 低優先級

6. 行事曆進階功能 (到期提醒等)
7. 檢核表預設模板

---

## 程式碼結構建議

### 建議拆分 SrlController

```
原 SrlController (5 方法)
    ├── calendar()      → 保留，補 CRUD
    ├── notes()         → 移至 VideoNoteController
    ├── noteDetail()    → 移至 VideoNoteController
    ├── checklist()     → 移至 ChecklistController (教師)
    └── questions()     → 移至 VideoAskController

新結構:
├── SrlCalendarController (或保留在 SrlController)
│   ├── list()
│   ├── create()
│   ├── update()
│   ├── delete()
│   ├── toggleDone()
│   └── reminder()
│
├── VideoNoteController (新建)
│   ├── list()
│   ├── like()
│   ├── favorite()
│   ├── feedbackList()
│   ├── createFeedback()
│   ├── deleteFeedback()
│   ├── toggleVisibility()
│   ├── delete()
│   ├── giveCoins()
│   ├── recommend()
│   ├── recommendedPrivateCount()
│   └── recommendedByVideo()
│
├── VideoAskController (新建)
│   ├── listForStudent()
│   ├── listForTeacher()
│   └── (回覆功能待定)
│
└── ChecklistController (新建，教師功能)
    ├── list()
    ├── create()
    ├── update()
    ├── delete()
    ├── toggleLock()
    ├── copy()
    └── templates()
```

---

## 下一步行動

### Phase 4 執行順序

1. [ ] 建立 VideoNoteController + VideoNoteService + VideoNoteDao
2. [ ] 建立 VideoAskController + VideoAskService + VideoAskDao
3. [ ] 補完 SrlController 行事曆 CRUD
4. [ ] 補完 EduExamController submit/result
5. [ ] 建立 ChecklistController (教師功能)

---

## 新檔案設計規劃

### 1. VideoNote 模組 (影片筆記)

**檔案路徑**:
```
ADLAPI/v3/App/
├── Controller/VideoNoteController.php
├── Service/VideoNoteService.php
└── Dao/VideoNoteDao.php
```

**資料表**:
- video_note (主表)
- video_note_favorite (按讚/收藏)
- video_note_feedback (回饋)
- video_note_recommend (推薦)
- role_coin_total (代幣累計)
- role_coin_history (代幣紀錄)
- video_concept_item (影片關聯)
- map_node, map_info (知識點)

**端點清單**:
| 端點 | 方法 | 角色 | 說明 |
|------|------|------|------|
| POST /video-note/list | list() | 全部 | 取得筆記列表 |
| POST /video-note/like | like() | 全部 | 按讚/取消 |
| POST /video-note/favorite | favorite() | 全部 | 收藏/取消 |
| POST /video-note/toggle-visibility | toggleVisibility() | 作者 | 公開/隱藏 |
| POST /video-note/delete | delete() | 作者/教師 | 刪除筆記 |
| POST /video-note/feedback/list | feedbackList() | 全部 | 取得回饋 |
| POST /video-note/feedback/create | createFeedback() | 全部 | 新增回饋 |
| POST /video-note/feedback/delete | deleteFeedback() | 作者 | 刪除回饋 |
| POST /video-note/give-coins | giveCoins() | 教師 | 給代幣 |
| POST /video-note/recommend | recommend() | 教師 | 推薦筆記 |
| POST /video-note/recommended-count | recommendedCount() | 學生 | 被推薦未公開數 |
| POST /video-note/recommended-by-video | recommendedByVideo() | 學生 | 影片推薦筆記 |

---

### 2. VideoAsk 模組 (影片提問)

**檔案路徑**:
```
ADLAPI/v3/App/
├── Controller/VideoAskController.php
├── Service/VideoAskService.php
└── Dao/VideoAskDao.php
```

**資料表**:
- video_noteask (主表)
- video_noteask_plus (回覆)
- seme_student (學期學生)
- seme_teacher_subject (科任班)
- user_group, seme_group (小組)
- video_concept_item (影片關聯)

**端點清單**:
| 端點 | 方法 | 角色 | 說明 |
|------|------|------|------|
| POST /video-ask/list | list() | 全部 | 取得提問列表 |
| POST /video-ask/detail | detail() | 全部 | 提問詳情含回覆 |
| POST /video-ask/reply | reply() | 全部 | 回覆提問 |
| POST /video-ask/delete | delete() | 作者/教師 | 刪除提問 |

---

### 3. SRL 行事曆 CRUD 補完

**現有檔案** (補完):
```
ADLAPI/v3/App/
├── Controller/SrlController.php  (補 5 方法)
├── Service/SrlService.php        (補對應方法)
└── Dao/SrlDao.php               (補對應方法)
```

**補充端點**:
| 端點 | 方法 | 說明 |
|------|------|------|
| POST /srl/calendar/create | createEvent() | 新增事件 |
| POST /srl/calendar/update | updateEvent() | 更新事件 |
| POST /srl/calendar/delete | deleteEvent() | 刪除事件 |
| POST /srl/calendar/toggle-done | toggleDone() | 標記完成 |
| POST /srl/calendar/reminder | reminder() | 到期提醒 |

---

### 預估工作量

| 任務 | 複雜度 | 說明 |
|------|--------|------|
| VideoNoteController | 高 | 13 方法，複雜 SQL |
| VideoAskController | 高 | 多 UNION 查詢 |
| 行事曆 CRUD | 低 | 標準 CRUD |
| EduExam 補完 | 中 | 需分析舊版提交邏輯 |
| ChecklistController | 中 | 8 方法，教師功能 |

---

## 參考文件

- [api-mapping-srl.md](api-mapping-srl.md) - SRL 模組詳細分析
- [api-scope-student.md](api-scope-student.md) - 學生端完整範圍
- [api-controller-status.md](api-controller-status.md) - Controller 狀態