# API-V3 對照表：Srl Controller

## Controller 資訊
- **檔案**: `ADLAPI/v3/App/Controller/SrlController.php`
- **Service**: `ADLAPI/v3/App/Service/SrlService.php`
- **DAO**: `ADLAPI/v3/App/Dao/SrlDao.php`

---

## 端點對照總覽

| # | V3 端點 | HTTP | 舊檔案 | 舊動作 | 狀態 |
|---|---------|------|--------|--------|------|
| 1 | `/srl/calendar` | POST | `modules/srl/prodb_calendar.php` | `get_all_events` | ⚠️ 有差異 |
| 2 | `/srl/notes` | POST | `modules/srl/prodb_learning_note.php` | `getNotes4Student` / `getNotes4Teacher` | 🔴 資料表不同 |
| 3 | `/srl/note-detail` | POST | `modules/srl/prodb_learning_note.php` | (無直接對應) | 🔴 資料表不同 |
| 4 | `/srl/checklist` | POST | `modules/srl/prodb_checklists_new.php` | `loadData` | 🔴 資料表不同 |
| 5 | `/srl/questions` | POST | `modules/srl/prodb_learning_ask.php` | `getQuestions4Student` / `getQuestions4Teacher` | 🔴 資料表不同 |

---

## 重大架構差異：資料表不匹配

V3 DAO 查詢的資料表與舊版使用的資料表存在根本性差異：

| 功能 | V3 查詢資料表 | 舊版實際資料表 | 說明 |
|------|-------------|-------------|------|
| 行事曆 | `srl_calendar` | `srl_calendar` | ✅ 一致 |
| 學習筆記 | `srl_learning_note` | `video_note` + `video_note_favorite` + `video_note_feedback` + `video_note_recommend` | 🔴 完全不同 |
| 檢核表 | `srl_checklist` | `check_list_table` | 🔴 不同資料表，且舊版為教師功能 |
| 學生提問 | `srl_learning_ask` | `video_noteask` + `video_noteask_plus` | 🔴 完全不同 |

> **注意**：`srl_learning_note`、`srl_checklist`、`srl_learning_ask` 這些資料表可能是 V3 新建的簡化表，或者資料庫中實際不存在。需要確認資料庫 schema。DAO 檔案頂部注釋已標註 `// 這邊資料庫可能有問題`。

---

## 1. 行事曆 `/srl/calendar`

### 參數對照
| V3 參數 | 舊參數 | 說明 |
|---------|--------|------|
| `year` (optional) | `$_POST['start']` (完整日期時間) | V3 只傳年月；舊版傳完整起始日期 |
| `month` (optional) | `$_POST['end']` (完整日期時間) | V3 只傳年月；舊版傳完整結束日期 |

### 邏輯對照
| 功能 | V3 | 舊 (action) | 差異 |
|------|------|------------|------|
| 依月份取事件 | `getCalendarEvents()` 計算月初到月末 | `get_all_events` 接受任意時間範圍 | ⚠️ V3 只能按月查詢，舊版任意範圍 |
| 當月事件 | `getCurrentMonthEvents()` (未被 Controller 使用) | `get_all_events_list` | ✅ SQL 一致 |
| 特定日期事件 | — | `now_allevent` | 🔴 V3 缺少 |
| 新增事件 | — | `add_event` | 🔴 V3 缺少 |
| 編輯事件 | — | `edit_event` | 🔴 V3 缺少 |
| 刪除事件 | — | `delete_event` (is_delete=1) | 🔴 V3 缺少 |
| 標記完成 | — | `edit_event_isDone` | 🔴 V3 缺少 |
| 到期提醒 | — | `reminder_events` (今/明天結束) | 🔴 V3 缺少 |

### SQL 對照
- ✅ V3 `getCalendarEvents()` 的三段式日期區間查詢 (`start > X AND start < Y OR start < X AND end > Y OR end > X AND end < Y`) 與舊版 `get_all_events` 一致
- ✅ V3 `getCurrentMonthEvents()` SQL 與舊版 `get_all_events_list` 一致
- ⚠️ V3 回傳不含 `string2hash()` 加密的 sn（舊版使用 `string2hash` 加密回傳）

