# IndicateTest API Mapping (縱貫診斷測驗)

## 模組概述
- **mission_type**: 3 (原始縱貫)、5 (題庫縱貫)
- **原始模組**: `modules/indicateTest/`
- **核心類別**: `indicateAdaptiveTestStructure` (位於 `classes/`)
- **功能說明**: 縱貫診斷測驗，適性化測驗系統，基於節點關係進行題目選擇

## 資料表結構

### 主要資料表
| 資料表 | 用途 |
|--------|------|
| `mission_info` | 任務資訊 (node, subject_id, mission_type, exam_type, endYear) |
| `indicate_test_predata` | 預計算節點資料 (mission_sn, predata, date) |
| `concept_item` | 原始題目 (mission_type=3) |
| `concept_itemBank` | 題庫題目 (mission_type=5) |
| `exam_record_indicate_tmp` | 暫存作答紀錄 |
| `exam_record_indicate` | 正式作答紀錄 |
| `map_info` | 知識地圖資訊 |
| `item_mission_rate` | 題目答對率統計 |

---

## API 端點設計

### 1. 開始測驗
```
POST /api/v3/indicate-test/start
```

#### 輸入參數
| 參數 | 類型 | 必填 | 說明 |
|------|------|------|------|
| mission_sn | int | Y | 任務編號 |

#### 輸出結構
```json
{
    "success": true,
    "data": {
        "exam_session_id": "string (加密的 session 識別碼)",
        "is_resume": false,
        "mission_name": "任務名稱",
        "mission_type": 5,
        "total_items": 50,
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
        },
        "adaptive_note": "實際作答題數會依據作答情形適性調整"
    }
}
```

#### 原始程式碼對應
- 檔案：`indicateAdaptiveTest.php` Line 15-110
- 類別：`indicateAdaptiveTestStructure::__construct()`
- 資料庫：
  - 查詢 `mission_info` 取得任務資訊
  - 查詢 `indicate_test_predata` 取得預計算資料 (或即時計算)
  - 查詢 `exam_record_indicate_tmp` 檢查是否有中斷紀錄

---

### 2. 提交答案（下一題）
```
POST /api/v3/indicate-test/answer
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
        "total_items": 50,
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
- 檔案：`indicateAdaptiveTestStructure.php` Line 266-324
- 類別方法：`checkAns($user_answer, $client_item_idle_time)`
- 適性邏輯：
  - `getNextItem()`: 取得下一題
  - `modifyNodeStatus()`: 修改節點狀態
  - `modifyLowNodeStatus()`: 預測下位節點狀態

---

### 3. 中斷測驗
```
POST /api/v3/indicate-test/pause
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
- 檔案：`indicateAdaptiveTest.php` Line 50-74
- 類別方法：`examPause()` (自動暫存於每題作答後)
- 資料庫：INSERT/UPDATE `exam_record_indicate_tmp`

---

### 4. 完成測驗
```
POST /api/v3/indicate-test/finish
```
*注意：當 answer API 返回 is_exam_finished = true 時，自動處理*

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
        "score": 75,
        "during_time": 600,
        "reward_coins": 8,
        "extra_coins": 0,
        "first_complete_bonus": 1,
        "is_first_100_percent": false,
        "bnode_results": [
            {
                "bnode": "3-n-01",
                "bnode_name": "大節點名稱",
                "status": 1,
                "correct_snodes": 5,
                "total_snodes": 6
            }
        ]
    }
}
```

#### 原始程式碼對應
- 類別方法：`checkExamEnd()`
- 資料庫：
  - INSERT `exam_record_indicate`
  - DELETE `exam_record_indicate_tmp`
  - 代幣計算：呼叫 `giveCoin` 類別

---

### 5. 取得測驗結果
```
GET /api/v3/indicate-test/result
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
        "exam_time_seconds": 600,
        "mission_name": "任務名稱",
        "subject_id": 1,
        "map_sn": 1,
        "score": 75,
        "bnode_results": [
            {
                "bnode": "3-n-01",
                "bnode_name": "大節點名稱",
                "grade": 3,
                "status": 1,
                "snode_results": [
                    {
                        "snode": "3-n-01-S01",
                        "snode_name": "小節點名稱",
                        "status": 1,
                        "video_progress": 100,
                        "practice_percent": 80,
                        "da_percent": 70
                    }
                ]
            }
        ],
        "reward_info": {
            "coins_earned": 8,
            "extra_coins": 0,
            "is_first_test": false
        }
    }
}
```

---

## 適性測驗邏輯

### 節點狀態碼
```
-1: 還沒做
0:  答錯
1:  答對
2:  預測對 (適性略過)
3:  預測對但做錯
4:  預測對也做對
5:  未測驗節點
```

### 適性演算法 (step=1)
1. 按照節點順序 (bNodes_order → sNodes_order) 出題
2. 每個小節點出 `sNode_items=2` 題
3. 若小節點全對，預測下位節點也會對 (狀態=2)
4. 若預測對的節點實際測驗答錯，狀態改為 3

### 全測模式 (step=2, exam_type=0)
- 適性階段完成後，對所有未測驗節點 (狀態=-1 或 5) 進行測驗

---

## Controller 設計

### IndicateTestController.php

```php
<?php

namespace App\Controller;

use App\Service\IndicateTest\IndicateTestService;

class IndicateTestController extends BaseController
{
    private IndicateTestService $indicateTestService;

    public function __construct(IndicateTestService $service)
    {
        $this->indicateTestService = $service;
    }

