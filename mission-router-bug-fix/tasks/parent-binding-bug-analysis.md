# 家長帳號綁定流程 - 完整分支說明

## 概述

家長帳號綁定有兩種方式：
1. **一般帳號綁定**（上方輸入欄）：輸入已存在的因材網帳號
2. **第三方帳號綁定**（下方教育雲）：透過教育雲第三方登入

---

## 路線一：一般帳號綁定（上方輸入欄）

### 流程圖

```
使用者輸入帳號 → 送出
    ↓
[檢查是否有輸入值]
    ├─ 無輸入 → 彈窗 A1
    └─ 有輸入 → [檢查是否已綁定]
                    ├─ 已綁定 → 彈窗 A2
                    └─ 未綁定 → [檢查帳號是否存在]
                                    ├─ 不存在 → 彈窗 A3
                                    └─ 存在 → [檢查是否輸入自己帳號]
                                                ├─ 是自己 → 彈窗 A4
                                                └─ 不是 → [檢查是否有第三方帳號]
                                                            ├─ 有第三方 → 彈窗 A5
                                                            └─ 無第三方 → [檢查 organization_id]
                                                                            ├─ 錯誤 → 彈窗 A6
                                                                            └─ 正確 → [檢查 access_level]
                                                                                        ├─ 非 4/9/11 → 彈窗 A6
                                                                                        └─ 是 4/9/11 → [檢查 email]
                                                                                                        ├─ 空/錯誤 → 彈窗 A7
                                                                                                        └─ 正確 → [展示學生檢查]
                                                                                                                    ├─ 已有因材網帳號 → 彈窗 A8
                                                                                                                    └─ 檢查通過 → 彈窗 A9 (成功)
```

### 彈窗對照表

| 代碼 | 錯誤碼 | 標題 | 內容 | 按鈕 |
|:----:|:-------|:-----|:-----|:-----|
| A1 | `inputEmpty` | 請輸入家長帳號! | (無) | [確認] |
| A2 | `combine_already` | 此帳號已綁定 | 請選擇另一筆帳號 | [確認] |
| A3 | `not_exist` | 無此訪客或家長帳號 | 請確認帳號是否輸入正確 | [確認] |
| A4 | `combine_yourself` | 請輸入其他帳號 | 不能綁自己的帳號喔 | [確認] |
| A5 | `third_party_login` | 找到帳號{帳號}, 這是您家長的帳號嗎? | 若是您的家長帳號, 請從第三方帳號「教育雲一般帳號」登入綁定即可 | [確認] |
| A6 | `orgId_error` / `accessLevel_error` | 此帳號無法綁定 | 僅限訪客或是家長帳號可以綁定 | [確認] |
| A7 | `email_empty` | 此帳號信箱尚未填入或格式錯誤 | 請從一般登入此帳號 {帳號}，前往「其他設定→個人資訊」修改信箱 | [確認] |
| A8 | `alreay_account_choose` | 已找到帳號 {帳號}，這是您的家長帳號嗎? | 若為您的家長帳號請「綁定此帳號」，或者前往「其他設定→個人資訊」更換另一信箱 | [綁定此帳號] [取消] |
| A9 | (成功) | 請確認以下資訊：家長 {姓名}，E-mail {信箱} | 即將送出「認證信件」, 點擊信件內連結即綁定。若需修改信箱, 請登入「家長帳號」, 前往「其他設定→個人資訊」修改信箱 | [寄出信件] [暫不發送] |

### 寄信後流程

| 代碼 | 情境 | 標題 | 內容 |
|:----:|:-----|:-----|:-----|
| A10 | 成功寄信 | 已寄出信件到 {email} | 請於「一小時內」前往信箱收取信件, 點擊「信件內網址進行認證綁定」 |
| A11 | `sendMailTooQuick` | 帳號: {帳號} 已發送過信件摟 | 如未收到信件, 請等待「60秒之後」再重新發送信件喔 |
| A12 | `mail-account-error` | 認證信對象帳號異常 | 請按下問題回報,感謝您的協助 |

