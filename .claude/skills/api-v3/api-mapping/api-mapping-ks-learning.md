# API-V3 對照表：KsLearning Controller

## Controller 資訊
- **檔案**: `ADLAPI/v3/App/Controller/KsLearningController.php`
- **Service**: `PracticeService`, `AssessmentService`, `VideoService` (in `App\Service\KsLearning\`)
- **DAO**: `PracticeDao`, `AssessmentDao`, `VideoDao` (in `App\Dao\KsLearning\`)

---

## 端點對照總覽

| # | V3 端點 | HTTP | 舊檔案 | 舊函式/動作 | 狀態 |
|---|---------|------|--------|-------------|------|
| 1 | `/ks-learning/practice` | POST | `modules/Practice/Practice_Structure.php` | `Practice_Structure` class | ✅ 基本對應 |
| 2 | `/ks-learning/practice-submit` | POST | `modules/Practice/practiceDB.php` | 整份腳本 | ⚠️ 有差異 |
| 3 | `/ks-learning/assessment` | POST | `classes/DynamicAssessment.php` | `DynamicAssessment` class | ✅ 基本對應 |
| 4 | `/ks-learning/assessment-submit` | POST | `modules/DynamicAdaptiveTest/ExamRecord2DaDb.php` | 整份腳本 | ⚠️ 有差異 |
| 5 | `/ks-learning/video` | POST | `modules/learn_video/function_video.php` | `insertVideoReviewRecord()` | ✅ 基本對應 |
| 6 | `/ks-learning/video/record` | POST | `modules/learn_video/prodb_video.php` | `fnUpdateRate()` / `fnUpdateRateEnd()` | ⚠️ 有差異 |
| 7 | `/ks-learning/checkpoint` | POST | `modules/learn_video/prodb_showQuiz.php` | `fnGetVideoQuestion()` | ✅ 基本對應 |
| 8 | `/ks-learning/checkpoint-submit` | POST | `modules/learn_video/prodb_exam_record.php` | 整份腳本 | ⚠️ 有差異 |

---

## 1. 練習題取得 `/ks-learning/practice`

### 參數對照
| V3 參數 | 舊參數來源 | 說明 |
|---------|-----------|------|
| `indicator` (required) | `$_indicator` (session/URL) | 知識點 ID |
| `map_sn` (required) | `$_SESSION['subject_id']` → `subject2mapSN()` | V3 直接傳入；舊版由 subject 轉換 |
| `mission_sn` (optional) | `$mission_sn` (URL param) | 任務編號 |

### 邏輯對照
- **V3**: `PracticeService::getQuestions()` → 使用 `require_once Practice_Structure.php` 取得題目
- **舊**: `Practice_Structure` class 直接在前端 PHP 使用
- **一致**: 都用 `Practice_Structure` class 的 `getItemsNum()`, `getItemsNumOrder()`, `getItemTopic()`, `getItemOptions()`, `getItemQid()`
- **回傳**: V3 包裝為 JSON `{indicator, indicator_name, map_sn, mission_sn, total_questions, questions}`

### 特殊邏輯
- ✅ `_SPLIT_SYMBOL` 分隔符處理保留
- ⚠️ **V3 的 `processMediaPath()` 是空實作** (line 329-334)：只 return content，未處理圖片路徑轉換。舊版有 `$sDirPath` 處理邏輯將相對路徑轉為完整 URL

---

## 2. 練習題提交 `/ks-learning/practice-submit`

### 參數對照
| V3 參數 | 舊參數來源 | 說明 |
|---------|-----------|------|
| `indicator` (required) | `$_indicator` | 知識點 ID |
| `map_sn` (required) | `subject2mapSN()` | 舊版由 subject 轉；V3 直接傳入 |
| `mission_sn` (optional) | `$mission_sn` | 任務編號 |
| `answers` (required, array) | session-based `$practice_question` | V3 為 `[{qid, answer, client_start_ms, client_end_ms}]` |

### 邏輯對照

#### 寫入 DB
| 步驟 | V3 (PracticeService) | 舊 (practiceDB.php) | 差異 |
|------|---------------------|---------------------|------|
| 歷史查詢 | `PracticeDao::getHistoryRecords()` | 直接 SQL 查 `prac_answer` JOIN `prac_questions` | ✅ 邏輯一致 |
| 首次100判斷 | `!$hasHistory100 && $historyCount < 3 && $scoreRate == 100` | `!$isHistory100 && $iSerchHistoryCount < 3 && $acc_rate == 100` | ✅ 一致 |
| 整卷寫入 | `PracticeDao::insertPracAnswer()` | INSERT INTO `prac_answer` | ✅ 一致 |
| 單題寫入 | `PracticeDao::insertSingleAnswer()` → INSERT `prac_single_answer` | **無** | ⚠️ **V3 新增功能** |
| 驗證組數 | `count($answers) !== $itemsNum` | `$questions_count === $idle_time_count` | ⚠️ 舊版驗證的是 questions 和 idle_time 組數相同；V3 驗證答案數=題數 |

#### 代幣邏輯
| 步驟 | V3 | 舊 | 差異 |
|------|------|------|------|
| 教師權限檢查 | `PracticeDao::getMissionTeacherLevel()` | 直接 SQL 查 `mission_info` JOIN `user_status` | ✅ 一致 |
| 學生判斷 | `in_array($accessLevel, USER_STUDENT_GROUP)` | `in_array($v_user_data['access_level'], USER_STUDENT_GROUP)` | ✅ 一致 |
| 全答錯處理 | `give_role_coin(-1, $extraCoins, $missionSn, 0)` | `give_role_coin($student_get_coins=-1, $student_get_extra_coins, $mission_sn, $student_iIsFirst_coinsNum)` | ⚠️ V3 的第4個參數固定 0，舊版傳 `$student_iIsFirst_coinsNum`（但全答錯時這個值也不會被計算）|
| 正常代幣 | `caculate_mission_reward()` + `caculate_extra_reward()` | 同上 | ✅ 一致 |

#### 任務/節點更新
- ✅ 任務進度: `update_mission_record("prac")` 一致
- ✅ 節點狀態: `UpdateNodeStatus($userId, $indicator, $mapSn, 'practice:', $acc_binary)` 一致

### 缺失邏輯
- ⚠️ **CSRF 驗證**: 舊版有 `$_POST['csrf_token'] !== $_SESSION['csrf_token']` 檢查；V3 用 token 驗證替代
- ⚠️ **Session 清理**: 舊版有 `unset($_SESSION['practice_question'])` 等清理邏輯；V3 為 stateless 不需要
- ⚠️ **組數驗證差異**: 舊版驗證 questions 和 idle_time 數量相同才寫入 DB，V3 無此驗證（但有答案數=題目數驗證）
- ⚠️ **stop_time 基準**: 舊版 `$stop_time` 有 `>= 1635696000` (2021-11-01) 過濾條件；需確認 V3 DAO 是否保留

---

## 3. 動態評量取得 `/ks-learning/assessment`

### 參數對照
| V3 參數 | 舊參數來源 | 說明 |
|---------|-----------|------|
| `indicator` (required) | URL/session `$indicator` | 知識點 ID |
| `mission_sn` (optional) | session `$mission_sn` | 任務編號 |

### 邏輯對照
- **V3**: `AssessmentService::getQuestions()` → 使用 `require_once DynamicAssessment.php`
- **舊**: `DynamicAssessment` class 在 DA.php / DA_new.php 中使用
- **一致**: `set_item_data()`, `set_item_sequence()`, `get_item_num()`, `get_selected_item_data()`

### 特殊邏輯
- ✅ V3 包含 feedback（回饋圖片）：從 `$question->sol_pieces` 和 `get_url_path()` 組裝
- ✅ V3 包含 `correct_answer`：直接回傳正確答案給前端（stateless 設計需要）
- ⚠️ **舊版不回傳正確答案**：答案在伺服器端驗證。V3 的設計將正確答案暴露給前端（為了 stateless），安全性降低

---

## 4. 動態評量提交 `/ks-learning/assessment-submit`

### 參數對照
| V3 參數 | 舊參數來源 | 說明 |
|---------|-----------|------|
| `indicator` (required) | `$DA->get_ind_id()` | 知識點 ID |
| `mission_sn` (optional) | `$_SESSION['mission_sn']` | 任務編號 |
| `answers` (required, array) | session `$_SESSION['response']` | V3: `[{attempts: [{answer, client_start_ms, client_end_ms}]}]` |

### 邏輯對照

#### 計分規則
| 規則 | V3 | 舊 | 差異 |
|------|------|------|------|
| 第一次答對 | `binaryRes = 1`, `totalScore += itemScore` | `tmp_item_response = 1` if `response[$i] >= 1` | ⚠️ **差異**: 舊版 `response[$i]` 的值由 `Student_Data` class 內部計算（1=第一次答對，其他=非第一次答對）；V3 用 `attempts[0].answer === correctAnswer` 判斷 |
| 補救後答對 | `binaryRes = 0` | 同上 | ✅ 一致 |
| 答錯 | `binaryRes = 0` | 同上 | ✅ 一致 |

#### 寫入 DB
| 步驟 | V3 | 舊 | 差異 |
|------|------|------|------|
| INSERT | `AssessmentDao::insertExamRecord()` → `exam_record_dynamic` | INSERT INTO `exam_record_dynamic` | ✅ 欄位一致 |
| `fn_is_empty` | V3 DAO 使用 `IF(fn_is_empty(:mission_sn), DEFAULT(mission_sn), :mission_sn2)` | 同樣用 `fn_is_empty` | ✅ 一致 |
| `seme_year` | `getSemeYear()` 函式 | `getNowSemeYear()` | ✅ 一致（V3 有 fallback） |
| `ip` | `$_SERVER['REMOTE_ADDR']` | `$ip` | ✅ 一致 |

#### 代幣邏輯
| 步驟 | V3 | 舊 | 差異 |
|------|------|------|------|
| 家長檢查 | `AssessmentDao::getMissionCreatorAccessLevel()` | 直接 SQL 查 `user_status` JOIN `mission_info` | ✅ 一致 |
| 全答錯處理 | `caculate_extra_reward` + return `$extraCoins` | `giveCoin('practice', ...)` + `caculate_extra_reward` | ⚠️ **BUG**: 舊版全答錯時用 `'practice'` 作為 action_name（應為 `'DA'`），V3 修正為 `'DA'` |
| 歷史查詢 | `AssessmentDao::getHistoryRecords()` 查 `exam_record_dynamic` | 直接 SQL 查 `exam_record_dynamic` WHERE `cs_id` | ✅ 一致 |
| 首次100 | `!$hasHistory100 && $historyCount < 3 && $scoreRate == 100` | 同上 | ✅ 一致 |

#### 任務/節點更新
- ✅ `update_mission_record($missionSn, "da", $userId, $indicator)` 一致
- ✅ `UpdateNodeStatus()` with `'DA:'` prefix 一致
- ⚠️ **V3 缺少 map_sn 參數**: Controller 未傳 `map_sn` 到 Service（用 `$mapSn ?? 3` 預設值）；舊版用 `subject2mapSN()` 計算

### 缺失邏輯
- ⚠️ **推薦筆記**: 舊版有 `checkIfHasRecommendNotes()` 判斷是否推薦筆記；V3 無此功能
- ⚠️ **SRL 警告**: 舊版當 `$thisTestSnode_rate <= CORRECT_PERCENT_DA` 時顯示 SRL 警告和重新觀看建議；V3 未實作
- ⚠️ **Session 清理**: 舊版有大量 session 清理（`$_SESSION['DA']`, `$_SESSION['Student_Data']`, `exam_clean_all()`）；V3 為 stateless 不需要
- ⚠️ **map_sn 取得方式**: 舊版用 `subject2mapSN($_SESSION['FromGet']['subject'])` 動態計算；V3 使用 Controller 傳入的 mapSn 但 **Controller 未要求此參數**（assessmentSubmit 的 validate 中無 map_sn）

---

## 5. 影片開始觀看 `/ks-learning/video`

### 參數對照
| V3 參數 | 舊參數來源 | 說明 |
|---------|-----------|------|
| `video_item_sn` (required) | `hash2string($oPostData['video_item_sn'])` | 影片編號，舊版需 hash 解碼 |
| `mission_sn` (optional) | session `$mission_sn` | 任務編號 |

### 邏輯對照
- **V3**: `VideoService::startVideo()` → `VideoDao::insertVideoReviewRecord()` + `insertVideoRecordPlus('browse')`
- **舊**: `function_video.php::insertVideoReviewRecord()` + `insertVideoReviewRecordPlus()`
- ✅ INSERT INTO `video_review_record` 欄位一致
- ✅ 自動寫入 browse action 一致
- ✅ 取得完成次數 `getVideoFinishCount()` 一致

### V3 額外回傳
- `video_sources`: 處理 MP4 和 HLS 兩種來源
- `checkpoints`: 檢核點列表
- `finish_count`: 完成次數

### 缺失邏輯
- ⚠️ **續看功能 (Continue Watch)**: 舊版有 `fnCheckContinueWatch()` 和 `fnContinueWatch()` 用於檢查/續接上次觀看紀錄；V3 未實作此功能
- ⚠️ **hash 加密**: 舊版 review_sn 回傳時用 `string2hash()` 加密；V3 直接回傳明文

---

## 6. 影片進度更新 `/ks-learning/video/record`

### 參數對照
| V3 參數 | 舊參數來源 | 說明 |
|---------|-----------|------|
| `review_sn` (required) | `hash2string($oPostData['review_sn'])` | 觀看紀錄編號 |
| `view_action` (required, string) | `base64_decode($oPostData['view_action'])` → VIEW_ACTIONS 對照 | V3 直接傳動作字串；舊版傳數字碼 |
| `end_timestamp` (required) | `base64_decode($oPostData['end_timestamp'])` | 影片時間戳 |
| `finish_rate` (required) | `base64_decode($oPostData['finish_rate'])` | 完成率 0-100 |

### 邏輯對照

#### 防作弊 (checkCheat)
| 規則 | V3 | 舊 | 差異 |
|------|------|------|------|
| checkType 0 | 國中小首次 | 同上 | ✅ 一致 |
| checkType 1 | 可快轉不可拖拉 | 同上 | ⚠️ V3 的 `determineCheckType()` 未回傳 1（只有 0 和 2）|
| checkType 2 | 高中+/老師/已完成 | 同上 | ✅ 一致 |
| map_sn=134 特殊 | 數位學習工作坊A 老師首次需檢查 | 同上 | ✅ 一致 |
| 已預習過 | **缺失** | `getVideoFinish()` 檢查是否預習過影片 | ⚠️ **V3 缺少**：舊版有 `getVideoFinish()` 判斷學生是否曾完成該影片（非本次紀錄），已完成者免檢查 |

#### 進度更新
| 步驟 | V3 | 舊 | 差異 |
|------|------|------|------|
| UPDATE record | `VideoDao::updateVideoProgress()` | `updateVideoReviewRecordByUpdateRate()` | ⚠️ 舊版 total_time 用 `strtotime(end_time) - strtotime(start_time)` 計算；V3 用 `time() - strtotime(start_time)` |
| INSERT plus | `VideoDao::insertVideoRecordPlus()` | `insertVideoReviewRecordPlus()` | ✅ 一致 |
| turbo 參數 | V3 的 insertVideoRecordPlus 固定傳 null | 舊版根據 view_action 類型決定是否傳 turbo | ⚠️ **V3 未處理 turbo**：播放速度變更的 turbo 值未傳入 |

### 缺失邏輯
- ⚠️ **updateRate vs updateRateEnd 分離**: 舊版將「進度中更新」和「觀看結束」分為兩個 action：`fnUpdateRate()` 不處理 end/100%，`fnUpdateRateEnd()` 專門處理完成。V3 合併為一個 `updateProgress()` 但缺少結束時的完整流程
- ⚠️ **影片完成時的代幣**: 舊版 `fnUpdateRateEnd()` 有完整的代幣發放（含 `caculate_reward` 和 `caculate_mission_reward`）；V3 的 `handleVideoComplete()` 只處理任務進度更新，**未處理影片觀看完成的代幣發放**
- ⚠️ **節點狀態更新**: 舊版完成時有 `UpdateNodeStatus($userId, $indicator, $mapSn, 'media_edu:', 1)` 更新（首次觀看時）；V3 未實作
- ⚠️ **SRL 警告/推薦筆記**: 舊版完成時檢查檢核點答對率和推薦筆記；V3 未回傳此資訊
- ⚠️ **拖拉/回轉處理**: 舊版有 `VIEW_ACTION_EXTRA['cangoback']` 特殊處理（可重設 start_timestamp）；V3 未處理
- ⚠️ **重複播放速度檢查**: 舊版有 `getLastTurbo()` 阻止重複操作；V3 無此機制
- ⚠️ **Transaction**: 舊版用 `$dbh->beginTransaction()` + commit/rollback；V3 無 Transaction

---

## 7. 檢核點取得 `/ks-learning/checkpoint`

### 參數對照
| V3 參數 | 舊參數來源 | 說明 |
|---------|-----------|------|
| `review_sn` (required) | `hash2string($oPostData['review_sn'])` | 觀看紀錄編號 |
| `question_sn` (required) | `hash2string($oPostData['question_sn'])` | 檢核點編號 |

### 邏輯對照
- **V3**: `VideoService::getCheckpointQuestion()` → `VideoDao::getCheckpointQuestion()` (SELECT * FROM video_concept_item_plus INNER JOIN video_concept_item)
- **舊**: `prodb_showQuiz.php::fnGetVideoQuestion()` → `getVideoQuestion()` (相同 SQL)
- ✅ SQL 查詢一致
- ✅ MCQ/CQ 題型判斷一致（用 `_SPLIT_SYMBOL` 判斷）
- ✅ server_timestamp 記錄一致（用於計算作答時間）

### 差異
- ⚠️ **hash 加密**: 舊版回傳時 `question_sn` 和 `question_timestamp` 用 `string2hash()` 加密；V3 回傳明文
- ⚠️ **session 記錄**: 舊版將 timestamp 存入 `$_SESSION['showQuiz_timestamp']` 作為 fallback；V3 直接回傳 server_timestamp 由前端傳回
- ⚠️ **語音檔**: 舊版回傳 `voice` (item_filename) 和 `has_voice`；V3 未回傳語音檔相關資訊

---

## 8. 檢核點提交 `/ks-learning/checkpoint-submit`

### 參數對照
| V3 參數 | 舊參數來源 | 說明 |
|---------|-----------|------|
| `review_sn` (required) | `hash2string($oPostData['review_sn'])` | 觀看紀錄編號 |
| `question_sn` (required) | `hash2string($oPostData['question_sn'])` | 檢核點編號 |
| `answer` (required) | `$_POST['op_ans']` | 使用者答案 |
| `finish_rate` (required) | `$_POST['finish_rate']` | 當前完成率 |
| `server_timestamp` (optional) | `hash2string($oPostData['server_start_timestamp'])` | 開始作答時間 |

### 邏輯對照

#### 答案判斷
| 步驟 | V3 | 舊 | 差異 |
|------|------|------|------|
| 正確性判斷 | `$question['op_ans'] == $answer` | `$row['op_ans'] == $sOpAns` | ✅ 一致 |
| 作答時間 | `microtime(true)*1000 - $serverStartTimestamp` | 同上（含 session fallback） | ⚠️ V3 無 session fallback |
| 合理性驗證 | `$ansTime < 0 || > 3600000` → clamp | 同上 | ✅ 一致 |

#### 代幣邏輯
| 步驟 | V3 | 舊 | 差異 |
|------|------|------|------|
| 條件 | 答對 + 無任務 + 未曾答對過 | 答對 + 無任務 + 未曾答對過 + 學生身份 | ✅ V3 有學生判斷 (在 `giveCheckpointCoin` 內) |
| 發放 | `giveCoin('video_question', $questionSn)` → `give_role_coin(0,0,false,1,0)` | 同上 | ✅ 一致 |

#### 後續處理
| 步驟 | V3 | 舊 | 差異 |
|------|------|------|------|
| video action record | 寫入 `chkptend` 到 `video_review_record_plus` | `doVideoActionRecord($oTmpParm)` with view_action=22 | ⚠️ V3 只寫 plus 不更新 record；舊版透過 `doVideoActionRecord()` 同時更新 record 和 plus（含防作弊檢查） |
| 答錯回應 | 回傳 `answer_text` (MCQ: "選項 X" / CQ: 原文) | 回傳 msg (同上) | ✅ 格式一致 |

### 缺失邏輯
- ⚠️ **Transaction**: 舊版用 `$dbh->beginTransaction()` 包裹整個流程（insert exam_record + doVideoActionRecord）；V3 無 Transaction
- ⚠️ **doVideoActionRecord 防作弊**: 舊版提交答案後會呼叫 `doVideoActionRecord()` 做完整的防作弊檢查和進度更新；V3 只寫 plus 紀錄，不做額外驗證
- ⚠️ **hash 加密**: 舊版 question_sn 和 question_timestamp 都用 hash 傳輸；V3 明文

---

## 關鍵差異總結

### 🔴 嚴重問題（影響功能正確性）

1. **影片完成代幣未發放**: `VideoService::updateProgress()` 完成時只處理任務進度，未發放影片觀看代幣（`giveCoin('video', $indicator)`）。舊版 `fnUpdateRateEnd()` 有完整的代幣計算和發放
2. **影片完成節點狀態未更新**: 未呼叫 `UpdateNodeStatus()` 更新 `media_edu:` 狀態
3. **動態評量 map_sn 缺失**: `assessmentSubmit()` 未接收 `map_sn` 參數，導致 `UpdateNodeStatus()` 使用預設值 3（僅對國語正確）
4. **動態評量正確答案暴露**: `getQuestions()` 回傳 `correct_answer`，安全性風險

### 🟡 中度問題（影響完整性）

5. **續看功能未實作**: 舊版 `fnCheckContinueWatch()` + `fnContinueWatch()` 可讓使用者繼續上次未完成的觀看；V3 每次都建新紀錄
6. **turbo（播放速度）未處理**: V3 的 video record 固定傳 null，遺失播放速度追蹤
7. **防作弊「已預習過」邏輯缺失**: 舊版 `getVideoFinish()` 判斷若學生曾完成觀看則免檢查（checkType=2）；V3 只看 finishCount
8. **拖拉/回轉特殊處理缺失**: 舊版 `VIEW_ACTION_EXTRA['cangoback']` 可重設 start_timestamp；V3 無此處理
9. **圖片路徑處理空實作**: `PracticeService::processMediaPath()` 只 return content
10. **影片檢核點提交缺少防作弊**: 舊版 `doVideoActionRecord()` 含完整防作弊；V3 只寫 plus 紀錄

### 🟢 設計改進

11. **Stateless 設計**: V3 使用 token 替代 session，移除 session 依賴
12. **練習題新增單題紀錄**: `prac_single_answer` 表，提供更細粒度的作答紀錄
13. **動態評量 DA 全答錯 bug 修正**: 舊版用 `'practice'` 作為 action_name（應為 `'DA'`），V3 已修正
14. **hash 加密移除**: V3 不再使用 `string2hash()` / `hash2string()` 加密 ID，改用 token 鑑權

### 📋 缺少的 API

| 功能 | 舊版位置 | 建議 |
|------|---------|------|
| 續看檢查 | `prodb_video.php::fnCheckContinueWatch()` | 建議新增 `/ks-learning/video/check-continue` |
| 續看執行 | `prodb_video.php::fnContinueWatch()` | 建議新增 `/ks-learning/video/continue` |
| 影片筆記 | `prodb_note_record.php` | 可依需求新增 |
| 影片討論 | `prodb_discuss.php` | 可依需求新增 |
| 影片管理 (老師端) | `prodb_video_manage_nomap.php` 等 | 不在學生學習流程範圍 |
