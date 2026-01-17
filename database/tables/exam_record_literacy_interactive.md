# exam_record_literacy_interactive

互動式素養題測驗結果表，記錄學生互動式素養題的作答結果與 AI 評分。

## 關聯

- `user_id` → `user_info.user_id` (CASCADE)
- `mission_sn` → `mission_info.mission_sn`
- `map_sn` → `map_info.map_sn`
- `item_li_sn` → `concept_itemBank_literacy_interactive.item_li_sn`
- `subject_id` → `subject.subject_id`

## CREATE TABLE

```sql
CREATE TABLE `exam_record_literacy_interactive` (
  `exam_sn` bigint NOT NULL AUTO_INCREMENT,
  `mission_sn` int DEFAULT NULL COMMENT '任務sn,若無則NULL',
  `subject_id` smallint unsigned NOT NULL,
  `map_sn` int NOT NULL,
  `item_li_sn` int unsigned DEFAULT NULL COMMENT '互動式素養題編號',
  `user_id` varchar(50) NOT NULL,
  `start_time` datetime NOT NULL,
  `questions_type` varchar(255) NOT NULL COMMENT '  1:是非題   2:選擇題 1234   3:文字回答, 只要有文字題就分類為文字   4:複選 123456789   5:順序題(拖拉題) (拖拉順序:)   6:回傳base64圖檔',
  `client_items_idle_time` varchar(255) DEFAULT NULL COMMENT '作答時間(毫秒)',
  `stuAns` longtext NOT NULL COMMENT '根據作答順序紀錄',
  `binary_res` mediumtext NOT NULL COMMENT '根據作答順序紀錄',
  `ai_feedback` json DEFAULT NULL COMMENT 'AI評分結果',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`exam_sn`),
  KEY `user_id` (`user_id`),
  KEY `mission_sn` (`mission_sn`),
  KEY `date` (`update_time`),
  KEY `map_sn` (`map_sn`),
  KEY `item_li_sn` (`item_li_sn`),
  KEY `start_time` (`start_time`),
  KEY `client_items_idle_time` (`client_items_idle_time`),
  KEY `subject_id` (`subject_id`),
  CONSTRAINT `__exam_record_literacy_interactive_ibfk_6` FOREIGN KEY (`subject_id`) REFERENCES `subject` (`subject_id`),
  CONSTRAINT `exam_record_literacy_interactive_ibfk_1` FOREIGN KEY (`mission_sn`) REFERENCES `mission_info` (`mission_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  CONSTRAINT `exam_record_literacy_interactive_ibfk_3` FOREIGN KEY (`map_sn`) REFERENCES `map_info` (`map_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  CONSTRAINT `exam_record_literacy_interactive_ibfk_4` FOREIGN KEY (`item_li_sn`) REFERENCES `concept_itemBank_literacy_interactive` (`item_li_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  CONSTRAINT `exam_record_literacy_interactive_ibfk_5` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE
) ENGINE=InnoDB AUTO_INCREMENT=812307 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
```
