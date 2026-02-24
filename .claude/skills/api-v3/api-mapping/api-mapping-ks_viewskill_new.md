# API-V3 對照表 - ks_viewskill_new.php

**舊檔案路徑**：`modules/assignMission/ks_viewskill_new.php`  
**舊檔案類型**：頁面載入型（Page Load）+ 前端渲染  
**分析日期**：2026-02-05  
**主要功能**：課程學習節點詳情頁面  
**分析者**：Claude Sonnet 4.5 Agent

---

## 檔案概述

### 核心功能
此檔案為**課程學習節點（知識點）的詳情展示頁面**，整合多種學習資源：

1. **頁面資訊載入**（Page Load Logic）
   - 顯示課程基本資訊（科目、年級、出版商、單元）
   - 查詢該節點的向上/向下學習路徑
   - 檢查該節點擁有的學習資源類型

2. **學習資源類型**
   - 教學媒體（影片）
   - 教學媒體列表
   - 練習題（選擇題）
   - 填充題
   - 學習單（檔案/連結）
   - 補充教材（檔案/連結）
   - 物理模擬題（互動式）
   - 動態評量教學（DA）
   - 互動教學
   - 小試身手（eGame）
   - 學科領域學習夥伴（AI Robot）

3. **前端互動**
   - Tab 切換學習資源
   - 動態載入子模組（video_new.php, practice_new.php 等）
   - 影片播放控制、筆記、提問功能

### 技術特點
- **混合架構**：PHP 頁面載入 + JavaScript 前端渲染
- **模組化設計**：根據資源類型動態引入子模組
- **權限控制**：根據 access_level 顯示不同內容
- **無 AJAX Router**：直接在頁面載入時完成所有查詢

---

## 端點對照清單

| 選擇 | 舊端點/功能 | Controller 方法 | 狀態 | Controller | 說明 |
|:----:|--------|----------------|:----:|-----------|------|
| [x] | Page Load: 課程節點詳情 | `node()` | ✅ 已完成 | **KnowledgeController** | 課程節點的完整資訊與資源清單 |
| [x] | Function: 取得上下位節點 | `relatedNodes()` | ✅ 已完成 | KnowledgeController | 向上學習、向下補救的節點清單 |
| [x] | SubModule: 影片播放 | `video()` | ✅ 已完成 | **KsLearningController** | video() + videoRecord() |
| [x] | SubModule: 練習題 | `practice()` | ✅ 已完成 | KsLearningController | practice() + practiceSubmit() |
| [x] | SubModule: 動態評量 | `assessment()` | ✅ 已完成 | KsLearningController | assessment() + assessmentSubmit() |
| [x] | Function: 取得補充教材 | - | ⚠️ 整合 | KnowledgeController | 整合於 node() 回傳的 files 欄位 |
| [x] | Function: 取得外部連結 | - | ⚠️ 整合 | KnowledgeController | 整合於 node() 回傳的 external_links 欄位 |
| [x] | Function: 取得 eGame 連結 | - | ⚠️ 整合 | KnowledgeController | 整合於 node() 回傳的 resources.e_game_url |

### API 端點路徑

| HTTP Method | 端點路徑 | Controller 方法 | 說明 |
|-------------|----------|----------------|------|
| POST | `/api/v3/knowledge/node` | KnowledgeController::node() | 節點詳情 |
| POST | `/api/v3/knowledge/related-nodes` | KnowledgeController::relatedNodes() | 上下位節點 |
| POST | `/api/v3/ks-learning/video` | KsLearningController::video() | 開始觀看影片 |
| POST | `/api/v3/ks-learning/video/record` | KsLearningController::videoRecord() | 更新影片進度 |
| POST | `/api/v3/ks-learning/practice` | KsLearningController::practice() | 取得練習題 |
| POST | `/api/v3/ks-learning/practice-submit` | KsLearningController::practiceSubmit() | 提交練習題 |
| POST | `/api/v3/ks-learning/assessment` | KsLearningController::assessment() | 取得動態評量 |
| POST | `/api/v3/ks-learning/assessment-submit` | KsLearningController::assessmentSubmit() | 提交動態評量 |

---

## 功能詳細分析

### 1. Page Load: 課程節點詳情

**舊代碼位置**：Line 10-73（主要邏輯）

**業務邏輯**：
當用戶訪問課程節點（知識點）時，載入該節點的完整資訊，包括：
- 節點基本資訊（科目、年級、出版商、單元名稱）
- 向上/向下學習路徑
- 該節點擁有的學習資源類型（影片、練習題等）
- 各類資源的詳細清單

**參數分析**：
```php
// ⭐ 支援兩種模式，擇一使用

// Mode 1: 一般課程（使用 mapping_sn）
$mapping_sn = $_REQUEST['mid'];             // publisher_mapping.mapping_sn

// Mode 2: 物理模擬題（使用 interactiveid + subject_id）
$interactiveid = $_REQUEST['interactiveid']; // concept_itemBank_literacy_interactive.item_li_sn
$subject_id = $_REQUEST['subject_id'];       // 科目 ID（Mode 2 必填）

// 通用參數（選填）
$mission_sn = $_REQUEST['mission_sn'];       // 任務序號，存入 Session 供子模組使用
```

