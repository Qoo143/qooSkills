# API-V3 對照表：Knowledge (知識結構)

## 遷移進度總覽

| 端點 | 狀態 | 待處理項目 |
|------|------|-----------|
| POST /knowledge/node | ✅ 90% | Robot 白名單、eGame URL |
| POST /knowledge/unit | ✅ 100% | - |
| POST /knowledge/list | ✅ 100% | - |
| POST /knowledge/related-nodes | ✅ 100% | - |

---

## 舊檔案

| 檔案 | 用途 | 對應新端點 |
|------|------|-----------|
| `modules/assignMission/ks_viewskill_newScrept.php` | 節點詳情頁邏輯 (PHP+JS 混合) | node(), relatedNodes() |
| `modules/assignMission/ks_viewskill_new.php` | 節點詳情頁 HTML 模板 | node() |
| `modules/assignMission/ks_viewunit_new.php` | 單元節點列表頁 | unit() |
| `modules/assignMission/assign_mission.php` | 出版商/單元列表查詢 | list() |

## 現有 V3 檔案

| 層級 | 檔案 |
|------|------|
| Controller | `ADLAPI/v3/App/Controller/KnowledgeController.php` |
| Service | `ADLAPI/v3/App/Service/KnowledgeService.php` |
| DAO/NodeDao | `ADLAPI/v3/App/Dao/Knowledge/NodeDao.php` |
| DAO/ResourceDao | `ADLAPI/v3/App/Dao/Knowledge/ResourceDao.php` |
| DAO/UnitPublisherDao | `ADLAPI/v3/App/Dao/Knowledge/UnitPublisherDao.php` |

## 主要資料表

| 資料表 | 用途 |
|--------|------|
| publisher_mapping | 出版商對應節點 (核心表) |
| map_node | 知識節點 (video/prac/DA/teach 標記) |
| map_info | 科目→map_sn 對應 |
| indicateTest | 上下位節點關聯 |
| video_concept_item | 教學影片 |
| map_node_file | 補充教材/學習單檔案 |
| map_node_external | 外部連結 |
| concept_item_interactive | 物理模擬題 |
| egame_info | 小試身手 |
| concept_itemBank_depiction | AI 學習夥伴 |
| subject_groupings | 科目群組 |
| seme_publisher | 學校預設出版商 |
| external_res_type | 資源類型對照 |

---

## Controller 方法對照

### 1. node() ✅ 實作完整

- **路由**: POST /knowledge/node
- **舊檔案對應**: `ks_viewskill_newScrept.php` 整個檔案
- **參數**: mapping_sn (必填)
- **驗證狀態**: ✅ 完整

**詳細比對**:

| 功能 | 舊系統函數 | 新系統方法 | 狀態 |
|------|-----------|-----------|------|
| 節點基本資訊 | `getSubIndData()` | `NodeDao::getNodeInfo()` | ✅ |
| 節點名稱 | `sn2nameForVideo()` | `NodeDao::getNodeName()` | ✅ |
| 資源標記 | `findIndData()` | `ResourceDao::getResourceFlags()` | ⚠️ 見下方 |
| 影片列表 | `getVideoInfo()` | `ResourceDao::getVideoList()` | ✅ |
| 補充教材檔案 | `getFileInfo()` | `ResourceDao::getFiles()` | ✅ |
| 外部連結 | `getExternalInfo()` | `ResourceDao::getExternalLinks()` | ✅ |
| 小試身手 | `getEGameInfo()` | `ResourceDao::getEGameUrl()` | ⚠️ |
| 上下位節點 | `getPrevNextIndicator()` | 另外拆到 `relatedNodes()` | ✅ 架構更好 |
| 權限過濾 | JS 前端 `bIsTeacher` | `getFilesWithPermission()` | ✅ 更安全 |

**差異說明**:

1. **物理模擬題影片連動** ✅:
   - 舊系統: `if ($sub_info['interactive'] == 1 && $sub_info['videolist'] > 0) { $sub_info['video'] = 1; }`
   - 新系統: `ResourceDao::getResourceFlags()` 第 59-62 行已實作相同邏輯
   - **狀態**: 已完整遷移

2. **Robot (AI 學習夥伴) 判斷** ⚠️:
   - 舊系統: 使用 `robotConfig('subject_open.map_sn')` 取得 map_sn 白名單 + 排除社會科(209) + 排除高中英語(subject_id=30, grade 10-12)
   - 新系統: 只排除社會科(209)，缺少 map_sn 白名單和高中英語排除
   - **影響**: 新系統可能在未開放科目顯示 Robot

3. **eGame URL 處理** ⚠️:
   - 舊系統: `createAPIDataURL($eGame_info[0], 'egame')` 加工過
   - 新系統: 直接回傳 raw link
   - **影響**: 前端需自行處理 URL 格式

4. **interactiveid 參數入口** ❌:
   - 舊系統: 支援 `interactiveid + subject_id` 參數組合 (物理模擬題入口)
   - 新系統: 只支援 `mapping_sn` 入口
   - **影響**: 物理模擬題的直接進入路徑不支援

5. **Session 寫入**:
   - 舊系統寫入大量 session (`indicator`, `mission_sn`, `subject_id`, `practice_mapSn`, `video_item_sn`)
   - 新系統無狀態，不寫 session → ✅ 正確做法

### 2. unit() ✅ 實作完整

- **路由**: POST /knowledge/unit
- **舊檔案對應**: `ks_viewunit_new.php` 中的節點列表查詢
- **參數**: subject_id, grade, sems, publisher_id, unit (必填), stage, system (選填)
- **驗證狀態**: ✅ 完整