---

## 路線二：第三方帳號綁定（教育雲）

### 流程圖

```
點擊「教育雲一般帳號」
    ↓
跳轉教育雲登入
    ↓
登入後返回
    ↓
[檢查第三方帳號是否存在]
    ├─ 不存在 + 無因材網帳號 → 彈窗 B1 (詢問創建)
    ├─ 不存在 + 有因材網帳號 → 彈窗 B2
    └─ 存在 → [檢查是否輸入自己帳號]
                ├─ 是自己 → 彈窗 B3
                └─ 不是 → [檢查是否已綁定]
                            ├─ 已綁定 → 彈窗 B4
                            └─ 未綁定 → [帳號轉換 + 權限更新 + 綁定]
                                            ├─ 某步驟失敗 → 彈窗 B5
                                            └─ 全部成功 → 彈窗 B6 (成功)
```

### 彈窗對照表

| 代碼 | 錯誤碼 | 標題 | 內容 | 按鈕 |
|:----:|:-------|:-----|:-----|:-----|
| B1 | `not_exist_thirdParty` | 該帳號「尚未註冊過因材網帳號」，要創建新帳號以綁定嗎? | (無) | [建立] [取消] |
| B2 | `alreay_account` | 已經有因材網帳號 {email} | 請由「因材網帳號」綁定 | [確認] |
| B3 | `combine_yourself` | 請輸入其他帳號 | 不能綁自己的帳號喔 | [確認] |
| B4 | `combine_already` | 此帳號已綁定 | 請選擇另一筆帳號 | [確認] |
| B5 | `error:{行號}` | 出現了一些錯誤 🔧 | 請按下問題回報,謝謝您的協助。錯誤代碼{行號} | [確認] |
| B6 | (成功) | ✓ 成功綁定家長 {姓名} | 家長帳號 {帳號} | [確定] |

### 創建新帳號後流程

| 代碼 | 情境 | 標題 | 內容 |
|:----:|:-----|:-----|:-----|
| B7 | 創建成功 | ✓ 成功綁定家長 {姓名} | 家長帳號 {帳號} |
| B8 | `data_error` | 帳號{帳號}異常 | 請按下問題回報,感謝您的協助 |
| B9 | `unknown` | 未知錯誤，請洽客服 | (無) |
| B10 | `failed_create_thirdParty` | 建立帳號 {帳號} 失敗 | 請按下問題回報,感謝您的協助 |

---

## 情境矩陣：不同身分的帳號綁定結果

### access_level 定義
| 值 | 身分 | 可綁定? |
|:--:|:-----|:-------:|
| 4 | 展示學生 (USER_STUDENT_DEMO1) | ✅ |
| 9 | 展示學生2 (USER_STUDENT_DEMO2) | ✅ |
| 11 | 家長 (USER_PARENTS) | ✅ |
| 21 | 教師 | ❌ |
| 25 | 校管 | ❌ |
| 其他 | 其他身分 | ❌ |

### 一般帳號綁定情境

| # | 輸入的帳號 | 帳號狀態 | 彈窗結果 |
|:-:|:-----------|:---------|:---------|
| 1 | 空白 | - | A1：請輸入家長帳號! |
| 2 | abc@test.com | 不存在 | A3：無此訪客或家長帳號 |
| 3 | abc@test.com | 存在，是自己帳號 | A4：不能綁自己的帳號喔 |
| 4 | abc@test.com | 存在，有第三方登入記錄 | A5：請從第三方帳號登入綁定 |
| 5 | abc@test.com | 存在，organization_id 不是 空/190041/190046 | A6：此帳號無法綁定 |
| 6 | abc@test.com | 存在，access_level 是 21 (教師) | A6：此帳號無法綁定 |
| 7 | abc@test.com | 存在，access_level 是 25 (校管) | A6：此帳號無法綁定 |
| 8 | abc@test.com | 存在，access_level=4，email 空 | A7：此帳號信箱尚未填入 |
| 9 | abc@test.com | 存在，access_level=4，email 正確，已綁定過 | A2：此帳號已綁定 |
| 10 | abc@test.com | 存在，access_level=4，email 正確 | A9：確認後寄信 → A10：成功 |
| 11 | abc@test.com | 存在，access_level=11 (家長) | A9：確認後寄信 → A10：成功 |

