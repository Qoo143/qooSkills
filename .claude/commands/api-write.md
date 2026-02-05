# API Write

根據對照表撰寫 API-V3 程式碼。

## 觸發

本指令啟動 `api-write` agent，依序產生 DAO、Service、Controller 層程式碼。

## 前置條件

- 已完成對照表（執行過 `/api-mapping-table`）
- 已確認資料表結構（執行過 `/database`）

## 輸出位置

- `ADLAPI/v3/App/Dao/XxxDao.php`
- `ADLAPI/v3/App/Service/XxxService.php`
- `ADLAPI/v3/App/Controller/XxxController.php`
