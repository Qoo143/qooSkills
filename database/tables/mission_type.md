# mission_type

任務類型表，定義系統中的各種任務類型。

## 關聯

被參照：
- `mission_info.mission_type` → `mission_type.mission_type_id`

## CREATE TABLE

```sql
CREATE TABLE `mission_type` (
  `sn` tinyint unsigned NOT NULL AUTO_INCREMENT,
  `mission_type_id` tinyint unsigned NOT NULL COMMENT 'mission_type 代號',
  `name` varchar(20) NOT NULL COMMENT '科目名稱',
  `mission_icon` mediumtext COMMENT '任務圖示',
  `unable` int NOT NULL DEFAULT '1' COMMENT '顯示: 1; 不顯示: 0',
  PRIMARY KEY (`sn`),
  KEY `mission_type_id` (`mission_type_id`) USING BTREE,
  KEY `unable` (`unable`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=21 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='學科名稱';
```
