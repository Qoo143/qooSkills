# 家長綁定 Session 遺失問題

**狀態**：待調查  
**優先度**：高  
**影響範圍**：未註冊第三方帳號的家長綁定功能

---

## 問題描述

使用**未註冊的第三方帳號**進行家長綁定時，綁定流程異常中斷。

### 預期行為
1. OAuth 回調後，`login_cloud.php` L92-99 檢測到 `combineFamily` Session
2. 設定 `cloud_sub`、`cloud_name`、`cloud_email`
3. 跳轉回學生頁面，顯示綁定確認彈窗
4. 確認後由 `prodb_stu_combine_parents.php` 建立**家長帳號**

### 實際行為
1. OAuth 回調後，`combineFamily` Session **遺失**
2. L92 判斷失敗，程式繼續往下執行
3. `login_cloud.php` L179 建立**學生帳號**並登入
4. 原學生 Session 被替換，彈窗無法顯示

---

## 根本原因

**Session 在 OAuth 過程中遺失**

從 `index_cloud.php` 跳轉到 `cloud.edu.tw` 再回來時，`$_SESSION['combineFamily']` 不存在。

### 可能原因
1. `php.ini` 的 `session.cookie_domain` 設定
2. `session.cookie_secure` 與 HTTPS 不符
3. 跨域 Cookie 傳遞問題
4. Session 存儲機制（file/redis）差異

---

## 程式碼確認

### login_cloud.php L92-99（正確邏輯）
```php
if (isset($_SESSION['combineFamily']) && $_SESSION['combineFamily'] == 'educloud') {
    $_SESSION['cloud_sub'] = $userinfo['sub'];
    $_SESSION['cloud_name'] = $userinfo['name'];
    $_SESSION['cloud_email'] = $userinfo['email'];
    header('Location:'.$_SESSION['cloud_redirect']);
    die();  // ← 不會執行後續建立帳號邏輯
}
```

### prodb_stu_combine_parents.php L617（正確身分）
```php
'access_level' => USER_PARENTS,  // 家長身分
```

### login_cloud.php L208（錯誤身分）
```php
'access_level' => USER_STUDENT_DEMO2,  // 學生身分
```

---

## 調查方向

- [ ] 檢查正式環境與測試環境的 `php.ini` Session 設定差異
- [ ] 確認 OAuth 前後的 Session ID 是否一致
- [ ] 檢查 Cookie domain 和 secure 設定
- [ ] 考慮使用 OAuth state 參數傳遞 `combineFamily` 標記

---

## 相關檔案

| 檔案 | 說明 |
|:-----|:-----|
| [login_cloud.php](file:///c:/Users/User/Desktop/ntcu-nbnat-aial/login_cloud.php) | OAuth 回調處理 |
| [login_cloud_v2.php](file:///c:/Users/User/Desktop/ntcu-nbnat-aial/login_cloud_v2.php) | 重構版本（參考用） |
| [prodb_stu_combine_parents.php](file:///c:/Users/User/Desktop/ntcu-nbnat-aial/modules/UserManage/prodb_stu_combine_parents.php) | 後端綁定邏輯 |
| [stu_combine_parents.php](file:///c:/Users/User/Desktop/ntcu-nbnat-aial/modules/UserManage/stu_combine_parents.php) | 前端綁定頁面 |
| [index_cloud.php](file:///c:/Users/User/Desktop/ntcu-nbnat-aial/modules/learningcloud/index_cloud.php) | OAuth 入口 |

---

## 流程文檔

詳細流程分析請見：
- [2026-01-21-parent-binding-flow.md](file:///c:/Users/User/Desktop/ntcu-nbnat-aial/.agent/docs/2026-01-21-parent-binding-flow.md)