**重點邏輯**:
- 高中分流 (system 參數): 普高/技高A/B/C/綜高 → ✅ 已處理
- 階段過濾 (stage 參數): 羅馬數字 I-V → ✅ 已處理
- 節點對應規則 (國語文/sub_indicator 科目/其他) → ✅ 已處理
- eGame 限定 subject_id=157 → ✅ 已處理
- 學習單檔案類型過濾 (學生只能看 pdf) → ✅ 已處理

### 3. list() ✅ 實作完整

- **路由**: POST /knowledge/list
- **舊檔案對應**: `assign_mission.php` 中的科目群組查詢
- **參數**: subject_groupings_id, grade (必填), publisher_id, sub_subject_id, system (選填)
- **驗證狀態**: ✅ 完整

**重點邏輯**:
- 出版商預設值: 學校設定 → 高中數學特殊 → 科目群組預設 → 第一個 → ✅ 完整
- 高中數學分流出版商: 10年級→117, 11→58, 12→72, 技高A→114 等 → ✅ 完整
- 學年度範圍: `getNowSemeYear() - 2` → ✅ 已處理
- sub_subject 處理: 排除新生物108(map_sn=121), 物理(map_sn=123) → ✅ 已處理
- 國語文 1 年級注音符號特殊處理: `Aa-Ⅰ` → unit=0, name='標音符號' → ✅ 已處理

### 4. relatedNodes() ✅ 實作完整

- **路由**: POST /knowledge/related-nodes
- **舊檔案對應**: `ks_viewskill_newScrept.php` → `getPrevNextIndicator()`
- **參數**: indicator, subject_id (必填)
- **驗證狀態**: ✅ 完整
- **邏輯**: 查 indicateTest 表 → 解析 `_SPLIT_SYMBOL` 分隔 → 處理 `[附加資訊]` → 取名稱

---

## 尚未遷移的功能

### A. 物理模擬題直接入口 ⚠️ 低優先

- **舊邏輯**: `interactiveid + subject_id` 參數組合，呼叫 `getSubIndData_Interactive()`
- **建議**: 可在 `node()` 增加選填參數 `interactiveid`，或另建端點
- **資料表**: `concept_itemBank_literacy_interactive`

### B. 子模組 API (影片/練習/填充/互動/DA/互動教學) ❌ 未實作

舊系統在節點頁面 require 以下子模組：
1. `modules/learn_video/video_new.php` - 影片播放
2. `modules/Practice/practice_new.php` - 練習題
3. `modules/blankFilling/blank_filling_new.php` - 填充題
4. `modules/physics_interactive/interactive_practice.php` - 物理模擬題
5. `modules/DynamicAdaptiveTest/DA_new.php` - 動態評量
6. `modules/New_CR/component_list_new.php` - 互動教學

這些已規劃在 **KsLearning Controller** 處理。

---

## 安全性檢查

| 項目 | 舊系統 | 新系統 | 狀態 |
|------|--------|--------|------|
| 權限過濾 | JS 前端判斷 bIsTeacher | Service 層 getFilesWithPermission() | ✅ 更安全 |
| 檔案路徑暴露 | `./data/materials/{map_sn}/{fileurl}` | 同樣暴露 | ⚠️ 待改善 |
| XSS | preventXss() | SanitizeMiddleware | ✅ OK |
| SQL Injection | PDO prepared | PDO prepared | ✅ OK |
| Session 依賴 | 大量 session | 無狀態 | ✅ 改善 |

---

## 特殊邏輯清單

1. **物理模擬題影片連動**: `interactive == 1 && videolist > 0` 時強制 `video = 1` ✅ 已實作
2. **Robot map_sn 白名單**: 舊系統使用 `robotConfig('subject_open.map_sn')` 動態取得，新系統無此機制
3. **Robot 高中英語排除**: 舊系統有 `subject_id = 30 AND grade BETWEEN 10 AND 12` 排除邏輯
4. **eGame URL 加工**: 舊系統使用 `createAPIDataURL()` 處理
5. **sub_indicator 科目清單**: 使用常數 `PUBLISHER_MAPPING_SNODE_SUBJECT`，fallback 為 `[2,3,28,29,117,120]`
6. **國語文注音符號**: subject_id=27, grade=1, `Aa-Ⅰ` 開頭的 indicate_id 特殊處理
7. **高中數學分流**: 5 種 system (普高/技高ABC/綜高)，影響 publisher_id 預設值和 indicate_id 過濾

---

## 總結

Knowledge Controller 是 6 個 controller 中**實作最完整**的。4 個端點全部 POST，核心業務邏輯遷移正確。

### 待補強項目 (依優先級排序)

| 優先級 | 項目 | 影響範圍 | 建議做法 |
|--------|------|----------|----------|
| 🔴 高 | Robot map_sn 白名單 | 可能在未開放科目顯示 Robot | 讀取 `robotConfig()` 或建立設定檔 |
| 🔴 高 | Robot 高中英語排除 | 高中英語(10-12年級) 錯誤顯示 | 加入 grade + subject_id 判斷 |
| 🟡 中 | eGame URL 加工 | 前端需額外處理 | 統一使用 `createAPIDataURL()` |
| 🟢 低 | 檔案下載路徑 | 路徑直接暴露 | 改用 API 端點代理下載 |
| 🟢 低 | interactiveid 入口 | 物理模擬題直接連結失效 | 視需求增加參數 |

### 已完成項目

- ✅ 4 個端點完整實作 (node, unit, list, related-nodes)
- ✅ 物理模擬題影片連動邏輯
- ✅ 高中數學分流出版商
- ✅ 權限過濾移至後端
- ✅ 無狀態設計 (不依賴 session)
