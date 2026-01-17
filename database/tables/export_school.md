# export_school

轉出入登記表，記錄學生跨校轉出轉入的歷程資料。

## 關聯

- `org_id` → `user_info.user_id` (SET NULL)
- `new_id` → `user_info.user_id` (SET NULL)
- `org_school` → `organization.organization_id`
- `new_school` → `organization.organization_id`
- `exporter` → `user_info.user_id` (SET NULL)
- `user_level` → `user_access.access_level`

## CREATE TABLE

```sql
CREATE TABLE `export_school` (
  `sn` int NOT NULL AUTO_INCREMENT COMMENT '流水號',
  `org_id` varchar(60) DEFAULT NULL COMMENT '原 user_id',
  `org_school` varchar(10) DEFAULT NULL COMMENT '原學校代碼',
  `export_seme` varchar(6) NOT NULL COMMENT '轉出學年度',
  `export_date` varchar(10) NOT NULL COMMENT '轉出日期',
  `exporter` varchar(60) DEFAULT NULL COMMENT '轉出人',
  `user_level` int NOT NULL COMMENT 'user 等級',
  `new_id` varchar(60) DEFAULT NULL COMMENT '新 user_id',
  `new_school` varchar(10) DEFAULT NULL COMMENT '新學校代碼',
  `import_seme` varchar(6) DEFAULT NULL COMMENT '轉入學年度',
  `import_date` varchar(10) DEFAULT NULL COMMENT '轉入日期',
  PRIMARY KEY (`sn`),
  KEY `org_id` (`org_id`),
  KEY `org_school` (`org_school`),
  KEY `new_id` (`new_id`),
  KEY `new_school` (`new_school`),
  KEY `import_seme` (`import_seme`),
  KEY `export_seme` (`export_seme`),
  KEY `export_date` (`export_date`),
  KEY `exporter` (`exporter`),
  KEY `user_level` (`user_level`),
  KEY `import_date` (`import_date`),
  CONSTRAINT `export_school_ibfk_1` FOREIGN KEY (`org_id`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL,
  CONSTRAINT `export_school_ibfk_2` FOREIGN KEY (`new_id`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL,
  CONSTRAINT `export_school_ibfk_3` FOREIGN KEY (`org_school`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  CONSTRAINT `export_school_ibfk_4` FOREIGN KEY (`new_school`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  CONSTRAINT `export_school_ibfk_5` FOREIGN KEY (`exporter`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL,
  CONSTRAINT `export_school_ibfk_6` FOREIGN KEY (`user_level`) REFERENCES `user_access` (`access_level`) ON DELETE RESTRICT ON UPDATE RESTRICT
) ENGINE=InnoDB AUTO_INCREMENT=149256 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='轉出入登記';
```
