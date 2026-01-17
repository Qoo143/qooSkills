# user_status

使用者狀態表，記錄使用者的登入狀態、活動時間、權限等級及 email 驗證資訊。

## 關聯

- `user_id` → `user_info.user_id` (CASCADE)
- `access_level` → `user_access.access_level`

## CREATE TABLE

```sql
CREATE TABLE `user_status` (
  `user_id` varchar(60) NOT NULL COMMENT '使用者帳號',
  `access_level` int DEFAULT '1' COMMENT '存取等級',
  `login_freq` int NOT NULL DEFAULT '0' COMMENT '登入次數',
  `login_from` varchar(9) DEFAULT NULL COMMENT '登入方式',
  `starttimestamp` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '上次登入時間(時間格式)',
  `stoptimestamp` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '上次登出時間(時間格式)',
  `video_status` varchar(50) NOT NULL DEFAULT '0' COMMENT '看影片狀態(0:未看;1@XX@能力指標:正在看哪一支影片)',
  `lasttimein` int unsigned DEFAULT NULL COMMENT '上次登入時間(數字格式)',
  `last_action_time` datetime NOT NULL COMMENT '最後活動時間',
  `lasttimeout` int unsigned DEFAULT NULL COMMENT '上次登出時間(數字格式)',
  `total_stay` int unsigned NOT NULL DEFAULT '0' COMMENT '總停留時間',
  `staytime` char(25) NOT NULL COMMENT '停留時間',
  `lastip` char(16) DEFAULT NULL COMMENT '登入ip',
  `email_verify` tinyint(1) DEFAULT NULL COMMENT 'email啟用',
  `email_verify_code` varchar(255) NOT NULL COMMENT 'email驗證碼',
  `email_verify_time` int DEFAULT NULL,
  `remedy_error` datetime DEFAULT NULL,
  `memo` varchar(500) NOT NULL,
  `due_date` varchar(20) NOT NULL COMMENT '授權終止日',
  `Enabled` char(1) NOT NULL DEFAULT '0' COMMENT '有效帳號',
  `PrivacyTimestamp` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00' COMMENT '簽署隱私同意書時間',
  `password_update` datetime DEFAULT CURRENT_TIMESTAMP,
  `PrivacyIP` varchar(16) NOT NULL COMMENT '簽署隱私政策當下IP',
  `guide_step` int NOT NULL,
  `unique_id` varchar(40) NOT NULL COMMENT '登入的唯一值',
  PRIMARY KEY (`user_id`),
  KEY `access_level` (`access_level`),
  KEY `last_action_time` (`last_action_time`),
  KEY `unique_id` (`unique_id`),
  KEY `Enabled` (`Enabled`),
  KEY `video_status` (`video_status`),
  KEY `email_verify` (`email_verify`),
  KEY `guide_step` (`guide_step`),
  KEY `idx_user_status_level` (`user_id`,`access_level`),
  CONSTRAINT `CON_userid` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  CONSTRAINT `user_status_ibfk_1` FOREIGN KEY (`access_level`) REFERENCES `user_access` (`access_level`) ON DELETE RESTRICT ON UPDATE RESTRICT
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
```
