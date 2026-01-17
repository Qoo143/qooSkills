# seme_student

學生學期資料表，記錄每學期學生的學校、年級、班級資訊。

## CREATE TABLE

```sql
CREATE TABLE `seme_student` (
  `sn` int unsigned NOT NULL AUTO_INCREMENT,
  `seme_year_seme` varchar(6) NOT NULL DEFAULT '' COMMENT '學年度學期',
  `organization_id` varchar(10) NOT NULL COMMENT '學校代號',
  `stud_id` varchar(60) NOT NULL COMMENT '學生帳號',
  `grade` int unsigned NOT NULL COMMENT '年級',
  `class` int unsigned NOT NULL COMMENT '班級',
  `seme_class_name` varchar(10) DEFAULT NULL COMMENT '班級中文名稱',
  `seme_num` tinyint unsigned DEFAULT NULL COMMENT '學生座號',
  `uid` int unsigned NOT NULL DEFAULT '0' COMMENT '學生帳號之流水號',
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`sn`),
  UNIQUE KEY `uk_seme_uid` (`seme_year_seme`,`uid`) USING BTREE,
  UNIQUE KEY `uk_seme_studid` (`seme_year_seme`,`stud_id`) USING BTREE,
  KEY `organization_id` (`organization_id`),
  KEY `CON_userid` (`stud_id`),
  KEY `grade` (`grade`),
  KEY `class` (`class`),
  KEY `seme_year_seme` (`seme_year_seme`),
  KEY `idx_seme_organizationid` (`seme_year_seme`,`organization_id`) USING BTREE,
  KEY `idx_seme_organizationid_grade_class` (`seme_year_seme`,`organization_id`,`grade`,`class`),
  KEY `idx_organizationid_grade_class` (`organization_id`,`grade`,`class`) USING BTREE,
  KEY `update_time` (`update_time`),
  KEY `update_time_2` (`update_time`),
  KEY `update_time_3` (`update_time`),
  KEY `stud_id_seme_year_seme` (`stud_id`,`seme_year_seme`),
  CONSTRAINT `_seme_student_ibfk_1` FOREIGN KEY (`organization_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  CONSTRAINT `_seme_student_ibfk_2` FOREIGN KEY (`stud_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE
) ENGINE=InnoDB AUTO_INCREMENT=34325608 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='學生學期';
```
