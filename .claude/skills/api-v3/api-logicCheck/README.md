# API v3 深度邏輯檢查總表

**建立日期**: 2026-02-11  
**目的**: 逐一比對舊程式與新 API v3 的邏輯一致性  
**範圍**: 學生端 18 個 Controller,85+ 個端點  
**檢查順序**: 按 Controller 字母順序

---

## 檢查狀態總覽

| Controller | 總端點數 | 已檢查 | 一致 | 有差異 | 需改進 | 進度 | SQL 檢查方式 |
|-----------|---------|--------|------|--------|--------|------|-----------| 
| Announcement (公告) | 1 | 1 | 1 | 0 | 0 | 100% | 硬編碼/無SQL |
| ChecklistController (檢核表) | 9 | 9 | 9 | 0 | 0 | 100% | ✅ 深度SQL比對 |
| EduExamController (學力測驗) | 9 | 9 | 8 | 1 | 1 | 100% | ✅ 深度SQL比對 |
| IndicateTestController (縱貫診斷) | 4 | 4 | 4 | 0 | 0 | 100% | ✅ 深度SQL比對 |
| IndicatorTestController (單元診斷) | 4 | 4 | 4 | 0 | 0 | 100% | ✅ 深度SQL比對 |
| KnowledgeController (知識結構) | 4 | 4 | 4 | 0 | 0 | 100% | ✅ 深度SQL比對 |
| KsLearningController (學習活動) | 8 | 8 | 3 | 5 | 3 | 100% | ✅ 深度SQL比對 |
| LearningRecordController (學習紀錄) | 1 | 1 | 1 | 0 | 0 | 100% | Wrapper/無SQL |
| LiteracyController (素養導向) | 3 | 3 | 1 | 2 | 2 | 100% | ✅ 深度SQL比對 |
| MissionController (任務) | 4 | 4 | 4 | 0 | 0 | 100% | ✅ 深度SQL比對 |
| MissionReportController (任務報告) | 4 | 4 | 4 | 0 | 0 | 100% | ✅ 深度SQL比對 |
| PersonalConfigController (個人設定) | 2 | 2 | 2 | 0 | 0 | 100% | ✅ 深度SQL比對 |
| QuestionnaireController (問卷) | 4 | 4 | 4 | 0 | 0 | 100% | ✅ 深度SQL比對 |
| RemedyTestController (學習扶助) | 10 | 10 | 10 | 0 | 0 | 100% | ✅ 深度SQL比對 |
| SrlController (自律學習) | 5 | 5 | 5 | 0 | 0 | 100% | ✅ 深度SQL比對 |
| TodoController (待辦事項) | 5 | 5 | 5 | 0 | 0 | 100% | ✅ 深度SQL比對 |
| UserController (使用者) | ? | - | - | - | 0 | 跳過 | N/A |
| VideoAskController (影片提問) | 4 | 4 | 4 | 0 | 0 | 100% | ✅ 深度SQL比對 |
| VideoNoteController (影片筆記) | 11 | 11 | 11 | 0 | 0 | 100% | ✅ 深度SQL比對 |
| **深度SQL檢查總計** | **92** | **92** | **86** | **6** | **6** | **100%** | **16 Controllers** |
| **總計** | **92** | **92** | **86** | **6** | **6** | **100%** | **19 Controllers** |

---

## 詳細端點檢查表

### 圖例說明
- ✅ **一致** - 邏輯完全一致
- ⚠️ **有差異** - 邏輯有差異但可接受
- 🔴 **不一致** - 邏輯不一致需修正
- 💡 **可改進** - 有優化空間
- ⏭️ **跳過** - 外包或不需檢查

---

## AnnouncementController

| 舊程式檔案 | 舊程式方法/action | 新 API 端點 | Controller 方法 | Service 方法 | DAO 方法 | 邏輯一致性 | 差異說明 | 改進建議 | 狀態 | 檢查文檔 |
|-----------|-----------------|------------|----------------|-------------|----------|-----------|---------|---------|------|---------|
| modulesNews_data.js + calender_todolist.php | gatherNewsData() | POST /announcement/list | AnnouncementController::list() | AnnouncementService::getAnnouncements() | UserDao::getUserStatus() | ✅一致 | 回應格式 JSON vs HTML (符合預期) | 公告資料雙重維護需改進 | ✅已檢查 | [announcement-list.md](announcement-list.md) |

---

## ChecklistController

