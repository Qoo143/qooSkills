# IndicatorTest API Mapping (單元診斷測驗)

## 模組概述
- **mission_type**: 4
- **原始模組**: `modules/indicatorTest/`
- **核心類別**: `indicatorTestStructure`
- **功能說明**: 單元診斷測驗，基於 paper_sn 組卷形式的測驗

## 資料表結構

### 主要資料表
| 資料表 | 用途 |
|--------|------|
| `concept_paper` | 試卷資訊 (paper_sn, item_sn, subject, name) |
| `concept_itemBank` | 題目資訊 (item_sn, op_ans, indicator, map_sn, filename, op_filename, template_id) |
| `exam_record_indicator_tmp` | 暫存作答紀錄 |
| `exam_record_indicator` | 正式作答紀錄 |
| `mission_info` | 任務資訊 |
| `item_mission_rate` | 題目答對率統計 |
| `map_node_student_status` | 學生節點狀態 |

---

## API 端點設計

### 1. 開始測驗
```
POST /api/v3/indicator-test/start
```

#### 輸入參數
| 參數 | 類型 | 必填 | 說明 |
|------|------|------|------|
| mission_sn | int | Y | 任務編號 |
| paper_sn | int | Y | 試卷編號 |

#### 輸出結構
```json
{
    "success": true,
    "data": {
        "exam_session_id": "string (加密的 session 識別碼)",
        "is_resume": false,
        "paper_name": "試卷名稱",
        "total_items": 10,
        "current_item": 1,
        "question": {
            "item_sn": 12345,
            "question_html": "題目 HTML 內容",
            "options": [
                {"value": 1, "html": "選項1 HTML"},
                {"value": 2, "html": "選項2 HTML"}
            ],
            "template_id": 1,
            "is_multiple_choice": false
        }
    }
}
```

#### 原始程式碼對應
- 檔案：`indicatorTest.php` Line 31-272
- 類別方法：`indicatorTestStructure::__construct()`
- 資料庫：
  - 查詢 `concept_paper` 取得試卷資訊
  - 查詢 `concept_itemBank` 取得題目資訊
  - 查詢 `exam_record_indicator_tmp` 檢查是否有中斷紀錄

---

### 2. 提交答案（下一題）
```
POST /api/v3/indicator-test/answer
```

#### 輸入參數
| 參數 | 類型 | 必填 | 說明 |
|------|------|------|------|
| exam_session_id | string | Y | 考試 session ID |
| user_answer | string/array | Y | 使用者答案 (單選:字串, 多選:陣列) |
| client_start_ms | int | N | 開始作答時間戳 |

#### 輸出結構
```json
{
    "success": true,
    "data": {
        "is_exam_finished": false,
        "current_item": 2,
        "total_items": 10,
        "question": {
            "item_sn": 12346,
            "question_html": "下一題 HTML 內容",
            "options": [...],
            "template_id": 1,
            "is_multiple_choice": false
        }
    }
}
```

#### 原始程式碼對應
- 檔案：`indicatorTestStructure.php` Line 228-253
- 類別方法：`processOfItemEnd($user_answer)`
- 資料庫：
  - 更新 `item_mission_rate` 答對率統計

---

### 3. 完成測驗
```
POST /api/v3/indicator-test/finish
```
*注意：當 answer API 返回 is_exam_finished = true 時，自動觸發*

#### 輸入參數
| 參數 | 類型 | 必填 | 說明 |
|------|------|------|------|
| exam_session_id | string | Y | 考試 session ID |

#### 輸出結構
```json
{
    "success": true,
    "data": {
        "exam_sn": 12345,
        "score": 80,
        "snode_rate": 75,
        "during_time": 300,
        "reward_coins": 5,
        "extra_coins": 0,
        "first_complete_bonus": 1,
        "is_first_100_percent": false,
        "result_summary": {
            "total_nodes": 10,
            "correct_nodes": 8,
            "history_best_rate": 70
        }
    }
}
```

#### 原始程式碼對應
- 檔案：`indicatorTestStructure.php` Line 289-322
- 類別方法：`processOfExamEnd()`
- 資料庫：
  - INSERT `exam_record_indicator`
  - DELETE `exam_record_indicator_tmp`
  - UPDATE `map_node_student_status`
  - 代幣計算：呼叫 `giveCoin` 類別

---

### 4. 中斷測驗
```
POST /api/v3/indicator-test/pause
```

#### 輸入參數
| 參數 | 類型 | 必填 | 說明 |
|------|------|------|------|
| exam_session_id | string | Y | 考試 session ID |

#### 輸出結構
```json
{
    "success": true,
    "data": {
        "message": "測驗已暫停，可稍後繼續"
    }
}
```

#### 原始程式碼對應
- 檔案：`indicatorTestStructure.php` Line 256-264, 510-554
- 類別方法：`stopExam()`, `stopExamDataToDB()`
- 資料庫：
  - INSERT/UPDATE `exam_record_indicator_tmp` (session_data 欄位儲存序列化狀態)

---

### 5. 取得測驗結果
```
GET /api/v3/indicator-test/result
```

#### 輸入參數
| 參數 | 類型 | 必填 | 說明 |
|------|------|------|------|
| exam_sn | int | Y | 考試記錄編號 |
| mission_sn | int | Y | 任務編號 |

