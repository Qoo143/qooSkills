# API Verify Mapping

驗證 API-V3 對照表的正確性，深度比對舊邏輯與規劃。

## 執行方式

使用 `api-verify-mapping` agent 進行 Gap Analysis。

## 驗證項目

1. **參數完整性**：對照表是否記錄所有參數
2. **SQL 查詢完整性**：JOIN、WHERE、特殊條件
3. **權限邏輯**：小組長、教師等特殊權限
4. **資料結構**：回傳欄位是否完整
5. **實作檢查**：Controller 是否涵蓋所有邏輯

## 輸出

驗證報告，包含：
- 🔴 CRITICAL（對照表遺漏重要邏輯）
- 🟡 WARNING（參數不完整、實作不完整）
- 🔵 SUGGESTION（補充說明建議）
- 📋 對照表更新建議