### 缺失邏輯
- 🔴 **無寫入操作**: 行事曆的 CRUD（新增/編輯/刪除/標記完成）完全缺失，只有讀取
- ⚠️ **查詢範圍受限**: V3 只能按自然月查詢，無法查特定日期或任意時間範圍
- ⚠️ **無到期提醒**: 舊版 `reminder_events` 查今明兩天到期事件，用於 dashboard 提醒

---

## 2. 學習筆記 `/srl/notes`

### 🔴 根本問題：資料表不同

V3 查詢 `srl_learning_note`（簡化表），但舊版使用 `video_note` 系統：

| 特性 | V3 (srl_learning_note) | 舊版 (video_note 體系) |
|------|----------------------|---------------------|
| 核心資料表 | srl_learning_note | video_note |
| 社交功能 | 無 | video_note_favorite (喜歡/收藏) |
| 回饋功能 | 無 | video_note_feedback (留言回饋) |
| 推薦功能 | 無 | video_note_recommend (教師推薦) |
| 代幣功能 | 無 | role_coin_total + role_coin_history |
| 關聯影片 | 無 | video_concept_item (影片+知識點) |
| 關聯科目 | 無 | map_node + map_info (科目+知識點名稱) |
| 班級/小組 | 無 | seme_student + seme_teacher_subject (班級歸屬) |
| 角色區分 | 無 | 教師版(getNotes4Teacher) / 學生版(getNotes4Student) |

### 舊版功能清單 (prodb_learning_note.php, 13 個 action)

| 功能 | action | V3 對應 | 說明 |
|------|--------|---------|------|
| 教師取筆記 | `getNotes4Teacher` | 無 | 含我的筆記、我的收藏、班級筆記（3個UNION查詢） |
| 學生取筆記 | `getNotes4Student` | 無 | 含我的筆記、我的收藏、班級筆記（3個UNION查詢） |
| 按讚 | `updateLike` | 無 | INSERT/UPDATE video_note_favorite.is_like |
| 收藏 | `updateFavorite` | 無 | INSERT/UPDATE video_note_favorite.is_favorite |
| 查看回饋 | `searchFeedback` | 無 | 查 video_note_feedback + user_info + role_info |
| 刪除回饋 | `deleteFeedback` | 無 | 軟刪除 (is_delete=1) |
| 寫入回饋 | `sentFeedback` | 無 | INSERT video_note_feedback |
| 公開/隱藏 | `displayNote` | 無 | UPDATE is_public |
| 給代幣 | `giveCoins` | 無 | 教師給學生筆記代幣（role_coin_total + role_coin_history） |
| 推薦筆記 | `recommendNotes` | 無 | DELETE + INSERT video_note_recommend |
| 刪除筆記 | `deleteNotes` | 無 | UPDATE is_public=2 |
| 推薦數量 | `getTeacherRecommend` | 無 | 查 90 天內被推薦的未公開筆記數 |
| 影片推薦筆記 | `getRecommendNotes` | 無 | 特定影片的推薦筆記列表 |

### V3 現有實作
- `getNotes()`: 只查 `srl_learning_note` 的 sn, mission_sn, note_title, note_content, create_time, update_time
- `getNoteDetail()`: 同上，加上 WHERE sn = :sn
- 無任何社交、推薦、代幣功能

---

## 3. 檢核表 `/srl/checklist`

### 🔴 根本問題：資料表與使用對象不同

| 特性 | V3 (srl_checklist) | 舊版 (check_list_table) |
|------|-------------------|----------------------|
| 資料表 | srl_checklist | check_list_table |
| 使用者 | user_id（學生） | teacher_id（教師功能） |
| 資料結構 | sn, mission_sn, checklist_title | check_list_table_sn, seme, type, title_name, qusetion (@XX@ 分隔), score (@XX@ 分隔), public |

### 舊版功能清單 (prodb_checklists_new.php, 8 個 action)

