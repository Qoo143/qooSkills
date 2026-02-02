# user_group

學生分組名單

## CREATE TABLE

```sql
CREATE TABLE `user_group` (
  `sn` int NOT NULL AUTO_INCREMENT COMMENT '流水號',
  `organization_id` varchar(10) NOT NULL COMMENT '學校代碼',
  `user_id` varchar(60) NOT NULL COMMENT '使用者id',
  `group_id` int NOT NULL COMMENT '小組id',
  `role_sn` int DEFAULT NULL COMMENT '小組角色sn',
  `create_date` datetime NOT NULL COMMENT '建立日期',
  `update_date` datetime NOT NULL COMMENT '修改日期',
  `group_leader` int DEFAULT '0' COMMENT '小組長=1; 非組長=0 (同組組長只有一人)',
  PRIMARY KEY (`sn`),
  KEY `group_id` (`group_id`),
  KEY `user_id` (`user_id`),
  KEY `create_date` (`create_date`),
  KEY `organization_id` (`organization_id`),
  KEY `role_sn` (`role_sn`),
  KEY `group_leader` (`group_leader`),
  CONSTRAINT `user_group_ibfk_1` FOREIGN KEY (`organization_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  CONSTRAINT `user_group_ibfk_2` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  CONSTRAINT `user_group_ibfk_3` FOREIGN KEY (`group_id`) REFERENCES `seme_group` (`group_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  CONSTRAINT `user_group_ibfk_4` FOREIGN KEY (`role_sn`) REFERENCES `group_role` (`group_role_sn`) ON DELETE SET NULL ON UPDATE SET NULL
) ENGINE=InnoDB AUTO_INCREMENT=720687 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='學生分組名單'
```
