# 經驗教訓 (Lessons Learned)

## 2026-02-24: PHP 7.4 相容性 — 禁止 Union Types

**問題**: `CalendarRepository.php` 第 175 行使用了 `): array|false`，這是 PHP 8.0+ 的 union type 語法，導致伺服器（PHP 7.4）解析錯誤。

**錯誤訊息**: `syntax error, unexpected '|', expecting ';' or '{'`

**根因**: AI 生成程式碼時未考慮伺服器 PHP 版本限制。

**修復**: 移除 return type declaration，改用 PHPDoc `@return array|false`。

**防護措施**:
- 已更新 `anti-patterns.md` 加入 PHP 8.0+ 語法檢查項
- `project-architecture.md` 第 351-355 行已有明確說明
- 產出任何 PHP 程式碼前，必須確認不使用：union types、match、enum、readonly、named arguments

## 2026-02-24: 嚴格校對 SQL 欄位名稱與 Schema 一致性

**問題**: 在 `ParentRepository` 查詢中使用了不存在於 `user_info` 的 `class_name` 欄位；在 `StudentMissionRepository` 中將 `endYear` 誤寫為 `endyear`。

**影響**: 導致 `SQLSTATE[42S22]: Column not found` 錯誤，且在 Linux 環境下可能因大小寫不匹配導致隱性錯誤。

**修復**: 
- 修正 `StudentMissionRepository` 的欄位大小寫。
- 移除 `ParentRepository` 中的不存在欄位。

**預防措施**:
1. 撰寫 SQL 前必先查閱 `kbnat.md` 全域 Schema 文件。
2. 注意欄位的大小寫細節（尤其是 `sn`、`endYear`、`eventColor` 等）。
3. 全域搜尋關鍵欄位，確保所有 Repository 對同一資料表的存取邏輯一致。

## 2026-03-11: PHP 全域變數慣例 — 使用 global 不傳參

**問題**: 新建 `prodb_mission_share.php` 時將 `$dbh`、`$dbh_slave` 作為函式參數傳入，與專案慣例不一致。

**專案慣例**: 所有既有 prodb 檔案（如 `assign_mission_prodb.php`）皆在函式內使用 `global $dbh` / `global $dbh_slave`。

**修復**: 移除函式參數，改用 `global` 關鍵字。

**防護措施**:
- 新建 prodb 檔案前，先 `grep "global \$dbh"` 確認同目錄慣例
- 函式簽名不傳資料庫連線物件

## 2026-03-11: 前端傳來的 ID 皆為 Hash 編碼值

**問題**: `prodb_mission_share.php` 的 `getSharedStaff()` 和 `updateShareStaff()` 用 `intval($_POST['mission_sn'])` 接收前端傳來的 mission_sn，但實際上是 hashed 值，導致解碼失敗永遠為 0。

**根因**: 未查看同模組既有 prodb 如何處理同一參數。`modules_mission_edit_prodb.php` 第 34 行明確使用 `hash2string($_POST['missionSn'], false)`。

**修復**: 改為 `intval(hash2string($_POST['mission_sn'], false))`。

**防護措施**:
1. 新函式接收前端 POST 的 ID 參數時，必須查看前端傳出的變數來源
2. 若前端變數來自 PHP echo 的 `string2hash()` 或 `preventXss($_GET[...])` 取得的 URL 參數，後端就必須 `hash2string()` 解碼
3. 比對同模組既有 prodb 的處理方式

## 2026-03-11: turnpage.php AJAX 路徑不加前綴

**問題**: 新增 AJAX 呼叫使用 `../../turnpage.php`，但頁面透過 `modules_new.php` 載入後工作目錄在專案根目錄。

**修復**: 改為 `turnpage.php`（無前綴）。

**防護措施**: 查看同檔案中其他 `$.post` / `$.ajax` 的路徑寫法。

## 2026-03-11: SQL 不寫顯示邏輯，前端用對照表

**問題**: 在 SQL 中用 `CONCAT('科任教師（', subject_name, '）') AS role` 和 `CASE WHEN` 組合中文標籤。

**更好做法**: SQL 只回傳原始欄位（`access_level`、`subject_name`），前端建立 `SHARE_ROLE_LABELS` 對照表轉換顯示名稱。

**原因**: 擴充角色時只改前端對照表，不需動 SQL。後端職責是查資料，不管顯示。

## 2026-03-11: 命名應考慮涵蓋範圍

**問題**: 變數/表名使用 `teacher`（如 `shared_teacher_ids`），但對象包含主任、校長等非教師身分。

**修復**: 全面改為 `staff`（如 `shared_staff_ids`、`mission_share_staff`）。

**防護措施**: 命名前先列出所有涵蓋對象，選擇能包含全部的泛化名稱。

## 2026-03-11: 新程式碼用 let，既有 var 不動

**問題**: 新增變數用 `var` 宣告（為了一致性），但使用者要求新碼用 `let`。

**規則**: 只改自己新寫的宣告為 `let`，不動既有的 `var`。
