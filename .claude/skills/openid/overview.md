# OpenID 模組概述

## 模組功能

OpenID 模組是因材網的統一身分認證核心,負責與教育部 OpenID Connect (OIDC) 系統整合,提供單點登入 (SSO) 功能。

主要功能:

- OAuth2 認證流程:與教育部 OpenID 伺服器進行授權碼交換
- 多重身分處理:支援學生、教師、校管、局管等多種身分
- 帳號自動創建:信任學校學生與管理人員可自動建立帳號
- 資料同步:同步 OpenID 的年班、座號、任課班級等資料
- Log 完整記錄:記錄 API 狀態、資料驗證結果供除錯

## 技術架構

```
使用者 --> 教育部 OpenID --> 因材網 OpenID 模組
                               |-- OAuth2 認證
                               |-- 呼叫 5 個 API
                               |-- 帳號查詢與創建
                               |-- 資料同步
                               +-- 登入因材網
```

## 檔案架構

```
modules/OpenID/
|-- prodb_choiceIdentity.php     # 核心邏輯:身分選擇、登入決策
|
|-- config_openid.php            # OpenID 設定 (Client ID, Secret, Endpoints)
|-- library.class.php            # OpenID API 呼叫封裝
|
|-- function_OpenID.php          # 業務邏輯與輔助函式
|-- function_addUser.php         # 帳號建立函式
|-- function_user_info.php       # 帳號查詢與資料同步函式
|
|-- index_openID.php             # OAuth2 授權導向入口
|-- pro_adduser_function.php     # 帳號建立輔助
|-- prodb_get_adl_classInfo.php  # 查詢因材網年班資料
|-- ref_OpenIdOrg.php            # OpenID 學校組織參照
|-- OpenIDLog.php                # OpenID Log 輔助
|-- turnpage.php                 # 模組內 AJAX 路由
|
classes/OpenID/
|-- openID_user_Info.php         # OpenID 使用者物件與 Log 記錄器
|-- openID_session_keeper.php    # Session 管理類別
|-- openID_common_tools.php      # 共用工具 (HttpStatusRecorder, Timer)
|-- openID_logger_config.php     # Log 設定常數
+-- common_user_Info.php         # 共用使用者資訊基底
```

## OpenID Endpoint 設定

設定檔位置: `modules/OpenID/config_openid.php`

### 環境設定

```php
// OpenID 伺服器位置
define('OPENID_DOMAIN', 'https://oidc.tanet.edu.tw');  // 正式區
// define('OPENID_DOMAIN', 'https://auth.sso.edu.tw');  // 測試區

// RP 申請獲得
define('CLIENT_ID', '33a438c202a2c7acf2b3601156e39274');
define('CLIENT_SECRET', '82e6df0a...');
```

### Endpoint 設定

```php
// Well-known URL (自動發現 endpoint)
define('WELL_KNOWN_URL', OPENID_DOMAIN.'/.well-known/openid-configuration');

// 動態 endpoint 模式 (0=使用固定設定, 1=每次從 well-known 取得)
define('DYNAMICAL_ENDPOINT', 0);

// DYNAMICAL_ENDPOINT=0 時需手動設定以下 endpoint
define('AUTH_ENDPOINT', OPENID_DOMAIN.'/oidc/v1/azp');              // 授權端點
define('TOKEN_ENDPOINT', OPENID_DOMAIN.'/oidc/v1/token');           // Token 端點
define('USERINFO_ENDPOINT', OPENID_DOMAIN.'/oidc/v1/userinfo');     // UserInfo API
define('RELATION_ENDPOINT', OPENID_DOMAIN.'/moeresource/api/v2/oidc/relation'); // Relation API
define('EDUINFO_ENDPOINT', OPENID_DOMAIN.'/moeresource/api/v2/oidc/eduinfo');   // EduInfo API
define('JWKS_URI', OPENID_DOMAIN.'/oidc/v1/jwksets');               // JWKS 端點
define('PROFILE_ENDPOINT', OPENID_DOMAIN.'/moeresource/api/v1/oidc/profile');   // Profile API
```

