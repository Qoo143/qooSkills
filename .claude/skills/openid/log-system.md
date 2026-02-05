# Log 記錄機制

## 記錄時機

OpenID 登入過程會在 2 個時機點記錄 Log:

### 時機 1: GET_OPENID_USER_DATA

- 觸發: 完成 OAuth2 授權,取得 access_token 後呼叫 5 個 API
- 記錄內容:
  - 5 個 API 的 HTTP 狀態碼與執行時間
  - OpenID 回傳的原始資料 (使用者、班級、身分)
  - 60+ 種資料驗證結果
- 特性: 不包含使用者選擇的登入身分
- 程式碼位置: `prodb_choiceIdentity.php:88-205`

### 時機 2: LOGIN_SELECTED_IDENTITY

- 觸發: 使用者選擇身分和學校後點擊登入
- 記錄內容:
  - 從 Session 取出的 API 狀態 (不重新呼叫 API)
  - OpenID 資料 (與時機 1 相同)
  - 使用者選擇的登入身分 (差異點)
- 特性: 多記錄了 `openid_user_login_title` 表

---

## Log 記錄流程

程式碼位置: `prodb_choiceIdentity.php:106-174`

```php
// 1. 建立 HTTP 狀態記錄器與計時器
$oHttpStatusRecorder = new OpenIDCommonTools\HttpStatusRecorder();
$oTimer = new OpenIDCommonTools\Timer();

// 2. 呼叫 API 前啟動計時
$oTimer->start();
$arrGetUserinfoResult = $oOpenIDInfoGetter->requestOpenIDData(
    $sUserInfoUrl, $_SESSION['access_token'], true
);
$oUserInfoAPIExecutionTime = $oTimer->getTimeObject(
    OpenIDCommonTools\Timer::GET_MILLI_SECOND
);

// 3. 記錄到 HttpStatusRecorder (準備寫入資料庫)
$oHttpStatusRecorder->insert(
    OpenIDCommonTools\OpenIDAPIConfigSet::API_OPENID_USER_INFO,  // API 類型
    $arrGetUserinfoResult['http_status'],                         // HTTP 狀態碼
    $oUserInfoAPIExecutionTime                                    // 執行時間 (毫秒)
);

// 4. 同時記錄到 Session (供時機 2 使用)
OpenIDSessionKeeper\OpenIDSessionAPIStatusKeeper::keepRequestInfo(
    OpenIDCommonTools\OpenIDAPIConfigSet::API_OPENID_USER_INFO,
    $arrGetUserinfoResult['http_status'],
    $oUserInfoAPIExecutionTime
);

// 5. 對 5 個 API 重複上述流程
// UserInfo, EduInfo, Relation, GetGuidToken, GetGUID

// 6. 建立 OpenIDUserInfo 物件 (自動驗證)
$oOpenIDUserInfo = new User\OpenIDInfo\OpenIDUserInfo($userinfo, $eduinfo);
$oOpenIDUserInfo->initializeClassInfo($eduinfo, $relation);

// 7. 取得驗證結果
$arrOpenIDValidationList = $oOpenIDUserInfo->exportValidationList();

// 8. 批次寫入資料庫
$bInsertResult = User\OpenIDInfo\OpenIDUserInfoLogger::insertRecord(
    $arrOpenIDUserInfo,                        // 使用者資料
    $arrOpenIDClassList['class_list'],         // 班級清單
    $arrOpenIDValidationList,                  // 驗證錯誤清單
    $oHttpStatusRecorder->exportStatusArray()  // API 狀態清單 (5 筆)
);
```

---

## OpenIDUserInfoLogger

程式碼位置: `classes/OpenID/openID_user_Info.php:607-711`

`insertRecord()` 方法內部依序寫入以下資料表:

```php
class OpenIDUserInfoLogger implements RecordOpenIDLoginDataInterface
{
    const OPENID_DATA_CORRECT = 1;  // 資料正確
    const OPENID_DATA_ERROR = 2;    // 有錯誤

    public static function insertRecord($arrUserInfo, $arrClassList,
        $arrCheckOpenIDDataStatusCodeList, $arrAPIInfoList,
        $bInsertLoginTitle = false)
    {
        // 1. 判斷 error_status
        $iLoginDataSatus = empty($arrCheckOpenIDDataStatusCodeList)
            ? self::OPENID_DATA_CORRECT    // 1
            : self::OPENID_DATA_ERROR;     // 2

        // 2. 寫入主記錄 (openid_user_login_record),取得 sn
        $iInsertSN = self::insertOpenIDLoginRecord($arrUserInfo['sub'], $iLoginDataSatus);

        // 3. 依序寫入子表 (各自獨立,不因前一個失敗而中斷)
        $bInsertLoginStatusResult = self::insertOpenIDLogintStatus($arrCheckOpenIDDataStatusCodeList, $iInsertSN);
        $bInsertUserInfoResult = self::insertOpenIDUserInfo($arrUserInfo, $iInsertSN);
        $bInsertOpendIDClassInfoResult = self::insertOpenIDClassInfoRecord($arrClassList, $iInsertSN);
        $bInsertOpenIDTitlesResult = self::insertOpenIDTitlesInfo($arrUserInfo['titles_list'], $iInsertSN);
        $bInsertAPIStatusResult = self::insertAPIStatusRecord($arrAPIInfoList, $iInsertSN);

        // 4. 時機 2 才寫入登入身分
        if ($bInsertLoginTitle) {
            $bInserSelectedTitlesResult = self::insertLogginedTitles(
                $arrUserInfo['login_titles']['title'],
                $arrUserInfo['login_titles']['organization_id'],
                $iInsertSN
            );
        }

        return [$bInsertResult, $iInsertSN];
    }
}
```

