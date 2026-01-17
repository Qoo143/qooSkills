# exam_record_indicate

縱貫測驗結果表，記錄學生縱貫式測驗的作答結果與分析資料。

## 關聯

- `user_id` → `user_info.user_id` (CASCADE)
- `result_sn` → `mission_info.mission_sn`

## CREATE TABLE

```sql
CREATE TABLE `exam_record_indicate` (
  `exam_sn` bigint NOT NULL AUTO_INCREMENT,
  `user_id` varchar(60) DEFAULT NULL,
  `cs_id` varchar(15) NOT NULL,
  `date` datetime NOT NULL,
  `time` time NOT NULL COMMENT '時:分',
  `start_time` varchar(14) NOT NULL,
  `stop_time` varchar(14) NOT NULL,
  `during_time` varchar(16) NOT NULL COMMENT '毫秒',
  `score` tinyint unsigned DEFAULT NULL COMMENT '正確率',
  `snode_rate` tinyint unsigned DEFAULT NULL COMMENT '小節點正確率',
  `client_items_idle_time` mediumtext COMMENT '作答時間(毫秒)',
  `questions` mediumtext NOT NULL,
  `stuAns` mediumtext NOT NULL COMMENT '原始作答情形(根據作答順序紀錄)',
  `binary_res` mediumtext NOT NULL COMMENT '二元作答情形(根據作答順序紀錄)',
  `skill_remedy_rate_s` mediumtext NOT NULL COMMENT '小節點(小節點(-1: 還沒做, 0: 錯, 1: 對, 2:預測對, 3: 預測對但做錯, 4:預測對也做對))',
  `skill_remedy_rate_b` mediumtext NOT NULL COMMENT '大節點內精熟的小節點數',
  `result_sn` int DEFAULT NULL COMMENT 'mission_sn',
  `seme_year` smallint unsigned NOT NULL COMMENT '學年度',
  `reward_mk` char(1) NOT NULL DEFAULT 'N' COMMENT '是否已計算獎勵',
  PRIMARY KEY (`exam_sn`),
  KEY `date` (`date`),
  KEY `user_id` (`user_id`),
  KEY `result_sn` (`result_sn`),
  KEY `cs_id` (`cs_id`),
  KEY `seme_year` (`seme_year`),
  KEY `reward_mk` (`reward_mk`),
  CONSTRAINT `exam_record_indicate_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  CONSTRAINT `exam_record_indicate_ibfk_2` FOREIGN KEY (`result_sn`) REFERENCES `mission_info` (`mission_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT
) ENGINE=InnoDB AUTO_INCREMENT=2759118 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='縱貫測驗結果';
```
