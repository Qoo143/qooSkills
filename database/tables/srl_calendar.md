# srl_calendar

自主學習行事曆，儲存學生的學習行程與任務排程。

## 關聯

- `user_id` → `user_info.user_id` (CASCADE)
- `mission_sn` → `mission_info.mission_sn`

## CREATE TABLE

```sql
CREATE TABLE `srl_calendar` (
  `sn` int NOT NULL AUTO_INCREMENT,
  `user_id` varchar(60) NOT NULL COMMENT '學生帳號',
  `events` text NOT NULL COMMENT '行程',
  `start_time` datetime NOT NULL COMMENT '開始時間',
  `end_time` datetime NOT NULL COMMENT '結束時間',
  `all_day` tinyint(1) NOT NULL DEFAULT '0' COMMENT '全天(是:1；否:0)',
  `mission_sn` int DEFAULT NULL COMMENT '任務sn',
  `is_done` tinyint DEFAULT '0' COMMENT '0未完成 1完成',
  `is_delete` tinyint(1) NOT NULL DEFAULT '0' COMMENT '使用者是否刪除',
  `eventColor` tinyint unsigned NOT NULL COMMENT '行事曆顏色',
  PRIMARY KEY (`sn`),
  KEY `user_id` (`user_id`),
  KEY `all_day` (`all_day`),
  KEY `mission_sn` (`mission_sn`),
  KEY `is_done` (`is_done`),
  KEY `is_delete` (`is_delete`),
  KEY `eventColor` (`eventColor`),
  CONSTRAINT `srl_calendar_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  CONSTRAINT `srl_calendar_ibfk_2` FOREIGN KEY (`mission_sn`) REFERENCES `mission_info` (`mission_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT
) ENGINE=InnoDB AUTO_INCREMENT=199442 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
```
