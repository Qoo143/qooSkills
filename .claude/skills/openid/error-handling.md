# 錯誤處理

## 錯誤處理總覽

| 錯誤類型 | 嚴重程度 | 處理方式 | 記錄位置 |
|:---------|:---------|:---------|:---------|
| API 呼叫失敗 | 高 | 阻止登入,顯示錯誤訊息 | openid_api_status_record |
| 資料驗證失敗 | 高 | 阻止登入,顯示錯誤代碼說明 | openid_user_data_check_status |
| 停用/轉出帳號 | 高 | 阻止登入,提示聯絡管理員 | 無 |
| 快速登入同步失敗 | 低 | 記錄錯誤,繼續登入 | error_log |
| 一般登入同步失敗 | 高 | 阻止登入,提示聯絡客服 | 無 |
| Transaction 失敗 | 高 | 回滾所有操作,阻止登入 | 無 |
| Session 遺失 | 高 | 提示重新登入 | 無 |

---

## API 呼叫錯誤

錯誤類型:
- HTTP 狀態碼非 200
- API 回應格式錯誤
- 網路連線逾時

處理方式:

```php
// 所有 API 都會記錄 HTTP 狀態碼和執行時間到 openid_api_status_record
$oHttpStatusRecorder->insert(
    OpenIDAPIConfigSet::API_OPENID_USER_INFO,
    $arrGetUserinfoResult['http_status'],
    $oUserInfoAPIExecutionTime
);

// HTTP 狀態碼非 200 時,前端顯示錯誤訊息
if ($arrGetUserinfoResult['http_status'] != 200) {
    // 使用者看到「無法取得 OpenID 資料,請稍後再試」
}
```

---

## 資料驗證錯誤

程式碼位置: `prodb_choiceIdentity.php:181-195`

驗證項目 (60+ 種錯誤代碼):

| 驗證類別 | 檢查內容 |
|:---------|:---------|
| 姓名格式 | 空值、含數字、特殊字元 |
| 年班格式 | 空值、非數字、負數、零 |
| 座號格式 | 空值、非數字、負數、零 |
| 身分證格式 | 雜湊值空值或格式錯誤 |

處理方式:

```php
// OpenIDUserInfo 物件建立時自動驗證
$oOpenIDUserInfo = new OpenIDUserInfo($userinfo, $eduinfo);
$arrValidationList = $oOpenIDUserInfo->exportValidationList();

// 過濾出 ERROR 等級錯誤
$arrErrorList = OpenIDUserInfoValidation::filter(
    $arrValidationList,
    OpenIDUserInfoValidation::TYPE_ERROR
);

if (!empty($arrErrorList)) {
    // 取得錯誤狀態的說明訊息
    $arrErrorInfoList = OpenIDUserInfoValidation::getValidationInfo($arrErrorList);

    // 阻止登入,顯示錯誤代碼給前端
    echo json_encode([
        'status' => RESPONSE_GET_USERINFO_FAIL,
        'error_data' => $arrErrorInfoList
    ]);
    die();
}
```

記錄方式:
- 所有驗證結果 (包含 WARNING) 都記錄到 `openid_user_data_check_status`
- `error_status` 欄位標記: 1=完全正確, 2=有錯誤

---

## 帳號查詢錯誤

| 情境 | 處理方式 | 使用者訊息 |
|:-----|:---------|:----------|
| 停用帳號 | 阻止登入 | 「目前查看系統您的帳號已經停用,請聯絡校管或客服人員啟用帳號」 |
| 轉出帳號 | 阻止登入 | 「目前查看系統您的因材網帳號已經轉出,請聯絡校管或因材網客服人員確認帳號狀態」 |
| 多筆帳號 | 顯示選單 | 前端顯示帳號選擇介面 --> SET_ENABLED_ACCOUNT |
| 同名學生 | 顯示綁定選項 | 前端顯示「創建新帳號」或「綁定舊帳號」選項 |
| 查無帳號 | 檢查自動創建 | 符合條件 --> 自動創建 / 不符合 --> 手動註冊介面 |

程式碼位置:

