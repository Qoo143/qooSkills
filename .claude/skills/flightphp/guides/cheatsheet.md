# FlightPHP API 速查卡

> AI 撰寫程式碼時的快速參考。所有方法均可透過 `Flight::xxx()` 靜態呼叫或 `$this->app->xxx()` 實例呼叫。

## 路由

| 方法 | 簽名 | 說明 |
|------|------|------|
| `route` | `(string $pattern, callable\|string\|array $callback, bool $pass_route = false, string $alias = ''): Route` | 通用路由 |
| `get` | `(string $pattern, $callback, bool $pass_route = false, string $alias = ''): Route` | GET 路由 |
| `post` | `(string $pattern, $callback, ...): Route` | POST 路由 |
| `put` | `(string $pattern, $callback, ...): Route` | PUT 路由 |
| `patch` | `(string $pattern, $callback, ...): Route` | PATCH 路由 |
| `delete` | `(string $pattern, $callback, ...): Route` | DELETE 路由 |
| `group` | `(string $prefix, callable $callback, array $middlewares = []): void` | 路由群組 |
| `resource` | `(string $pattern, string $controllerClass, array $options = []): void` | RESTful 資源 |
| `getUrl` | `(string $alias, array $params = []): string` | 從別名取 URL |

## 回應

| 方法 | 簽名 | 行為 |
|------|------|------|
| `json` | `($data, int $code = 200, bool $encode = true, string $charset = 'utf-8', int $option = 0): void` | JSON 輸出，**不中止** |
| `jsonHalt` | `($data, int $code = 200, ...): void` | JSON 輸出 + **立即中止** |
| `jsonp` | `($data, string $param = 'jsonp', int $code = 200, ...): void` | JSONP 輸出 |
| `halt` | `(int $code = 200, string $message = ''): void` | 停止執行 |
| `redirect` | `(string $url, int $code = 303): void` | 重導向 |
| `download` | `(string $filePath): void` | 檔案下載 |
| `render` | `(string $file, ?array $data = null, ?string $key = null): void` | 模板渲染 |

## 框架擴充

| 方法 | 簽名 | 說明 |
|------|------|------|
| `map` | `(string $name, callable $callback): void` | 映射自訂方法 |
| `register` | `(string $name, string $class, array $params = [], ?callable $callback = null): void` | 註冊服務（延遲載入+單例） |
| `before` | `(string $name, callable $callback): void` | 前置 filter |
| `after` | `(string $name, callable $callback): void` | 後置 filter |
| `set` | `(string\|iterable $key, $value = null): void` | 設定變數 |
| `get` | `(?string $key = null): mixed` | 取得變數 |
| `has` | `(string $key): bool` | 檢查變數 |
| `clear` | `(?string $key = null): void` | 清除變數 |
| `path` | `(string $dir): void` | 加入 autoload 路徑 |

## 事件

| 方法 | 簽名 |
|------|------|
| `onEvent` | `(string $event, callable $callback): void` |
| `triggerEvent` | `(string $event, ...$args): void` |

## 請求 (Request)

| 存取方式 | 型別 | 說明 |
|----------|------|------|
| `->query` | Collection | GET 參數 `$_GET` |
| `->data` | Collection | POST 表單/JSON body |
| `->cookies` | Collection | Cookie |
| `->files` | Collection | 上傳檔案 |
| `->method` | string | HTTP 方法 |
| `->url` | string | 請求路徑 |
| `->ip` | string | 客戶端 IP |
| `->type` | string | Content-Type |
| `->getBody()` | string | 原始 body |
| `->getHeader(name)` | string | 取 header |
| `->getHeaders()` | array | 所有 headers |
| `->getFullUrl()` | string | 完整 URL |
| `->getUploadedFiles()` | array | UploadedFile 陣列 |

## 回應 (Response)

| 方法 | 說明 |
|------|------|
| `->status(int $code)` | 設定 HTTP 狀態碼 |
| `->header(name, value)` | 設定 header |
| `->write(string $str)` | 寫入 body |
| `->clearBody()` | 清除 body |
| `->cache($expires)` | 設定快取 |
| `->send()` | 送出回應 |

## SimplePdo

| 方法 | 回傳 | 說明 |
|------|------|------|
| `fetchRow($sql, $params)` | `?Collection` | 取單行 |
| `fetchAll($sql, $params)` | `array<Collection>` | 取所有行 |
| `fetchColumn($sql, $params)` | `array<mixed>` | 取單欄 |
| `fetchPairs($sql, $params)` | `array<key, value>` | 取鍵值對 |
| `runQuery($sql, $params)` | `PDOStatement` | 執行查詢 |
| `insert($table, $data)` | `string` | 插入，回傳 lastInsertId |
| `update($table, $data, $where, $params)` | `int` | 更新，回傳影響行數 |
| `delete($table, $where, $params)` | `int` | 刪除，回傳影響行數 |
| `transaction(callable)` | `mixed` | 交易 |
