---
name: kbnat-sql
description: kbnat 資料庫結構參考。當需要查詢資料表欄位、型別、索引、關聯，或撰寫 SQL / DAO 時觸發。
---

# kbnat 資料庫結構

因材網主資料庫 `kbnat` 的完整 Schema 參考（381 張資料表）。

## 資料來源

| 檔案 | 說明 |
|:-----|:-----|
| [kbnat.md](./kbnat.md) | phpMyAdmin SQL Dump，包含所有 CREATE TABLE 定義（欄位、型別、註解、索引、FK） |

## 使用方式

此檔案為完整 SQL Dump（~579KB），**不要一次讀取全部內容**。請依需求使用以下方式搜尋：

### 查詢特定資料表結構
```
Grep pattern: "資料表結構 `table_name`"
path: .claude/skills/kbnat-sql/kbnat.md
output_mode: content
-A: 50
```

### 查詢特定欄位出現在哪些資料表
```
Grep pattern: "`column_name`"
path: .claude/skills/kbnat-sql/kbnat.md
output_mode: content
-C: 3
```

### 查詢索引定義
```
Grep pattern: "資料表索引 `table_name`"
path: .claude/skills/kbnat-sql/kbnat.md
output_mode: content
-A: 30
```

### 列出所有資料表名稱
```
Grep pattern: "CREATE TABLE `"
path: .claude/skills/kbnat-sql/kbnat.md
output_mode: content
```

## 注意事項

- 資料庫版本：MySQL 8.0.26-google
- 字元集：utf8mb4
- 欄位註解（COMMENT）包含中文說明，是理解欄位用途的重要參考
- 撰寫 SQL 時務必使用 PDO prepared statements
- 讀取操作使用 `$dbh_slave`，寫入操作使用 `$dbh`
