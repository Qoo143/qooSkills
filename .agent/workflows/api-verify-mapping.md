---
description: 驗證 API-V3 對照表與實作正確性
---

# Workflow: 驗證 API-V3 對照表與實作正確性

本工作流程用於檢查已建立的 API 對照表是否準確，並驗證 Controller 的實作是否真正涵蓋了舊 PHP 檔案的完整邏輯（尤其是篩選條件、特殊判斷與資料結構）。

## 核心目的

避免「看起來有名稱對應但功能不符」的情況（例如將「系統任務清單」誤對應為「個人待辦事項」）。

## 執行步驟

### 1. 深度分析舊檔案邏輯

**目標**：找出舊檔案中的「關鍵邏輯特徵」。

1.  **開啟舊檔案**（如 `modules_student.php` 及其引用的 `mission_action_general.php`）。
2.  **識別輸入參數**：
    *   `GET`/`POST` 參數（如 `missionType`, `dowh_val`, `sSelfPractice`）。
    *   Session 變數（如 `user_id`, `grade`, `class_name`）。
3.  **識別核心查詢**：
    *   主要的 SQL 查詢結構（`WHERE` 條件）。
    *   特殊的 `SWITCH/CASE` 邏輯（如「過期」、「進行中」、「家長指派」的不同查詢）。
4.  **識別特殊權限邏輯**（以下為常見範例，請依實際檔案內容判斷）：
    *   是否有「組長 (Group Leader)」視角？
    *   是否有「家長 (Parent)」視角？

### 2. 檢查 Controller 實作

**目標**：確認 Controller 與 Service 是否能處理上述特徵。

1.  **開啟對照表指定的 Controller**（如 `MissionController.php`）。
2.  **檢查 `validate` 參數**：
    *   是否包含舊檔案的關鍵參數（如 `status` 對應 `dowh_val`, `self_practice` 對應 `sSelfPractice`）？
3.  **檢查 Service 方法**：
    *   追蹤 Service 內的 SQL 或查詢邏輯。
    *   確認特殊的過濾條件（如 `months_ago` 限制）是否已實作。

### 3. 執行 GAP 分析

將兩者進行比對，填寫差異表：

| 特徵 | 舊檔案邏輯 | Controller 實作 | 符合 |
|------|-----------|----------------|------|
| **篩選** | `dowh_val` (2=過期, 3=完成) | `status` (expired, completed) | ✅ |
| **參數** | `sSelfPractice` (0/1/2) | `self_practice` (0,1,2) | ✅ |
| **權限** | 支援小組長查看組員進度 | **未知 / 未發現** | ❓ |
| **範圍** | 預設顯示 9 個月 | `months_ago` 參數 (預設 9) | ✅ |

### 4. 修正對照表與提出修改建議

如果發現不符（GAP）：

1.  **修正對照表**：將 Controller 改為正確的對象（如 `TodoController` -> `MissionController`）。
2.  **標記遺漏功能**：在對照表中新增「待實作」項目，描述缺少的邏輯（如「缺少小組長進度查詢功能」）。
3.  **建立修改任務**：如果 Controller 缺少邏輯，應建立新的 Task 來補全。

## 範例：modules_student.php 驗證

**舊邏輯發現**：
- 主要顯示「老師指派的任務」。
- 參數 `dowh_val` 控制顯示過期/進行中/已完成。
- 參數 `sSelfPractice` 控制顯示家長/自己/老師指派。

**Controller 比對**：
- **TodoController**：僅處理 `events` 與 `start/end time`，偏向個人行事曆。 -> **❌ 不符合**
- **MissionController**：參數包含 `status`, `self_practice`, `mission_type`。 -> **✅ 符合**

**行動**：
- 更新 `api-mapping-modules_student.md`，將 `list/detail` 重導向至 `MissionController`。
