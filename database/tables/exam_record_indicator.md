# exam_record_indicator

單元測驗結果表，記錄學生單元式測驗的作答結果與分析資料。

## CREATE TABLE

```sql
CREATE TABLE `exam_record_indicator` (
  `exam_sn` bigint NOT NULL AUTO_INCREMENT,
  `user_id` varchar(60) NOT NULL,
  `paper_sn` bigint unsigned DEFAULT NULL,
  `date` datetime NOT NULL,
  `semester` int unsigned DEFAULT NULL COMMENT '學年度學期',
  `start_time` varchar(14) NOT NULL,
  `stop_time` varchar(14) NOT NULL,
  `during_time` varchar(16) NOT NULL,
  `score` tinyint unsigned DEFAULT NULL COMMENT '正確率',
  `snode_rate` tinyint unsigned DEFAULT NULL COMMENT '小節點正確率',
  `client_items_idle_time` mediumtext COMMENT '作答時間(毫秒)',
  `questions` mediumtext NOT NULL,
  `stuAns` mediumtext NOT NULL COMMENT '根據作答順序紀錄',
  `binary_res` mediumtext NOT NULL COMMENT '根據作答順序紀錄',
  `skill_remedy_rate_s` mediumtext NOT NULL COMMENT '小節點(0:待補救，1:精熟)',
  `skill_remedy_rate_b` mediumtext NOT NULL COMMENT '大節點內精熟的小節點數',
  `map_sn` int NOT NULL,
  `mission_sn` int DEFAULT NULL,
  `reward_mk` char(1) NOT NULL DEFAULT 'N' COMMENT '是否已計算獎勵',
  PRIMARY KEY (`exam_sn`),
  KEY `user_id` (`user_id`),
  KEY `paper_sn` (`paper_sn`),
  KEY `mission_sn` (`mission_sn`),
  KEY `date` (`date`),
  KEY `map_sn` (`map_sn`),
  KEY `semester` (`semester`),
  CONSTRAINT `exam_record_indicator_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  CONSTRAINT `exam_record_indicator_ibfk_2` FOREIGN KEY (`mission_sn`) REFERENCES `mission_info` (`mission_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  CONSTRAINT `exam_record_indicator_ibfk_3` FOREIGN KEY (`paper_sn`) REFERENCES `concept_paper` (`paper_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  CONSTRAINT `exam_record_indicator_ibfk_4` FOREIGN KEY (`map_sn`) REFERENCES `map_info` (`map_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT
) ENGINE=InnoDB AUTO_INCREMENT=14376701 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='單元測驗結果';
```
