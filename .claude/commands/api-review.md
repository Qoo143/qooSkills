# API Review

對 API-V3 程式碼進行系統化審查。

## 觸發

本指令啟動 `api-review` agent，檢查架構合規性、安全性及程式碼品質。

## 前置條件

- 已完成程式碼撰寫（執行過 `/api-write`）
- 指定待審查的 Controller 或目錄

## 輸出格式

審查報告，依嚴重程度分類：CRITICAL、WARNING、SUGGESTION、PASSED
