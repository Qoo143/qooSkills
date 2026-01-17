# remedial_student

補救教學學生表，記錄參與補救教學班級的學生名單。

## 關聯

- `class_sn` → `remedial_class.class_sn`
- `student_id` → `user_info.user_id` (CASCADE)
- `new_class_agree` → `new_class.new_class_sn`

## CREATE TABLE

```sql
CREATE TABLE `remedial_student` (
  `remedial_student_sn` int NOT NULL AUTO_INCREMENT,
  `class_sn` int DEFAULT NULL COMMENT '對應補救教學班級編號',
  `cp_id` varchar(20) DEFAULT NULL COMMENT '補救教學唯一ID',
  `student_id` varchar(60) NOT NULL COMMENT '學生ID',
  `new_class_agree` int DEFAULT NULL COMMENT '學生對應課堂班級代碼',
  PRIMARY KEY (`remedial_student_sn`),
  KEY `class_sn` (`class_sn`),
  KEY `new_class_agree` (`new_class_agree`),
  KEY `student_id` (`student_id`),
  CONSTRAINT `remedial_student_ibfk_1` FOREIGN KEY (`class_sn`) REFERENCES `remedial_class` (`class_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  CONSTRAINT `remedial_student_ibfk_2` FOREIGN KEY (`student_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  CONSTRAINT `remedial_student_ibfk_3` FOREIGN KEY (`new_class_agree`) REFERENCES `new_class` (`new_class_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT
) ENGINE=InnoDB AUTO_INCREMENT=1015875 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='補救教學測驗(主檔)';
```
