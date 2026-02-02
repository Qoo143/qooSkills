# api_client_config

API 客戶端設定表，儲存申請 API 存取的系統設定與認證資訊。

## CREATE TABLE

```sql
CREATE TABLE `api_client_config` (
  `api_client_sn` int unsigned NOT NULL AUTO_INCREMENT,
  `system_id` varchar(30) NOT NULL COMMENT '申請API的系統英文名稱',
  `debug_mode` tinyint(1) NOT NULL DEFAULT '0' COMMENT '除錯模式(0關1開)',
  `system_name` varchar(25) NOT NULL COMMENT '申請API的系統名稱',
  `redirect_uri` varchar(100) DEFAULT NULL COMMENT '轉址網址',
  `client_id` varchar(50) NOT NULL COMMENT '32位亂碼',
  `client_secret` varchar(64) NOT NULL COMMENT '64位亂碼',
  `update_time` datetime NOT NULL ON UPDATE CURRENT_TIMESTAMP COMMENT '上次異動時間',
  PRIMARY KEY (`api_client_sn`),
  KEY `system_name` (`system_name`),
  KEY `client_id` (`client_id`),
  KEY `client_secret` (`client_secret`),
  KEY `system_id` (`system_id`)
) ENGINE=InnoDB AUTO_INCREMENT=46 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
```
