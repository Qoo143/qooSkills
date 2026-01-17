# teacher_class_member

教師班級成員表，記錄教師自訂班級的學生名單。

## 關聯

- `class_sn` → `teacher_class.teacher_class_sn`
- `student_id` → `user_info.user_id` (CASCADE)

## CREATE TABLE

```sql
CREATE TABLE `teacher_class_member` (
  `teacher_class_member_sn` int NOT NULL AUTO_INCREMENT,
  `class_sn` int NOT NULL COMMENT '對應teacher_class編號',
  `student_id` varchar(60) NOT NULL COMMENT '學生ID',
  `teacher_agree` int NOT NULL DEFAULT '0' COMMENT '學生自行申請：0 同意：1',
  `add_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `update_time` datetime NOT NULL ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`teacher_class_member_sn`),
  KEY `class_sn` (`class_sn`),
  KEY `teacher_agree` (`teacher_agree`),
  KEY `student_id` (`student_id`),
  CONSTRAINT `teacher_class_member_ibfk_1` FOREIGN KEY (`class_sn`) REFERENCES `teacher_class` (`teacher_class_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  CONSTRAINT `teacher_class_member_ibfk_2` FOREIGN KEY (`student_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE
) ENGINE=InnoDB AUTO_INCREMENT=909925 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
```
