# OpenID 登入流程

## 完整流程總覽

```
步驟 1: 使用者點擊 OpenID 登入
  --> index_openID.php 產生授權 URL,redirect 到教育部 OpenID
  |
步驟 2: OAuth2 授權流程
  --> login_openID.php 接收 callback,取得 access_token
  --> 存入 Session,redirect 到 choiceIdentity.php
  |
步驟 3: GET_OPENID_USER_DATA
  --> 呼叫 5 個 API (UserInfo, EduInfo, Relation, GetGuidToken, GetGUID)
  --> 記錄 Log (時機1: API 狀態 + 資料驗證)
  |
步驟 4: QUICK_LOGIN_CHECK
  --> 檢查是否符合快速登入條件 (5 個條件)
  |-- 符合 --> 步驟 5
  +-- 不符合 --> 步驟 6
  |
步驟 5: QUICK_LOGIN (快速登入)
  --> 執行資料同步 (學生:年班座號 / 教師:任課班級)
  --> 同步失敗不阻止登入
  --> 登入成功
  |
步驟 6: 前端:選擇身分與學校
  |
步驟 7: 記錄 Log (時機2: +登入身分)
  |
步驟 8: LOGIN_SELECTED_IDENTITY (一般登入)
  --> 帳號查詢 (4 層查詢邏輯)
  |-- 找到 1 筆啟用帳號 --> 資料同步 --> 補綁識別碼 --> 登入成功
  |-- 找到多筆 --> 前端:選擇帳號 --> 登入成功
  |-- 停用帳號 --> 顯示:帳號已停用
  |-- 轉出帳號 --> 顯示:帳號已轉出
  +-- 查無帳號 --> 檢查自動創建條件
      |-- 校管/局管 --> 創建管理員帳號 --> 登入成功
      |-- 信任學校學生 --> 創建學生帳號 --> 登入成功
      +-- 否 --> 前端:註冊介面 --> REGISTER_ACCOUNT --> 登入成功
```

## 關鍵決策點

| 決策點 | 判斷邏輯 | 影響 |
|:-------|:---------|:-----|
| 快速登入檢查 | 5 個條件全符合 | 跳過身分選擇,直接登入 |
| 帳號查詢 | 4 層查詢 (sub -> guid -> 年班 -> 同名) | 決定是否有帳號、哪個帳號 |
| 自動創建 | 身分 + 信任學校 | 自動建帳 vs 手動註冊 |
| 資料同步 | 信任學校 + 學期 + 年班有效 | 是否更新年班/任課資料 |

---

## 快速登入機制

### 設計目的

快速登入的特色:

- 不需要選擇身分、學校
- 登入速度快
- 同步失敗不阻止登入 (容錯性高)

### 觸發條件 (5 個條件全滿足)

程式碼位置: `prodb_choiceIdentity.php:207-308` (QUICK_LOGIN_CHECK)

快速登入分為兩個 ACTION:

1. `QUICK_LOGIN_CHECK` - 前端先呼叫,檢查是否符合條件
2. `QUICK_LOGIN` - 條件符合後執行實際登入

條件檢查邏輯:

```php
// 條件 1: 只有 1 種身分
$identityCount = count($arrEduinfo['titles'][0]['titles']);
if ($identityCount != 1) {
    // 回傳 status=empty,降級到一般登入
}

// 條件 2: OpenID 有年班資料 (mentor_class)
if (empty($arrOpenIDClassList['mentor_class']['grade']) ||
    empty($arrOpenIDClassList['mentor_class']['class'])) {
    // 回傳 status=empty,降級到一般登入
}

// 條件 3: 只綁定 1 個因材網帳號
$sql = "SELECT COUNT(*) FROM user_info
        WHERE OpenID_sub = :OpenID_sub AND used = 1";
if ($accountCount > 1) {
    // 回傳 status=empty,降級到一般登入
}

// 條件 4: openid_data 完全一致
$sImportOpenIDdata = json_encode($arrOpenIDClassList);
$sql = "SELECT * FROM user_info
        WHERE OpenID_sub = :OpenID_sub
        AND openid_data = :openid_data";
if ($matchCount === 0) {
    // 回傳 status=error,降級到一般登入
}

// 條件 5: 帳號狀態為啟用 (used=1)
// 遍歷查詢結果,尋找 used=1 的記錄
if (!$validRecord) {
    // 回傳 status=error,降級到一般登入
}
```

### 條件分析

| 條件 | 原因 | 不符合時的處理 |
|:-----|:-----|:--------------|
| 單一身分 | 多重身分需使用者選擇 | 進入一般登入,顯示身分選單 |
| 有年班資料 | 快速登入需比對年班 | 進入一般登入 |
| 單一帳號 | 多帳號需使用者選擇 | 進入一般登入,顯示帳號選單 |
| openid_data 一致 | 確保資料完全沒變化 | 進入一般登入,需重新同步 |
| 帳號啟用 | 停用帳號無法登入 | 顯示錯誤訊息 |

### openid_data 完全比對機制

條件 4 的 `openid_data` 是 JSON 字串完全比對,包含空格、欄位順序都必須一致。

```php
// 範例: 學生從三年級升到四年級
// 上次登入儲存的 openid_data
{
  "mentor_class": {
    "year": "113",
    "semester": "01",
    "grade": "3",      // 三年級
    "class": "5"
  }
}

// 本次 OpenID 回傳
{
  "mentor_class": {
    "year": "113",
    "semester": "02",  // 學期變了
    "grade": "4",      // 升級了
    "class": "5"
  }
}

// 結果: 不一致 --> 降級到一般登入 --> 執行資料同步
```