#### 輸出結構
```json
{
    "success": true,
    "data": {
        "exam_date": "2024-01-15 10:30:00",
        "exam_time_seconds": 300,
        "mission_name": "任務名稱",
        "score": 80,
        "snode_rate": 75,
        "node_results": [
            {
                "node": "3-n-01-S01",
                "node_name": "節點名稱",
                "grade": 3,
                "status": 1,
                "video_progress": 100,
                "video_count": 2,
                "practice_percent": 80,
                "practice_count": 5,
                "da_percent": 70,
                "da_count": 3,
                "has_teach": true,
                "has_note": true
            }
        ],
        "reward_info": {
            "coins_earned": 5,
            "extra_coins": 0,
            "is_first_test": false,
            "is_first_100_percent": false
        }
    }
}
```

#### 原始程式碼對應
- 檔案：`indicatorTestResult.php` Line 16-464
- 函數：`indicatorTestResult()`
- 資料庫：
  - 查詢 `exam_record_indicator`
  - 查詢 `video_review_record` (影片觀看紀錄)
  - 查詢 `prac_answer` (練習題紀錄)
  - 查詢 `exam_record_dynamic` (動態評量紀錄)

---

## Controller 設計

### IndicatorTestController.php

```php
<?php

namespace App\Controller;

use App\Service\IndicatorTest\IndicatorTestService;

class IndicatorTestController extends BaseController
{
    private IndicatorTestService $indicatorTestService;

    public function __construct(IndicatorTestService $service)
    {
        $this->indicatorTestService = $service;
    }

    // POST /api/v3/indicator-test/start
    public function start(): array
    {
        $missionSn = $this->getIntParam('mission_sn');
        $paperSn = $this->getIntParam('paper_sn');

        return $this->indicatorTestService->startExam($missionSn, $paperSn);
    }

    // POST /api/v3/indicator-test/answer
    public function answer(): array
    {
        $sessionId = $this->getStringParam('exam_session_id');
        $userAnswer = $this->getParam('user_answer');
        $clientStartMs = $this->getIntParam('client_start_ms', 0);

        return $this->indicatorTestService->submitAnswer($sessionId, $userAnswer, $clientStartMs);
    }

    // POST /api/v3/indicator-test/pause
    public function pause(): array
    {
        $sessionId = $this->getStringParam('exam_session_id');

        return $this->indicatorTestService->pauseExam($sessionId);
    }

    // GET /api/v3/indicator-test/result
    public function result(): array
    {
        $examSn = $this->getIntParam('exam_sn');
        $missionSn = $this->getIntParam('mission_sn');

        return $this->indicatorTestService->getResult($examSn, $missionSn);
    }
}
```

---

## Service 層設計

### IndicatorTestService.php

主要方法：
- `startExam(int $missionSn, int $paperSn)`: 開始或恢復測驗
- `submitAnswer(string $sessionId, $userAnswer, int $clientStartMs)`: 提交答案
- `pauseExam(string $sessionId)`: 暫停測驗
- `getResult(int $examSn, int $missionSn)`: 取得測驗結果

內部方法：
- `checkResumeExam()`: 檢查是否有中斷紀錄
- `buildQuestionHtml()`: 建構題目 HTML
- `calculateScore()`: 計算分數
- `updateNodeStatus()`: 更新節點狀態
- `calculateReward()`: 計算代幣獎勵

---

## DAO 層設計

### IndicatorTestDao.php

```php
// 試卷相關
public function getPaperInfo(int $paperSn): ?array
public function getItemsByPaperSn(int $paperSn): array

// 暫存記錄相關
public function getTmpRecord(string $userId, int $paperSn, int $missionSn): ?array
public function saveTmpRecord(array $data): bool
public function deleteTmpRecord(string $userId, int $paperSn, int $missionSn): bool

// 正式記錄相關
public function insertExamRecord(array $data): int
public function getExamRecord(int $examSn, int $missionSn, string $userId): ?array

// 統計相關
public function updateItemMissionRate(int $itemSn, int $missionSn, int $isCorrect): bool
public function updateNodeStatus(string $userId, int $mapSn, array $sNodeStatus, array $bNodeStatus): bool
```

---

## 狀態管理

### Session 替代方案
原始程式使用 `$_SESSION['indicatorTest']` 儲存測驗狀態。API 版本改用：

1. **資料庫暫存**: 使用 `exam_record_indicator_tmp.session_data` 欄位
2. **加密 Token**: 返回給前端的 `exam_session_id` 包含 user_id + paper_sn + mission_sn

### 狀態欄位
```php
[
    'paper_sn' => int,
    'mission_sn' => int,
    'items_sn_order' => array,     // 剩餘題目順序
    'item_num' => int,             // 目前題號
    'rec_questions' => string,     // 已作答題目
    'rec_org_res' => string,       // 原始作答
    'rec_binary_res' => string,    // 二元作答
    'indicator_info' => array,     // 節點資訊
    'time' => [
        'start_time' => int,
        'date' => string,
        'client_items_idle_time' => string
    ]
]
```

---

## 注意事項

1. **多選題處理**: `template_id = 5` 為多選題，答案需用 `@XX3@` 分隔
2. **隨機出題**: `exam_type = 3` 時題目順序需打亂
3. **代幣計算**: 依據節點答對率或節點數計算，需參考 `adp_core_class.php` 的 `giveCoin` 類別
4. **節點狀態更新**: 完成測驗後需更新 `map_node_student_status` 表

---

## 相關檔案參考

| 檔案 | 說明 |
|------|------|
| `modules/indicatorTest/indicatorTest.php` | 主程式入口 |
| `modules/indicatorTest/indicatorTestStructure.php` | 核心測驗類別 |
| `modules/indicatorTest/indicatorTestResult.php` | 結果顯示 |
| `include/adp_core_class.php` | 代幣計算類別 |
| `prodb_mission_record_update.php` | 任務紀錄更新 |
