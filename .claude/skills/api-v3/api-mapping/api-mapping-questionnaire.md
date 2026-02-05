# API-V3 對照表：Questionnaire Controller

## Controller 資訊
- **檔案**: `ADLAPI/v3/App/Controller/QuestionnaireController.php`
- **Service**: `ADLAPI/v3/App/Service/QuestionnaireService.php`
- **DAO**: `ADLAPI/v3/App/Dao/QuestionnaireDao.php`

---

## 端點對照總覽

| # | V3 端點 | HTTP | 舊檔案 | 舊動作 | 狀態 |
|---|---------|------|--------|--------|------|
| 1 | `/questionnaire/types` | POST | (無對應，前端硬編碼) | — | ✅ 新設計 |
| 2 | `/questionnaire/detail` | POST | `mission_google_master` 資料表 | 前端讀取 form_title | ⚠️ 嚴重不足 |
| 3 | `/questionnaire/status` | POST | `5c.php` 前端直接查 `mission_stud_record` | — | ⚠️ 有差異 |
| 4 | `/questionnaire/history` | POST | (無直接對應) | — | ✅ 新設計 |
| — | **缺少: submit** | — | `prodb_5c.php`, `prodb_ct.php`, `prodb_network109.php`, `prodb_srl.php`, `prodb_open_5c.php`, `prodb_open_ntcu.php` | INSERT mission_google_form + 代幣 + 任務更新 | 🔴 完全缺失 |
| — | **缺少: report** | — | `prodb_open_improvement.php` | getSemeList / getlist / getReportData | 🔴 完全缺失 |

---

## 1. 問卷類型列表 `/questionnaire/types`

### V3 實作
- `QuestionnaireService::QUESTIONNAIRE_TYPES` 常數定義 6 種類型
- 回傳 code, name, description

### 問卷類型對照
| V3 code | 對應舊檔 | 舊 form_type | 說明 |
|---------|---------|-------------|------|
| `5c` | `prodb_5c.php` | POST 傳入 | 5C 核心素養（任務版） |
| `CT109` | `prodb_ct.php` | POST 傳入 | 運算思維 |
| `network109` | `prodb_network109.php` | POST 傳入 | 網路使用 |
| `ntcu` | `prodb_open_ntcu.php` | POST 傳入 | NTCU 師培問卷 |
| `improvement_st` | `prodb_open_improvement.php` | 10 (GOOGLE_FORM_IMPROVEMENT_ST) | 學生高中職精進計畫 |
| `improvement_th` | `prodb_open_improvement.php` | 11 (GOOGLE_FORM_IMPROVEMENT_TH) | 教師高中職精進計畫 |

### 缺失的問卷類型
| 舊版 form_type | 舊檔案 | 說明 |
|---------------|--------|------|
| `srl` / 12 (GOOGLE_FORM_SRL) | `prodb_srl.php` | SRL 問卷（舊版） |
| 14 (GOOGLE_FORM_NEW_SRL) | `prodb_srl.php` | SRL 問卷（新版） |
| 15 (GOOGLE_FORM_114_SRL) | `prodb_srl.php` | SRL 114 學年 |
| 16 (GOOGLE_FORM_115_SRL) | `prodb_open_srl.php` | SRL 115 學年（最新） |
| `open_5c` | `prodb_open_5c.php` | 5C 開放版（免任務） |

---

## 2. 問卷詳情 `/questionnaire/detail`

### 參數對照
| V3 參數 | 舊參數 | 說明 |
|---------|--------|------|
| `type` (required) | 前端硬編碼 / `mission_google_master.sn` | 問卷類型 |

### 邏輯對照

#### V3 實作
- `getQuestions($type)` 為**硬編碼**的題目結構
- 5c：僅 2 個 section（問題解決 3 題 + 創造思考 2 題），實際問卷遠不止此
- CT109：僅 1 個 section 2 題
- 其他類型（network109, ntcu, improvement_st, improvement_th）：回傳**空陣列**

