# 前端整合

## 頁面流程

```
1. 使用者點擊 OpenID 登入按鈕
   --> modules/OpenID/index_openID.php
   --> 產生 OAuth2 授權 URL
   --> 301 Redirect 到教育部 OpenID

2. 教育部認證完成,callback 回來
   --> login_openID.php
   --> 取得 access_token,存入 Session
   --> Redirect 到 choiceIdentity.php

3. 身分選擇頁面
   --> choiceIdentity.php
   --> 前端 AJAX 與 prodb_choiceIdentity.php 互動
   --> 完成登入
```

---

## 前端檔案

### index_openID.php

位置: `modules/OpenID/index_openID.php`

功能: OAuth2 授權導向入口

```php
$obj = new openid();
$_SESSION['azp_state'] = random_int(0, 9999999);
$_SESSION['nonce'] = isset($_SESSION['nonce'])
    ? $_SESSION['nonce']
    : base64_encode($_SESSION['azp_state']);

$link = AUTH_ENDPOINT . "?response_type=code&client_id=" . CLIENT_ID .
    "&redirect_uri=" . urlencode(REDIR_URI0) .
    "&scope=relation+openid+openid2+email+profile+eduinfo+googlemeet+mscalendar" .
    "&state=" . $_SESSION['azp_state'] .
    "&nonce=" . $_SESSION['nonce'];

header("HTTP/1.1 301 Moved Permanently");
header("Location: $link");
```

OAuth2 scope 包含: `relation`, `openid`, `openid2`, `email`, `profile`, `eduinfo`, `googlemeet`, `mscalendar`

### login_openID.php

位置: 根目錄 `login_openID.php`

功能: OAuth2 callback,取回 access_token

主要流程:
1. 驗證 `$_GET['code']` 和 `$_GET['state']`
2. 以 authorization code 換取 access_token
3. 存入 `$_SESSION['access_token']`
4. 成功則 redirect 到 `choiceIdentity.php`
5. 失敗則顯示錯誤訊息和重新整理按鈕

### choiceIdentity.php

位置: 根目錄 `choiceIdentity.php`

功能: 身分選擇頁面,前端 UI 與 AJAX 互動的主要頁面

技術構成:
- jQuery AJAX 呼叫
- SweetAlert2 訊息框
- DOMPurify XSS 防護
- 自定義 JavaScript 類別 (SemesterDataSet)

---

## AJAX 路由設定

所有 AJAX 呼叫經由 `turnpage_out.php` (非登入狀態的 AJAX 路由)。

```javascript
const ROUTER_CHOICE_IDENTITY = "<?php echo encodeAjaxUrlBase64('OpenID', 'prodb_choiceIdentity'); ?>";
const ROUTER_ADL_CLASSINFO = "<?php echo encodeAjaxUrlBase64('OpenID', 'prodb_get_adl_classInfo'); ?>";
```

注意: 使用 `turnpage_out.php` 而非 `turnpage.php`,因為 OpenID 登入時使用者尚未登入因材網。

---

## AJAX 呼叫對照表

### prodb_choiceIdentity.php 相關

| 前端函式 | ACTION | 傳送參數 | 回傳格式 |
|:---------|:-------|:---------|:---------|
| `getUserData(sOrgID)` | GET_OPENID_USER_DATA | organization_id | `{status, result}` 或 `{status, error_data}` |
| `quicklogincheck()` | QUICK_LOGIN_CHECK | (無) | `{status, message}` |
| `quicklogin()` | QUICK_LOGIN | (無) | redirect 或 `{status}` |
| `login(iActionCode, oCondition)` | LOGIN_SELECTED_IDENTITY | actionMode, access_level, organization_id, grade, class, year_semester, seatno | `{status}` 或 redirect |
| `bindUserAccount(sType, sOrgID, sAccount, sPass)` | BIND_USER_ACCOUNT | bindtype, organization_id, user_account, user_password | `{status}` |
| `setEnabledUserAccount(sUserID)` | SET_ENABLED_ACCOUNT | user_id | redirect 或 `{status}` |
| `modifiedUserClassInfo(sOrgID, iGrade, iClass, iSeatno)` | SET_USER_CLASS_INFO | organization_id, grade, class, seatno | redirect 或 `{status}` |
| `registerUserAccuount(sOrgID, iLevel, iGrade, iClass, iSeatno)` | REGISTER_ACCOUNT | access_level, grade, class, seatno, organization_id | redirect 或 `{status}` |

### prodb_get_adl_classInfo.php 相關

