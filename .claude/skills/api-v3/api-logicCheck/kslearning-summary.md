# KsLearningController 完整邏輯檢查總結

## 1. 基本資訊

| 項目 | 內容 |
|------|------|
| **Controller** | KsLearningController |
| **端點總數** | 8 |
| **功能** | 知識結構學習活動（練習題、動態評量、影片） |
| **舊程式檔案** | Practice_Structure.php, practiceDB.php, DynamicAssessment.php, function_video.php, prodb_video.php |
| **新 API Service** | PracticeService, AssessmentService, VideoService |
| **新 API DAO** | PracticeDao, AssessmentDao, VideoDao |
| **相關資料表** | prac_answer, prac_single_answer, exam_record_dynamic, video_review_record, video_concept_item |
| **檢查日期** | 2026-02-11 |
| **檢查依據** | API mapping 文檔（詳細比對） |

---

## 2. 端點檢查總覽

| 端點 | 舊程式對應 | 邏輯一致性 | 主要問題 | 優先級 |
|------|-----------|-----------|---------|--------|
| practice | Practice_Structure.php | ✅ 基本對應 | 圖片路徑處理空實作 | 🟡 中 |
| practice-submit | practiceDB.php | ⚠️ 有差異 | 驗證邏輯不同 | 🟡 中 |
| assessment | DynamicAssessment.php | ✅ 基本對應 | 正確答案暴露 | 🔴 高 |
| assessment-submit | ExamRecord2DaDb.php | ⚠️ 有差異 | map_sn缺失 | 🔴 高 |
| video | function_video.php | ✅ 基本對應 | 續看功能缺失 | 🟡 中 |
| video/record | prodb_video.php | 🔴 嚴重問題 | 代幣未發放! | 🔴 Critical |
| checkpoint | prodb_showQuiz.php | ✅ 基本對應 | 語音檔缺失 | 🟢 低 |
| checkpoint-submit | prodb_exam_record.php | ⚠️ 有差異 | 防作弊檢查缺失 | 🟡 中 |

---

## 3. 🔴 嚴重問題（Critical - 影響功能正確性）

### 3.1 影片完成代幣未發放 🔴
```php
// 舊版 (prodb_video.php::fnUpdateRateEnd())
if ($finish_rate >= 100) {
    // 計算並發放代幣
    caculate_reward(...);
    caculate_mission_reward(...);
    giveCoin('video', $indicator, ...);
}

// 新版 (VideoService::updateProgress())
// ❌ 完成時只處理任務進度，未發放代幣！
if ($finishRate >= 100) {
    $this->handleVideoComplete(...);  // 只更新任務狀態
    // ❌ 缺少: giveCoin('video', ...)
}
```

**影響**: 🔴 **Critical** - 學生觀看影片完成後無法獲得代幣獎勵

### 3.2 影片完成節點狀態未更新 🔴
```php
// 舊版
UpdateNodeStatus($userId, $indicator, $mapSn, 'media_edu:', 1);

// 新版
// ❌ 完全缺失
```

**影響**: 🔴 **High** - 節點學習進度無法正確追蹤

### 3.3 動態評量 map_sn 缺失 🔴
```php
// Controller (KsLearningController::assessmentSubmit())
$validated = $this->validate([
    'indicator' => 'required|string',
    'mission_sn' => 'integer',
    'answers' => 'required|array'
    // ❌ 缺少: 'map_sn' => 'required|integer'
]);

// Service
$mapSn = $mapSn ?? 3;  // 預設值 3（僅對國語正確）
UpdateNodeStatus($userId, $indicator, $mapSn, ...);
```

**影響**: 🔴 **High** - 非國語科目的節點狀態更新錯誤

### 3.4 動態評量正確答案暴露 ⚠️
```php
// 新版 (AssessmentService::getQuestions())
return [
    'questions' => [...],
    'correct_answer' => $question->op_ans  // ❌ 暴露正確答案
];

// 舊版
// 不回傳正確答案，僅在後端驗證
```

**影響**: ⚠️ **Security** - 正確答案暴露給前端（為了 stateless 設計的犧牲）

---

## 4. 🟡 中度問題（影響完整性）

### 4.1 續看功能未實作
```php
// 舊版
fnCheckContinueWatch()  // 檢查是否有未完成的觀看
fnContinueWatch()       // 續接上次觀看

// 新版
// ❌ 未實作，每次都建新紀錄
```

**影響**: 🟡 使用者體驗下降（無法續看上次未完成的影片）

### 4.2 播放速度（turbo）未處理
```php
// 舊版
insertVideoReviewRecordPlus(..., $turbo);  // 記錄播放速度

// 新版
insertVideoRecordPlus(..., null);  // ❌ 固定傳 null
```

**影響**: 🟡 無法追蹤播放速度變

更（統計分析缺失）

### 4.3 防作弊「已預習過」邏輯缺失
```php
// 舊版
$hasFinished = getVideoFinish(...);  // 檢查是否曾完成
if ($hasFinished) {
    $checkType = 2;  // 免檢查
}

// 新版
// ❌ 只看 finishCount，未判斷「已預習過」
```

