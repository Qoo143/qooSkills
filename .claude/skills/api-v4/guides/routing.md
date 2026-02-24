# 路由系統完整指南

> 參考來源：[FlightPHP Routing 官方文檔](https://docs.flightphp.com/learn/routing)

## 基本路由方法

```php
// 通用路由（pattern 前加方法名）
Flight::route('GET /path', $callback);
Flight::route('POST /path', $callback);
Flight::route('GET|POST /path', $callback);  // 多方法

// 快捷方法（推薦）
Flight::get('/path', $callback);       // → Router::map('GET /path', ...)
Flight::post('/path', $callback);      // → Router::map('POST /path', ...)
Flight::put('/path', $callback);       // → Router::map('PUT /path', ...)
Flight::patch('/path', $callback);     // → Router::map('PATCH /path', ...)
Flight::delete('/path', $callback);    // → Router::map('DELETE /path', ...)
```

## 路由參數

```php
// 命名參數：用 @ 前綴
Flight::route('GET /users/@id', function (string $id) { ... });

// 帶正則約束
Flight::route('GET /users/@id:[0-9]+', function (string $id) { ... });

// 可選參數：用括號包裹
Flight::route('GET /blog(/@year(/@month))', function (?string $year = null, ?string $month = null) { ... });

// ⚠️ 萬用字元
Flight::route('/api/*', function () { ... });
```

## 路由群組

```php
// group(string $prefix, callable $callback, array $middlewares = [])
Flight::group('/api/v4', function () {
    Flight::get('/users', [UserController::class, 'index']);

    Flight::group('/admin', function () {
        Flight::get('/stats', [AdminController::class, 'stats']);
    }, [AdminMiddleware::class]);
}, [AuthMiddleware::class]);  // 群組中間件
```

## 路由別名

```php
Flight::route('GET /users/@id', [UserController::class, 'show'], false, 'user.show');

// 透過別名取得 URL
$url = Flight::getUrl('user.show', ['id' => 42]); // → /users/42
```

## RESTful Resource

```php
// 自動產生 CRUD 路由
// resource(string $pattern, string $controllerClass, array $options = [])
Flight::resource('/users', UserController::class);

// 產生以下路由：
// GET    /users       → index()
// GET    /users/@id   → show(string $id)
// POST   /users       → create()
// PUT    /users/@id   → update(string $id)
// DELETE /users/@id   → destroy(string $id)

// 自訂選項
Flight::resource('/users', UserController::class, [
    'only' => ['index', 'show'],           // 只產生部分
    'middleware' => [AuthMiddleware::class], // 套用中間件
]);
```

## 串流路由

```php
// 基本串流
Flight::route('GET /stream', function () {
    // 處理串流回應
})->stream();

// 帶自訂 header 的串流（如 SSE）
Flight::route('GET /sse', function () {
    echo "data: hello\n\n";
    flush();
})->streamWithHeaders([
    'Content-Type' => 'text/event-stream',
    'Cache-Control' => 'no-cache',
]);
```

## ⚠️ 常見路由錯誤

```php
// ❌ 錯誤：使用 Laravel 風格的 {id}
Flight::get('/users/{id}', ...);

// ✅ 正確：使用 @id
Flight::get('/users/@id', ...);

// ❌ 錯誤：在 group callback 內使用 $router 參數
Flight::group('/api', function ($router) { ... });

// ✅ 正確：callback 為無參數，繼續使用 Flight 靜態方法
Flight::group('/api', function () {
    Flight::get('/test', ...);
});
```

> **注意**：官方部落格範例中使用 `function(Router $router)` 的寫法，是因為版本差異。
> 在本專案使用的 Engine 實例模式下，group callback **不應帶參數**，繼續使用 `Flight::xxx()` 或 `$this->app->xxx()`。
