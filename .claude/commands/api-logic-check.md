# API Logic Check

深度檢查 API v3 與舊程式的邏輯一致性。

## 觸發

本指令啟動 `api-logic-check` agent，逐一比對舊程式與新 API 的 SQL、參數、業務邏輯、錯誤處理等,並生成詳細檢查報告。

## 前置條件

- 已完成 API v3 開發（執行過 `/api-write`）
- 舊程式檔案可存取

## 輸出位置

- `.claude/skills/api-v3/api-logicCheck/README.md` - 檢查總表
- `.claude/skills/api-v3/api-logicCheck/{controller}-{endpoint}.md` - 詳細檢查文檔

## 檢查範圍

每個端點檢查包含:
1. SQL 查詢比對
2. 參數驗證比對
3. 業務邏輯流程比對
4. 回應格式比對
5. 錯誤處理比對
6. 重複邏輯識別
7. 改進建議
8. 檢查結論