**影響**: 🟡 已預習過的學生仍需防作弊檢查（體驗差）

### 4.4 圖片路徑處理空實作
```php
// PracticeService::processMediaPath() (L329-334)
private function processMediaPath($content)
{
    // TODO: Process media paths if needed
    return $content;  // ❌ 空實作，未處理圖片路徑
}
```

**影響**: 🟡 練習題圖片可能無法正確顯示

### 4.5 檢核點提交缺少防作弊
```php
// 舊版
doVideoActionRecord($oTmpParm);  // 含完整防作弊檢查

// 新版
VideoDao::insertVideoRecordPlus('chkptend', ...);  // ❌ 只寫 plus，不做防作弊
```

**影響**: 🟡 檢核點答題可能被作弊

---

## 5. 其他差異

### 5.1 練習題提交驗證差異
```php
// 舊版
if ($questions_count === $idle_time_count) {
    // 驗證題數和作答時間筆數相同
}

// 新版
if (count($answers) !== $itemsNum) {
    // 驗證答案數 = 題目數
}
```

**差異**: 驗證邏輯不同，但都合理

### 5.2 動態評量 DA 全答錯 Bug 修正 ✅
```php
// 舊版 (BUG)
giveCoin('practice', ...);  // ❌ 應該是 'DA'

// 新版
giveCoin('DA', ...);  // ✅ 已修正
```

**狀態**: ✅ 新版修正了舊版 Bug

### 5.3 Transaction 缺失
舊版多處使用 `$dbh->beginTransaction()` + commit/rollback，新版無 Transaction

**影響**: ⚪ 低（大部分操作為單一 INSERT）

---

## 6. 設計改進 ✅

### 6.1 Stateless 設計
- ✅ 移除 session 依賴
- ✅ 使用 token 鑑權

### 6.2 練習題單題紀錄
- ✅ 新增 `prac_single_answer` 表
- ✅ 提供更細粒度的作答紀錄

### 6.3 Hash 加密移除
- ✅ 不再使用 `string2hash()` / `hash2string()`
- ✅ 改用 token 鑑權（更現代）

---

## 7. 缺少的 API

| 功能 | 舊版位置 | 建議 | 優先級 |
|------|---------|------|--------|
| 續看檢查 | fnCheckContinueWatch() | 新增 /ks-learning/video/check-continue | 🟡 中 |
| 續看執行 | fnContinueWatch() | 新增 /ks-learning/video/continue | 🟡 中 |
| 影片筆記 | prodb_note_record.php | 視需求新增 | 🟢 低 |
| 影片討論 | prodb_discuss.php | 視需求新增 | 🟢 低 |

---

## 8. 改進建議總結

### 🔴 Critical - 必須立即修正

| 項目 | 影響 | 建議做法 |
|------|------|----------|
| 影片完成代幣未發放 | 學生無法獲得代幣 | 在 VideoService::handleVideoComplete() 中加入 giveCoin('video', ...) |
| 影片完成節點狀態未更新 | 節點進度追蹤錯誤 | 加入 UpdateNodeStatus(..., 'media_edu:', 1) |
| 動態評量 map_sn 缺失 | 非國語科目狀態錯誤 | Controller 增加 map_sn 參數驗證 |

### 🟡 Medium - 建議改善

| 項目 | 影響 | 建議做法 |
|------|------|----------|
| 續看功能缺失 | 使用者體驗下降 | 實作 check-continue 和 continue API |
| turbo 未處理 | 無法追蹤播放速度 | 傳入實際 turbo 值 |
| 防作弊邏輯不完整 | 可能被作弊 | 加入「已預習過」判斷 |
| 圖片路徑處理空實作 | 圖片可能無法顯示 | 實作 processMediaPath() |

### ⚠️ Security - 需評估

| 項目 | 影響 | 建議做法 |
|------|------|----------|
| 正確答案暴露 | 安全性降低 | 評估是否必要，或改為後端驗證 |

---

## 9. 總體檢查結論

| 檢查項目 | 評估結果 |
|---------|----------|
| **邏輯一致性** | ⚠️ 70% 一致（有多項缺失） |
| **安全性** | ⚠️ 有安全性問題（正確答案暴露） |
| **功能完整性** | 🔴 缺少關鍵功能（代幣、節點狀態） |
| **優先級評估** | 🔴 **Critical - 影片代幣和節點狀態必須立即修正** |

### 整體評估
**KsLearningController 有嚴重功能缺失！**

🔴 **Critical 問題**:
- 影片完成代幣未發放（破壞激勵機制）
- 影片完成節點狀態未更新（破壞學習追蹤）
- 動態評量 map_sn 缺失（破壞非國語科目）

✅ **優點**:
- Stateless 設計改進
- 練習題單題紀錄（更細粒度）
- 修正舊版 DA 全答錯 Bug

⚠️ **中度問題**:
- 續看功能缺失
- 防作弊邏輯不完整
- 播放速度追蹤缺失

### 結論
**邏輯基本一致，但有多項嚴重缺失需立即修正**，特別是影片相關的代幣發放和節點狀態更新。
