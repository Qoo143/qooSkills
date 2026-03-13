# 框架擴充與事件系統

> 參考來源：[FlightPHP Extending 官方文檔](https://docs.flightphp.com/learn/extending)

## map() — 映射自訂方法

將自訂函式掛載到 Flight，使其可透過 `Flight::xxx()` 呼叫。

```php
// map(string $name, callable $callback)
Flight::map('success', function ($data = null, int $code = 200) {
    Flight::json(['success' => true, 'data' => $data], $code);
});

Flight::map('fail', function (string $message, int $code = 400, $extra = null) {
    $error = ['code' => $code, 'message' => $message];
    if ($extra !== null) {
        $error = array_merge($error, $extra);
    }
    Flight::jsonHalt(['success' => false, 'error' => $error], $code);
});

// 使用
Flight::success($data);
Flight::success($data, 201);
Flight::fail('參數錯誤', 400);
Flight::fail('找不到', 404, ['field' => 'id']);
```

> **特性**：`map()` 在 init 時就註冊，適合輕量工具函式。

### 重新映射原生方法

你可以重新映射 Flight 的原生方法。原方法會加上 `_` 前綴：

```php
// 重新映射 json() 來簡化參數順序
Flight::map('json', function ($data, $code = 200, $options = 0) {
    Flight::_json($data, $code, true, 'utf-8', $options);
});

// 使用：不再需要 true, 'utf-8' 參數
Flight::json(['id' => 123], 200, JSON_PRETTY_PRINT);
```

## register() — 註冊服務類別

延遲載入 + 單例模式。第一次呼叫才實例化。

```php
// register(string $name, string $class, array $params = [], callable $callback = null)
Flight::register('cache', 'App\Service\CacheService', [], function ($cache) {
    $cache->setTtl(3600);
});

// 使用（自動單例，第一次呼叫才實例化）
Flight::cache()->get('key');
```

## set() / get() / has() / clear() — 變數管理

```php
// 儲存設定值
Flight::set('app.version', '4.0');
Flight::set(['key1' => 'a', 'key2' => 'b']);  // 批量設定

// 讀取
$version = Flight::get('app.version');
$all = Flight::get();  // 取得全部

// 檢查
Flight::has('app.version'); // true

// 清除
Flight::clear('app.version');
Flight::clear();            // 清除全部
```

## before() / after() — 全域 Filter

可以在任何可覆寫方法前/後掛鉤：

```php
Flight::before('start', function (&$params, &$output) {
    // 每個請求開始前
});

Flight::after('json', function (&$params, &$output) {
    // 每次 json 輸出後
});
```

## 事件系統

Flight 內建的 `EventDispatcher`（Singleton 模式）：

```php
// onEvent(string $event, callable $callback)
Flight::onEvent('user.created', function ($userId) {
    // 處理事件
});

// triggerEvent(string $event, ...$args)
Flight::triggerEvent('user.created', $userId);
```

### 框架內建事件

| 事件名稱 | 參數 |
|----------|------|
| `flight.request.received` | Request |
| `flight.route.matched` | Route |
| `flight.route.executed` | Route, float $duration |
| `flight.middleware.executed` | Route, middleware, eventName, float $duration |
| `flight.middleware.before` | Route |
| `flight.middleware.after` | Route |
| `flight.error` | Throwable |
| `flight.redirect` | url, code |
| `flight.view.rendered` | file, float $duration |
| `flight.cache.checked` | type, bool $hit, float $duration |

## 框架內部配置選項

```php
// Engine::init() 中的預設值
Flight::set('flight.base_url', null);         // 基底 URL（子目錄部署時覆寫）
Flight::set('flight.case_sensitive', false);   // 路由大小寫敏感
Flight::set('flight.handle_errors', true);     // 錯誤處理（使用 Tracy 時設 false）
Flight::set('flight.log_errors', false);       // 記錄錯誤到 Web Server 的 error log
Flight::set('flight.views.path', './views');    // 模板路徑
Flight::set('flight.views.extension', '.php'); // 模板副檔名
Flight::set('flight.content_length', true);    // 自動 Content-Length（使用 Tracy 時設 false）
```

> **注意**：`Flight::set()` 存取的變數本質上是全域狀態，應節制使用。過多全域變數會使 bug 難以追蹤，也會使單元測試複雜化。

---

## 錯誤處理

### 全域錯誤攔截

當 `flight.handle_errors = true` 時，所有錯誤和例外都會被 Flight 捕獲並傳遞到 `error` 方法。

```php
// 覆寫預設的 500 錯誤處理
Flight::map('error', function (Throwable $error) {
    $code = $error->getCode() ?: 500;
    Flight::jsonHalt([
        'success' => false,
        'error'   => [
            'code'    => $code,
            'message' => $error->getMessage()
        ]
    ], $code);
});
```

### 404 Not Found

```php
Flight::map('notFound', function () {
    Flight::jsonHalt([
        'success' => false,
        'error'   => [
            'code'    => 404,
            'message' => '路由不存在'
        ]
    ], 404);
});
```

> **本專案最佳實踐**：在 `bootstrap.php` 中統一覆寫 `error` 和 `notFound`，確保所有錯誤回應都是統一 JSON 格式。

---

## 完整方法參考

### Core Methods（不可覆寫）

```php
Flight::map(string $name, callable $callback)       // 建立自訂方法
Flight::register(string $name, string $class, ...)  // 註冊類別
Flight::unregister(string $name)                    // 取消註冊
Flight::before(string $name, callable $callback)    // 方法前置 filter
Flight::after(string $name, callable $callback)     // 方法後置 filter
Flight::path(string $path)                          // 自動載入路徑
Flight::get(string $key)                            // 取得變數
Flight::set(string $key, mixed $value)              // 設定變數
Flight::has(string $key)                            // 檢查變數
Flight::clear(array|string $key = [])               // 清除變數
Flight::init()                                      // 初始化到預設值
Flight::app()                                       // 取得 Engine 實例
Flight::request()                                   // 取得 Request 物件
Flight::response()                                  // 取得 Response 物件
Flight::router()                                    // 取得 Router 物件
Flight::view()                                      // 取得 View 物件
```

### Extensible Methods（可覆寫、可加 filter）

```php
Flight::start()                                     // 啟動框架
Flight::stop()                                      // 停止框架並送出回應
Flight::halt(int $code = 200, string $message = '') // 立即停止
Flight::route(string $pattern, callable $callback)  // 路由
Flight::post/put/patch/delete(...)                  // HTTP 方法路由
Flight::group(string $pattern, callable $callback)  // 路由群組
Flight::getUrl(string $name, array $params = [])    // 路由別名 URL
Flight::redirect(string $url, int $code)            // 重導向
Flight::download(string $filePath)                  // 檔案下載
Flight::render(string $file, array $data, ...)      // 渲染模板
Flight::error(Throwable $error)                     // 500 錯誤處理
Flight::notFound()                                  // 404 處理
Flight::etag(string $id)                            // ETag 快取
Flight::lastModified(int $time)                     // Last-Modified 快取
Flight::json(mixed $data, int $code = 200, ...)     // JSON 輸出
Flight::jsonp(mixed $data, ...)                     // JSONP 輸出
Flight::jsonHalt(mixed $data, int $code = 200, ...) // JSON + halt
Flight::onEvent(string $event, callable $callback)  // 事件監聽
Flight::triggerEvent(string $event, ...$args)       // 觸發事件
```

> **⚠️ 注意**：用 `map()` 和 `register()` 新增的自訂方法也可以使用 `before()`/`after()` filter。