### 第三方帳號綁定情境

| # | 登入的第三方帳號 | 帳號狀態 | 彈窗結果 | ⚠️ 潛在問題 |
|:-:|:-----------------|:---------|:---------|:------------|
| 1 | 從未登入過因材網 | 無記錄 | B1：詢問是否創建 → B7：成功 | - |
| 2 | 有第三方登入，無因材網帳號 | user_third_party 有記錄 | B1 或 B6 | - |
| 3 | 無第三方登入，有因材網帳號 | user_info 有記錄 | B2：請由因材網帳號綁定 | - |
| 4 | 登入自己的帳號 | - | B3：不能綁自己 | - |
| 5 | 已經綁定過 | user_family 有記錄 | B4：此帳號已綁定 | - |
| 6 | 帳號格式 `190046-xxx@gmail.com` | access_level=4/9 | B6：成功 | ✅ 會更新權限 |
| 7 | **帳號格式 `xxx@gmail.com`** | **access_level=4/9** | **B6：顯示成功** | **❌ 權限不會更新** |

---

## 問題情境 #7 詳細說明

這是目前發現的主要 Bug。

### 為什麼會發生？

**程式碼** (prodb_stu_combine_parents.php#L383-L396)：
```php
$user_account_explode = explode("190046-", $fuser_account);
$new_fuser_acct = end($user_account_explode);

// 條件判斷
if ($fuser_account != $new_fuser_acct) {
    change_history(...);
    change_account_info(...);
    change_account_status($fuser_id);  // ← 改權限
}

combine_family($fuser_id, $suser_id);  // ← 綁定
```

### 情境分析

| 帳號格式 | $fuser_account | $new_fuser_acct | 條件結果 | 執行步驟 |
|:---------|:---------------|:----------------|:---------|:---------|
| `190046-abc@gmail.com` | `190046-abc@gmail.com` | `abc@gmail.com` | **不相等** | ✅ 執行 change_account_status |
| `abc@gmail.com` | `abc@gmail.com` | `abc@gmail.com` | **相等** | ❌ 跳過 change_account_status |

### 結果

- 帳號有 `190046-` 前綴：權限會更新為 11 (家長) ✅
- 帳號無 `190046-` 前綴：權限維持 4/9 (學生) ❌

---

## 問題情境 #8：第三方綁定首次點擊直接登入新帳號

### 使用者回報現象

1. 學生登入後，點擊「教育雲一般帳號」進行綁定
2. 跳轉到教育雲登入頁面
3. **登入後進入新帳號的首頁**（而非返回綁定頁面顯示彈窗）
4. 登出，重新登入學生帳號
5. 再次點擊「教育雲一般帳號」
6. 這次直接顯示彈窗詢問是否綁定剛才的帳號

### 問題根源分析

**問題在於 `login_cloud.php` 的條件判斷：**

```php
// login_cloud.php 第 92-99 行
if (isset($_SESSION['combineFamily']) && $_SESSION['combineFamily'] == 'educloud') {
    $_SESSION['cloud_sub'] = $userinfo['sub'];
    $_SESSION['cloud_name'] = $userinfo['name'];
    $_SESSION['cloud_email'] = $userinfo['email'];
    header('Location:'.$_SESSION['cloud_redirect']);  // ← 需要 cloud_redirect
    unset($_SESSION['loginfrom']);
    die();
}
```

**但在第一次點擊時的流程：**

1. **stu_combine_parents.php#L374-389** 發送 AJAX 設定 `$_SESSION['combineFamily'] = 'educloud'`
2. 然後跳轉到 `/modules/learningcloud/index_cloud.php`
3. **index_cloud.php#L20-22** 設定 `$_SESSION['cloud_redirect']`
4. 跳轉到教育雲登入
5. 登入成功後回到 **login_cloud.php**

**問題：教育雲 OAuth 會創建新的 Session！**

當使用者從教育雲登入後：
- 教育雲 OAuth 回調可能會 **覆蓋或重置 Session**
- `$_SESSION['combineFamily']` 可能在 OAuth 過程中遺失
- 導致 `login_cloud.php#L92` 條件判斷失敗
- 進入正常登入流程（第 135+ 行），直接登入新帳號

### 第二次點擊為何有效？

因為第一次登入後：
- `$_SESSION['cloud_sub']`、`$_SESSION['cloud_email']` 等資料已存在
- **stu_combine_parents.php#L32-34** 檢查到這些 Session：

```php
if (isset($_SESSION['combineFamily']) && isset($_SESSION['cloud_sub'])) {
    $third_party_combine = true;
}
```

- 直接顯示彈窗，不需再次跳轉

### 程式碼流程圖

```
第一次點擊：
學生點擊「教育雲」 
    → AJAX 設定 SESSION['combineFamily']='educloud'
    → 跳轉 index_cloud.php 
    → 設定 SESSION['cloud_redirect']
    → 跳轉教育雲登入
    ↓
教育雲 OAuth 登入
    → OAuth 回調可能重置 Session（問題點！）
    → 回到 login_cloud.php
    → SESSION['combineFamily'] 遺失
    → 條件判斷失敗，進入正常登入流程
    → 直接登入新帳號首頁 ❌

第二次點擊：
學生點擊「教育雲」
    → AJAX 設定 SESSION['combineFamily']='educloud'
    → stu_combine_parents.php 檢查 SESSION['cloud_sub'] 存在
    → 直接顯示彈窗「是否綁定為家長？」 ✅
```

### 建議修復

**方案 A：在 OAuth 回調前保存必要資訊到 Cookie**

```php
// prodb_stu_combine_parents.php - setData case
case 'educloud':
    $_SESSION['combineFamily'] = 'educloud';
    setcookie('combineFamily', 'educloud', time() + 600, '/');  // 10分鐘過期
    break;

// login_cloud.php - 檢查時同時檢查 Cookie
if ((isset($_SESSION['combineFamily']) && $_SESSION['combineFamily'] == 'educloud') 
    || (isset($_COOKIE['combineFamily']) && $_COOKIE['combineFamily'] == 'educloud')) {
    // ...
    setcookie('combineFamily', '', time() - 3600, '/');  // 清除 Cookie
}
```

**方案 B：使用 state 參數傳遞綁定標記**

在教育雲 OAuth 跳轉時，將 `combineFamily` 編碼到 state 參數中，OAuth 回調時解析。

---

## 驗證用 SQL

```sql
-- 查看特定帳號的綁定狀態
SELECT 
    ui.user_account,
    ui.email,
    us.access_level,
    CASE us.access_level 
        WHEN 4 THEN '展示學生'
        WHEN 9 THEN '展示學生2'
        WHEN 11 THEN '家長'
        ELSE '其他'
    END as 身分,
    uf.user_id as 綁定的學生,
    uf.create_date as 綁定時間
FROM user_info ui
LEFT JOIN user_status us ON ui.user_id = us.user_id
LEFT JOIN user_family uf ON ui.user_id = uf.fuser_id
WHERE ui.user_account = '{帳號}';

-- 找出可能有問題的綁定（已綁定但權限不是家長）
SELECT 
    ui.user_account,
    us.access_level,
    uf.user_id as 綁定學生,
    uf.create_date
FROM user_family uf
JOIN user_info ui ON uf.fuser_id = ui.user_id
JOIN user_status us ON ui.user_id = us.user_id
WHERE us.access_level IN (4, 9)
ORDER BY uf.create_date DESC
LIMIT 20;
```
