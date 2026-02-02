# API Review

對 API-V3 程式碼進行全面審查，確保符合架構規範和安全標準。

## 執行方式

使用 `api-review` agent 進行系統化審查。

## 審查項目

### 分層職責
- Controller：只處理請求，不包含業務邏輯
- Service：包含業務邏輯，使用 Read-before-Write
- DAO：只操作資料庫，使用 Prepared Statements

### 安全性（CRITICAL）
- SQL 注入檢查
- XSS 防護（echo 必須經過 preventXss()）
- 權限驗證
- 敏感資料保護

### 錯誤處理
- Controller 不使用 try-catch
- Service 使用 AppException
- DAO 讓 PDOException 冒泡

## 輸出

審查報告，包含：
- 🔴 CRITICAL（必須修正）
- 🟡 WARNING（應該修正）
- 🔵 SUGGESTION（建議改進）
- ✅ 通過檢查項目
