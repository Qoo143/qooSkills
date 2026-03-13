# 回應系統指南

> 參考來源：[FlightPHP Responses 官方文檔](https://docs.flightphp.com/learn/responses)

## JSON 回應

```php
// json(mixed $data, int $code = 200, bool $encode = true, string $charset = 'utf-8', int $option = 0)
Flight::json(['success' => true, 'data' => $data]);
Flight::json($data, 201);  // 指定狀態碼

// 預設 header: Content-Type: application/json
// 預設 flags: JSON_THROW_ON_ERROR | JSON_UNESCAPED_SLASHES
```

### ⚠️ json() 不會中止執行

```php
// ❌ 危險：json() 不會中止，後面的程式碼仍會執行
Flight::json(['error' => 'bad'], 400);
echo '這行仍會執行!';  // ← 會執行

// ✅ 正確做法 1：使用 jsonHalt()（v3.10.0+）
Flight::jsonHalt(['error' => 'bad'], 400);
echo '這行不會執行';  // ← 不會執行

// ✅ 正確做法 2：json() + return
if ($error) {
    Flight::json(['error' => $msg], 400);
    return;
}
```

### jsonHalt() — JSON 輸出 + 立即中止

`jsonHalt()` = `json()` + `halt()`，會清除既有的 body 內容並立即停止執行。

```php
Flight::route('/users', function () {
    $authorized = someAuthorizationCheck();
    if ($authorized === false) {
        Flight::jsonHalt(['error' => 'Unauthorized'], 401);
        // 不需要 exit; 也不需要 return
    }

    // 繼續路由的剩餘邏輯
});
```

## 停止執行

```php
// halt() — 丟棄既有回應內容，立即停止
Flight::halt(403, '禁止存取');

// stop() — 輸出當前回應後停止（⚠️ 行為特殊，建議用 halt）
Flight::stop();
```

> **注意**：`Flight::stop()` 會輸出回應但腳本可能繼續執行。建議使用 `Flight::halt()`。

## 重導向

```php
// redirect(string $url, int $code = 303)
Flight::redirect('/login', 302);
```

## 檔案下載

```php
// download(string $filePath, ?string $filename = null)（v3.12.0+）
Flight::download('/path/to/file.pdf');
Flight::download('/path/to/file.pdf', 'custom_name.pdf'); // v3.17.1+
```

## JSONP 回應

```php
// jsonp(mixed $data, string $param = 'jsonp', int $code = 200, ...)
Flight::jsonp($data, 'callback');
// GET ?callback=myFunc → myFunc({"id":123});
```

## 直接操作 Response 物件

```php
Flight::response()->write('Hello');
Flight::response()->status(200);
Flight::response()->header('X-Custom', 'value');
Flight::response()->send();

// 清除
Flight::response()->clearBody();  // 只清 body
Flight::response()->clear();      // 清 body + headers，重設 200
```

## HTTP 快取

```php
// 路由級快取
Flight::response()->cache(time() + 300);    // 快取 5 分鐘
Flight::response()->cache('+5 minutes');    // 同上（strtotime 格式）
Flight::response()->cache(false);           // 不快取

// Last-Modified
Flight::lastModified(filemtime($file));

// ETag
Flight::etag('unique-id');
```

> `lastModified` 和 `etag` 會同時設定和檢查快取值。如果快取值相同，Flight 會立即回傳 `HTTP 304` 並停止處理。

## 回應格式 Body 輸出方式

Flight 使用 `ob_start()` 緩衝輸出，所以可以使用 `echo` 或 `print`：

```php
Flight::route('/', function () {
    echo "Hello, World!"; // Flight 會捕獲並加上適當 header
});
```

也可以用 `write()` 方法：

```php
Flight::route('/', function () {
    Flight::response()->write("Hello, World!");

    // 取回已設定的 body
    $body = Flight::response()->getBody();
});
```

## ⚠️ 回應常見錯誤

```php
// ❌ 錯誤：直接 echo JSON
echo json_encode($data);

// ✅ 正確：使用框架方法
Flight::json($data);
// 或在 Controller 中：
$this->app->json($data);
```
