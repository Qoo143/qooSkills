# API Mapping Table

建立 API-V3 遷移對照表。

## 觸發

本指令啟動 `api-mapping-table` agent，執行舊 PHP 檔案分析與對照表建立。

## 前置條件

- 確認待分析的舊 PHP 檔案路徑
- 建議先執行 `/database` 了解相關資料表結構

## 輸出位置

`.claude/skills/api-v3/api-mapping/api-mapping-{filename}.md`
