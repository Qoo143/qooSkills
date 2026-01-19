# modules_student.php 功能對照表

學生儀表板頁面 - 包含任務列表、行事曆待辦事項、公告功能。

---

## 原始程式檔案

| 檔案 | 路徑 | 說明 |
|------|------|------|
| modules_student.php | `modules/dashboard/modules_student.php` | 主頁面 (前端 + 部分邏輯) |
| mission_action_general.php | `modules/assignMission/mission_action_general.php` | 任務查詢核心邏輯 |
| calender_todolist.php | `modules/srl/calender_todolist.php` | 行事曆待辦事項整合 |
| todolist.php | `modules/srl/todolist.php` | 待辦事項元件 |
| prodb_calendar.php | `modules/srl/prodb_calendar.php` | 行事曆 AJAX 後端 |

---

## 功能拆解與 API 對應

### 1. 任務模組 (Mission)

#### 舊程式功能

| 功能 | 舊程式位置 | 實作方式 |
|------|------------|----------|
| 取得任務類型 | `mission_action_general.php:142-153` | SQL 查詢 `mission_type` |
| 任務清單 (進行中) | `mission_action_general.php:569-649` | switch case '1' |
| 任務清單 (過期) | `mission_action_general.php:160-235` | switch case '2' |
| 任務清單 (已完成) | `mission_action_general.php:237-303` | switch case '3' |
| 家長任務清單 | `mission_action_general.php:346-567` | switch case '5'/'6'/'7' |
| 任務詳情展開 | `modules_student.php:652-684` | AJAX `prodb_mission_info_seri` |
| 分頁處理 | `mission_action_general.php` | `$_POST['page']` + LIMIT |

#### v3 API 對應

| v3 API | Controller 方法 | 說明 |
|--------|-----------------|------|
| `POST /mission/types` | `MissionController::types()` | ✅ 對應取得任務類型 |
| `POST /mission/list` | `MissionController::list()` | ✅ 整合進行中/過期/已完成/家長篩選 |
| `POST /mission/detail` | `MissionController::detail()` | ✅ 對應任務詳情展開 |

#### 參數對照

| 舊參數 | 新參數 | 值對應 |
|--------|--------|--------|
| `$_POST['sSelfPractice']` | `self_practice` | 0=老師, 1=自己, 2=家長 |
| `$_POST['dowh_val']` | `status` | 1→ongoing, 2→expired, 3→completed |
| `$_POST['missionType']` | `mission_type` | all/type_id |
| `$_POST['page']` | `page` | 分頁頁碼 |
| - | `per_page` | 🆕 每頁數量 (舊版固定 15) |
| - | `months_ago` | 🆕 查詢月數範圍 (舊版固定 9 個月) |

#### 設計異動

> [!NOTE]
> v3 API 將家長篩選 (舊 case 5/6/7) 整合到相同的 `status` 參數，後端透過 `self_practice=2` 自動處理。

#### SQL 比較

| 項目 | 舊程式 | v3 API | 差異說明 |
|------|--------|--------|----------|
| **主要 JOIN** | `mission_info` → `mission_stud_record` → `seme_student` → `mission_type` + `mission_group_leader` + `seme_group` | `mission_info` → `mission_stud_record` → `seme_student` → `mission_type` + `user_info` | ⚠️ v3 移除小組長相關 JOIN |
| **班級對象篩選** | `a.target_id = CONCAT(c.organization_id,'-',c.grade,LPAD(c.class,2,0))` 加上 `$sTC_SqlAdd` (自組班) + `$sRC_SqlAdd` (學扶班) | 透過 `mission_stud_record.user_id` 直接篩選 | ✅ v3 簡化：不再重複計算班級組合 |
| **教師名稱** | 另外查詢 `teachername()` 函數 | LEFT JOIN `user_info` 一次取得 | ✅ v3 減少一次 DB 查詢 |
| **小組長資料** | `GROUP_CONCAT(mgl.group_id)`, `GROUP_CONCAT(sg.group_nm)` | ❌ 未實作 | ⚠️ 功能缺失 |
| **時間範圍** | `$four_m` 固定 9 個月 | `$monthsAgo` 參數可調 | ✅ v3 更靈活 |

