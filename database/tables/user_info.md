# user_info

使用者基本資料表，儲存所有使用者帳號與個人資訊。

## 關聯

- `class_sn` → `school_class_info.class_sn`
- `organization_id` → `organization.organization_id`
- `update_id` → `user_info.user_id` (自關聯)

## CREATE TABLE

```sql
CREATE TABLE `user_info` (
  `uid` int unsigned NOT NULL AUTO_INCREMENT COMMENT '流水號',
  `user_id` varchar(60) NOT NULL COMMENT '登入帳號',
  `user_account` varchar(60) DEFAULT NULL COMMENT '登入帳號',
  `uname` varchar(60) NOT NULL COMMENT '姓名',
  `email` varchar(100) DEFAULT NULL COMMENT '電子郵件',
  `sex` varchar(2) DEFAULT NULL COMMENT '性別',
  `user_regdate` date NOT NULL COMMENT '註冊日期',
  `birthday` date DEFAULT NULL COMMENT '生日',
  `organization_id` varchar(10) DEFAULT NULL COMMENT '單位(學校)編號',
  `pass` varchar(60) NOT NULL COMMENT '加密密碼',
  `viewpass` varchar(20) DEFAULT NULL COMMENT '使用者密碼',
  `city_code` int NOT NULL DEFAULT '0' COMMENT '城市編號',
  `grade` int NOT NULL DEFAULT '0' COMMENT '年級',
  `class` int NOT NULL DEFAULT '0' COMMENT '班級',
  `class_sn` int unsigned DEFAULT NULL COMMENT '學校班級資訊流水號',
  `identity` varchar(20) DEFAULT NULL COMMENT '身份證字號',
  `hash_guid` varchar(100) DEFAULT NULL COMMENT 'HASH16進制，身份証字號',
  `tel` varchar(20) DEFAULT NULL COMMENT '住所電話',
  `mobil` varchar(20) DEFAULT NULL COMMENT '手機號碼',
  `address` varchar(255) DEFAULT NULL COMMENT '居住地址',
  `class_group` varchar(10) DEFAULT NULL COMMENT '班系',
  `seme` smallint NOT NULL DEFAULT '0' COMMENT '學年度',
  `Open_Map` int DEFAULT NULL,
  `OpenID` varchar(100) NOT NULL COMMENT 'OpenID URL',
  `OpenID_sub` varchar(100) NOT NULL COMMENT 'sub',
  `openid_data` text COMMENT 'OpenID的學校年班資料',
  `openid_updatetime` datetime DEFAULT NULL,
  `OpenID_update` smallint DEFAULT NULL COMMENT '若年班資料從OpenID更新過就會記錄在此 (學期)',
  `update_id` varchar(60) DEFAULT NULL COMMENT '更新者ID',
  `check_code` varchar(10) DEFAULT NULL COMMENT '檢查碼',
  `used` int NOT NULL DEFAULT '1' COMMENT '狀態(啟用1/停用0/刪除2)',
  `priori_name` varchar(100) DEFAULT NULL COMMENT '補救教學對應使用',
  `update_date` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新日期',
  PRIMARY KEY (`uid`),
  UNIQUE KEY `uk_user_id` (`user_id`) USING BTREE,
  UNIQUE KEY `uk_organizationid_userid` (`organization_id`,`user_id`),
  UNIQUE KEY `uk_user_account` (`user_account`),
  KEY `city_code` (`city_code`),
  KEY `organization_id` (`organization_id`),
  KEY `idx_userid_grade_class` (`user_id`,`grade`,`class`) USING BTREE,
  KEY `idx_organizationid_grade_class` (`organization_id`,`grade`,`class`),
  KEY `user_name` (`uname`) USING BTREE,
  KEY `priori_name` (`priori_name`),
  KEY `user_regdate` (`user_regdate`),
  KEY `update_id` (`update_id`),
  KEY `openid_sub` (`OpenID_sub`),
  KEY `hash_guid` (`hash_guid`),
  KEY `email` (`email`),
  KEY `class_sn` (`class_sn`),
  KEY `sex` (`sex`),
  KEY `used` (`used`),
  CONSTRAINT `user_info_ibfk_1` FOREIGN KEY (`class_sn`) REFERENCES `school_class_info` (`class_sn`) ON DELETE SET NULL ON UPDATE SET NULL,
  CONSTRAINT `user_info_ibfk_2` FOREIGN KEY (`organization_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  CONSTRAINT `user_info_ibfk_3` FOREIGN KEY (`update_id`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL
) ENGINE=InnoDB AUTO_INCREMENT=5155380 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci ROW_FORMAT=COMPACT COMMENT='使用者基本資料';
```