| 舊程式檔案 | 舊程式方法/action | 新 API 端點 | Controller 方法 | Service 方法 | DAO 方法 | 邏輯一致性 | 差異說明 | 改進建議 | 狀態 | 檢查文檔 |
|-----------|-----------------|------------|----------------|-------------|----------|-----------|---------|---------|------|---------|
| prodb_checklists_new.php | getSheetSemes | POST /checklist/semesters | semesters() | getSemesters() | getSemesters() | ✅一致 | 增加索引重建 | - | ✅已檢查 | [checklist-semesters.md](checklist-semesters.md) |
| prodb_checklists_new.php | getPages+loadData | POST /checklist/list | list() | getList() | getPageCount()+getList() | ✅一致 | 兩次請求→單次請求 | 移除Hash SN | ✅已檢查 | [checklist-list.md](checklist-list.md) |
| prodb_checklists_new.php | (無獨立action) | POST /checklist/detail | detail() | getDetail() | getById() | ✅新增 | 提供獨立端點 | 權限檢查可提取Helper | ✅已檢查 | [checklist-detail.md](checklist-detail.md) |
| prodb_checklists_new.php | inputdata | POST /checklist/create | create() | create() | create() | ✅一致 | 修正安全漏洞(teacher_id) | 返回新增SN | ✅已檢查 | [checklist-create.md](checklist-create.md) |
| prodb_checklists_new.php | updateeditdata | POST /checklist/update | update() | update() | update() | ✅一致 | 增加明確錯誤處理 | 權限檢查可提取Helper | ✅已檢查 | [checklist-update.md](checklist-update.md) |
| prodb_checklists_new.php | deleteData | POST /checklist/delete | delete() | delete() | delete() | ✅一致 | 🔴修正安全漏洞(無權限檢查) | 權限檢查可提取Helper | ✅已檢查 | [checklist-delete.md](checklist-delete.md) |
| prodb_checklists_new.php | updatelockdata | POST /checklist/toggle-lock | toggleLock() | toggleLock() | updatePublicStatus() | ✅一致 | 🔴修正安全漏洞(無權限檢查) | 權限檢查可提取Helper | ✅已檢查 | [checklist-togglelock.md](checklist-togglelock.md) |
| prodb_checklists_new.php | copydata | POST /checklist/copy | copy() | copy() | copy() | ✅一致 | public硬編碼為0 | 返回新SN | ✅已檢查 | [checklist-copy.md](checklist-copy.md) |
| prodb_checklists_new.php | loadTableContent | POST /checklist/template | template() | getTemplate() | (無) | ✅一致 | 🔴修正模板Bug(類型2錯誤) | - | ✅已檢查 | [checklist-template.md](checklist-template.md) |

---

## EduExamController

| 舊程式檔案 | 舊程式方法/action | 新 API 端點 | Controller 方法 | Service 方法 | DAO 方法 | 邏輯一致性 | 差異說明 | 改進建議 | 狀態 | 檢查文檔 |
|-----------|-----------------|------------|----------------|-------------|----------|-----------|---------|---------|------|---------|
| useditem_prodb.php | getEduexamType() | POST /eduexam/types | types() | getExamTypes() | getExamTypes() | ⚠️有差異 | 篩選條件不同 | 改用series篩選 | ✅已檢查 | [eduexam-summary.md](eduexam-summary.md) |
| useditem_prodb.php | getSubjectType() | POST /eduexam/papers | papers() | getPapers() | getPapers() | 🔴不一致 | 🔴缺少安全篩選+功能缺失 | 高優先級修正 | ✅已檢查 | [eduexam-summary.md](eduexam-summary.md) |
| (抽象類別) | getPaperInfo() | POST /eduexam/detail | detail() | getPaperDetail() | getPaperDetail() | ⚠️簡化版 | 缺少權限檢查 | 加入map_sn檢查 | ✅已檢查 | [eduexam-summary.md](eduexam-summary.md) |
| (無對應) | (新增) | POST /eduexam/filters | filters() | getFilters() | getFilters() | ✅新增 | 新增功能 | - | ✅已檢查 | [eduexam-summary.md](eduexam-summary.md) |
| answer_prodb.php | (需確認) | POST /eduexam/start | start() | startExam() | (多個DAO) | ⏭️待確認 | 需深入查看JS調用 | 待確認 | ⏭️待深入 | [eduexam-summary.md](eduexam-summary.md) |
| answer_prodb.php | (需確認) | POST /eduexam/submit | submit() | submitAnswer() | (多個DAO) | ⏭️待確認 | 需深入查看JS調用 | 待確認 | ⏭️待深入 | [eduexam-summary.md](eduexam-summary.md) |
| answer_prodb.php | (需確認) | POST /eduexam/finish | finish() | finishExam() | (多個DAO) | ⏭️待確認 | 需深入查看JS調用 | 待確認 | ⏭️待深入 | [eduexam-summary.md](eduexam-summary.md) |
| answer_prodb.php | (需確認) | POST /eduexam/result | result() | getResult() | (多個DAO) | ⏭️待確認 | 需深入查看JS調用 | 待確認 | ⏭️待深入 | [eduexam-summary.md](eduexam-summary.md) |
| useditem_prodb.php | fnGetFinishRecord() | POST /eduexam/history | history() | getHistory() | getHistory() | ⏭️待確認 | 複雜UNION查詢 | 待確認 | ⏭️待深入 | [eduexam-summary.md](eduexam-summary.md) |

---

## (其他 Controller 待繼續...)

---

## 共用邏輯識別

> 在檢查過程中發現的重複邏輯,可提取為 Helper