---

## Log 資料表結構

### 主記錄表

#### openid_user_login_record

- 記錄內容: 登入主記錄 (主鍵表)
- 記錄時機: 每次選擇身分登入 (時機 1 和時機 2 都會記錄)
- 關鍵欄位: `sn` (PK, AUTO_INCREMENT), `sub` (OpenID_sub), `access_token`, `error_status`, `login_time`

### 子記錄表

所有子表透過 `openid_user_login_record_sn` (FK) 關聯到主記錄。

#### openid_user_info

- 記錄內容: OpenID UserInfo API 回傳的使用者資料
- 記錄時機: 每次登入
- 關鍵欄位: `name`, `guid`, `guid_token`, `prefferd_name`, `school_id`, `mentor_grade`, `mentor_class`, `email`
- 特殊用途: 記錄 OpenID 傳來的原始值,可觀察資料是否含中文或格式異常

#### openid_class_info

- 記錄內容: OpenID EduInfo API 回傳的班級資料
- 記錄時機: 每次登入
- 特性: 一個人可能有多筆 (教師有多個任課班級)
- 關鍵欄位: `year`, `semester`, `grade`, `class`, `organization_id`

#### openid_titles_info

- 記錄內容: OpenID EduInfo API 回傳的身分資料
- 記錄時機: 每次登入
- 特性: 一個人可能有多筆 (多重身分或多校身分)
- 關鍵欄位: `organization_id`, `title`

#### openid_user_login_title

- 記錄內容: 使用者選擇的登入身分
- 記錄時機: 僅時機 2 記錄 (`$bInsertLoginTitle = true`)
- 用途: 分析使用者身分選擇習慣
- 關鍵欄位: `school_id`, `titles`

#### openid_user_data_check_status

- 記錄內容: 資料驗證錯誤記錄
- 記錄時機: OpenID 資料驗證時
- 特性: 一次登入可能有多筆錯誤記錄
- 關鍵欄位: `check_status_sn`

#### openid_api_status_record

- 記錄內容: API 請求狀態 (HTTP 狀態碼和回應時間)
- 記錄時機: 每次 API 呼叫 (5 個 API = 5 筆)
- 用途: 監控 OpenID API 效能和可用性
- 關鍵欄位: `api_type`, `http_status`, `response_time_ms`

---

## error_status 判斷邏輯

```php
$iLoginDataSatus = empty($arrCheckOpenIDDataStatusCodeList)
    ? self::OPENID_DATA_CORRECT    // 1 = 資料正確
    : self::OPENID_DATA_ERROR;     // 2 = 有錯誤
```

- `error_status = 1`: OpenID 資料完全正確,無任何驗證錯誤
- `error_status = 2`: 有任何驗證錯誤 (即使只有 1 個錯誤)

---

## 驗證錯誤參照表

#### openid_user_data_check_status_discription

- 用途: 錯誤代碼說明表 (靜態參照表)
- 定義: 60+ 種驗證錯誤類型和說明訊息
- 操作: 只查詢不新增
- 關鍵欄位: `sn`, `description`

驗證項目包含:

| 驗證類別 | 錯誤範例 |
|:---------|:---------|
| 姓名格式 | 空值、含數字、特殊字元 |
| 年班格式 | 空值、非數字、負數、零 |
| 座號格式 | 空值、非數字、負數、零 |
| 身分證格式 | 雜湊值空值或格式錯誤 |

---

## Log 資料表與 API 資料量對照

| 資料表 | 每次登入資料量 | 說明 |
|:-------|:-------------|:-----|
| openid_user_login_record | 1 筆 | 主記錄 |
| openid_user_info | 1 筆 | 使用者資料快照 |
| openid_class_info | 多筆 | 班級數量 (教師可能很多) |
| openid_titles_info | 多筆 | 身分數量 |
| openid_api_status_record | 5 筆 | 5 個 API 各 1 筆 |
| openid_user_data_check_status | 0~多筆 | 有錯誤時才記錄 |
| openid_user_login_title | 0~1 筆 | 僅時機 2 記錄 |

---

## Session 備份機制

API 狀態同時記錄到 Session,供時機 2 使用 (不需重新呼叫 API):

```php
// 記錄到 Session
OpenIDSessionKeeper\OpenIDSessionAPIStatusKeeper::keepRequestInfo(
    $apiType, $httpStatus, $executionTime
);

// 時機 2 取出
$arrAPIStatusArray = OpenIDSessionKeeper\OpenIDSessionAPIStatusKeeper::exportAPIStatusArray();
```

## 相關文件

- [登入流程](./login-flow.md) - Log 在流程中的記錄時機
- [錯誤處理](./error-handling.md) - 驗證錯誤的處理方式
- [資料庫](./database.md) - 完整資料表結構