    // POST /api/v3/indicate-test/start
    public function start(): array
    {
        $missionSn = $this->getIntParam('mission_sn');

        return $this->indicateTestService->startExam($missionSn);
    }

    // POST /api/v3/indicate-test/answer
    public function answer(): array
    {
        $sessionId = $this->getStringParam('exam_session_id');
        $userAnswer = $this->getParam('user_answer');
        $clientStartMs = $this->getIntParam('client_start_ms', 0);

        return $this->indicateTestService->submitAnswer($sessionId, $userAnswer, $clientStartMs);
    }

    // POST /api/v3/indicate-test/pause
    public function pause(): array
    {
        $sessionId = $this->getStringParam('exam_session_id');

        return $this->indicateTestService->pauseExam($sessionId);
    }

    // GET /api/v3/indicate-test/result
    public function result(): array
    {
        $examSn = $this->getIntParam('exam_sn');
        $missionSn = $this->getIntParam('mission_sn');

        return $this->indicateTestService->getResult($examSn, $missionSn);
    }
}
```

---

## Service 層設計

### IndicateTestService.php

主要方法：
- `startExam(int $missionSn)`: 開始或恢復測驗
- `submitAnswer(string $sessionId, $userAnswer, int $clientStartMs)`: 提交答案並取得下一題
- `pauseExam(string $sessionId)`: 暫停測驗
- `getResult(int $examSn, int $missionSn)`: 取得測驗結果

內部方法：
- `getMissionData(int $missionSn)`: 取得任務資料
- `getPreData(int $missionSn)`: 取得或生成預計算資料
- `getNodeLinkData()`: 取得節點上下位關係
- `getNextItem()`: 適性選題邏輯
- `checkAns()`: 檢查答案
- `modifyNodeStatus()`: 修改節點狀態
- `modifyLowNodeStatus()`: 預測下位節點
- `checkExamEnd()`: 檢查測驗是否結束

---

## DAO 層設計

### IndicateTestDao.php

```php
// 任務相關
public function getMissionInfo(int $missionSn): ?array
public function getMapInfo(int $subjectId): ?array

// 預計算資料
public function getPreData(int $missionSn): ?array
public function savePreData(int $missionSn, string $predata): bool

// 題目相關 (根據 mission_type 選擇資料表)
public function getConceptItem(int $itemSn): ?array      // mission_type=3
public function getConceptItemBank(int $itemSn): ?array  // mission_type=5
public function getSNodeItems(string $sNode, int $missionType, int $mapSn): array

// 暫存記錄
public function getTmpRecord(string $userId, string $csId, int $missionSn): ?array
public function saveTmpRecord(array $data): bool
public function deleteTmpRecord(string $userId, string $csId, int $missionSn): bool

// 正式記錄
public function insertExamRecord(array $data): int
public function getExamRecord(int $examSn, int $missionSn, string $userId): ?array

// 統計
public function updateItemMissionRate(int $itemSn, int $missionSn, int $isCorrect): bool
```

---

## 狀態管理

### Session 替代方案
原始程式使用 `$_SESSION['indicateTest']` 儲存測驗狀態。API 版本改用：

1. **資料庫暫存**: 使用 `exam_record_indicate_tmp` 儲存序列化狀態
2. **加密 Token**: 返回給前端的 `exam_session_id`

### 主要狀態欄位
```php
[
    'mission_sn' => int,
    'mission_data' => [
        'upmost_nodes' => array,
        'subject_id' => int,
        'exam_type' => int,        // 0:適性全測, 1:適性選題
        'endYear' => array,
        'map_sn' => int,
        'mission_type' => int      // 3:原始縱貫, 5:題庫縱貫
    ],
    'node_link_data' => [
        'bNodes_link' => array,
        'sNodes_link' => array,
        'bNodes_order' => array,
        'sNodes_order' => array
    ],
    'record_bNodes_status' => array,
    'record_sNodes_status' => array,
    'sNode_now' => string,
    'step' => int,                 // 1:適性, 2:全測
    'sNode_items_left' => array,
    'sNode_response' => array,
    'item_num' => int,
    'record_answer' => array,
    'record_response' => array,
    'record_client_time' => array,
    'record_item_sn' => array,
    'start_time' => int,
    'start_date' => string,
    'total_items' => int
]
```

---

## mission_type 差異

| 項目 | mission_type=3 | mission_type=5 |
|------|----------------|----------------|
| 題目來源 | concept_item | concept_itemBank |
| 題目路徑 | ./data/CS_db/{cs_id}/ | ./data/itemBank/{map_sn}/ |
| cs_id 格式 | 026{subject_id}1899 | 無 |
| 暫存表 key | cs_id + result_sn | cs_id + result_sn |

---

## 注意事項

1. **預計算資料**: 首次測驗會計算並存入 `indicate_test_predata`，後續直接讀取
2. **多選題處理**: `template_id = 5` 為多選題
3. **適性略過**: 預測對的節點直接略過，減少題數
4. **節點順序**: 按照大節點→小節點的順序出題
5. **總題數**: 因適性調整，實際題數可能少於預計

---

## 相關檔案參考

| 檔案 | 說明 |
|------|------|
| `modules/indicateTest/indicateAdaptiveTest.php` | 主程式入口 |
| `classes/indicateAdaptiveTestStructure.php` | 核心測驗類別 (900+ 行) |
| `modules/indicateTest/indicateResult.php` | 結果顯示 |
| `include/adp_core_class.php` | 代幣計算類別 |
| `prodb_mission_record_update.php` | 任務紀錄更新 |