**參數邏輯說明**：
- **Mode 1**: 透過 `mapping_sn` 從 `publisher_mapping` 查詢 indicator、subject_id
- **Mode 2**: 透過 `interactiveid` 從 `concept_itemBank_literacy_interactive` 查詢 indicator，搭配 `subject_id` 取得課程資訊

**對應 Validator 規則**：
```php
[
    // Mode 1 參數
    'mapping_sn' => 'integer|exists:publisher_mapping,mapping_sn',

    // Mode 2 參數
    'interactiveid' => 'integer|exists:concept_itemBank_literacy_interactive,item_li_sn',
    'subject_id' => 'integer|exists:subject,subject_id',

    // 通用參數
    'mission_sn' => 'integer|exists:mission_info,mission_sn',
]

// ⭐ 自訂驗證：至少提供一種模式
// if (empty($mapping_sn) && empty($interactiveid)) {
//     throw new AppException('必須提供 mapping_sn 或 interactiveid', 400);
// }
// if (!empty($interactiveid) && empty($subject_id)) {
//     throw new AppException('使用 interactiveid 時必須提供 subject_id', 400);
// }
```

**SQL 查詢總覽**：

1. **取得節點基本資訊**（Line 81-103）
2. **取得上下位節點**（Line 137-144）
3. **檢查資源類型 - map_node**（Line 188-202）
4. **檢查資源類型 - 影片清單數量**（Line 206-215）
5. **檢查資源類型 - 物理模擬題**（Line 222-228）
6. **檢查資源類型 - 補充教材檔案**（Line 245-255）
7. **檢查資源類型 - 外部連結**（Line 283-292）
8. **檢查資源類型 - eGame**（Line 316-325）
9. **檢查資源類型 - AI 學習夥伴**（Line 336-360）
10. **取得影片詳細資訊**（Line 376-398）
11. **取得補充教材檔案**（Line 411-422）
12. **取得外部連結**（Line 437-448）
13. **取得 eGame 連結**（Line 459-468）

**權限要求**：
- **基本權限**：已登入用戶（學生、教師、管理員）
- **學習單檔案限制**：
  - 學生：只能查看 PDF 格式
  - 教師/助理：可查看 PDF, ODT, DOC, DOCX
- **補充教材檔案限制**：
  - 學生：只能下載 PDF（file_type = 3 的特殊規則）
  - 教師/助理：可下載所有格式

**回傳格式**：

```json
{
    "node_info": {
        "indicator": "1-n-01",
        "indicator_name": "整數的加減運算",
        "subject_id": 5,
        "subject_name": "數學",
        "grade": 3,
        "sems": "1091",
        "publisher_id": 1,
        "publisher": "南一",
        "unit": "1",
        "unit_name": "10000以內的數"
    },
    "related_nodes": {
        "high_challenge": [
            {
                "indicator": "1-n-02",
                "indicator_name": "整數的乘除運算"
            }
        ],
        "low_challenge": []
    },
    "resource_types": {
        "video": 1,
        "videolist": 3,
        "prac": 1,
        "DA": 0,
        "teach": 0,
        "blank_filling": 0,
        "interactive": 0,
        "fileres": 1,
        "external": 1,
        "worksheets_fileres": 1,
        "worksheets_link": 0,
        "eGame": 0,
        "robot": 1
    },
    "video_list": [
        {
            "video_item_sn": 12345,
            "video_nm": "10000以內的數-位值概念",
            "indicator": "1-n-01"
        }
    ],
    "file_resources": [
        {
            "file_type": 1,
            "type_name": "教學簡報",
            "filename": "10000以內的數-教學簡報",
            "fileurl": "teaching_slide_001.pdf"
        }
    ],
    "external_resources": [
        {
            "res_type": 2,
            "type_name": "教學動畫",
            "res_url": "https://example.com/animation",
            "res_caption": "位值概念動畫"
        }
    ]
}
```

**對應 API 端點**：`GET /api/v3/learning-resource/detail`

---

## 資料表關聯

### 主要資料表

1. **publisher_mapping** - 課程對應表
   - mapping_sn (PK), indicator, subject_id, grade, publisher_id, sems, unit, unit_name

2. **indicateTest** - 知識點關聯表
   - indicate, subject, indicate_high, indicate_low

3. **map_node** - 地圖節點（資源類型標記）
   - map_sn (FK), indicate_id, video, prac, DA, teach, blank_filling

4. **video_concept_item** - 教學影片
   - video_item_sn (PK), indicator, map_sn, video_nm, cs_id (FK), public, end_mk

5. **map_node_file** - 補充教材檔案
   - map_node_file_sn (PK), indicator, map_sn, file_type, filename, fileurl, public

6. **map_node_external** - 外部連結
   - res_sn (PK), indicate_id, subject_id, res_type, res_url, res_caption, public