| 功能 | action | V3 對應 | 說明 |
|------|--------|---------|------|
| 學期列表 | `getSheetSemes` | 無 | 取得教師有檢核表的學期 |
| 分頁數 | `getPages` | 無 | 按類型/學期篩選，計算分頁 |
| 載入資料 | `loadData` | 部分 | V3 查不同表且無分頁/篩選 |
| 新增 | `inputdata` | 無 | INSERT check_list_table |
| 刪除 | `deleteData` | 無 | 軟刪除 (is_delete) |
| 鎖定/解鎖 | `updatelockdata` | 無 | UPDATE public 欄位 |
| 編輯 | `updateeditdata` | 無 | UPDATE（需驗證未鎖定且為本人） |
| 複製 | `copydata` | 無 | 複製一份新的檢核表 |
| 預設模板 | `loadTableContent` | 無 | 4 種預設模板（檢核單/同儕評分/組間評分/組內評分） |

### 缺失邏輯
- 🔴 **教師功能完全缺失**: 舊版是教師建立檢核表功能，V3 用 user_id 查詢
- 🔴 **無 CRUD**: 只有讀取
- ⚠️ **無預設模板**: 舊版提供 4 種預設模板，V3 無此功能
- ⚠️ **無分頁**: 舊版有分頁功能（每頁 10 筆）

---

## 4. 學生提問 `/srl/questions`

### 🔴 根本問題：資料表完全不同

| 特性 | V3 (srl_learning_ask) | 舊版 (video_noteask 體系) |
|------|----------------------|--------------------------|
| 核心表 | srl_learning_ask | video_noteask |
| 回覆表 | 無 | video_noteask_plus |
| 欄位 | question_title, question_content, answer_content, is_answered | question_content, stop_time, file_path, upload_server, group_id, grade_class, is_show |
| 角色區分 | 無 | 教師4個查詢 / 學生4個查詢 |
| 小組 | 無 | user_group + seme_group |
| 影片關聯 | 無 | video_concept_item + map_node |

### 舊版功能清單 (prodb_learning_ask.php)

| 功能 | action | V3 對應 | 說明 |
|------|--------|---------|------|
| 教師取提問 | `getQuestions4Teacher` | 無 | 4 組查詢：我的提問、學生提問、小組提問、已刪除提問 |
| 學生取提問 | `getQuestions4Student` | 部分 | 4 組查詢：我的提問、我的回覆、小組提問、最新提問 |
| 保存 Session | `saveSession` | 無 | 保存提問的 session 資料 |

### 舊版查詢複雜度
- 每個查詢包含 2-4 個 UNION
- 教師版：透過 `seme_teacher_subject` 查科任班 + 透過 `user_info` 查班導班
- 學生版：透過 `seme_student` 查同班同學 + 透過 `user_group` 查小組 + 透過 `seme_teacher_subject` 查班級教師
- 回覆功能透過 `video_noteask_plus` 表
- 圖片處理：`file_path` 含 `@XX@` 分隔的多張圖片路徑，需處理伺服器路徑轉換

### V3 現有實作
- `getQuestions()`: 只查 `srl_learning_ask` 的 sn, mission_sn, question_title, question_content, answer_content, is_answered
- 無任何回覆、小組、影片關聯功能

---

## 關鍵差異總結

### 🔴 嚴重問題

1. **學習筆記資料表錯誤**: V3 查 `srl_learning_note`，舊版用 `video_note` 系統（含 favorite, feedback, recommend 4 張關聯表）。整個社交筆記系統（按讚、收藏、回饋、推薦、代幣）完全缺失

2. **檢核表資料表錯誤 + 角色錯誤**: V3 查 `srl_checklist` 用 user_id，舊版用 `check_list_table` 用 teacher_id。這是教師功能，V3 當成學生功能

3. **學生提問資料表錯誤**: V3 查 `srl_learning_ask`，舊版用 `video_noteask` + `video_noteask_plus`。整個影片關聯提問與回覆系統缺失

4. **所有模組皆為唯讀**: 行事曆 CRUD、筆記社交互動、檢核表管理、提問回覆 —— 全部寫入操作缺失

### 🟡 中度問題

5. **行事曆只能按月查詢**: 舊版支援任意時間範圍，V3 固定按年月

6. **無角色區分**: 舊版嚴格區分教師/學生視角（不同的 SQL 查詢路徑），V3 統一用 userId 查

7. **無小組功能**: 舊版提問/筆記都有小組維度（user_group + seme_group），V3 無此概念

8. **無 SN 加密**: 舊版使用 `string2hash()` 加密回傳 sn，`hash2string()` 解密接收，V3 直接使用原始 sn

