# KnowledgeController 完整邏輯檢查總結

## 1. 基本資訊

| 項目 | 內容 |
|------|------|
| **Controller** | KnowledgeController |
| **端點總數** | 4 |
| **功能** | 知識結構（出版商節點、單元列表、學習資源） |
| **舊程式檔案** | `ks_viewskill_newScrept.php`, `ks_viewunit_new.php`, `assign_mission.php` |
| **新 API Service** | `KnowledgeService.php` |
| **新 API DAO** | `NodeDao.php`, `ResourceDao.php`, `UnitPublisherDao.php` |
| **相關資料表** | publisher_mapping, map_node, map_info, video_concept_item, map_node_file |
| **檢查日期** | 2026-02-11 |
| **檢查依據** | API mapping 文檔 |

---

## 2. 端點檢查總覽

| 端點 | 舊程式對應 | 邏輯一致性 | 主要差異 | 優先級 |
|------|-----------|-----------|---------|--------|
| node | ks_viewskill_newScrept.php | ✅ 90% | Robot白名單、eGame URL | 🔴 高 |
| unit | ks_viewunit_new.php | ✅ 100% | 無 | - |
| list | assign_mission.php | ✅ 100% | 無 | - |
| relatedNodes | getPrevNextIndicator() | ✅ 100% | 架構更好 | - |

---

## 3. node 端點詳細檢查

### 3.1 功能對照

| 功能 | 舊系統函數 | 新系統方法 | 狀態 |
|------|-----------|-----------|------|
| 節點基本資訊 | getSubIndData() | NodeDao::getNodeInfo() | ✅ |
| 節點名稱 | sn2nameForVideo() | NodeDao::getNodeName() | ✅ |
| 資源標記 | findIndData() | ResourceDao::getResourceFlags() | ✅ |
| 影片列表 | getVideoInfo() | ResourceDao::getVideoList() | ✅ |
| 補充教材檔案 | getFileInfo() | ResourceDao::getFiles() | ✅ |
| 外部連結 | getExternalInfo() | ResourceDao::getExternalLinks() | ✅ |
| 小試身手 | getEGameInfo() | ResourceDao::getEGameUrl() | ⚠️ |
| 上下位節點 | getPrevNextIndicator() | relatedNodes() | ✅ 架構更好 |
| 權限過濾 | JS 前端 bIsTeacher | getFilesWithPermission() | ✅ 更安全 |

### 3.2 重要邏輯檢查

#### ✅ 物理模擬題影片連動
```php
// 舊版 (ks_viewskill_newScrept.php)
if ($sub_info['interactive'] == 1 && $sub_info['videolist'] > 0) {
    $sub_info['video'] = 1;
}

// 新版 (ResourceDao::getResourceFlags() L59-62)
// 已實作相同邏輯
```
**狀態**: ✅ 已完整遷移

#### ⚠️ Robot (AI 學習夥伴) 判斷
```php
// 舊版
$robot_open_map_sn = robotConfig('subject_open.map_sn'); // 白名單
if (in_array($map_sn, $robot_open_map_sn) 
    && $map_sn != 209  // 排除社會科
    && !($subject_id == 30 && $grade >= 10 && $grade <= 12)) { // 排除高中英語
    $robot = 1;
}

// 新版
// 只排除社會科(209)，缺少map_sn白名單和高中英語排除
```
**影響**: 🔴 新系統可能在未開放科目顯示 Robot

#### ⚠️ eGame URL 處理
```php
// 舊版
$eGame_url = createAPIDataURL($eGame_info[0], 'egame'); // 加工過

// 新版
// 直接回傳 raw link
```
**影響**: ⚠️ 前端需自行處理 URL 格式

#### ❌ interactiveid 參數入口 (低優先級)
```php
// 舊版支援
if (isset($_POST['interactiveid'])) {
    // 物理模擬題直接入口
}

// 新版不支援
```
**影響**: 🟢 物理模擬題的直接進入路徑不支援（低優先級）

#### ✅ Session 處理改善
```php
// 舊版
$_SESSION['indicator'] = $indicator;
$_SESSION['mission_sn'] = $mission_sn;
// ... 大量 session 寫入

// 新版
// 無狀態，不寫 session
```
**狀態**: ✅ 改善（正確做法）

---

## 4. unit 端點詳細檢查

### 4.1 參數
- subject_id, grade, sems, publisher_id, unit (必填)
- stage, system (選填)

### 4.2 重點邏輯
✅ **高中分流 (system 參數)**: 普高/技高A/B/C/綜高 → 已處理  
✅ **階段過濾 (stage 參數)**: 羅馬數字 I-V → 已處理  
✅ **節點對應規則**: 國語文/sub_indicator 科目/其他 → 已處理  
✅ **eGame 限定**: subject_id=157 → 已處理  
✅ **學習單檔案類型過濾**: 學生只能看 pdf → 已處理

### 4.3 檢查結論
**邏輯一致性**: ✅ **100% 完整**  
**狀態**: 無需改進

---

## 5. list 端點詳細檢查

### 5.1 參數
- subject_groupings_id, grade (必填)
- publisher_id, sub_subject_id, system (選填)