設計理由:

- 任何資料變化都需要重新同步
- 簡單可靠,不需要複雜的欄位比對邏輯
- 即使 JSON 格式變化 (空格、順序) 也會觸發重新驗證

### openid_data 寫入時機

`openid_data` 在一般登入 (`LOGIN_SELECTED_IDENTITY`) 的年班比對前寫入:

```php
// prodb_choiceIdentity.php:936-949
$sOpenIDData = json_encode($arrOpenIDClassList);
$update = $dbh->prepare("UPDATE user_info SET openid_data = :openid_data WHERE user_id = :user_id");
```

---

## QUICK_LOGIN 執行流程

程式碼位置: `prodb_choiceIdentity.php:309-467`

```
步驟 1: 取得通過 QUICK_LOGIN_CHECK 驗證的帳號資訊
  |
步驟 2: 重建 OpenID 使用者物件並驗證資料有效性
  --> 從 Session 取回 OpenID 原始資料
  --> 建立 OpenIDLoggedInUserInfo 物件
  --> 驗證年班、座號有效性
  |
步驟 3: 執行資料同步 (基本條件: 信任學校 + 年班有效)
  |-- 學生: 驗證學期 --> 同步年班座號
  +-- 教師: 過濾本校任課 --> Transaction 同步任課班級
  --> 同步失敗只記錄 error_log,不阻止登入
  |
步驟 4: 執行登入
  --> clearSessionOpenIDPrepareLoginInfo()
  --> loginWithOpenID()
```

---

## 一般登入流程 (LOGIN_SELECTED_IDENTITY)

程式碼位置: `prodb_choiceIdentity.php:470-1053`

一般登入是最複雜的 ACTION,包含:

1. 參數取得與 Session 驗證
2. 記錄 Log (時機2)
3. 帳號查詢 (4 層) - 詳見 [account-query.md](./account-query.md)
4. 帳號查詢結果處理
5. 自動創建帳號 - 詳見 [account-create.md](./account-create.md)
6. openid_data 寫入
7. 資料同步 - 詳見 [data-sync.md](./data-sync.md)
8. 補綁識別碼 (OpenID_sub / hash_guid)
9. 登入

### 帳號查詢結果處理

| 查詢結果 | 系統行為 | 後續處理 |
|:---------|:---------|:---------|
| 找到 1 筆啟用帳號 | 直接登入 | 執行資料同步、補綁識別碼 |
| 找到多筆啟用帳號 | 顯示選單 | 使用者選擇 --> SET_ENABLED_ACCOUNT |
| 找到停用帳號 | 阻止登入 | 顯示「帳號已停用,請聯絡管理員」 |
| 找到轉出帳號 | 阻止登入 | 顯示「帳號已轉出,請聯絡管理員」 |
| 同校同名 | 可能有帳號 | 顯示綁定選項 |
| 完全查無 | 檢查自動創建條件 | 自動創建 or 手動註冊 |

---

## 其他 ACTION 說明

### getCurrentSemester

程式碼位置: `prodb_choiceIdentity.php:84-86`

```php
case "getCurrentSemester":
    echo json_encode(getYearSeme());
    break;
```

- 功能:回傳當前學期
- 用途:前端登入介面初始化時取得系統當前學期

### LOGIN_TRANSFORM_TITLE_ACCOUNT

程式碼位置: `prodb_choiceIdentity.php:1055-1062`

```php
case LOGIN_TRANSFORM_TITLE_ACCOUNT:
    $sUserID = getSessionLoginIDPrepareLoginUserID();
    $sPassWord = getSessionLoginIDPrepareLoginPassword();
    clearSessionOpenIDPrepareLoginInfo();
    loginWithOpenID($sUserID, $sPassWord, time());
    break;
```

- 功能:登入職稱轉換後的帳號
- 流程:從 Session 取得準備登入的帳號密碼 --> 清除 Session --> 執行登入
- 觸發場景:使用者在前面流程中已完成職稱轉換,Session 中已儲存準備登入的帳密

### SET_ENABLED_ACCOUNT

程式碼位置: `prodb_choiceIdentity.php:1123-1169`

- 功能:當使用者有多個帳號時,啟用指定帳號並停用其他帳號
- 資料保護:使用 Transaction 確保「停用其他 + 啟用指定」操作的原子性
- 觸發場景:第一層查詢透過 OpenID_sub 找到多筆啟用帳號,前端顯示帳號選擇介面,使用者選擇後觸發

### SET_USER_CLASS_INFO

程式碼位置: `prodb_choiceIdentity.php:1332-1396`

- 功能:非信任學校學生登入時,年班不一致需手動設定
- 觸發條件:`LOGIN_MODE = LOGIN_MODE_MODIFIED_CLASSINFO`
- 限制:僅學生身分可使用
- 觸發場景:非信任學校 (`openid_trust=0`) 的學生登入,OpenID 年班與 user_info 不一致,前端顯示年班選擇介面

## 相關文件

- [帳號查詢](./account-query.md) - 4 層查詢結構細節
- [帳號創建](./account-create.md) - 自動創建機制
- [資料同步](./data-sync.md) - 同步條件與邏輯
- [前端整合](./frontend.md) - 前端 AJAX 呼叫流程