```php
// 停用帳號 - prodb_choiceIdentity.php:775-778
if (!$bIsAccountEnabled) {
    responseFailure("目前查看系統您的帳號已經停用,請聯絡校管或客服人員啟用帳號");
    die();
}

// 轉出帳號 - prodb_choiceIdentity.php:800-803
if ($bIsAccountExportSchool) {
    responseFailure("目前查看系統您的因材網帳號已經轉出,請聯絡校管或因材網客服人員確認帳號狀態");
    die();
}
```

---

## 資料同步錯誤

### 快速登入同步錯誤

```php
try {
    updateStudentSemesterInfo(...);
} catch (Exception $e) {
    // 只記錄錯誤,不阻止登入
    error_log("Quick Login - Student sync failed: " . $e->getMessage());
}
// 繼續執行登入流程
```

### 一般登入同步錯誤

```php
if (!updateStudentSemesterInfo(...)) {
    // 阻止登入,顯示錯誤訊息
    responseFailure("學生年班資料與OpenID資料比對有差異,自動更新年班失敗,請聯絡因材網客服");
}
```

設計理由:
- 快速登入: `openid_data` 已驗證一致,優先使用者體驗
- 一般登入: 資料可能不一致,需嚴格檢查

---

## Transaction 錯誤

### 教師同步 Transaction

```php
try {
    $dbh->beginTransaction();
    // 執行多次 replaceSemeTeacherSubject()
    $bReplaceSubjectResult = true;
} catch (Exception $e) {
    $bReplaceSubjectResult = false;
    $sRelaceSubjectErrorMessage = $e->getMessage();
} finally {
    if ($bReplaceSubjectResult) {
        $dbh->commit();
    } else {
        $dbh->rollback();
        // 快速登入: error_log 繼續登入
        // 一般登入: responseFailure 阻止登入
    }
}
```

### 帳號創建 Transaction

```php
try {
    $dbh->beginTransaction();
    insertUserInfo(...);
    insertUserStatus(...);
    insertSemeStudent(...);  // 學生
    $dbh->commit();
} catch (Exception $e) {
    $dbh->rollBack();
    // 創建失敗,阻止登入
    responseFailure($e->getMessage());
}
```

---

## Session 遺失錯誤

```php
if (!isset($_SESSION['openIDchc'])) {
    responseFailure(HIINT_IS_LOG_OUT);  // "目前系統已登出,請重新登入"
}
```

常見原因:
- Session 逾時 (使用者停留太久)
- 多分頁登入衝突
- 瀏覽器 Cookie 被清除

---

## openid_data 寫入失敗

```php
// prodb_choiceIdentity.php:938-949
try {
    $update = $dbh->prepare(
        "UPDATE user_info SET openid_data = :openid_data WHERE user_id = :user_id"
    );
    $update->execute();
} catch (PDOException $e) {
    // 只記錄錯誤,不阻止登入流程
    error_log("PDO Error: " . $e->getMessage());
}
```

openid_data 寫入失敗不阻止登入,但會影響下次快速登入的比對結果。

---

## 非信任學校年班不一致

```php
// prodb_choiceIdentity.php:951-957
if (!$bOpenIdTrust && $bIsClassInfoDiffrence && isStudentRole($sUserLoginTitle)) {
    setSessionModifiedClassOpenIDUserID($arrLoginAccountInfo['user_id']);
    setSessionOpenIDLoginMode(LOGIN_MODE_MODIFIED_CLASSINFO);
    echo json_encode(['status' => RESPONSE_CLASSINFO_COMPARE_DIFFRENT]);
    die();
}
```

- 不視為錯誤,而是引導使用者手動確認年班
- 前端顯示年班選擇介面
- 使用者確認後觸發 SET_USER_CLASS_INFO

---

## responseFailure 函式

定義於 `function_OpenID.php:169-174`:

```php
function responseFailure($sMsg) {
    echo json_encode([
        'status' => RESPONSE_LOGIN_FAILURE,
        'message' => $sMsg
    ]);
    die();
}
```

所有阻止登入的錯誤都透過此函式回傳 JSON 並終止程式。

## 相關文件

- [登入流程](./login-flow.md) - 錯誤在流程中的發生位置
- [Log 系統](./log-system.md) - 錯誤記錄機制
- [資料同步](./data-sync.md) - 同步錯誤的處理差異
