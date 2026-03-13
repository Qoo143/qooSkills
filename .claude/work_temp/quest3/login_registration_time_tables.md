# 登入與註冊 — 時間記錄相關資料表

## 總覽

| 資料表 | 用途 | PHP INSERT | PHP UPDATE | DB TRIGGER | 狀態 |
|--------|------|:----------:|:----------:|:----------:|------|
| `user_info` | 註冊時間 | ✓ | ✓ | 無 | **正常** |
| `user_status` | 最後登入快照 | ✓ | ✓ | 無 | **正常**（但只保留最後一次） |
| `user_login_log` | 每次登入歷史 | **無** | ✓（回填） | **無** | **異常 — 無寫入機制** |
| `user_login_log_status` | 登入失敗紀錄 | ✓ | 無 | 無 | **正常** |
| `user_status_log` | 帳號操作紀錄 | **無** | 無 | **無** | **異常 — 無寫入機制** |
| `openid_user_login_record` | OpenID 登入 Log | ✓ | 無 | 無 | **正常** |

---

## 1. user_info（註冊時間）

### 結構（時間相關欄位）

| 欄位 | 型別 | 說明 |
|------|------|------|
| `user_regdate` | varchar | 帳號建立日期 |
| `openid_updatetime` | varchar | OpenID 資訊最後更新時間 |

### 寫入時機與格式

| 欄位 | 寫入時機 | 格式 | PHP 函式 |
|------|---------|------|---------|
| `user_regdate` | 建立帳號 | `date("Y-m-d")` → `2026-03-13` | `exportCreateUserAccountUserInfoByOpenIDUserInfo()` |
| `openid_updatetime` | 建立帳號 / 每次 OpenID 登入同步 | `date("Y-m-d, H:i:s")` → `2026-03-13, 14:30:45` | 同上 |

### 關鍵程式碼位置

| 場景 | 檔案 | 行號 |
|------|------|------|
| 自動建帳（信任學校） | `modules/OpenID/prodb_choiceIdentity.php` | 872, 883 |
| 手動建帳（選擇身分） | `modules/OpenID/prodb_choiceIdentity.php` | 1250, 1261 |
| 註冊帳號 | `modules/OpenID/prodb_choiceIdentity.php` | 1285, 1296 |
| 匯出建帳資料 | `modules/OpenID/function_OpenID.php` | 640, 651 |
| 實際 INSERT | `modules/OpenID/function_addUser.php` | 227-284（`insertUserInfo()`） |
| 綁定舊帳號 | `modules/OpenID/pro_adduser_function.php` | 42-43, 95, 245, 257 |

---

## 2. user_status（最後登入快照）

### 結構（時間相關欄位）

| 欄位 | 型別 | 說明 |
|------|------|------|
| `login_freq` | int | 累計登入次數 |
| `login_from` | varchar(9) | 登入方式（`adl` / `oidc` / `sso`） |
| `starttimestamp` | timestamp | 最後登入時間（時間格式） |
| `lasttimein` | int unsigned | 最後登入時間（Unix 時間戳） |
| `lastip` | char(16) | 最後登入 IP |
| `stoptimestamp` | timestamp | 最後登出時間 |
| `lasttimeout` | int unsigned | 最後登出時間（Unix 時間戳） |
| `last_action_time` | datetime | 最後活動時間 |
| `total_stay` | int unsigned | 總停留時間 |

### 寫入時機

**建立帳號時（INSERT）：** 只寫入 `user_id`, `access_level`, `due_date`，**不設定** 任何時間欄位。

**每次登入時（UPDATE）：** 更新 `login_freq`（+1）、`starttimestamp`、`lasttimein`、`lastip`、`login_from`。

### 問題

每次 UPDATE 會**覆蓋**上一次的值，無法查詢登入歷史，只知道「最後一次」。

### 所有登入入口