> [!WARNING]
> v3 缺少小組長 (`mission_group_leader`) 相關欄位，若前端需顯示「查看小組進度」功能，需補充實作。

---

### 2. 公告模組 (Announcement)

#### 舊程式功能

| 功能 | 舊程式位置 | 實作方式 |
|------|------------|----------|
| 公告列表 | `calender_todolist.php:163-207` | 前端 JS 讀取 `modulesNews_data.js` |
| 公告詳情 | `calender_todolist.php:247-268` | 前端 JS 展示 |

#### v3 API 對應

| v3 API | Controller 方法 | 說明 |
|--------|-----------------|------|
| `POST /announcement/list` | `AnnouncementController::list()` | ✅ 對應公告列表 |

#### 設計異動

> [!IMPORTANT]
> 舊版公告資料來自前端靜態 JS 檔案 (`modulesNews_data.js`)，v3 改為後端 API 動態取得，資料來源需確認是否需要建立資料庫表。

---

### 3. 待辦事項模組 (Todo)

#### 舊程式功能

| 功能 | 舊程式位置 | 實作方式 |
|------|------------|----------|
| 待辦清單 | `prodb_calendar.php` | AJAX `act=get_event` |
| 新增待辦 | `modules_student.php:884-926` | AJAX `act=add_event` |
| 更新待辦 | `prodb_calendar.php` | AJAX `act=update_event` |
| 刪除待辦 | `prodb_calendar.php` | AJAX `act=delete_event` |
| 任務加入行事曆 | `modules_student.php:744-812` | 開窗 → add_event |

#### v3 API 對應

| v3 API | Controller 方法 | 說明 |
|--------|-----------------|------|
| `POST /todo/list` | `TodoController::list()` | ✅ 對應待辦清單 |
| `POST /todo/detail` | `TodoController::detail()` | ✅ 對應單筆查詢 |
| `POST /todo/save` | `TodoController::save()` | ✅ 整合新增/更新 (依 sn 判斷) |
| `DELETE /todo/delete` | `TodoController::delete()` | ✅ 對應刪除 (軟刪除) |

#### 參數對照

| 舊參數 | 新參數 | 說明 |
|--------|--------|------|
| `events` | `events` | 事件內容 |
| `start_time` | `start_time` | 開始時間 |
| `end_time` | `end_time` | 結束時間 |
| `all_day` | `all_day` | 全天事件 (0/1) |
| `event_color` | `event_color` | 標籤顏色 (0-6) |
| - | `is_done` | 🆕 完成狀態 |

#### 設計異動

> [!NOTE]
> v3 將新增與更新整合為 `/todo/save`，透過是否帶 `sn` 參數區分：有 sn 為更新 (PUT)，無 sn 為新增 (POST)。

---

## 未對應功能

| 舊功能 | 說明 | 狀態 |
|--------|------|------|
| 個人設定 (卡片/列表切換) | `prodb_calendar.php` `updatePersonalStatus` | ❌ 未實作 |
| 行事曆顯示/隱藏設定 | `calender_todolist.php:271-293` | ❌ 未實作 |
| 小組長進度報告 | `modules_student.php:469-481` | ❌ 未實作 |
| 課程包任務狀態同步 | `mission_action_general.php:671-730` | ❌ 未實作 |

---

## 驗證檢查清單

- [x] Mission types API 功能對應
- [x] Mission list API 支援所有篩選條件
- [x] Mission detail API 功能對應
- [ ] Announcement API 資料來源確認
- [x] Todo CRUD 功能完整
- [ ] 個人設定 API 需補充
- [ ] 課程包任務同步需評估
