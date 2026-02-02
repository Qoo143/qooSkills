# eduexam_paper

教育部大考考古題試題卷表，儲存大考考古題的試卷資訊。

## CREATE TABLE

```sql
CREATE TABLE `eduexam_paper` (
  `paper_sn` int unsigned NOT NULL AUTO_INCREMENT COMMENT '卷流水號',
  `year` tinyint unsigned NOT NULL COMMENT '學年度',
  `map_sn` int NOT NULL COMMENT '星空圖流水號',
  `exam_type` tinyint unsigned NOT NULL COMMENT '大考類型',
  `paper_name` varchar(100) NOT NULL COMMENT '卷名',
  `status` tinyint unsigned NOT NULL DEFAULT '0' COMMENT '試卷狀態: 0(未上鎖), 1(已上鎖), 2(已刪除), 3(已上鎖(但未公開))',
  `updater` varchar(60) DEFAULT NULL COMMENT '更新者',
  `creation_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '建立時間',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新時間',
  PRIMARY KEY (`paper_sn`),
  KEY `updater` (`updater`),
  KEY `exam_type` (`exam_type`),
  KEY `semester` (`year`),
  KEY `map_sn` (`map_sn`),
  KEY `status` (`status`),
  CONSTRAINT `eduexam_paper_ibfk_1` FOREIGN KEY (`updater`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL,
  CONSTRAINT `eduexam_paper_ibfk_2` FOREIGN KEY (`map_sn`) REFERENCES `map_info` (`map_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  CONSTRAINT `eduexam_paper_ibfk_3` FOREIGN KEY (`exam_type`) REFERENCES `exam_type` (`exam_type`) ON DELETE RESTRICT ON UPDATE RESTRICT
) ENGINE=InnoDB AUTO_INCREMENT=120121 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='教育部大考考古題-試題卷';
```