#### 舊版實作
- 題目存在 `mission_google_master` 資料表的 `form_title` 欄位
- 以 `@XX@` (`_SPLIT_SYMBOL`) 分隔各題目文字
- 前端動態渲染問卷 UI（radio button 群組）
- `prodb_open_improvement.php:getStudentInprovementItemData()` 查詢範例：
  ```sql
  SELECT MGM.form_title FROM mission_google_master MGM WHERE sn = :sn
  ```

### 缺失邏輯
- ⚠️ **題目應從 DB 讀取**: 應查詢 `mission_google_master` 而非硬編碼
- ⚠️ **題目不完整**: 5c 實際約 30+ 題（5 個 section 各 6+ 題），V3 只有 5 題
- ⚠️ **問卷結構缺失**: 選項格式（Likert 5 點量表）、題號對應、section 分組規則均不完整
- ⚠️ **improvement 問卷有 69 題**: 含 64 題同意度 + 2 題平台使用 + 1 題裝置時間 + 2 題使用頻率，結構完全不同

---

## 3. 問卷填寫狀態 `/questionnaire/status`

### 參數對照
| V3 參數 | 舊參數 | 說明 |
|---------|--------|------|
| `type` (required) | form_type | 問卷類型 |
| `mission_sn` (optional) | mission_sn (GET 'm' 參數) | 任務 SN |

### 邏輯對照
| 功能 | V3 | 舊 | 差異 |
|------|------|------|------|
| 查詢來源 | `mission_google_form` | `mission_stud_record` (5c.php) | ⚠️ 查詢資料表不同 |
| 判斷依據 | record !== null | `is_all_fin` 欄位 | ⚠️ 舊版判斷任務整體完成狀態 |
| mission_sn 處理 | 直接用 | `hash2string()` 解碼 | ✅ V3 不需加密 |

### V3 DAO SQL
```sql
SELECT ... FROM mission_google_form
WHERE user_id = :user_id
AND (mission_sn = :mission_sn OR (mission_sn IS NULL AND :mission_sn2 = 0))
AND form_type = :form_type
ORDER BY sn DESC LIMIT 1
```

### 缺失邏輯
- ⚠️ **舊版用 mission_stud_record**: 前端 `5c.php` 查 `mission_stud_record.is_all_fin` 來判斷任務是否已全部完成（不只問卷），V3 只查 `mission_google_form` 是否有紀錄
- ⚠️ **closeTime 未實作**: 舊版多個問卷有 `$closeTime` 截止日期檢查（如 `2025-07-11 18:00:00`、`2026-01-30 18:00:00`），V3 未判斷問卷是否開放

---

## 4. 問卷歷史 `/questionnaire/history`

### 參數對照
| V3 參數 | 舊參數 | 說明 |
|---------|--------|------|
| `type` (optional) | — | 問卷類型，不傳查全部 |

### 邏輯對照
- V3 查詢 `mission_google_form` 按 `sn DESC` 排序
- 回傳：sn, form_type, mission_sn, submit_time
- **舊版無直接對應功能**（為 V3 新增端點）

### 缺失邏輯
- ⚠️ **未回傳 content**: 歷史紀錄未包含實際作答內容，前端若需回顧答案會缺少資料
- ⚠️ **未分頁**: 若使用者填寫大量問卷，回傳量可能過大

---

## 關鍵差異總結

### 🔴 嚴重問題

1. **完全缺少提交端點 (submit)**
   - 舊版每個問卷類型各有獨立 prodb_ 提交檔案
   - 共同邏輯：INSERT INTO `mission_google_form` (form_type, seme, user_id, city, org_name, grade, class, stu_name, stu_sex, content, start_time, mission_sn)
   - V3 **無任何寫入操作**，問卷系統僅能讀取，無法作答
   - DAO 也僅注入 `$dbh_slave`（唯讀），無寫入 DB 連線

