# API Verify Mapping

驗證對照表與實作的一致性。

## 觸發

本指令啟動 `api-verify-mapping` agent，比對舊邏輯與新 Controller 實作。

## 前置條件

- 已完成對照表（執行過 `/api-mapping-table`）
- 已完成部分或全部實作（執行過 `/api-write`）

## 輸出格式

Gap Analysis 報告，包含遺漏項目與對照表更新建議
