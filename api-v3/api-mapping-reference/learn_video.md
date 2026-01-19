# learn_video 功能對照表

個人學習紀錄頁面 - 學習紀錄視覺化分析 (影片、練習題、動態評量、AI學習夥伴)。

---

## 原始程式檔案

| 檔案 | 路徑 | 說明 |
|------|------|------|
| video_personal_file.php | `modules/learn_video/video_personal_file.php` | 主頁面 (前端 + 資料處理) |
| report_learningeffect_class.php | `classes/report_learningeffect_class.php` | 學習報表 Class (SQL 查詢核心) |

---

## 功能拆解與 API 對應

### 1. 個人學習紀錄模組

#### 舊程式功能

| 功能 | 舊程式位置 | 實作方式 |
|------|------------|----------|
| 取得科目選項 | `video_personal_file.php:12-17` | `getSubjectSelector()` |
| 家長查詢學生 | `video_personal_file.php:174-195` | SQL 查詢 `user_family` |
| 取得年級節點數量 | `report_learningeffect_class.php:1642-1727` | `createPersonalData()` |
| 取得學習紀錄原始資料 | `report_learningeffect_class.php:1938-2007` | `handlePersonalData()` |
| 計算影片統計 | `video_personal_file.php:569-613` | 前端迴圈計算 |
| 計算練習題統計 | `video_personal_file.php:615-652` | 前端迴圈計算 |
| 計算動態評量統計 | `video_personal_file.php:654-691` | 前端迴圈計算 |
| 計算 AI 學習夥伴統計 | `video_personal_file.php:708-717` | 前端迴圈計算 |
| 周/月檢視切換 | `video_personal_file.php:58-68` | `funChangRange()` JS |
| 歷史紀錄計算 (共7期間) | `video_personal_file.php:901-1275` | for 迴圈 7 次 |

#### v3 API 對應

| v3 API | Controller 方法 | 說明 |
|--------|-----------------|------|
| `POST /learning-record/personal` | `LearningRecordController::personal()` | ✅ 整合所有統計邏輯 |

#### 參數對照

| 舊參數 | 新參數 | 型別 | 說明 |
|--------|--------|------|------|
| `$_REQUEST['subject']` | `subject` | integer | 必填，科目 ID |
| `$_REQUEST['grade']` | `grade` | integer | 選填，起始年級 (1-12) |
| `$_REQUEST['end']` | `end_grade` | integer | 選填，終止年級 (1-12) |
| `$_REQUEST['time']` (w/m) | `time_view` | string | 選填，week/month (預設 week) |
| `$_REQUEST['uname']` | - | - | ❌ 家長功能，尚未實作 |

#### 設計異動

> [!NOTE]
> v3 API 將原本分散在前端 PHP 的統計邏輯全部整合到 `LearningRecordService`，一次回傳 8 個期間 (現在 + 7 筆歷史) 的完整資料。

> [!IMPORTANT]
> 原始程式依賴 `$_SESSION['user_data']`，v3 Service 暫時透過手動設置 `$_SESSION['user_data']` 來相容舊的 `ReportLearningeffectClass`。

#### SQL 比較

| 項目 | 舊程式 | v3 API | 結論 |
|------|--------|--------|------|
| **資料來源** | `ReportLearningeffectClass` | 直接 `require_once` 引用同一個 Class | ✅ SQL 完全相同 |
| **年級節點查詢** | `searchGradeNode()` | 呼叫 `createPersonalData()` | ✅ 相同 |
| **影片紀錄查詢** | `searchVideoRecord()` | 呼叫 `createPersonalData()` | ✅ 相同 |
| **練習題紀錄查詢** | `searchPracticeRecord()` | 呼叫 `createPersonalData()` | ✅ 相同 |
| **動態評量查詢** | `searchDynamicRecord()` | 呼叫 `createPersonalData()` | ✅ 相同 |
| **AI學習夥伴查詢** | `searchRobotRecord()` | 呼叫 `createPersonalData()` | ✅ 相同 |
| **統計計算** | 前端 PHP 迴圈 (`video_personal_file.php:569-717`) | `LearningRecordService` 後端計算 | ✅ 邏輯遷移至 Service |

> [!TIP]
> v3 架構：`LearningRecordService` → (require_once) → `ReportLearningeffectClass`
> SQL 查詢由舊 Class 負責，v3 Service 僅負責統計計算和格式化回應。

---

## 回應結構

```json
{
  "periods": [
    {
      "type": "current",
      "label": "01-13 ~ 01-19",
      "year": 2026,
      "totalProgress": 25,
      "learnedProgress": 20,
      "newProgress": 5,
      "video": {
        "count": 3,
        "time": "00:45:30",
        "details": [...]
      },
      "practice": { ... },
      "dynamic": { ... },
      "robot": { ... }
    },
    { "type": "history", ... }
  ]
}
```

---

## 未對應功能

| 舊功能 | 說明 | 狀態 |
|--------|------|------|
| 家長查詢學生 | `$_REQUEST['uname']` 帶入指定學生 ID | ❌ 未實作 |
| 國語文朗讀/寫作跳轉 | 特殊科目 (-99, -98) 直接開新視窗 | ❌ 前端處理 |
| 科目選單 API | 取得科目下拉選項 | ❌ 未實作 (前端固定) |

---

## 驗證檢查清單

- [x] 學生個人學習紀錄 API 對應
- [x] 週/月檢視模式切換
- [x] 8 個期間 (現在 + 7 筆歷史) 統計
- [x] 影片/練習題/動態評量/AI學習夥伴 統計
- [x] 細節展開資料
- [ ] 家長查詢學生功能
- [ ] 科目選單 API
