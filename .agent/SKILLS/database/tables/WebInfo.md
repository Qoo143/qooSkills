# WebInfo

網站資訊內容管理

## CREATE TABLE

```sql
CREATE TABLE `WebInfo` (
  `id` int NOT NULL AUTO_INCREMENT,
  `title` varchar(50) NOT NULL,
  `content` longtext NOT NULL,
  `motherboard` varchar(2) DEFAULT 'Y',
  `isdelete` varchar(2) NOT NULL DEFAULT 'N',
  `created_at` datetime NOT NULL,
  `updated_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=5 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci
```
