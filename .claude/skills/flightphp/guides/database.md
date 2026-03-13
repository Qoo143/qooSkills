# SimplePdo 資料庫操作指南

> 參考來源：[FlightPHP 原始碼](https://github.com/flightphp/core) — `database/SimplePdo.php`

## 概述

`SimplePdo` 繼承 `PDO`，提供便捷的查詢方法。它是 FlightPHP v3 推薦的資料庫封裝層。

> **⚠️ `PdoWrapper` 已 deprecated，請使用 `SimplePdo`。**

## 註冊方式

```php
// bootstrap.php 中
Flight::register('db', 'flight\database\SimplePdo', [
    'pgsql:host=localhost;dbname=mydb',
    'username',
    'password'
]);
```

本專案使用舊有的 `config.php` 全域連線，在 `bootstrap.php` 中直接封裝：

```php
// 使用 config.php 的全域變數（實際用的是 map()，不是 register()）
Flight::map('db', function () {
    global $dbh;
    return $dbh;
});
Flight::map('dbSlave', function () {
    global $dbh_slave;
    return $dbh_slave;
});
// 使用方式：Flight::db() (寫入) / Flight::dbSlave() (讀取)
```

> **注意**：`map()` 和 `register()` 的差異：`register()` 是延遲載入單例模式，`map()` 則是每次呼叫都執行回呼。
> 本專案用 `map()` 是因為 `$dbh` / `$dbh_slave` 已在 `config.php` 中建立，只需返回現有實例。

## 查詢方法一覽

### fetchRow — 取單一行

```php
// 回傳 ?Collection（null 表示無結果）
$row = $db->fetchRow("SELECT * FROM users WHERE id = ?", [$id]);

// ⚠️ 務必檢查 null
if ($row === null) {
    Flight::jsonHalt(['error' => '找不到'], 404);
}
$name = $row->name;   // Collection 支援物件存取
$name = $row['name']; // 也支援陣列存取
```

### fetchAll — 取所有行

```php
// 回傳 array<int, Collection>
$rows = $db->fetchAll("SELECT * FROM users WHERE active = ?", [1]);
```

### fetchColumn — 取單欄陣列

```php
// 回傳 array<int, mixed>
$ids = $db->fetchColumn("SELECT id FROM users WHERE active = ?", [1]);
```

### fetchPairs — 取鍵值對

```php
// 回傳 array<string|int, mixed>
$nameMap = $db->fetchPairs("SELECT id, name FROM users");
```

### runQuery — 原始查詢

```php
// 回傳 PDOStatement（適用 INSERT/UPDATE/DELETE/複雜 SELECT）
$stmt = $db->runQuery("INSERT INTO users (name) VALUES (?)", ['John']);
$affected = $stmt->rowCount();
```

### insert — 快速插入

```php
// 回傳 lastInsertId
$lastId = $db->insert('users', ['name' => 'John', 'email' => 'j@e.com']);

// 批量插入
$lastId = $db->insert('users', [
    ['name' => 'A', 'email' => 'a@e.com'],
    ['name' => 'B', 'email' => 'b@e.com'],
]);
```

### update — 快速更新

```php
// 回傳影響行數
$affected = $db->update('users', ['name' => 'Jane'], 'id = ?', [1]);
```

### delete — 快速刪除

```php
// 回傳影響行數
$deleted = $db->delete('users', 'id = ?', [1]);
```

### IN 查詢

```php
// 自動展開陣列
$rows = $db->fetchAll(
    "SELECT * FROM users WHERE id IN(?)",
    [[1, 2, 3]]
);
```

### 交易

```php
$result = $db->transaction(function ($db) {
    $db->insert('users', ['name' => 'John']);
    $id = $db->lastInsertId();
    $db->insert('logs', ['user_id' => $id, 'action' => 'created']);
    return $id;
});
```

## 本專案的資料庫使用模式

本專案因為使用舊版 `config.php` 的 PDO 連線（非 SimplePdo），所以 Repository 使用原生 PDO 語法：

```php
class XxxRepository
{
    public function findList(string $userId, int $limit, int $offset): array
    {
        $sql = "SELECT * FROM xxx WHERE user_id = ? LIMIT ? OFFSET ?";

        $stmt = \Flight::dbSlave()->prepare($sql);
        $stmt->execute([$userId, $limit, $offset]);
        return $stmt->fetchAll(\PDO::FETCH_ASSOC);
    }
}
```

> 讀取操作用 `Flight::dbSlave()`，寫入操作用 `Flight::db()`。

## Collection 與陣列轉換

當使用 `SimplePdo` 時，`fetchAll()` 回傳的每行是 `Collection` 物件：

```php
$rows = Flight::db()->fetchAll($sql, $params);

// Collection → array
$arrayRows = array_map(function ($row) {
    return $row instanceof Collection ? $row->getData() : $row;
}, $rows);
```

## ⚠️ 常見資料庫錯誤

```php
// ❌ 使用已 deprecated 的 PdoWrapper
Flight::register('db', 'flight\database\PdoWrapper', [...]);

// ✅ 使用 SimplePdo
Flight::register('db', 'flight\database\SimplePdo', [...]);

// ❌ fetchRow 結果沒有 null 檢查
$row = $db->fetchRow($sql);
$name = $row['name'];  // 如果查無結果會出錯

// ✅ 先檢查 null
$row = $db->fetchRow($sql, [$id]);
if ($row === null) {
    Flight::jsonHalt(['error' => '找不到'], 404);
}

// ❌ SQL 字串拼接
$db->fetchAll("SELECT * FROM users WHERE name = '$name'");

// ✅ 使用參數綁定
$db->fetchAll("SELECT * FROM users WHERE name = ?", [$name]);
```
