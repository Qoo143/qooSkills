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
**開發原則**：⚠️ 按照原本功能性，只轉換成 API，不做額外優化

> **🆕 重要發現**：API v3 的 `Mission` 模組已採用新版資料夾結構設計，包含：
> - `Processor/` - 策略模式處理不同任務類型
> - `Builder/` - 建造者模式構建執行參數
> - `TaskCounter/` - 策略模式計算任務節點數
> - 詳見：[完整驗證報告](file:///C:/Users/User/.gemini/antigravity/brain/3ebaabbe-16c5-4299-a985-bb5508223dcd/verification_report.md)

> **⚠️ 外包任務類型**：以下任務類型由外包廠商負責：
> - 類型 2 (學習問卷)
> - 類型 7
> - 類型 8 (學科素養)
> - 類型 12
> - 類型 17
> - 類型 18
> - 類型 19
>
> **處理原則**：
> 1. 若能實現轉為 API，仍應實作（優先）
> 2. 若外包程式可直接呼叫 `new XxxService()` 或 `new XxxDao()`，可考慮讓外包直接使用 v3 類別
> 3. 最終目標是統一架構，減少重複代碼


---

## 參數對照表

### 任務清單篩選參數

| 舊參數名稱 | 新參數名稱 | 值對照 | 說明 |
|-----------|-----------|--------|------|
| `dowh_val` | `status` | `1` → `ongoing`<br>`2` → `expired`<br>`3` → `completed` | 任務狀態篩選 |
| `sSelfPractice` | `self_practice` | `0` → `0` (老師指派)<br>`1` → `1` (自己指派)<br>`2` → `2` (家長/大學伴指派) | 指派來源篩選 |
| `missionType` | `mission_type` | `all` → `all`<br>`{id}` → `{id}` | 任務類型篩選 |
| `page` | `page` | 直接對應 | 分頁頁碼 |
| - | `per_page` | 預設 `15` | 每頁筆數 (舊版固定 15) |

### 任務詳情參數

| 舊參數名稱 | 新參數名稱 | 說明 |
|-----------|-----------|------|
| `mission_sn` | `mission_sn` | 任務序號 (hash 格式) |

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
| [x] | 公告列表 (`modulesNews_data.js`) | `list()` | ✅ 已完成 | AnnouncementController（現有） | 目前先寫死資料，與舊版靜態檔一致 |

> **📌 公告模組說明**：舊版公告資料來自 `modulesNews_data.js` 靜態檔案，目前 API v3 先寫死資料回傳，未來可視需求遷移至資料庫。

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

### ✅ 已完成項目（新增）

| 特徵 | 舊檔案邏輯 | v3 API 實作 | 實作位置 | 完成日期 |
|------|-----------|-------------|---------|---------|
| 自組班級任務篩選 | `teacher_class_member` (L95-111) | ✅ 已實作 | `MissionDao::getTeacherClassIds()` | 2026-02-02 |
| 學習扶助班任務篩選 | `remedial_student` (L112-129) | ✅ 已實作 | `MissionDao::getRemedialClassIds()` | 2026-02-02 |
| target_id 檢查邏輯 | `mission_action_general.php` L205-208 | ✅ 已實作 | `MissionDao::getMissionsByStatus()` L92-94 | 2026-02-02 |

### 🏢 外包負責項目（不實作）

| 特徵 | 舊檔案邏輯 | 說明 |
|------|-----------|------|
| 問卷連結查詢 | `QUESTIONNAIRE_Data()` (L95-111) | 任務類型 2 由外包負責 |
| 素養任務節點 | `getLiteracySrc` | 任務類型 8 由外包負責 |
| 其他外包任務 | 類型 7, 12, 17, 18, 19 | 由外包廠商處理 |

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

### 已完成
- [x] 建立 PersonalConfigController (DAO/Service/Controller)
- [x] GAP 分析驗證
- [x] Code Review 完成
- [x] 外包任務類型確認
- [x] 實作自組班級任務篩選（2026-02-02）
- [x] 實作學習扶助班任務篩選（2026-02-02）
- [x] 補充 target_id 檢查邏輯（2026-02-02）

### 不實作項目（外包負責）
- [x] 問卷連結功能（類型 2）
- [x] 素養任務節點（類型 8）
- [x] 其他外包任務類型（7, 12, 17, 18, 19）

---

## 資料遷移檢查清單

### 資料表完整性
- [ ] `mission_info` - 確認所有任務資料完整
- [ ] `mission_type` - 確認任務類型定義正確（排除外包類型）
- [ ] `mission_stud_record` - 確認學生任務記錄完整
- [ ] `mission_group_leader` - 確認小組長資料正確
- [ ] `user_person_config` - 確認使用者設定初始值
- [ ] `teacher_class_member` - 自組班級資料完整
- [ ] `remedial_student` - 學習扶助班資料完整
- [ ] `srl_calendar` - 行事曆待辦事項資料

### 功能驗證
- [ ] 一般任務顯示正常（類型 1, 3, 4, 5, 6 等）
- [ ] 自組班級任務顯示正常（class_type = 1）
- [ ] 學習扶助班任務顯示正常（class_type = 2）
- [ ] 小組長功能運作正常
- [ ] 任務詳情展開完整
- [ ] 分頁功能正常
- [ ] 任務狀態篩選正確（進行中/過期/已完成）
- [ ] 指派來源篩選正確（老師/自己/家長）

### 外包任務類型（不需遷移）
- [x] 類型 2 (學習問卷) - 外包負責
- [x] 類型 7 - 外包負責
- [x] 類型 8 (學科素養) - 外包負責
- [x] 類型 12 - 外包負責
- [x] 類型 17 - 外包負責
- [x] 類型 18 - 外包負責
- [x] 類型 19 - 外包負責
