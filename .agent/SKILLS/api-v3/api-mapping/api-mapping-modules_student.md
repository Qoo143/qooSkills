# API-V3 對照表 - modules_student.php

**舊檔案路徑**：`modules/dashboard/modules_student.php`  
**相關檔案**：
- `modules/assignMission/mission_action_general.php` (任務查詢核心邏輯)
- `modules/srl/calender_todolist.php` (行事曆待辦事項)
- `modules/srl/todolist.php` (待辦事項元件)
- `modules/srl/prodb_calendar.php` (行事曆 AJAX 後端)

**最後更新**：2026-02-02  
**驗證狀態**：✅ 已驗證 (GAP 分析完成)  
**新版結構**：✅ Mission 模組已採用模組化設計

> **🆕 重要發現**：API v3 的 `Mission` 模組已採用新版資料夾結構設計，包含：
> - `Processor/` - 策略模式處理不同任務類型
> - `Builder/` - 建造者模式構建執行參數
> - `TaskCounter/` - 策略模式計算任務節點數
> - 詳見：[完整驗證報告](file:///C:/Users/User/.gemini/antigravity/brain/3ebaabbe-16c5-4299-a985-bb5508223dcd/verification_report.md)


---

## 端點對照清單

### 任務模組 (Mission)

| 選擇 | 舊端點（功能區塊/邏輯） | Controller 方法 | 狀態 | Controller 類型 | 說明 |
|------|-------------------------|-----------------|------|-----------------|------|
| [x] | 取得任務類型 (`mission_type` 表) | `types()` | ✅ 已完成 | MissionController（現有） | L165-167 |
| [x] | 任務清單 - 進行中 (case '1') | `list()` | ✅ 已完成 | MissionController（現有） | `status=ongoing` |
| [x] | 任務清單 - 過期 (case '2') | `list()` | ✅ 已完成 | MissionController（現有） | `status=expired` |
| [x] | 任務清單 - 已完成 (case '3') | `list()` | ✅ 已完成 | MissionController（現有） | `status=completed` |
| [x] | 家長任務清單 (case '5'/'6'/'7') | `list()` | ✅ 已完成 | MissionController（現有） | `self_practice=2` |
| [x] | 任務詳情展開 (`prodb_mission_info_seri`) | `detail()` | ✅ 已完成 | MissionController（現有） | L652-684 AJAX |
| [x] | 分頁處理 (`$_POST['page']`) | `list()` | ✅ 已完成 | MissionController（現有） | `page`, `per_page` 參數 |
| [x] | 小組長進度報告 (`assign_report`) | `groupReport()` | ✅ 已完成 | MissionController（現有） | L469-481 按鈕跳轉 |

---

### 待辦事項模組 (Todo)

| 選擇 | 舊端點（功能區塊/邏輯） | Controller 方法 | 狀態 | Controller 類型 | 說明 |
|------|-------------------------|-----------------|------|-----------------|------|
| [x] | 取得所有事件 (`get_all_events`) | `list()` | ✅ 已完成 | TodoController（現有） | prodb_calendar.php |
| [x] | 當月事件 (`get_all_events_list`) | `list()` | ✅ 已完成 | TodoController（現有） | 整合至 list |
| [x] | 當日事件 (`now_allevent`) | `list()` | ✅ 已完成 | TodoController（現有） | 整合至 list |
| [x] | 事項完成標記 (`edit_event_isDone`) | `save()` | ✅ 已完成 | TodoController（現有） | PUT + `is_done` 參數 |
| [x] | 新增事件 (`add_event`) | `save()` | ✅ 已完成 | TodoController（現有） | POST 無 sn |
| [x] | 編輯事件 (`edit_event`) | `save()` | ✅ 已完成 | TodoController（現有） | PUT 有 sn |
| [x] | 刪除事件 (`delete_event`) | `delete()` | ✅ 已完成 | TodoController（現有） | DELETE 軟刪除 |
| [x] | 單筆查詢 | `detail()` | ✅ 已完成 | TodoController（現有） | 新增功能 |

---

### 公告模組 (Announcement)

| 選擇 | 舊端點（功能區塊/邏輯） | Controller 方法 | 狀態 | Controller 類型 | 說明 |
|------|-------------------------|-----------------|------|-----------------|------|
| [x] | 公告列表 (`modulesNews_data.js`) | `list()` | ⚠️ 待確認 | AnnouncementController（現有） | 舊版資料來自 JS 靜態檔 |

---

### 個人設定模組 (PersonalConfig) - 待建立

| 選擇 | 舊端點（功能區塊/邏輯） | Controller 方法 | 狀態 | Controller 類型 | 說明 |
|------|-------------------------|-----------------|------|-----------------|------|
| [x] | 取得個人設定 (`getPersonalConfig`) | `get()` | ✅ 已完成 | PersonalConfigController（新建） | POST `/personal-config/get` |
| [x] | 更新個人設定 (`updatePersonalStatus`) | `update()` | ✅ 已完成 | PersonalConfigController（新建） | PUT `/personal-config/update` |

**設定項目：**
- `item_card_or_list`: 任務卡片/列表切換 (1=卡片, 2=列表)
- `calendar`: 行事曆顯示/隱藏 (0=隱藏, 1=顯示)

---

## GAP 分析結果

### ✅ 符合項目

| 特徵 | 舊檔案邏輯 | v3 API 實作 | 驗證 |
|------|-----------|-------------|------|
| 任務狀態篩選 | `dowh_val` (1=進行中, 2=過期, 3=完成) | `status` (ongoing, expired, completed) | ✅ |
| 指派來源篩選 | `sSelfPractice` (0/1/2) | `self_practice` (0,1,2) | ✅ |
| 任務類型篩選 | `missionType` | `mission_type` | ✅ |
| 時間範圍 | 固定 9 個月 (`$four_m`) | `months_ago` 參數 (預設 9) | ✅ |
| 分頁 | `$_POST['page']` + LIMIT 15 | `page`, `per_page` 參數 | ✅ |
| 教師名稱 | `teachername()` 函數 | LEFT JOIN `user_info` | ✅ 優化 |
| 小組長資料 | `mission_group_leader` JOIN | `mission_group_leader` JOIN + `group_ids`, `group_names` | ✅ |
| 小組進度報告 | `assign_report.php` | `groupReport()` API | ✅ |

### ❌ 缺失項目

| 特徵 | 舊檔案邏輯 | v3 API 實作 | 行動 |
|------|-----------|-------------|------|
| 個人設定 (卡片/列表) | `user_person_config.item_card_or_list` | ✅ 已實作 | `PersonalConfigController::get/update` |
| 行事曆顯示設定 | `user_person_config.calendar` | ✅ 已實作 | `PersonalConfigController::get/update` |

---

## 狀態說明

- `⏳ 待處理`：尚未開始
- `🚧 進行中`：正在開發
- `✅ 已完成`：已完成並測試
- `⚠️ 待確認`：功能存在但需確認資料來源

---

## 開發記錄

### 2026-02-02
- [x] 執行新版結構驗證 (發現 Mission 模組已採用模組化設計)
- [x] 深度分析舊檔案邏輯 (含 `mission_action_general.php` SQL 查詢)
- [x] 驗證 Controller、Service、DAO 三層架構
- [x] 確認策略模式、建造者模式、工廠模式應用
- [x] 建立完整驗證報告

### 2026-01-30
- [x] 建立正式版對照表
- [x] 完成 GAP 分析驗證
- [x] 確認任務模組所有端點已完成
- [x] 確認小組長功能已實作
- [x] 建立 PersonalConfigController (DAO/Service/Controller)

---

## 後續步驟

1. ~~使用 `/api-write` workflow 建立 PersonalConfigController~~ ✅ 已完成
2. ~~確認 AnnouncementController 資料來源~~ ⚠️ 標記為"待確認"
3. 確認 QuestionnaireController 是否涵蓋舊版問卷連結功能 (L95-111)
4. 檢查課程包任務進度同步邏輯 (L671-730 CURL) 是否已遷移
5. 使用 `/api-review` workflow 進行 code review