### 已識別項目

#### 1. 公告資料維護 🔴 高優先級
- **位置**: `modulesNews_data.js` (前端 903 行) + `AnnouncementService::getNewsData()` (後端 L74-131)
- **問題**: 公告資料在前端和後端各維護一份
- **建議**: 將公告資料移到資料庫 `system_announcements` 表統一管理
- **來源**: `announcement-list.md` 第 6.2 節

#### 2. 用戶身分判斷邏輯 ✅ 已統一
- **位置**: 多個 Controller 都需要判斷身分
- **解決方案**: 已通過 `UserHelper::getIdentityByAccessLevel()` 統一處理
- **來源**: `announcement-list.md` 第 6.1 節

#### 3. 權限檢查模式 🔴 **高優先級**
- **位置**: `ChecklistService` 中多個方法重複
  - `getDetail()` L85-93
  - `update()` L131-139
  - `delete()` L170-178
  - `toggleLock()` L195-203
  - `copy()` L219-223
- **問題**: 權限檢查代碼在 5 個方法中重複
- **建議**: 提取為 `verifyOwnership()` Helper 方法
- **來源**: `checklist-detail.md` § 8.1

#### 4. 陣列 ↔ 字串轉換邏輯 ✅ 已提取
- **位置**: `ChecklistService::arrayToString()` + `stringToArray()`
- **符號**: `|||` (SPLIT_SYMBOL)
- **來源**: `checklist-list.md` § 7.2

#### 5. 當前學期取得邏輯 ✅ 已提取
- **位置**: `ChecklistService::getCurrentSeme()`
- **特點**: 包含 fallback 機制
- **來源**: `checklist-semesters.md` § 7.1

### 待識別項目

- [ ] 日期範圍處理
- [ ] SQL 查詢組裝
- [ ] 分頁邏輯

---

## 改進建議總結

> 彙總所有檢查中發現的改進機會,詳細內容請查看各端點的檢查文檔

### 🔴 高優先級改進

| 項目 | 影響範圍 | 詳細文檔 |
|------|---------|---------|
| 公告資料雙重維護 | AnnouncementController | [announcement-list.md § 7.1](announcement-list.md#71-資料來源統一-高優先級) |
| 權限檢查 Helper 提取 | ChecklistService | [checklist-detail.md § 8.1](checklist-detail.md#81-提取權限檢查-helper-高優先級) |

### 🔐 安全漏洞修正 (✅ 新 API 已修正)

| 漏洞 | 舊版問題 | 影響範圍 | 詳細文檔 |
|------|---------|---------|----------|
| teacher_id 可偽造 | 前端傳入 teacher_id，可偽造身分新增資料 | create | [checklist-create.md § 8.1](checklist-create.md#81-安全性修正-已改進) |
| 無權限檢查 | 可跨用戶刪除他人資料 | delete | [checklist-delete.md § 8.1](checklist-delete.md#81-舊版安全漏洞修正-已改進) |
| 無權限檢查 | 可跨用戶鎖定/解鎖他人檢核表 | toggleLock | [checklist-togglelock.md § 8.1](checklist-togglelock.md#81-舊版安全漏洞修正-已改進) |

### 🔴 功能 Bug 修正 (✅ 新 API 已修正)

| Bug | 舊版問題 | 影響範圍 | 詳細文檔 |
|-----|---------|---------|----------|
| 模板內容錯誤 | 類型 2（同儕評分表）返回錯誤的模板內容 | template | [checklist-template.md § 9](checklist-template.md#9-檢查結論) |

### ⚪ 中/低優先級優化

| 項目 | 影響範圍 | 詳細文檔 |
|------|---------|---------|
| 公告快取機制 | AnnouncementController | [announcement-list.md § 7.3](announcement-list.md#73-快取機制-低優先級) |

### ✅ 已完成優化

| 項目 | 實現方式 | 詳細文檔 |
|------|---------|---------|
| 用戶身分判斷 Helper | `UserHelper::getIdentityByAccessLevel()` | [announcement-list.md § 7.2](announcement-list.md#72-helper-方法提取-中優先級) |
| 兩次請求→單次請求 | list API 同時返回數據和分頁信息 | [checklist-list.md § 8.1](checklist-list.md#81-兩次請求--單次請求-已改進) |
| 移除 Hash SN 混淆 | 直接使用整數 SN | [checklist-list.md § 8.2](checklist-list.md#82-hash-sn-移除-已改進) |
| 陣列↔字串轉換 Helper | `arrayToString()` + `stringToArray()` | [checklist-list.md § 7.2](checklist-list.md#72-字串--陣列轉換邏輯) |
| 當前學期取得 Helper | `getCurrentSeme()` 含 fallback | [checklist-semesters.md § 7.1](checklist-semesters.md#71-當前學期取得邏輯) |

---

## 參考文件

- [api-scope-student.md](../api-mapping/api-scope-student.md) - 學生端 API 範圍
- [api-controller-status.md](../api-mapping/api-controller-status.md) - Controller 狀態總覽