### Redirect URI

```php
define('OPENID_WEB_STIE_REDIRECT',
    (preg_match('/cron-server/i', gethostname()))
        ? 'http://'.$_SERVER['SERVER_NAME']   // cron-server 使用 http
        : 'https://'.$_SERVER['SERVER_NAME']  // 其他使用 https
);
define('REDIR_URI0', OPENID_WEB_STIE_REDIRECT.'/login_openID.php');
```

重點說明:

- `DYNAMICAL_ENDPOINT` 設為 0 可提升效能,避免每次登入都呼叫 well-known
- Redirect URI 需與 RP 申請時填寫的 URI 一致

## 支援的 OpenID API

在 `GET_OPENID_USER_DATA` 階段會依序呼叫以下 5 個 API:

| API 名稱 | 呼叫時機 | 回傳資料 | 資料使用 |
|:---------|:---------|:---------|:---------|
| UserInfo | GET_OPENID_USER_DATA | 姓名、email、OpenID_sub | 所有身分 |
| EduInfo | GET_OPENID_USER_DATA | 身分、年班、學期 | 所有身分 |
| Relation | GET_OPENID_USER_DATA | 任課班級與科目 | 教師同步 |
| GetGuidToken | GET_OPENID_USER_DATA | 用於查詢 GUID 的授權 | 所有身分 |
| GetGUID | GET_OPENID_USER_DATA | SHA256 雜湊後的身分證字號 | 所有身分 |

### Relation API 補充說明

- 呼叫時機:在 `GET_OPENID_USER_DATA` 階段呼叫(尚未選擇身分,所有使用者都會執行)
- 回傳內容:OpenID 會回傳 `$arrRelation['relation']` 任課班級與科目資料
- 資料使用:只有教師身分在資料同步時會使用 `$arrRelation['relation']`
- 學生登入時也會呼叫此 API,但程式碼不會使用其回傳資料

## 關鍵常數定義

定義於 `prodb_choiceIdentity.php` 開頭:

```php
// 登入模式
define("LOGIN_MODE_LOGIN", 1);              // 一般登入
define("LOGIN_MODE_REGISTER", 2);           // 註冊模式
define("LOGIN_MODE_MODIFIED_CLASSINFO", 3); // 修改年班模式
define("LOGIN_MODE_SET_ENABLED_ACCOUNT", 4);// 啟用帳號模式

// 無效年班錯誤清單
define("INVALID_CLASSINFO_ERROR_LIST", [
    OPENID_CLASSINFO_IS_EMPTY,
    OPENID_GRADE_IS_NOT_NUMERIC,
    OPENID_GRADE_IS_ZERO,
    OPENID_GRADE_IS_NEGATIVE,
    OPENID_CLASSNO_EMPTY,
    OPENID_CLASSNO_IS_NOT_NUMERIC,
    OPENID_CLASSNO_ZERO,
    OPENID_CLASSNO_NEGATIVE
]);

// 無效座號錯誤清單
define("INVALID_SEATNUMBER_ERROR_LIST", [
    OPENID_SEATNO_EMPTY,
    OPENID_SEATNO_IS_NOT_NUMERIC,
    OPENID_SEATNO_NEGATIVE,
    OPENID_SEATNO_ZERO
]);
```

## 相關文件

- [登入流程](./login-flow.md) - 快速登入與一般登入機制
- [帳號查詢](./account-query.md) - 4 層查詢結構
- [帳號創建](./account-create.md) - 自動創建規則
- [資料同步](./data-sync.md) - 學生與教師同步機制
- [Log 系統](./log-system.md) - Log 記錄機制
- [錯誤處理](./error-handling.md) - 錯誤類型與處理方式
- [資料庫](./database.md) - 相關資料表
- [前端整合](./frontend.md) - 前端頁面與 AJAX
