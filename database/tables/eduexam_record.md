# eduexam_record

教育部大考考古題測驗紀錄表，記錄學生大考考古題的作答紀錄。

## CREATE TABLE

```sql
CREATE TABLE `eduexam_record` (
  `record_sn` int unsigned NOT NULL AUTO_INCREMENT COMMENT '紀錄流水號',
  `user_id` varchar(60) DEFAULT NULL COMMENT '使用者ID',
  `paper_sn` int unsigned NOT NULL COMMENT '卷流水號',
  `mission_sn` int DEFAULT NULL COMMENT '任務流水號',
  `is_finished` tinyint unsigned NOT NULL DEFAULT '0' COMMENT '是否已完成',
  `finished_time` datetime DEFAULT NULL COMMENT '完成時間',
  `creation_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '建立時間',
  PRIMARY KEY (`record_sn`),
  KEY `user_id` (`user_id`),
  KEY `mission_sn` (`mission_sn`),
  KEY `is_finished` (`is_finished`),
  KEY `paper_sn` (`paper_sn`),
  KEY `idx_papersn_isfinished_userid` (`paper_sn`,`is_finished`,`user_id`) USING BTREE,
  CONSTRAINT `eduexam_record_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL,
  CONSTRAINT `eduexam_record_ibfk_2` FOREIGN KEY (`mission_sn`) REFERENCES `mission_info` (`mission_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  CONSTRAINT `eduexam_record_ibfk_3` FOREIGN KEY (`paper_sn`) REFERENCES `eduexam_paper` (`paper_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT
) ENGINE=InnoDB AUTO_INCREMENT=2586272 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='教育部大考考古題-測驗紀錄';
```