2. **提交時的代幣與任務更新缺失**
   - 舊版提交流程（以 prodb_5c.php 為例）：
     1. INSERT 問卷紀錄到 mission_google_form
     2. 學生角色 → `update_mission_record($mission_sn, "questionaire", $user_id, 'viewform')` 更新任務進度
     3. 若任務未全部完成 → `giveCoin($user_id, 'questionaire', $mission_sn)` 發放代幣
     4. `caculate_mission_reward($mission_info)` (mission_type = 2)
     5. `caculate_extra_reward($mission_sn)` 額外獎勵
     6. `give_role_coin($student_get_coins, $extra_coin, $mission_sn)`
   - V3 完全沒有此流程

3. **問卷題目硬編碼且嚴重不完整**
   - 舊版從 `mission_google_master.form_title` 動態讀取
   - V3 `getQuestions()` 只有 5c（5題/30+題）和 CT109（2題）的簡化版
   - network109, ntcu, improvement_st, improvement_th 回傳空陣列

### 🟡 中度問題

4. **closeTime 問卷開放期限未實作**: 舊版幾乎每個問卷都有截止時間檢查，V3 無此機制

5. **seme（學期）欄位缺失**: 舊版用 `getYearSeme()` 記錄學期，V3 提交時無此欄位

6. **使用者資訊不足**: 舊版提交時記錄 city, org_name, grade, class, stu_name, stu_sex（從 session 取），V3 若要實作 submit 需考慮這些欄位的來源（token 中可能沒有完整使用者資訊）

7. **SRL 問卷類型未列入**: 舊版有 4 個 SRL 相關 form_type（12, 14, 15, 16），V3 types 常數中未定義

8. **content 格式未定義**: 舊版用 `@XX@` 分隔的字串存答案（如 "非常同意@XX@同意@XX@..."），V3 未定義答案格式

### 🟢 設計改進

9. **types 統一端點**: 舊版問卷類型散落在各檔案，V3 集中定義

10. **status 檢查**: 舊版在前端查詢，V3 提供獨立 API

11. **history 歷史查詢**: 完全新功能，舊版無對應

### 📋 舊版功能未實作

#### 報表與統計 (prodb_open_improvement.php)
| 舊功能 | action | 說明 |
|--------|--------|------|
| 學期列表 | `getSemeList` | 取得有問卷資料的學期清單 |
| 檢查學生資料 | `checkStudentData` | 檢查特定學期是否有學生資料 |
| 作答清單 | `getlist` | 取得學校的學生/教師問卷作答記錄，按班級分組 |
| 統計報表 | `getReportData` | 產生 Excel 報表（全國/全校/班級統計），含同意度、平台使用、裝置時間、使用頻率 |

#### SRL 報表 (prodb_open_srl.php)
| 舊功能 | action | 說明 |
|--------|--------|------|
| 學期列表 | `getSemeList` | 取得有 SRL 問卷的學期清單 |
| 作答清單 | `getlist` | 取得學校的 SRL 作答記錄 |
| 作答資料 | `getdata` | 取得問卷題目與學生作答明細 |
| 資料修正 | `updateUserData` | 修正問卷中的年班資料（與 seme_student 同步） |

> 注意：報表與統計功能屬於教師/管理端使用場景，如需實作建議另建 `QuestionnaireReportController`

---

## 資料表結構參考

### mission_google_form（問卷作答紀錄）
| 欄位 | 說明 | 舊版來源 |
|------|------|---------|
| sn | 自增主鍵 | — |
| form_type | 問卷類型代碼 | POST['form_type'] |
| seme | 學期（如 1141, 1142） | getYearSeme() |
| user_id | 使用者 ID | SESSION['user_id'] |
| city | 縣市 | SESSION / POST |
| org_name | 學校名稱 | SESSION / POST |
| grade | 年級 | SESSION / POST |
| class | 班級 | SESSION / POST |
| stu_name | 姓名 | SESSION / POST |
| stu_sex | 性別 | SESSION / POST |
| content | 作答內容（@XX@ 分隔） | POST['content'] |
| start_time | 提交時間 | POST['start_time'] |
| mission_sn | 任務 SN | POST['mission_sn'] |

### mission_google_master（問卷主檔）
| 欄位 | 說明 |
|------|------|
| sn | 主鍵（對應 form_type） |
| form_name | 問卷名稱 |
| form_title | 題目列表（@XX@ 分隔） |
