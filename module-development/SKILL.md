---
name: module-development
description: >
  新模組開發流程。當詢問新功能開發、新增模組、
  開發流程、ACL 相關問題時觸發。
---

# 新模組開發流程

## 模組目錄結構

```
modules/[module_name]/
├── index.php              # 主進入頁面
├── component.php          # 功能頁面
├── prodb_component.php    # 後端處理
└── js/                    # JavaScript 檔案
    └── main.js
```

## 開發步驟

### 1. 建立模組目錄

```bash
mkdir modules/[module_name]
```

### 2. 建立前端頁面

```php
<?php
// modules/[module_name]/index.php
require_once '../../include/config.php';
// ...頁面內容
```

### 3. 建立後端處理

```php
<?php
// modules/[module_name]/prodb_[action].php
$action = $_POST['act'] ?? '';
switch ($action) {
    case 'get_data':
        // 處理邏輯
        break;
}
```

### 4. 設定 ACL 權限

在 `modules/menu/acl.php` 中新增路由權限：

```php
$menu_items[] = [
    'path' => '/modules/[module_name]/',
    'roles' => [21, 25]  // 教師角色
];
```

---

## 檔案命名慣例

| 類型 | 規範 | 範例 |
|:-----|:-----|:-----|
| 前端顯示 | descriptive_name.php | `learning_reflection.php` |
| 後端處理 | prodb_name.php | `prodb_learning_reflection.php` |
| 角色特定 | name_role.php | `reflection_student.php` |

---

## 開發檢查清單

- [ ] 建立模組目錄
- [ ] 實作前端頁面
- [ ] 實作後端處理（prodb_）
- [ ] 設定 ACL 權限
- [ ] 測試不同角色存取
- [ ] 確認安全性（preventXss, PDO）
