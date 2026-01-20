# user_third_party

第三方帳號紀錄，記錄使用者綁定的各種第三方登入帳號（教育雲、Google、酷課雲、CK）。

## CREATE TABLE

```sql
CREATE TABLE `user_third_party` (
  `user_id` varchar(60) NOT NULL COMMENT '帳號',
  `cloud_sub` varchar(100) DEFAULT NULL COMMENT '教育雲第三方帳號',
  `google_sub` varchar(100) DEFAULT NULL COMMENT 'goole第三方帳號',
  `cooc_sub` varchar(100) DEFAULT NULL COMMENT '酷課雲綁定',
  `ck_sub` varchar(100) DEFAULT NULL COMMENT 'CK第三方帳號',
  `create_by_third` int NOT NULL COMMENT '1=由第三方創帳號的, 0=原有帳號連動第三方',
  PRIMARY KEY (`user_id`),
  KEY `create_by_third` (`create_by_third`),
  KEY `cloud_sub` (`cloud_sub`),
  KEY `ck_sub` (`ck_sub`),
  CONSTRAINT `user_third_party_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='第三方帳號紀錄'
```
