# mission_stud_record

學生任務狀態表，記錄學生對每個任務的完成進度與狀態。

## CREATE TABLE

```sql
CREATE TABLE `mission_stud_record` (
  `stu_mission_sn` int NOT NULL AUTO_INCREMENT,
  `mission_sn` int NOT NULL COMMENT '任務sn',
  `user_id` varchar(60) NOT NULL COMMENT '學生id',
  `finish_node` mediumtext NOT NULL COMMENT '已完成的節點',
  `finish_node_count` tinyint unsigned NOT NULL DEFAULT '0' COMMENT '已完成的節點數量',
  `action_compelete_count` mediumtext NOT NULL COMMENT '每個學習任務完成次數',
  `is_all_fin` int NOT NULL COMMENT '是全部完成',
  `is_get_extra_coin` tinyint NOT NULL DEFAULT '0' COMMENT '標記是否已取得額外的代幣',
  `finish_time` datetime NOT NULL COMMENT '完成時間',
  `update_time` datetime NOT NULL ON UPDATE CURRENT_TIMESTAMP COMMENT '最後更新時間',
  `semester` int NOT NULL COMMENT '學年度',
  PRIMARY KEY (`stu_mission_sn`),
  KEY `mission_sn` (`mission_sn`),
  KEY `user_id` (`user_id`),
  KEY `semester` (`semester`),
  KEY `is_all_fin` (`is_all_fin`),
  KEY `is_get_extra_coin` (`is_get_extra_coin`),
  CONSTRAINT `mission_stud_record_ibfk_1` FOREIGN KEY (`mission_sn`) REFERENCES `mission_info` (`mission_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  CONSTRAINT `mission_stud_record_ibfk_2` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE
) ENGINE=InnoDB AUTO_INCREMENT=78450942 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci ROW_FORMAT=COMPACT COMMENT='學生的任務狀態';
```
