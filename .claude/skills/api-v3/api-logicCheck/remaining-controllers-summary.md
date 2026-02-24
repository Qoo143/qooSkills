# 剩餘 Controllers 快速檢查總結

本文檔總結剩餘 Controllers 的檢查狀態（基於代碼結構快速掃描，未深度比對 SQL）

---

## 已完成深度檢查的 Controllers (7個，35端點)

1. **AnnouncementController** (1端點) ✅
2. **ChecklistController** (9端點) ✅
3. **EduExamController** (9端點) ⚠️ papers 端點有安全漏洞
4. **KnowledgeController** (4端點) ⭐ 實作最完整
5. **KsLearningController** (8端點) 🔴 代幣未發放、節點狀態未更新
6. **LearningRecordController** (1端點) 💯 Wrapper模式
7. **LiteracyController** (3端點) ✅ 符合外包系統需求

---

## 快速掃描的 Controllers (11個，約50端點)

### 8. MissionController (4端點)
- types
- list
- detail
- groupReport

**狀態**: 已實作，無 API mapping 文檔

---

### 9. MissionReportController (4端點)
- list (學生任務報告清單)
- filters (教師端篩選選項)
- assigners (取得指派老師清單)
- classReport (教師端班級報告)

**對應文檔**: api-mapping-missionReport.md  
**狀態**: ✅ 根據文檔，教師端和學生端功能已完整實作

---

### 10. QuestionnaireController (4端點)
- types (問卷類型列表)
- detail (問卷詳情)
- status (檢查填寫狀態)
- history (填寫歷史)

**對應文檔**: api-mapping-questionnaire.md  
**狀態**: 已實作，處理 6 種問卷類型（5c/CT109/network109/ntcu/improvement_st/improvement_th）

---

### 11. RemedyTestController (10端點)
- filters (學生測驗篩選選項)
- records (學生測驗紀錄)
- result (測驗結果)
- selfMission (學生自派診斷任務)
- students (補救學生列表 - 教師端)
- grades (年級列表 - 教師端)
- classes (班級列表 - 教師端)
- teachers (教師列表)
- subjects (科目列表)
- dates (測驗日期列表)

**對應文檔**: api-mapping-remedy-test.md  
**狀態**: 已實作，整合科技化評量與縣市學力檢測

---

### 12. SrlController (5端點)
- calendar (行事曆事件)
- notes (學習筆記列表)
- noteDetail (學習筆記詳情)
- checklist (檢核表)
- questions (學生提問)

**對應文檔**: api-mapping-srl.md  
**狀態**: 已實作，自律學習表單相關功能

---

### 13. VideoAskController (4端點)
- list (提問列表)
- detail (提問詳情含回覆)
- reply (回覆提問)
- delete (刪除提問)

**對應文檔**: api-mapping-video-ask.md  
**狀態**: 已實作

---

### 14. VideoNoteController (11端點)
- list (筆記列表)
- like (按讚/取消按讚)
- favorite (收藏/取消收藏)
- toggleVisibility (公開/隱藏筆記)
- delete (刪除筆記)
- feedbackList (回饋列表)
- createFeedback (新增回饋)
- deleteFeedback (刪除回饋)
- giveCoins (教師給代幣)
- recommend (推薦筆記)
- recommendedCount (被推薦未公開數)

**對應文檔**: api-mapping-video-note.md  
**狀態**: 已實作，功能完整

---

### 15. TodoController (5端點)
- list (待辦清單)
- detail (單筆待辦)
- create (新增待辦)
- update (更新待辦)
- delete (刪除待辦)

**狀態**: 已實作，無 API mapping 文檔

---

### 16. PersonalConfigController (2端點)
- get (取得個人設定)
- update (更新個人設定)

**狀態**: 已實作，無 API mapping 文檔，處理兩類設定（item_card_or_list, calendar）

---

### 17. UserController (?端點)
**狀態**: 存在但未查看端點數

---

## 總結

### 端點統計
- **深度檢查**: 7 Controllers, 35 端點
- **快速掃描**: 10 Controllers, ~50 端點
- **未檢查**: 1 Controller (UserController)
- **總計已知**: 18 Controllers, 85+ 端點

### 有 API Mapping 文檔的 Controllers (13個)
1. api-mapping-edu-exam.md → EduExamController ✅
2. api-mapping-knowledge.md → KnowledgeController ✅
3. api-mapping-ks-learning.md → KsLearningController ✅
4. api-mapping-ks_viewskill_new.md → (相關)
5. api-mapping-literacy.md → LiteracyController ✅
6. api-mapping-missionReport.md → MissionReportController ✅
7. api-mapping-modules_student.md → (相關)
8. api-mapping-questionnaire.md → QuestionnaireController ✅
9. api-mapping-remedy-test.md → RemedyTestController ✅
10. api-mapping-srl.md → SrlController ✅
11. api-mapping-video-ask.md → VideoAskController ✅
12. api-mapping-video-note.md → VideoNoteController ✅
13. api-mapping-video_personal_file.md → (相關)

### 檢查建議
對於快速掃描的 Controllers，建議：
1. **高優先級**（核心功能）: MissionController, MissionReportController
2. **中優先級**（常用功能）: QuestionnaireController, RemedyTestController
3. **低優先級**（輔助功能）: TodoController, PersonalConfigController

若需深度檢查，應：
1. 查看對應的 API mapping 文檔（如有）
2. 比對 Service/DAO 的 SQL 查詢
3. 確認與舊程式的邏輯一致性
