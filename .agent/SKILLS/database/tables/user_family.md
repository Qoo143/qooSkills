# user_family

學生家長名單，記錄使用者與家長帳號的綁定關係。

## CREATE TABLE

```sql
CREATE TABLE `user_family` (
  `sn` int NOT NULL AUTO_INCREMENT COMMENT '流水號',
  `organization_id` varchar(10) DEFAULT NULL COMMENT '學校代碼',
  `user_id` varchar(60) NOT NULL COMMENT '使用者id',
  `fuser_id` varchar(60) NOT NULL COMMENT '家長id',
  `create_date` datetime NOT NULL COMMENT '建立日期',
  `update_date` datetime NOT NULL COMMENT '修改日期',
  `create_userid` varchar(60) DEFAULT NULL COMMENT '建立者id',
  PRIMARY KEY (`sn`),
  KEY `user_id` (`user_id`),
  KEY `fuser_id` (`fuser_id`),
  KEY `organization_id` (`organization_id`),
  KEY `create_userid` (`create_userid`),
  CONSTRAINT `user_family_ibfk_1` FOREIGN KEY (`organization_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  CONSTRAINT `user_family_ibfk_2` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  CONSTRAINT `user_family_ibfk_3` FOREIGN KEY (`fuser_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  CONSTRAINT `user_family_ibfk_4` FOREIGN KEY (`create_userid`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL
) ENGINE=InnoDB AUTO_INCREMENT=71427 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='學生家長名單'
```