9. **DAO 註釋已標記問題**: `SrlDao.php` 第 4 行 `// 這邊資料庫可能有問題`，開發者已意識到問題

### 🟢 設計改進

10. **統一 API 入口**: 舊版散落在 4 個 prodb_ 檔案，V3 集中管理
11. **BaseDao 繼承**: V3 使用 BaseDao 統一管理 DB 連線

### 📋 舊版功能未實作

#### 行事曆 CRUD (prodb_calendar.php)
| 功能 | action | 說明 |
|------|--------|------|
| 新增事件 | `add_event` | INSERT srl_calendar |
| 編輯事件 | `edit_event` | UPDATE events, start_time, end_time, all_day, eventColor |
| 刪除事件 | `delete_event` | 軟刪除 is_delete=1 |
| 標記完成 | `edit_event_isDone` | UPDATE is_done |
| 查特定日期 | `now_allevent` | 查指定日期的所有事件 |
| 到期提醒 | `reminder_events` | 查今/明天到期的事件 |

#### 影片筆記社交系統 (prodb_learning_note.php)
| 功能 | 說明 |
|------|------|
| 按讚/取消讚 | video_note_favorite.is_like |
| 收藏/取消收藏 | video_note_favorite.is_favorite |
| 查看/寫入/刪除回饋 | video_note_feedback |
| 公開/隱藏筆記 | video_note.is_public |
| 教師推薦筆記 | video_note_recommend |
| 教師給代幣 | role_coin_total + role_coin_history |
| 刪除筆記 | is_public=2 |
| 取推薦筆記 | 特定影片的推薦筆記 |

#### 檢核表管理 (prodb_checklists_new.php)
| 功能 | 說明 |
|------|------|
| 新增檢核表 | INSERT check_list_table (教師功能) |
| 編輯/刪除/鎖定 | UPDATE check_list_table |
| 複製檢核表 | 複製一份新的 |
| 預設模板 | 4 種類型預設內容 |
| 學期篩選+分頁 | getSheetSemes + getPages |

#### 提問回覆系統 (prodb_learning_ask.php)
| 功能 | 說明 |
|------|------|
| 新增提問 | (在前端其他模組處理) |
| 回覆提問 | video_noteask_plus |
| 小組提問 | user_group + seme_group 關聯 |
| 教師看學生提問 | 按科任班/導師班分組 |
| 刪除的提問 | is_show=0 |

#### 其他 SRL 模組（未在 V3 中）
| 舊版檔案 | 說明 |
|---------|------|
| `prodb_learning_regulated_fillout.php` | 學習反思/目標設定表單 |
| `todolist.php` | 待辦事項（與行事曆關聯但獨立模組） |
| `calender_todolist.php` | 行事曆+待辦整合頁 |
| `checklists_scoresheet_function_new.php` | 檢核表評分功能 |
| `learning_regulated_groupreport.php` | 小組報告 |

---

## 資料表結構對照

### V3 使用的資料表（可能為簡化/新建表）
| 資料表 | 欄位 |
|--------|------|
| srl_calendar | sn, user_id, events, start_time, end_time, all_day, eventColor, is_done, is_delete |
| srl_learning_note | sn, user_id, mission_sn, note_title, note_content, create_time, update_time, is_delete |
| srl_checklist | sn, user_id, mission_sn, checklist_title, create_time, update_time, is_delete |
| srl_learning_ask | sn, user_id, mission_sn, question_title, question_content, answer_content, is_answered, create_time, update_time |

### 舊版實際使用的資料表
| 資料表 | 說明 |
|--------|------|
| srl_calendar | 行事曆 ✅ 一致 |
| video_note | 影片筆記（含 video_item_sn, stop_time, content, is_public, user_id） |
| video_note_favorite | 筆記喜歡/收藏 |
| video_note_feedback | 筆記回饋留言 |
| video_note_recommend | 教師推薦 |
| check_list_table | 檢核表（teacher_id, type, title_name, qusetion, score, public, seme） |
| video_noteask | 影片提問（user_id, video_item_sn, question_content, stop_time, file_path, group_id） |
| video_noteask_plus | 提問回覆（ask_sn, user_id, reply_time, is_show） |
| role_coin_total | 代幣累計 |
| role_coin_history | 代幣紀錄 |