| 登入方式 | 檔案 | 行號 | starttimestamp 格式 | lasttimein 格式 |
|---------|------|------|-------------------|----------------|
| 一般帳密 (ADL) | `login.php` | 418-428 | `date('Y-m-d H:i:s')` | `time()`（Unix） |
| OpenID (OIDC) | `login.php`（經 `loginWithOpenID` 導向） | 同上 | 同上 | 同上 |
| 教育雲 SSO | `login_cloud.php` | 288-298 | `CURRENT_TIMESTAMP` | `UNIX_TIMESTAMP()` |
| COOC | `roles_cooc.php` | 300-304 | `CURRENT_TIMESTAMP` | `UNIX_TIMESTAMP()` |
| 學習雲 | `modules/learningcloud/prodb_login.php` | 17-22 | `CURRENT_TIMESTAMP` | `UNIX_TIMESTAMP()` |
| 快速登入 (LAB) | `quickLogin4LAB.php` | 34-39 | `date('Y-m-d H:i:s')` | `time()` |
| Auth 套件 | `classes/PEAR/Auth.php` | 1588-1597 | `date('Y-m-d H:i:s')` | `time()` |
| quickLogin session | `modules_new.php` | 158-167 | `date('Y-m-d H:i:s')` | `time()` |
| loginAuth | `loginAuth.php` | 178-181 | `CURRENT_TIMESTAMP` | `UNIX_TIMESTAMP()` |
| 舊帳號綁定 | `modules/OpenID/prodb_insert_userinfo.php` | 124 | `:starttime` 參數 | — |

---

## 3. user_login_log（登入歷史明細）⚠️ 無寫入機制

### 結構

