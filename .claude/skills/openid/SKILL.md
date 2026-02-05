---
name: openid
description: >
  OpenID 模組技術參考。當詢問 OpenID 登入流程、帳號查詢、資料同步、
  Log 記錄、快速登入、帳號創建、身分認證 相關問題時觸發。
---

# OpenID 模組

因材網與教育部 OpenID Connect (OIDC) 整合的統一身分認證模組。

## 子文件索引

| 文件 | 說明 |
|:-----|:-----|
| [overview.md](./overview.md) | 模組概述、技術架構、Endpoint 設定、OpenID API 清單 |
| [login-flow.md](./login-flow.md) | 登入流程總覽、快速登入機制、一般登入流程、關鍵決策點 |
| [account-query.md](./account-query.md) | 帳號查詢 4 層結構、優先順序邏輯、台南 OpenID 臨時方案 |
| [account-create.md](./account-create.md) | 自動創建帳號觸發條件、不同身分創建規則、Transaction 保護 |
| [data-sync.md](./data-sync.md) | 學生與教師資料同步、同步條件差異、學期驗證機制 |
| [log-system.md](./log-system.md) | Log 記錄時機、資料表結構、error_status 判斷、API 狀態記錄 |
| [error-handling.md](./error-handling.md) | 各類錯誤處理方式、快速登入與一般登入的錯誤處理差異 |
| [database.md](./database.md) | 相關資料表清單、一般資料表與 Log 資料表說明 |
| [frontend.md](./frontend.md) | 前端頁面架構、AJAX 呼叫模式、ACTION 常數對應 |

## 核心檔案

| 檔案 | 位置 | 職責 |
|:-----|:-----|:-----|
| prodb_choiceIdentity.php | modules/OpenID/ | 登入流程主控:處理所有 ACTION |
| config_openid.php | modules/OpenID/ | OpenID Endpoint 與憑證設定 |
| library.class.php | modules/OpenID/ | OpenID API 呼叫封裝 |
| function_OpenID.php | modules/OpenID/ | 業務邏輯與輔助函式 |
| function_addUser.php | modules/OpenID/ | 帳號建立函式 |
| function_user_info.php | modules/OpenID/ | 帳號查詢與資料同步函式 |
| openID_user_Info.php | classes/OpenID/ | OpenID 使用者物件與 Log 記錄器 |
| openID_session_keeper.php | classes/OpenID/ | Session 管理類別 |

## 前端檔案

| 檔案 | 位置 | 職責 |
|:-----|:-----|:-----|
| index_openID.php | modules/OpenID/ | OAuth2 授權導向(產生授權 URL 並 redirect) |
| login_openID.php | 根目錄 | OAuth2 callback:取回 access_token |
| choiceIdentity.php | 根目錄 | 身分選擇頁面:前端 UI 與 AJAX 互動 |

## ACTION 一覽

prodb_choiceIdentity.php 處理的 ACTION (透過 `$_POST['action']`):

| ACTION 常數 | 值 | 說明 |
|:------------|:---|:-----|
| (字串) | getCurrentSemester | 取得當前學期 |
| GET_OPENID_USER_DATA | getOpenIDUserData | 呼叫 OpenID API 取得使用者資料 |
| QUICK_LOGIN_CHECK | quicklogincheck | 檢查是否符合快速登入條件 |
| QUICK_LOGIN | quicklogin | 執行快速登入 |
| LOGIN_SELECTED_IDENTITY | loginSelectedIdentity | 一般登入流程 |
| LOGIN_TRANSFORM_TITLE_ACCOUNT | loginTransformTitleAccount | 登入職稱轉換後的資料 |
| BIND_USER_ACCOUNT | bindUserAccount | 綁定舊帳號 |
| SET_ENABLED_ACCOUNT | setEnabledAccount | 啟用指定帳號並停用其他帳號 |
| REGISTER_ACCOUNT | registerAccount | 帳號註冊 |
| SET_USER_CLASS_INFO | setUserClassInfo | 設定年班資料 |