| 前端函式 | ACTION | 傳送參數 | 用途 |
|:---------|:-------|:---------|:-----|
| `getSchoolAllGrade()` | GetSchoolAllGrade | router | 取得學校所有年級 |
| `getGradeIncludeClass()` | GetGradeIncludeClass | router | 取得年級包含的班級 |

---

## 前端常數定義

```javascript
// 回應狀態碼
const RESPONSE_GET_USERINFO_SUCCESS = 1;         // 取得資料成功
const RESPONSE_GET_USERINFO_FAIL = 2;            // 取得資料失敗
const RESPONSE_REDIRECT = 1;                      // 登入成功,redirect
const RESPONSE_HAS_ERROR = 2;                     // 登入失敗
const RESPONSE_CLASSINFO_COMPARE_DIFFRENT = 4;    // 年班不一致,需手動設定
const RESPONSE_USER_DOES_NOT_HAVE_ACCOUNT = 7;    // 查無帳號
const RESPONSE_HAS_DUPLICATE_ACCOUNT = 9;         // 有重複帳號
const RESPONSE_MAY_HAS_DUPLICATE_ACCOUNT = 10;    // 可能有重複帳號

// 登入模式
const LOGIN_MODE_LOGIN = 1;
```

---

## 前端流程詳細

### 步驟 1: 頁面載入

```
choiceIdentity.php 載入
  --> 自動呼叫 getUserData()
  --> 取得 OpenID 使用者資料
  |
  |-- 成功 (status=1):
  |   --> 解析身分與學校清單
  |   --> 嘗試快速登入 quicklogincheck()
  |       |-- success: quicklogin() --> 登入
  |       +-- empty/error: 顯示身分選擇介面
  |
  +-- 失敗 (status=2):
      --> 顯示錯誤訊息
```

### 步驟 2: 使用者選擇身分

```
使用者選擇身分和學校
  --> 點擊登入按鈕
  --> login(actionCode, loginCondition)
  |
  |-- RESPONSE_REDIRECT (1): redirect 到首頁 (登入成功)
  |-- RESPONSE_HAS_ERROR (2): 顯示錯誤訊息
  |-- RESPONSE_CLASSINFO_COMPARE_DIFFRENT (4): 顯示年班選擇介面
  |-- RESPONSE_USER_DOES_NOT_HAVE_ACCOUNT (7): 顯示註冊介面
  |-- RESPONSE_HAS_DUPLICATE_ACCOUNT (9): 顯示帳號選擇介面
  +-- RESPONSE_MAY_HAS_DUPLICATE_ACCOUNT (10): 顯示綁定/創建選項
```

### 步驟 3: 後續互動

依據步驟 2 的回應:

| 回應 | 前端介面 | 使用者操作 | 觸發 ACTION |
|:-----|:---------|:----------|:------------|
| 年班不一致 | 年班選擇下拉選單 | 選擇年班座號 | SET_USER_CLASS_INFO |
| 查無帳號 | 註冊表單 | 填寫年班座號 | REGISTER_ACCOUNT |
| 重複帳號 | 帳號列表 | 選擇帳號 | SET_ENABLED_ACCOUNT |
| 可能有帳號 | 綁定/創建選項 | 選擇「綁定」或「創建」 | BIND_USER_ACCOUNT 或 REGISTER_ACCOUNT |

---

## timeout 處理

所有 AJAX 回傳都會檢查 timeout:

```javascript
success: function(data) {
    if (data.status != '' && data.status == 'timeout') {
        location.href = 'logout_msg.php';
    }
    resolve(data);
}
```

Session 逾時時後端回傳 `{status: 'timeout'}`,前端自動導向 `logout_msg.php`。

---

## SemesterDataSet 類別

前端自定義類別,用於管理學期資料:

```javascript
class SemesterDataSet {
    arrSemesterList = [];

    constructor(arrClassInfoList) {
        // 解析 OpenID 班級資料,建立學期清單 (去重複)
    }

    hasSemester(sOrganizationID, iYear, iSemester) {
        // 檢查學期是否已存在
    }

    insert(sOrganizationID, iYear, iSemester, sOrganizationName) {
        // 新增學期資料
    }
}
```

用途: 前端身分選擇介面中,整理出可選擇的學校和學期組合。

## 相關文件

- [登入流程](./login-flow.md) - 後端流程對應
- [概述](./overview.md) - ACTION 常數定義
- [錯誤處理](./error-handling.md) - 前端錯誤顯示
