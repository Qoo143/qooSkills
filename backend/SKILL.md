---
name: backend
description: >
  後端開發規範。當詢問 PHP、資料庫、PDO、turnpage、
  AJAX router、prodb_ 相關問題時觸發。
---

# 後端開發規範

## Master-Slave 資料庫架構

| 變數 | 用途 | 使用時機 |
|:-----|:-----|:---------|
| `$dbh` | Master (寫入) | INSERT, UPDATE, DELETE |
| `$dbh_slave` | Slave (讀取) | SELECT |

```php
// 讀取操作 - 使用 slave
$stmt = $dbh_slave->prepare("SELECT * FROM users WHERE id = :id");
$stmt->execute([':id' => $user_id]);
$user = $stmt->fetch(PDO::FETCH_ASSOC);

// 寫入操作 - 使用 master
$stmt = $dbh->prepare("INSERT INTO logs (action) VALUES (:action)");
$stmt->execute([':action' => $action]);
```

---

## turnpage.php AJAX 路由系統

### 架構

- **Entry Point**: `turnpage.php` - 中央 AJAX 請求處理器
- **Token 系統**: 儲存於 `$_SESSION['ADL_ROUTER_PASS']`
- **編碼**: 使用 `encodeAjaxUrlBase64()` 產生 router token

### 實作模式

**Frontend:**
```php
var router = '<?php echo encodeAjaxUrlBase64('module_name','prodb_file_name');?>';

$.post('../../turnpage.php', {
    router: router,
    act: 'action_name',
    // other parameters
}, function(response) {
    // handle response
});
```

**Backend (prodb_file_name.php):**
```php
$action = $_POST['act'] ?? '';
switch ($action) {
    case 'action_name':
        // handle action
        break;
}
```

---

## 檔案命名規範

| 類型 | 規範 | 範例 |
|:-----|:-----|:-----|
| 前端顯示 | 小寫底線 | `learning_reflection.php` |
| 後端處理 | `prodb_` 前綴 | `prodb_learning_reflection.php` |
| 測試檔案 | `test_` 前綴 | `test_group_evaluation.php` |

---

## 強制規則

1. **始終使用 prepared statements** - 禁止字串拼接
2. **讀寫分離** - 讀取用 `$dbh_slave`，寫入用 `$dbh`
3. **使用 PDO::FETCH_ASSOC** - 取得關聯陣列
