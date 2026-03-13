# API v4 開發工作流程

## 核心規則

> ⚠️ **以下規則不可違反**

1. **SQL 不可自己寫** — 只標註舊檔案中 SQL 的行號位置，由用戶手動貼到指定位置
2. **資料庫連線用現有的** — `$dbh`（寫入）/ `$dbh_slave`（讀取），來自 `config.php`，不建新連線
3. **Session 驗證用 `auth_chk.php`** — 入口 `index.php` 已 `require config.php`，Session 可用
4. **一次只做一個端點** — 逐一討論、逐一實作

---

## Session 與認證

### auth_chk.php 驗證流程

```
auth_chk.php
  → require config.php (Session + $dbh + $dbh_slave)
  → Auth::checkAuth()       → 檢查 Session 有效性
  → 建立 $_SESSION['user_data']  (UserData 物件)
  → 建立 $_SESSION['user_id']
  → 檢查帳號狀態 (id_used)
  → 檢查 access_level
  → 檢查學校啟用狀態
  → 檢查維護時間
  → 檢查閒置超時 (_IDLETIME)
  → 驗證 ACL
```

### v4 API 使用方式

v4 的 `index.php` 已 `require config.php`，所以 Session 和 $dbh 都可直接使用。  
認證透過 `AuthMiddleware` 處理，檢查 `$_SESSION['user_id']` 和 `$_SESSION['user_data']`。

### 可用的 Session 資料

```php
$_SESSION['user_id']              // 使用者 ID
$_SESSION['user_data']            // stdClass 物件，包含：
  ->user_id                       // 使用者 ID
  ->access_level                  // 權限等級
  ->user_type                     // 使用者類型 (1=學生, 21=教師, 等)
  ->organization_id               // 學校 ID
  ->id_used                       // 帳號啟用狀態
  ->organization_used              // 學校啟用狀態
```

### bootstrap.php 已註冊的 Helper

```php
Flight::currentUserId()    // → $_SESSION['user_id']
Flight::currentUserType()  // → (int) $_SESSION['user_type']
Flight::isAuthenticated()  // → !empty($_SESSION['user_id'])
Flight::isParent()         // → user_type in [14, 15]
Flight::isStudent()        // → user_type in [1, 4, 9]
Flight::isTeacher()        // → user_type in [21, 25]
```

### 資料庫連線

```php
// bootstrap.php 已註冊（直接使用 config.php 的全域變數）
Flight::db()       // → $dbh      (寫入/主庫)
Flight::dbSlave()  // → $dbh_slave (讀取/從庫)

// 在 Repository 中使用
$stmt = Flight::dbSlave()->prepare($sql);
$stmt->execute([$param]);
$rows = $stmt->fetchAll(PDO::FETCH_ASSOC);
```

---

## 開發流程

### Step 1: 分析端點

用戶提供舊 PHP 檔案後，產出端點分析表：

```markdown
### 端點分析：prodb_xxx.php

| # | act 值 | 說明 | HTTP 方法 | v4 路由建議 |
|---|--------|------|-----------|-------------|
| 1 | get_list | 取得清單 | GET | /xxx |
| 2 | get_detail | 取得詳情 | GET | /xxx/@id |
| 3 | save | 儲存資料 | POST | /xxx |
| 4 | update | 更新資料 | PUT | /xxx/@id |
| 5 | delete | 刪除資料 | DELETE | /xxx/@id |
```

### Step 2: 逐一討論端點

用戶選擇一個端點，AI 產出：

```markdown
#### 端點：get_list

**參數分析**

| 參數 | 來源 | 類型 | 說明 |
|------|------|------|------|
| user_id | SESSION | string | 使用者 ID |
| page | GET/POST | int | 頁碼 |

**SQL 位置**（AI 不複製 SQL，只標行號）

| 用途 | 舊檔案行號 | 目標方法 |
|------|------------|----------|
| 查詢清單 | :45-78 | XxxRepository::findList() |
| 計算總數 | :80-85 | XxxRepository::count() |

**依賴 Class**

| Class | 方法 | 說明 |
|-------|------|------|
| Mission | getUserMissions | 可 require_once 使用 |
```

### Step 3: 用戶確認 SQL

1. AI 建立 Repository 檔案，SQL 位置留空（用註解標記）
2. 用戶查看舊檔案的指定行號，把 SQL 貼到 Repository
3. AI 與用戶討論微調：WHERE 條件、JOIN、欄位映射

### Step 4: 實作

AI 依序建立：

1. **Repository** (SQL 由用戶提供)
2. **Service** (業務邏輯)
3. **Controller** (HTTP 層)
4. **routes.php** (加入路由)

### Step 5: 命名確認

```markdown
| 項目 | 建議 | 說明 |
|------|------|------|
| Controller | MissionController | |
| Service | MissionService | |
| Repository | MissionRepository | |
| 方法 | getStudentMissions | 駝峰 |
| 路由 | GET /student/missions | RESTful |
```

---

## 檔案模板

### Repository（SQL 預留位置）

```php
<?php
declare(strict_types=1);
namespace App\Repository;

use PDO;

class XxxRepository
{
    public function findList(string $userId, int $limit, int $offset): array
    {
        // [SQL] 用戶貼入 — 來源: prodb_xxx.php :45-78
        $sql = "
            -- TODO: 用戶貼入 SQL
        ";

        $stmt = \Flight::dbSlave()->prepare($sql);
        $stmt->execute([$userId, $limit, $offset]);
        return $stmt->fetchAll(PDO::FETCH_ASSOC);
    }
}
```

---

## 複用舊 Class

```php
// 在 Service 或 Repository 中引入
require_once __DIR__ . '/../../../../classes/Mission.php';
$mission = new \Mission();
$result = $mission->getUserMissions($userId);
```

常用位置：
- `classes/Mission.php`
- `classes/Quiz.php`
- `classes/Video.php`
- `classes/adp_core_class.php`
