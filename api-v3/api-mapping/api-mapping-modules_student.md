# API-V3 對照表 - modules_student.php

**舊檔案路徑**：`modules/dashboard/modules_student.php`  
**最後更新**：2026-01-30  
**主要 Controller**：MissionController (主要), TodoController (自訂), QuestionnaireController, StudentController

---

## 目的

本文件記錄 `modules_student.php` 的 API 轉換規劃，經過程式碼比對驗證 (Gap Analysis)，已修正對應的 Controller。

## 端點對照清單

| 選擇 | 舊端點（功能區塊/邏輯） | Controller 方法 | 狀態 | Controller 類型 | 說明 |
|------|-------------------|----------------|------|----------------|------|
| [x] | **Page Load / Filter** (Get Mission List) | `list()` | ✅ 已完成 | MissionController（現有） | **更正對應**：老師指派與系統任務清單 (`mission_action_general.php`) |
| [x] | **AJAX**: `getmissiondetail` | `detail()` | ✅ 已完成 | MissionController（現有） | **更正對應**：任務詳情 (`MissionService::getMissionDetail`) |
| [x] | **Filter Parameters** (`dowh_val`, `sSelfPractice`) | `list()` (w/ params) | ✅ 已完成 | MissionController（現有） | 支援 `status` (expired/ongoing/completed) 與 `self_practice` |
| [x] | **Action**: Create Personal Todo | `create()` | ✅ 已完成 | TodoController（現有） | 學生自訂待辦事項 (若有此功能) |
| [x] | **Function**: `QUESTIONNAIRE_Data` | `status()` / `detail()` | ✅ 已完成 | QuestionnaireController（現有） | 取得問卷連結與狀態 |
| [ ] | **Logic**: Personal Config | `profile()` | ⏳ 待處理 | StudentController（新建） | 取得個人設定 (顯示模式、行事曆設定) |
| [x] | **Logic**: Group Leader View | `list()` (w/ group param) | ✅ 已完成 | MissionController | 已實作 `mission_group_leader` JOIN 與 `groups` 回傳欄位。 |
| [x] | **Action**: Group Report (View Group Progress) | `groupReport()` | ✅ 已完成 | MissionController | **Target**: `assign_report.php`。點擊「查看小組進度」後，根據 `group_id` 與 `mission_sn` 查詢組員完成狀況。 |

---

## 詳細邏輯分析與驗證 (Verification Results)

### 1. 任務清單 (Mission List) - **已更正**
- **舊邏輯**: `modules_student.php` 引用 `mission_action_general.php`，處理老師指派 (`mission_info`) 與相關篩選。
- **誤判**: 原本誤以為是 `TodoController` (個人待辦)。
- **驗證**: `MissionController::list` 支援 `self_practice`, `status`, `mission_type`，完全對應舊檔案的 `sSelfPractice`, `dowh_val`, `missionType`。
- **Done**: 已在 `MissionDao` 加入 `mission_group_leader` 查詢，並在 `MissionService` 回傳 `groups` 資訊。

### 2. 任務詳情 (Mission Detail) - **已更正**
- **舊邏輯**: AJAX `act=getmissiondetail`。
- **驗證**: `MissionController::detail` 提供任務詳細資訊。

### 3. 個人設定 (Personal Config)
- **舊邏輯**: `modules_student.php` (L114) 讀取 `user_person_config`。
- **驗證**: 需建立 `StudentController` 或 `UserProfileController` 來處理此部分。

---

## 狀態說明

- `⏳ 待處理`：尚未開始
- `🚧 進行中`：正在開發
- `✅ 已完成`：已完成並測試
- `❓ 待確認`：Controller 實作可能缺失，需進一步檢查

## Controller 類型說明

- **現有**：寫入已存在的 Controller（如 TodoController、MissionController）
- **新建**：需要建立新的 Controller

---

## 開發記錄

### 2026-01-30
- [x] **執行驗證流程 (/api-verify-mapping)**：發現 `TodoController` 僅處理個人待辦，無法與 `modules_student.php` 的系統任務邏輯匹配。
- [x] **修正 Mapping**：將主要列表與詳情功能重導向至 `MissionController`。
- [x] **修正 GAP**: 補完小組長 (Group Leader) 功能，支援 `groups` 回傳。
- [x] **新增 API**: 實作 `groupReport()`，對應 legacy `assign_report.php` 的小組進度查詢功能。