```sql
CREATE TABLE user_login_log (
  sn             bigint UNSIGNED NOT NULL AUTO_INCREMENT,
  user_id        varchar(60),
  organization_id varchar(10),
  loginstamp     timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,  -- 登入時間
  active_time    int DEFAULT NULL,                               -- 停留時間(秒)
  login_from     varchar(9),                                     -- 登入方式
  ip_address     char(16) NOT NULL DEFAULT '0.0.0.0',
  update_time    timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

### 現況

| 操作 | 有無 | 位置 | 說明 |
|------|------|------|------|
| **INSERT** | **無** | — | PHP 無、DB TRIGGER 無、Stored Procedure 未確認 |
| **UPDATE** | 有 | `include/function_studentDailyReport.php:1063` | 排程回填 `active_time`（停留時間） |
| **SELECT** | 有 | `php_cli/rs_personalDailyReport.php` 等多處 | 報表統計：登入次數、停留時間 |

### 誰在讀這張表

| 檔案 | 用途 |
|------|------|
| `include/getTime_function.php:240` | 計算學生停留時間 |
| `include/function_studentDailyReport.php:314, 493` | 每日報表：登入時間、停留時間 |
| `php_cli/rs_personalDailyReport.php:149-150` | 每日報表：登入次數、active_time |
| `php_cli/rs_personalMonthlyReport.php:781, 798` | 每月報表 |
| `php_cli/rs_Yilan_Report.php:101` | 宜蘭報表 |
| `php_cli/rs_mitac.php:597` | 神達報表 |
| `classes/report_learningeffect_class.php:967, 3330` | 學習成效報表 |

### 問題

**這張表預期每次登入 INSERT 一筆**，但目前沒有任何已知的寫入機制。報表依賴此表，若無資料則報表數據不正確。

---

## 4. user_login_log_status（登入失敗紀錄）✓ 正常

### 結構

```sql
CREATE TABLE user_login_log_status (
  sn             bigint UNSIGNED NOT NULL AUTO_INCREMENT,
  user_id        varchar(60) NOT NULL,
  loginstamp     timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  login_status   int DEFAULT NULL,     -- 1=成功, 0=失敗
  ip_address     char(16) NOT NULL DEFAULT '0.0.0.0'
);
```

### 寫入時機

| 時機 | 檔案 | 行號 |
|------|------|------|
| 帳密登入失敗 | `classes/PEAR/Auth/Frontend/Html.php` | 117（`login_status=0`） |
| 被鎖定拒絕登入 | `classes/PEAR/Auth/Frontend/Html.php` | 352（`status=0`） |

### 讀取時機

| 時機 | 檔案 | 行號 | 說明 |
|------|------|------|------|
| 登入前檢查 | `classes/PEAR/Auth.php` | 616 | 15 分鐘內連續失敗 5 次 → 鎖定帳號 |

---

## 5. user_status_log（帳號操作紀錄）⚠️ 無寫入機制

### 結構

```sql
CREATE TABLE user_status_log (
  log_sn          bigint UNSIGNED NOT NULL AUTO_INCREMENT,
  organization_id varchar(10),
  user_id         varchar(60) NOT NULL,
  user_account    varchar(60),
  action          enum('Disable','Enable','Delete','Register','Login'),
  update_id       varchar(60),          -- 操作者（誰建的/誰停的）
  record_time     datetime NOT NULL DEFAULT CURRENT_TIMESTAMP
) PARTITION BY HASH (month(record_time)) PARTITIONS 12;
```

### 現況

| 操作 | 有無 | 說明 |
|------|------|------|
| **INSERT** | **無** | PHP 無、DB TRIGGER 無 |
| **SELECT** | 有 | 查詢用 |

### 誰在讀這張表

| 檔案 | 用途 |
|------|------|
| `classes/UserStatusLog.php:14-27` | 查「某管理員建了哪些帳號」（`action='Register'`） |
| `modules/UserManage/UserMFunction.php:709, 2484, 3136` | 帳號管理：查建立者、年班 |
| `php_cli/rs_deluser.php:54` | 刪除重複帳號：查 Register 紀錄 |
| `php_cli/rs_learningeffect_detail.php:79, 194...` | 學習成效報表：排除已停用帳號 |

### 問題

**五種 action 都沒有 INSERT 機制。** 無法記錄誰建了帳號、誰停用了帳號、誰登入了。

---

## 6. openid_user_login_record（OpenID 登入 Log）✓ 正常

### 結構（時間相關）

| 欄位 | 說明 |
|------|------|
| `login_time` | 登入時間（自動填入） |
| `error_status` | 1=資料正確, 2=有錯誤 |

### 寫入時機

| 時機 | 檔案 | 說明 |
|------|------|------|
| 時機 1: GET_OPENID_USER_DATA | `prodb_choiceIdentity.php:88-205` | 取得 OpenID 資料後記錄 |
| 時機 2: LOGIN_SELECTED_IDENTITY | `prodb_choiceIdentity.php:470+` | 選擇身分登入時記錄 |

### 子表

| 表 | 關係 | 內容 |
|---|---|---|
| `openid_user_info` | 1:1 | 使用者資料快照 |
| `openid_class_info` | 1:N | 班級資料 |
| `openid_titles_info` | 1:N | 身分資料 |
| `openid_user_login_title` | 1:0..1 | 選擇的登入身分（僅時機 2） |
| `openid_user_data_check_status` | 1:N | 資料驗證錯誤 |
| `openid_api_status_record` | 1:5 | API 狀態（5 個 API） |

---

## 問題彙整

### 缺失的寫入機制

```
註冊流程:
  ✓ user_info (user_regdate, openid_updatetime)
  ✓ user_status (INSERT: user_id, access_level, due_date)
  ✗ user_status_log (action='Register') ← 無 INSERT
  ✗ user_login_log ← 無 INSERT（首次登入也沒有紀錄）

登入流程:
  ✓ user_status (UPDATE: starttimestamp, lasttimein, login_freq)
  ✓ openid_user_login_record（僅 OpenID 登入）
  ✗ user_login_log ← 無 INSERT（所有登入方式都沒有）
  ✗ user_status_log (action='Login') ← 無 INSERT

登入失敗:
  ✓ user_login_log_status (login_status=0)

帳號停用/啟用/刪除:
  ✗ user_status_log (action='Disable'/'Enable'/'Delete') ← 無 INSERT
```

### 影響

1. **user_login_log 無資料** → 每日/每月報表的登入次數和停留時間數據不正確
2. **user_status_log 無資料** → 無法追蹤帳號操作歷史（誰建的、誰停的）
3. **user_status 只保留最後一次** → 歷史登入時間全部被覆蓋

### 待確認

- `user_login_log` 裡確實有新帳號資料，但 PHP / TRIGGER / Stored Procedure 都找不到 INSERT 來源
- 需確認：是否有不在此 repo 的外部服務或 DB proxy 在寫入？
- 建議執行：`SELECT ROUTINE_NAME FROM information_schema.ROUTINES WHERE ROUTINE_SCHEMA = DATABASE();` 和 `SHOW EVENTS;`
