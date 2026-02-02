# API Mapping Table

建立 API-V3 遷移對照表，分析舊 PHP 檔案的業務邏輯並規劃 Controller 對應。

## 執行方式

使用 `api-mapping-table` agent 執行深度分析。

## 步驟

1. 詢問用戶要分析的舊 PHP 檔案路徑
2. 讀取並分析舊檔案的：
   - 功能端點（函式或邏輯區塊）
   - 參數定義和驗證規則
   - SQL 查詢和資料表關聯
   - 權限檢查和安全性
3. 規劃 Controller 方法對應
4. 建立詳細的對照表於 `.claude/skills/api-v3/api-mapping/`
5. 輸出分析摘要和下一步建議

## 輸出

- 對照表檔案：`api-mapping-{filename}.md`
- 分析摘要：端點數量、Controller 建議、安全性問題
- 下一步：執行 `/api-write` 開始開發
