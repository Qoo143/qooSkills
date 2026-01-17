# api_token_data

API Token 資料表，儲存 API 存取 Token 資訊。

## CREATE TABLE

```sql
CREATE TABLE `api_token_data` (
  `token_sn` bigint NOT NULL AUTO_INCREMENT,
  `api_client_sn` int unsigned NOT NULL COMMENT '哪一個系統呼叫的',
  `user_id` varchar(60) NOT NULL,
  `behavior` varchar(20) DEFAULT NULL COMMENT '呼叫此API的行為',
  `item_li_sn` bigint unsigned DEFAULT NULL COMMENT '題目代號',
  `mission_sn` int DEFAULT NULL COMMENT '任務SN',
  `access_token` varchar(128) NOT NULL COMMENT 'md5',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`token_sn`),
  UNIQUE KEY `uk_access_token` (`access_token`) USING BTREE,
  KEY `api_client_sn` (`api_client_sn`),
  KEY `user_id` (`user_id`),
  KEY `behavior` (`behavior`),
  KEY `item_sn` (`item_li_sn`),
  KEY `accesstoken` (`access_token`),
  KEY `mission_sn` (`mission_sn`),
  CONSTRAINT `api_token_data_ibfk_1` FOREIGN KEY (`api_client_sn`) REFERENCES `api_client_config` (`api_client_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  CONSTRAINT `api_token_data_ibfk_2` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  CONSTRAINT `api_token_data_ibfk_3` FOREIGN KEY (`mission_sn`) REFERENCES `mission_info` (`mission_sn`) ON DELETE CASCADE ON UPDATE CASCADE
) ENGINE=InnoDB AUTO_INCREMENT=9592780 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
```
