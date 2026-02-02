# API Write

根據對照表撰寫 API-V3 標準化程式碼。

## 執行方式

使用 `api-write` agent 按照 DAO → Service → Controller 順序開發。

## 開發順序

1. **DAO 層**：資料存取，使用 Prepared Statements
2. **Service 層**：業務邏輯，使用 Read-before-Write 模式
3. **Controller 層**：請求處理，使用 guard() 和 validate()

## 輸出

- DAO 檔案：`ADLAPI/v3/App/Dao/XxxDao.php`
- Service 檔案：`ADLAPI/v3/App/Service/XxxService.php`
- Controller 檔案：`ADLAPI/v3/App/Controller/XxxController.php`
- 對照表更新：狀態改為 ✅
- 測試指令：curl 範例
