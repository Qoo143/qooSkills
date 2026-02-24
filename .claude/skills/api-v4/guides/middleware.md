# 中間件開發指南

> 參考來源：[FlightPHP Middleware 官方文檔](https://docs.flightphp.com/learn/middleware)

## 中間件結構

Flight 中間件必須具備 `before()` 和/或 `after()` 方法。Engine 實例透過建構子自動注入。

```php
<?php

declare(strict_types=1);

namespace App\Middleware;

use flight\Engine;

class AuthMiddleware
{
    private Engine $app;

    public function __construct(Engine $app)
    {
        $this->app = $app;
    }

    /**
     * 路由執行前
     * @param array $params 路由參數
     * @return bool|void  返回 false 中止路由，觸發 403
     */
    public function before(array $params = [])
    {
        if (empty($_SESSION['user_id'])) {
            $this->app->jsonHalt([
                'success' => false,
                'error'   => ['code' => 401, 'message' => '未授權']
            ], 401);
            return false;
        }
    }

    /**
     * 路由執行後（可選）
     */
    public function after()
    {
        // 記錄 API 存取日誌等
    }
}
```

## 執行順序

`before()` 按添加順序執行，`after()` 按**倒序**執行：

```
[Middleware1, Middleware2]
before:  Middleware1::before() → Middleware2::before()
after:   Middleware2::after()  → Middleware1::after()
```

## 套用方式

```php
// 1. 套用到路由群組
Flight::group('/api', function () { ... }, [AuthMiddleware::class]);

// 2. 套用到單一路由
Flight::route('GET /admin', [AdminController::class, 'index'])
    ->addMiddleware([AdminMiddleware::class]);

// 3. 全域 before hook（非中間件，但類似效果）
Flight::before('start', function () {
    // 每個請求都會執行
});
```

## 處理中間件中止

在中間件中阻止路由執行有三種方式：

### 1. 簡單 return false

```php
class MyMiddleware {
    public function before($params) {
        if (!$this->isAuthorized()) {
            return false; // → 自動觸發 403 Forbidden
        }
    }
}
```

### 2. 重導向

```php
class MyMiddleware {
    public function before($params) {
        if (!$this->isLoggedIn()) {
            Flight::redirect('/login');
            exit;
        }
    }
}
```

### 3. 自訂 JSON 錯誤（API 常用）

```php
class MyMiddleware {
    public function before($params) {
        $auth = Flight::request()->getHeader('Authorization');
        if (empty($auth)) {
            Flight::jsonHalt(['error' => '請先登入'], 403);
        }
    }
}
```

## 常見使用情境

### API Key 驗證中間件

```php
use flight\Engine;

class ApiMiddleware
{
    protected Engine $app;

    public function __construct(Engine $app)
    {
        $this->app = $app;
    }

    public function before(array $params)
    {
        $authHeader = $this->app->request()->getHeader('Authorization');
        $apiKey = str_replace('Bearer ', '', $authHeader);

        $apiKeyHash = hash('sha256', $apiKey);
        $isValid = !!Flight::dbSlave()->fetchRow(
            "SELECT 1 FROM api_keys WHERE hash = ? AND valid_date >= NOW()",
            [$apiKeyHash]
        );

        if (!$isValid) {
            $this->app->jsonHalt(['error' => 'Invalid API Key'], 401);
        }
    }
}
```

### 登入驗證中間件

```php
use flight\Engine;

class LoggedInMiddleware
{
    protected Engine $app;

    public function __construct(Engine $app)
    {
        $this->app = $app;
    }

    public function before(array $params)
    {
        if (empty($_SESSION['user_id'])) {
            $this->app->jsonHalt([
                'success' => false,
                'error'   => ['code' => 401, 'message' => '未登入']
            ], 401);
            return false;
        }
    }
}
```

### 路由參數安全性驗證

```php
use flight\Engine;

class RouteSecurityMiddleware
{
    protected Engine $app;

    public function __construct(Engine $app)
    {
        $this->app = $app;
    }

    public function before(array $params)
    {
        $clientId = $params['clientId'];
        $jobId = $params['jobId'];

        $isValid = !!Flight::dbSlave()->fetchRow(
            "SELECT 1 FROM client_jobs WHERE client_id = ? AND job_id = ?",
            [$clientId, $jobId]
        );

        if (!$isValid) {
            $this->app->jsonHalt(['error' => '無權存取此資源'], 403);
            return false;
        }
    }
}
```

### Config 驅動驗證中間件

建構子接收規則陣列（不需 Engine），掛載到個別路由：

```php
Flight::route('GET /items', [ItemController::class, 'index'])
    ->addMiddleware([new ValidateQueryMiddleware([
        'page' => ['type' => 'int', 'min' => 1, 'default' => 1],
        'name' => ['type' => 'string', 'in' => ['a','b'], 'default' => 'a'],
    ])]);
// 規則: type(int|string), default, in(白名單), min, max, required
// 驗證後的值直接寫回 $query，Controller 直接讀
```

## ⚠️ 中間件關鍵注意事項

```php
// 1. before() 返回 false → 觸發 403 Forbidden
//    如果要自訂錯誤碼，需在 before() 內自行輸出回應

// 2. before() 接收 $params 陣列（路由參數），但不接收路由參數作為直接引數
public function before(array $params = []) { ... }

// 3. ❌ 錯誤：用 Closure 方式定義方法
class Bad {
    public $before;
    public function __construct() {
        $this->before = function() { ... };  // ❌ 不會被呼叫
    }
}

// ✅ 正確：必須是實際的 before/after 方法
class Good {
    public function before() { ... }  // ✅
}
```