### 5.2 重點邏輯
✅ **出版商預設值**: 學校設定 → 高中數學特殊 → 科目群組預設 → 第一個  
✅ **高中數學分流出版商**: 10年級→117, 11→58, 12→72, 技高A→114 等  
✅ **學年度範圍**: getNowSemeYear() - 2  
✅ **sub_subject 處理**: 排除新生物108(map_sn=121), 物理(map_sn=123)  
✅ **國語文注音符號**: subject_id=27, grade=1, Aa-Ⅰ → unit=0, name='標音符號'

### 5.3 檢查結論
**邏輯一致性**: ✅ **100% 完整**  
**狀態**: 無需改進

---

## 6. relatedNodes 端點詳細檢查

### 6.1 功能
取得節點的上下位關聯節點

### 6.2 邏輯
- 查 indicateTest 表
- 解析 `_SPLIT_SYMBOL` 分隔
- 處理 `[附加資訊]`
- 取節點名稱

### 6.3 檢查結論
**邏輯一致性**: ✅ **100% 完整**  
**狀態**: ✅ 架構更好（從 node 端點拆分出來）

---

## 7. 安全性檢查

| 項目 | 舊系統 | 新系統 | 狀態 |
|------|--------|--------|------|
| 權限過濾 | JS 前端判斷 bIsTeacher | Service 層 getFilesWithPermission() | ✅ 更安全 |
| 檔案路徑暴露 | `./data/materials/{map_sn}/{fileurl}` | 同樣暴露 | ⚠️ 待改善 |
| XSS | preventXss() | SanitizeMiddleware | ✅ OK |
| SQL Injection | PDO prepared | PDO prepared | ✅ OK |
| Session 依賴 | 大量 session | 無狀態 | ✅ 改善 |

---

## 8. 改進建議總結

### 🔴 高優先級

| 項目 | 影響 | 建議做法 |
|------|------|----------|
| Robot map_sn 白名單缺失 | 可能在未開放科目顯示 Robot | 讀取 robotConfig() 或建立設定檔 |
| Robot 高中英語排除缺失 | 高中英語(10-12年級) 錯誤顯示 Robot | 加入 grade + subject_id 判斷 |

### 🟡 中優先級

| 項目 | 影響 | 建議做法 |
|------|------|----------|
| eGame URL 加工缺失 | 前端需額外處理 | 統一使用 createAPIDataURL() |

### 🟢 低優先級

| 項目 | 影響 | 建議做法 |
|------|------|----------|
| 檔案下載路徑暴露 | 路徑直接暴露 | 改用 API 端點代理下載 |
| interactiveid 入口缺失 | 物理模擬題直接連結失效 | 視需求增加參數 |

---

## 9. 特殊邏輯清單

1. **物理模擬題影片連動**: interactive == 1 && videolist > 0 時強制 video = 1 ✅
2. **Robot map_sn 白名單**: 使用 robotConfig('subject_open.map_sn') ⚠️ 缺失
3. **Robot 高中英語排除**: subject_id = 30 AND grade BETWEEN 10 AND 12 ⚠️ 缺失
4. **eGame URL 加工**: createAPIDataURL() 處理 ⚠️ 缺失
5. **sub_indicator 科目清單**: 使用常數 PUBLISHER_MAPPING_SNODE_SUBJECT ✅
6. **國語文注音符號**: Aa-Ⅰ 特殊處理 ✅
7. **高中數學分流**: 5 種 system 影響 publisher_id 預設值 ✅

---

## 10. 未實作功能

### 子模組 API ❌ (已規劃在 KsLearningController)
舊系統在節點頁面 require 以下子模組：
1. `learn_video/video_new.php` - 影片播放
2. `Practice/practice_new.php` - 練習題
3. `blankFilling/blank_filling_new.php` - 填充題
4. `physics_interactive/interactive_practice.php` - 物理模擬題
5. `DynamicAdaptiveTest/DA_new.php` - 動態評量
6. `New_CR/component_list_new.php` - 互動教學

**狀態**: 已規劃在 **KsLearningController** 處理

---

## 11. 總體檢查結論

| 檢查項目 | 評估結果 |
|---------|----------|
| **邏輯一致性** | ✅ 90% 一致（node 端點有小問題） |
| **安全性** | ✅ 改善（權限後移、無狀態） |
| **功能完整性** | ✅ 4個端點全部實作完整 |
| **優先級評估** | 🔴 高優先級 - Robot 判斷邏輯需補強 |

### 整體評估
**KnowledgeController 是 6 個 Controller 中實作最完整的！**

✅ **優點**:
- 4 個端點完整實作 (node, unit, list, relatedNodes)
- 物理模擬題影片連動邏輯完整
- 高中數學分流出版商處理完整
- 權限過濾移至後端（更安全）
- 無狀態設計（不依賴 session）

⚠️ **待補強**:
- Robot map_sn 白名單機制
- Robot 高中英語排除邏輯
- eGame URL 加工處理

### 結論
**邏輯基本一致，僅需小幅補強 Robot 判斷邏輯**
