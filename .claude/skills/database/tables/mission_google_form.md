# mission_google_form

Google 表單問卷回應表，儲存學生填寫問卷的回應資料。

## CREATE TABLE

```sql
CREATE TABLE `mission_google_form` (
  `sn` int NOT NULL AUTO_INCREMENT,
  `form_type` bigint unsigned DEFAULT NULL COMMENT 'mission_google_master的SN',
  `seme` varchar(6) NOT NULL,
  `user_id` varchar(60) DEFAULT NULL,
  `city` varchar(16) DEFAULT NULL,
  `org_name` varchar(40) DEFAULT NULL,
  `grade` int NOT NULL,
  `class` int NOT NULL,
  `stu_name` varchar(50) NOT NULL,
  `stu_sex` varchar(2) NOT NULL,
  `content` longtext CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci,
  `start_time` datetime NOT NULL COMMENT '開始問卷的時間',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '結束時間',
  `mission_sn` int DEFAULT NULL,
  `org_id` varchar(10) DEFAULT NULL COMMENT '學校代號',
  PRIMARY KEY (`sn`),
  KEY `form_type` (`form_type`),
  KEY `user_id` (`user_id`),
  KEY `city` (`city`),
  KEY `org_name` (`org_name`),
  KEY `grade` (`grade`),
  KEY `class` (`class`),
  KEY `stu_name` (`stu_name`),
  KEY `start_time` (`start_time`),
  KEY `create_time` (`create_time`),
  KEY `seme` (`seme`),
  KEY `stu_sex` (`stu_sex`),
  KEY `mission_sn` (`mission_sn`),
  KEY `org_id` (`org_id`),
  CONSTRAINT `mission_google_form_ibfk_1` FOREIGN KEY (`form_type`) REFERENCES `mission_google_master` (`sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  CONSTRAINT `mission_google_form_ibfk_2` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL,
  CONSTRAINT `mission_google_form_ibfk_3` FOREIGN KEY (`mission_sn`) REFERENCES `mission_info` (`mission_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  CONSTRAINT `mission_google_form_ibfk_4` FOREIGN KEY (`org_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT
) ENGINE=InnoDB AUTO_INCREMENT=774467 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
```