7. **concept_item_interactive** - 物理模擬題
   - item_sn (PK), indicator, map_sn, public

8. **egame_info** - 小試身手遊戲
   - egame_id (PK), map_sn, indicate_id, link

9. **concept_itemBank_depiction** - AI 學習夥伴題庫
   - depiction_sn (PK), item_sn (FK)

10. **external_res_type** - 資源類型對照表
    - type_sn (PK), type_name

---

## 安全性檢查

- [x] SQL 使用 prepared statements
- [x] 輸出使用 preventXss()
- [x] 權限檢查：根據 access_level 限制檔案下載權限
- [ ] CSRF token 驗證（需在 API 中補上）
- [x] 參數驗證：必填欄位、類型檢查
- [ ] 檔案下載權限檢查（需確認用戶是否有權限存取該課程）

**安全性問題**：
1. **檔案下載驗證不足**：目前只檢查 access_level，未檢查用戶是否有權限存取該課程
2. **物理路徑暴露**：`./data/materials/{map_sn}/{fileurl}` 路徑直接暴露在前端
3. **外部連結未驗證**：外部連結未檢查是否為安全的 URL

---

## 業務邏輯特殊處理

### 1. 學習單權限規則（Line 264-270）
```php
// 學生權限：只有 PDF 才可以顯示
if (!in_array($sAccessLevel, USER_TEACHER_GROUP) && 
    $vFileCntValue['fileurl_type'] == 'pdf') {
    $sub_info['worksheets_fileres'] = 1;
}
// 教師 & 助理權限：無限制
else if (in_array($sAccessLevel, USER_TEACHER_GROUP) || 
         in_array($sAccessLevel, [USER_OPERATOR])) {
    $sub_info['worksheets_fileres'] = 1;
}
```

### 2. 物理模擬題影片特殊處理（Line 233-236）
```php
// 如果同時有物理模擬題和影片，強制開啟影片功能
if ($sub_info['interactive'] == 1 && $sub_info['videolist'] > 0) {
    $sub_info['video'] = 1;
}
```

### 3. 上下位節點解析（Line 154-180）
支援多個節點，使用 `_SPLIT_SYMBOL` 分隔，支援節點名稱格式：`indicator[name]`

### 4. AI 學習夥伴開放規則（Line 336-365）
- 排除社會科（subject_id = 209）
- 排除高中英語科（subject_id = 30, grade 10-12）
- 只開放特定 map_sn 清單（從 robotConfig 取得）

---

## 開發優先級

### 高優先級（核心功能）
- [ ] `detail()` - 課程節點詳情
- [ ] `getNodeInfo()` - 節點基本資訊
- [ ] `checkResourceTypes()` - 檢查資源類型

### 中優先級（常用功能）
- [ ] `getRelatedNodes()` - 上下位節點
- [ ] `getVideoList()` - 影片清單
- [ ] `getFileResources()` - 補充教材檔案

### 低優先級（輔助功能）
- [ ] `getEGameLink()` - 小試身手連結
- [ ] `checkRobot()` - AI 學習夥伴檢查

---

## 遷移注意事項

### 1. 架構差異
- **舊代碼**：頁面載入時完成所有查詢，資料傳給前端 JavaScript 渲染
- **新 API**：RESTful API，回傳 JSON 格式，前端 Vue 完全負責渲染

### 2. 參數邏輯複雜
支援 `mapping_sn` 或 `interactiveid + subject_id` 兩種參數組合，需保留這兩種邏輯分支

### 3. 權限控制細緻
學習單檔案根據 access_level 和檔案類型雙重限制，需在 Service 層實作完整權限邏輯

### 4. Session 依賴
舊代碼使用 $_SESSION 儲存狀態，新 API 應無狀態化，所有參數透過 API 傳遞

### 5. 子模組整合
原檔案動態 require 多個子模組，這些需各自建立 API 端點

---

## 相關檔案清單

### 子模組（需另外建立 API）
- `modules/learn_video/video_new.php` - 影片播放、筆記、提問
- `modules/Practice/practice_new.php` - 練習題（選擇題）
- `modules/blankFilling/blank_filling_new.php` - 填充題
- `modules/physics_interactive/interactive_practice.php` - 物理模擬題
- `modules/DynamicAdaptiveTest/DA_new.php` - 動態評量教學
- `modules/New_CR/component_list_new.php` - 互動教學

---

## 開發記錄

### 2026-02-05
- [x] 完成對照表建立
- [x] 分析 1 個頁面載入功能，13 個 SQL 查詢
- [x] 規劃 LearningResourceController（新建）
- [x] 識別 11 個子模組需另外開發
- [ ] 待轉換：1 個主功能 + 11 個子模組

---

## 下一步建議

1. **執行開發命令**：開始開發課程節點詳情 API
2. **確認資料表結構**：使用 MCP MySQL 查詢相關資料表 Schema
3. **子模組優先級排序**：決定先開發哪個子模組（建議：影片播放 > 練習題）
4. **前端路由規劃**：與前端團隊討論 Vue Router 設計
