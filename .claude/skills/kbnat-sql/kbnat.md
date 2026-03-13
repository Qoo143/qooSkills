-- phpMyAdmin SQL Dump
-- version 5.2.1-1.el9.remi
-- https://www.phpmyadmin.net/
--
-- 主機： 34.80.153.120
-- 產生時間： 2026 年 02 月 04 日 15:57
-- 伺服器版本： 8.0.26-google
-- PHP 版本： 7.4.33

SET SQL_MODE = "NO_AUTO_VALUE_ON_ZERO";
START TRANSACTION;
SET time_zone = "+00:00";


/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
/*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */;
/*!40101 SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION */;
/*!40101 SET NAMES utf8mb4 */;

--
-- 資料庫： `kbnat`
--

-- --------------------------------------------------------

--
-- 資料表結構 `api_client_config`
--

CREATE TABLE `api_client_config` (
  `api_client_sn` int UNSIGNED NOT NULL,
  `system_id` varchar(30) NOT NULL COMMENT '申請API的系統英文名稱	',
  `debug_mode` tinyint(1) NOT NULL DEFAULT '0' COMMENT '除錯模式(0關1開)',
  `system_name` varchar(25) NOT NULL COMMENT '申請API的系統名稱',
  `redirect_uri` varchar(100) DEFAULT NULL COMMENT '轉址網址',
  `client_id` varchar(50) NOT NULL COMMENT '32位亂碼',
  `client_secret` varchar(64) NOT NULL COMMENT '64位亂碼',
  `update_time` datetime NOT NULL ON UPDATE CURRENT_TIMESTAMP COMMENT '上次異動時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `api_token_data`
--

CREATE TABLE `api_token_data` (
  `token_sn` bigint NOT NULL,
  `api_client_sn` int UNSIGNED NOT NULL COMMENT '哪一個系統呼叫的',
  `user_id` varchar(60) NOT NULL,
  `behavior` varchar(20) DEFAULT NULL COMMENT '呼叫此API的行為',
  `item_li_sn` bigint UNSIGNED DEFAULT NULL COMMENT '題目代號',
  `mission_sn` int DEFAULT NULL COMMENT '任務SN',
  `access_token` varchar(128) NOT NULL COMMENT 'md5',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `chatroom`
--

CREATE TABLE `chatroom` (
  `chatroom_id` int NOT NULL,
  `mission_sn` int NOT NULL,
  `group_id` int DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `chatroom_member`
--

CREATE TABLE `chatroom_member` (
  `sn` int NOT NULL,
  `chatroom_id` int NOT NULL,
  `member` varchar(60) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `chat_log`
--

CREATE TABLE `chat_log` (
  `sn` int NOT NULL,
  `user_id` varchar(60) NOT NULL,
  `content` text NOT NULL,
  `date_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `reply_sn` int DEFAULT NULL,
  `chatroom_id` int NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `check_list_table`
--

CREATE TABLE `check_list_table` (
  `check_list_table_sn` int NOT NULL COMMENT '流水號SN',
  `seme` varchar(4) NOT NULL,
  `teacher_id` varchar(60) DEFAULT NULL COMMENT '老師的ID',
  `type` tinyint(1) NOT NULL COMMENT '1為檢核表,2為同儕評分表,3為組間評分表,4為組內評分表',
  `title_name` varchar(100) NOT NULL COMMENT '表的標題',
  `qusetion` text NOT NULL COMMENT '表內的題目',
  `score` text NOT NULL COMMENT '每題可以給的分數',
  `public` int NOT NULL DEFAULT '0' COMMENT '1:public(公開-已上鎖);0:public(不公開-未上鎖)',
  `is_delete` smallint NOT NULL DEFAULT '1' COMMENT '1:顯示0:刪除'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `city`
--

CREATE TABLE `city` (
  `city_code` int NOT NULL COMMENT '城市代號',
  `city_name` varchar(16) NOT NULL COMMENT '城市名',
  `openid_ftp_status` tinyint(1) NOT NULL,
  `display_order` int NOT NULL COMMENT '顯示順序',
  `city_id` varchar(16) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '城市代碼'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='城市';

-- --------------------------------------------------------

--
-- 資料表結構 `city_area`
--

CREATE TABLE `city_area` (
  `sn` smallint UNSIGNED NOT NULL COMMENT '流水號',
  `post_code` varchar(4) NOT NULL COMMENT '郵遞區號',
  `city` varchar(12) NOT NULL COMMENT '城市名',
  `area` varchar(20) NOT NULL COMMENT '鄉鎮區名'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='鄉鎮區名稱';

-- --------------------------------------------------------

--
-- 資料表結構 `class_name`
--

CREATE TABLE `class_name` (
  `sn` int NOT NULL,
  `type` int NOT NULL COMMENT '名稱類型編號',
  `num` int NOT NULL COMMENT '班級編號',
  `cht_name` varchar(20) NOT NULL COMMENT '中文班名'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `competency_unit`
--

CREATE TABLE `competency_unit` (
  `sn` int NOT NULL,
  `genres_en` varchar(3) NOT NULL,
  `genres_cht` text NOT NULL,
  `unit_id` varchar(10) NOT NULL,
  `unit_nm` text NOT NULL,
  `subject_id` tinyint(1) DEFAULT NULL COMMENT '領域(1=寫作;2=視覺;3=社會;4=科學)',
  `description` text,
  `src` text,
  `video_url` text,
  `used` tinyint(1) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `concept_info`
--

CREATE TABLE `concept_info` (
  `cs_sn` bigint NOT NULL COMMENT '概念流水號',
  `cs_id` varchar(12) NOT NULL COMMENT '單元結構編號',
  `publisher_id` smallint UNSIGNED DEFAULT NULL COMMENT '出版公司',
  `subject_id` smallint UNSIGNED NOT NULL COMMENT '學科名稱',
  `vol` int NOT NULL COMMENT '書籍冊別',
  `unit` int NOT NULL COMMENT '課程單元',
  `grade` int NOT NULL COMMENT '課程年級',
  `concept` varchar(30) NOT NULL COMMENT '概念名稱',
  `matrix_map` varchar(50) NOT NULL,
  `remedy_file` varchar(50) NOT NULL COMMENT '試卷概念表',
  `skill_file` varchar(50) NOT NULL COMMENT 'skill檔案',
  `item_remedy_file` varchar(50) NOT NULL COMMENT '試題概念表',
  `bn_data_file` varchar(50) NOT NULL,
  `indicator_nums` int NOT NULL DEFAULT '0',
  `percent_map` varchar(50) NOT NULL COMMENT '百分對照表',
  `percent_gif` varchar(50) NOT NULL COMMENT '百分等級圖',
  `structure_gif` varchar(50) NOT NULL,
  `dag_map` varchar(50) NOT NULL COMMENT 'Bug_Item對照',
  `skill_item` varchar(50) DEFAULT NULL COMMENT 'Skill與Item的對照',
  `indicator_relation` varchar(50) NOT NULL,
  `indicator_item` varchar(50) NOT NULL,
  `indicator_threshold` varchar(50) NOT NULL,
  `indicator_item_nums` varchar(50) NOT NULL,
  `indicator_item_relation` varchar(50) NOT NULL,
  `remedy_instruction` mediumtext NOT NULL,
  `skill_instruction` mediumtext NOT NULL,
  `book_ref` mediumtext NOT NULL,
  `q_matrix` varchar(50) DEFAULT NULL,
  `sg_item` varchar(50) DEFAULT NULL COMMENT '試題參數s_g'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='單元結構資訊';

-- --------------------------------------------------------

--
-- 資料表結構 `concept_info_plus`
--

CREATE TABLE `concept_info_plus` (
  `cs_id` varchar(10) NOT NULL,
  `matrix_map_csv` varchar(50) DEFAULT NULL,
  `expert_map_csv` varchar(50) DEFAULT NULL,
  `item_sequence_2` mediumtext,
  `item_sequence_1` mediumtext,
  `item_sequence_3` mediumtext,
  `remedySort` mediumtext NOT NULL COMMENT '概念順序',
  `ready` int NOT NULL,
  `stu_stru_ready` tinyint(1) NOT NULL DEFAULT '0',
  `expert_stru_ready` tinyint(1) NOT NULL DEFAULT '0',
  `min_test_nums_stu` smallint NOT NULL DEFAULT '0',
  `min_test_nums_expert` smallint NOT NULL DEFAULT '0',
  `max_item_nums` smallint NOT NULL DEFAULT '0'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `concept_item`
--

CREATE TABLE `concept_item` (
  `item_sn` bigint UNSIGNED NOT NULL,
  `exam_paper_id` varchar(15) NOT NULL COMMENT '試題編號',
  `item_num` int UNSIGNED NOT NULL,
  `cs_id` varchar(12) NOT NULL COMMENT '概念id',
  `item_content` varchar(50) NOT NULL COMMENT '試題作答區內容',
  `item_filename` varchar(50) NOT NULL,
  `op_content` mediumtext NOT NULL COMMENT '選項文字內容',
  `op_filename` mediumtext NOT NULL COMMENT '選項圖片檔名',
  `op_ans` varchar(1) NOT NULL COMMENT '正確答案',
  `points` float NOT NULL DEFAULT '0' COMMENT '該題配分',
  `paper_vol` tinyint UNSIGNED NOT NULL COMMENT '概念試卷編號',
  `edu_parameter` varchar(50) DEFAULT NULL,
  `indicator` varchar(15) NOT NULL,
  `template_id` tinyint UNSIGNED NOT NULL COMMENT '填充/選擇/2動態評量',
  `indicator_item_num` tinyint UNSIGNED NOT NULL COMMENT '能力指標子題號',
  `double_item` int NOT NULL DEFAULT '0'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='試題資訊';

-- --------------------------------------------------------

--
-- 資料表結構 `concept_itemBank`
--

CREATE TABLE `concept_itemBank` (
  `item_sn` bigint UNSIGNED NOT NULL COMMENT '關聯',
  `sems` varchar(3) NOT NULL COMMENT '學年度',
  `subject_id` smallint UNSIGNED NOT NULL COMMENT '科目',
  `map_sn` int DEFAULT NULL,
  `filename` mediumtext NOT NULL COMMENT '題目檔名',
  `op_filename` mediumtext NOT NULL COMMENT '選項檔名',
  `detail_filename` text COMMENT '本題詳解',
  `op_ans` text CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '答案',
  `indicator` varchar(50) DEFAULT NULL COMMENT '能力指標',
  `template_id` tinyint UNSIGNED NOT NULL COMMENT '	試題類別:1單選題樣版、2動態評量樣版、3選填題樣版、4單選題、5多選題、6是非題	',
  `itembank_group` int NOT NULL COMMENT '題庫分組編號',
  `answer_option_type` tinyint UNSIGNED NOT NULL DEFAULT '1' COMMENT '選項是否適用文字敘述(1：適用 0：不適用)',
  `edit` tinyint UNSIGNED NOT NULL COMMENT '是否允許編修(1=可；0=否)(fuyun:應該沒在用了)',
  `public` tinyint UNSIGNED NOT NULL COMMENT '是否公開(1=可；0=否；2=被刪除)',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新時間',
  `delete_time` datetime DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `concept_itemBank_0320`
--

CREATE TABLE `concept_itemBank_0320` (
  `item_sn` bigint UNSIGNED NOT NULL COMMENT '關聯',
  `sems` varchar(3) NOT NULL COMMENT '學年度',
  `subject_id` smallint UNSIGNED NOT NULL COMMENT '科目',
  `map_sn` int DEFAULT NULL,
  `filename` mediumtext NOT NULL COMMENT '題目檔名',
  `op_filename` mediumtext NOT NULL COMMENT '選項檔名',
  `detail_filename` text COMMENT '本題詳解',
  `op_ans` text CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '答案',
  `indicator` varchar(50) DEFAULT NULL COMMENT '能力指標',
  `template_id` tinyint UNSIGNED NOT NULL COMMENT '	試題類別:1單選題樣版、2動態評量樣版、3選填題樣版、4單選題、5多選題、6是非題	',
  `itembank_group` int NOT NULL COMMENT '題庫分組編號',
  `answer_option_type` tinyint UNSIGNED NOT NULL DEFAULT '1' COMMENT '選項是否適用文字敘述(1：適用 0：不適用)',
  `edit` tinyint UNSIGNED NOT NULL COMMENT '是否允許編修(1=可；0=否)(fuyun:應該沒在用了)',
  `public` tinyint UNSIGNED NOT NULL COMMENT '是否公開(1=可；0=否；2=被刪除)',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新時間',
  `delete_time` datetime DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `concept_itemBank_0521`
--

CREATE TABLE `concept_itemBank_0521` (
  `item_sn` bigint UNSIGNED NOT NULL COMMENT '關聯',
  `sems` varchar(3) NOT NULL COMMENT '學年度',
  `subject_id` smallint UNSIGNED NOT NULL COMMENT '科目',
  `map_sn` int DEFAULT NULL,
  `filename` mediumtext NOT NULL COMMENT '題目檔名',
  `op_filename` mediumtext NOT NULL COMMENT '選項檔名',
  `detail_filename` text COMMENT '本題詳解',
  `op_ans` tinyint UNSIGNED NOT NULL COMMENT '答案',
  `indicator` varchar(50) DEFAULT NULL COMMENT '能力指標',
  `template_id` tinyint UNSIGNED NOT NULL COMMENT '	試題類別:1單選題樣版、2動態評量樣版、3選填題樣版、4單選題、5多選題、6是非題	',
  `itembank_group` int NOT NULL COMMENT '題庫分組編號',
  `answer_option_type` tinyint UNSIGNED NOT NULL DEFAULT '1' COMMENT '選項是否適用文字敘述(1：適用 0：不適用)',
  `edit` tinyint UNSIGNED NOT NULL COMMENT '是否允許編修(1=可；0=否)(fuyun:應該沒在用了)',
  `public` tinyint UNSIGNED NOT NULL COMMENT '是否公開(1=可；0=否；2=被刪除)',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新時間',
  `delete_time` datetime DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `concept_itemBank_20240904`
--

CREATE TABLE `concept_itemBank_20240904` (
  `item_sn` bigint UNSIGNED NOT NULL COMMENT '關聯',
  `sems` varchar(3) NOT NULL COMMENT '學年度',
  `subject_id` smallint UNSIGNED NOT NULL COMMENT '科目',
  `map_sn` int DEFAULT NULL,
  `filename` mediumtext NOT NULL COMMENT '題目檔名',
  `op_filename` mediumtext NOT NULL COMMENT '選項檔名',
  `detail_filename` text COMMENT '本題詳解',
  `op_ans` tinyint UNSIGNED NOT NULL COMMENT '答案',
  `indicator` varchar(50) DEFAULT NULL COMMENT '能力指標',
  `template_id` tinyint UNSIGNED NOT NULL COMMENT '	試題類別:1單選題樣版、2動態評量樣版、3選填題樣版、4單選題、5多選題、6是非題	',
  `itembank_group` int NOT NULL COMMENT '題庫分組編號',
  `answer_option_type` tinyint UNSIGNED NOT NULL DEFAULT '1' COMMENT '選項是否適用文字敘述(1：適用 0：不適用)',
  `edit` tinyint UNSIGNED NOT NULL COMMENT '是否允許編修(1=可；0=否)(fuyun:應該沒在用了)',
  `public` tinyint UNSIGNED NOT NULL COMMENT '是否公開(1=可；0=否；2=被刪除)',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新時間',
  `delete_time` datetime DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `concept_itemBank_20250512`
--

CREATE TABLE `concept_itemBank_20250512` (
  `item_sn` bigint UNSIGNED NOT NULL COMMENT '關聯',
  `sems` varchar(3) NOT NULL COMMENT '學年度',
  `subject_id` smallint UNSIGNED NOT NULL COMMENT '科目',
  `map_sn` int DEFAULT NULL,
  `filename` mediumtext NOT NULL COMMENT '題目檔名',
  `op_filename` mediumtext NOT NULL COMMENT '選項檔名',
  `detail_filename` text COMMENT '本題詳解',
  `op_ans` text CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '答案',
  `indicator` varchar(50) DEFAULT NULL COMMENT '能力指標',
  `template_id` tinyint UNSIGNED NOT NULL COMMENT '	試題類別:1單選題樣版、2動態評量樣版、3選填題樣版、4單選題、5多選題、6是非題	',
  `itembank_group` int NOT NULL COMMENT '題庫分組編號',
  `answer_option_type` tinyint UNSIGNED NOT NULL DEFAULT '1' COMMENT '選項是否適用文字敘述(1：適用 0：不適用)',
  `edit` tinyint UNSIGNED NOT NULL COMMENT '是否允許編修(1=可；0=否)(fuyun:應該沒在用了)',
  `public` tinyint UNSIGNED NOT NULL COMMENT '是否公開(1=可；0=否；2=被刪除)',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新時間',
  `delete_time` datetime DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `concept_itemBank_20250819`
--

CREATE TABLE `concept_itemBank_20250819` (
  `item_sn` bigint UNSIGNED NOT NULL COMMENT '關聯',
  `sems` varchar(3) NOT NULL COMMENT '學年度',
  `subject_id` smallint UNSIGNED NOT NULL COMMENT '科目',
  `map_sn` int DEFAULT NULL,
  `filename` mediumtext NOT NULL COMMENT '題目檔名',
  `op_filename` mediumtext NOT NULL COMMENT '選項檔名',
  `detail_filename` text COMMENT '本題詳解',
  `op_ans` text CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '答案',
  `indicator` varchar(50) DEFAULT NULL COMMENT '能力指標',
  `template_id` tinyint UNSIGNED NOT NULL COMMENT '	試題類別:1單選題樣版、2動態評量樣版、3選填題樣版、4單選題、5多選題、6是非題	',
  `itembank_group` int NOT NULL COMMENT '題庫分組編號',
  `answer_option_type` tinyint UNSIGNED NOT NULL DEFAULT '1' COMMENT '選項是否適用文字敘述(1：適用 0：不適用)',
  `edit` tinyint UNSIGNED NOT NULL COMMENT '是否允許編修(1=可；0=否)(fuyun:應該沒在用了)',
  `public` tinyint UNSIGNED NOT NULL COMMENT '是否公開(1=可；0=否；2=被刪除)',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新時間',
  `delete_time` datetime DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `concept_itemBank_20250827`
--

CREATE TABLE `concept_itemBank_20250827` (
  `item_sn` bigint UNSIGNED NOT NULL COMMENT '關聯',
  `sems` varchar(3) NOT NULL COMMENT '學年度',
  `subject_id` smallint UNSIGNED NOT NULL COMMENT '科目',
  `map_sn` int DEFAULT NULL,
  `filename` mediumtext NOT NULL COMMENT '題目檔名',
  `op_filename` mediumtext NOT NULL COMMENT '選項檔名',
  `detail_filename` text COMMENT '本題詳解',
  `op_ans` text CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '答案',
  `indicator` varchar(50) DEFAULT NULL COMMENT '能力指標',
  `template_id` tinyint UNSIGNED NOT NULL COMMENT '	試題類別:1單選題樣版、2動態評量樣版、3選填題樣版、4單選題、5多選題、6是非題	',
  `itembank_group` int NOT NULL COMMENT '題庫分組編號',
  `answer_option_type` tinyint UNSIGNED NOT NULL DEFAULT '1' COMMENT '選項是否適用文字敘述(1：適用 0：不適用)',
  `edit` tinyint UNSIGNED NOT NULL COMMENT '是否允許編修(1=可；0=否)(fuyun:應該沒在用了)',
  `public` tinyint UNSIGNED NOT NULL COMMENT '是否公開(1=可；0=否；2=被刪除)',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新時間',
  `delete_time` datetime DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `concept_itemBank_20251105`
--

CREATE TABLE `concept_itemBank_20251105` (
  `item_sn` bigint UNSIGNED NOT NULL COMMENT '關聯',
  `sems` varchar(3) NOT NULL COMMENT '學年度',
  `subject_id` smallint UNSIGNED NOT NULL COMMENT '科目',
  `map_sn` int DEFAULT NULL,
  `filename` mediumtext NOT NULL COMMENT '題目檔名',
  `op_filename` mediumtext NOT NULL COMMENT '選項檔名',
  `detail_filename` text COMMENT '本題詳解',
  `op_ans` text CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '答案',
  `indicator` varchar(50) DEFAULT NULL COMMENT '能力指標',
  `template_id` tinyint UNSIGNED NOT NULL COMMENT '	試題類別:1單選題樣版、2動態評量樣版、3選填題樣版、4單選題、5多選題、6是非題	',
  `itembank_group` int NOT NULL COMMENT '題庫分組編號',
  `answer_option_type` tinyint UNSIGNED NOT NULL DEFAULT '1' COMMENT '選項是否適用文字敘述(1：適用 0：不適用)',
  `edit` tinyint UNSIGNED NOT NULL COMMENT '是否允許編修(1=可；0=否)(fuyun:應該沒在用了)',
  `public` tinyint UNSIGNED NOT NULL COMMENT '是否公開(1=可；0=否；2=被刪除)',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新時間',
  `delete_time` datetime DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `concept_itemBank_20251223`
--

CREATE TABLE `concept_itemBank_20251223` (
  `item_sn` bigint UNSIGNED NOT NULL COMMENT '關聯',
  `sems` varchar(3) NOT NULL COMMENT '學年度',
  `subject_id` smallint UNSIGNED NOT NULL COMMENT '科目',
  `map_sn` int DEFAULT NULL,
  `filename` mediumtext NOT NULL COMMENT '題目檔名',
  `op_filename` mediumtext NOT NULL COMMENT '選項檔名',
  `detail_filename` text COMMENT '本題詳解',
  `op_ans` text CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '答案',
  `indicator` varchar(50) DEFAULT NULL COMMENT '能力指標',
  `template_id` tinyint UNSIGNED NOT NULL COMMENT '	試題類別:1單選題樣版、2動態評量樣版、3選填題樣版、4單選題、5多選題、6是非題	',
  `itembank_group` int NOT NULL COMMENT '題庫分組編號',
  `answer_option_type` tinyint UNSIGNED NOT NULL DEFAULT '1' COMMENT '選項是否適用文字敘述(1：適用 0：不適用)',
  `edit` tinyint UNSIGNED NOT NULL COMMENT '是否允許編修(1=可；0=否)(fuyun:應該沒在用了)',
  `public` tinyint UNSIGNED NOT NULL COMMENT '是否公開(1=可；0=否；2=被刪除)',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新時間',
  `delete_time` datetime DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `concept_itemBank_bat`
--

CREATE TABLE `concept_itemBank_bat` (
  `item_sn` int UNSIGNED NOT NULL COMMENT '關聯',
  `sems` varchar(3) NOT NULL COMMENT '學年度',
  `subject_id` smallint UNSIGNED NOT NULL COMMENT '科目',
  `map_sn` int NOT NULL,
  `filename` mediumtext NOT NULL COMMENT '題目檔名',
  `op_filename` mediumtext NOT NULL COMMENT '選項檔名',
  `detail_filename` text COMMENT '本題詳解',
  `op_ans` mediumtext NOT NULL COMMENT '答案',
  `indicator` varchar(50) NOT NULL COMMENT '能力指標，對應到concept_itemBank_bat_inds',
  `template_id` varchar(2) NOT NULL COMMENT '	試題類別:1單選題樣版、2動態評量樣版、3選填題樣版、4單選題、5多選題、6是非題	',
  `itembank_group` int NOT NULL COMMENT '題庫分組編號',
  `edit` varchar(1) NOT NULL COMMENT '是否允許編修(1=可；0=否)(fuyun:應該沒在用了)',
  `public` varchar(1) NOT NULL COMMENT '是否公開(1=可；0=否；2=被刪除)',
  `delete_time` datetime DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `concept_itemBank_bat_inds`
--

CREATE TABLE `concept_itemBank_bat_inds` (
  `sn` int UNSIGNED NOT NULL COMMENT '關聯',
  `item_sn` int UNSIGNED NOT NULL COMMENT 'item_sn',
  `indicator` varchar(150) NOT NULL COMMENT '能力指標'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `concept_itemBank_depiction`
--

CREATE TABLE `concept_itemBank_depiction` (
  `depiction_sn` int UNSIGNED NOT NULL,
  `item_sn` bigint UNSIGNED NOT NULL,
  `main_text` text CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci,
  `sub_text` text CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci,
  `detail_text` text,
  `updater` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL,
  `creation_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `concept_itemBank_depiction_0320`
--

CREATE TABLE `concept_itemBank_depiction_0320` (
  `depiction_sn` int UNSIGNED NOT NULL,
  `item_sn` bigint UNSIGNED NOT NULL,
  `main_text` text CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci,
  `sub_text` text CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci,
  `detail_text` text,
  `updater` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL,
  `creation_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `concept_itemBank_depiction_0710`
--

CREATE TABLE `concept_itemBank_depiction_0710` (
  `depiction_sn` int UNSIGNED NOT NULL,
  `item_sn` bigint UNSIGNED NOT NULL,
  `main_text` text CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci,
  `sub_text` text CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci,
  `detail_text` text,
  `AI_analyze_text` text CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci,
  `AI_analyze` tinyint(1) DEFAULT NULL,
  `updater` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL,
  `creation_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `concept_itemBank_depiction_20240904`
--

CREATE TABLE `concept_itemBank_depiction_20240904` (
  `depiction_sn` int UNSIGNED NOT NULL,
  `item_sn` bigint UNSIGNED NOT NULL,
  `main_text` text CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci,
  `sub_text` text CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci,
  `detail_text` text,
  `updater` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL,
  `creation_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `concept_itemBank_depiction_20250512`
--

CREATE TABLE `concept_itemBank_depiction_20250512` (
  `depiction_sn` int UNSIGNED NOT NULL,
  `item_sn` bigint UNSIGNED NOT NULL,
  `main_text` text CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci,
  `sub_text` text CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci,
  `detail_text` text,
  `AI_analyze_text` text CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci,
  `AI_analyze` tinyint(1) DEFAULT NULL,
  `updater` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL,
  `creation_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `concept_itemBank_depiction_20250819`
--

CREATE TABLE `concept_itemBank_depiction_20250819` (
  `depiction_sn` int UNSIGNED NOT NULL,
  `item_sn` bigint UNSIGNED NOT NULL,
  `main_text` text CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci,
  `sub_text` text CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci,
  `detail_text` text,
  `updater` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL,
  `creation_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `concept_itemBank_depiction_20250827`
--

CREATE TABLE `concept_itemBank_depiction_20250827` (
  `depiction_sn` int UNSIGNED NOT NULL,
  `item_sn` bigint UNSIGNED NOT NULL,
  `main_text` text CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci,
  `sub_text` text CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci,
  `detail_text` text,
  `updater` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL,
  `creation_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `concept_itemBank_depiction_20251105`
--

CREATE TABLE `concept_itemBank_depiction_20251105` (
  `depiction_sn` int UNSIGNED NOT NULL,
  `item_sn` bigint UNSIGNED NOT NULL,
  `main_text` text CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci,
  `sub_text` text CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci,
  `detail_text` text,
  `updater` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL,
  `creation_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `concept_itemBank_depiction_20251223`
--

CREATE TABLE `concept_itemBank_depiction_20251223` (
  `depiction_sn` int UNSIGNED NOT NULL,
  `item_sn` bigint UNSIGNED NOT NULL,
  `main_text` text CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci,
  `sub_text` text CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci,
  `detail_text` text,
  `updater` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL,
  `creation_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `concept_itemBank_literacy`
--

CREATE TABLE `concept_itemBank_literacy` (
  `item_sn` int UNSIGNED NOT NULL,
  `subject_id` smallint UNSIGNED NOT NULL COMMENT '科目',
  `map_sn` int NOT NULL,
  `sub_subject_id` tinyint UNSIGNED DEFAULT NULL COMMENT '子科目',
  `learning_stage` tinyint(1) NOT NULL COMMENT '學習階段 1:國小第一階段 2:國小第二階段 3:國小第三階段 4:國高中',
  `literacy_group_sn` int UNSIGNED DEFAULT NULL COMMENT '對應題組的sn，若無則為單一題資料為NULL',
  `indicator` mediumtext NOT NULL COMMENT '本題對應到的能力指標',
  `theme` varchar(25) DEFAULT NULL COMMENT '題目主題 (代數,數量,統計...)',
  `item_situation` mediumtext NOT NULL COMMENT '試題情境',
  `item_type` tinyint NOT NULL COMMENT '題型類別(0=選擇題；1=是非題)',
  `item_name` varchar(25) NOT NULL COMMENT '題目標題',
  `filename` mediumtext NOT NULL COMMENT '題目檔名',
  `op_filename` mediumtext NOT NULL COMMENT '選項檔名',
  `answer` varchar(1) NOT NULL COMMENT '答案',
  `thought_type` varchar(10) DEFAULT NULL COMMENT '思考類型',
  `learning_performance` varchar(20) DEFAULT NULL COMMENT '學習表現',
  `core` mediumtext NOT NULL COMMENT '核心素養',
  `cross_subject` mediumtext NOT NULL COMMENT '本題是否與其他科目有關連',
  `item_status` varchar(1) NOT NULL DEFAULT '0' COMMENT '是否上鎖(0=否；1=是；2=被刪除)',
  `create_user` varchar(60) DEFAULT NULL COMMENT '建立者',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `concept_itemBank_literacy_20250512`
--

CREATE TABLE `concept_itemBank_literacy_20250512` (
  `item_sn` int UNSIGNED NOT NULL,
  `subject_id` smallint UNSIGNED NOT NULL COMMENT '科目',
  `map_sn` int NOT NULL,
  `sub_subject_id` tinyint UNSIGNED DEFAULT NULL COMMENT '子科目',
  `learning_stage` tinyint(1) NOT NULL COMMENT '學習階段 1:國小第一階段 2:國小第二階段 3:國小第三階段 4:國高中',
  `literacy_group_sn` int UNSIGNED DEFAULT NULL COMMENT '對應題組的sn，若無則為單一題資料為NULL',
  `indicator` mediumtext NOT NULL COMMENT '本題對應到的能力指標',
  `theme` varchar(25) DEFAULT NULL COMMENT '題目主題 (代數,數量,統計...)',
  `item_situation` mediumtext NOT NULL COMMENT '試題情境',
  `item_type` tinyint NOT NULL COMMENT '題型類別(0=選擇題；1=是非題)',
  `item_name` varchar(25) NOT NULL COMMENT '題目標題',
  `filename` mediumtext NOT NULL COMMENT '題目檔名',
  `op_filename` mediumtext NOT NULL COMMENT '選項檔名',
  `answer` varchar(1) NOT NULL COMMENT '答案',
  `thought_type` varchar(10) DEFAULT NULL COMMENT '思考類型',
  `learning_performance` varchar(20) DEFAULT NULL COMMENT '學習表現',
  `core` mediumtext NOT NULL COMMENT '核心素養',
  `cross_subject` mediumtext NOT NULL COMMENT '本題是否與其他科目有關連',
  `item_status` varchar(1) NOT NULL DEFAULT '0' COMMENT '是否上鎖(0=否；1=是；2=被刪除)',
  `create_user` varchar(60) DEFAULT NULL COMMENT '建立者',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `concept_itemBank_literacy_20250819`
--

CREATE TABLE `concept_itemBank_literacy_20250819` (
  `item_sn` int UNSIGNED NOT NULL,
  `subject_id` smallint UNSIGNED NOT NULL COMMENT '科目',
  `map_sn` int NOT NULL,
  `sub_subject_id` tinyint UNSIGNED DEFAULT NULL COMMENT '子科目',
  `learning_stage` tinyint(1) NOT NULL COMMENT '學習階段 1:國小第一階段 2:國小第二階段 3:國小第三階段 4:國高中',
  `literacy_group_sn` int UNSIGNED DEFAULT NULL COMMENT '對應題組的sn，若無則為單一題資料為NULL',
  `indicator` mediumtext NOT NULL COMMENT '本題對應到的能力指標',
  `theme` varchar(25) DEFAULT NULL COMMENT '題目主題 (代數,數量,統計...)',
  `item_situation` mediumtext NOT NULL COMMENT '試題情境',
  `item_type` tinyint NOT NULL COMMENT '題型類別(0=選擇題；1=是非題)',
  `item_name` varchar(25) NOT NULL COMMENT '題目標題',
  `filename` mediumtext NOT NULL COMMENT '題目檔名',
  `op_filename` mediumtext NOT NULL COMMENT '選項檔名',
  `answer` varchar(1) NOT NULL COMMENT '答案',
  `thought_type` varchar(10) DEFAULT NULL COMMENT '思考類型',
  `learning_performance` varchar(20) DEFAULT NULL COMMENT '學習表現',
  `core` mediumtext NOT NULL COMMENT '核心素養',
  `cross_subject` mediumtext NOT NULL COMMENT '本題是否與其他科目有關連',
  `item_status` varchar(1) NOT NULL DEFAULT '0' COMMENT '是否上鎖(0=否；1=是；2=被刪除)',
  `create_user` varchar(60) DEFAULT NULL COMMENT '建立者',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `concept_itemBank_literacy_20250827`
--

CREATE TABLE `concept_itemBank_literacy_20250827` (
  `item_sn` int UNSIGNED NOT NULL,
  `subject_id` smallint UNSIGNED NOT NULL COMMENT '科目',
  `map_sn` int NOT NULL,
  `sub_subject_id` tinyint UNSIGNED DEFAULT NULL COMMENT '子科目',
  `learning_stage` tinyint(1) NOT NULL COMMENT '學習階段 1:國小第一階段 2:國小第二階段 3:國小第三階段 4:國高中',
  `literacy_group_sn` int UNSIGNED DEFAULT NULL COMMENT '對應題組的sn，若無則為單一題資料為NULL',
  `indicator` mediumtext NOT NULL COMMENT '本題對應到的能力指標',
  `theme` varchar(25) DEFAULT NULL COMMENT '題目主題 (代數,數量,統計...)',
  `item_situation` mediumtext NOT NULL COMMENT '試題情境',
  `item_type` tinyint NOT NULL COMMENT '題型類別(0=選擇題；1=是非題)',
  `item_name` varchar(25) NOT NULL COMMENT '題目標題',
  `filename` mediumtext NOT NULL COMMENT '題目檔名',
  `op_filename` mediumtext NOT NULL COMMENT '選項檔名',
  `answer` varchar(1) NOT NULL COMMENT '答案',
  `thought_type` varchar(10) DEFAULT NULL COMMENT '思考類型',
  `learning_performance` varchar(20) DEFAULT NULL COMMENT '學習表現',
  `core` mediumtext NOT NULL COMMENT '核心素養',
  `cross_subject` mediumtext NOT NULL COMMENT '本題是否與其他科目有關連',
  `item_status` varchar(1) NOT NULL DEFAULT '0' COMMENT '是否上鎖(0=否；1=是；2=被刪除)',
  `create_user` varchar(60) DEFAULT NULL COMMENT '建立者',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `concept_itemBank_literacy_20251105`
--

CREATE TABLE `concept_itemBank_literacy_20251105` (
  `item_sn` int UNSIGNED NOT NULL,
  `subject_id` smallint UNSIGNED NOT NULL COMMENT '科目',
  `map_sn` int NOT NULL,
  `sub_subject_id` tinyint UNSIGNED DEFAULT NULL COMMENT '子科目',
  `learning_stage` tinyint(1) NOT NULL COMMENT '學習階段 1:國小第一階段 2:國小第二階段 3:國小第三階段 4:國高中',
  `literacy_group_sn` int UNSIGNED DEFAULT NULL COMMENT '對應題組的sn，若無則為單一題資料為NULL',
  `indicator` mediumtext NOT NULL COMMENT '本題對應到的能力指標',
  `theme` varchar(25) DEFAULT NULL COMMENT '題目主題 (代數,數量,統計...)',
  `item_situation` mediumtext NOT NULL COMMENT '試題情境',
  `item_type` tinyint NOT NULL COMMENT '題型類別(0=選擇題；1=是非題)',
  `item_name` varchar(25) NOT NULL COMMENT '題目標題',
  `filename` mediumtext NOT NULL COMMENT '題目檔名',
  `op_filename` mediumtext NOT NULL COMMENT '選項檔名',
  `answer` varchar(1) NOT NULL COMMENT '答案',
  `thought_type` varchar(10) DEFAULT NULL COMMENT '思考類型',
  `learning_performance` varchar(20) DEFAULT NULL COMMENT '學習表現',
  `core` mediumtext NOT NULL COMMENT '核心素養',
  `cross_subject` mediumtext NOT NULL COMMENT '本題是否與其他科目有關連',
  `item_status` varchar(1) NOT NULL DEFAULT '0' COMMENT '是否上鎖(0=否；1=是；2=被刪除)',
  `create_user` varchar(60) DEFAULT NULL COMMENT '建立者',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `concept_itemBank_literacy_20251223`
--

CREATE TABLE `concept_itemBank_literacy_20251223` (
  `item_sn` int UNSIGNED NOT NULL,
  `subject_id` smallint UNSIGNED NOT NULL COMMENT '科目',
  `map_sn` int NOT NULL,
  `sub_subject_id` tinyint UNSIGNED DEFAULT NULL COMMENT '子科目',
  `learning_stage` tinyint(1) NOT NULL COMMENT '學習階段 1:國小第一階段 2:國小第二階段 3:國小第三階段 4:國高中',
  `literacy_group_sn` int UNSIGNED DEFAULT NULL COMMENT '對應題組的sn，若無則為單一題資料為NULL',
  `indicator` mediumtext NOT NULL COMMENT '本題對應到的能力指標',
  `theme` varchar(25) DEFAULT NULL COMMENT '題目主題 (代數,數量,統計...)',
  `item_situation` mediumtext NOT NULL COMMENT '試題情境',
  `item_type` tinyint NOT NULL COMMENT '題型類別(0=選擇題；1=是非題)',
  `item_name` varchar(25) NOT NULL COMMENT '題目標題',
  `filename` mediumtext NOT NULL COMMENT '題目檔名',
  `op_filename` mediumtext NOT NULL COMMENT '選項檔名',
  `answer` varchar(1) NOT NULL COMMENT '答案',
  `thought_type` varchar(10) DEFAULT NULL COMMENT '思考類型',
  `learning_performance` varchar(20) DEFAULT NULL COMMENT '學習表現',
  `core` mediumtext NOT NULL COMMENT '核心素養',
  `cross_subject` mediumtext NOT NULL COMMENT '本題是否與其他科目有關連',
  `item_status` varchar(1) NOT NULL DEFAULT '0' COMMENT '是否上鎖(0=否；1=是；2=被刪除)',
  `create_user` varchar(60) DEFAULT NULL COMMENT '建立者',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `concept_itemBank_literacy_conversation`
--

CREATE TABLE `concept_itemBank_literacy_conversation` (
  `literacy_conversation_sn` int UNSIGNED NOT NULL,
  `item_li_sn` int UNSIGNED NOT NULL COMMENT 'concept_itemBank_literacy_interactive.item_sn',
  `item_li_theme` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL,
  `item_li_theme_number` int NOT NULL,
  `item_li_theme_detail` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '全部詳細內容',
  `item_li_theme_topic` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '題目',
  `item_li_theme_logic` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '解題邏輯',
  `item_li_theme_answer` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '答案'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `concept_itemBank_literacy_group`
--

CREATE TABLE `concept_itemBank_literacy_group` (
  `literacy_group_sn` int UNSIGNED NOT NULL,
  `subject_id` smallint UNSIGNED NOT NULL COMMENT '科目',
  `map_sn` int NOT NULL,
  `sub_subject_id` tinyint UNSIGNED DEFAULT NULL COMMENT '子科目',
  `item_name` varchar(25) NOT NULL COMMENT '題組名稱',
  `create_user` varchar(60) DEFAULT NULL COMMENT '建立者',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新時間',
  `status` tinyint(1) NOT NULL DEFAULT '0' COMMENT '是否上鎖(0=否；1=是；2=被刪除)'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `concept_itemBank_literacy_interactive`
--

CREATE TABLE `concept_itemBank_literacy_interactive` (
  `item_li_sn` int UNSIGNED NOT NULL,
  `subject_id` smallint UNSIGNED NOT NULL,
  `map_sn` int NOT NULL,
  `sub_subject_id` tinyint UNSIGNED DEFAULT NULL COMMENT '子科目',
  `indicator` varchar(50) NOT NULL COMMENT '能力指標',
  `item_show_num` int(3) UNSIGNED ZEROFILL DEFAULT NULL COMMENT '上架後試題編號',
  `theme` varchar(25) NOT NULL COMMENT '題目主題 (代數,數量,統計...)',
  `item_name` varchar(60) NOT NULL COMMENT '題目標題	',
  `learning_stage` tinyint(1) NOT NULL COMMENT '學習階段 10:高中(混合題)',
  `grade` int UNSIGNED DEFAULT NULL,
  `type` tinyint UNSIGNED NOT NULL DEFAULT '1' COMMENT '1:互動題，0:一般題',
  `thought_type` varchar(10) NOT NULL COMMENT '思考類型',
  `core` varchar(255) NOT NULL COMMENT '核心素養',
  `core_code` varchar(100) DEFAULT NULL COMMENT '學習內容',
  `learning_content` varchar(100) DEFAULT NULL COMMENT '核心素養代碼',
  `core_code_list` varchar(250) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '題目核心素養代號',
  `learning_content_list` varchar(250) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '題目學習內容',
  `item_situation` varchar(10) NOT NULL COMMENT '試題情境	',
  `questions_word` mediumtext COMMENT '題目文字',
  `solution_word` mediumtext COMMENT '解答文字',
  `questions_img` varchar(255) NOT NULL COMMENT '圖片',
  `solution_img` varchar(255) NOT NULL COMMENT '解答圖檔',
  `solution_video` varchar(255) DEFAULT NULL COMMENT '作答詳解影片檔',
  `action_log_player` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '本題操作行為的紀錄撥放器',
  `correct_answer` varchar(255) NOT NULL COMMENT 'literacy_interactive_answer sn',
  `src` varchar(255) NOT NULL,
  `used` tinyint(1) NOT NULL DEFAULT '1' COMMENT '1 顯示 0不顯示',
  `api_config_sn` int UNSIGNED DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='素養題-互動式';

-- --------------------------------------------------------

--
-- 資料表結構 `concept_itemBank_option`
--

CREATE TABLE `concept_itemBank_option` (
  `option_sn` int UNSIGNED NOT NULL,
  `item_sn` bigint UNSIGNED NOT NULL,
  `option_number` tinyint UNSIGNED NOT NULL,
  `option_text` text NOT NULL,
  `option_reaction_type` text,
  `updater` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL,
  `creation_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `concept_itemBank_option_0320`
--

CREATE TABLE `concept_itemBank_option_0320` (
  `option_sn` int UNSIGNED NOT NULL,
  `item_sn` bigint UNSIGNED NOT NULL,
  `option_number` tinyint UNSIGNED NOT NULL,
  `option_text` text CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci,
  `option_reaction_type` text,
  `updater` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL,
  `creation_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `concept_itemBank_option_0710`
--

CREATE TABLE `concept_itemBank_option_0710` (
  `option_sn` int UNSIGNED NOT NULL,
  `item_sn` bigint UNSIGNED NOT NULL,
  `option_number` tinyint UNSIGNED NOT NULL,
  `option_text` text CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci,
  `option_reaction_type` text,
  `updater` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL,
  `creation_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `concept_item_interactive`
--

CREATE TABLE `concept_item_interactive` (
  `item_sn` bigint UNSIGNED NOT NULL COMMENT '關聯',
  `sems` varchar(3) NOT NULL COMMENT '學年度',
  `template_id` varchar(2) NOT NULL COMMENT '試題類別',
  `map_sn` int NOT NULL,
  `subject_id` smallint UNSIGNED NOT NULL COMMENT '科目',
  `indicator` varchar(50) NOT NULL COMMENT '能力指標',
  `filename` mediumtext NOT NULL COMMENT '題目檔名',
  `op_filename` mediumtext NOT NULL COMMENT '選項檔名',
  `detail_filename` text COMMENT '本題詳解',
  `op_ans` tinyint UNSIGNED NOT NULL COMMENT '答案',
  `public` tinyint UNSIGNED NOT NULL COMMENT '是否公開(1=可；0=否；2=被刪除)',
  `create_user_id` varchar(60) DEFAULT NULL COMMENT '建立者	',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `concept_its`
--

CREATE TABLE `concept_its` (
  `its_sn` int NOT NULL COMMENT '流水號',
  `its_name` varchar(100) NOT NULL COMMENT '中文名稱',
  `indicate_id` varchar(25) NOT NULL COMMENT '能力指標代號',
  `subject_id` smallint UNSIGNED NOT NULL COMMENT '科目',
  `grade` int DEFAULT NULL,
  `item_difficulty` int DEFAULT NULL COMMENT '難易度',
  `item_num` tinyint DEFAULT NULL COMMENT '元件編號',
  `file_type` varchar(100) NOT NULL COMMENT '副檔名',
  `type` varchar(10) NOT NULL COMMENT '來源類型 C,外部連結(API)	',
  `display` tinyint(1) NOT NULL COMMENT '是否顯示',
  `file_name` varchar(40) NOT NULL COMMENT '程式檔名',
  `para1` varchar(10) DEFAULT NULL,
  `para2` varchar(10) DEFAULT NULL,
  `para3` varchar(10) DEFAULT NULL,
  `para4` varchar(10) DEFAULT NULL,
  `folder_name` varchar(20) DEFAULT NULL COMMENT '資料夾名稱'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='互動教學元件';

-- --------------------------------------------------------

--
-- 資料表結構 `concept_its_exam_record`
--

CREATE TABLE `concept_its_exam_record` (
  `sn` int UNSIGNED NOT NULL COMMENT '資料流水號',
  `user_id` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '使用者id',
  `concept_ist_sn` int NOT NULL COMMENT 'concept_ist_sn 編號',
  `time_spent` int UNSIGNED NOT NULL COMMENT '花費時間(毫秒)',
  `time_created` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '建立時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `concept_paper`
--

CREATE TABLE `concept_paper` (
  `paper_sn` bigint UNSIGNED NOT NULL,
  `name` mediumtext NOT NULL COMMENT '試卷名稱',
  `sems` varchar(4) NOT NULL,
  `publisher` smallint UNSIGNED NOT NULL,
  `subject` smallint UNSIGNED NOT NULL,
  `grade` int NOT NULL,
  `unit` int NOT NULL,
  `ower_id` varchar(60) DEFAULT NULL COMMENT '建立試卷者id',
  `item_sn` mediumtext NOT NULL COMMENT '選擇的試題sn',
  `public` int NOT NULL COMMENT '0unlock(未鎖定);1lock(顯示);2hide(隱藏)',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `concept_priori`
--

CREATE TABLE `concept_priori` (
  `cp_sn` int NOT NULL,
  `cp_id` varchar(20) NOT NULL,
  `subject_id` smallint UNSIGNED NOT NULL,
  `organization_id` varchar(10) DEFAULT NULL,
  `grade` int DEFAULT '0' COMMENT '年級',
  `exam_range` varchar(100) NOT NULL,
  `concept` varchar(100) NOT NULL,
  `import_date` datetime NOT NULL,
  `ftp_import_filename` varchar(200) DEFAULT NULL COMMENT 'ftp匯入檔案名稱',
  `create_user` varchar(60) DEFAULT NULL,
  `version` tinyint NOT NULL COMMENT '匯入次數',
  `exam_type` varchar(10) NOT NULL COMMENT '空白:補救教學,A:縣市學力測驗;B:學力考古題C會考考古題',
  `indicator_item` mediumtext NOT NULL COMMENT '能力指標',
  `indicator_priori` mediumtext NOT NULL COMMENT '基本學習內容',
  `status1_num` mediumtext NOT NULL,
  `status2_num` mediumtext NOT NULL,
  `status3_num` mediumtext NOT NULL,
  `total_num` mediumtext NOT NULL,
  `cp_info_sn` int DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='補救教學測驗(主檔)';

-- --------------------------------------------------------

--
-- 資料表結構 `concept_priori_info`
--

CREATE TABLE `concept_priori_info` (
  `sn` int NOT NULL,
  `subject_id` smallint UNSIGNED NOT NULL,
  `exam_range` varchar(100) NOT NULL,
  `concept` varchar(100) NOT NULL,
  `import_date` datetime NOT NULL,
  `file_name` varchar(200) NOT NULL,
  `create_user` varchar(60) DEFAULT NULL,
  `exam_type` varchar(10) NOT NULL COMMENT '空白:學習扶助，A：學力',
  `indicator_item` mediumtext NOT NULL,
  `indicator_priori` mediumtext NOT NULL,
  `status1_num` mediumtext NOT NULL,
  `status2_num` mediumtext NOT NULL,
  `status3_num` mediumtext NOT NULL,
  `total_num` mediumtext NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `concept_remedy`
--

CREATE TABLE `concept_remedy` (
  `sn` int UNSIGNED NOT NULL,
  `cs_id` varchar(12) NOT NULL,
  `remedy_sn` smallint NOT NULL,
  `remedy` varchar(255) NOT NULL,
  `threshold` tinyint NOT NULL DEFAULT '70',
  `instruction` varchar(50) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `contestexam_item`
--

CREATE TABLE `contestexam_item` (
  `item_sn` int UNSIGNED NOT NULL COMMENT '試題流水號',
  `map_sn` int NOT NULL COMMENT '星空圖流水號',
  `exam_type` tinyint UNSIGNED NOT NULL COMMENT '大考類型',
  `item_type` tinyint UNSIGNED NOT NULL COMMENT '題型',
  `item_title` varchar(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '題目標題',
  `main_filename` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '題目主檔名稱',
  `sub_filename` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '題目副檔名稱',
  `solution_filename` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '詳解檔名稱',
  `answer_option_type` tinyint UNSIGNED DEFAULT NULL COMMENT '答案選項類型',
  `keyboard_sn` smallint UNSIGNED DEFAULT NULL COMMENT '小鍵盤流水號',
  `status` tinyint UNSIGNED NOT NULL DEFAULT '0' COMMENT '試題狀態: 0(未上鎖), 1(已上鎖), 2(已刪除)	',
  `updater` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '更新者',
  `creation_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '建立時間',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='競賽試題';

-- --------------------------------------------------------

--
-- 資料表結構 `contestexam_itemgroup`
--

CREATE TABLE `contestexam_itemgroup` (
  `group_sn` int UNSIGNED NOT NULL COMMENT '題組流水號',
  `map_sn` int NOT NULL COMMENT '星空圖流水號',
  `exam_type` tinyint UNSIGNED NOT NULL COMMENT '大考類型',
  `group_title` varchar(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '題組標題',
  `group_filename` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '題組檔名稱',
  `status` tinyint UNSIGNED NOT NULL DEFAULT '0' COMMENT '題組狀態: 0(未上鎖), 1(已上鎖), 2(已刪除)	',
  `updater` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '更新者',
  `creation_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '建立時間',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='競賽-題組';

-- --------------------------------------------------------

--
-- 資料表結構 `contestexam_itemgroup_detail`
--

CREATE TABLE `contestexam_itemgroup_detail` (
  `sn` int UNSIGNED NOT NULL COMMENT '流水號',
  `group_sn` int UNSIGNED NOT NULL COMMENT '題組流水號',
  `item_sn` int UNSIGNED NOT NULL COMMENT '試題流水號',
  `item_order` tinyint UNSIGNED NOT NULL COMMENT '試題順序',
  `updater` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '更新者',
  `creation_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '建立時間',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='競賽-題組詳細內容';

-- --------------------------------------------------------

--
-- 資料表結構 `contestexam_item_answer_option`
--

CREATE TABLE `contestexam_item_answer_option` (
  `option_sn` int UNSIGNED NOT NULL COMMENT '選項流水號',
  `item_sn` int UNSIGNED NOT NULL COMMENT '試題流水號',
  `option_number` tinyint UNSIGNED NOT NULL COMMENT '選項編號',
  `answer_filename` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '選項檔名稱',
  `answer_text` varchar(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '文字答案',
  `is_correct` tinyint UNSIGNED NOT NULL DEFAULT '0' COMMENT '是否為正確答案',
  `updater` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '更新者',
  `creation_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '建立時間',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='競賽試題-答案選項';

-- --------------------------------------------------------

--
-- 資料表結構 `contestexam_item_indicator_mapping`
--

CREATE TABLE `contestexam_item_indicator_mapping` (
  `sn` int UNSIGNED NOT NULL COMMENT '流水號',
  `item_sn` int UNSIGNED NOT NULL COMMENT '試題流水號',
  `map_sn` int NOT NULL COMMENT '星空圖流水號',
  `indicate_id` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '能力指標',
  `updater` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '更新者',
  `creation_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '建立時間',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='競賽試題-能力指標對應';

-- --------------------------------------------------------

--
-- 資料表結構 `contestexam_old_uploaded_filename`
--

CREATE TABLE `contestexam_old_uploaded_filename` (
  `sn` int UNSIGNED NOT NULL,
  `filename` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `contestexam_paper`
--

CREATE TABLE `contestexam_paper` (
  `paper_sn` int UNSIGNED NOT NULL COMMENT '卷流水號',
  `year` tinyint UNSIGNED NOT NULL COMMENT '學年度',
  `map_sn` int NOT NULL COMMENT '星空圖流水號',
  `exam_type` tinyint UNSIGNED NOT NULL COMMENT '大考類型',
  `paper_name` varchar(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '卷名',
  `status` tinyint UNSIGNED NOT NULL DEFAULT '0' COMMENT '試卷狀態: 0(未上鎖), 1(已上鎖), 2(已刪除), 3(已上鎖(但未公開))',
  `updater` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '更新者',
  `creation_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '建立時間',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='競賽-試題卷';

-- --------------------------------------------------------

--
-- 資料表結構 `contestexam_paper_detail`
--

CREATE TABLE `contestexam_paper_detail` (
  `sn` int UNSIGNED NOT NULL COMMENT '流水號',
  `paper_sn` int UNSIGNED NOT NULL COMMENT '卷流水號',
  `group_sn` int UNSIGNED DEFAULT NULL COMMENT '題組流水號',
  `item_sn` int UNSIGNED NOT NULL COMMENT '試題流水號',
  `item_order` tinyint UNSIGNED NOT NULL COMMENT '題組/試題順序',
  `updater` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '更新者',
  `creation_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '建立時間',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='競賽-試題卷詳細內容';

-- --------------------------------------------------------

--
-- 資料表結構 `contestexam_record`
--

CREATE TABLE `contestexam_record` (
  `record_sn` int UNSIGNED NOT NULL COMMENT '紀錄流水號',
  `user_id` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '使用者ID',
  `paper_sn` int UNSIGNED NOT NULL COMMENT '卷流水號',
  `mission_sn` int DEFAULT NULL COMMENT '任務流水號',
  `is_finished` tinyint UNSIGNED NOT NULL DEFAULT '0' COMMENT '是否已完成',
  `finished_time` datetime DEFAULT NULL COMMENT '完成時間',
  `correct_rate` int DEFAULT NULL COMMENT '答對率(完成後計算存放)',
  `creation_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '建立時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='競賽-測驗紀錄';

-- --------------------------------------------------------

--
-- 資料表結構 `contestexam_record_connection_log_nouse`
--

CREATE TABLE `contestexam_record_connection_log_nouse` (
  `log_sn` int NOT NULL COMMENT '紀錄流水號',
  `record_sn` int NOT NULL COMMENT '紀錄流水號',
  `action` varchar(20) NOT NULL COMMENT '動作',
  `timeleft` int NOT NULL COMMENT '倒數時間',
  `creation_time` datetime(3) NOT NULL DEFAULT CURRENT_TIMESTAMP(3) ON UPDATE CURRENT_TIMESTAMP(3) COMMENT '建立時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='競賽-測驗紀錄';

-- --------------------------------------------------------

--
-- 資料表結構 `contestexam_record_detail`
--

CREATE TABLE `contestexam_record_detail` (
  `sn` int UNSIGNED NOT NULL COMMENT '流水號',
  `record_sn` int UNSIGNED NOT NULL COMMENT '紀錄流水號',
  `group_sn` int UNSIGNED DEFAULT NULL COMMENT '題組流水號',
  `item_sn` int UNSIGNED NOT NULL COMMENT '試題流水號',
  `option_sn` int UNSIGNED DEFAULT NULL COMMENT '選項流水號',
  `answer_text` varchar(1000) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '文字作答',
  `time_spent` int UNSIGNED NOT NULL COMMENT '花費時間(毫秒)',
  `creation_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '建立時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='競賽-測驗紀錄詳細內容';

-- --------------------------------------------------------

--
-- 資料表結構 `contest_agreement`
--

CREATE TABLE `contest_agreement` (
  `sn` int NOT NULL,
  `contest_sn` int NOT NULL COMMENT '競賽sn',
  `user_id` varchar(60) NOT NULL,
  `agree_date` datetime NOT NULL,
  `used` int NOT NULL DEFAULT '1' COMMENT '0不同意1同意'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `contest_certificate`
--

CREATE TABLE `contest_certificate` (
  `sn` int NOT NULL,
  `user_id` varchar(60) NOT NULL,
  `org_id` varchar(10) DEFAULT NULL,
  `contest_sn` int NOT NULL COMMENT '競賽SN',
  `certificate_sn` varchar(15) NOT NULL COMMENT '證書編號',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `contest_content`
--

CREATE TABLE `contest_content` (
  `content_sn` int NOT NULL,
  `contest_sn` int NOT NULL COMMENT '競賽類別',
  `group_sn` int NOT NULL COMMENT '團體組sn',
  `teacher_id` varchar(60) DEFAULT NULL COMMENT '指派人ID',
  `node` text NOT NULL,
  `topic_num` int NOT NULL COMMENT '題目數量',
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '建立時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `contest_dailyreport`
--

CREATE TABLE `contest_dailyreport` (
  `sn` int NOT NULL,
  `org_id` varchar(10) NOT NULL,
  `contest_sn` int NOT NULL COMMENT '競賽sn',
  `warn_sum` int DEFAULT NULL COMMENT '暖身賽人數',
  `challen_sum` int DEFAULT NULL COMMENT '挑戰賽人數',
  `trylotto_sum` int DEFAULT NULL COMMENT '試試如意獎人數',
  `tirelesslotto_sum` int DEFAULT NULL,
  `toplotto_sum` int DEFAULT NULL COMMENT '登峰造極獎人數',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `contest_group`
--

CREATE TABLE `contest_group` (
  `sn` int NOT NULL,
  `contest_sn` int NOT NULL COMMENT '競賽sn',
  `group_name` varchar(100) NOT NULL,
  `org_id` varchar(10) NOT NULL,
  `grade` int NOT NULL DEFAULT '0' COMMENT '年級',
  `class` int NOT NULL DEFAULT '0' COMMENT '班級',
  `teacher_id` varchar(60) DEFAULT NULL COMMENT '組隊老師',
  `teacher_name` varchar(60) NOT NULL COMMENT '老師名稱',
  `jobtitle` varchar(50) NOT NULL COMMENT '職稱',
  `email` varchar(100) NOT NULL COMMENT '電子郵件',
  `phone` varchar(20) NOT NULL COMMENT '連絡電話',
  `signupdate` datetime NOT NULL COMMENT '實際填表報名時間',
  `date` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '建立時間',
  `used` int NOT NULL COMMENT '0:取消報名;1:符合報名資格'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `contest_group_changelog`
--

CREATE TABLE `contest_group_changelog` (
  `sn` int NOT NULL,
  `group_name` varchar(100) NOT NULL,
  `org_id` varchar(10) NOT NULL,
  `teacher_id` varchar(60) DEFAULT NULL COMMENT '組隊老師',
  `email` varchar(100) NOT NULL,
  `contest_sn` int NOT NULL COMMENT '競賽sn',
  `date` datetime NOT NULL COMMENT '異動時間',
  `used` int NOT NULL COMMENT '0:取消報名;1:符合報名資格'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `contest_group_comparison`
--

CREATE TABLE `contest_group_comparison` (
  `sn` int NOT NULL COMMENT '流水號',
  `warm_sn` int NOT NULL COMMENT '暖身賽group_sn',
  `challenge_sn` int NOT NULL COMMENT '挑戰賽group_sn'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `contest_info`
--

CREATE TABLE `contest_info` (
  `sn` int NOT NULL,
  `contest_code` varchar(50) NOT NULL COMMENT '競賽代號',
  `contest_name` varchar(100) NOT NULL COMMENT '競賽名稱',
  `contest_type` varchar(10) NOT NULL COMMENT '競賽類別(warmup:暖身, challenge:挑戰)',
  `repeat_answer` int NOT NULL DEFAULT '0' COMMENT '0:不能重複作答;1:能重複作答',
  `sign_start_time` datetime NOT NULL COMMENT '報名開始時間',
  `sign_end_time` datetime NOT NULL COMMENT '報名結束時間',
  `contest_start_time` datetime NOT NULL COMMENT '比賽起始時間',
  `contest_end_time` datetime NOT NULL COMMENT '比賽結束時間',
  `used` tinyint NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `contest_itembank`
--

CREATE TABLE `contest_itembank` (
  `item_li_sn` bigint UNSIGNED NOT NULL,
  `contest_sn` int DEFAULT NULL,
  `subject_id` smallint UNSIGNED NOT NULL,
  `map_sn` int NOT NULL,
  `question_num` int NOT NULL,
  `sub_subject_id` tinyint UNSIGNED DEFAULT NULL COMMENT '子科目 (若非子科目則為: 0)',
  `indicator` varchar(50) NOT NULL COMMENT '能力指標',
  `item_show_num` int(3) UNSIGNED ZEROFILL DEFAULT NULL COMMENT '上架後試題編號',
  `theme` varchar(25) NOT NULL COMMENT '題目主題 (代數,數量,統計...)',
  `item_name` varchar(60) NOT NULL COMMENT '題目標題	',
  `learning_stage` tinyint(1) NOT NULL COMMENT '學習階段',
  `grade` int UNSIGNED DEFAULT NULL,
  `type` tinyint UNSIGNED NOT NULL DEFAULT '1' COMMENT '1:互動題，0:一般題',
  `thought_type` varchar(10) NOT NULL COMMENT '思考類型',
  `core` varchar(255) NOT NULL COMMENT '核心素養',
  `core_code` varchar(100) DEFAULT NULL COMMENT '學習內容',
  `learning_content` varchar(100) DEFAULT NULL COMMENT '核心素養代碼',
  `core_code_list` varchar(100) DEFAULT NULL COMMENT '核心素養代碼',
  `learning_content_list` varchar(100) DEFAULT NULL COMMENT '題目核心素養代號',
  `item_situation` varchar(10) NOT NULL COMMENT '試題情境	',
  `questions_img` varchar(255) NOT NULL COMMENT '圖片',
  `solution_img` varchar(255) NOT NULL COMMENT '解答圖檔',
  `solution_video` varchar(255) DEFAULT NULL COMMENT '作答詳解影片檔',
  `correct_answer` varchar(255) NOT NULL COMMENT 'literacy_interactive_answer sn',
  `src` varchar(255) NOT NULL,
  `used` tinyint(1) NOT NULL DEFAULT '1' COMMENT '1 顯示 0不顯示',
  `api_config_sn` int UNSIGNED DEFAULT NULL,
  `answerable_time` int NOT NULL DEFAULT '0' COMMENT '可作答時間(分鐘)'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='素養題-互動式';

-- --------------------------------------------------------

--
-- 資料表結構 `contest_itembank_2021mathscience`
--

CREATE TABLE `contest_itembank_2021mathscience` (
  `item_li_sn` bigint UNSIGNED NOT NULL,
  `contest_sn` int NOT NULL,
  `subject_id` smallint UNSIGNED NOT NULL,
  `map_sn` int NOT NULL,
  `question_num` int NOT NULL,
  `sub_subject_id` tinyint UNSIGNED DEFAULT NULL COMMENT '子科目',
  `theme` varchar(25) NOT NULL COMMENT '題目主題 (代數,數量,統計...)',
  `item_name` varchar(40) NOT NULL COMMENT '題目標題	',
  `learning_stage` tinyint(1) NOT NULL COMMENT '學習階段',
  `type` tinyint UNSIGNED NOT NULL DEFAULT '1' COMMENT '1:互動題，0:一般題',
  `thought_type` varchar(10) NOT NULL COMMENT '思考類型',
  `core` varchar(255) NOT NULL COMMENT '核心素養',
  `item_situation` varchar(10) NOT NULL COMMENT '試題情境	',
  `questions_img` varchar(255) NOT NULL COMMENT '圖片',
  `solution_img` varchar(255) NOT NULL COMMENT '解答圖檔',
  `correct_answer` varchar(255) NOT NULL COMMENT 'literacy_interactive_answer sn',
  `src` varchar(255) NOT NULL,
  `used` tinyint(1) NOT NULL DEFAULT '1' COMMENT '1 顯示 0不顯示',
  `answerable_time` int NOT NULL DEFAULT '0' COMMENT '可作答時間(分鐘)'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='素養題-互動式';

-- --------------------------------------------------------

--
-- 資料表結構 `contest_itembank_2022math`
--

CREATE TABLE `contest_itembank_2022math` (
  `item_li_sn` bigint UNSIGNED NOT NULL,
  `contest_sn` int DEFAULT NULL,
  `subject_id` tinyint UNSIGNED NOT NULL,
  `map_sn` int NOT NULL,
  `question_num` int NOT NULL,
  `sub_subject_id` tinyint UNSIGNED DEFAULT NULL COMMENT '子科目 (若非子科目則為: 0)',
  `indicator` varchar(50) NOT NULL COMMENT '能力指標',
  `item_show_num` int(3) UNSIGNED ZEROFILL DEFAULT NULL COMMENT '上架後試題編號',
  `theme` varchar(25) NOT NULL COMMENT '題目主題 (代數,數量,統計...)',
  `item_name` varchar(60) NOT NULL COMMENT '題目標題	',
  `learning_stage` tinyint(1) NOT NULL COMMENT '學習階段',
  `grade` int UNSIGNED DEFAULT NULL,
  `type` tinyint UNSIGNED NOT NULL DEFAULT '1' COMMENT '1:互動題，0:一般題',
  `thought_type` varchar(10) NOT NULL COMMENT '思考類型',
  `core` varchar(255) NOT NULL COMMENT '核心素養',
  `core_code` varchar(100) DEFAULT NULL COMMENT '學習內容',
  `learning_content` varchar(100) DEFAULT NULL COMMENT '核心素養代碼',
  `item_situation` varchar(10) NOT NULL COMMENT '試題情境	',
  `questions_img` varchar(255) NOT NULL COMMENT '圖片',
  `solution_img` varchar(255) NOT NULL COMMENT '解答圖檔',
  `solution_video` varchar(255) DEFAULT NULL COMMENT '作答詳解影片檔',
  `correct_answer` varchar(255) NOT NULL COMMENT 'literacy_interactive_answer sn',
  `src` varchar(255) NOT NULL,
  `used` tinyint(1) NOT NULL DEFAULT '1' COMMENT '1 顯示 0不顯示',
  `api_config_sn` int UNSIGNED DEFAULT NULL,
  `answerable_time` int NOT NULL DEFAULT '0' COMMENT '可作答時間(分鐘)'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='素養題-互動式';

-- --------------------------------------------------------

--
-- 資料表結構 `contest_itembank_2023mathscience`
--

CREATE TABLE `contest_itembank_2023mathscience` (
  `item_li_sn` bigint UNSIGNED NOT NULL,
  `contest_sn` int DEFAULT NULL,
  `subject_id` smallint UNSIGNED NOT NULL,
  `map_sn` int NOT NULL,
  `question_num` int NOT NULL,
  `sub_subject_id` tinyint UNSIGNED DEFAULT NULL COMMENT '子科目 (若非子科目則為: 0)',
  `indicator` varchar(50) NOT NULL COMMENT '能力指標',
  `item_show_num` int(3) UNSIGNED ZEROFILL DEFAULT NULL COMMENT '上架後試題編號',
  `theme` varchar(25) NOT NULL COMMENT '題目主題 (代數,數量,統計...)',
  `item_name` varchar(60) NOT NULL COMMENT '題目標題	',
  `learning_stage` tinyint(1) NOT NULL COMMENT '學習階段',
  `grade` int UNSIGNED DEFAULT NULL,
  `type` tinyint UNSIGNED NOT NULL DEFAULT '1' COMMENT '1:互動題，0:一般題',
  `thought_type` varchar(10) NOT NULL COMMENT '思考類型',
  `core` varchar(255) NOT NULL COMMENT '核心素養',
  `core_code` varchar(100) DEFAULT NULL COMMENT '學習內容',
  `learning_content` varchar(100) DEFAULT NULL COMMENT '核心素養代碼',
  `core_code_list` varchar(100) DEFAULT NULL COMMENT '核心素養代碼',
  `learning_content_list` varchar(100) DEFAULT NULL COMMENT '題目核心素養代號',
  `item_situation` varchar(10) NOT NULL COMMENT '試題情境	',
  `questions_img` varchar(255) NOT NULL COMMENT '圖片',
  `solution_img` varchar(255) NOT NULL COMMENT '解答圖檔',
  `solution_video` varchar(255) DEFAULT NULL COMMENT '作答詳解影片檔',
  `correct_answer` varchar(255) NOT NULL COMMENT 'literacy_interactive_answer sn',
  `src` varchar(255) NOT NULL,
  `used` tinyint(1) NOT NULL DEFAULT '1' COMMENT '1 顯示 0不顯示',
  `api_config_sn` int UNSIGNED DEFAULT NULL,
  `answerable_time` int NOT NULL DEFAULT '0' COMMENT '可作答時間(分鐘)'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='素養題-互動式';

-- --------------------------------------------------------

--
-- 資料表結構 `contest_itembank_group`
--

CREATE TABLE `contest_itembank_group` (
  `literacy_group_sn` bigint UNSIGNED NOT NULL,
  `contest_sn` int NOT NULL COMMENT '競賽類別',
  `subject_id` smallint UNSIGNED NOT NULL COMMENT '科目',
  `map_sn` int NOT NULL,
  `sub_subject_id` tinyint UNSIGNED DEFAULT NULL COMMENT '子科目',
  `item_name` varchar(25) NOT NULL COMMENT '題組名稱',
  `create_user` varchar(60) DEFAULT NULL COMMENT '建立者',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新時間',
  `status` tinyint(1) NOT NULL DEFAULT '0' COMMENT '是否上鎖(0=否；1=是；2=被刪除)'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `contest_itemBank_literacy`
--

CREATE TABLE `contest_itemBank_literacy` (
  `item_sn` bigint UNSIGNED NOT NULL,
  `contest_sn` int NOT NULL COMMENT '競賽類別',
  `subject_id` smallint UNSIGNED NOT NULL COMMENT '科目',
  `map_sn` int NOT NULL,
  `sub_subject_id` tinyint UNSIGNED DEFAULT NULL COMMENT '子科目',
  `learning_stage` tinyint(1) NOT NULL COMMENT '學習階段 1:國小第一階段 2:國小第二階段 3:國小第三階段 4:國高中',
  `literacy_group_sn` bigint UNSIGNED DEFAULT NULL COMMENT '對應題組的sn，若無則為單一題資料為NULL',
  `indicator` mediumtext NOT NULL COMMENT '本題對應到的能力指標',
  `item_situation` mediumtext NOT NULL COMMENT '試題情境',
  `item_type` tinyint NOT NULL COMMENT '題型類別(0=選擇題；1=是非題)',
  `item_name` varchar(25) NOT NULL COMMENT '題目標題',
  `filename` mediumtext NOT NULL COMMENT '題目檔名',
  `op_filename` mediumtext NOT NULL COMMENT '選項檔名',
  `answer` tinyint UNSIGNED NOT NULL COMMENT '答案',
  `thought_type` varchar(10) DEFAULT NULL COMMENT '思考類型',
  `learning_performance` varchar(20) DEFAULT NULL COMMENT '學習表現',
  `core` mediumtext NOT NULL COMMENT '核心素養',
  `cross_subject` mediumtext NOT NULL COMMENT '本題是否與其他科目有關連',
  `item_status` tinyint NOT NULL DEFAULT '0' COMMENT '是否上鎖(0=否；1=是；2=被刪除)',
  `create_user` varchar(60) DEFAULT NULL COMMENT '建立者',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `contest_itembank_summer`
--

CREATE TABLE `contest_itembank_summer` (
  `item_li_sn` bigint UNSIGNED NOT NULL,
  `contest_sn` int NOT NULL,
  `subject_id` smallint UNSIGNED NOT NULL,
  `map_sn` int NOT NULL,
  `question_num` int NOT NULL,
  `sub_subject_id` tinyint UNSIGNED DEFAULT NULL COMMENT '子科目',
  `theme` varchar(25) NOT NULL COMMENT '題目主題 (代數,數量,統計...)',
  `item_name` varchar(40) NOT NULL COMMENT '題目標題	',
  `learning_stage` tinyint(1) NOT NULL COMMENT '學習階段',
  `type` tinyint UNSIGNED NOT NULL DEFAULT '1' COMMENT '1:互動題，0:一般題',
  `thought_type` varchar(10) NOT NULL COMMENT '思考類型',
  `core` varchar(255) NOT NULL COMMENT '核心素養',
  `item_situation` varchar(10) NOT NULL COMMENT '試題情境	',
  `questions_img` varchar(255) NOT NULL COMMENT '圖片',
  `solution_img` varchar(255) NOT NULL COMMENT '解答圖檔',
  `correct_answer` varchar(255) NOT NULL COMMENT 'literacy_interactive_answer sn',
  `src` varchar(255) NOT NULL,
  `used` tinyint(1) NOT NULL DEFAULT '1' COMMENT '1 顯示 0不顯示',
  `answerable_time` int NOT NULL DEFAULT '0' COMMENT '可作答時間(分鐘)'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='素養題-互動式';

-- --------------------------------------------------------

--
-- 資料表結構 `contest_publisher_mapping`
--

CREATE TABLE `contest_publisher_mapping` (
  `literacy_publisher_sn` int UNSIGNED NOT NULL COMMENT '流水號',
  `literacy_item_sn` int NOT NULL COMMENT '素養試題編號(1-concept_itemBank_literacy_interactive/2-concept_itemBank_literacy/3-concept_itemBank_literacy_group)',
  `type` int NOT NULL COMMENT '素養試題類型(互動1/一般2/題組3)',
  `grade` int NOT NULL COMMENT '年級',
  `seme` varchar(4) NOT NULL COMMENT '學年度學期',
  `publisher_id` smallint UNSIGNED NOT NULL COMMENT '版本',
  `unit` int NOT NULL COMMENT '單元',
  `unit_name` varchar(255) NOT NULL COMMENT '單元名稱',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `contest_publisher_mapping_2021mathscience`
--

CREATE TABLE `contest_publisher_mapping_2021mathscience` (
  `literacy_publisher_sn` int UNSIGNED NOT NULL COMMENT '流水號',
  `literacy_item_sn` int NOT NULL COMMENT '素養試題編號(1-concept_itemBank_literacy_interactive/2-concept_itemBank_literacy/3-concept_itemBank_literacy_group)',
  `type` int NOT NULL COMMENT '素養試題類型(互動1/一般2/題組3)',
  `grade` int NOT NULL COMMENT '年級',
  `seme` varchar(4) NOT NULL COMMENT '學年度學期',
  `publisher_id` smallint UNSIGNED NOT NULL COMMENT '版本',
  `unit` int NOT NULL COMMENT '單元',
  `unit_name` varchar(255) NOT NULL COMMENT '單元名稱',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `contest_publisher_mapping_2022math`
--

CREATE TABLE `contest_publisher_mapping_2022math` (
  `literacy_publisher_sn` int UNSIGNED NOT NULL COMMENT '流水號',
  `literacy_item_sn` int NOT NULL COMMENT '素養試題編號(1-concept_itemBank_literacy_interactive/2-concept_itemBank_literacy/3-concept_itemBank_literacy_group)',
  `type` int NOT NULL COMMENT '素養試題類型(互動1/一般2/題組3)',
  `grade` int NOT NULL COMMENT '年級',
  `seme` varchar(4) NOT NULL COMMENT '學年度學期',
  `publisher_id` smallint UNSIGNED NOT NULL COMMENT '版本',
  `unit` int NOT NULL COMMENT '單元',
  `unit_name` varchar(255) NOT NULL COMMENT '單元名稱',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `contest_record`
--

CREATE TABLE `contest_record` (
  `exam_sn` bigint NOT NULL,
  `contest_info_sn` int NOT NULL COMMENT '競賽sn',
  `mission_sn` int DEFAULT NULL COMMENT '任務sn,若無則NULL',
  `subject_id` smallint UNSIGNED DEFAULT NULL,
  `map_sn` int DEFAULT NULL,
  `item_li_sn` bigint UNSIGNED NOT NULL COMMENT '互動式素養題編號',
  `user_id` varchar(60) NOT NULL,
  `start_time` datetime NOT NULL,
  `questions_type` mediumtext NOT NULL COMMENT '  1:是非題   2:選擇題 1234   3:文字回答, 只要有文字題就分類為文字   4:複選 123456789   5:順序題(拖拉題) (拖拉順序:)   6:回傳base64圖檔',
  `client_items_idle_time` mediumtext COMMENT '作答時間(毫秒)',
  `stuAns` longtext NOT NULL COMMENT '根據作答順序紀錄',
  `binary_res` mediumtext NOT NULL COMMENT '根據作答順序紀錄',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `over_time` int NOT NULL DEFAULT '0'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `contest_record_literacy`
--

CREATE TABLE `contest_record_literacy` (
  `exam_sn` bigint NOT NULL,
  `contest_sn` int NOT NULL COMMENT '	',
  `user_id` varchar(60) NOT NULL,
  `start_time` datetime NOT NULL,
  `void` tinyint NOT NULL DEFAULT '0' COMMENT '1:作廢 0:正常的作答記錄	',
  `client_items_idle_time` mediumtext COMMENT '作答時間(毫秒)',
  `literacy_group_sn` mediumtext COMMENT '題組題的group_sn',
  `questions` mediumtext COMMENT '試題代號',
  `stu_qusetion` mediumtext COMMENT '根據作答順序紀錄',
  `stu_answer` mediumtext COMMENT '根據作答順序紀錄',
  `binary_res` mediumtext COMMENT '二元作答反應',
  `correct_rate` tinyint UNSIGNED DEFAULT NULL COMMENT '答對率，最高100',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `contest_record_respond`
--

CREATE TABLE `contest_record_respond` (
  `sn` int NOT NULL,
  `user_id` varchar(60) NOT NULL,
  `contest_sn` int NOT NULL COMMENT '競賽SN',
  `org_id` varchar(10) NOT NULL,
  `ansCount` int NOT NULL COMMENT '答題數(小題)',
  `ansCurrent` int NOT NULL COMMENT '答對題數(小題)',
  `ansRowCount` int NOT NULL COMMENT '答題_題組數',
  `currentPercent` int NOT NULL COMMENT '答對率',
  `type` tinyint NOT NULL COMMENT '1:個人;2:團體',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `contest_signup`
--

CREATE TABLE `contest_signup` (
  `sn` int NOT NULL,
  `user_id` varchar(60) NOT NULL,
  `email` varchar(100) DEFAULT NULL COMMENT '電子郵件',
  `contest_sn` int NOT NULL COMMENT '競賽sn',
  `type` tinyint NOT NULL COMMENT '1:個人;2:團體',
  `group_sn` int DEFAULT NULL COMMENT '團體sn',
  `organization_id` varchar(10) DEFAULT NULL COMMENT '因報表所需，非190046則記錄原本的學校	',
  `school_system` int NOT NULL COMMENT '學制(0：沒有分類、1：普通、2：技術)	',
  `date` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `used` int NOT NULL DEFAULT '1' COMMENT '0:取消報名;1:符合報名資格',
  `agree_time` datetime DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `contest_signup_20220422`
--

CREATE TABLE `contest_signup_20220422` (
  `sn` int NOT NULL,
  `user_id` varchar(60) NOT NULL,
  `email` varchar(100) NOT NULL,
  `contest_sn` int NOT NULL COMMENT '競賽sn',
  `type` tinyint NOT NULL COMMENT '1:個人;2:團體',
  `group_sn` int DEFAULT NULL COMMENT '團體sn',
  `organization_id` varchar(10) DEFAULT NULL COMMENT '因報表所需，非190046則記錄原本的學校	',
  `school_system` int NOT NULL COMMENT '學制(0：沒有分類、1：普通、2：技術)	',
  `date` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `used` int NOT NULL DEFAULT '1' COMMENT '0:取消報名;1:符合報名資格'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `contest_signup_changelog`
--

CREATE TABLE `contest_signup_changelog` (
  `sn` int NOT NULL,
  `user_id` varchar(60) NOT NULL,
  `email` varchar(100) DEFAULT NULL,
  `contest_sn` int NOT NULL COMMENT '競賽sn',
  `type` tinyint NOT NULL COMMENT '1:個人;2:團體',
  `group_sn` int DEFAULT NULL COMMENT '團體sn',
  `date` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '異動時間',
  `used` int NOT NULL DEFAULT '1' COMMENT '0:取消報名;1:符合報名資格'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `conversation`
--

CREATE TABLE `conversation` (
  `sn` int NOT NULL,
  `user_id` varchar(60) NOT NULL,
  `robot_type` tinyint NOT NULL COMMENT '1：通用式/2：診斷式/3：星空圖/4：任務/5：ITS/6:學習夥伴/7:素養題',
  `prompt_type` int NOT NULL DEFAULT '1' COMMENT '1：一般、2：自然探索、3：自主學習、4：寫作、5：繪圖、6：樂學、7：文思、8：認知診斷',
  `completed_flag` tinyint NOT NULL DEFAULT '0' COMMENT '判斷對話是否完成',
  `exam_sn` bigint DEFAULT NULL COMMENT '診斷報告sn',
  `mission_sn` int DEFAULT NULL COMMENT '任務sn',
  `item_sn` bigint UNSIGNED DEFAULT NULL COMMENT '題目sn concept_itemBank',
  `map_sn` int DEFAULT NULL COMMENT '星空圖使用',
  `indicator` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '星空圖使用',
  `creation_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '建立時間',
  `finished_time` datetime NOT NULL DEFAULT '0000-00-00 00:00:00',
  `item_li_sn` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '素養題單元(concept_itemBank_literacy_conversation.item_li_sn)',
  `item_li_sn_type` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '素養題單元-題目(concept_itemBank_literacy_conversation.item_li_theme_number)',
  `ai_service` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT 'AI服務'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `conversation_access`
--

CREATE TABLE `conversation_access` (
  `sn` int NOT NULL,
  `semester` varchar(4) NOT NULL COMMENT '學期',
  `user_id` varchar(60) NOT NULL,
  `subject_conversation` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL,
  `normal_conversation` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `conversation_detail`
--

CREATE TABLE `conversation_detail` (
  `sn` int NOT NULL,
  `conversation_sn` int NOT NULL,
  `role` varchar(20) NOT NULL,
  `content` text NOT NULL,
  `mission_sn` int DEFAULT NULL,
  `api_return_data` text COMMENT 'API回傳的data完整內容',
  `con_feedback` varchar(20) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '使用者回饋',
  `creation_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `conversation_detail_images`
--

CREATE TABLE `conversation_detail_images` (
  `sn` int NOT NULL COMMENT 'cdi.sn',
  `conversation_detail_sn` int NOT NULL COMMENT 'conversation_detail.sn',
  `image_src` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '圖片路徑',
  `image_understanding_result` text NOT NULL COMMENT 'AI圖片理解的內容',
  `api_return_data` text NOT NULL COMMENT 'API回傳的data完整內容',
  `creation_time` datetime NOT NULL COMMENT '圖片上傳時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `conversation_detail_images_error`
--

CREATE TABLE `conversation_detail_images_error` (
  `sn` int NOT NULL,
  `conversation_detail_sn` int NOT NULL COMMENT 'conversation_detail.sn',
  `error_code` varchar(100) NOT NULL COMMENT '自訂的ERROR CODE',
  `error_message` text NOT NULL COMMENT '自訂的ERROR MESSAGE',
  `api_return_data` text COMMENT 'API回傳的data完整內容',
  `creation_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `conversation_error`
--

CREATE TABLE `conversation_error` (
  `sn` int NOT NULL,
  `conversation_sn` int DEFAULT NULL,
  `user_id` varchar(60) NOT NULL,
  `robot_type` tinyint NOT NULL COMMENT '1：通用式/2：診斷式',
  `exam_sn` bigint DEFAULT NULL COMMENT '診斷式',
  `mission_sn` int DEFAULT NULL COMMENT '任務式mission_info.mission_sn ',
  `item_sn` bigint DEFAULT NULL COMMENT '學科型有題目concept_itemBank.item_sn ',
  `map_sn` int DEFAULT NULL COMMENT '學科型map_info.map_sn',
  `indicator` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '學科型map_node.indicate_id ',
  `content` text NOT NULL,
  `error_code` int NOT NULL,
  `error_message` mediumtext CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL,
  `ai_service` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '使用的AI服務',
  `creation_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `conversation_old`
--

CREATE TABLE `conversation_old` (
  `sn` int NOT NULL,
  `user_id` varchar(60) NOT NULL,
  `robot_type` tinyint NOT NULL COMMENT '1：通用式/2：診斷式',
  `completed_flag` tinyint NOT NULL,
  `exam_sn` bigint DEFAULT NULL COMMENT '診斷報告sn',
  `mission_sn` int DEFAULT NULL COMMENT '任務sn',
  `item_sn` bigint UNSIGNED DEFAULT NULL COMMENT '題目sn',
  `map_sn` int DEFAULT NULL,
  `indicator` varchar(50) DEFAULT NULL,
  `role` varchar(20) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '角色(system、user、assistant)',
  `content` text CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '輸入或回覆內容',
  `creation_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '建立時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `conversation_times`
--

CREATE TABLE `conversation_times` (
  `sn` int NOT NULL,
  `user_id` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL,
  `conversation_sn` int NOT NULL,
  `robot_type` tinyint NOT NULL,
  `prompt_type` int DEFAULT NULL,
  `exam_sn` bigint DEFAULT NULL,
  `mission_sn` int DEFAULT NULL,
  `item_sn` bigint DEFAULT NULL,
  `map_sn` int DEFAULT NULL,
  `indicator` varchar(50) DEFAULT NULL,
  `creation_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `cooperation_city_area`
--

CREATE TABLE `cooperation_city_area` (
  `sn` int NOT NULL COMMENT '序',
  `area_code` varchar(1) NOT NULL COMMENT '北N中C南S東E',
  `area_name` varchar(2) NOT NULL COMMENT '區域名稱',
  `city_name` varchar(3) NOT NULL COMMENT '縣市名稱',
  `hs_advisory_group_area` varchar(2) NOT NULL COMMENT '高中職輔導團 北N中C南S東E'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='區域對應縣市';

-- --------------------------------------------------------

--
-- 資料表結構 `cooperation_organization`
--

CREATE TABLE `cooperation_organization` (
  `sn` int NOT NULL COMMENT '序',
  `area_code` varchar(1) NOT NULL COMMENT '區域代碼 cooperation_city_area.area_code',
  `organization_id` varchar(10) NOT NULL COMMENT '學校代碼',
  `co_type` int NOT NULL DEFAULT '1' COMMENT '預設值為1 國小/1 高中職區域/2',
  `seme` int NOT NULL COMMENT '學年度'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='區域合作學校';

-- --------------------------------------------------------

--
-- 資料表結構 `course_subject`
--

CREATE TABLE `course_subject` (
  `cs_sn` int NOT NULL,
  `type` varchar(10) NOT NULL,
  `college` varchar(30) NOT NULL,
  `department` varchar(30) NOT NULL,
  `subject_id` smallint UNSIGNED NOT NULL,
  `subject_order` tinyint UNSIGNED NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `cross_school_student`
--

CREATE TABLE `cross_school_student` (
  `cross_school_student_sn` int NOT NULL,
  `student_id` varchar(60) NOT NULL COMMENT '學生ID',
  `teacher_id` varchar(60) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='補救教學測驗(主檔)';

-- --------------------------------------------------------

--
-- 資料表結構 `csrf_security_log`
--

CREATE TABLE `csrf_security_log` (
  `id` bigint UNSIGNED NOT NULL,
  `created_at` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `event_type` varchar(50) NOT NULL COMMENT '事件類型',
  `user_id` varchar(50) DEFAULT NULL COMMENT '使用者 ID',
  `ip_address` varchar(45) NOT NULL COMMENT '來源 IP',
  `request_uri` varchar(512) NOT NULL COMMENT '請求 URI',
  `referer` varchar(512) DEFAULT NULL COMMENT 'HTTP Referer',
  `error_code` int DEFAULT NULL COMMENT 'CSRF 錯誤代碼',
  `is_blocked` tinyint(1) DEFAULT '0' COMMENT '是否已阻擋'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `ct_node_mapping`
--

CREATE TABLE `ct_node_mapping` (
  `ct_node_mapping_sn` int NOT NULL COMMENT '流水號',
  `ct_node` varchar(50) NOT NULL COMMENT '運算思維節點',
  `adl_node` varchar(50) NOT NULL COMMENT '對應因材網節點',
  `update_date` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '修改時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `datashop_data_table`
--

CREATE TABLE `datashop_data_table` (
  `id` int NOT NULL,
  `SearchType` varchar(20) NOT NULL COMMENT '搜尋類型',
  `Subject` tinyint NOT NULL COMMENT '科目',
  `StartTime` date NOT NULL COMMENT '開始時間',
  `EndTime` date NOT NULL COMMENT '結束時間',
  `LimitStart` int NOT NULL COMMENT '流水號起',
  `LimitEnd` int NOT NULL COMMENT '流水號迄',
  `LimitNum` int NOT NULL COMMENT '匯出筆數',
  `ExportTime` datetime NOT NULL COMMENT '匯出時間',
  `DataAddr` mediumtext NOT NULL COMMENT '檔案位址',
  `user_id` varchar(40) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `dbuser`
--

CREATE TABLE `dbuser` (
  `sn` int UNSIGNED NOT NULL,
  `dbusername` varchar(32) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL,
  `user_id` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `eduexam_item`
--

CREATE TABLE `eduexam_item` (
  `item_sn` int UNSIGNED NOT NULL COMMENT '試題流水號',
  `map_sn` int NOT NULL COMMENT '星空圖流水號',
  `exam_type` tinyint UNSIGNED NOT NULL COMMENT '大考類型',
  `item_type` tinyint UNSIGNED NOT NULL COMMENT '題型',
  `item_title` varchar(100) NOT NULL COMMENT '題目標題',
  `main_filename` varchar(60) NOT NULL COMMENT '題目主檔名稱',
  `sub_filename` varchar(60) DEFAULT NULL COMMENT '題目副檔名稱',
  `solution_filename` varchar(60) DEFAULT NULL COMMENT '詳解檔名稱',
  `answer_option_type` tinyint UNSIGNED DEFAULT NULL COMMENT '答案選項類型',
  `keyboard_sn` smallint UNSIGNED DEFAULT NULL COMMENT '小鍵盤流水號',
  `status` tinyint UNSIGNED NOT NULL DEFAULT '0' COMMENT '試題狀態: 0(未上鎖), 1(已上鎖), 2(已刪除)	',
  `updater` varchar(60) DEFAULT NULL COMMENT '更新者',
  `creation_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '建立時間',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='教育部大考考古題';

-- --------------------------------------------------------

--
-- 資料表結構 `eduexam_itemgroup`
--

CREATE TABLE `eduexam_itemgroup` (
  `group_sn` int UNSIGNED NOT NULL COMMENT '題組流水號',
  `map_sn` int NOT NULL COMMENT '星空圖流水號',
  `exam_type` tinyint UNSIGNED NOT NULL COMMENT '大考類型',
  `group_title` varchar(100) NOT NULL COMMENT '題組標題',
  `group_filename` varchar(60) DEFAULT NULL COMMENT '題組檔名稱',
  `status` tinyint UNSIGNED NOT NULL DEFAULT '0' COMMENT '題組狀態: 0(未上鎖), 1(已上鎖), 2(已刪除)	',
  `updater` varchar(60) DEFAULT NULL COMMENT '更新者',
  `creation_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '建立時間',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='教育部大考考古題-題組';

-- --------------------------------------------------------

--
-- 資料表結構 `eduexam_itemgroup_detail`
--

CREATE TABLE `eduexam_itemgroup_detail` (
  `sn` int UNSIGNED NOT NULL COMMENT '流水號',
  `group_sn` int UNSIGNED NOT NULL COMMENT '題組流水號',
  `item_sn` int UNSIGNED NOT NULL COMMENT '試題流水號',
  `item_order` tinyint UNSIGNED NOT NULL COMMENT '試題順序',
  `updater` varchar(60) DEFAULT NULL COMMENT '更新者',
  `creation_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '建立時間',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='教育部大考考古題-題組詳細內容';

-- --------------------------------------------------------

--
-- 資料表結構 `eduexam_item_answer_option`
--

CREATE TABLE `eduexam_item_answer_option` (
  `option_sn` int UNSIGNED NOT NULL COMMENT '選項流水號',
  `item_sn` int UNSIGNED NOT NULL COMMENT '試題流水號',
  `option_number` tinyint UNSIGNED NOT NULL COMMENT '選項編號',
  `answer_filename` varchar(60) DEFAULT NULL COMMENT '選項檔名稱',
  `answer_text` varchar(100) DEFAULT NULL COMMENT '文字答案',
  `is_correct` tinyint UNSIGNED NOT NULL DEFAULT '0' COMMENT '是否為正確答案',
  `updater` varchar(60) DEFAULT NULL COMMENT '更新者',
  `creation_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '建立時間',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='教育部大考考古題-答案選項';

-- --------------------------------------------------------

--
-- 資料表結構 `eduexam_item_indicator_mapping`
--

CREATE TABLE `eduexam_item_indicator_mapping` (
  `sn` int UNSIGNED NOT NULL COMMENT '流水號',
  `item_sn` int UNSIGNED NOT NULL COMMENT '試題流水號',
  `map_sn` int NOT NULL COMMENT '星空圖流水號',
  `indicate_id` varchar(50) NOT NULL COMMENT '能力指標',
  `updater` varchar(60) DEFAULT NULL COMMENT '更新者',
  `creation_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '建立時間',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='教育部大考考古題-能力指標對應';

-- --------------------------------------------------------

--
-- 資料表結構 `eduexam_old_uploaded_filename`
--

CREATE TABLE `eduexam_old_uploaded_filename` (
  `sn` int UNSIGNED NOT NULL,
  `filename` varchar(60) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `eduexam_paper`
--

CREATE TABLE `eduexam_paper` (
  `paper_sn` int UNSIGNED NOT NULL COMMENT '卷流水號',
  `year` tinyint UNSIGNED NOT NULL COMMENT '學年度',
  `map_sn` int NOT NULL COMMENT '星空圖流水號',
  `exam_type` tinyint UNSIGNED NOT NULL COMMENT '大考類型',
  `paper_name` varchar(100) NOT NULL COMMENT '卷名',
  `status` tinyint UNSIGNED NOT NULL DEFAULT '0' COMMENT '試卷狀態: 0(未上鎖), 1(已上鎖), 2(已刪除), 3(已上鎖(但未公開))',
  `updater` varchar(60) DEFAULT NULL COMMENT '更新者',
  `creation_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '建立時間',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='教育部大考考古題-試題卷';

-- --------------------------------------------------------

--
-- 資料表結構 `eduexam_paper_detail`
--

CREATE TABLE `eduexam_paper_detail` (
  `sn` int UNSIGNED NOT NULL COMMENT '流水號',
  `paper_sn` int UNSIGNED NOT NULL COMMENT '卷流水號',
  `group_sn` int UNSIGNED DEFAULT NULL COMMENT '題組流水號',
  `item_sn` int UNSIGNED NOT NULL COMMENT '試題流水號',
  `item_order` tinyint UNSIGNED NOT NULL COMMENT '題組/試題順序',
  `updater` varchar(60) DEFAULT NULL COMMENT '更新者',
  `creation_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '建立時間',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='教育部大考考古題-試題卷詳細內容';

-- --------------------------------------------------------

--
-- 資料表結構 `eduexam_record`
--

CREATE TABLE `eduexam_record` (
  `record_sn` int UNSIGNED NOT NULL COMMENT '紀錄流水號',
  `user_id` varchar(60) DEFAULT NULL COMMENT '使用者ID',
  `paper_sn` int UNSIGNED NOT NULL COMMENT '卷流水號',
  `mission_sn` int DEFAULT NULL COMMENT '任務流水號',
  `is_finished` tinyint UNSIGNED NOT NULL DEFAULT '0' COMMENT '是否已完成',
  `finished_time` datetime DEFAULT NULL COMMENT '完成時間',
  `creation_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '建立時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='教育部大考考古題-測驗紀錄';

-- --------------------------------------------------------

--
-- 資料表結構 `eduexam_record_detail`
--

CREATE TABLE `eduexam_record_detail` (
  `sn` int UNSIGNED NOT NULL COMMENT '流水號',
  `record_sn` int UNSIGNED NOT NULL COMMENT '紀錄流水號',
  `group_sn` int UNSIGNED DEFAULT NULL COMMENT '題組流水號',
  `item_sn` int UNSIGNED NOT NULL COMMENT '試題流水號',
  `option_sn` int UNSIGNED DEFAULT NULL COMMENT '選項流水號',
  `answer_text` varchar(1000) DEFAULT NULL COMMENT '文字作答',
  `time_spent` int UNSIGNED NOT NULL COMMENT '花費時間(毫秒)',
  `creation_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '建立時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='教育部大考考古題-測驗紀錄詳細內容';

-- --------------------------------------------------------

--
-- 資料表結構 `egame_exam_record`
--

CREATE TABLE `egame_exam_record` (
  `sn` int UNSIGNED NOT NULL COMMENT '資料流水號',
  `user_id` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '使用者id',
  `egame_id` int UNSIGNED NOT NULL COMMENT 'egame 編號',
  `blocks` int UNSIGNED NOT NULL COMMENT '運算思維積木使用數量',
  `rating` int UNSIGNED NOT NULL COMMENT '星星數量(1-3顆星)',
  `time_spent` int UNSIGNED NOT NULL COMMENT '花費時間(毫秒)',
  `time_created` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '建立時間	'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `egame_info`
--

CREATE TABLE `egame_info` (
  `egame_id` int UNSIGNED NOT NULL COMMENT 'egame 編號',
  `island_id` varchar(10) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '島嶼名稱',
  `chapter` int UNSIGNED NOT NULL COMMENT '章節編號',
  `stage` int UNSIGNED NOT NULL COMMENT '關卡編號',
  `unit_name` varchar(20) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '單元名稱',
  `map_sn` int DEFAULT NULL,
  `indicate_id` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '對應節點\r\n',
  `link` text
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='Egame島嶼清冊';

-- --------------------------------------------------------

--
-- 資料表結構 `exam_country_student_tmp`
--

CREATE TABLE `exam_country_student_tmp` (
  `stud_sn` int UNSIGNED NOT NULL COMMENT '流水號',
  `user_id` varchar(60) NOT NULL COMMENT '學生帳號',
  `uname` varchar(40) NOT NULL COMMENT '學生名字',
  `organization_id` varchar(10) NOT NULL COMMENT '學校代碼',
  `grade` tinyint UNSIGNED NOT NULL COMMENT '年級',
  `class` tinyint UNSIGNED NOT NULL COMMENT '班級',
  `seme_num` tinyint UNSIGNED NOT NULL COMMENT '座號',
  `seme` varchar(6) NOT NULL COMMENT '學期',
  `pass` varchar(40) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='縣市能力測驗學生暫存表';

-- --------------------------------------------------------

--
-- 資料表結構 `exam_edu_parameter`
--

CREATE TABLE `exam_edu_parameter` (
  `sn` int NOT NULL COMMENT '流水號',
  `edu_parameter` varchar(50) NOT NULL COMMENT '教育目標參數',
  `explanation` mediumtext NOT NULL COMMENT '參數內容說明',
  `show` int NOT NULL COMMENT '是否顯示'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='診斷報告之教育目標參數說明';

-- --------------------------------------------------------

--
-- 資料表結構 `exam_item_answer_option_type`
--

CREATE TABLE `exam_item_answer_option_type` (
  `answer_option_type` tinyint UNSIGNED NOT NULL COMMENT '答案選項類型',
  `answer_option_type_name` varchar(10) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '類型名稱',
  `answer_option_type_code` varchar(10) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '類型代號'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='測驗題答案選項類型';

-- --------------------------------------------------------

--
-- 資料表結構 `exam_item_type`
--

CREATE TABLE `exam_item_type` (
  `item_type` tinyint UNSIGNED NOT NULL COMMENT '題型',
  `item_type_name` varchar(10) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '題型名稱',
  `item_type_name_eng` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '題型名稱(英)',
  `item_type_code` varchar(5) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '題型代號'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='測驗題型';

-- --------------------------------------------------------

--
-- 資料表結構 `exam_onscreen_keyboard`
--

CREATE TABLE `exam_onscreen_keyboard` (
  `keyboard_sn` smallint UNSIGNED NOT NULL COMMENT '小鍵盤流水號',
  `map_sn` int NOT NULL COMMENT '星空圖流水號',
  `keyboard_name` varchar(20) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '小鍵盤名稱',
  `keyboard_keys` json NOT NULL COMMENT '小鍵盤按鍵'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `exam_paper`
--

CREATE TABLE `exam_paper` (
  `sn` bigint NOT NULL COMMENT '試卷流水號',
  `cs_id` varchar(12) NOT NULL COMMENT '對應之單元概念結構代號',
  `paper_vol` int NOT NULL COMMENT '試卷別',
  `item_nums` int NOT NULL COMMENT '總題數',
  `ready` tinyint(1) NOT NULL DEFAULT '0'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='試卷列表';

-- --------------------------------------------------------

--
-- 資料表結構 `exam_paper_access`
--

CREATE TABLE `exam_paper_access` (
  `sn` bigint NOT NULL COMMENT '流水號',
  `cs_id` varchar(12) NOT NULL COMMENT '概念編號',
  `paper_vol` int NOT NULL COMMENT '試卷別',
  `exam_paper_id` varchar(15) NOT NULL COMMENT '試卷編號',
  `school_id` varchar(10) NOT NULL COMMENT '學校代號',
  `grade` int NOT NULL,
  `class` int NOT NULL,
  `type_id` int NOT NULL COMMENT '施測出題類型',
  `semester` smallint NOT NULL DEFAULT '1052'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='試卷存取權限';

-- --------------------------------------------------------

--
-- 資料表結構 `exam_record`
--

CREATE TABLE `exam_record` (
  `exam_sn` bigint NOT NULL COMMENT '流水號',
  `user_id` varchar(60) DEFAULT NULL COMMENT '使用者id',
  `cs_id` varchar(15) NOT NULL COMMENT '施測單元',
  `date` datetime NOT NULL COMMENT '施測日期',
  `time` time NOT NULL COMMENT '時:分',
  `start_time` varchar(14) NOT NULL COMMENT '開始時間',
  `stop_time` varchar(14) NOT NULL DEFAULT '' COMMENT '完成時間',
  `during_time` varchar(16) NOT NULL COMMENT '測驗時間(秒)',
  `client_items_idle_time` mediumtext NOT NULL COMMENT '作答時間(毫秒)',
  `firm_id` varchar(10) NOT NULL COMMENT '施測地點id',
  `score` float NOT NULL COMMENT '得分',
  `questions` mediumtext NOT NULL COMMENT '試卷內容題號',
  `org_res` mediumtext NOT NULL COMMENT '原始作答情形',
  `binary_res` mediumtext NOT NULL COMMENT '二元作答情形',
  `exam_title` varchar(20) NOT NULL COMMENT '試卷名稱',
  `paper_vol` int NOT NULL COMMENT '試卷別',
  `bug_remedy_rate` mediumtext NOT NULL,
  `skill_remedy_rate` mediumtext NOT NULL,
  `bNodeStatus` mediumtext NOT NULL,
  `sNodeStatus` mediumtext NOT NULL,
  `degree` int NOT NULL,
  `type_id` int NOT NULL COMMENT '試卷施測類型',
  `seme_year` smallint UNSIGNED NOT NULL COMMENT '學期',
  `reward_mk` char(1) NOT NULL DEFAULT 'N' COMMENT '是否已計算獎勵',
  `ip` varchar(25) DEFAULT NULL COMMENT '來源ip'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='測驗結果';

-- --------------------------------------------------------

--
-- 資料表結構 `exam_record_aassessment_detail`
--

CREATE TABLE `exam_record_aassessment_detail` (
  `sn` int UNSIGNED NOT NULL COMMENT '流水號',
  `rfh_sn` int NOT NULL COMMENT 'remedial_ftp_history.sn',
  `organization_id` varchar(10) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '測驗資料學校代碼',
  `user_id` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '學生帳號',
  `uname` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '姓名',
  `OpenID_sub` varchar(100) NOT NULL COMMENT 'sub',
  `seme` varchar(10) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '施測學年度',
  `grade` int NOT NULL DEFAULT '0' COMMENT '測驗資料年級',
  `class` int NOT NULL DEFAULT '0' COMMENT '測驗資料班級',
  `seat` smallint UNSIGNED NOT NULL COMMENT '座號',
  `correct_num` int NOT NULL COMMENT '答對題數',
  `correct_rate` int NOT NULL COMMENT '答對率',
  `total_num` int NOT NULL COMMENT '總題數',
  `org_res` text CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '原始作答',
  `binary_res` text CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '二元作答情形',
  `import_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '匯入日期',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '更新時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='學習扶助測驗細節';

-- --------------------------------------------------------

--
-- 資料表結構 `exam_record_aassessment_detail_1028`
--

CREATE TABLE `exam_record_aassessment_detail_1028` (
  `sn` int UNSIGNED NOT NULL COMMENT '流水號',
  `rfh_sn` int NOT NULL COMMENT 'remedial_ftp_history.sn',
  `organization_id` varchar(10) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '測驗資料學校代碼',
  `user_id` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '學生帳號',
  `uname` varchar(60) NOT NULL COMMENT '姓名',
  `OpenID_sub` varchar(100) NOT NULL COMMENT 'sub',
  `seme` varchar(10) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '施測學年度',
  `grade` int NOT NULL DEFAULT '0' COMMENT '測驗資料年級',
  `class` int NOT NULL DEFAULT '0' COMMENT '測驗資料班級',
  `seat` smallint UNSIGNED NOT NULL COMMENT '座號',
  `correct_num` int NOT NULL COMMENT '答對題數',
  `correct_rate` int NOT NULL COMMENT '答對率',
  `total_num` int NOT NULL COMMENT '總題數',
  `org_res` text CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '原始作答',
  `binary_res` text CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '二元作答情形',
  `import_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '匯入日期',
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='學習扶助測驗細節';

-- --------------------------------------------------------

--
-- 資料表結構 `exam_record_bat_detail`
--

CREATE TABLE `exam_record_bat_detail` (
  `sn` int UNSIGNED NOT NULL COMMENT '流水號',
  `user_id` varchar(60) NOT NULL COMMENT '學生帳號',
  `city_code` int NOT NULL COMMENT '城市代號 city.city_code',
  `exam_range` varchar(100) NOT NULL COMMENT '測驗時間',
  `subject_id` smallint UNSIGNED NOT NULL COMMENT '科目',
  `organization_id` varchar(10) NOT NULL COMMENT '測驗資料學校代碼',
  `grade` int NOT NULL DEFAULT '0' COMMENT '測驗資料年級',
  `class` int NOT NULL DEFAULT '0' COMMENT '測驗資料班級',
  `seme_num` tinyint UNSIGNED DEFAULT NULL COMMENT '測驗資料座號',
  `uname` varchar(60) NOT NULL COMMENT '測驗資料姓名',
  `import_time` datetime NOT NULL COMMENT '匯入日期',
  `update_time` datetime NOT NULL ON UPDATE CURRENT_TIMESTAMP COMMENT '更新時間',
  `scores` double(14,14) UNSIGNED NOT NULL COMMENT '答對率(滿分1)',
  `total_num` int NOT NULL COMMENT '總題數',
  `rank` int NOT NULL COMMENT '縣市排名',
  `PR` int NOT NULL COMMENT '縣市PR值',
  `dimension` mediumtext NOT NULL COMMENT '各向度序列化資料(ind_name:向度中文, correct_num: 答對題數, correct_percent: 該向度答對率)',
  `org_res` mediumtext NOT NULL COMMENT '原始做答紀錄',
  `binary_res` mediumtext NOT NULL COMMENT '二元作答情形'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='學力測驗細節';

-- --------------------------------------------------------

--
-- 資料表結構 `exam_record_dynamic`
--

CREATE TABLE `exam_record_dynamic` (
  `exam_sn` bigint NOT NULL COMMENT '流水號',
  `user_id` varchar(60) DEFAULT NULL COMMENT '使用者id',
  `cs_id` varchar(15) NOT NULL COMMENT '施測單元',
  `date` datetime NOT NULL COMMENT '施測日期',
  `start_time` varchar(14) NOT NULL COMMENT '開始時間',
  `stop_time` varchar(14) NOT NULL DEFAULT '' COMMENT '完成時間',
  `during_time` varchar(16) NOT NULL COMMENT '測驗時間(秒)',
  `client_items_idle_time` varchar(200) NOT NULL COMMENT 'client端每題作答時間(毫秒)',
  `score` tinyint UNSIGNED NOT NULL COMMENT '得分',
  `score_rate` tinyint UNSIGNED DEFAULT NULL COMMENT '正確率',
  `questions` varchar(100) NOT NULL COMMENT '作答題號順序',
  `org_res` varchar(100) NOT NULL COMMENT '原始作答情形',
  `binary_res` varchar(100) NOT NULL COMMENT '二元作答情形',
  `exam_title` varchar(20) NOT NULL COMMENT '試卷名稱',
  `paper_vol` tinyint UNSIGNED NOT NULL COMMENT '試卷別',
  `indicator` varchar(15) NOT NULL COMMENT '能力指標子技能代號',
  `bNodeStatus` mediumtext NOT NULL,
  `sNodeStatus` mediumtext NOT NULL,
  `seme_year` smallint UNSIGNED NOT NULL COMMENT '學期',
  `mission_sn` int DEFAULT NULL,
  `ip` varchar(25) DEFAULT NULL COMMENT '來源ip'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='動態評量結果';

-- --------------------------------------------------------

--
-- 資料表結構 `exam_record_indicate`
--

CREATE TABLE `exam_record_indicate` (
  `exam_sn` bigint NOT NULL,
  `user_id` varchar(60) DEFAULT NULL,
  `cs_id` varchar(15) NOT NULL,
  `date` datetime NOT NULL,
  `time` time NOT NULL COMMENT '時:分',
  `start_time` varchar(14) NOT NULL,
  `stop_time` varchar(14) NOT NULL,
  `during_time` varchar(16) NOT NULL COMMENT '毫秒',
  `score` tinyint UNSIGNED DEFAULT NULL COMMENT '正確率',
  `snode_rate` tinyint UNSIGNED DEFAULT NULL COMMENT '小節點正確率',
  `client_items_idle_time` mediumtext COMMENT '作答時間(毫秒)',
  `questions` mediumtext NOT NULL,
  `stuAns` mediumtext NOT NULL COMMENT '原始作答情形(根據作答順序紀錄)',
  `binary_res` mediumtext NOT NULL COMMENT '二元作答情形(根據作答順序紀錄)',
  `skill_remedy_rate_s` mediumtext NOT NULL COMMENT '小節點(小節點(-1: 還沒做, 0: 錯, 1: 對, 2:預測對, 3: 預測對但做錯, 4:預測對也做對))',
  `skill_remedy_rate_b` mediumtext NOT NULL COMMENT '大節點內精熟的小節點數',
  `result_sn` int DEFAULT NULL COMMENT 'mission_sn',
  `seme_year` smallint UNSIGNED NOT NULL COMMENT '學年度',
  `reward_mk` char(1) NOT NULL DEFAULT 'N' COMMENT '是否已計算獎勵'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='縱貫測驗結果';

-- --------------------------------------------------------

--
-- 資料表結構 `exam_record_indicate_tmp`
--

CREATE TABLE `exam_record_indicate_tmp` (
  `exam_sn` bigint NOT NULL COMMENT '流水號',
  `user_id` varchar(60) DEFAULT NULL COMMENT '使用者id',
  `cs_id` varchar(15) NOT NULL COMMENT '施測單元',
  `date` datetime NOT NULL COMMENT '施測日期',
  `start_time` varchar(14) NOT NULL COMMENT '開始時間',
  `stop_time` varchar(14) NOT NULL DEFAULT '' COMMENT '完成時間',
  `during_time` varchar(16) NOT NULL COMMENT '測驗時間',
  `client_items_idle_time` mediumtext COMMENT '作答時間(毫秒)',
  `questions` mediumtext NOT NULL COMMENT '試卷內容題號',
  `stuAns` mediumtext NOT NULL COMMENT '原始作答情形',
  `binary_res` mediumtext NOT NULL COMMENT '二元作答情形',
  `skill_remedy_rate_s` mediumtext NOT NULL,
  `skill_remedy_rate_b` mediumtext NOT NULL,
  `itemSort_b` mediumtext NOT NULL COMMENT '大節點的出題順序',
  `itemSort_s` mediumtext NOT NULL COMMENT '小節點的出題順序',
  `itemSession` mediumtext NOT NULL COMMENT '試題順序',
  `type_id` int DEFAULT NULL COMMENT '試卷施測類型',
  `seme_year` smallint UNSIGNED DEFAULT NULL COMMENT '學期',
  `ip` varchar(25) DEFAULT NULL COMMENT '來源ip',
  `result_sn` int DEFAULT NULL COMMENT '任務id'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='測驗結果';

-- --------------------------------------------------------

--
-- 資料表結構 `exam_record_indicator`
--

CREATE TABLE `exam_record_indicator` (
  `exam_sn` bigint NOT NULL,
  `user_id` varchar(60) NOT NULL,
  `paper_sn` bigint UNSIGNED DEFAULT NULL,
  `date` datetime NOT NULL,
  `semester` int UNSIGNED DEFAULT NULL COMMENT '學年度學期',
  `start_time` varchar(14) NOT NULL,
  `stop_time` varchar(14) NOT NULL,
  `during_time` varchar(16) NOT NULL,
  `score` tinyint UNSIGNED DEFAULT NULL COMMENT '正確率',
  `snode_rate` tinyint UNSIGNED DEFAULT NULL COMMENT '小節點正確率',
  `client_items_idle_time` mediumtext COMMENT '作答時間(毫秒)',
  `questions` mediumtext NOT NULL,
  `stuAns` mediumtext NOT NULL COMMENT '根據作答順序紀錄',
  `binary_res` mediumtext NOT NULL COMMENT '根據作答順序紀錄',
  `skill_remedy_rate_s` mediumtext NOT NULL COMMENT '小節點(0:待補救，1:精熟)',
  `skill_remedy_rate_b` mediumtext NOT NULL COMMENT '大節點內精熟的小節點數',
  `map_sn` int NOT NULL,
  `mission_sn` int DEFAULT NULL,
  `reward_mk` char(1) NOT NULL DEFAULT 'N' COMMENT '是否已計算獎勵'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='單元測驗結果';

-- --------------------------------------------------------

--
-- 資料表結構 `exam_record_indicator_bat`
--

CREATE TABLE `exam_record_indicator_bat` (
  `exam_sn` bigint NOT NULL,
  `user_id` varchar(60) NOT NULL,
  `paper_sn` bigint UNSIGNED DEFAULT NULL,
  `date` datetime NOT NULL,
  `semester` int UNSIGNED DEFAULT NULL COMMENT '學年度學期',
  `start_time` varchar(14) NOT NULL,
  `stop_time` varchar(14) NOT NULL,
  `during_time` varchar(16) NOT NULL,
  `score` tinyint UNSIGNED DEFAULT NULL COMMENT '正確率',
  `snode_rate` int UNSIGNED DEFAULT NULL COMMENT '小節點正確率',
  `client_items_idle_time` mediumtext COMMENT '作答時間(毫秒)',
  `questions` mediumtext NOT NULL,
  `stuAns` mediumtext NOT NULL COMMENT '根據作答順序紀錄',
  `binary_res` mediumtext NOT NULL COMMENT '根據作答順序紀錄',
  `skill_remedy_rate_s` mediumtext NOT NULL COMMENT '小節點(0:待補救，1:精熟)',
  `skill_remedy_rate_b` mediumtext NOT NULL COMMENT '大節點內精熟的小節點數',
  `map_sn` int NOT NULL,
  `mission_sn` int DEFAULT NULL,
  `bat_data` mediumtext NOT NULL COMMENT 'for 補救教學資料(應為沒在用此欄位)',
  `reward_mk` char(1) NOT NULL DEFAULT 'N' COMMENT '是否已計算獎勵'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `exam_record_indicator_bat_tmp`
--

CREATE TABLE `exam_record_indicator_bat_tmp` (
  `exam_sn` bigint NOT NULL COMMENT '流水號',
  `user_id` varchar(60) NOT NULL COMMENT '使用者id',
  `paper_sn` bigint UNSIGNED NOT NULL COMMENT '施測單元',
  `date` datetime NOT NULL COMMENT '暫存日期',
  `session_data` longtext NOT NULL COMMENT '暫存資料',
  `mission_sn` int NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='測驗結果';

-- --------------------------------------------------------

--
-- 資料表結構 `exam_record_indicator_cap`
--

CREATE TABLE `exam_record_indicator_cap` (
  `exam_sn` bigint NOT NULL,
  `user_id` varchar(60) NOT NULL,
  `paper_sn` bigint UNSIGNED NOT NULL,
  `date` datetime NOT NULL,
  `semester` int UNSIGNED DEFAULT NULL COMMENT '學年度學期',
  `start_time` varchar(14) NOT NULL,
  `stop_time` varchar(14) NOT NULL,
  `during_time` varchar(16) NOT NULL,
  `score` tinyint UNSIGNED DEFAULT NULL COMMENT '正確率',
  `snode_rate` int UNSIGNED DEFAULT NULL COMMENT '小節點正確率',
  `client_items_idle_time` mediumtext COMMENT '作答時間(毫秒)',
  `questions` mediumtext NOT NULL,
  `stuAns` mediumtext NOT NULL COMMENT '根據作答順序紀錄',
  `binary_res` mediumtext NOT NULL COMMENT '根據作答順序紀錄',
  `skill_remedy_rate_s` mediumtext NOT NULL COMMENT '小節點(0:待補救，1:精熟)',
  `skill_remedy_rate_b` mediumtext NOT NULL COMMENT '大節點內精熟的小節點數',
  `map_sn` int NOT NULL,
  `mission_sn` int NOT NULL,
  `cap_data` mediumtext NOT NULL COMMENT 'for 補救教學資料',
  `reward_mk` char(1) NOT NULL DEFAULT 'N' COMMENT '是否已計算獎勵'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `exam_record_indicator_cap_tmp`
--

CREATE TABLE `exam_record_indicator_cap_tmp` (
  `exam_sn` bigint NOT NULL COMMENT '流水號',
  `user_id` varchar(60) NOT NULL COMMENT '使用者id',
  `paper_sn` bigint UNSIGNED NOT NULL COMMENT '施測單元',
  `date` datetime NOT NULL COMMENT '暫存日期',
  `session_data` longtext NOT NULL COMMENT '暫存資料',
  `mission_sn` int NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='測驗結果';

-- --------------------------------------------------------

--
-- 資料表結構 `exam_record_indicator_tmp`
--

CREATE TABLE `exam_record_indicator_tmp` (
  `exam_sn` bigint NOT NULL COMMENT '流水號',
  `user_id` varchar(60) NOT NULL COMMENT '使用者id',
  `paper_sn` bigint UNSIGNED NOT NULL COMMENT '施測單元',
  `date` datetime NOT NULL COMMENT '暫存日期',
  `session_data` longtext NOT NULL COMMENT '暫存資料',
  `mission_sn` int NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='測驗結果';

-- --------------------------------------------------------

--
-- 資料表結構 `exam_record_interactive`
--

CREATE TABLE `exam_record_interactive` (
  `exam_sn` bigint NOT NULL,
  `mission_sn` int DEFAULT NULL COMMENT '任務sn,若無則NULL	',
  `subject_id` smallint UNSIGNED NOT NULL,
  `map_sn` int NOT NULL,
  `user_id` varchar(60) NOT NULL,
  `start_time` datetime NOT NULL,
  `client_items_idle_time` mediumtext COMMENT '作答時間(毫秒)',
  `literacy_group_sn` mediumtext COMMENT '題組題的group_sn',
  `questions` mediumtext NOT NULL COMMENT '試題代號',
  `stu_qusetion` mediumtext NOT NULL COMMENT '根據作答順序紀錄',
  `stu_answer` mediumtext NOT NULL COMMENT '根據作答順序紀錄',
  `binary_res` mediumtext NOT NULL COMMENT '二元作答反應',
  `correct_rate` varchar(3) NOT NULL COMMENT '答對率，最高100',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `exam_record_its`
--

CREATE TABLE `exam_record_its` (
  `its_sn` int NOT NULL,
  `user_id` varchar(60) NOT NULL COMMENT '使用者id',
  `indicate_id` varchar(15) NOT NULL COMMENT '能力指標代號',
  `date` varchar(30) NOT NULL COMMENT '施測日期',
  `start_time` varchar(14) NOT NULL COMMENT '開始時間',
  `stop_time` varchar(14) NOT NULL DEFAULT '' COMMENT '完成時間',
  `during_time` varchar(16) NOT NULL COMMENT '測驗時間',
  `item_num` mediumtext NOT NULL COMMENT '元件編號',
  `org_res` longtext COMMENT '原始作答情形',
  `pattern_type` varchar(200) DEFAULT NULL COMMENT '作答類型代號',
  `bug_pattern` varchar(200) DEFAULT NULL COMMENT '錯誤類型樣態'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='互動教學元件結果';

-- --------------------------------------------------------

--
-- 資料表結構 `exam_record_literacy`
--

CREATE TABLE `exam_record_literacy` (
  `exam_sn` bigint NOT NULL,
  `mission_sn` int DEFAULT NULL COMMENT '任務sn,若無則NULL	',
  `subject_id` smallint UNSIGNED NOT NULL,
  `map_sn` int NOT NULL,
  `user_id` varchar(60) NOT NULL,
  `start_time` datetime NOT NULL,
  `client_items_idle_time` mediumtext COMMENT '作答時間(毫秒)',
  `literacy_group_sn` mediumtext COMMENT '題組題的group_sn',
  `questions` mediumtext NOT NULL COMMENT '試題代號',
  `stu_qusetion` mediumtext NOT NULL COMMENT '根據作答順序紀錄',
  `stu_answer` mediumtext NOT NULL COMMENT '根據作答順序紀錄',
  `binary_res` mediumtext NOT NULL COMMENT '二元作答反應',
  `correct_rate` tinyint UNSIGNED NOT NULL COMMENT '答對率，最高100',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `exam_record_literacy_interactive`
--

CREATE TABLE `exam_record_literacy_interactive` (
  `exam_sn` bigint NOT NULL,
  `mission_sn` int DEFAULT NULL COMMENT '任務sn,若無則NULL',
  `subject_id` smallint UNSIGNED NOT NULL,
  `map_sn` int NOT NULL,
  `item_li_sn` int UNSIGNED DEFAULT NULL COMMENT '互動式素養題編號',
  `user_id` varchar(50) NOT NULL,
  `start_time` datetime NOT NULL,
  `questions_type` varchar(255) NOT NULL COMMENT '  1:是非題   2:選擇題 1234   3:文字回答, 只要有文字題就分類為文字   4:複選 123456789   5:順序題(拖拉題) (拖拉順序:)   6:回傳base64圖檔',
  `client_items_idle_time` varchar(255) DEFAULT NULL COMMENT '作答時間(毫秒)',
  `stuAns` longtext NOT NULL COMMENT '根據作答順序紀錄',
  `binary_res` mediumtext NOT NULL COMMENT '根據作答順序紀錄',
  `ai_feedback` json DEFAULT NULL COMMENT 'AI評分結果',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `exam_record_literacy_interactive_log`
--

CREATE TABLE `exam_record_literacy_interactive_log` (
  `log_sn` bigint NOT NULL,
  `exam_sn` bigint NOT NULL,
  `item_li_sn` int UNSIGNED DEFAULT NULL COMMENT '互動式素養題編號',
  `action_log` json NOT NULL COMMENT '行為紀錄的JSON',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `exam_record_priori`
--

CREATE TABLE `exam_record_priori` (
  `exam_sn` bigint NOT NULL COMMENT '流水號',
  `exam_user` varchar(50) DEFAULT '',
  `user_id` varchar(60) NOT NULL COMMENT '使用者id',
  `cp_id` varchar(20) NOT NULL COMMENT '學習扶助唯一碼',
  `date` datetime NOT NULL COMMENT '施測日期',
  `grade` int DEFAULT '0' COMMENT '年級',
  `score` float NOT NULL COMMENT '得分',
  `exam_title` varchar(20) NOT NULL COMMENT '試卷名稱',
  `priori_remedy_rate` mediumtext NOT NULL,
  `skill_remedy_rate` mediumtext NOT NULL,
  `bNodeStatus` mediumtext NOT NULL,
  `sNodeStatus` mediumtext NOT NULL,
  `seme_year` smallint UNSIGNED NOT NULL COMMENT '學期',
  `reward_mk` char(1) NOT NULL DEFAULT 'N' COMMENT '是否已計算獎勵',
  `ip` varchar(25) DEFAULT NULL COMMENT '來源ip'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='測驗結果';

-- --------------------------------------------------------

--
-- 資料表結構 `exam_record_priori_detail`
--

CREATE TABLE `exam_record_priori_detail` (
  `sn` int UNSIGNED NOT NULL COMMENT '流水號',
  `prior_name` varchar(100) NOT NULL COMMENT '補救教學對應使用',
  `user_id` varchar(60) DEFAULT NULL COMMENT '學生帳號',
  `organization_id` varchar(10) NOT NULL COMMENT '測驗資料學校代碼',
  `grade` int NOT NULL DEFAULT '0' COMMENT '測驗資料年級',
  `class` int NOT NULL DEFAULT '0' COMMENT '測驗資料班級',
  `exam_range` varchar(100) NOT NULL COMMENT '施測日期',
  `subject_id` smallint UNSIGNED NOT NULL COMMENT '科目',
  `exam_grade` int NOT NULL COMMENT '測驗年級',
  `import_time` datetime NOT NULL COMMENT '匯入日期',
  `update_time` datetime NOT NULL ON UPDATE CURRENT_TIMESTAMP COMMENT '更新時間',
  `scores` int NOT NULL COMMENT '分數(滿分100)',
  `correct_num` int NOT NULL COMMENT '答對題數',
  `total_num` int NOT NULL COMMENT '總題數',
  `pass` int NOT NULL DEFAULT '0' COMMENT '是否通過(1:通過; 0:未通過)',
  `total_time` varchar(16) NOT NULL COMMENT '測驗時間(秒)',
  `exam_time` mediumtext NOT NULL COMMENT '每題反應時間',
  `org_res` text NOT NULL COMMENT '原始作答',
  `binary_res` text NOT NULL COMMENT '二元作答情形',
  `indicator_priori` mediumtext NOT NULL COMMENT '基本學習內容代號',
  `indicator_item` mediumtext NOT NULL COMMENT '能力指標'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='學習扶助測驗細節';

-- --------------------------------------------------------

--
-- 資料表結構 `exam_record_priori_error`
--

CREATE TABLE `exam_record_priori_error` (
  `sn` bigint NOT NULL COMMENT '流水號',
  `cp_info_sn` int DEFAULT NULL COMMENT '學扶批次/學力匯入 concept_priori_info.sn',
  `exam_sn` bigint DEFAULT NULL COMMENT 'exam_record_priori.exam_sn',
  `user_id` varchar(60) DEFAULT NULL COMMENT 'for學力',
  `exam_user` varchar(100) NOT NULL COMMENT 'user_info.priori_name',
  `cp_id` varchar(20) DEFAULT NULL COMMENT '手動匯入學習扶助cp_id',
  `cp_grade` int DEFAULT '0' COMMENT '手動匯入 concept_priori.grade',
  `organization_id` varchar(7) NOT NULL COMMENT 'for 學力 999999 為轉學',
  `grade` int NOT NULL COMMENT 'for 學力',
  `class` int NOT NULL COMMENT 'for 學力',
  `seme_num` tinyint NOT NULL COMMENT 'for 學力',
  `date` datetime NOT NULL,
  `score` float NOT NULL COMMENT '得分',
  `exam_title` varchar(30) NOT NULL,
  `priori_remedy_rate` mediumtext NOT NULL,
  `skill_remedy_rate` mediumtext NOT NULL,
  `bNodeStatus` mediumtext NOT NULL,
  `sNodeStatus` mediumtext NOT NULL,
  `seme_year` smallint NOT NULL,
  `reward_mk` char(1) NOT NULL DEFAULT 'N',
  `ip` varchar(25) DEFAULT NULL,
  `is_connect` smallint NOT NULL COMMENT '是否連結至學生帳號'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `exam_record_priori_his`
--

CREATE TABLE `exam_record_priori_his` (
  `sn` bigint NOT NULL COMMENT '流水號',
  `exam_user` varchar(50) DEFAULT '',
  `user_id` varchar(60) DEFAULT NULL COMMENT '使用者id',
  `cp_id` varchar(15) NOT NULL COMMENT '施測單元 concept_priori.cp_id',
  `mission_sn` int DEFAULT NULL COMMENT '任務編號 mission_info.mission_sn',
  `exam_type` varchar(20) NOT NULL COMMENT '測驗類型 concept_priori.exam_type',
  `exam_sn` bigint DEFAULT NULL COMMENT '測驗紀錄表的 sn',
  `date` datetime NOT NULL COMMENT '施測日期',
  `score` float NOT NULL COMMENT '得分',
  `exam_title` varchar(20) NOT NULL COMMENT '試卷名稱',
  `priori_remedy_rate` mediumtext NOT NULL COMMENT '施測紀錄',
  `skill_remedy_rate` mediumtext NOT NULL,
  `bNodeStatus` mediumtext NOT NULL,
  `sNodeStatus` mediumtext NOT NULL,
  `seme_year` smallint UNSIGNED NOT NULL COMMENT '學期',
  `reward_mk` char(1) NOT NULL DEFAULT 'N' COMMENT '是否已計算獎勵',
  `ip` varchar(25) DEFAULT NULL COMMENT '來源ip'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='測驗結果';

-- --------------------------------------------------------

--
-- 資料表結構 `exam_record_tmp`
--

CREATE TABLE `exam_record_tmp` (
  `exam_sn` bigint NOT NULL COMMENT '流水號',
  `user_id` varchar(60) DEFAULT NULL COMMENT '使用者id',
  `cs_id` varchar(15) NOT NULL COMMENT '施測單元',
  `date` varchar(30) NOT NULL COMMENT '施測日期',
  `start_time` varchar(14) NOT NULL COMMENT '開始時間',
  `stop_time` varchar(14) NOT NULL DEFAULT '' COMMENT '完成時間',
  `during_time` varchar(16) NOT NULL COMMENT '測驗時間',
  `client_items_idle_time` mediumtext NOT NULL COMMENT '作答時間(毫秒)',
  `firm_id` varchar(10) NOT NULL COMMENT '施測地點id',
  `score` float NOT NULL COMMENT '得分',
  `questions` mediumtext NOT NULL COMMENT '試卷內容題號',
  `org_res` mediumtext NOT NULL COMMENT '原始作答情形',
  `binary_res` mediumtext NOT NULL COMMENT '二元作答情形',
  `exam_title` varchar(20) NOT NULL COMMENT '試卷名稱',
  `paper_vol` int NOT NULL COMMENT '試卷別',
  `bug_remedy_rate` mediumtext NOT NULL,
  `skill_remedy_rate` mediumtext NOT NULL,
  `degree` int NOT NULL,
  `type_id` int NOT NULL COMMENT '試卷施測類型',
  `seme_year` smallint UNSIGNED NOT NULL COMMENT '學期',
  `ip` varchar(25) DEFAULT NULL COMMENT '來源ip'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='測驗結果';

-- --------------------------------------------------------

--
-- 資料表結構 `exam_template`
--

CREATE TABLE `exam_template` (
  `template_id` tinyint UNSIGNED NOT NULL COMMENT '測驗之樣版檔代號',
  `file_name` varchar(20) NOT NULL COMMENT '樣版檔檔名',
  `name` varchar(50) NOT NULL COMMENT '樣版名稱'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='測驗樣版檔';

-- --------------------------------------------------------

--
-- 資料表結構 `exam_type`
--

CREATE TABLE `exam_type` (
  `exam_type` tinyint UNSIGNED NOT NULL,
  `exam_type_name` varchar(10) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '類型名稱',
  `exam_type_name_eng` varchar(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '類型名稱(英)',
  `exam_type_code` varchar(5) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '類型代號',
  `series` varchar(20) NOT NULL COMMENT '測驗系列',
  `mission_type` tinyint UNSIGNED DEFAULT NULL COMMENT '任務類型'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='測驗類型';

-- --------------------------------------------------------

--
-- 資料表結構 `export_school`
--

CREATE TABLE `export_school` (
  `sn` int NOT NULL COMMENT '流水號',
  `org_id` varchar(60) DEFAULT NULL COMMENT '原 user_id',
  `org_school` varchar(10) DEFAULT NULL COMMENT '原學校代碼',
  `export_seme` varchar(6) NOT NULL COMMENT '轉出學年度',
  `export_date` varchar(10) NOT NULL COMMENT '轉出日期',
  `exporter` varchar(60) DEFAULT NULL COMMENT '轉出人',
  `user_level` int NOT NULL COMMENT 'user 等級',
  `new_id` varchar(60) DEFAULT NULL COMMENT '新 user_id',
  `new_school` varchar(10) DEFAULT NULL COMMENT '新學校代碼',
  `import_seme` varchar(6) DEFAULT NULL COMMENT '轉入學年度',
  `import_date` varchar(10) DEFAULT NULL COMMENT '轉入日期'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='轉出入登記';

-- --------------------------------------------------------

--
-- 資料表結構 `external_res_type`
--

CREATE TABLE `external_res_type` (
  `type_sn` tinyint NOT NULL,
  `type_name` varchar(10) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `ftp_upload_log`
--

CREATE TABLE `ftp_upload_log` (
  `id` int NOT NULL,
  `user_id` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL,
  `action` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL,
  `filename` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL,
  `status` varchar(20) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL,
  `message` text CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci,
  `created_at` timestamp NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `group_check_result`
--

CREATE TABLE `group_check_result` (
  `sn` int NOT NULL,
  `user_id` varchar(60) NOT NULL,
  `check_list_sn` int NOT NULL,
  `org_ans` text NOT NULL COMMENT '原始作答',
  `ans` text NOT NULL COMMENT '對應作答分數',
  `total_score` int NOT NULL COMMENT '總計',
  `suggest` text NOT NULL COMMENT '建議',
  `mission_sn` int NOT NULL,
  `date` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='檢核表結果';

-- --------------------------------------------------------

--
-- 資料表結構 `group_member_score_result`
--

CREATE TABLE `group_member_score_result` (
  `sn` int NOT NULL,
  `user_id` varchar(60) NOT NULL,
  `check_list_sn` int NOT NULL,
  `score_user_id` varchar(60) NOT NULL,
  `score` text NOT NULL,
  `total_score` int NOT NULL,
  `mission_sn` int NOT NULL,
  `date` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `group_role`
--

CREATE TABLE `group_role` (
  `group_role_sn` int NOT NULL COMMENT '小組角色流水號',
  `teacher_id` varchar(60) NOT NULL COMMENT '老師的ID',
  `role_name` varchar(20) NOT NULL COMMENT '角色名稱',
  `role_info` mediumtext NOT NULL COMMENT '角色資訊/任務分配'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='小組角色資料表';

-- --------------------------------------------------------

--
-- 資料表結構 `group_score_ingroup_result`
--

CREATE TABLE `group_score_ingroup_result` (
  `sn` int NOT NULL,
  `user_id` varchar(60) NOT NULL,
  `check_list_sn` int NOT NULL,
  `score_user_id` varchar(60) NOT NULL,
  `group_id` int DEFAULT NULL,
  `score` text NOT NULL,
  `total_score` int NOT NULL,
  `mission_sn` int NOT NULL,
  `date` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `group_score_result`
--

CREATE TABLE `group_score_result` (
  `sn` int NOT NULL,
  `user_id` varchar(60) NOT NULL,
  `check_list_sn` int NOT NULL COMMENT '評分表的SN',
  `user_group_id` int NOT NULL COMMENT '評分(小組長)的小組',
  `group_id` int NOT NULL COMMENT '被評分的小組',
  `score` text NOT NULL COMMENT '原始作答',
  `total_score` int NOT NULL COMMENT '總計',
  `mission_sn` int NOT NULL,
  `date` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='評分表結果';

-- --------------------------------------------------------

--
-- 資料表結構 `gtm_logs`
--

CREATE TABLE `gtm_logs` (
  `id` bigint NOT NULL COMMENT 'primary key',
  `user_id` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL COMMENT '使用者 ID',
  `session_id` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci DEFAULT NULL COMMENT '會話 ID',
  `event` varchar(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL COMMENT 'GTM 事件名稱',
  `url` text CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL COMMENT '頁面 URL',
  `action` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL COMMENT '動作描述',
  `timestamp` bigint NOT NULL COMMENT '時間戳記 (毫秒)',
  `additional_data` json DEFAULT NULL COMMENT '額外的 xAPI 或其他資料',
  `created_at` timestamp NULL DEFAULT CURRENT_TIMESTAMP COMMENT '建立時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='GTM 日誌記錄表';

-- --------------------------------------------------------

--
-- 資料表結構 `high_level_comment`
--

CREATE TABLE `high_level_comment` (
  `comment_sn` int NOT NULL,
  `ans_id` int NOT NULL,
  `comment_user` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '做出評論的個人',
  `comment_group` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '做出評論的小組',
  `target` int NOT NULL COMMENT '1:作法,2:想法',
  `comment` text NOT NULL COMMENT '評論內容',
  `type` int NOT NULL COMMENT '1:同意,2:不同意',
  `reply_sn` int DEFAULT NULL COMMENT '回覆對象',
  `datetime` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `high_level_group_ans_list`
--

CREATE TABLE `high_level_group_ans_list` (
  `ans_id` int NOT NULL,
  `mission_sn` int NOT NULL,
  `question_sn` int NOT NULL,
  `content` json NOT NULL,
  `group_id` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL,
  `sub_question` varchar(10) NOT NULL COMMENT '對應的小題',
  `according` int NOT NULL COMMENT '根據哪一個個人答案'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `high_level_joint_discovery`
--

CREATE TABLE `high_level_joint_discovery` (
  `discovery_sn` int NOT NULL,
  `mission_sn` int NOT NULL,
  `question_sn` int NOT NULL,
  `group_id` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL,
  `user_id` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL,
  `group_agree` json DEFAULT NULL COMMENT '小組是否同意',
  `content` text NOT NULL COMMENT '共同發現內容'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `high_level_log`
--

CREATE TABLE `high_level_log` (
  `sn` int NOT NULL,
  `user_id` varchar(60) NOT NULL,
  `mission_sn` int NOT NULL COMMENT '任務編號',
  `content` text NOT NULL COMMENT '內容',
  `node` varchar(10) NOT NULL COMMENT '單元+題目',
  `start_time` datetime DEFAULT NULL COMMENT '開始時間',
  `time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '時間',
  `action` varchar(60) DEFAULT NULL COMMENT '行為',
  `remark` varchar(60) NOT NULL COMMENT '備註'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `high_level_personal_ans_list`
--

CREATE TABLE `high_level_personal_ans_list` (
  `ans_id` int NOT NULL,
  `mission_sn` int NOT NULL,
  `question_sn` int NOT NULL,
  `content` json DEFAULT NULL,
  `user_id` varchar(60) NOT NULL,
  `sub_question` varchar(10) NOT NULL COMMENT '對應的小題'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `high_level_question`
--

CREATE TABLE `high_level_question` (
  `question_sn` int NOT NULL COMMENT '題目sn',
  `group_id` int NOT NULL COMMENT '題組id',
  `file` text CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '對應檔案'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `high_level_question_group`
--

CREATE TABLE `high_level_question_group` (
  `type_id` int NOT NULL COMMENT '題型id',
  `group_id` int NOT NULL COMMENT '題組id',
  `name` varchar(50) NOT NULL COMMENT '名稱',
  `useable` tinyint(1) DEFAULT '0' COMMENT '是否顯示'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `high_level_type`
--

CREATE TABLE `high_level_type` (
  `type_id` int NOT NULL COMMENT '題型',
  `code` varchar(10) NOT NULL COMMENT '代號',
  `name` varchar(50) NOT NULL COMMENT '名稱',
  `useable` tinyint(1) NOT NULL DEFAULT '0' COMMENT '是否顯示'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `high_school_dept`
--

CREATE TABLE `high_school_dept` (
  `dept_sn` int UNSIGNED NOT NULL COMMENT '高中職科別流水號',
  `dept_name` varchar(20) NOT NULL COMMENT '高中職科別名稱',
  `dept_code` varchar(5) NOT NULL COMMENT '高中職科別代碼'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='高中職科別';

-- --------------------------------------------------------

--
-- 資料表結構 `high_school_dept_mapping`
--

CREATE TABLE `high_school_dept_mapping` (
  `dept_mapping_sn` int UNSIGNED NOT NULL COMMENT '高中職科別對應流水號',
  `school_type` int UNSIGNED NOT NULL COMMENT '高中職類型流水號',
  `group_sn` int UNSIGNED NOT NULL COMMENT '高中職群別流水號',
  `dept_sn` int UNSIGNED NOT NULL COMMENT '高中職科別流水號'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='高中職科別對應關係';

-- --------------------------------------------------------

--
-- 資料表結構 `high_school_group`
--

CREATE TABLE `high_school_group` (
  `group_sn` int UNSIGNED NOT NULL COMMENT '高中職群別流水號',
  `group_name` varchar(20) NOT NULL COMMENT '高中職群別名稱',
  `group_code` int UNSIGNED NOT NULL COMMENT '高中職群別代碼'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='高中職群別';

-- --------------------------------------------------------

--
-- 資料表結構 `high_school_type`
--

CREATE TABLE `high_school_type` (
  `school_type` int UNSIGNED NOT NULL COMMENT '高中職類型流水號',
  `school_type_name` varchar(30) NOT NULL COMMENT '高中職類型名稱',
  `school_type_code` varchar(1) NOT NULL COMMENT '高中職類型代碼'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='高中職類型';

-- --------------------------------------------------------

--
-- 資料表結構 `indicateTest`
--

CREATE TABLE `indicateTest` (
  `sn` int NOT NULL,
  `indicate` varchar(30) DEFAULT NULL COMMENT '節點代號',
  `synonym` varchar(10) DEFAULT NULL COMMENT '同等能力指標',
  `indicate_low` mediumtext COMMENT '下位節點',
  `indicate_high` mediumtext COMMENT '上位節點',
  `class` varchar(25) DEFAULT NULL,
  `subject` smallint UNSIGNED NOT NULL COMMENT '科目'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `indicate_test_predata`
--

CREATE TABLE `indicate_test_predata` (
  `sn` int NOT NULL,
  `mission_sn` int NOT NULL COMMENT '任務 sn',
  `predata` longtext NOT NULL COMMENT '預存資料',
  `date` datetime NOT NULL COMMENT '更新時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `indicator_info`
--

CREATE TABLE `indicator_info` (
  `indicator_sn` int NOT NULL,
  `subject` smallint UNSIGNED NOT NULL COMMENT '科目',
  `indicator` varchar(15) NOT NULL COMMENT '能力指標代號(數學:分年細目寫入)',
  `indicate_name` mediumtext NOT NULL COMMENT '中文能力指標說明',
  `note` mediumtext COMMENT '說明(數學:能力指標)'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='能力指標表';

-- --------------------------------------------------------

--
-- 資料表結構 `indicator_priori`
--

CREATE TABLE `indicator_priori` (
  `priori_sn` int NOT NULL,
  `subject` smallint UNSIGNED NOT NULL,
  `priori` varchar(100) NOT NULL,
  `priori_name` mediumtext NOT NULL,
  `indicator` mediumtext NOT NULL,
  `priori_note` mediumtext NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='基本學習內容';

-- --------------------------------------------------------

--
-- 資料表結構 `ItemBank_bat_fix_record`
--

CREATE TABLE `ItemBank_bat_fix_record` (
  `fix_record_sn` bigint UNSIGNED NOT NULL,
  `user_id` varchar(60) NOT NULL COMMENT '儲存申請者SN，對映至user_info的uid',
  `item_sn` bigint NOT NULL COMMENT '儲存題目編號，資料型態對應至題庫內同欄位，並根據item_type對應至對應至mission_info內的mission_type的題庫類型',
  `item_type` tinyint(1) NOT NULL COMMENT '對應至mission_info內的mission_type的題庫類型',
  `modified_ans` tinyint(1) NOT NULL COMMENT '儲存答案選項',
  `status` tinyint(1) NOT NULL COMMENT '排程狀態 1表示完成 0 表示未完成',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '創建這筆資料的時間',
  `complete_time` datetime DEFAULT NULL COMMENT '完成修正的時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `ItemBank_fix_record`
--

CREATE TABLE `ItemBank_fix_record` (
  `fix_record_sn` bigint UNSIGNED NOT NULL,
  `user_id` varchar(60) DEFAULT NULL COMMENT '儲存申請者SN，對映至user_info的uid',
  `item_sn` bigint NOT NULL COMMENT '儲存題目編號，資料型態對應至題庫內同欄位，並根據item_type對應至對應至mission_info內的mission_type的題庫類型',
  `item_type` tinyint UNSIGNED NOT NULL COMMENT '對應至mission_info內的mission_type的題庫類型',
  `modified_ans` tinyint(1) NOT NULL COMMENT '儲存答案選項',
  `status` tinyint(1) NOT NULL COMMENT '排程狀態 1表示完成 0 表示未完成',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '創建這筆資料的時間',
  `complete_time` datetime DEFAULT NULL COMMENT '完成修正的時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `item_mission_rate`
--

CREATE TABLE `item_mission_rate` (
  `item_mission_rate_sn` bigint NOT NULL,
  `mission_sn` int NOT NULL COMMENT '任務編號',
  `item_sn` int NOT NULL COMMENT '試題編號	',
  `total_student` int NOT NULL COMMENT '作答學生人數',
  `correct_student` int NOT NULL COMMENT '答對學生人數',
  `correct_percent` int NOT NULL COMMENT '答對率',
  `datetime` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `lecturer_list`
--

CREATE TABLE `lecturer_list` (
  `sn` int UNSIGNED NOT NULL COMMENT '流水號',
  `city` varchar(5) NOT NULL COMMENT '縣市',
  `school_name` varchar(20) NOT NULL COMMENT '學校',
  `lecturer_name` varchar(20) NOT NULL COMMENT '姓名',
  `email` varchar(100) NOT NULL COMMENT 'E-mail',
  `specialty` varchar(20) NOT NULL DEFAULT '' COMMENT '專長加註',
  `field` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '領域',
  `remark` varchar(50) NOT NULL COMMENT '備註',
  `category` int NOT NULL COMMENT '講師類別',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '建立日期'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='合格講師名單';

-- --------------------------------------------------------

--
-- 資料表結構 `literacy_publisher_mapping`
--

CREATE TABLE `literacy_publisher_mapping` (
  `literacy_publisher_sn` int UNSIGNED NOT NULL COMMENT '流水號',
  `literacy_item_sn` int NOT NULL COMMENT '素養試題編號(1-concept_itemBank_literacy_interactive/2-concept_itemBank_literacy/3-concept_itemBank_literacy_group)',
  `type` int NOT NULL COMMENT '素養試題類型(互動1/一般2/題組3)',
  `grade` int NOT NULL COMMENT '年級',
  `seme` varchar(4) NOT NULL COMMENT '學年度學期',
  `publisher_id` smallint UNSIGNED NOT NULL COMMENT '版本',
  `unit` int NOT NULL COMMENT '單元',
  `unit_name` varchar(255) NOT NULL COMMENT '單元名稱',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `log_exam_bug`
--

CREATE TABLE `log_exam_bug` (
  `sn` int NOT NULL COMMENT '流水號',
  `user_id` varchar(60) NOT NULL,
  `time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '發生時間',
  `file` varchar(100) NOT NULL COMMENT '程式名稱',
  `file_line` int NOT NULL COMMENT '發生行數',
  `session_data` text NOT NULL COMMENT 'SESSION資料',
  `request_data` text NOT NULL COMMENT 'REQUEST資料',
  `server_ip` varchar(20) NOT NULL COMMENT '哪一台分流機'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `log_exam_record_aassessment_detail`
--

CREATE TABLE `log_exam_record_aassessment_detail` (
  `log_id` int UNSIGNED NOT NULL COMMENT '日誌流水號',
  `original_sn` int UNSIGNED NOT NULL COMMENT '原表流水號',
  `rfh_sn` int NOT NULL COMMENT 'remedial_ftp_history.sn',
  `organization_id` varchar(10) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '測驗資料學校代碼',
  `user_id` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '學生帳號',
  `uname` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '姓名',
  `OpenID_sub` varchar(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT 'sub',
  `seme` varchar(10) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '施測學年度',
  `grade` varchar(5) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL DEFAULT '0' COMMENT '測驗資料年級',
  `class` int NOT NULL DEFAULT '0' COMMENT '測驗資料班級',
  `seat` smallint UNSIGNED NOT NULL COMMENT '座號',
  `correct_num` int NOT NULL COMMENT '答對題數',
  `correct_rate` int NOT NULL COMMENT '答對率',
  `total_num` int NOT NULL COMMENT '總題數',
  `org_res` text CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '原始作答',
  `binary_res` text CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '二元作答情形',
  `original_import_time` datetime NOT NULL COMMENT '原表匯入日期',
  `original_update_time` timestamp NOT NULL COMMENT '原表更新時間',
  `action_type` enum('USER_ID_NULL','DELETE') NOT NULL COMMENT '操作類型：USER_ID_NULL=user_id被設為null, DELETE=資料被刪除',
  `log_date` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '日誌記錄日期',
  `log_user` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '執行操作的用戶'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='exam_record_assessment_detail 變更日誌表';

-- --------------------------------------------------------

--
-- 資料表結構 `mad_exam_record`
--

CREATE TABLE `mad_exam_record` (
  `exam_sn` bigint NOT NULL COMMENT '流水號',
  `user_id` varchar(60) DEFAULT NULL COMMENT '使用者id',
  `cs_id` varchar(15) NOT NULL COMMENT '施測單元',
  `date` varchar(30) NOT NULL COMMENT '施測日期',
  `start_time` varchar(14) NOT NULL COMMENT '開始時間',
  `stop_time` varchar(14) NOT NULL DEFAULT '' COMMENT '完成時間',
  `during_time` varchar(16) NOT NULL COMMENT '測驗時間',
  `firm_id` varchar(10) DEFAULT NULL COMMENT '施測地點id',
  `score` tinyint DEFAULT NULL COMMENT '得分',
  `item_num` smallint NOT NULL COMMENT '試卷內容題號',
  `org_res` longtext COMMENT '原始作答情形',
  `saveSel` int DEFAULT NULL,
  `pattern_type` varchar(200) DEFAULT NULL COMMENT '作答類型代號',
  `exam_title` varchar(20) NOT NULL COMMENT '試卷名稱',
  `paper_vol` int NOT NULL COMMENT '試卷別',
  `oth_ans` mediumtext NOT NULL,
  `mad_type_id` varchar(20) NOT NULL DEFAULT '0' COMMENT '建構題類型代號',
  `formal` tinyint(1) NOT NULL DEFAULT '1' COMMENT '正式紀錄',
  `type_id` int NOT NULL COMMENT '試卷施測類型',
  `bug_pattern` mediumtext COMMENT '錯誤類型樣態'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='MAD測驗結果';

-- --------------------------------------------------------

--
-- 資料表結構 `mad_item`
--

CREATE TABLE `mad_item` (
  `sn` int UNSIGNED NOT NULL COMMENT '流水號',
  `item_sn` bigint NOT NULL COMMENT '試題流水號',
  `mad_type_id` varchar(20) NOT NULL COMMENT '建構題型代號',
  `item_pic` varchar(100) DEFAULT NULL COMMENT '建構題題目圖片',
  `mad_ans` mediumtext COMMENT '建構題答案'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='建構反應題型連結';

-- --------------------------------------------------------

--
-- 資料表結構 `mad_rule`
--

CREATE TABLE `mad_rule` (
  `sn` int UNSIGNED NOT NULL COMMENT '流水號',
  `mad_type_id` varchar(20) NOT NULL COMMENT '建構反應題型代號',
  `block_id` tinyint NOT NULL COMMENT '區塊代號',
  `require_file` tinyint(1) NOT NULL COMMENT '是否需要外掛檔案',
  `filename` varchar(50) NOT NULL COMMENT '外掛檔案名稱'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='建構反應題型規則';

-- --------------------------------------------------------

--
-- 資料表結構 `mad_type`
--

CREATE TABLE `mad_type` (
  `mad_type_id` int NOT NULL COMMENT '類型id',
  `name` varchar(50) NOT NULL COMMENT '類型名稱',
  `portfolio` varchar(50) DEFAULT NULL COMMENT '展示檔案'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='建構反應題型';

-- --------------------------------------------------------

--
-- 資料表結構 `main_data`
--

CREATE TABLE `main_data` (
  `num` int NOT NULL,
  `member_num` varchar(11) NOT NULL,
  `c_question_num` varchar(50) NOT NULL COMMENT '題目編號',
  `title_dsc` varchar(100) NOT NULL,
  `content_dsc` mediumtext NOT NULL,
  `option_dsc` mediumtext NOT NULL COMMENT '操作說明',
  `up_date` varchar(16) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='國教署CR題庫';

-- --------------------------------------------------------

--
-- 資料表結構 `map_arrows`
--

CREATE TABLE `map_arrows` (
  `arrow_sn` int NOT NULL,
  `x1` mediumint NOT NULL,
  `y1` mediumint NOT NULL,
  `x2` mediumint NOT NULL,
  `y2` mediumint NOT NULL,
  `class` varchar(10) NOT NULL,
  `map_sn` int NOT NULL,
  `nodes_connection` mediumtext NOT NULL COMMENT '分年細目與能力指標關聯性',
  `arrow_creater` varchar(60) DEFAULT NULL COMMENT '線段建立者',
  `date` varchar(30) NOT NULL COMMENT '建立時間日期'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `map_info`
--

CREATE TABLE `map_info` (
  `map_sn` int NOT NULL COMMENT '星空圖代號',
  `map_name` varchar(20) NOT NULL COMMENT '中文名稱',
  `subject_id` smallint UNSIGNED DEFAULT NULL COMMENT '科目代號',
  `create_user_id` varchar(60) DEFAULT NULL COMMENT '建立者名稱',
  `itembank_group` int NOT NULL COMMENT '縱貫題庫組別',
  `display` tinyint(1) NOT NULL COMMENT '是否顯示',
  `memo` varchar(50) NOT NULL COMMENT '附註',
  `map_main_file` varchar(40) DEFAULT NULL COMMENT '星空圖主檔名稱',
  `map_grade` varchar(500) NOT NULL COMMENT '可使用年級 (sub_subject_id = 0: 無子科目)',
  `time_created` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `date` datetime NOT NULL ON UPDATE CURRENT_TIMESTAMP COMMENT '更新時間 '
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `map_info_0903`
--

CREATE TABLE `map_info_0903` (
  `map_sn` int NOT NULL COMMENT '星空圖代號',
  `map_name` varchar(20) NOT NULL COMMENT '中文名稱',
  `subject_id` smallint UNSIGNED DEFAULT NULL COMMENT '科目代號',
  `create_user_id` varchar(60) DEFAULT NULL COMMENT '建立者名稱',
  `itembank_group` int NOT NULL COMMENT '縱貫題庫組別',
  `display` tinyint(1) NOT NULL COMMENT '是否顯示',
  `memo` varchar(50) NOT NULL COMMENT '附註',
  `map_main_file` varchar(40) DEFAULT NULL COMMENT '星空圖主檔名稱',
  `map_grade` varchar(500) NOT NULL COMMENT '可使用年級 (sub_subject_id = 0: 無子科目)',
  `time_created` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `date` datetime NOT NULL ON UPDATE CURRENT_TIMESTAMP COMMENT '更新時間 '
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `map_node`
--

CREATE TABLE `map_node` (
  `node_sn` int NOT NULL,
  `transform` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '節點位置',
  `indicate_id` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '能力指標ID',
  `indicate_name` varchar(500) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '能力指標名稱',
  `class` varchar(25) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '大小節點區分',
  `r` tinyint NOT NULL COMMENT '節點大小',
  `cx` mediumint NOT NULL COMMENT '節點X位置',
  `cy` mediumint NOT NULL COMMENT '節點Y位置',
  `map_sn` int NOT NULL,
  `subject` smallint UNSIGNED NOT NULL,
  `CR` tinyint(1) DEFAULT '0' COMMENT '有無CR',
  `DA` tinyint(1) DEFAULT '0' COMMENT '有無DA',
  `video` tinyint NOT NULL COMMENT '影片上傳註記',
  `prac` tinyint NOT NULL COMMENT '練習題上傳註記',
  `blank_filling` tinyint UNSIGNED NOT NULL DEFAULT '0' COMMENT '填充題',
  `teach` tinyint NOT NULL COMMENT '互動元件註記',
  `node_file` tinyint NOT NULL DEFAULT '0' COMMENT '補充教材(教材檔案)',
  `external` tinyint NOT NULL DEFAULT '0' COMMENT '補充教材(外部連結)',
  `sub_subject_id` tinyint UNSIGNED DEFAULT NULL COMMENT '子科目',
  `node_creater` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '節點建立者',
  `date` varchar(30) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '建立時間',
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `map_node_0521`
--

CREATE TABLE `map_node_0521` (
  `node_sn` int NOT NULL,
  `transform` varchar(255) NOT NULL COMMENT '節˙點位置',
  `indicate_id` varchar(50) NOT NULL COMMENT '能力指標ID',
  `indicate_name` varchar(500) NOT NULL COMMENT '能力指標名稱',
  `class` varchar(25) NOT NULL COMMENT '大小節點區分',
  `r` tinyint NOT NULL COMMENT '節點大小',
  `cx` mediumint NOT NULL COMMENT '節點X位置',
  `cy` mediumint NOT NULL COMMENT '節點Y位置',
  `map_sn` int NOT NULL,
  `subject` smallint UNSIGNED NOT NULL,
  `CR` tinyint(1) DEFAULT '0' COMMENT '有無CR',
  `DA` tinyint(1) DEFAULT '0' COMMENT '有無DA',
  `video` tinyint NOT NULL COMMENT '影片上傳註記',
  `prac` tinyint NOT NULL COMMENT '練習題上傳註記',
  `blank_filling` tinyint UNSIGNED NOT NULL DEFAULT '0' COMMENT '填充題',
  `teach` tinyint NOT NULL COMMENT '互動元件註記',
  `node_file` tinyint NOT NULL DEFAULT '0' COMMENT '	補充教材(教材檔案)',
  `external` tinyint NOT NULL DEFAULT '0' COMMENT '補充教材(外部連結)	',
  `sub_subject_id` tinyint UNSIGNED DEFAULT NULL COMMENT '子科目',
  `node_creater` varchar(60) DEFAULT NULL COMMENT '節點建立者',
  `date` varchar(30) NOT NULL COMMENT '建立時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `map_node_20240813`
--

CREATE TABLE `map_node_20240813` (
  `node_sn` int NOT NULL,
  `transform` varchar(255) NOT NULL COMMENT '節˙點位置',
  `indicate_id` varchar(50) NOT NULL COMMENT '能力指標ID',
  `indicate_name` varchar(500) NOT NULL COMMENT '能力指標名稱',
  `class` varchar(25) NOT NULL COMMENT '大小節點區分',
  `r` tinyint NOT NULL COMMENT '節點大小',
  `cx` mediumint NOT NULL COMMENT '節點X位置',
  `cy` mediumint NOT NULL COMMENT '節點Y位置',
  `map_sn` int NOT NULL,
  `subject` smallint UNSIGNED NOT NULL,
  `CR` tinyint(1) DEFAULT '0' COMMENT '有無CR',
  `DA` tinyint(1) DEFAULT '0' COMMENT '有無DA',
  `video` tinyint NOT NULL COMMENT '影片上傳註記',
  `prac` tinyint NOT NULL COMMENT '練習題上傳註記',
  `blank_filling` tinyint UNSIGNED NOT NULL DEFAULT '0' COMMENT '填充題',
  `teach` tinyint NOT NULL COMMENT '互動元件註記',
  `node_file` tinyint NOT NULL DEFAULT '0' COMMENT '	補充教材(教材檔案)',
  `external` tinyint NOT NULL DEFAULT '0' COMMENT '補充教材(外部連結)	',
  `sub_subject_id` tinyint UNSIGNED DEFAULT NULL COMMENT '子科目',
  `node_creater` varchar(60) DEFAULT NULL COMMENT '節點建立者',
  `date` varchar(30) NOT NULL COMMENT '建立時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `map_node_20251105`
--

CREATE TABLE `map_node_20251105` (
  `node_sn` int NOT NULL,
  `transform` varchar(255) NOT NULL COMMENT '節˙點位置',
  `indicate_id` varchar(50) NOT NULL COMMENT '能力指標ID',
  `indicate_name` varchar(500) NOT NULL COMMENT '能力指標名稱',
  `class` varchar(25) NOT NULL COMMENT '大小節點區分',
  `r` tinyint NOT NULL COMMENT '節點大小',
  `cx` mediumint NOT NULL COMMENT '節點X位置',
  `cy` mediumint NOT NULL COMMENT '節點Y位置',
  `map_sn` int NOT NULL,
  `subject` smallint UNSIGNED NOT NULL,
  `CR` tinyint(1) DEFAULT '0' COMMENT '有無CR',
  `DA` tinyint(1) DEFAULT '0' COMMENT '有無DA',
  `video` tinyint NOT NULL COMMENT '影片上傳註記',
  `prac` tinyint NOT NULL COMMENT '練習題上傳註記',
  `blank_filling` tinyint UNSIGNED NOT NULL DEFAULT '0' COMMENT '填充題',
  `teach` tinyint NOT NULL COMMENT '互動元件註記',
  `node_file` tinyint NOT NULL DEFAULT '0' COMMENT '	補充教材(教材檔案)',
  `external` tinyint NOT NULL DEFAULT '0' COMMENT '補充教材(外部連結)	',
  `sub_subject_id` tinyint UNSIGNED DEFAULT NULL COMMENT '子科目',
  `node_creater` varchar(60) DEFAULT NULL COMMENT '節點建立者',
  `date` varchar(30) NOT NULL COMMENT '建立時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `map_node_20251223`
--

CREATE TABLE `map_node_20251223` (
  `node_sn` int NOT NULL,
  `transform` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '節點位置',
  `indicate_id` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '能力指標ID',
  `indicate_name` varchar(500) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '能力指標名稱',
  `class` varchar(25) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '大小節點區分',
  `r` tinyint NOT NULL COMMENT '節點大小',
  `cx` mediumint NOT NULL COMMENT '節點X位置',
  `cy` mediumint NOT NULL COMMENT '節點Y位置',
  `map_sn` int NOT NULL,
  `subject` smallint UNSIGNED NOT NULL,
  `CR` tinyint(1) DEFAULT '0' COMMENT '有無CR',
  `DA` tinyint(1) DEFAULT '0' COMMENT '有無DA',
  `video` tinyint NOT NULL COMMENT '影片上傳註記',
  `prac` tinyint NOT NULL COMMENT '練習題上傳註記',
  `blank_filling` tinyint UNSIGNED NOT NULL DEFAULT '0' COMMENT '填充題',
  `teach` tinyint NOT NULL COMMENT '互動元件註記',
  `node_file` tinyint NOT NULL DEFAULT '0' COMMENT '補充教材(教材檔案)',
  `external` tinyint NOT NULL DEFAULT '0' COMMENT '補充教材(外部連結)',
  `sub_subject_id` tinyint UNSIGNED DEFAULT NULL COMMENT '子科目',
  `node_creater` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '節點建立者',
  `date` varchar(30) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '建立時間',
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `map_node_bk2`
--

CREATE TABLE `map_node_bk2` (
  `node_sn` int NOT NULL,
  `transform` varchar(255) NOT NULL COMMENT '節˙點位置',
  `indicate_id` varchar(50) NOT NULL COMMENT '能力指標ID',
  `indicate_name` varchar(500) NOT NULL COMMENT '能力指標名稱',
  `class` varchar(25) NOT NULL COMMENT '大小節點區分',
  `r` tinyint NOT NULL COMMENT '節點大小',
  `cx` mediumint NOT NULL COMMENT '節點X位置',
  `cy` mediumint NOT NULL COMMENT '節點Y位置',
  `map_sn` int NOT NULL,
  `subject` smallint UNSIGNED NOT NULL,
  `CR` tinyint(1) DEFAULT '0' COMMENT '有無CR',
  `DA` tinyint(1) DEFAULT '0' COMMENT '有無DA',
  `video` tinyint NOT NULL COMMENT '影片上傳註記',
  `prac` tinyint NOT NULL COMMENT '練習題上傳註記',
  `blank_filling` tinyint UNSIGNED NOT NULL DEFAULT '0' COMMENT '填充題',
  `teach` tinyint NOT NULL COMMENT '互動元件註記',
  `node_file` tinyint NOT NULL DEFAULT '0' COMMENT '	補充教材(教材檔案)',
  `external` tinyint NOT NULL DEFAULT '0' COMMENT '補充教材(外部連結)	',
  `sub_subject_id` tinyint UNSIGNED DEFAULT NULL COMMENT '子科目',
  `node_creater` varchar(60) DEFAULT NULL COMMENT '節點建立者',
  `date` varchar(30) NOT NULL COMMENT '建立時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `map_node_external`
--

CREATE TABLE `map_node_external` (
  `res_sn` mediumint UNSIGNED NOT NULL COMMENT '流水號',
  `indicate_id` varchar(50) NOT NULL COMMENT '細能力指標代號',
  `map_sn` int NOT NULL COMMENT '星空圖代號',
  `subject_id` smallint UNSIGNED NOT NULL COMMENT '科目代號',
  `res_type` tinyint NOT NULL COMMENT '外部資源類別 :  1-推薦影片  2-推薦連結  3-教案分享  4-相關模擬',
  `res_url` varchar(1000) NOT NULL COMMENT '資源url',
  `res_caption` varchar(100) NOT NULL COMMENT '該資源的中文描述',
  `creater` varchar(60) DEFAULT NULL COMMENT '編修者',
  `date` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '上次編修時間',
  `public` tinyint UNSIGNED NOT NULL COMMENT '2:隱藏，1:公開，0:不公開'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='各節點的外部資源';

-- --------------------------------------------------------

--
-- 資料表結構 `map_node_file`
--

CREATE TABLE `map_node_file` (
  `map_node_file_sn` bigint UNSIGNED NOT NULL,
  `map_node_sn` int NOT NULL,
  `map_sn` int NOT NULL,
  `indicator` varchar(50) NOT NULL,
  `file_type` tinyint UNSIGNED NOT NULL COMMENT '檔案類別 : 5-學習單 6-補充教材',
  `filename` mediumtext NOT NULL,
  `fileurl` mediumtext NOT NULL,
  `create_user` varchar(60) DEFAULT NULL,
  `public` tinyint(1) NOT NULL DEFAULT '0',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `map_node_grade`
--

CREATE TABLE `map_node_grade` (
  `sn` int UNSIGNED NOT NULL,
  `subject_id` smallint UNSIGNED NOT NULL,
  `map_sn` int NOT NULL,
  `indicator` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL,
  `sub_indicator` varchar(10) NOT NULL,
  `concat_indicator` varchar(255) NOT NULL,
  `grade_list` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='資料來自 sp_create_map_node_grade';

-- --------------------------------------------------------

--
-- 資料表結構 `map_node_mapping`
--

CREATE TABLE `map_node_mapping` (
  `sn` int UNSIGNED NOT NULL,
  `map_sn` int NOT NULL,
  `bnode_class` varchar(25) NOT NULL,
  `bnode_indicate_id` varchar(50) NOT NULL,
  `snode_class` varchar(25) NOT NULL,
  `snode_indicate_id` varchar(50) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `map_node_mapping_0521`
--

CREATE TABLE `map_node_mapping_0521` (
  `sn` int UNSIGNED NOT NULL,
  `map_sn` int NOT NULL,
  `bnode_class` varchar(25) NOT NULL,
  `bnode_indicate_id` varchar(50) NOT NULL,
  `snode_class` varchar(25) NOT NULL,
  `snode_indicate_id` varchar(50) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `map_node_student_status`
--

CREATE TABLE `map_node_student_status` (
  `status_sn` int NOT NULL,
  `bNodes_Status` mediumtext NOT NULL COMMENT '大節點精熟度',
  `sNodes_Status_FR` mediumtext NOT NULL COMMENT '小節點精熟及完成度',
  `node_sn` int NOT NULL,
  `user_id` varchar(60) DEFAULT NULL,
  `map_sn` int NOT NULL,
  `recommended_mission` int DEFAULT NULL COMMENT '推薦課程任務',
  `time_created` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `time_updated` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `math_laboratory_exam_record`
--

CREATE TABLE `math_laboratory_exam_record` (
  `sn` int UNSIGNED NOT NULL COMMENT '資料流水號',
  `user_id` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '使用者id',
  `page_id` varchar(50) NOT NULL COMMENT '指標代號',
  `progression` int NOT NULL COMMENT '完成進度',
  `time_spent` int UNSIGNED NOT NULL COMMENT '花費時間(毫秒)',
  `time_created` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '建立時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `message_fileattached`
--

CREATE TABLE `message_fileattached` (
  `file_sn` int NOT NULL COMMENT '流水號',
  `message_sn` int NOT NULL COMMENT '對應主訊息編號',
  `file_orgnialname` varchar(100) NOT NULL COMMENT '原始檔名',
  `file_replacename` varchar(100) NOT NULL COMMENT '編碼檔名',
  `filesrc` varchar(200) NOT NULL COMMENT '下載路徑',
  `upload_server` varchar(30) DEFAULT NULL COMMENT '在哪一台主機上傳',
  `filetype` varchar(20) NOT NULL COMMENT '檔案類型',
  `filesize` varchar(20) NOT NULL COMMENT '檔案大小(MB)',
  `upload_userid` varchar(60) DEFAULT NULL COMMENT '上傳者',
  `file_backup` tinyint(1) NOT NULL DEFAULT '0' COMMENT '0:未備份至主機；1已備份至主機',
  `upload_time` datetime NOT NULL COMMENT '上傳時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `message_fileattached_20230316`
--

CREATE TABLE `message_fileattached_20230316` (
  `file_sn` int NOT NULL COMMENT '流水號',
  `message_sn` int NOT NULL COMMENT '對應主訊息編號',
  `file_orgnialname` varchar(100) NOT NULL COMMENT '原始檔名',
  `file_replacename` varchar(100) NOT NULL COMMENT '編碼檔名',
  `filesrc` varchar(200) NOT NULL COMMENT '下載路徑',
  `upload_server` varchar(30) DEFAULT NULL COMMENT '在哪一台主機上傳',
  `filetype` varchar(20) NOT NULL COMMENT '檔案類型',
  `filesize` varchar(20) NOT NULL COMMENT '檔案大小(MB)',
  `upload_userid` varchar(60) DEFAULT NULL COMMENT '上傳者',
  `file_backup` tinyint(1) NOT NULL DEFAULT '0' COMMENT '0:未備份至主機；1已備份至主機',
  `upload_time` datetime NOT NULL COMMENT '上傳時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `message_master`
--

CREATE TABLE `message_master` (
  `msg_sn` int NOT NULL COMMENT '訊息id',
  `touser_id` varchar(60) DEFAULT NULL COMMENT '接收者id',
  `togroup` varchar(200) DEFAULT NULL COMMENT '群發對象 [學校][年級][班級]',
  `msg_type` int NOT NULL COMMENT '0:系統訊息，1:教師訊息，2:親師訊息',
  `msg_content` mediumtext NOT NULL COMMENT '問題內容',
  `create_time` datetime NOT NULL COMMENT '建立時間',
  `create_user` varchar(60) NOT NULL COMMENT '新增者',
  `attachefile` tinyint UNSIGNED NOT NULL COMMENT '是否有上傳檔案0:否；1:是',
  `read_mk` tinyint UNSIGNED NOT NULL DEFAULT '0' COMMENT '是否讀取註記',
  `delete_flag` tinyint UNSIGNED NOT NULL COMMENT '是否刪除 0:否；1:是'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci ROW_FORMAT=COMPACT;

-- --------------------------------------------------------

--
-- 資料表結構 `message_response`
--

CREATE TABLE `message_response` (
  `response_sn` int NOT NULL COMMENT '流水號',
  `message_sn` int NOT NULL COMMENT '對應主訊息編號',
  `response_content` mediumtext NOT NULL COMMENT '回覆內容',
  `remsg_create_user` varchar(60) NOT NULL COMMENT '回覆訊息者',
  `res_resmsgto_user` varchar(60) DEFAULT NULL COMMENT '單獨回覆給user (只有本人能看到)',
  `remsg_create_time` datetime NOT NULL COMMENT '回覆時間',
  `remsg_delete_flag` tinyint UNSIGNED NOT NULL COMMENT '是否刪除 0:否；1:是'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `message_response_20230316`
--

CREATE TABLE `message_response_20230316` (
  `response_sn` int NOT NULL COMMENT '流水號',
  `message_sn` int NOT NULL COMMENT '對應主訊息編號',
  `response_content` mediumtext NOT NULL COMMENT '回覆內容',
  `remsg_create_user` varchar(60) NOT NULL COMMENT '回覆訊息者',
  `res_resmsgto_user` varchar(60) DEFAULT NULL COMMENT '單獨回覆給user (只有本人能看到)',
  `remsg_create_time` datetime NOT NULL COMMENT '回覆時間',
  `remsg_delete_flag` tinyint UNSIGNED NOT NULL COMMENT '是否刪除 0:否；1:是'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `mimes`
--

CREATE TABLE `mimes` (
  `filetype` varchar(5) NOT NULL,
  `mimetype` varchar(50) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `mission_google_form`
--

CREATE TABLE `mission_google_form` (
  `sn` int NOT NULL,
  `form_type` bigint UNSIGNED DEFAULT NULL COMMENT 'mission_google_master的SN',
  `seme` varchar(6) NOT NULL,
  `user_id` varchar(60) DEFAULT NULL,
  `city` varchar(16) DEFAULT NULL,
  `org_name` varchar(40) DEFAULT NULL,
  `grade` int NOT NULL,
  `class` int NOT NULL,
  `stu_name` varchar(50) NOT NULL,
  `stu_sex` varchar(2) NOT NULL,
  `content` longtext CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci,
  `start_time` datetime NOT NULL COMMENT '開始問卷的時間',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '結束時間',
  `mission_sn` int DEFAULT NULL,
  `org_id` varchar(10) DEFAULT NULL COMMENT '學校代號'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `mission_google_master`
--

CREATE TABLE `mission_google_master` (
  `sn` bigint UNSIGNED NOT NULL,
  `form_name` mediumtext NOT NULL,
  `url` varchar(100) DEFAULT NULL,
  `form_title` mediumtext NOT NULL,
  `used` tinyint NOT NULL COMMENT '是否開放'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `mission_group_leader`
--

CREATE TABLE `mission_group_leader` (
  `sn` bigint UNSIGNED NOT NULL COMMENT '流水號',
  `user_id` varchar(60) NOT NULL COMMENT '小組長id',
  `group_id` int NOT NULL COMMENT '小組id',
  `mission_sn` int NOT NULL COMMENT '任務流水號',
  `is_used` smallint NOT NULL DEFAULT '1' COMMENT '啟用=1; 停用=0'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='任務指派小組長';

-- --------------------------------------------------------

--
-- 資料表結構 `mission_info`
--

CREATE TABLE `mission_info` (
  `mission_sn` int NOT NULL COMMENT '任務 sn ',
  `mission_nm` varchar(200) NOT NULL COMMENT '任務名稱',
  `target_id` mediumtext NOT NULL COMMENT '對象 id:user_id, class_id',
  `group_id` varchar(2000) DEFAULT NULL COMMENT '小組ID',
  `class_type` int DEFAULT NULL COMMENT 'NULL: 一般任務, 1: 課堂班級, 2: 學習扶助',
  `subject_id` smallint UNSIGNED DEFAULT NULL COMMENT '科目',
  `cp_id` varchar(15) DEFAULT NULL COMMENT '補救教學施測單元',
  `remdial_node` mediumtext COMMENT '補救教學對應節點',
  `node` mediumtext NOT NULL COMMENT 'epid、節點、paper_sn',
  `mid` int DEFAULT NULL COMMENT 'publisher_mapping_sn',
  `check_list_sn` varchar(100) DEFAULT NULL COMMENT 'check_list_table的sn',
  `date` datetime NOT NULL COMMENT '結束日期',
  `semester` smallint NOT NULL COMMENT '學年度學期',
  `history_mk` char(1) NOT NULL DEFAULT 'N' COMMENT '是否已寫入歷程',
  `teacher_id` varchar(60) DEFAULT NULL COMMENT '指派人id',
  `self_practice` int NOT NULL DEFAULT '0' COMMENT '0：老師指派 1：學生自派 2：家長指派',
  `target_type` varchar(2) NOT NULL COMMENT '對象任務種類: 個人 I  or 全班  C',
  `mission_type` tinyint UNSIGNED NOT NULL COMMENT '任務type: 0單元、1知識結構、2學習、3縱貫、4題庫單元、5題庫縱貫、6學力模擬測驗、7核心素養評量',
  `assign_type` mediumtext COMMENT '指派任務類型:1.影片2.練習題3.動態評量4.互動元件5.學習問卷NULL(練習題+影片)',
  `exam_type` int NOT NULL DEFAULT '0' COMMENT '縱貫式測驗模式(0=適性全測,1=適性),單元式測驗模式(0=順序出題,3=隨機出題)',
  `is_auto_finish` tinyint NOT NULL DEFAULT '1' COMMENT '是否開啟偵測自動完成',
  `create_date` datetime NOT NULL COMMENT '建立日期',
  `start_time` datetime NOT NULL COMMENT '開始時間',
  `end_mk` char(1) NOT NULL DEFAULT 'N' COMMENT '停用註記',
  `endYear` varchar(2000) NOT NULL COMMENT '節點終止年級',
  `topic_num` int NOT NULL COMMENT '題目數量',
  `unable` int NOT NULL DEFAULT '1' COMMENT '顯示：1 不顯示：0',
  `share` varchar(100) NOT NULL COMMENT '任務分享(第一碼：0不分享、1校內分享、2私人分享)',
  `subject_mission_share` tinyint NOT NULL DEFAULT '0' COMMENT '科任老師任務分享狀態',
  `extra_coin` int NOT NULL COMMENT '任務額外的代幣獎勵',
  `can_view_answer` tinyint(1) NOT NULL DEFAULT '0' COMMENT '是否每題作答看的到答案 1:顯示 0:不顯示',
  `copy_record` int DEFAULT NULL COMMENT '合併帳號時複製資料，對應回原始的mission_sn',
  `allowed_fastforward` tinyint(1) NOT NULL DEFAULT '0'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci ROW_FORMAT=COMPACT;

-- --------------------------------------------------------

--
-- 資料表結構 `mission_result`
--

CREATE TABLE `mission_result` (
  `result_sn` int NOT NULL COMMENT '任務結果sn',
  `user_id` varchar(60) NOT NULL COMMENT '個人id',
  `mission_sn` int NOT NULL COMMENT '任務 sn',
  `nodes_pass` mediumtext NOT NULL COMMENT '通過節點',
  `all_pass` tinyint UNSIGNED NOT NULL COMMENT '是否完成',
  `update_date` datetime NOT NULL COMMENT '更新日期',
  `history_mk` char(1) NOT NULL DEFAULT 'N' COMMENT '是否已寫入歷程'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `mission_stud_record`
--

CREATE TABLE `mission_stud_record` (
  `stu_mission_sn` int NOT NULL,
  `mission_sn` int NOT NULL COMMENT '任務sn',
  `user_id` varchar(60) NOT NULL COMMENT '學生id',
  `finish_node` mediumtext NOT NULL COMMENT '已完成的節點',
  `finish_node_count` tinyint UNSIGNED NOT NULL DEFAULT '0' COMMENT '已完成的節點數量',
  `action_compelete_count` mediumtext NOT NULL COMMENT '每個學習任務完成次數',
  `is_all_fin` int NOT NULL COMMENT '是全部完成',
  `is_get_extra_coin` tinyint NOT NULL DEFAULT '0' COMMENT '	標記是否已取得額外的代幣',
  `finish_time` datetime NOT NULL COMMENT '完成時間',
  `update_time` datetime NOT NULL ON UPDATE CURRENT_TIMESTAMP COMMENT '最後更新時間',
  `semester` int NOT NULL COMMENT '學年度'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='學生的任務狀態' ROW_FORMAT=COMPACT;

-- --------------------------------------------------------

--
-- 資料表結構 `mission_type`
--

CREATE TABLE `mission_type` (
  `sn` tinyint UNSIGNED NOT NULL,
  `mission_type_id` tinyint UNSIGNED NOT NULL COMMENT 'mission_type 代號',
  `name` varchar(20) NOT NULL COMMENT '科目名稱',
  `mission_icon` mediumtext COMMENT '任務圖示',
  `unable` int NOT NULL DEFAULT '1' COMMENT '顯示: 1; 不顯示: 0'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='學科名稱';

-- --------------------------------------------------------

--
-- 資料表結構 `mission_voice_recognition`
--

CREATE TABLE `mission_voice_recognition` (
  `voice_recognition_sn` int UNSIGNED NOT NULL,
  `voice_recognition_type` varchar(1) NOT NULL COMMENT '1指定教材; 2 自訂文本',
  `mission_sn` int UNSIGNED NOT NULL,
  `should_listen_video` varchar(2) NOT NULL COMMENT 'Y,N 是否要聆聽語音範例',
  `listen_voice_first` varchar(2) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT 'NULL不聽; Y,N 是否要先聽範例音檔',
  `voice_type` varchar(10) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT 'NULL;female;male',
  `recording_limited` int NOT NULL COMMENT '學生錄音時長限制',
  `attempt_limited` int NOT NULL COMMENT '學生錄音次數'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='語音辨識任務設定';

-- --------------------------------------------------------

--
-- 資料表結構 `monthly_student_usage`
--

CREATE TABLE `monthly_student_usage` (
  `sn` int NOT NULL,
  `user_id` varchar(60) NOT NULL,
  `data_time` json NOT NULL COMMENT '時數資料',
  `data_rank` json NOT NULL COMMENT '排名資料',
  `date` date NOT NULL,
  `insert_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `monthly_student_usage_0806`
--

CREATE TABLE `monthly_student_usage_0806` (
  `sn` int NOT NULL,
  `user_id` varchar(60) NOT NULL,
  `data_time` json NOT NULL COMMENT '時數資料',
  `data_rank` json NOT NULL COMMENT '排名資料',
  `date` date NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `monthly_student_usage_backup`
--

CREATE TABLE `monthly_student_usage_backup` (
  `sn` int NOT NULL,
  `organization_id` varchar(10) NOT NULL,
  `organization_type` varchar(4) NOT NULL,
  `postcode` varchar(10) NOT NULL,
  `city_area` varchar(10) NOT NULL,
  `city_name` varchar(10) NOT NULL,
  `user_id` varchar(60) NOT NULL,
  `grade` int NOT NULL,
  `class` int NOT NULL,
  `data` json NOT NULL,
  `date` date NOT NULL COMMENT '紀錄資料所屬月份的第一天，例如 2025-07-01 表示 2025 年 7 月的資料'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `monthly_student_usage_bcakup0804`
--

CREATE TABLE `monthly_student_usage_bcakup0804` (
  `sn` int NOT NULL,
  `organization_id` varchar(10) NOT NULL,
  `organization_type` varchar(4) NOT NULL,
  `postcode` varchar(10) NOT NULL,
  `city_area` varchar(10) NOT NULL,
  `city_name` varchar(10) NOT NULL,
  `user_id` varchar(60) NOT NULL,
  `grade` int NOT NULL,
  `class` int NOT NULL,
  `data_time` json NOT NULL COMMENT '時數資料',
  `data_rank` json NOT NULL COMMENT '排名資料'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `monthly_student_usage_rankings`
--

CREATE TABLE `monthly_student_usage_rankings` (
  `sn` bigint NOT NULL,
  `month` date NOT NULL COMMENT '排名所屬月份',
  `user_id` varchar(64) NOT NULL,
  `organization_id` varchar(10) NOT NULL,
  `school` varchar(20) NOT NULL,
  `city_name` varchar(10) NOT NULL,
  `city_area` varchar(10) NOT NULL,
  `category` varchar(64) NOT NULL COMMENT '排名類別',
  `category_value` varchar(128) DEFAULT NULL COMMENT '分類值，如縣市名稱、學校ID或年級',
  `rank_no` int NOT NULL COMMENT '名次',
  `score` int NOT NULL COMMENT '排名依據的分數，如 active_time、total_used、total_coin',
  `insert_date` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `new_class`
--

CREATE TABLE `new_class` (
  `new_class_sn` int NOT NULL,
  `organization_id` varchar(10) NOT NULL,
  `seme` smallint NOT NULL,
  `grade` int NOT NULL,
  `class_num` int NOT NULL,
  `class_name` text NOT NULL,
  `teacher_id` varchar(60) DEFAULT NULL,
  `class_join` text NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `old_nodes`
--

CREATE TABLE `old_nodes` (
  `id` int NOT NULL,
  `uuid` char(32) NOT NULL,
  `c_question_num` varchar(200) NOT NULL COMMENT '題目編號',
  `name` varchar(255) NOT NULL,
  `x` int DEFAULT NULL,
  `y` int DEFAULT NULL,
  `lid` mediumtext,
  `ch_lid` mediumtext,
  `pid` int NOT NULL,
  `type` mediumtext NOT NULL,
  `level` int NOT NULL,
  `km_id` int NOT NULL DEFAULT '1',
  `rule` varchar(255) DEFAULT NULL,
  `lock` enum('lock','unlock') NOT NULL DEFAULT 'lock',
  `open_answer` enum('close','open') NOT NULL DEFAULT 'close',
  `limit_time` int NOT NULL DEFAULT '0',
  `media` varchar(255) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='國教署練習題';

-- --------------------------------------------------------

--
-- 資料表結構 `old_uploaded_filename`
--

CREATE TABLE `old_uploaded_filename` (
  `sn` int UNSIGNED NOT NULL,
  `module` varchar(20) NOT NULL,
  `filename` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `onversation_times`
--

CREATE TABLE `onversation_times` (
  `sn` int NOT NULL,
  `user_id` varchar(60) NOT NULL,
  `conversation_sn` int NOT NULL,
  `robot_type` tinyint NOT NULL,
  `prompt_type` int DEFAULT NULL,
  `exam_sn` bigint DEFAULT NULL,
  `mission_sn` int DEFAULT NULL,
  `item_sn` bigint DEFAULT NULL,
  `map_sn` int DEFAULT NULL,
  `indicator` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL,
  `creation_time` datetime NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `openid_api_status_record`
--

CREATE TABLE `openid_api_status_record` (
  `sn` bigint UNSIGNED NOT NULL,
  `openid_user_login_record_sn` bigint UNSIGNED NOT NULL,
  `api_type` smallint UNSIGNED NOT NULL COMMENT '1/user_info, 2/edu_info, 3/relation, 4/getHashGuidTokken, 5/getHashGuid',
  `http_status_code` smallint UNSIGNED NOT NULL,
  `execution_time` int UNSIGNED DEFAULT NULL COMMENT '單位為ms'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `openid_class_info`
--

CREATE TABLE `openid_class_info` (
  `sn` bigint UNSIGNED NOT NULL,
  `openid_user_login_record_sn` bigint UNSIGNED NOT NULL,
  `grade` char(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL,
  `class_no` char(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL,
  `school_id` varchar(10) DEFAULT NULL COMMENT 'OpenID傳來的學校代號',
  `year` char(3) DEFAULT NULL,
  `semester` char(2) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `openid_titles_info`
--

CREATE TABLE `openid_titles_info` (
  `sn` bigint UNSIGNED NOT NULL,
  `openid_user_login_record_sn` bigint UNSIGNED NOT NULL,
  `titles` varchar(30) DEFAULT NULL,
  `school_id` varchar(10) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `openid_user_data_check_status`
--

CREATE TABLE `openid_user_data_check_status` (
  `sn` bigint UNSIGNED NOT NULL,
  `openid_user_login_record_sn` bigint UNSIGNED NOT NULL,
  `openid_user_data_check_status_discription_sn` bigint UNSIGNED NOT NULL COMMENT 'OpenID傳來的學校代號'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `openid_user_data_check_status_discription`
--

CREATE TABLE `openid_user_data_check_status_discription` (
  `sn` bigint UNSIGNED NOT NULL,
  `error_discription` varchar(255) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `openid_user_info`
--

CREATE TABLE `openid_user_info` (
  `sn` bigint UNSIGNED NOT NULL,
  `openid_user_login_record_sn` bigint UNSIGNED NOT NULL COMMENT 'openid_user_login_record的PKEY',
  `name` varchar(30) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '姓名',
  `guid` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '處理過的身分證',
  `guid_token` varchar(255) DEFAULT NULL COMMENT '取得guid用的token',
  `prefferd_name` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT 'OpenID帳號',
  `school_id` varchar(10) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT 'OpenID傳來的學校代號',
  `mentor_grade` varchar(10) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '老師為導師班，學生為就讀班的年級',
  `mentor_class` varchar(10) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '老師為導師班，學生為就讀班的班級',
  `seat_no` varchar(10) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '老師身分無座號，學生才有',
  `email` varchar(256) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT 'OpenID帳號紀錄的電子信箱'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `openid_user_login_record`
--

CREATE TABLE `openid_user_login_record` (
  `sn` bigint UNSIGNED NOT NULL,
  `sub` varchar(36) DEFAULT NULL,
  `login_time` datetime DEFAULT NULL,
  `error_status` tinyint DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `openid_user_login_title`
--

CREATE TABLE `openid_user_login_title` (
  `sn` bigint UNSIGNED NOT NULL,
  `openid_user_login_record_sn` bigint UNSIGNED NOT NULL,
  `titles` varchar(30) DEFAULT NULL,
  `school_id` varchar(10) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `organization`
--

CREATE TABLE `organization` (
  `organization_id` varchar(10) NOT NULL COMMENT '學校代號',
  `type` varchar(4) DEFAULT NULL COMMENT '類型',
  `name` varchar(40) NOT NULL COMMENT '學校名稱',
  `city_code` varchar(20) NOT NULL COMMENT '城市名稱',
  `address` varchar(80) DEFAULT NULL COMMENT '地址',
  `telno` varchar(20) DEFAULT NULL,
  `location_terrain` varchar(2) DEFAULT NULL COMMENT '所在地地形',
  `class_name_type` int NOT NULL DEFAULT '0' COMMENT '班級名稱種類',
  `used` tinyint(1) NOT NULL DEFAULT '0',
  `openid_trust` tinyint(1) NOT NULL DEFAULT '0' COMMENT 'OpenID全權信任(0：關、1：開)(Default ：0) ',
  `openid_classinfo_trust` tinyint UNSIGNED NOT NULL DEFAULT '0' COMMENT 'OpenID班級資料信任(0：關、1：開)(Default ：0)',
  `openid_seatno_trust` tinyint UNSIGNED NOT NULL DEFAULT '0' COMMENT 'OpenID座號信任(0：關、1：開)(Default ：0)',
  `time_created` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `time_updated` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='單位(學校)資訊';

-- --------------------------------------------------------

--
-- 資料表結構 `organization_ace_setting`
--

CREATE TABLE `organization_ace_setting` (
  `sn` int UNSIGNED NOT NULL,
  `organization_id` varchar(10) NOT NULL,
  `access_level` int NOT NULL,
  `module_id` smallint UNSIGNED NOT NULL,
  `enabled` tinyint UNSIGNED NOT NULL DEFAULT '1'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='學校-網站功能權限設定 (ACE，access control entries)';

-- --------------------------------------------------------

--
-- 資料表結構 `organization_Education_Administration`
--

CREATE TABLE `organization_Education_Administration` (
  `sn` int UNSIGNED NOT NULL,
  `organization_id` varchar(10) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `organization_exception`
--

CREATE TABLE `organization_exception` (
  `organization_id` varchar(10) NOT NULL,
  `basical_ability` int NOT NULL DEFAULT '0' COMMENT '指派學力測驗任務功能',
  `report` tinyint(1) NOT NULL DEFAULT '1' COMMENT '報表查詢是否排除展示國小 0使用 1關閉',
  `rank` tinyint(1) NOT NULL DEFAULT '1' COMMENT '榮譽榜'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `organization_project_participation`
--

CREATE TABLE `organization_project_participation` (
  `sn` int UNSIGNED NOT NULL,
  `organization_id` varchar(10) NOT NULL,
  `project_name` varchar(50) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `organization_subject_group`
--

CREATE TABLE `organization_subject_group` (
  `sn` int NOT NULL,
  `year` varchar(6) NOT NULL COMMENT '學年',
  `level` tinyint NOT NULL COMMENT '領域合作學校(核心1中心2種子3基地學校4數位學習5夥伴學校6)	',
  `organization_id` varchar(10) NOT NULL,
  `subject_id` tinyint UNSIGNED DEFAULT NULL COMMENT '學校領域 ,與 subject 無關係 '
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `paper_teacher`
--

CREATE TABLE `paper_teacher` (
  `sn` bigint NOT NULL,
  `map_sn` int NOT NULL COMMENT 'map_sn',
  `teacher_user_id` varchar(60) NOT NULL COMMENT '教師帳號'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='協助組卷名單 for movePaperToOperator.php';

-- --------------------------------------------------------

--
-- 資料表結構 `prac_answer`
--

CREATE TABLE `prac_answer` (
  `sn` int NOT NULL,
  `user_id` varchar(60) NOT NULL COMMENT '使用者id',
  `indicator` varchar(50) NOT NULL COMMENT '能力指標',
  `date` datetime NOT NULL COMMENT '施測日期',
  `score_rate` tinyint UNSIGNED DEFAULT NULL COMMENT '正確率',
  `start_time` varchar(14) NOT NULL COMMENT '開始時間',
  `stop_time` varchar(14) NOT NULL COMMENT '結束時間',
  `during_time` varchar(14) NOT NULL COMMENT '測驗時間',
  `questions` mediumtext NOT NULL COMMENT '試卷題號',
  `org_res` mediumtext NOT NULL COMMENT '原始作答情形',
  `binary_res` mediumtext NOT NULL COMMENT '二元作答情形',
  `mission_sn` int DEFAULT NULL,
  `reward_mk` char(1) NOT NULL DEFAULT 'N' COMMENT '是否已計算獎勵',
  `client_items_idle_time` varchar(200) NOT NULL COMMENT 'client端每題作答時間(毫秒)'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `prac_blank_filling_itembank`
--

CREATE TABLE `prac_blank_filling_itembank` (
  `item_sn` int UNSIGNED NOT NULL,
  `map_sn` int NOT NULL,
  `indicator` varchar(50) NOT NULL,
  `src` varchar(255) NOT NULL,
  `used` int NOT NULL DEFAULT '1' COMMENT '1:顯示; 0: 不顯示',
  `questions_img` varchar(255) NOT NULL,
  `solution_img` varchar(255) NOT NULL COMMENT '解答圖檔'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `prac_blank_filling_itembank_20240813`
--

CREATE TABLE `prac_blank_filling_itembank_20240813` (
  `item_sn` int UNSIGNED NOT NULL,
  `map_sn` int NOT NULL,
  `indicator` varchar(50) NOT NULL,
  `src` varchar(255) NOT NULL,
  `used` int NOT NULL DEFAULT '1' COMMENT '1:顯示; 0: 不顯示',
  `questions_img` varchar(255) NOT NULL,
  `solution_img` varchar(255) NOT NULL COMMENT '解答圖檔'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `prac_blank_filling_record`
--

CREATE TABLE `prac_blank_filling_record` (
  `sn` int UNSIGNED NOT NULL,
  `user_id` varchar(60) NOT NULL,
  `item_sn` int UNSIGNED NOT NULL,
  `correct` int UNSIGNED NOT NULL COMMENT '0: 答錯; 1: 答對',
  `mission_sn` int DEFAULT NULL,
  `client_items_idle_time` varchar(200) NOT NULL COMMENT 'client端每題作答時間(毫秒)',
  `answer_record` mediumtext NOT NULL COMMENT '作答紀錄',
  `binary_res` varchar(20) NOT NULL COMMENT '二元作答情形',
  `start_time` datetime NOT NULL,
  `end_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `prac_options`
--

CREATE TABLE `prac_options` (
  `id` int NOT NULL,
  `value` mediumtext NOT NULL,
  `correct` enum('true','false') NOT NULL,
  `questions_id` int NOT NULL,
  `oid` int DEFAULT NULL,
  `time_created` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `time_updated` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `prac_questions`
--

CREATE TABLE `prac_questions` (
  `id` int NOT NULL,
  `topic` mediumtext NOT NULL,
  `detail_filename` text COMMENT '本題詳解',
  `type` enum('choose','multi_choose','fill') NOT NULL,
  `difficulty` tinyint UNSIGNED NOT NULL DEFAULT '1' COMMENT '難易度(1:一般, 2:挑戰)',
  `media_url` mediumtext,
  `tips` mediumtext,
  `score` int NOT NULL,
  `public` enum('true','false') NOT NULL,
  `nodes_uuid` char(32) DEFAULT NULL,
  `indicator` varchar(50) DEFAULT NULL,
  `map_sn` int DEFAULT NULL COMMENT 'indicator map_sn',
  `oid` int DEFAULT NULL,
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `prac_session_data`
--

CREATE TABLE `prac_session_data` (
  `session_id` bigint NOT NULL COMMENT '流水號',
  `user_id` varchar(60) NOT NULL COMMENT '登入帳號',
  `date` varchar(30) NOT NULL,
  `session_data` longtext NOT NULL COMMENT 'session 資料',
  `post_data` longtext NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `prac_single_answer`
--

CREATE TABLE `prac_single_answer` (
  `sn` int NOT NULL,
  `user_id` varchar(60) NOT NULL COMMENT '使用者id',
  `indicator` varchar(50) NOT NULL COMMENT '能力指標',
  `date` datetime NOT NULL COMMENT '施測日期',
  `score_rate` tinyint UNSIGNED DEFAULT NULL COMMENT '正確率',
  `start_time` varchar(14) NOT NULL COMMENT '開始時間',
  `stop_time` varchar(14) NOT NULL COMMENT '結束時間',
  `during_time` varchar(14) NOT NULL COMMENT '測驗時間',
  `questions` mediumtext NOT NULL COMMENT '試卷題號',
  `org_res` mediumtext NOT NULL COMMENT '原始作答情形',
  `binary_res` mediumtext NOT NULL COMMENT '二元作答情形',
  `mission_sn` int DEFAULT NULL,
  `reward_mk` char(1) NOT NULL DEFAULT 'N' COMMENT '是否已計算獎勵',
  `client_items_idle_time` varchar(200) NOT NULL COMMENT 'client端每題作答時間(毫秒)'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `priori_itembank`
--

CREATE TABLE `priori_itembank` (
  `priori_item_sn` int UNSIGNED NOT NULL,
  `exam_range` varchar(30) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '施測日期',
  `subject_name` varchar(10) NOT NULL COMMENT '施測科目',
  `exam_grade` int NOT NULL COMMENT '測驗年級',
  `html_filename` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '題目主檔路徑包含名(.htm)',
  `img_filename` varchar(60) DEFAULT NULL COMMENT '題目圖片路徑包含名',
  `item_desc` varchar(200) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '題目的中文描述，可用Latex語法須加上前墜 dispaly',
  `option_filename` varchar(200) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '選項檔案路徑包含名',
  `option_desc` varchar(200) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '選項的中文描述，可用Latex語法須加上前墜 dispaly',
  `correct_ans` tinyint(1) NOT NULL COMMENT '正確答案，對照選項的順序',
  `indicator_priori` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '本題能力指標代號，用@XX@符號切割',
  `creation_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '建立時間',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '更新時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='科技化評量歷年題目資訊';

-- --------------------------------------------------------

--
-- 資料表結構 `publisher`
--

CREATE TABLE `publisher` (
  `publisher_id` smallint UNSIGNED NOT NULL COMMENT '出版商編號',
  `publisher` varchar(20) NOT NULL COMMENT '出版商',
  `remark` char(2) NOT NULL DEFAULT '0' COMMENT '1:版本列表',
  `remark2` tinyint NOT NULL COMMENT '任務_單元診斷',
  `semeyear` varchar(3) DEFAULT NULL COMMENT '學年度(*表不分學年度)'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `publisher_mapping`
--

CREATE TABLE `publisher_mapping` (
  `mapping_sn` int NOT NULL COMMENT '流水號 (課程列表mid)',
  `sems` varchar(4) NOT NULL COMMENT '學年度學期',
  `publisher_id` smallint UNSIGNED NOT NULL COMMENT '版本',
  `subject_id` smallint UNSIGNED NOT NULL COMMENT '科目',
  `grade` int NOT NULL COMMENT '年級',
  `unit` int NOT NULL COMMENT '單元',
  `unit_sub` tinyint NOT NULL,
  `unit_name` varchar(50) NOT NULL COMMENT '單元name',
  `indicator` varchar(255) NOT NULL COMMENT '能力指標',
  `sub_indicator` varchar(10) NOT NULL COMMENT '-年級(ex. -01,第三碼)',
  `updater` varchar(60) DEFAULT NULL,
  `creation_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `publisher_mapping_0521`
--

CREATE TABLE `publisher_mapping_0521` (
  `mapping_sn` int NOT NULL COMMENT '流水號 (課程列表mid)',
  `sems` varchar(4) NOT NULL COMMENT '學年度學期',
  `publisher_id` smallint UNSIGNED NOT NULL COMMENT '版本',
  `subject_id` smallint UNSIGNED NOT NULL COMMENT '科目',
  `grade` int NOT NULL COMMENT '年級',
  `unit` int NOT NULL COMMENT '單元',
  `unit_sub` tinyint NOT NULL,
  `unit_name` varchar(50) NOT NULL COMMENT '單元name',
  `indicator` varchar(255) NOT NULL COMMENT '能力指標',
  `sub_indicator` varchar(10) NOT NULL COMMENT '-年級(ex. -01,第三碼)',
  `updater` varchar(60) DEFAULT NULL,
  `creation_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `publisher_mapping_20240813`
--

CREATE TABLE `publisher_mapping_20240813` (
  `mapping_sn` int NOT NULL COMMENT '流水號 (課程列表mid)',
  `sems` varchar(4) NOT NULL COMMENT '學年度學期',
  `publisher_id` smallint UNSIGNED NOT NULL COMMENT '版本',
  `subject_id` smallint UNSIGNED NOT NULL COMMENT '科目',
  `grade` int NOT NULL COMMENT '年級',
  `unit` int NOT NULL COMMENT '單元',
  `unit_sub` tinyint NOT NULL,
  `unit_name` varchar(50) NOT NULL COMMENT '單元name',
  `indicator` varchar(255) NOT NULL COMMENT '能力指標',
  `sub_indicator` varchar(10) NOT NULL COMMENT '-年級(ex. -01,第三碼)',
  `updater` varchar(60) DEFAULT NULL,
  `creation_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `publisher_mapping_20250207`
--

CREATE TABLE `publisher_mapping_20250207` (
  `mapping_sn` int NOT NULL COMMENT '流水號 (課程列表mid)',
  `sems` varchar(4) NOT NULL COMMENT '學年度學期',
  `publisher_id` smallint UNSIGNED NOT NULL COMMENT '版本',
  `subject_id` smallint UNSIGNED NOT NULL COMMENT '科目',
  `grade` int NOT NULL COMMENT '年級',
  `unit` int NOT NULL COMMENT '單元',
  `unit_sub` tinyint NOT NULL,
  `unit_name` varchar(50) NOT NULL COMMENT '單元name',
  `indicator` varchar(255) NOT NULL COMMENT '能力指標',
  `sub_indicator` varchar(10) NOT NULL COMMENT '-年級(ex. -01,第三碼)',
  `updater` varchar(60) DEFAULT NULL,
  `creation_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `publisher_mapping_backup_20220608`
--

CREATE TABLE `publisher_mapping_backup_20220608` (
  `mapping_sn` int NOT NULL COMMENT '流水號 (課程列表mid)',
  `sems` varchar(4) NOT NULL COMMENT '學年度學期',
  `publisher_id` int NOT NULL COMMENT '版本',
  `subject_id` int NOT NULL COMMENT '科目',
  `grade` int NOT NULL COMMENT '年級',
  `unit` int NOT NULL COMMENT '單元',
  `unit_sub` tinyint NOT NULL,
  `unit_name` varchar(50) NOT NULL COMMENT '單元name',
  `indicator` varchar(255) NOT NULL COMMENT '能力指標',
  `sub_indicator` varchar(10) NOT NULL COMMENT '-年級(ex. -01,第三碼)'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `publisher_mapping_backup_20220628`
--

CREATE TABLE `publisher_mapping_backup_20220628` (
  `mapping_sn` int NOT NULL COMMENT '流水號 (課程列表mid)',
  `sems` varchar(4) NOT NULL COMMENT '學年度學期',
  `publisher_id` int NOT NULL COMMENT '版本',
  `subject_id` int NOT NULL COMMENT '科目',
  `grade` int NOT NULL COMMENT '年級',
  `unit` int NOT NULL COMMENT '單元',
  `unit_sub` tinyint NOT NULL,
  `unit_name` varchar(50) NOT NULL COMMENT '單元name',
  `indicator` varchar(255) NOT NULL COMMENT '能力指標',
  `sub_indicator` varchar(10) NOT NULL COMMENT '-年級(ex. -01,第三碼)'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `publisher_mapping_log`
--

CREATE TABLE `publisher_mapping_log` (
  `sn` int UNSIGNED NOT NULL,
  `updater_id` varchar(60) DEFAULT NULL,
  `updater_account` varchar(60) DEFAULT NULL,
  `sems` varchar(4) NOT NULL COMMENT '學年度學期',
  `publisher_id` smallint NOT NULL COMMENT '版本',
  `subject_id` tinyint UNSIGNED NOT NULL COMMENT '科目',
  `grade` int UNSIGNED NOT NULL COMMENT '年級',
  `unit` int NOT NULL COMMENT '單元',
  `unit_name` varchar(50) NOT NULL COMMENT '單元name',
  `indicator` varchar(255) NOT NULL COMMENT '能力指標',
  `sub_indicator` varchar(10) NOT NULL COMMENT '-年級(ex. -01,第三碼)	',
  `deletion_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `questionaire_export_permission`
--

CREATE TABLE `questionaire_export_permission` (
  `sn` int UNSIGNED NOT NULL,
  `access_level` int NOT NULL,
  `form_type` bigint UNSIGNED NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `question_main`
--

CREATE TABLE `question_main` (
  `num` int NOT NULL,
  `c_question_num` varchar(50) NOT NULL COMMENT '題型編號',
  `c_title` mediumtext NOT NULL COMMENT '題目',
  `c_memo` mediumtext NOT NULL COMMENT '備註',
  `c_answer` tinyint UNSIGNED NOT NULL COMMENT '答案',
  `c_ans_1_dsc` mediumtext NOT NULL COMMENT '答案選項1的值',
  `c_ans_2_dsc` mediumtext NOT NULL COMMENT '答案選項2的值',
  `c_ans_3_dsc` mediumtext NOT NULL COMMENT '答案選項3的值',
  `c_ans_4_dsc` mediumtext NOT NULL COMMENT '答案選項4的值',
  `up_date` varchar(16) NOT NULL COMMENT '更新日期'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='國教署DA題庫';

-- --------------------------------------------------------

--
-- 資料表結構 `remedial_class`
--

CREATE TABLE `remedial_class` (
  `class_sn` int NOT NULL,
  `cp_id` varchar(20) NOT NULL COMMENT '補救教學唯一ID',
  `mission_sn` int DEFAULT NULL COMMENT '任務SN (NULL表示學習扶助班級, 有的話是學力模擬測驗)',
  `new_class_sn` int DEFAULT NULL COMMENT '課堂班級編號(0表示不是課堂班級)',
  `class_name` mediumtext NOT NULL COMMENT '補救教學班級名稱',
  `subject_id` smallint UNSIGNED DEFAULT NULL COMMENT '科目',
  `seme` int NOT NULL COMMENT '學期',
  `create_id` varchar(60) DEFAULT NULL COMMENT '建立者id',
  `create_date` date NOT NULL COMMENT '建立日期',
  `delete_mark` int NOT NULL DEFAULT '0'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='補救教學測驗(主檔)';

-- --------------------------------------------------------

--
-- 資料表結構 `remedial_ftp_history`
--

CREATE TABLE `remedial_ftp_history` (
  `sn` int NOT NULL,
  `start_time` datetime NOT NULL COMMENT '匯入開始日期-時間',
  `file_upload_time` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00' ON UPDATE CURRENT_TIMESTAMP COMMENT '更新時間',
  `status` int NOT NULL COMMENT '(0=尚未加入排程 1=加入排程, 2=正在匯入, 3=匯入成功, 4=匯入失敗)',
  `type` int NOT NULL COMMENT '匯入方式(1=批次, 2=手動)',
  `user_id` varchar(60) DEFAULT NULL COMMENT '手動匯入=user_id, 批次=system',
  `file_name` varchar(50) NOT NULL COMMENT '檔案名稱',
  `file_title` varchar(50) NOT NULL COMMENT '表頭資訊',
  `file_type` int NOT NULL COMMENT '匯入類別(1=學生名單, 2=測驗統計結果,3=個人反應)',
  `subject_id` smallint UNSIGNED DEFAULT NULL COMMENT '科目代碼(從檔案名稱取出判斷)',
  `version` int NOT NULL COMMENT '版本',
  `import_record` int NOT NULL COMMENT '匯入資訊'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `remedial_result_history`
--

CREATE TABLE `remedial_result_history` (
  `sn` bigint UNSIGNED NOT NULL,
  `organization_id` varchar(10) NOT NULL,
  `exam_range` varchar(100) NOT NULL,
  `file_name` char(255) NOT NULL,
  `json_data` mediumtext NOT NULL,
  `import_date` date NOT NULL,
  `import_time` time NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `remedial_student`
--

CREATE TABLE `remedial_student` (
  `remedial_student_sn` int NOT NULL,
  `class_sn` int DEFAULT NULL COMMENT '對應補救教學班級編號',
  `cp_id` varchar(20) DEFAULT NULL COMMENT '補救教學唯一ID',
  `student_id` varchar(60) NOT NULL COMMENT '學生ID',
  `new_class_agree` int DEFAULT NULL COMMENT '學生對應課堂班級代碼'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='補救教學測驗(主檔)';

-- --------------------------------------------------------

--
-- 資料表結構 `remedial_teacher`
--

CREATE TABLE `remedial_teacher` (
  `remedial_teacher_sn` int NOT NULL,
  `class_sn` int NOT NULL COMMENT '對應補救教學班級編號',
  `teacher_id` varchar(60) NOT NULL COMMENT '老師ID'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='補救教學測驗(主檔)';

-- --------------------------------------------------------

--
-- 資料表結構 `report_dailyusage`
--

CREATE TABLE `report_dailyusage` (
  `sn` int NOT NULL,
  `organization_id` varchar(10) DEFAULT NULL COMMENT '學校代號',
  `city_name` varchar(10) NOT NULL,
  `postcode` varchar(10) NOT NULL,
  `city_area` varchar(10) NOT NULL,
  `name` varchar(50) NOT NULL,
  `type` varchar(4) NOT NULL,
  `grade` int NOT NULL,
  `class` int NOT NULL,
  `teacher_count_name` blob NOT NULL COMMENT '教師人名',
  `teacher_active_name` blob NOT NULL COMMENT '沒有潛水的老師名',
  `teacher_idle_name` blob NOT NULL COMMENT '潛水的老師名',
  `student_count_name` blob NOT NULL COMMENT '學生人名',
  `student_active_name` blob NOT NULL COMMENT '沒有潛水的學生名',
  `student_idle_name` blob NOT NULL COMMENT '潛水的學生名',
  `teacher_count` int NOT NULL COMMENT '教師人數',
  `teacher_active` int NOT NULL COMMENT '沒有潛水的老師',
  `teacher_idle` int NOT NULL COMMENT '潛水的老師',
  `student_count` int NOT NULL COMMENT '學生人數',
  `student_active` int NOT NULL COMMENT '沒有潛水的學生',
  `student_idle` int NOT NULL COMMENT '潛水的學生',
  `datetime_log` date NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `report_grad_student_dynamic`
--

CREATE TABLE `report_grad_student_dynamic` (
  `sn` int NOT NULL,
  `organization_id` varchar(40) NOT NULL COMMENT '學校代號',
  `city_name` varchar(10) NOT NULL,
  `postcode` varchar(10) NOT NULL,
  `city_area` varchar(10) NOT NULL,
  `name` varchar(50) NOT NULL,
  `type` varchar(4) NOT NULL,
  `map_sn` int NOT NULL,
  `grade` int NOT NULL,
  `class` int NOT NULL,
  `question_student` int NOT NULL COMMENT '學生作答題數',
  `number_student` int NOT NULL COMMENT '學生作答次數',
  `people_student` int NOT NULL COMMENT '學生作答人數',
  `name_student` blob NOT NULL COMMENT '學生作答名字',
  `time_student` int NOT NULL,
  `datetime_log` varchar(30) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `report_grad_student_indicate`
--

CREATE TABLE `report_grad_student_indicate` (
  `sn` int NOT NULL,
  `organization_id` varchar(40) NOT NULL COMMENT '學校代號',
  `city_name` varchar(10) NOT NULL,
  `postcode` varchar(10) NOT NULL,
  `city_area` varchar(10) NOT NULL,
  `name` varchar(50) NOT NULL,
  `type` varchar(4) NOT NULL,
  `map_sn` int NOT NULL,
  `grade` int NOT NULL,
  `class` int NOT NULL,
  `question_student` int NOT NULL COMMENT '學生作答題數',
  `number_student` int NOT NULL COMMENT '學生作答次數',
  `people_student` int NOT NULL COMMENT '學生作答人數',
  `name_student` blob NOT NULL COMMENT '學生作答名字',
  `time_student` int NOT NULL,
  `datetime_log` varchar(30) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `report_grad_student_node`
--

CREATE TABLE `report_grad_student_node` (
  `sn` int NOT NULL,
  `organization_id` varchar(10) DEFAULT NULL COMMENT '學校代號',
  `city_name` varchar(10) NOT NULL,
  `postcode` varchar(10) NOT NULL,
  `city_area` varchar(10) NOT NULL,
  `name` varchar(50) NOT NULL,
  `type` varchar(4) NOT NULL,
  `map_sn` int NOT NULL,
  `grade` int NOT NULL,
  `class` int NOT NULL,
  `passnode_student` int NOT NULL COMMENT '學生精通節點',
  `failnode_student` int NOT NULL COMMENT '學生未精通節點',
  `people_student` int NOT NULL COMMENT '學生測驗人數',
  `name_student` blob NOT NULL COMMENT '學生測驗名字',
  `datetime_log` varchar(30) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `report_grad_student_practice`
--

CREATE TABLE `report_grad_student_practice` (
  `sn` int NOT NULL,
  `organization_id` varchar(10) DEFAULT NULL COMMENT '學校代號',
  `city_name` varchar(10) NOT NULL,
  `postcode` varchar(10) NOT NULL,
  `city_area` varchar(10) NOT NULL,
  `name` varchar(50) NOT NULL,
  `type` varchar(4) NOT NULL,
  `map_sn` int NOT NULL,
  `grade` int NOT NULL,
  `class` int NOT NULL,
  `question_student` int NOT NULL COMMENT '學生作答題數',
  `number_student` int NOT NULL COMMENT '學生作答次數',
  `people_student` int NOT NULL COMMENT '學生作答人數',
  `name_student` blob NOT NULL COMMENT '學生作答名字',
  `time_student` int NOT NULL,
  `datetime_log` varchar(30) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `report_grad_student_robot`
--

CREATE TABLE `report_grad_student_robot` (
  `sn` int NOT NULL,
  `organization_id` varchar(10) DEFAULT NULL COMMENT '學校代號',
  `city_name` varchar(10) NOT NULL,
  `postcode` varchar(10) NOT NULL,
  `city_area` varchar(10) NOT NULL,
  `name` varchar(50) NOT NULL,
  `type` varchar(4) NOT NULL,
  `robot_type` tinyint NOT NULL,
  `map_sn` int NOT NULL,
  `grade` int NOT NULL,
  `class` int NOT NULL,
  `number_teacher` int NOT NULL COMMENT '教師作答次數',
  `people_teacher` int NOT NULL COMMENT '教師作答人數',
  `name_teacher` blob NOT NULL COMMENT '教師作答名字',
  `number_student` int NOT NULL COMMENT '學生作答次數',
  `people_student` int NOT NULL COMMENT '學生作答人數',
  `name_student` blob NOT NULL COMMENT '學生作答名字',
  `time_teacher` int NOT NULL,
  `time_student` int NOT NULL,
  `datetime_log` varchar(30) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `report_grad_student_unit`
--

CREATE TABLE `report_grad_student_unit` (
  `sn` int NOT NULL,
  `organization_id` varchar(10) DEFAULT NULL COMMENT '學校代號',
  `city_name` varchar(10) NOT NULL,
  `postcode` varchar(10) NOT NULL,
  `city_area` varchar(10) NOT NULL,
  `name` varchar(50) NOT NULL,
  `type` varchar(4) NOT NULL,
  `map_sn` int NOT NULL,
  `grade` int NOT NULL,
  `class` int NOT NULL,
  `question_student` int NOT NULL COMMENT '學生作答題數',
  `number_student` int NOT NULL COMMENT '學生作答次數',
  `people_student` int NOT NULL COMMENT '學生作答人數',
  `name_student` blob NOT NULL COMMENT '學生作答名字',
  `time_student` int NOT NULL,
  `datetime_log` varchar(30) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `report_grad_student_video`
--

CREATE TABLE `report_grad_student_video` (
  `sn` int NOT NULL,
  `organization_id` varchar(10) DEFAULT NULL COMMENT '學校代號',
  `city_name` varchar(10) NOT NULL,
  `postcode` varchar(10) NOT NULL,
  `city_area` varchar(10) NOT NULL,
  `name` varchar(50) NOT NULL,
  `type` varchar(4) NOT NULL,
  `map_sn` int NOT NULL,
  `grade` int NOT NULL,
  `class` int NOT NULL,
  `time_student` int NOT NULL COMMENT '學生觀看時間',
  `number_student` int NOT NULL COMMENT '學生觀看次數',
  `people_student` int NOT NULL COMMENT '學生觀看人數',
  `name_student` blob NOT NULL COMMENT '學生觀看名字',
  `datetime_log` varchar(30) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `report_learningeffect_dynamic`
--

CREATE TABLE `report_learningeffect_dynamic` (
  `sn` int NOT NULL,
  `organization_id` varchar(10) DEFAULT NULL COMMENT '學校代號',
  `city_name` varchar(10) NOT NULL,
  `postcode` varchar(10) NOT NULL,
  `city_area` varchar(10) NOT NULL,
  `name` varchar(50) NOT NULL,
  `type` varchar(4) NOT NULL,
  `map_sn` int NOT NULL,
  `grade` int NOT NULL,
  `class` int NOT NULL,
  `question_teacher` int NOT NULL COMMENT '教師作答題數',
  `number_teacher` int NOT NULL COMMENT '教師作答次數',
  `people_teacher` int NOT NULL COMMENT '教師作答人數',
  `name_teacher` blob NOT NULL COMMENT '教師作答名字',
  `question_student` int NOT NULL COMMENT '學生作答題數',
  `number_student` int NOT NULL COMMENT '學生作答次數',
  `people_student` int NOT NULL COMMENT '學生作答人數',
  `name_student` blob NOT NULL COMMENT '學生作答名字',
  `time_teacher` int NOT NULL,
  `time_student` int NOT NULL,
  `datetime_log` varchar(30) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `report_learningeffect_indicate`
--

CREATE TABLE `report_learningeffect_indicate` (
  `sn` int NOT NULL,
  `organization_id` varchar(10) DEFAULT NULL COMMENT '學校代號',
  `city_name` varchar(10) NOT NULL,
  `postcode` varchar(10) NOT NULL,
  `city_area` varchar(10) NOT NULL,
  `name` varchar(50) NOT NULL,
  `type` varchar(4) NOT NULL,
  `map_sn` int NOT NULL,
  `grade` int NOT NULL,
  `class` int NOT NULL,
  `question_teacher` int NOT NULL COMMENT '教師作答題數',
  `number_teacher` int NOT NULL COMMENT '教師作答次數',
  `people_teacher` int NOT NULL COMMENT '教師作答人數',
  `name_teacher` blob NOT NULL COMMENT '教師作答名字',
  `question_student` int NOT NULL COMMENT '學生作答題數',
  `number_student` int NOT NULL COMMENT '學生作答次數',
  `people_student` int NOT NULL COMMENT '學生作答人數',
  `name_student` blob NOT NULL COMMENT '學生作答名字',
  `time_teacher` int NOT NULL,
  `time_student` int NOT NULL,
  `datetime_log` varchar(30) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `report_learningeffect_node`
--

CREATE TABLE `report_learningeffect_node` (
  `sn` int NOT NULL,
  `organization_id` varchar(10) DEFAULT NULL COMMENT '學校代號',
  `city_name` varchar(10) NOT NULL,
  `postcode` varchar(10) NOT NULL,
  `city_area` varchar(10) NOT NULL,
  `name` varchar(50) NOT NULL,
  `type` varchar(4) NOT NULL,
  `map_sn` int NOT NULL,
  `grade` int NOT NULL,
  `class` int NOT NULL,
  `passnode_teacher` int NOT NULL COMMENT '教師精通節點',
  `failnode_teacher` int NOT NULL COMMENT '教師未精通節點',
  `people_teacher` int NOT NULL COMMENT '教師測驗人數',
  `name_teacher` blob NOT NULL COMMENT '教師測驗名字',
  `passnode_student` int NOT NULL COMMENT '學生精通節點',
  `failnode_student` int NOT NULL COMMENT '學生未精通節點',
  `people_student` int NOT NULL COMMENT '學生測驗人數',
  `name_student` blob NOT NULL COMMENT '學生測驗名字',
  `datetime_log` varchar(30) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `report_learningeffect_practice`
--

CREATE TABLE `report_learningeffect_practice` (
  `sn` int NOT NULL,
  `organization_id` varchar(10) DEFAULT NULL COMMENT '學校代號',
  `city_name` varchar(10) NOT NULL,
  `postcode` varchar(10) NOT NULL,
  `city_area` varchar(10) NOT NULL,
  `name` varchar(50) NOT NULL,
  `type` varchar(4) NOT NULL,
  `map_sn` int NOT NULL,
  `grade` int NOT NULL,
  `class` int NOT NULL,
  `question_teacher` int NOT NULL COMMENT '教師作答題數',
  `number_teacher` int NOT NULL COMMENT '教師作答次數',
  `people_teacher` int NOT NULL COMMENT '教師作答人數',
  `name_teacher` blob NOT NULL COMMENT '教師作答名字',
  `question_student` int NOT NULL COMMENT '學生作答題數',
  `number_student` int NOT NULL COMMENT '學生作答次數',
  `people_student` int NOT NULL COMMENT '學生作答人數',
  `name_student` blob NOT NULL COMMENT '學生作答名字',
  `time_teacher` int NOT NULL,
  `time_student` int NOT NULL,
  `datetime_log` varchar(30) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `report_learningeffect_robot`
--

CREATE TABLE `report_learningeffect_robot` (
  `sn` int NOT NULL,
  `organization_id` varchar(10) DEFAULT NULL COMMENT '學校代號',
  `city_name` varchar(10) NOT NULL,
  `postcode` varchar(10) NOT NULL,
  `city_area` varchar(10) NOT NULL,
  `name` varchar(50) NOT NULL,
  `type` varchar(4) NOT NULL,
  `robot_type` tinyint NOT NULL,
  `map_sn` int NOT NULL,
  `grade` int NOT NULL,
  `class` int NOT NULL,
  `number_teacher` int NOT NULL COMMENT '教師作答次數',
  `people_teacher` int NOT NULL COMMENT '教師作答人數',
  `name_teacher` blob NOT NULL COMMENT '教師作答名字',
  `number_student` int NOT NULL COMMENT '學生作答次數',
  `people_student` int NOT NULL COMMENT '學生作答人數',
  `name_student` blob NOT NULL COMMENT '學生作答名字',
  `time_teacher` int NOT NULL,
  `time_student` int NOT NULL,
  `datetime_log` varchar(30) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `report_learningeffect_unit`
--

CREATE TABLE `report_learningeffect_unit` (
  `sn` int NOT NULL,
  `organization_id` varchar(10) DEFAULT NULL COMMENT '學校代號',
  `city_name` varchar(10) NOT NULL,
  `postcode` varchar(10) NOT NULL,
  `city_area` varchar(10) NOT NULL,
  `name` varchar(50) NOT NULL,
  `type` varchar(4) NOT NULL,
  `map_sn` int NOT NULL,
  `grade` int NOT NULL,
  `class` int NOT NULL,
  `question_teacher` int NOT NULL COMMENT '教師作答題數',
  `number_teacher` int NOT NULL COMMENT '教師作答次數',
  `people_teacher` int NOT NULL COMMENT '教師作答人數',
  `name_teacher` blob NOT NULL COMMENT '教師作答名字',
  `question_student` int NOT NULL COMMENT '學生作答題數',
  `number_student` int NOT NULL COMMENT '學生作答次數',
  `people_student` int NOT NULL COMMENT '學生作答人數',
  `name_student` blob NOT NULL COMMENT '學生作答名字',
  `time_teacher` int NOT NULL,
  `time_student` int NOT NULL,
  `datetime_log` varchar(30) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `report_learningeffect_video`
--

CREATE TABLE `report_learningeffect_video` (
  `sn` int NOT NULL,
  `organization_id` varchar(10) DEFAULT NULL COMMENT '學校代號',
  `city_name` varchar(10) NOT NULL,
  `postcode` varchar(10) NOT NULL,
  `city_area` varchar(10) NOT NULL,
  `name` varchar(50) NOT NULL,
  `type` varchar(4) NOT NULL,
  `map_sn` int NOT NULL,
  `grade` int NOT NULL,
  `class` int NOT NULL,
  `time_teacher` int NOT NULL COMMENT '教師觀看時間',
  `number_teacher` int NOT NULL COMMENT '教師觀看次數',
  `people_teacher` int NOT NULL COMMENT '教師觀人數',
  `name_teacher` blob NOT NULL COMMENT '教師觀看名字',
  `time_student` int NOT NULL COMMENT '學生觀看時間',
  `number_student` int NOT NULL COMMENT '學生觀看次數',
  `people_student` int NOT NULL COMMENT '學生觀看人數',
  `name_student` blob NOT NULL COMMENT '學生觀看名字',
  `datetime_log` varchar(30) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `report_workshop_effect`
--

CREATE TABLE `report_workshop_effect` (
  `sn` int NOT NULL,
  `organization_id` varchar(10) NOT NULL,
  `processing_date` date NOT NULL,
  `teacher_name` varchar(100) NOT NULL,
  `feedback` varchar(500) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `resource_annex`
--

CREATE TABLE `resource_annex` (
  `id` int NOT NULL COMMENT '流水號',
  `resource_id` int NOT NULL COMMENT '資源編號=>課程包資源清單(resource_id)	',
  `annex_id` int NOT NULL COMMENT '附件檔案編號=>附件檔案(annex_id)',
  `status` int NOT NULL COMMENT '狀態[1：啟用、2：停用]',
  `deleted_at` datetime NOT NULL COMMENT '刪除時間',
  `created_at` datetime NOT NULL COMMENT '建立時間',
  `updated_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `resource_choose_hint`
--

CREATE TABLE `resource_choose_hint` (
  `id` int NOT NULL COMMENT '流水號',
  `option_id` int NOT NULL COMMENT '選項編號',
  `hint` varchar(50) NOT NULL COMMENT '選擇錯誤提示訊息',
  `sort` int NOT NULL COMMENT '第幾次選錯',
  `status` int NOT NULL COMMENT '狀態[1：啟用、2：停用]',
  `deleted_at` datetime NOT NULL COMMENT '刪除時間 ',
  `created_at` datetime NOT NULL COMMENT '建立時間 ',
  `updated_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新時間 '
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `resource_exam_annex`
--

CREATE TABLE `resource_exam_annex` (
  `id` int NOT NULL COMMENT '流水號',
  `quiz_id` int NOT NULL COMMENT '試題編號',
  `annex_id` int NOT NULL COMMENT '附件檔案編號=>附件檔案(annex_id) ',
  `status` int NOT NULL COMMENT '狀態[1：啟用、2：停用] ',
  `deleted_at` datetime NOT NULL COMMENT '刪除時間 ',
  `created_at` datetime NOT NULL COMMENT '建立時間 ',
  `updated_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新時間 '
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `resource_exam_filling_answer`
--

CREATE TABLE `resource_exam_filling_answer` (
  `id` int NOT NULL,
  `filling_id` int NOT NULL,
  `answer` varchar(50) NOT NULL,
  `sort` int NOT NULL,
  `status` int NOT NULL,
  `deleted_at` datetime NOT NULL,
  `created_at` datetime NOT NULL,
  `updated_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `resource_filling_answer`
--

CREATE TABLE `resource_filling_answer` (
  `id` int NOT NULL COMMENT '流水號',
  `filling_id` int NOT NULL COMMENT '問答題標號=>課程包填答資源(id)',
  `answer` varchar(50) NOT NULL COMMENT '答案',
  `sort` int NOT NULL COMMENT '答案順序',
  `status` int NOT NULL COMMENT '狀態[1：啟用、2：停用] ',
  `deleted_at` datetime NOT NULL COMMENT '刪除時間 ',
  `created_at` datetime NOT NULL COMMENT '建立時間 ',
  `updated_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '更新時間 '
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `robot_config`
--

CREATE TABLE `robot_config` (
  `sn` int NOT NULL,
  `name` varchar(100) NOT NULL COMMENT '應用名',
  `value` text CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci COMMENT '數值',
  `created_at` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '建立時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `role_carrer_info`
--

CREATE TABLE `role_carrer_info` (
  `carrer_id` int NOT NULL,
  `carrer_name` varchar(10) NOT NULL COMMENT '職業名稱',
  `carrer_equipment` varchar(10) NOT NULL COMMENT '職業裝備'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='角色職業資訊表';

-- --------------------------------------------------------

--
-- 資料表結構 `role_coin_history`
--

CREATE TABLE `role_coin_history` (
  `sn` int UNSIGNED NOT NULL,
  `coin_type` tinyint UNSIGNED NOT NULL COMMENT '代幣類型 (1:系統幣, 2:教師幣)',
  `operator` varchar(60) NOT NULL COMMENT '操作人',
  `action` varchar(10) NOT NULL COMMENT '代幣操作動作',
  `coin` int UNSIGNED NOT NULL COMMENT '代幣數量',
  `student_id` varchar(60) NOT NULL,
  `rule_name` varchar(50) NOT NULL COMMENT '代幣操作名目',
  `memo` varchar(500) NOT NULL,
  `mission_sn` int DEFAULT NULL,
  `video_note_sn` int DEFAULT NULL,
  `timestamp` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `role_coin_history_20230316`
--

CREATE TABLE `role_coin_history_20230316` (
  `sn` int UNSIGNED NOT NULL,
  `coin_type` tinyint UNSIGNED NOT NULL COMMENT '代幣類型 (1:系統幣, 2:教師幣)',
  `operator` varchar(60) NOT NULL COMMENT '操作人',
  `action` varchar(10) NOT NULL COMMENT '代幣操作動作',
  `coin` int UNSIGNED NOT NULL COMMENT '代幣數量',
  `student_id` varchar(60) NOT NULL,
  `rule_name` varchar(50) NOT NULL COMMENT '代幣操作名目',
  `memo` varchar(500) NOT NULL,
  `mission_sn` int DEFAULT NULL,
  `video_note_sn` int DEFAULT NULL,
  `timestamp` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `role_coin_rule`
--

CREATE TABLE `role_coin_rule` (
  `rule_sn` int NOT NULL,
  `user_id` varchar(60) NOT NULL COMMENT '老師id',
  `rule_name` varchar(20) NOT NULL COMMENT '自定義獎勵項目',
  `rule_type` varchar(10) NOT NULL COMMENT '是增加代幣or扣除代幣',
  `rule_point` int NOT NULL COMMENT '代幣數值',
  `use_count` int NOT NULL COMMENT '使用次數',
  `display` tinyint NOT NULL DEFAULT '1' COMMENT '1=顯示；0=不顯示'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `role_coin_total`
--

CREATE TABLE `role_coin_total` (
  `coin_sn` int NOT NULL,
  `teacher_id` varchar(60) NOT NULL COMMENT 'sys:系統幣',
  `student_id` varchar(60) NOT NULL,
  `coin` int UNSIGNED NOT NULL DEFAULT '0' COMMENT '現有代幣數量',
  `accumulation_coin` int UNSIGNED NOT NULL DEFAULT '0' COMMENT '歷史獲得代幣數量',
  `updatatime` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='記錄學生對老師的代幣現有及獲得代幣數量';

-- --------------------------------------------------------

--
-- 資料表結構 `role_food_item`
--

CREATE TABLE `role_food_item` (
  `food_id` int NOT NULL COMMENT '流水號',
  `food_type` varchar(50) NOT NULL COMMENT '食物代碼',
  `food_name` varchar(50) NOT NULL COMMENT '食物名稱',
  `food_effect` tinyint NOT NULL COMMENT '該食物的效用，ex：體力值+10',
  `coin_num` int NOT NULL COMMENT '兌換的代幣數'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='食物類別';

-- --------------------------------------------------------

--
-- 資料表結構 `role_info`
--

CREATE TABLE `role_info` (
  `sn` int NOT NULL COMMENT '流水號',
  `organization_id` varchar(10) DEFAULT NULL COMMENT '學校代碼',
  `stud_id` varchar(60) NOT NULL COMMENT '學生id',
  `role_head` varchar(10) NOT NULL COMMENT '頭貼名稱',
  `carrer_id` int DEFAULT '0' COMMENT '職業類別: 劍士=1弓箭手=2魔法師=3',
  `coin` int NOT NULL DEFAULT '0' COMMENT '已有的代幣數量',
  `accumulation_coin` int NOT NULL DEFAULT '0' COMMENT '總累積代幣數',
  `body_weight` int NOT NULL DEFAULT '0' COMMENT '角色體重',
  `body_height` int NOT NULL DEFAULT '0' COMMENT '角色身高',
  `health_points` int NOT NULL DEFAULT '0' COMMENT '角色體力值',
  `hat` int DEFAULT NULL COMMENT '道具帽子',
  `clothes` int DEFAULT NULL COMMENT '道具衣服',
  `shoes` int DEFAULT NULL COMMENT '道具鞋子',
  `carrer_equipment` int DEFAULT NULL COMMENT '職業裝備',
  `create_time` datetime NOT NULL COMMENT '角色建立時間',
  `update_date` datetime DEFAULT NULL COMMENT '角色更新時間',
  `give_time` datetime NOT NULL COMMENT '贈送代幣的時間',
  `give_count` int DEFAULT '3' COMMENT '贈送代幣剩餘次數'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='個人角色資訊';

-- --------------------------------------------------------

--
-- 資料表結構 `role_prop_item`
--

CREATE TABLE `role_prop_item` (
  `prop_id` int NOT NULL COMMENT '流水號',
  `prop_type` varchar(50) NOT NULL COMMENT '道具代碼',
  `prop_name` varchar(50) NOT NULL COMMENT '道具名稱',
  `prop_effect` tinyint NOT NULL COMMENT '道具的效用',
  `coin_num` int NOT NULL COMMENT '兌換的代幣數'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='道具類別';

-- --------------------------------------------------------

--
-- 資料表結構 `role_reward_way`
--

CREATE TABLE `role_reward_way` (
  `reward_sn` smallint NOT NULL COMMENT '流水號',
  `program_id` varchar(12) NOT NULL COMMENT '程式代號',
  `program_name` varchar(30) NOT NULL COMMENT '學習方式名稱',
  `program_action` mediumtext NOT NULL COMMENT '學習動作',
  `action_count` smallint NOT NULL COMMENT '執行次數',
  `judge_correct` smallint NOT NULL COMMENT '是否要判斷答對',
  `is_correct` smallint NOT NULL COMMENT '是否答對',
  `reward_coin_num` smallint NOT NULL COMMENT '要給予的代幣數量',
  `reward_percent` smallint NOT NULL COMMENT '要給予的數量比例%'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='獎勵方式';

-- --------------------------------------------------------

--
-- 資料表結構 `role_reword_card_item`
--

CREATE TABLE `role_reword_card_item` (
  `reward_card_id` int NOT NULL COMMENT '流水號',
  `reward_card_type` varchar(50) NOT NULL COMMENT '獎勵卡代碼',
  `reword_card_name` varchar(50) NOT NULL COMMENT '獎勵卡名稱',
  `reword_card_effect` tinyint NOT NULL COMMENT '獎勵卡的效用',
  `coin_num` int NOT NULL COMMENT '兌換的代幣數'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='獎勵卡類別';

-- --------------------------------------------------------

--
-- 資料表結構 `school_class_info`
--

CREATE TABLE `school_class_info` (
  `class_sn` int UNSIGNED NOT NULL COMMENT '學校班級資訊流水號',
  `semester` int UNSIGNED NOT NULL COMMENT '學期',
  `organization_id` varchar(10) NOT NULL COMMENT '學校代號',
  `grade` int UNSIGNED NOT NULL COMMENT '年級',
  `high_school_dept_mapping_sn` int UNSIGNED DEFAULT NULL COMMENT '高中職科別對應流水號',
  `class_name` varchar(10) NOT NULL COMMENT '班級名稱(使用者自定義)',
  `class_number` int UNSIGNED NOT NULL COMMENT '班級編號(內部用)',
  `updater` varchar(60) DEFAULT NULL COMMENT '更新者',
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '建立時間',
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='學校班級資訊';

-- --------------------------------------------------------

--
-- 資料表結構 `seme_grad_student`
--

CREATE TABLE `seme_grad_student` (
  `sn` int UNSIGNED NOT NULL,
  `seme_year` varchar(6) NOT NULL DEFAULT '' COMMENT '學年度',
  `organization_id` varchar(10) NOT NULL COMMENT '學校代號',
  `stud_id` varchar(60) NOT NULL COMMENT '學生帳號',
  `grade` tinyint UNSIGNED NOT NULL COMMENT '年級',
  `class` tinyint UNSIGNED NOT NULL COMMENT '班級',
  `seme_num` tinyint UNSIGNED DEFAULT NULL COMMENT '學生座號',
  `next_organization_id` varchar(10) DEFAULT NULL COMMENT '新學校代號',
  `uid` int UNSIGNED NOT NULL DEFAULT '0' COMMENT '學生帳號之流水號'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='每學年畢業學生';

-- --------------------------------------------------------

--
-- 資料表結構 `seme_group`
--

CREATE TABLE `seme_group` (
  `group_id` int NOT NULL COMMENT '小組id',
  `group_nm` varchar(50) NOT NULL COMMENT '小組名稱',
  `teacher_id` varchar(60) DEFAULT NULL COMMENT '老師id',
  `create_date` datetime NOT NULL COMMENT '建立日期',
  `is_used` smallint NOT NULL DEFAULT '1' COMMENT '1=未刪除；0=已刪除'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='學習小組';

-- --------------------------------------------------------

--
-- 資料表結構 `seme_publisher`
--

CREATE TABLE `seme_publisher` (
  `sn` int NOT NULL COMMENT '流水號',
  `sems` varchar(4) NOT NULL COMMENT '學年度學期',
  `organization_id` varchar(10) NOT NULL COMMENT '學校代號',
  `subject_id` smallint UNSIGNED NOT NULL COMMENT '科目',
  `publisher_id` smallint UNSIGNED NOT NULL COMMENT '版本',
  `grade` int NOT NULL COMMENT '年級',
  `create_id` varchar(60) DEFAULT NULL,
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='各學期各學校之預設版本';

-- --------------------------------------------------------

--
-- 資料表結構 `seme_publisher_tmp`
--

CREATE TABLE `seme_publisher_tmp` (
  `sn` int NOT NULL,
  `sems` varchar(4) NOT NULL COMMENT '學年度學期',
  `organization_id` varchar(10) NOT NULL COMMENT '學校代號',
  `subject_id` smallint UNSIGNED NOT NULL COMMENT '科目',
  `publisher_id` smallint UNSIGNED NOT NULL COMMENT '版本',
  `grade` int NOT NULL COMMENT '年級'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='學校學期科目版本暫存用';

-- --------------------------------------------------------

--
-- 資料表結構 `seme_student`
--

CREATE TABLE `seme_student` (
  `sn` int UNSIGNED NOT NULL,
  `seme_year_seme` varchar(6) NOT NULL DEFAULT '' COMMENT '學年度學期',
  `organization_id` varchar(10) NOT NULL COMMENT '學校代號',
  `stud_id` varchar(60) NOT NULL COMMENT '學生帳號',
  `grade` int UNSIGNED NOT NULL COMMENT '年級',
  `class` int UNSIGNED NOT NULL COMMENT '班級',
  `seme_class_name` varchar(10) DEFAULT NULL COMMENT '班級中文名稱',
  `seme_num` tinyint UNSIGNED DEFAULT NULL COMMENT '學生座號',
  `uid` int UNSIGNED NOT NULL DEFAULT '0' COMMENT '學生帳號之流水號'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='學生學期';

-- --------------------------------------------------------

--
-- 資料表結構 `seme_student_rebackschool`
--

CREATE TABLE `seme_student_rebackschool` (
  `sn` int UNSIGNED NOT NULL,
  `seme_year` varchar(6) NOT NULL DEFAULT '' COMMENT '學年度',
  `organization_id` varchar(10) NOT NULL COMMENT '學校代號',
  `stud_id` varchar(60) NOT NULL COMMENT '學生帳號',
  `grade` tinyint UNSIGNED NOT NULL COMMENT '年級',
  `class` tinyint UNSIGNED NOT NULL COMMENT '班級',
  `seme_num` tinyint UNSIGNED DEFAULT NULL COMMENT '學生座號',
  `next_organization_id` varchar(10) DEFAULT NULL COMMENT '新學校代號',
  `uid` int UNSIGNED NOT NULL DEFAULT '0' COMMENT '學生帳號之流水號',
  `update_id` varchar(60) DEFAULT NULL,
  `update_time` datetime NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='每學年畢業學生';

-- --------------------------------------------------------

--
-- 資料表結構 `seme_student_tmp`
--

CREATE TABLE `seme_student_tmp` (
  `sn` int UNSIGNED NOT NULL,
  `seme_year_seme` varchar(6) NOT NULL DEFAULT '' COMMENT '學年度學期',
  `organization_id` varchar(10) NOT NULL COMMENT '學校代號',
  `stud_id` varchar(60) NOT NULL COMMENT '學生帳號',
  `grade` tinyint UNSIGNED NOT NULL COMMENT '年級',
  `class` tinyint UNSIGNED NOT NULL COMMENT '班級',
  `seme_class_name` varchar(10) DEFAULT NULL COMMENT '班級中文名稱',
  `seme_num` tinyint UNSIGNED DEFAULT NULL COMMENT '學生座號',
  `uid` int UNSIGNED NOT NULL DEFAULT '0' COMMENT '學生帳號之流水號'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='學生學期';

-- --------------------------------------------------------

--
-- 資料表結構 `seme_teacher_subject`
--

CREATE TABLE `seme_teacher_subject` (
  `sn` int NOT NULL,
  `seme_year_seme` varchar(6) NOT NULL DEFAULT '' COMMENT '學年度學期',
  `organization_id` varchar(10) NOT NULL COMMENT '學校代號',
  `teacher_id` varchar(60) NOT NULL DEFAULT '' COMMENT '教師帳號',
  `grade` int UNSIGNED NOT NULL COMMENT '任教年級',
  `class` int UNSIGNED NOT NULL COMMENT '任教班級',
  `subject_id` smallint UNSIGNED DEFAULT NULL COMMENT '任教科目(已棄用)改用subject_mapping對應',
  `subject_mapping` int NOT NULL,
  `uid` int UNSIGNED NOT NULL DEFAULT '0' COMMENT '已棄用'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='教師學期科目';

-- --------------------------------------------------------

--
-- 資料表結構 `seme_teacher_subject_mapping`
--

CREATE TABLE `seme_teacher_subject_mapping` (
  `mapping_sn` int NOT NULL,
  `organization_id` varchar(10) DEFAULT NULL COMMENT '學校代碼(NULL為因材網預設全站共用)',
  `subject_name` varchar(30) NOT NULL COMMENT '科目名稱'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `seme_teacher_subject_tmp`
--

CREATE TABLE `seme_teacher_subject_tmp` (
  `sn` int NOT NULL,
  `seme_year_seme` varchar(6) NOT NULL DEFAULT '' COMMENT '學年度學期',
  `organization_id` varchar(10) NOT NULL COMMENT '學校代號',
  `teacher_id` varchar(60) NOT NULL COMMENT '教師帳號',
  `grade` tinyint UNSIGNED NOT NULL COMMENT '任教年級',
  `class` tinyint UNSIGNED NOT NULL COMMENT '任教班級',
  `subject_id` smallint UNSIGNED NOT NULL COMMENT '任教科目',
  `subject_mapping` int NOT NULL,
  `uid` int UNSIGNED NOT NULL DEFAULT '0' COMMENT '教師帳號之流水號'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='教師學期科目暫存用';

-- --------------------------------------------------------

--
-- 資料表結構 `seme_teacher_subject_tmp_0208`
--

CREATE TABLE `seme_teacher_subject_tmp_0208` (
  `sn` int NOT NULL,
  `seme_year_seme` varchar(6) NOT NULL DEFAULT '' COMMENT '學年度學期',
  `organization_id` varchar(7) NOT NULL COMMENT '學校代號',
  `teacher_id` varchar(50) NOT NULL DEFAULT '' COMMENT '教師帳號',
  `grade` tinyint UNSIGNED NOT NULL COMMENT '任教年級',
  `class` tinyint UNSIGNED NOT NULL COMMENT '任教班級',
  `subject_id` tinyint UNSIGNED NOT NULL COMMENT '任教科目',
  `uid` int UNSIGNED NOT NULL DEFAULT '0' COMMENT '教師帳號之流水號'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='教師學期科目暫存用';

-- --------------------------------------------------------

--
-- 資料表結構 `srl_calendar`
--

CREATE TABLE `srl_calendar` (
  `sn` int NOT NULL,
  `user_id` varchar(60) NOT NULL COMMENT '學生帳號',
  `events` text NOT NULL COMMENT '行程',
  `start_time` datetime NOT NULL COMMENT '開始時間',
  `end_time` datetime NOT NULL COMMENT '結束時間',
  `all_day` tinyint(1) NOT NULL DEFAULT '0' COMMENT '全天(是:1；否:0)',
  `mission_sn` int DEFAULT NULL COMMENT '任務sn',
  `is_done` tinyint DEFAULT '0' COMMENT '0未完成 1完成',
  `is_delete` tinyint(1) NOT NULL DEFAULT '0' COMMENT '使用者是否刪除',
  `eventColor` tinyint UNSIGNED NOT NULL COMMENT '行事曆顏色'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `subject`
--

CREATE TABLE `subject` (
  `subject_id` smallint UNSIGNED NOT NULL COMMENT '科目代號',
  `name` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '科目名稱',
  `display` tinyint(1) NOT NULL DEFAULT '1' COMMENT '是否顯示',
  `subject_groupings_id` tinyint UNSIGNED DEFAULT NULL COMMENT '科別代號',
  `subject_optgroups_id` int DEFAULT NULL COMMENT '科目下拉選單分類id'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='學科名稱';

-- --------------------------------------------------------

--
-- 資料表結構 `subject_0903`
--

CREATE TABLE `subject_0903` (
  `subject_id` smallint UNSIGNED NOT NULL COMMENT '科目代號',
  `name` varchar(20) NOT NULL COMMENT '科目名稱',
  `display` tinyint(1) NOT NULL DEFAULT '1' COMMENT '是否顯示',
  `subject_groupings_id` tinyint UNSIGNED DEFAULT NULL COMMENT '科別代號'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='學科名稱';

-- --------------------------------------------------------

--
-- 資料表結構 `subject_acnt`
--

CREATE TABLE `subject_acnt` (
  `sn` int UNSIGNED NOT NULL,
  `subject` varchar(6) NOT NULL,
  `acnt` varchar(20) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `subject_bk`
--

CREATE TABLE `subject_bk` (
  `subject_id` tinyint UNSIGNED NOT NULL COMMENT '科目代號',
  `name` varchar(20) NOT NULL COMMENT '科目名稱',
  `display` tinyint(1) NOT NULL DEFAULT '1' COMMENT '是否顯示'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='學科名稱';

-- --------------------------------------------------------

--
-- 資料表結構 `subject_groupings`
--

CREATE TABLE `subject_groupings` (
  `subject_groupings_id` tinyint UNSIGNED NOT NULL COMMENT '科別代號',
  `subject_groupings_name` varchar(20) NOT NULL COMMENT '科別名稱',
  `subject_id` smallint UNSIGNED DEFAULT NULL COMMENT '預設科目代號'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `subject_optgroups`
--

CREATE TABLE `subject_optgroups` (
  `subject_optgroups_id` int NOT NULL,
  `name` varchar(20) NOT NULL COMMENT '分類名稱',
  `display` int NOT NULL DEFAULT '0' COMMENT '顯示'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `sub_subject`
--

CREATE TABLE `sub_subject` (
  `sub_subject_id` tinyint UNSIGNED NOT NULL COMMENT '科目代號',
  `sub_subject_name` varchar(20) NOT NULL COMMENT '科目名稱',
  `map_sn` int NOT NULL COMMENT '星空圖代號',
  `subject_id` smallint UNSIGNED NOT NULL COMMENT '主科目代號'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='子學科名稱';

-- --------------------------------------------------------

--
-- 資料表結構 `system_user`
--

CREATE TABLE `system_user` (
  `sn` tinyint UNSIGNED NOT NULL,
  `user_id` varchar(60) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `talper_custom_agent`
--

CREATE TABLE `talper_custom_agent` (
  `sn` int NOT NULL COMMENT 'agent_sn',
  `agent_name` varchar(150) NOT NULL COMMENT '代理人名稱',
  `user_id` varchar(60) NOT NULL COMMENT '建立人id',
  `agent_describe` text COMMENT '代理人用途說明',
  `greeting_message` text COMMENT '代理人問候訊息',
  `agent_prompt` text COMMENT '代理人prompt',
  `model_name` varchar(100) NOT NULL COMMENT '模型名稱',
  `agent_files` text COMMENT '代理人上傳的檔案(RAG使用)',
  `creation_time` datetime NOT NULL COMMENT '建立時間',
  `updated_time` datetime NOT NULL ON UPDATE CURRENT_TIMESTAMP COMMENT '最後更新時間',
  `status` varchar(20) DEFAULT 'draft' COMMENT 'Agent 狀態: draft=草稿, created=已建立, assigned=已指派'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `talper_custom_agent_files`
--

CREATE TABLE `talper_custom_agent_files` (
  `sn` int NOT NULL,
  `agent_sn` int NOT NULL COMMENT 'talper_custom_agent.sn',
  `file_name` varchar(150) NOT NULL COMMENT '上傳檔案名稱',
  `file_path` text NOT NULL COMMENT '上傳路徑',
  `upload_time` datetime NOT NULL COMMENT '上傳時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `talper_custom_agent_test_history`
--

CREATE TABLE `talper_custom_agent_test_history` (
  `sn` int NOT NULL,
  `agent_sn` int NOT NULL COMMENT 'talper_custom_agent.sn',
  `role` varchar(100) NOT NULL COMMENT '對話時的角色 ( assistant、user、system )',
  `content` text NOT NULL COMMENT '對話內容',
  `api_return_data` text NOT NULL COMMENT 'api回傳的資料',
  `error_code` varchar(100) NOT NULL,
  `error_message` text NOT NULL,
  `creation_time` datetime NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `teacher_class`
--

CREATE TABLE `teacher_class` (
  `teacher_class_sn` int NOT NULL,
  `organization_id` varchar(10) DEFAULT NULL,
  `seme` smallint NOT NULL,
  `class_num` varchar(20) NOT NULL COMMENT '班級編號(學校代碼-學年度-以學校計數的流水號)',
  `class_name` text NOT NULL,
  `teacher_id` varchar(60) DEFAULT NULL COMMENT '指派教師',
  `pass_word` varchar(4) NOT NULL COMMENT '邀請碼',
  `create_id` varchar(60) DEFAULT NULL,
  `status` int NOT NULL DEFAULT '1' COMMENT '是否顯示(0=隱藏；1=顯示)',
  `create_date` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '建立時間',
  `update_time` datetime NOT NULL ON UPDATE CURRENT_TIMESTAMP COMMENT '更新時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `teacher_class_member`
--

CREATE TABLE `teacher_class_member` (
  `teacher_class_member_sn` int NOT NULL,
  `class_sn` int NOT NULL COMMENT '對應teacher_class編號',
  `student_id` varchar(60) NOT NULL COMMENT '學生ID',
  `teacher_agree` int NOT NULL DEFAULT '0' COMMENT '學生自行申請：0 同意：1',
  `add_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `update_time` datetime NOT NULL ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `test123`
--

CREATE TABLE `test123` (
  `t1` int NOT NULL,
  `t2` int NOT NULL,
  `t3` varchar(100) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `triangle_class_conclusion`
--

CREATE TABLE `triangle_class_conclusion` (
  `conclusion_sn` int NOT NULL,
  `thought_id` int NOT NULL,
  `group_id` int NOT NULL,
  `agree` tinyint(1) NOT NULL COMMENT '1:同意；0不同意'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `triangle_conclusion`
--

CREATE TABLE `triangle_conclusion` (
  `conclusion_id` int NOT NULL,
  `mission_sn` int NOT NULL,
  `question_sn` int NOT NULL,
  `sub_question_sn` int NOT NULL COMMENT '對應的小題 1 ~ 4',
  `user_id` varchar(60) NOT NULL,
  `thought` text NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `triangle_conclusion_proofs`
--

CREATE TABLE `triangle_conclusion_proofs` (
  `proof_sn` int NOT NULL,
  `conclusion_id` int NOT NULL,
  `content` text NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `triangle_proof`
--

CREATE TABLE `triangle_proof` (
  `proof_id` int NOT NULL,
  `mission_sn` int NOT NULL,
  `question_sn` int NOT NULL,
  `user_id` varchar(60) NOT NULL,
  `file` text NOT NULL COMMENT '證據圖檔',
  `depiction` text NOT NULL COMMENT '證據敘述',
  `other_detail` json NOT NULL COMMENT '其他資訊'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `triangle_thought`
--

CREATE TABLE `triangle_thought` (
  `thought_id` int NOT NULL,
  `mission_sn` int NOT NULL,
  `question_sn` int NOT NULL,
  `depiction` text NOT NULL COMMENT '猜想內容',
  `user_id` varchar(60) NOT NULL,
  `personal_selected` tinyint(1) NOT NULL DEFAULT '0' COMMENT '個人是否送出',
  `group_selected` tinyint(1) NOT NULL DEFAULT '0' COMMENT '小組是否送出',
  `group_reserved` tinyint(1) NOT NULL DEFAULT '0' COMMENT '小組是否保留',
  `class_agree` json DEFAULT NULL COMMENT '班級是否同意成為結論'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `triangle_thought_comment`
--

CREATE TABLE `triangle_thought_comment` (
  `comment_sn` int NOT NULL,
  `thought_id` int NOT NULL,
  `comment_user` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '做出評論的個人',
  `comment_group` int DEFAULT NULL COMMENT '做出評論的小組',
  `comment` text NOT NULL COMMENT '評論內容',
  `type` tinyint NOT NULL COMMENT '1:支持；2反對',
  `reply_sn` int DEFAULT NULL COMMENT '回覆對象',
  `datetime` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `triangle_thought_proofs`
--

CREATE TABLE `triangle_thought_proofs` (
  `sn` int NOT NULL,
  `thought_id` int NOT NULL,
  `proof_id` int NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `users_pwd`
--

CREATE TABLE `users_pwd` (
  `id` int NOT NULL COMMENT '密碼流水號',
  `aid` varchar(60) NOT NULL COMMENT '會員編號',
  `pwd` varchar(255) NOT NULL COMMENT '密碼',
  `status` int NOT NULL COMMENT '狀態[1：啟用、2：停用]',
  `deleted_at` datetime NOT NULL COMMENT '刪除時間 ',
  `created_at` datetime NOT NULL COMMENT '建立時間 ',
  `updated_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新時間 '
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `user_access`
--

CREATE TABLE `user_access` (
  `access_level` int NOT NULL COMMENT '存取等級',
  `access_title` varchar(20) NOT NULL COMMENT '存取等級中文名稱',
  `access_en_title` varchar(20) NOT NULL COMMENT '存取等級英文名稱',
  `remark` mediumtext COMMENT '註解',
  `block_id` int NOT NULL COMMENT '功能表代號',
  `block_content` mediumtext NOT NULL COMMENT '功能表內容',
  `module_id` varchar(255) NOT NULL COMMENT '可使用之模組代號',
  `time_created` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `time_updated` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='使用者存取等級';

-- --------------------------------------------------------

--
-- 資料表結構 `user_account_change`
--

CREATE TABLE `user_account_change` (
  `sn` int NOT NULL,
  `new_user_account` varchar(60) NOT NULL,
  `ori_user_account` varchar(60) NOT NULL,
  `update_date` datetime NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `user_account_history`
--

CREATE TABLE `user_account_history` (
  `sn` int NOT NULL COMMENT '流水號',
  `user_id` varchar(60) NOT NULL COMMENT '使用者id',
  `new_user_account` varchar(60) NOT NULL COMMENT '新帳號',
  `ori_user_account` varchar(60) NOT NULL COMMENT '原始帳號',
  `ori_access_level` int NOT NULL COMMENT '原始權限',
  `ori_organization_id` varchar(10) DEFAULT NULL COMMENT '原始org_id',
  `update_date` datetime NOT NULL COMMENT '異動日期'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='帳號異動紀錄';

-- --------------------------------------------------------

--
-- 資料表結構 `user_activity`
--

CREATE TABLE `user_activity` (
  `sn` int UNSIGNED NOT NULL,
  `date` date NOT NULL,
  `organization_id` varchar(10) NOT NULL,
  `idle_seconds` bigint UNSIGNED DEFAULT '0' COMMENT '使用時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `user_combine_history`
--

CREATE TABLE `user_combine_history` (
  `sn` int NOT NULL,
  `user_id` varchar(60) NOT NULL COMMENT '正在使用的帳號(被綁定)',
  `connect_user_id` varchar(60) DEFAULT NULL COMMENT '綁定的帳號 (若為空則為第一次綁定)',
  `datetime` datetime NOT NULL COMMENT '日期時間',
  `by_user_id` varchar(60) DEFAULT NULL COMMENT '操作者 (通常為user_id)',
  `type` varchar(10) NOT NULL COMMENT '綁定帳號類型'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='帳號綁定紀錄';

-- --------------------------------------------------------

--
-- 資料表結構 `user_entering_interactive_learning_log`
--

CREATE TABLE `user_entering_interactive_learning_log` (
  `sn` bigint UNSIGNED NOT NULL,
  `user_id` varchar(60) NOT NULL,
  `organization_id` varchar(10) DEFAULT NULL,
  `its_sn` int DEFAULT NULL COMMENT '互動式編號，null為對話式',
  `its_type` varchar(20) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT 'physics_interactive物理模擬/concept_its一般互動/mathITSbk對話式',
  `start_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `user_family`
--

CREATE TABLE `user_family` (
  `sn` int NOT NULL COMMENT '流水號',
  `organization_id` varchar(10) DEFAULT NULL COMMENT '學校代碼',
  `user_id` varchar(60) NOT NULL COMMENT '使用者id',
  `fuser_id` varchar(60) NOT NULL COMMENT '家長id',
  `create_date` datetime NOT NULL COMMENT '建立日期',
  `update_date` datetime NOT NULL COMMENT '修改日期',
  `create_userid` varchar(60) DEFAULT NULL COMMENT '建立者id'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='學生家長名單';

-- --------------------------------------------------------

--
-- 資料表結構 `user_feedback`
--

CREATE TABLE `user_feedback` (
  `sn` int NOT NULL,
  `email` varchar(255) NOT NULL COMMENT '電子信箱',
  `phone` varchar(15) DEFAULT NULL,
  `textdata` mediumtext NOT NULL COMMENT '問題描述',
  `chk1` char(5) NOT NULL DEFAULT '0' COMMENT 'check1',
  `create_user` varchar(60) DEFAULT NULL COMMENT '回題回報人',
  `create_date` datetime NOT NULL COMMENT '回報日期',
  `chk2` char(5) DEFAULT '0' COMMENT 'check2',
  `chk3` smallint UNSIGNED DEFAULT NULL COMMENT '科目 subject_id',
  `session_data` longtext NOT NULL COMMENT 'session data',
  `img_path` mediumtext NOT NULL COMMENT '圖片連結',
  `upload_server` varchar(50) DEFAULT NULL COMMENT '在哪一台主機上傳',
  `file_backup` tinyint(1) NOT NULL DEFAULT '0' COMMENT '0:未備份至主機；1已備份至主機',
  `re_statu` int NOT NULL DEFAULT '2' COMMENT '1處理中；2未處理',
  `re_name` varchar(20) DEFAULT NULL,
  `have_read` tinyint(1) NOT NULL DEFAULT '0' COMMENT '0:未讀 1:已讀'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='使用者問題反饋';

-- --------------------------------------------------------

--
-- 資料表結構 `user_games_play_time`
--

CREATE TABLE `user_games_play_time` (
  `sn` int UNSIGNED NOT NULL,
  `api_client_sn` int UNSIGNED NOT NULL,
  `user_id` varchar(60) NOT NULL,
  `user_logintimes` int UNSIGNED NOT NULL,
  `total_time` int UNSIGNED NOT NULL,
  `date` date NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `user_group`
--

CREATE TABLE `user_group` (
  `sn` int NOT NULL COMMENT '流水號',
  `organization_id` varchar(10) NOT NULL COMMENT '學校代碼',
  `user_id` varchar(60) NOT NULL COMMENT '使用者id',
  `group_id` int NOT NULL COMMENT '小組id',
  `role_sn` int DEFAULT NULL COMMENT '小組角色sn',
  `create_date` datetime NOT NULL COMMENT '建立日期',
  `update_date` datetime NOT NULL COMMENT '修改日期',
  `group_leader` int DEFAULT '0' COMMENT '小組長=1; 非組長=0 (同組組長只有一人)'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='學生分組名單';

-- --------------------------------------------------------

--
-- 資料表結構 `user_group_20230316`
--

CREATE TABLE `user_group_20230316` (
  `sn` int NOT NULL COMMENT '流水號',
  `organization_id` varchar(10) NOT NULL COMMENT '學校代碼',
  `user_id` varchar(60) NOT NULL COMMENT '使用者id',
  `group_id` int NOT NULL COMMENT '小組id',
  `role_sn` int DEFAULT NULL COMMENT '小組角色sn',
  `create_date` datetime NOT NULL COMMENT '建立日期',
  `update_date` datetime NOT NULL COMMENT '修改日期',
  `group_leader` int DEFAULT '0' COMMENT '小組長=1; 非組長=0 (同組組長只有一人)'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='學生分組名單';

-- --------------------------------------------------------

--
-- 資料表結構 `user_info`
--

CREATE TABLE `user_info` (
  `uid` int UNSIGNED NOT NULL COMMENT '流水號',
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
  `class_sn` int UNSIGNED DEFAULT NULL COMMENT '學校班級資訊流水號',
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
  `OpenID_update` smallint DEFAULT NULL COMMENT '若年班資料從OpenID更新過就會記錄在此 (學期)	',
  `update_id` varchar(60) DEFAULT NULL COMMENT '更新者ID',
  `check_code` varchar(10) DEFAULT NULL COMMENT '檢查碼',
  `used` int NOT NULL DEFAULT '1' COMMENT '狀態(啟用1/停用0/刪除2)',
  `priori_name` varchar(100) DEFAULT NULL COMMENT '補救教學對應使用',
  `update_date` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新日期'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='使用者基本資料' ROW_FORMAT=COMPACT;

-- --------------------------------------------------------

--
-- 資料表結構 `user_info_temp`
--

CREATE TABLE `user_info_temp` (
  `uid` int UNSIGNED NOT NULL COMMENT '流水號',
  `user_id` varchar(50) NOT NULL COMMENT '登入帳號',
  `access_level` tinyint NOT NULL DEFAULT '1' COMMENT '存取等級',
  `uname` varchar(40) NOT NULL COMMENT '姓名',
  `email` varchar(100) DEFAULT NULL COMMENT '電子郵件',
  `sex` varchar(2) DEFAULT NULL COMMENT '性別',
  `user_regdate` varchar(20) NOT NULL COMMENT '註冊日期',
  `birthday` varchar(14) DEFAULT NULL COMMENT '生日',
  `organization_id` varchar(7) NOT NULL COMMENT '單位(學校)編號',
  `pass` varchar(40) NOT NULL COMMENT 'md5加密密碼',
  `viewpass` varchar(15) DEFAULT NULL COMMENT '使用者密碼',
  `city_code` int NOT NULL DEFAULT '0' COMMENT '城市編號',
  `grade` int NOT NULL DEFAULT '0' COMMENT '年級',
  `class` int NOT NULL DEFAULT '0' COMMENT '班級',
  `hash_guid` varchar(100) DEFAULT NULL COMMENT 'HASH16進制的身份証字號',
  `seme` smallint NOT NULL DEFAULT '0' COMMENT '學年度',
  `Open_Map` int DEFAULT NULL,
  `OpenID` varchar(100) NOT NULL COMMENT 'OpenID URL',
  `OpenID_sub` varchar(100) NOT NULL COMMENT 'sub',
  `update_id` varchar(40) NOT NULL COMMENT '更新者ID',
  `status` varchar(1) NOT NULL DEFAULT '0' COMMENT '審核狀態'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='使用者基本資料' ROW_FORMAT=COMPACT;

-- --------------------------------------------------------

--
-- 資料表結構 `user_login_log`
--

CREATE TABLE `user_login_log` (
  `sn` bigint UNSIGNED NOT NULL,
  `user_id` varchar(60) DEFAULT NULL,
  `organization_id` varchar(10) DEFAULT NULL,
  `loginstamp` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '登入時間',
  `active_time` int DEFAULT NULL COMMENT '停留時間(秒)',
  `login_from` varchar(9) DEFAULT NULL COMMENT '登入方式',
  `ip_address` char(16) NOT NULL DEFAULT '0.0.0.0',
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `user_login_log_status`
--

CREATE TABLE `user_login_log_status` (
  `sn` bigint UNSIGNED NOT NULL,
  `user_id` varchar(60) NOT NULL,
  `loginstamp` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '當下時間',
  `login_status` int DEFAULT NULL COMMENT '登入成功1,失敗0',
  `ip_address` char(16) NOT NULL DEFAULT '0.0.0.0'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `user_map_permission`
--

CREATE TABLE `user_map_permission` (
  `sn` smallint UNSIGNED NOT NULL,
  `user_id` varchar(60) NOT NULL,
  `map_sn` int NOT NULL,
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `updater` varchar(60) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `user_map_permission_log`
--

CREATE TABLE `user_map_permission_log` (
  `sn` smallint UNSIGNED NOT NULL,
  `user_id` varchar(60) NOT NULL,
  `user_account` varchar(60) NOT NULL,
  `map_sn` int NOT NULL,
  `updater_id` varchar(60) NOT NULL,
  `updater_account` varchar(60) NOT NULL,
  `action_type` tinyint UNSIGNED NOT NULL COMMENT '1.新增 2.刪除',
  `log_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `user_past_password`
--

CREATE TABLE `user_past_password` (
  `sn` bigint NOT NULL,
  `user_id` varchar(60) NOT NULL,
  `pass` varchar(60) NOT NULL,
  `modifier` varchar(60) DEFAULT NULL COMMENT '更改人',
  `confirmation` int NOT NULL DEFAULT '0' COMMENT '0:失敗,沒去收信;1:成功',
  `timestamp` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `user_person_config`
--

CREATE TABLE `user_person_config` (
  `user_id` varchar(60) NOT NULL,
  `item_card_or_list` tinyint(1) NOT NULL DEFAULT '1' COMMENT '卡片呈現或列表呈現(1卡片2列表)',
  `calendar` tinyint(1) NOT NULL DEFAULT '1' COMMENT '顯示行事曆與公告(1顯示0關閉)'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `user_play_game_log`
--

CREATE TABLE `user_play_game_log` (
  `game_log_sn` int UNSIGNED NOT NULL COMMENT '流水號',
  `user_id` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '使用者ID',
  `organization_id` varchar(10) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '學校ID',
  `api_client_sn` int UNSIGNED DEFAULT NULL COMMENT '遊戲ID',
  `other_system_name` varchar(30) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '沒編號的遊戲存中文名稱',
  `start_time` datetime NOT NULL COMMENT '時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `user_recovery_record`
--

CREATE TABLE `user_recovery_record` (
  `sn` bigint NOT NULL,
  `user_id` varchar(60) NOT NULL,
  `token` varchar(100) NOT NULL,
  `confirmation` int NOT NULL DEFAULT '0',
  `timestamp` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `user_status`
--

CREATE TABLE `user_status` (
  `user_id` varchar(60) NOT NULL COMMENT '使用者帳號',
  `access_level` int DEFAULT '1' COMMENT '存取等級',
  `login_freq` int NOT NULL DEFAULT '0' COMMENT '登入次數',
  `login_from` varchar(9) DEFAULT NULL COMMENT '登入方式',
  `starttimestamp` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '上次登入時間(時間格式)',
  `stoptimestamp` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '上次登出時間(時間格式)',
  `video_status` varchar(50) NOT NULL DEFAULT '0' COMMENT '看影片狀態(0:未看;1@XX@能力指標:正在看哪一支影片)',
  `lasttimein` int UNSIGNED DEFAULT NULL COMMENT '上次登入時間(數字格式)',
  `last_action_time` datetime NOT NULL COMMENT '最後活動時間',
  `lasttimeout` int UNSIGNED DEFAULT NULL COMMENT '上次登出時間(數字格式)',
  `total_stay` int UNSIGNED NOT NULL DEFAULT '0' COMMENT '總停留時間',
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
  `unique_id` varchar(40) NOT NULL COMMENT '登入的唯一值'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `user_status_log`
--

CREATE TABLE `user_status_log` (
  `log_sn` bigint UNSIGNED NOT NULL COMMENT '紀錄碼',
  `organization_id` varchar(10) DEFAULT NULL COMMENT '學校代碼',
  `user_id` varchar(60) NOT NULL COMMENT '使用者名稱',
  `user_account` varchar(60) DEFAULT NULL,
  `action` enum('Disable','Enable','Delete','Register','Login') NOT NULL COMMENT '操作',
  `update_id` varchar(60) DEFAULT NULL COMMENT '更新人',
  `record_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '紀錄時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci
PARTITION BY HASH (month(`record_time`))
PARTITIONS 12;

-- --------------------------------------------------------

--
-- 資料表結構 `user_third_party`
--

CREATE TABLE `user_third_party` (
  `user_id` varchar(60) NOT NULL COMMENT '帳號',
  `cloud_sub` varchar(100) DEFAULT NULL COMMENT '教育雲第三方帳號',
  `google_sub` varchar(100) DEFAULT NULL COMMENT 'goole第三方帳號',
  `cooc_sub` varchar(100) DEFAULT NULL COMMENT '酷課雲綁定',
  `ck_sub` varchar(100) DEFAULT NULL COMMENT 'CK第三方帳號',
  `create_by_third` int NOT NULL COMMENT '1=由第三方創帳號的, 0=原有帳號連動第三方'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='第三方帳號紀錄';

-- --------------------------------------------------------

--
-- 資料表結構 `video_concept_item`
--

CREATE TABLE `video_concept_item` (
  `video_item_sn` int NOT NULL COMMENT '影片試題id',
  `video_id` int NOT NULL COMMENT '影片id',
  `indicator` varchar(50) DEFAULT NULL COMMENT '能力指標',
  `map_sn` int DEFAULT NULL COMMENT '星空圖代號',
  `cs_id` varchar(12) NOT NULL COMMENT '概念id',
  `item_num` tinyint UNSIGNED NOT NULL COMMENT '項次',
  `video_nm` varchar(150) NOT NULL DEFAULT '影片' COMMENT '影片名稱',
  `video_len` decimal(15,3) NOT NULL COMMENT '影片長度',
  `video_src` varchar(100) DEFAULT '/' COMMENT '影片來源',
  `video_stream_url` mediumtext COMMENT '影片串流網址',
  `video_type` varchar(12) NOT NULL DEFAULT 'ovg' COMMENT '影片類型',
  `video_version` smallint UNSIGNED NOT NULL DEFAULT '1' COMMENT '影片版本',
  `video_subtitles` text COMMENT '影片字幕',
  `video_abstract` text COMMENT '影片摘要',
  `create_user_id` varchar(60) DEFAULT NULL COMMENT '建立者',
  `create_time` varchar(14) NOT NULL COMMENT '建立時間',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新時間',
  `public` int NOT NULL,
  `end_mk` char(1) NOT NULL DEFAULT 'N' COMMENT '停用註記',
  `video_tags` json NOT NULL COMMENT '影片tag'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='影片試題資訊' ROW_FORMAT=COMPACT;

-- --------------------------------------------------------

--
-- 資料表結構 `video_concept_item3`
--

CREATE TABLE `video_concept_item3` (
  `video_item_sn` int NOT NULL COMMENT '影片試題id',
  `video_id` int NOT NULL COMMENT '影片id',
  `indicator` varchar(100) NOT NULL COMMENT '能力指標',
  `map_sn` smallint NOT NULL COMMENT '星空圖代號',
  `cs_id` varchar(12) NOT NULL COMMENT '概念id',
  `item_num` tinyint UNSIGNED NOT NULL COMMENT '項次',
  `video_nm` varchar(150) NOT NULL DEFAULT '影片' COMMENT '影片名稱',
  `video_len` varchar(14) NOT NULL DEFAULT '0' COMMENT '影片長度',
  `video_src` varchar(100) DEFAULT '/' COMMENT '影片來源',
  `video_stream_url` mediumtext COMMENT '影片串流網址',
  `video_type` varchar(12) NOT NULL DEFAULT 'ovg' COMMENT '影片類型',
  `video_version` smallint UNSIGNED NOT NULL DEFAULT '1' COMMENT '影片版本',
  `create_user_id` varchar(50) NOT NULL COMMENT '建立者',
  `create_time` varchar(14) NOT NULL COMMENT '建立時間',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新時間',
  `public` int NOT NULL,
  `end_mk` char(1) NOT NULL DEFAULT 'N' COMMENT '停用註記'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='影片試題資訊' ROW_FORMAT=COMPACT;

-- --------------------------------------------------------

--
-- 資料表結構 `video_concept_item_0730`
--

CREATE TABLE `video_concept_item_0730` (
  `video_item_sn` int NOT NULL COMMENT '影片試題id',
  `video_id` int NOT NULL COMMENT '影片id',
  `indicator` varchar(50) DEFAULT NULL COMMENT '能力指標',
  `map_sn` int DEFAULT NULL COMMENT '星空圖代號',
  `cs_id` varchar(12) NOT NULL COMMENT '概念id',
  `item_num` tinyint UNSIGNED NOT NULL COMMENT '項次',
  `video_nm` varchar(150) NOT NULL DEFAULT '影片' COMMENT '影片名稱',
  `video_len` decimal(15,3) NOT NULL COMMENT '影片長度',
  `video_src` varchar(100) DEFAULT '/' COMMENT '影片來源',
  `video_stream_url` mediumtext COMMENT '影片串流網址',
  `video_type` varchar(12) NOT NULL DEFAULT 'ovg' COMMENT '影片類型',
  `video_version` smallint UNSIGNED NOT NULL DEFAULT '1' COMMENT '影片版本',
  `create_user_id` varchar(60) DEFAULT NULL COMMENT '建立者',
  `create_time` varchar(14) NOT NULL COMMENT '建立時間',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新時間',
  `public` int NOT NULL,
  `end_mk` char(1) NOT NULL DEFAULT 'N' COMMENT '停用註記',
  `video_tags` json NOT NULL COMMENT '影片tag'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='影片試題資訊' ROW_FORMAT=COMPACT;

-- --------------------------------------------------------

--
-- 資料表結構 `video_concept_item_20220427`
--

CREATE TABLE `video_concept_item_20220427` (
  `video_item_sn` int NOT NULL COMMENT '影片試題id',
  `video_id` int NOT NULL COMMENT '影片id',
  `indicator` varchar(100) NOT NULL COMMENT '能力指標',
  `map_sn` smallint NOT NULL COMMENT '星空圖代號',
  `cs_id` varchar(12) NOT NULL COMMENT '概念id',
  `item_num` tinyint UNSIGNED NOT NULL COMMENT '項次',
  `video_nm` varchar(150) NOT NULL DEFAULT '影片' COMMENT '影片名稱',
  `video_len` varchar(14) NOT NULL DEFAULT '0' COMMENT '影片長度',
  `video_src` varchar(100) DEFAULT '/' COMMENT '影片來源',
  `video_stream_url` mediumtext COMMENT '影片串流網址',
  `video_type` varchar(12) NOT NULL DEFAULT 'ovg' COMMENT '影片類型',
  `video_version` smallint UNSIGNED NOT NULL DEFAULT '1' COMMENT '影片版本',
  `create_user_id` varchar(50) NOT NULL COMMENT '建立者',
  `create_time` varchar(14) NOT NULL COMMENT '建立時間',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新時間',
  `public` int NOT NULL,
  `end_mk` char(1) NOT NULL DEFAULT 'N' COMMENT '停用註記'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='影片試題資訊' ROW_FORMAT=COMPACT;

-- --------------------------------------------------------

--
-- 資料表結構 `video_concept_item_20220428`
--

CREATE TABLE `video_concept_item_20220428` (
  `video_item_sn` int NOT NULL COMMENT '影片試題id',
  `video_id` int NOT NULL COMMENT '影片id',
  `indicator` varchar(100) NOT NULL COMMENT '能力指標',
  `map_sn` smallint NOT NULL COMMENT '星空圖代號',
  `cs_id` varchar(12) NOT NULL COMMENT '概念id',
  `item_num` tinyint UNSIGNED NOT NULL COMMENT '項次',
  `video_nm` varchar(150) NOT NULL DEFAULT '影片' COMMENT '影片名稱',
  `video_len` varchar(14) NOT NULL DEFAULT '0' COMMENT '影片長度',
  `video_src` varchar(100) DEFAULT '/' COMMENT '影片來源',
  `video_stream_url` mediumtext COMMENT '影片串流網址',
  `video_type` varchar(12) NOT NULL DEFAULT 'ovg' COMMENT '影片類型',
  `video_version` smallint UNSIGNED NOT NULL DEFAULT '1' COMMENT '影片版本',
  `create_user_id` varchar(50) NOT NULL COMMENT '建立者',
  `create_time` varchar(14) NOT NULL COMMENT '建立時間',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新時間',
  `public` int NOT NULL,
  `end_mk` char(1) NOT NULL DEFAULT 'N' COMMENT '停用註記'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='影片試題資訊' ROW_FORMAT=COMPACT;

-- --------------------------------------------------------

--
-- 資料表結構 `video_concept_item_plus`
--

CREATE TABLE `video_concept_item_plus` (
  `question_sn` int NOT NULL COMMENT '問題id',
  `video_item_sn` int NOT NULL COMMENT '試題id',
  `video_id` int DEFAULT NULL,
  `item_num` smallint UNSIGNED NOT NULL COMMENT '題號',
  `question_timestamp` smallint UNSIGNED NOT NULL COMMENT '問題時間點',
  `item_content` varchar(2000) NOT NULL COMMENT '題目文字內容',
  `item_filename` varchar(50) DEFAULT NULL COMMENT '題目圖片檔名',
  `op_content` mediumtext NOT NULL COMMENT '選項文字內容',
  `op_filename` varchar(50) DEFAULT NULL COMMENT '選項圖片檔名',
  `fb_filename` varchar(255) DEFAULT NULL COMMENT '作答的回饋',
  `op_ans` varchar(255) NOT NULL COMMENT '正確答案',
  `points` tinyint NOT NULL COMMENT '該題配分',
  `create_user_id` varchar(60) DEFAULT NULL COMMENT '建立者',
  `create_time` varchar(14) NOT NULL COMMENT '建立時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='影片問題記錄檔';

-- --------------------------------------------------------

--
-- 資料表結構 `video_concept_item_plus_bk`
--

CREATE TABLE `video_concept_item_plus_bk` (
  `question_sn` int NOT NULL COMMENT '問題id',
  `video_item_sn` int NOT NULL COMMENT '試題id',
  `video_id` int DEFAULT NULL,
  `item_num` smallint UNSIGNED NOT NULL COMMENT '題號',
  `question_timestamp` varchar(14) NOT NULL COMMENT '問題時間點',
  `item_content` varchar(2000) NOT NULL COMMENT '題目文字內容',
  `item_filename` varchar(50) DEFAULT NULL COMMENT '題目圖片檔名',
  `op_content` mediumtext NOT NULL COMMENT '選項文字內容',
  `op_filename` varchar(50) DEFAULT NULL COMMENT '選項圖片檔名',
  `fb_filename` varchar(255) DEFAULT NULL COMMENT '作答的回饋',
  `op_ans` varchar(255) NOT NULL COMMENT '正確答案',
  `points` tinyint NOT NULL COMMENT '該題配分',
  `create_user_id` varchar(60) DEFAULT NULL COMMENT '建立者',
  `create_time` varchar(14) NOT NULL COMMENT '建立時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='影片問題記錄檔';

-- --------------------------------------------------------

--
-- 資料表結構 `video_concept_stream_error`
--

CREATE TABLE `video_concept_stream_error` (
  `sn` bigint NOT NULL,
  `user_id` varchar(60) NOT NULL COMMENT '使用者帳號',
  `indicator` varchar(15) NOT NULL COMMENT '影片節點',
  `stream_url` text NOT NULL COMMENT '影片串流網址',
  `error_message` text NOT NULL COMMENT '錯誤訊息',
  `error_time` datetime NOT NULL COMMENT '錯誤發生時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='撥放影片串流錯誤的紀錄';

-- --------------------------------------------------------

--
-- 資料表結構 `video_exam_record`
--

CREATE TABLE `video_exam_record` (
  `video_exam_sn` int NOT NULL COMMENT '影片問題回答id',
  `review_sn` int NOT NULL COMMENT '影片瀏覽ID',
  `user_id` varchar(60) NOT NULL COMMENT '學生id',
  `video_item_sn` int NOT NULL COMMENT '試題id',
  `video_id` varchar(12) DEFAULT NULL,
  `question_sn` int NOT NULL COMMENT '問題id',
  `question_timestamp` varchar(14) DEFAULT NULL COMMENT '問題時間點',
  `ans_time` varchar(14) NOT NULL COMMENT '回答問題時間(毫秒)',
  `org_res` mediumtext NOT NULL COMMENT '原始作答情形',
  `binary_res` mediumtext NOT NULL COMMENT '二元作答情形(1:答對, 0:答錯)',
  `op_ans` varchar(1) NOT NULL COMMENT '正確答案',
  `indicator` varchar(50) NOT NULL COMMENT '能力指標',
  `timestamp` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='影片問題回答記錄檔';

-- --------------------------------------------------------

--
-- 資料表結構 `video_exam_record_20230316`
--

CREATE TABLE `video_exam_record_20230316` (
  `video_exam_sn` int NOT NULL COMMENT '影片問題回答id',
  `review_sn` int NOT NULL COMMENT '影片瀏覽ID',
  `user_id` varchar(60) NOT NULL COMMENT '學生id',
  `video_item_sn` int NOT NULL COMMENT '試題id',
  `video_id` varchar(12) DEFAULT NULL,
  `question_sn` int NOT NULL COMMENT '問題id',
  `question_timestamp` varchar(14) DEFAULT NULL COMMENT '問題時間點',
  `ans_time` varchar(14) NOT NULL COMMENT '回答問題時間(毫秒)',
  `org_res` mediumtext NOT NULL COMMENT '原始作答情形',
  `binary_res` mediumtext NOT NULL COMMENT '二元作答情形(1:答對, 0:答錯)',
  `op_ans` varchar(1) NOT NULL COMMENT '正確答案',
  `indicator` varchar(50) NOT NULL COMMENT '能力指標',
  `timestamp` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='影片問題回答記錄檔';

-- --------------------------------------------------------

--
-- 資料表結構 `video_note`
--

CREATE TABLE `video_note` (
  `video_note_sn` int NOT NULL COMMENT '影片筆記id',
  `user_id` varchar(60) NOT NULL COMMENT '學生id',
  `video_item_sn` int NOT NULL COMMENT '試題id',
  `stop_time` varchar(14) DEFAULT NULL COMMENT '時間點',
  `content` longtext NOT NULL COMMENT '筆記內容',
  `filename` mediumtext COMMENT '圖片檔案紀錄',
  `file_path` mediumtext COMMENT '上傳的檔案存取路徑(含檔名)',
  `upload_server` varchar(50) DEFAULT NULL COMMENT '在哪一台主機上傳',
  `file_backup` tinyint(1) DEFAULT '0' COMMENT '0:未備份至主機；1已備份至主機',
  `is_public` smallint NOT NULL DEFAULT '0' COMMENT '是否公開筆記，1=公開，0=不公開',
  `create_time` datetime NOT NULL COMMENT '建立時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='影片筆記記錄檔';

-- --------------------------------------------------------

--
-- 資料表結構 `video_noteask`
--

CREATE TABLE `video_noteask` (
  `ask_sn` int NOT NULL,
  `video_id` varchar(12) DEFAULT NULL,
  `video_item_sn` int DEFAULT NULL COMMENT '試題id',
  `user_id` varchar(60) NOT NULL,
  `stop_time` varchar(14) NOT NULL COMMENT '時間點',
  `question_content` longtext NOT NULL COMMENT '問題內容',
  `filename` mediumtext COMMENT '圖片檔案紀錄',
  `file_path` mediumtext COMMENT '上傳的檔案存取路徑(含檔名)',
  `upload_server` varchar(50) DEFAULT NULL COMMENT '在哪一台主機上傳',
  `file_backup` tinyint(1) NOT NULL DEFAULT '0' COMMENT '0:未備份至主機；1已備份至主機',
  `is_chooseBestAns` tinyint DEFAULT '0' COMMENT '已選擇最佳解答=1；反之=0',
  `create_time` datetime NOT NULL COMMENT '建立時間',
  `group_id` int DEFAULT NULL COMMENT '小組問題id',
  `grade_class` varchar(5) DEFAULT NULL COMMENT '年_班，限定班級回覆',
  `is_all_open` smallint NOT NULL DEFAULT '1' COMMENT '是否公開問題，1=公開，0=不公開',
  `is_public` smallint NOT NULL DEFAULT '1' COMMENT '是否限定小組，1為不限定，0為限定',
  `is_show` smallint NOT NULL DEFAULT '1' COMMENT '此問題是否已刪除；0=已刪除；1=未刪除',
  `hidden_reply` tinyint NOT NULL DEFAULT '0' COMMENT '0，開放學生互看 1，禁止學生互看'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='影片發問檔';

-- --------------------------------------------------------

--
-- 資料表結構 `video_noteask_plus`
--

CREATE TABLE `video_noteask_plus` (
  `reply_sn` int NOT NULL COMMENT '影片發問回答id',
  `ask_sn` int NOT NULL COMMENT '問題id',
  `user_id` varchar(60) NOT NULL COMMENT '老師id',
  `reply_content` mediumtext NOT NULL COMMENT '回答結果',
  `filename` mediumtext COMMENT '圖片檔案紀錄',
  `file_path` mediumtext COMMENT '上傳的檔案存取路徑(含檔名)',
  `upload_server` varchar(128) DEFAULT NULL COMMENT '在哪一台主機上傳',
  `file_backup` tinyint(1) NOT NULL DEFAULT '0' COMMENT '0:未備份至主機；1已備份至主機',
  `reply_time` datetime NOT NULL COMMENT '回答時間',
  `role_type` tinyint UNSIGNED NOT NULL DEFAULT '1' COMMENT '0:提問者, 1:回答者',
  `is_bestAnswer` int NOT NULL DEFAULT '0' COMMENT '1=最佳解答；0=不是',
  `is_show` int NOT NULL DEFAULT '1' COMMENT '是否顯示，1為顯示，0為隱藏',
  `deleter` varchar(60) DEFAULT NULL COMMENT '刪除者',
  `delete_time` datetime NOT NULL COMMENT '刪除時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='影片發問回答檔';

-- --------------------------------------------------------

--
-- 資料表結構 `video_noteask_plus_20230316`
--

CREATE TABLE `video_noteask_plus_20230316` (
  `reply_sn` int NOT NULL COMMENT '影片發問回答id',
  `ask_sn` int NOT NULL COMMENT '問題id',
  `user_id` varchar(60) NOT NULL COMMENT '老師id',
  `reply_content` mediumtext NOT NULL COMMENT '回答結果',
  `filename` mediumtext COMMENT '圖片檔案紀錄',
  `file_path` mediumtext COMMENT '上傳的檔案存取路徑(含檔名)',
  `upload_server` varchar(128) DEFAULT NULL COMMENT '在哪一台主機上傳',
  `file_backup` tinyint(1) NOT NULL DEFAULT '0' COMMENT '0:未備份至主機；1已備份至主機',
  `reply_time` datetime NOT NULL COMMENT '回答時間',
  `role_type` tinyint UNSIGNED NOT NULL DEFAULT '1' COMMENT '0:提問者, 1:回答者',
  `is_bestAnswer` int NOT NULL DEFAULT '0' COMMENT '1=最佳解答；0=不是',
  `is_show` int NOT NULL DEFAULT '1' COMMENT '是否顯示，1為顯示，0為隱藏',
  `deleter` varchar(60) DEFAULT NULL COMMENT '刪除者',
  `delete_time` datetime NOT NULL COMMENT '刪除時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='影片發問回答檔';

-- --------------------------------------------------------

--
-- 資料表結構 `video_note_favorite`
--

CREATE TABLE `video_note_favorite` (
  `favorite_sn` int NOT NULL COMMENT 'favoriteID',
  `user_id` varchar(60) NOT NULL COMMENT '學生id',
  `video_note_sn` int NOT NULL COMMENT '影片筆記id',
  `is_like` smallint NOT NULL DEFAULT '0' COMMENT '0:不喜歡;1:喜歡',
  `is_favorite` smallint NOT NULL DEFAULT '0' COMMENT '0:未收藏;1:收藏',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '建立時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `video_note_favorite_20230316`
--

CREATE TABLE `video_note_favorite_20230316` (
  `favorite_sn` int NOT NULL COMMENT 'favoriteID',
  `user_id` varchar(60) NOT NULL COMMENT '學生id',
  `video_note_sn` int NOT NULL COMMENT '影片筆記id',
  `is_like` smallint NOT NULL DEFAULT '0' COMMENT '0:不喜歡;1:喜歡',
  `is_favorite` smallint NOT NULL DEFAULT '0' COMMENT '0:未收藏;1:收藏',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '建立時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `video_note_feedback`
--

CREATE TABLE `video_note_feedback` (
  `feedback_sn` int NOT NULL COMMENT '回饋id',
  `user_id` varchar(60) NOT NULL COMMENT '學生id',
  `video_note_sn` int NOT NULL COMMENT '影片筆記id',
  `feedback_content` mediumtext NOT NULL COMMENT '筆記回饋內容',
  `deleter` varchar(60) DEFAULT NULL COMMENT '刪除者',
  `is_delete` int NOT NULL DEFAULT '0' COMMENT '0:未刪除;1:刪除',
  `delete_time` datetime NOT NULL COMMENT '刪除時間',
  `feedback_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '回饋時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `video_note_feedback_20230316`
--

CREATE TABLE `video_note_feedback_20230316` (
  `feedback_sn` int NOT NULL COMMENT '回饋id',
  `user_id` varchar(60) NOT NULL COMMENT '學生id',
  `video_note_sn` int NOT NULL COMMENT '影片筆記id',
  `feedback_content` mediumtext NOT NULL COMMENT '筆記回饋內容',
  `deleter` varchar(60) DEFAULT NULL COMMENT '刪除者',
  `is_delete` int NOT NULL DEFAULT '0' COMMENT '0:未刪除;1:刪除',
  `delete_time` datetime NOT NULL COMMENT '刪除時間',
  `feedback_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '回饋時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `video_note_recommend`
--

CREATE TABLE `video_note_recommend` (
  `video_recommend_sn` int NOT NULL COMMENT '推薦筆記id',
  `video_note_sn` int NOT NULL COMMENT '影片筆記id',
  `referrer_id` varchar(60) DEFAULT NULL COMMENT '推薦人id',
  `recommend_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '推薦時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `video_note_recommend_20230316`
--

CREATE TABLE `video_note_recommend_20230316` (
  `video_recommend_sn` int NOT NULL COMMENT '推薦筆記id',
  `video_note_sn` int NOT NULL COMMENT '影片筆記id',
  `referrer_id` varchar(60) DEFAULT NULL COMMENT '推薦人id',
  `recommend_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '推薦時間'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `video_review_record`
--

CREATE TABLE `video_review_record` (
  `review_sn` int NOT NULL,
  `user_id` varchar(60) DEFAULT NULL COMMENT '學生id',
  `video_id` varchar(12) DEFAULT NULL COMMENT '影片id',
  `video_item_sn` int NOT NULL COMMENT '試題id',
  `start_timestamp` decimal(14,2) NOT NULL COMMENT '開始影片時間',
  `start_time` datetime NOT NULL COMMENT '開始瀏覽影片時間',
  `end_timestamp` decimal(14,2) NOT NULL COMMENT '結束影片時間',
  `end_time` datetime NOT NULL,
  `total_time` decimal(14,2) NOT NULL COMMENT '總瀏覽時間',
  `total_grade` int NOT NULL DEFAULT '0' COMMENT '總分數',
  `mission_sn` int DEFAULT NULL,
  `finish_rate` int NOT NULL DEFAULT '0' COMMENT '完成度%',
  `seme_year` smallint UNSIGNED NOT NULL DEFAULT '106' COMMENT '學年度',
  `reward_mk` char(1) NOT NULL DEFAULT 'N' COMMENT '是否已計算獎勵'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='影片瀏覽記錄檔';

-- --------------------------------------------------------

--
-- 資料表結構 `video_review_record_plus`
--

CREATE TABLE `video_review_record_plus` (
  `review_plus_sn` int NOT NULL COMMENT '瀏覽明細id',
  `review_sn` int NOT NULL COMMENT '影片瀏覽 id',
  `view_time` datetime NOT NULL COMMENT '瀏覽時間',
  `view_action` varchar(20) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '瀏覽動作',
  `timestamp` decimal(9,3) NOT NULL COMMENT '影片時間',
  `entertime` decimal(9,3) DEFAULT NULL,
  `turbo` decimal(5,2) DEFAULT NULL COMMENT '影片撥放速率'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='影片瀏覽記錄檔';

-- --------------------------------------------------------

--
-- 資料表結構 `video_review_record_plus_20230316`
--

CREATE TABLE `video_review_record_plus_20230316` (
  `review_plus_sn` int NOT NULL COMMENT '瀏覽明細id',
  `review_sn` int NOT NULL COMMENT '影片瀏覽 id',
  `view_time` varchar(14) NOT NULL COMMENT '瀏覽時間',
  `view_action` varchar(12) NOT NULL COMMENT '瀏覽動作',
  `coordinate_x` mediumint DEFAULT NULL COMMENT '滑鼠坐標X',
  `coordinate_y` mediumint DEFAULT NULL COMMENT '滑鼠坐標Y',
  `timestamp` varchar(14) NOT NULL COMMENT '影片時間',
  `entertime` varchar(14) NOT NULL,
  `turbo` varchar(14) NOT NULL COMMENT '影片撥放速率'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='影片瀏覽記錄檔';

-- --------------------------------------------------------

--
-- 資料表結構 `video_review_record_tmp`
--

CREATE TABLE `video_review_record_tmp` (
  `review_sn` int NOT NULL,
  `user_id` varchar(50) NOT NULL COMMENT '學生id',
  `video_id` varchar(12) DEFAULT NULL COMMENT '影片id',
  `video_item_sn` int NOT NULL COMMENT '試題id',
  `start_timestamp` varchar(14) NOT NULL COMMENT '開始影片時間',
  `start_time` datetime NOT NULL COMMENT '開始瀏覽影片時間',
  `end_timestamp` varchar(14) NOT NULL COMMENT '結束影片時間',
  `end_time` datetime NOT NULL,
  `total_time` decimal(14,2) NOT NULL COMMENT '總瀏覽時間',
  `total_grade` int NOT NULL DEFAULT '0' COMMENT '總分數',
  `finish_rate` int NOT NULL DEFAULT '0' COMMENT '完成度%'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='影片瀏覽記錄檔';

-- --------------------------------------------------------

--
-- 資料表結構 `website_module`
--

CREATE TABLE `website_module` (
  `module_id` smallint UNSIGNED NOT NULL,
  `module_title` varchar(20) NOT NULL,
  `icon_class` varchar(50) NOT NULL COMMENT 'Font Awesone CSS class',
  `folder_name` varchar(30) NOT NULL,
  `sub_folder_name` varchar(30) NOT NULL,
  `filename` varchar(100) NOT NULL,
  `specific_uri` varchar(50) NOT NULL,
  `note` varchar(50) NOT NULL COMMENT '備註'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='網站功能模組';

-- --------------------------------------------------------

--
-- 資料表結構 `z_system_identity_log`
--

CREATE TABLE `z_system_identity_log` (
  `sn` int NOT NULL,
  `user_loginto` varchar(60) NOT NULL COMMENT '登入誰的帳號',
  `user_loginfrom` varchar(60) NOT NULL COMMENT '操作者的帳號',
  `ip` varchar(20) NOT NULL COMMENT '操作者的ip',
  `log_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `z_system_team_feedback_status`
--

CREATE TABLE `z_system_team_feedback_status` (
  `id` int NOT NULL,
  `pre_user` varchar(255) DEFAULT NULL,
  `user` varchar(255) NOT NULL COMMENT '處理人',
  `status` varchar(255) NOT NULL COMMENT '狀態',
  `reply_content` varchar(255) DEFAULT NULL COMMENT '指派處理',
  `note` varchar(255) DEFAULT '' COMMENT '回覆使用者',
  `q_id` int DEFAULT NULL COMMENT 'user_feedback.id',
  `update_date` datetime NOT NULL COMMENT '更新日期',
  `staff_only` varchar(255) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- --------------------------------------------------------

--
-- 資料表結構 `z_system_team_users`
--

CREATE TABLE `z_system_team_users` (
  `id` int NOT NULL,
  `user_account` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL,
  `user_id` varchar(60) NOT NULL COMMENT 'user_id為user_account',
  `subject_id` smallint UNSIGNED NOT NULL COMMENT '領域 1:語文科, 2:數學科, 3:自然科, 30:英語科, 12:師資生, 97:帳號問題&操作問題, 98:系統錯誤',
  `name` varchar(100) NOT NULL,
  `ip` varchar(100) NOT NULL,
  `login_to_user` int NOT NULL DEFAULT '0' COMMENT '快速登入功能(1：可使用/0：不能使用)',
  `time_created` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

--
-- 已傾印資料表的索引
--

--
-- 資料表索引 `api_client_config`
--
ALTER TABLE `api_client_config`
  ADD PRIMARY KEY (`api_client_sn`),
  ADD KEY `system_name` (`system_name`),
  ADD KEY `client_id` (`client_id`),
  ADD KEY `client_secret` (`client_secret`),
  ADD KEY `system_id` (`system_id`);

--
-- 資料表索引 `api_token_data`
--
ALTER TABLE `api_token_data`
  ADD PRIMARY KEY (`token_sn`),
  ADD UNIQUE KEY `uk_access_token` (`access_token`) USING BTREE,
  ADD KEY `api_client_sn` (`api_client_sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `behavior` (`behavior`),
  ADD KEY `item_sn` (`item_li_sn`),
  ADD KEY `accesstoken` (`access_token`),
  ADD KEY `mission_sn` (`mission_sn`);

--
-- 資料表索引 `chatroom`
--
ALTER TABLE `chatroom`
  ADD PRIMARY KEY (`chatroom_id`),
  ADD KEY `mission_sn` (`mission_sn`),
  ADD KEY `group_id` (`group_id`);

--
-- 資料表索引 `chatroom_member`
--
ALTER TABLE `chatroom_member`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `chatroom_id` (`chatroom_id`),
  ADD KEY `member` (`member`);

--
-- 資料表索引 `chat_log`
--
ALTER TABLE `chat_log`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `chatroom_id` (`chatroom_id`),
  ADD KEY `user_id` (`user_id`);

--
-- 資料表索引 `check_list_table`
--
ALTER TABLE `check_list_table`
  ADD PRIMARY KEY (`check_list_table_sn`),
  ADD KEY `teacher_id` (`teacher_id`),
  ADD KEY `type` (`type`),
  ADD KEY `public` (`public`),
  ADD KEY `is_delete` (`is_delete`),
  ADD KEY `seme` (`seme`);

--
-- 資料表索引 `city`
--
ALTER TABLE `city`
  ADD PRIMARY KEY (`city_code`),
  ADD KEY `city_name` (`city_name`),
  ADD KEY `display_order` (`display_order`),
  ADD KEY `openid_ftp_status` (`openid_ftp_status`);

--
-- 資料表索引 `city_area`
--
ALTER TABLE `city_area`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `city` (`city`),
  ADD KEY `post_code` (`post_code`);

--
-- 資料表索引 `class_name`
--
ALTER TABLE `class_name`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `type` (`type`),
  ADD KEY `num` (`num`);

--
-- 資料表索引 `competency_unit`
--
ALTER TABLE `competency_unit`
  ADD PRIMARY KEY (`sn`),
  ADD UNIQUE KEY `uk_unit_id` (`unit_id`),
  ADD KEY `used` (`used`),
  ADD KEY `genres_en` (`genres_en`),
  ADD KEY `subject_id` (`subject_id`);

--
-- 資料表索引 `concept_info`
--
ALTER TABLE `concept_info`
  ADD PRIMARY KEY (`cs_sn`),
  ADD KEY `publisher_id` (`publisher_id`),
  ADD KEY `subject_id` (`subject_id`),
  ADD KEY `grade` (`grade`),
  ADD KEY `cs_id` (`cs_id`),
  ADD KEY `vol` (`vol`),
  ADD KEY `unit` (`unit`),
  ADD KEY `indicator_nums` (`indicator_nums`);

--
-- 資料表索引 `concept_info_plus`
--
ALTER TABLE `concept_info_plus`
  ADD PRIMARY KEY (`cs_id`),
  ADD KEY `ready` (`ready`),
  ADD KEY `stu_stru_ready` (`stu_stru_ready`),
  ADD KEY `expert_stru_ready` (`expert_stru_ready`),
  ADD KEY `min_test_nums_stu` (`min_test_nums_stu`),
  ADD KEY `min_test_nums_expert` (`min_test_nums_expert`),
  ADD KEY `max_item_nums` (`max_item_nums`);

--
-- 資料表索引 `concept_item`
--
ALTER TABLE `concept_item`
  ADD PRIMARY KEY (`item_sn`),
  ADD KEY `exam_paper_id` (`exam_paper_id`),
  ADD KEY `cs_id` (`cs_id`),
  ADD KEY `paper_vol` (`paper_vol`),
  ADD KEY `indicator` (`indicator`),
  ADD KEY `template_id` (`template_id`),
  ADD KEY `item_num` (`item_num`),
  ADD KEY `op_ans` (`op_ans`),
  ADD KEY `indicator_item_num` (`indicator_item_num`),
  ADD KEY `double_item` (`double_item`);

--
-- 資料表索引 `concept_itemBank`
--
ALTER TABLE `concept_itemBank`
  ADD PRIMARY KEY (`item_sn`),
  ADD KEY `sems` (`sems`),
  ADD KEY `subject_id` (`subject_id`),
  ADD KEY `indicator` (`indicator`),
  ADD KEY `edit` (`edit`),
  ADD KEY `public` (`public`),
  ADD KEY `template_id` (`template_id`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `itembank_group` (`itembank_group`),
  ADD KEY `answer_option_type` (`answer_option_type`),
  ADD KEY `idx_mapsn_indicator_public` (`map_sn`,`indicator`,`public`);

--
-- 資料表索引 `concept_itemBank_0320`
--
ALTER TABLE `concept_itemBank_0320`
  ADD PRIMARY KEY (`item_sn`),
  ADD KEY `sems` (`sems`),
  ADD KEY `subject_id` (`subject_id`),
  ADD KEY `indicator` (`indicator`),
  ADD KEY `edit` (`edit`),
  ADD KEY `public` (`public`),
  ADD KEY `template_id` (`template_id`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `itembank_group` (`itembank_group`),
  ADD KEY `concept_itemBank_ibfk_1` (`map_sn`,`indicator`),
  ADD KEY `answer_option_type` (`answer_option_type`),
  ADD KEY `idx_mapsn_indicator_public` (`map_sn`,`indicator`,`public`);

--
-- 資料表索引 `concept_itemBank_0521`
--
ALTER TABLE `concept_itemBank_0521`
  ADD PRIMARY KEY (`item_sn`),
  ADD KEY `sems` (`sems`),
  ADD KEY `subject_id` (`subject_id`),
  ADD KEY `indicator` (`indicator`),
  ADD KEY `edit` (`edit`),
  ADD KEY `public` (`public`),
  ADD KEY `template_id` (`template_id`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `itembank_group` (`itembank_group`),
  ADD KEY `concept_itemBank_ibfk_1` (`map_sn`,`indicator`),
  ADD KEY `answer_option_type` (`answer_option_type`),
  ADD KEY `idx_mapsn_indicator_public` (`map_sn`,`indicator`,`public`);

--
-- 資料表索引 `concept_itemBank_20240904`
--
ALTER TABLE `concept_itemBank_20240904`
  ADD PRIMARY KEY (`item_sn`),
  ADD KEY `sems` (`sems`),
  ADD KEY `subject_id` (`subject_id`),
  ADD KEY `indicator` (`indicator`),
  ADD KEY `edit` (`edit`),
  ADD KEY `public` (`public`),
  ADD KEY `template_id` (`template_id`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `itembank_group` (`itembank_group`),
  ADD KEY `concept_itemBank_ibfk_1` (`map_sn`,`indicator`),
  ADD KEY `answer_option_type` (`answer_option_type`),
  ADD KEY `idx_mapsn_indicator_public` (`map_sn`,`indicator`,`public`);

--
-- 資料表索引 `concept_itemBank_20250512`
--
ALTER TABLE `concept_itemBank_20250512`
  ADD PRIMARY KEY (`item_sn`),
  ADD KEY `sems` (`sems`),
  ADD KEY `subject_id` (`subject_id`),
  ADD KEY `indicator` (`indicator`),
  ADD KEY `edit` (`edit`),
  ADD KEY `public` (`public`),
  ADD KEY `template_id` (`template_id`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `itembank_group` (`itembank_group`),
  ADD KEY `concept_itemBank_ibfk_1` (`map_sn`,`indicator`),
  ADD KEY `answer_option_type` (`answer_option_type`),
  ADD KEY `idx_mapsn_indicator_public` (`map_sn`,`indicator`,`public`);

--
-- 資料表索引 `concept_itemBank_20250819`
--
ALTER TABLE `concept_itemBank_20250819`
  ADD PRIMARY KEY (`item_sn`),
  ADD KEY `sems` (`sems`),
  ADD KEY `subject_id` (`subject_id`),
  ADD KEY `indicator` (`indicator`),
  ADD KEY `edit` (`edit`),
  ADD KEY `public` (`public`),
  ADD KEY `template_id` (`template_id`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `itembank_group` (`itembank_group`),
  ADD KEY `concept_itemBank_ibfk_1` (`map_sn`,`indicator`),
  ADD KEY `answer_option_type` (`answer_option_type`),
  ADD KEY `idx_mapsn_indicator_public` (`map_sn`,`indicator`,`public`);

--
-- 資料表索引 `concept_itemBank_20250827`
--
ALTER TABLE `concept_itemBank_20250827`
  ADD PRIMARY KEY (`item_sn`),
  ADD KEY `sems` (`sems`),
  ADD KEY `subject_id` (`subject_id`),
  ADD KEY `indicator` (`indicator`),
  ADD KEY `edit` (`edit`),
  ADD KEY `public` (`public`),
  ADD KEY `template_id` (`template_id`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `itembank_group` (`itembank_group`),
  ADD KEY `concept_itemBank_ibfk_1` (`map_sn`,`indicator`),
  ADD KEY `answer_option_type` (`answer_option_type`),
  ADD KEY `idx_mapsn_indicator_public` (`map_sn`,`indicator`,`public`);

--
-- 資料表索引 `concept_itemBank_20251105`
--
ALTER TABLE `concept_itemBank_20251105`
  ADD PRIMARY KEY (`item_sn`),
  ADD KEY `sems` (`sems`),
  ADD KEY `subject_id` (`subject_id`),
  ADD KEY `indicator` (`indicator`),
  ADD KEY `edit` (`edit`),
  ADD KEY `public` (`public`),
  ADD KEY `template_id` (`template_id`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `itembank_group` (`itembank_group`),
  ADD KEY `concept_itemBank_ibfk_1` (`map_sn`,`indicator`),
  ADD KEY `answer_option_type` (`answer_option_type`),
  ADD KEY `idx_mapsn_indicator_public` (`map_sn`,`indicator`,`public`);

--
-- 資料表索引 `concept_itemBank_20251223`
--
ALTER TABLE `concept_itemBank_20251223`
  ADD PRIMARY KEY (`item_sn`),
  ADD KEY `sems` (`sems`),
  ADD KEY `subject_id` (`subject_id`),
  ADD KEY `indicator` (`indicator`),
  ADD KEY `edit` (`edit`),
  ADD KEY `public` (`public`),
  ADD KEY `template_id` (`template_id`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `itembank_group` (`itembank_group`),
  ADD KEY `answer_option_type` (`answer_option_type`),
  ADD KEY `idx_mapsn_indicator_public` (`map_sn`,`indicator`,`public`);

--
-- 資料表索引 `concept_itemBank_bat`
--
ALTER TABLE `concept_itemBank_bat`
  ADD PRIMARY KEY (`item_sn`),
  ADD KEY `sems` (`sems`),
  ADD KEY `subject_id` (`subject_id`),
  ADD KEY `indicator` (`indicator`),
  ADD KEY `edit` (`edit`),
  ADD KEY `template_id` (`template_id`),
  ADD KEY `public` (`public`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `itembank_group` (`itembank_group`),
  ADD KEY `map_sn_2` (`map_sn`,`indicator`);

--
-- 資料表索引 `concept_itemBank_bat_inds`
--
ALTER TABLE `concept_itemBank_bat_inds`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `indicator` (`indicator`),
  ADD KEY `item_sn` (`item_sn`);

--
-- 資料表索引 `concept_itemBank_depiction`
--
ALTER TABLE `concept_itemBank_depiction`
  ADD PRIMARY KEY (`depiction_sn`),
  ADD KEY `concept_itemBank_depiction_ibfk_1` (`item_sn`),
  ADD KEY `updater` (`updater`);

--
-- 資料表索引 `concept_itemBank_depiction_0320`
--
ALTER TABLE `concept_itemBank_depiction_0320`
  ADD PRIMARY KEY (`depiction_sn`),
  ADD KEY `concept_itemBank_depiction_ibfk_1` (`item_sn`),
  ADD KEY `updater` (`updater`);

--
-- 資料表索引 `concept_itemBank_depiction_0710`
--
ALTER TABLE `concept_itemBank_depiction_0710`
  ADD PRIMARY KEY (`depiction_sn`),
  ADD KEY `concept_itemBank_depiction_ibfk_1` (`item_sn`),
  ADD KEY `updater` (`updater`);

--
-- 資料表索引 `concept_itemBank_depiction_20240904`
--
ALTER TABLE `concept_itemBank_depiction_20240904`
  ADD PRIMARY KEY (`depiction_sn`),
  ADD KEY `concept_itemBank_depiction_ibfk_1` (`item_sn`),
  ADD KEY `updater` (`updater`);

--
-- 資料表索引 `concept_itemBank_depiction_20250512`
--
ALTER TABLE `concept_itemBank_depiction_20250512`
  ADD PRIMARY KEY (`depiction_sn`),
  ADD KEY `concept_itemBank_depiction_ibfk_1` (`item_sn`),
  ADD KEY `updater` (`updater`);

--
-- 資料表索引 `concept_itemBank_depiction_20250819`
--
ALTER TABLE `concept_itemBank_depiction_20250819`
  ADD PRIMARY KEY (`depiction_sn`),
  ADD KEY `concept_itemBank_depiction_ibfk_1` (`item_sn`),
  ADD KEY `updater` (`updater`);

--
-- 資料表索引 `concept_itemBank_depiction_20250827`
--
ALTER TABLE `concept_itemBank_depiction_20250827`
  ADD PRIMARY KEY (`depiction_sn`),
  ADD KEY `concept_itemBank_depiction_ibfk_1` (`item_sn`),
  ADD KEY `updater` (`updater`);

--
-- 資料表索引 `concept_itemBank_depiction_20251105`
--
ALTER TABLE `concept_itemBank_depiction_20251105`
  ADD PRIMARY KEY (`depiction_sn`),
  ADD KEY `concept_itemBank_depiction_ibfk_1` (`item_sn`),
  ADD KEY `updater` (`updater`);

--
-- 資料表索引 `concept_itemBank_depiction_20251223`
--
ALTER TABLE `concept_itemBank_depiction_20251223`
  ADD PRIMARY KEY (`depiction_sn`),
  ADD KEY `concept_itemBank_depiction_ibfk_1` (`item_sn`),
  ADD KEY `updater` (`updater`);

--
-- 資料表索引 `concept_itemBank_literacy`
--
ALTER TABLE `concept_itemBank_literacy`
  ADD PRIMARY KEY (`item_sn`),
  ADD KEY `subject_id` (`subject_id`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `item_type` (`item_type`),
  ADD KEY `thinking_type` (`thought_type`),
  ADD KEY `item_status` (`item_status`),
  ADD KEY `create_user` (`create_user`),
  ADD KEY `literacy_group_sn` (`literacy_group_sn`),
  ADD KEY `theme` (`theme`),
  ADD KEY `sub_subject_id` (`sub_subject_id`),
  ADD KEY `learning_stage` (`learning_stage`),
  ADD KEY `answer` (`answer`);

--
-- 資料表索引 `concept_itemBank_literacy_20250512`
--
ALTER TABLE `concept_itemBank_literacy_20250512`
  ADD PRIMARY KEY (`item_sn`),
  ADD KEY `subject_id` (`subject_id`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `item_type` (`item_type`),
  ADD KEY `thinking_type` (`thought_type`),
  ADD KEY `item_status` (`item_status`),
  ADD KEY `create_user` (`create_user`),
  ADD KEY `literacy_group_sn` (`literacy_group_sn`),
  ADD KEY `theme` (`theme`),
  ADD KEY `sub_subject_id` (`sub_subject_id`),
  ADD KEY `learning_stage` (`learning_stage`),
  ADD KEY `answer` (`answer`);

--
-- 資料表索引 `concept_itemBank_literacy_20250819`
--
ALTER TABLE `concept_itemBank_literacy_20250819`
  ADD PRIMARY KEY (`item_sn`),
  ADD KEY `subject_id` (`subject_id`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `item_type` (`item_type`),
  ADD KEY `thinking_type` (`thought_type`),
  ADD KEY `item_status` (`item_status`),
  ADD KEY `create_user` (`create_user`),
  ADD KEY `literacy_group_sn` (`literacy_group_sn`),
  ADD KEY `theme` (`theme`),
  ADD KEY `sub_subject_id` (`sub_subject_id`),
  ADD KEY `learning_stage` (`learning_stage`),
  ADD KEY `answer` (`answer`);

--
-- 資料表索引 `concept_itemBank_literacy_20250827`
--
ALTER TABLE `concept_itemBank_literacy_20250827`
  ADD PRIMARY KEY (`item_sn`),
  ADD KEY `subject_id` (`subject_id`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `item_type` (`item_type`),
  ADD KEY `thinking_type` (`thought_type`),
  ADD KEY `item_status` (`item_status`),
  ADD KEY `create_user` (`create_user`),
  ADD KEY `literacy_group_sn` (`literacy_group_sn`),
  ADD KEY `theme` (`theme`),
  ADD KEY `sub_subject_id` (`sub_subject_id`),
  ADD KEY `learning_stage` (`learning_stage`),
  ADD KEY `answer` (`answer`);

--
-- 資料表索引 `concept_itemBank_literacy_20251105`
--
ALTER TABLE `concept_itemBank_literacy_20251105`
  ADD PRIMARY KEY (`item_sn`),
  ADD KEY `subject_id` (`subject_id`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `item_type` (`item_type`),
  ADD KEY `thinking_type` (`thought_type`),
  ADD KEY `item_status` (`item_status`),
  ADD KEY `create_user` (`create_user`),
  ADD KEY `literacy_group_sn` (`literacy_group_sn`),
  ADD KEY `theme` (`theme`),
  ADD KEY `sub_subject_id` (`sub_subject_id`),
  ADD KEY `learning_stage` (`learning_stage`),
  ADD KEY `answer` (`answer`);

--
-- 資料表索引 `concept_itemBank_literacy_20251223`
--
ALTER TABLE `concept_itemBank_literacy_20251223`
  ADD PRIMARY KEY (`item_sn`),
  ADD KEY `subject_id` (`subject_id`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `item_type` (`item_type`),
  ADD KEY `thinking_type` (`thought_type`),
  ADD KEY `item_status` (`item_status`),
  ADD KEY `create_user` (`create_user`),
  ADD KEY `literacy_group_sn` (`literacy_group_sn`),
  ADD KEY `theme` (`theme`),
  ADD KEY `sub_subject_id` (`sub_subject_id`),
  ADD KEY `learning_stage` (`learning_stage`),
  ADD KEY `answer` (`answer`);

--
-- 資料表索引 `concept_itemBank_literacy_conversation`
--
ALTER TABLE `concept_itemBank_literacy_conversation`
  ADD PRIMARY KEY (`literacy_conversation_sn`),
  ADD KEY `concept_itemBank_literacy_conversation_ibfk_1` (`item_li_sn`);

--
-- 資料表索引 `concept_itemBank_literacy_group`
--
ALTER TABLE `concept_itemBank_literacy_group`
  ADD PRIMARY KEY (`literacy_group_sn`),
  ADD KEY `subject_id` (`subject_id`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `create_user` (`create_user`),
  ADD KEY `sub_subject_id` (`sub_subject_id`),
  ADD KEY `status` (`status`);

--
-- 資料表索引 `concept_itemBank_literacy_interactive`
--
ALTER TABLE `concept_itemBank_literacy_interactive`
  ADD PRIMARY KEY (`item_li_sn`),
  ADD KEY `subject_id` (`subject_id`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `learning_stage` (`learning_stage`),
  ADD KEY `used` (`used`),
  ADD KEY `type` (`type`),
  ADD KEY `theme` (`theme`),
  ADD KEY `item_name` (`item_name`),
  ADD KEY `grade` (`grade`),
  ADD KEY `api_config_sn` (`api_config_sn`),
  ADD KEY `item_show_num` (`item_show_num`),
  ADD KEY `sub_subject_id` (`sub_subject_id`);

--
-- 資料表索引 `concept_itemBank_option`
--
ALTER TABLE `concept_itemBank_option`
  ADD PRIMARY KEY (`option_sn`),
  ADD UNIQUE KEY `uk_itemsn_optionnumber` (`item_sn`,`option_number`),
  ADD KEY `updater` (`updater`),
  ADD KEY `concept_itemBank_option_ibfk_1` (`item_sn`);

--
-- 資料表索引 `concept_itemBank_option_0320`
--
ALTER TABLE `concept_itemBank_option_0320`
  ADD PRIMARY KEY (`option_sn`),
  ADD UNIQUE KEY `uk_itemsn_optionnumber` (`item_sn`,`option_number`),
  ADD KEY `updater` (`updater`),
  ADD KEY `concept_itemBank_option_ibfk_1` (`item_sn`);

--
-- 資料表索引 `concept_itemBank_option_0710`
--
ALTER TABLE `concept_itemBank_option_0710`
  ADD PRIMARY KEY (`option_sn`),
  ADD UNIQUE KEY `uk_itemsn_optionnumber` (`item_sn`,`option_number`),
  ADD KEY `updater` (`updater`),
  ADD KEY `concept_itemBank_option_ibfk_1` (`item_sn`);

--
-- 資料表索引 `concept_item_interactive`
--
ALTER TABLE `concept_item_interactive`
  ADD PRIMARY KEY (`item_sn`),
  ADD KEY `sems` (`sems`),
  ADD KEY `subject_id` (`subject_id`),
  ADD KEY `indicator` (`indicator`),
  ADD KEY `public` (`public`),
  ADD KEY `template_id` (`template_id`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `op_ans` (`op_ans`),
  ADD KEY `create_user_id` (`create_user_id`),
  ADD KEY `map_sn_2` (`map_sn`,`indicator`);

--
-- 資料表索引 `concept_its`
--
ALTER TABLE `concept_its`
  ADD PRIMARY KEY (`its_sn`),
  ADD KEY `indicate_id` (`indicate_id`),
  ADD KEY `subject_id` (`subject_id`),
  ADD KEY `display` (`display`),
  ADD KEY `file_name` (`file_name`),
  ADD KEY `grade` (`grade`),
  ADD KEY `item_num` (`item_num`),
  ADD KEY `type` (`type`);

--
-- 資料表索引 `concept_paper`
--
ALTER TABLE `concept_paper`
  ADD PRIMARY KEY (`paper_sn`),
  ADD KEY `sems` (`sems`),
  ADD KEY `subject` (`subject`),
  ADD KEY `grade` (`grade`),
  ADD KEY `publisher` (`publisher`),
  ADD KEY `ower_id` (`ower_id`),
  ADD KEY `public` (`public`),
  ADD KEY `unit` (`unit`);

--
-- 資料表索引 `concept_priori`
--
ALTER TABLE `concept_priori`
  ADD PRIMARY KEY (`cp_sn`),
  ADD KEY `grade` (`grade`),
  ADD KEY `organization` (`organization_id`),
  ADD KEY `exam_range` (`exam_range`),
  ADD KEY `cp_id` (`cp_id`),
  ADD KEY `subject_id` (`subject_id`),
  ADD KEY `create_user` (`create_user`),
  ADD KEY `exam_type` (`exam_type`),
  ADD KEY `cp_info_sn` (`cp_info_sn`);

--
-- 資料表索引 `concept_priori_info`
--
ALTER TABLE `concept_priori_info`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `exam_range` (`exam_range`),
  ADD KEY `subject_id` (`subject_id`),
  ADD KEY `create_user` (`create_user`),
  ADD KEY `exam_type` (`exam_type`);

--
-- 資料表索引 `concept_remedy`
--
ALTER TABLE `concept_remedy`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `cs_id` (`cs_id`),
  ADD KEY `remedy_sn` (`remedy_sn`),
  ADD KEY `threshold` (`threshold`);

--
-- 資料表索引 `contestexam_item`
--
ALTER TABLE `contestexam_item`
  ADD PRIMARY KEY (`item_sn`),
  ADD KEY `exam_type` (`exam_type`),
  ADD KEY `item_type` (`item_type`),
  ADD KEY `answer_option_type` (`answer_option_type`),
  ADD KEY `keyboard_sn` (`keyboard_sn`),
  ADD KEY `status` (`status`),
  ADD KEY `updater` (`updater`),
  ADD KEY `map_sn` (`map_sn`);

--
-- 資料表索引 `contestexam_itemgroup`
--
ALTER TABLE `contestexam_itemgroup`
  ADD PRIMARY KEY (`group_sn`),
  ADD KEY `updater` (`updater`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `exam_type` (`exam_type`),
  ADD KEY `status` (`status`);

--
-- 資料表索引 `contestexam_itemgroup_detail`
--
ALTER TABLE `contestexam_itemgroup_detail`
  ADD PRIMARY KEY (`sn`),
  ADD UNIQUE KEY `uk_groupsn_itemsn` (`group_sn`,`item_sn`) USING BTREE,
  ADD UNIQUE KEY `uk_groupsn_itemorder` (`group_sn`,`item_order`) USING BTREE,
  ADD KEY `group_sn` (`group_sn`),
  ADD KEY `updater` (`updater`),
  ADD KEY `item_order` (`item_order`) USING BTREE,
  ADD KEY `item_sn` (`item_sn`) USING BTREE;

--
-- 資料表索引 `contestexam_item_answer_option`
--
ALTER TABLE `contestexam_item_answer_option`
  ADD PRIMARY KEY (`option_sn`),
  ADD UNIQUE KEY `uk_itemsn_optionnumber` (`item_sn`,`option_number`) USING BTREE,
  ADD KEY `option_number` (`option_number`),
  ADD KEY `is_correct` (`is_correct`),
  ADD KEY `updater` (`updater`),
  ADD KEY `item_sn` (`item_sn`) USING BTREE;

--
-- 資料表索引 `contestexam_item_indicator_mapping`
--
ALTER TABLE `contestexam_item_indicator_mapping`
  ADD PRIMARY KEY (`sn`) USING BTREE,
  ADD UNIQUE KEY `uk_itemsn_mapsn_indicateid` (`item_sn`,`map_sn`,`indicate_id`) USING BTREE,
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `indicate_id` (`indicate_id`),
  ADD KEY `updater` (`updater`),
  ADD KEY `item_sn` (`item_sn`) USING BTREE,
  ADD KEY `idx_mapsn_indicateid` (`map_sn`,`indicate_id`) USING BTREE;

--
-- 資料表索引 `contestexam_old_uploaded_filename`
--
ALTER TABLE `contestexam_old_uploaded_filename`
  ADD PRIMARY KEY (`sn`);

--
-- 資料表索引 `contestexam_paper`
--
ALTER TABLE `contestexam_paper`
  ADD PRIMARY KEY (`paper_sn`),
  ADD KEY `updater` (`updater`),
  ADD KEY `exam_type` (`exam_type`),
  ADD KEY `semester` (`year`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `status` (`status`);

--
-- 資料表索引 `contestexam_paper_detail`
--
ALTER TABLE `contestexam_paper_detail`
  ADD PRIMARY KEY (`sn`),
  ADD UNIQUE KEY `uk_papersn_itemsn` (`paper_sn`,`item_sn`) USING BTREE,
  ADD KEY `paper_sn` (`paper_sn`),
  ADD KEY `group_sn` (`group_sn`),
  ADD KEY `updater` (`updater`),
  ADD KEY `item_sn` (`item_sn`) USING BTREE,
  ADD KEY `item_order` (`item_order`),
  ADD KEY `idx_groupsn_itemsn` (`group_sn`,`item_sn`) USING BTREE;

--
-- 資料表索引 `contestexam_record`
--
ALTER TABLE `contestexam_record`
  ADD PRIMARY KEY (`record_sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `mission_sn` (`mission_sn`),
  ADD KEY `is_finished` (`is_finished`),
  ADD KEY `paper_sn` (`paper_sn`),
  ADD KEY `idx_papersn_isfinished_userid` (`paper_sn`,`is_finished`,`user_id`) USING BTREE;

--
-- 資料表索引 `contestexam_record_connection_log_nouse`
--
ALTER TABLE `contestexam_record_connection_log_nouse`
  ADD PRIMARY KEY (`log_sn`),
  ADD KEY `record_sn` (`record_sn`);

--
-- 資料表索引 `contestexam_record_detail`
--
ALTER TABLE `contestexam_record_detail`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `record_sn` (`record_sn`),
  ADD KEY `option_sn` (`option_sn`),
  ADD KEY `item_sn` (`item_sn`) USING BTREE,
  ADD KEY `group_sn` (`group_sn`),
  ADD KEY `idx_groupsn_itemsn` (`group_sn`,`item_sn`);

--
-- 資料表索引 `contest_agreement`
--
ALTER TABLE `contest_agreement`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `contest_sn` (`contest_sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `used` (`used`);

--
-- 資料表索引 `contest_certificate`
--
ALTER TABLE `contest_certificate`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `org_id` (`org_id`),
  ADD KEY `contest_sn` (`contest_sn`);

--
-- 資料表索引 `contest_content`
--
ALTER TABLE `contest_content`
  ADD PRIMARY KEY (`content_sn`),
  ADD KEY `contest_sn` (`contest_sn`),
  ADD KEY `group_sn` (`group_sn`),
  ADD KEY `topic_num` (`topic_num`),
  ADD KEY `teacher_id` (`teacher_id`) USING BTREE;

--
-- 資料表索引 `contest_dailyreport`
--
ALTER TABLE `contest_dailyreport`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `org_id` (`org_id`),
  ADD KEY `contest_sn` (`contest_sn`);

--
-- 資料表索引 `contest_group`
--
ALTER TABLE `contest_group`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `teacher_id` (`teacher_id`),
  ADD KEY `used` (`used`),
  ADD KEY `email` (`email`),
  ADD KEY `contest_sn` (`contest_sn`),
  ADD KEY `org_id` (`org_id`),
  ADD KEY `grade` (`grade`),
  ADD KEY `class` (`class`);

--
-- 資料表索引 `contest_group_changelog`
--
ALTER TABLE `contest_group_changelog`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `teacher_id` (`teacher_id`),
  ADD KEY `contest_sn` (`contest_sn`),
  ADD KEY `used` (`used`),
  ADD KEY `email` (`email`),
  ADD KEY `org_id` (`org_id`);

--
-- 資料表索引 `contest_group_comparison`
--
ALTER TABLE `contest_group_comparison`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `warm_sn` (`warm_sn`),
  ADD KEY `challenge_sn` (`challenge_sn`);

--
-- 資料表索引 `contest_info`
--
ALTER TABLE `contest_info`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `repeat_answer` (`repeat_answer`),
  ADD KEY `used` (`used`),
  ADD KEY `contest_code` (`contest_code`),
  ADD KEY `contest_type` (`contest_type`);

--
-- 資料表索引 `contest_itembank`
--
ALTER TABLE `contest_itembank`
  ADD PRIMARY KEY (`item_li_sn`),
  ADD KEY `subject_id` (`subject_id`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `learning_stage` (`learning_stage`),
  ADD KEY `used` (`used`),
  ADD KEY `type` (`type`),
  ADD KEY `theme` (`theme`),
  ADD KEY `item_name` (`item_name`),
  ADD KEY `grade` (`grade`),
  ADD KEY `api_config_sn` (`api_config_sn`),
  ADD KEY `item_show_num` (`item_show_num`),
  ADD KEY `contest_sn` (`contest_sn`),
  ADD KEY `question_num` (`question_num`),
  ADD KEY `sub_subject_id` (`sub_subject_id`),
  ADD KEY `indicator` (`indicator`);

--
-- 資料表索引 `contest_itembank_2021mathscience`
--
ALTER TABLE `contest_itembank_2021mathscience`
  ADD PRIMARY KEY (`item_li_sn`),
  ADD KEY `subject_id` (`subject_id`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `learning_stage` (`learning_stage`),
  ADD KEY `used` (`used`),
  ADD KEY `type` (`type`),
  ADD KEY `contest_sn` (`contest_sn`),
  ADD KEY `answerable_time` (`answerable_time`),
  ADD KEY `sub_subject_id` (`sub_subject_id`);

--
-- 資料表索引 `contest_itembank_2022math`
--
ALTER TABLE `contest_itembank_2022math`
  ADD PRIMARY KEY (`item_li_sn`),
  ADD KEY `subject_id` (`subject_id`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `learning_stage` (`learning_stage`),
  ADD KEY `used` (`used`),
  ADD KEY `type` (`type`),
  ADD KEY `theme` (`theme`),
  ADD KEY `item_name` (`item_name`),
  ADD KEY `grade` (`grade`),
  ADD KEY `api_config_sn` (`api_config_sn`),
  ADD KEY `item_show_num` (`item_show_num`),
  ADD KEY `contest_sn` (`contest_sn`),
  ADD KEY `question_num` (`question_num`),
  ADD KEY `sub_subject_id` (`sub_subject_id`),
  ADD KEY `indicator` (`indicator`);

--
-- 資料表索引 `contest_itembank_2023mathscience`
--
ALTER TABLE `contest_itembank_2023mathscience`
  ADD PRIMARY KEY (`item_li_sn`),
  ADD KEY `subject_id` (`subject_id`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `learning_stage` (`learning_stage`),
  ADD KEY `used` (`used`),
  ADD KEY `type` (`type`),
  ADD KEY `theme` (`theme`),
  ADD KEY `item_name` (`item_name`),
  ADD KEY `grade` (`grade`),
  ADD KEY `api_config_sn` (`api_config_sn`),
  ADD KEY `item_show_num` (`item_show_num`),
  ADD KEY `contest_sn` (`contest_sn`),
  ADD KEY `question_num` (`question_num`),
  ADD KEY `sub_subject_id` (`sub_subject_id`),
  ADD KEY `indicator` (`indicator`);

--
-- 資料表索引 `contest_itembank_group`
--
ALTER TABLE `contest_itembank_group`
  ADD PRIMARY KEY (`literacy_group_sn`),
  ADD KEY `subject_id` (`subject_id`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `create_user` (`create_user`),
  ADD KEY `contest_sn` (`contest_sn`),
  ADD KEY `sub_subject_id` (`sub_subject_id`),
  ADD KEY `status` (`status`);

--
-- 資料表索引 `contest_itemBank_literacy`
--
ALTER TABLE `contest_itemBank_literacy`
  ADD PRIMARY KEY (`item_sn`),
  ADD KEY `subject_id` (`subject_id`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `item_type` (`item_type`),
  ADD KEY `thinking_type` (`thought_type`),
  ADD KEY `item_status` (`item_status`),
  ADD KEY `create_user` (`create_user`),
  ADD KEY `literacy_group_sn` (`literacy_group_sn`),
  ADD KEY `contest_sn` (`contest_sn`),
  ADD KEY `sub_subject_id` (`sub_subject_id`),
  ADD KEY `learning_stage` (`learning_stage`),
  ADD KEY `answer` (`answer`);

--
-- 資料表索引 `contest_itembank_summer`
--
ALTER TABLE `contest_itembank_summer`
  ADD PRIMARY KEY (`item_li_sn`),
  ADD KEY `subject_id` (`subject_id`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `learning_stage` (`learning_stage`),
  ADD KEY `used` (`used`),
  ADD KEY `type` (`type`),
  ADD KEY `contest_sn` (`contest_sn`),
  ADD KEY `answerable_time` (`answerable_time`),
  ADD KEY `question_num` (`question_num`),
  ADD KEY `sub_subject_id` (`sub_subject_id`);

--
-- 資料表索引 `contest_publisher_mapping`
--
ALTER TABLE `contest_publisher_mapping`
  ADD PRIMARY KEY (`literacy_publisher_sn`),
  ADD KEY `literacy_item_sn` (`literacy_item_sn`),
  ADD KEY `type` (`type`),
  ADD KEY `grade` (`grade`),
  ADD KEY `seme` (`seme`),
  ADD KEY `publisher_id` (`publisher_id`),
  ADD KEY `unit` (`unit`);

--
-- 資料表索引 `contest_publisher_mapping_2021mathscience`
--
ALTER TABLE `contest_publisher_mapping_2021mathscience`
  ADD PRIMARY KEY (`literacy_publisher_sn`),
  ADD KEY `literacy_item_sn` (`literacy_item_sn`),
  ADD KEY `type` (`type`),
  ADD KEY `grade` (`grade`),
  ADD KEY `seme` (`seme`),
  ADD KEY `publisher_id` (`publisher_id`),
  ADD KEY `unit` (`unit`);

--
-- 資料表索引 `contest_publisher_mapping_2022math`
--
ALTER TABLE `contest_publisher_mapping_2022math`
  ADD PRIMARY KEY (`literacy_publisher_sn`),
  ADD KEY `literacy_item_sn` (`literacy_item_sn`),
  ADD KEY `type` (`type`),
  ADD KEY `grade` (`grade`),
  ADD KEY `seme` (`seme`),
  ADD KEY `publisher_id` (`publisher_id`),
  ADD KEY `unit` (`unit`);

--
-- 資料表索引 `contest_record`
--
ALTER TABLE `contest_record`
  ADD PRIMARY KEY (`exam_sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `mission_sn` (`mission_sn`),
  ADD KEY `date` (`update_time`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `over_time` (`over_time`),
  ADD KEY `item_li_sn` (`item_li_sn`),
  ADD KEY `contest_info_sn` (`contest_info_sn`),
  ADD KEY `subject_id` (`subject_id`);

--
-- 資料表索引 `contest_record_literacy`
--
ALTER TABLE `contest_record_literacy`
  ADD PRIMARY KEY (`exam_sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `mission_sn` (`contest_sn`),
  ADD KEY `date` (`update_time`),
  ADD KEY `correct_rate` (`correct_rate`),
  ADD KEY `void` (`void`);

--
-- 資料表索引 `contest_record_respond`
--
ALTER TABLE `contest_record_respond`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `contest_sn` (`contest_sn`),
  ADD KEY `org_id` (`org_id`),
  ADD KEY `type` (`type`);

--
-- 資料表索引 `contest_signup`
--
ALTER TABLE `contest_signup`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `contest_sn` (`contest_sn`),
  ADD KEY `type` (`type`),
  ADD KEY `group_sn` (`group_sn`),
  ADD KEY `used` (`used`),
  ADD KEY `email` (`email`),
  ADD KEY `organization_id` (`organization_id`),
  ADD KEY `school_system` (`school_system`);

--
-- 資料表索引 `contest_signup_20220422`
--
ALTER TABLE `contest_signup_20220422`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `contest_sn` (`contest_sn`),
  ADD KEY `type` (`type`),
  ADD KEY `group_sn` (`group_sn`),
  ADD KEY `used` (`used`),
  ADD KEY `email` (`email`),
  ADD KEY `organization_id` (`organization_id`);

--
-- 資料表索引 `contest_signup_changelog`
--
ALTER TABLE `contest_signup_changelog`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `contest_sn` (`contest_sn`),
  ADD KEY `type` (`type`),
  ADD KEY `group_sn` (`group_sn`),
  ADD KEY `used` (`used`),
  ADD KEY `email` (`email`);

--
-- 資料表索引 `conversation`
--
ALTER TABLE `conversation`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `exam_sn` (`exam_sn`),
  ADD KEY `conversation_ibfk_2` (`mission_sn`),
  ADD KEY `conversation_ibfk_3` (`item_sn`),
  ADD KEY `conversation_ibfk_1` (`user_id`),
  ADD KEY `robot_type` (`robot_type`),
  ADD KEY `completed_flag` (`completed_flag`),
  ADD KEY `map_sn` (`map_sn`,`indicator`),
  ADD KEY `idx_conv_user_robot_time` (`user_id`,`robot_type`,`creation_time`,`sn`);

--
-- 資料表索引 `conversation_access`
--
ALTER TABLE `conversation_access`
  ADD PRIMARY KEY (`sn`),
  ADD UNIQUE KEY `semester` (`semester`,`user_id`),
  ADD KEY `user_id` (`user_id`);

--
-- 資料表索引 `conversation_detail`
--
ALTER TABLE `conversation_detail`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `conversation_detail_vincent_ibfk_1` (`conversation_sn`),
  ADD KEY `idx_cd_sn_time` (`conversation_sn`,`creation_time`);

--
-- 資料表索引 `conversation_detail_images`
--
ALTER TABLE `conversation_detail_images`
  ADD PRIMARY KEY (`sn`);

--
-- 資料表索引 `conversation_detail_images_error`
--
ALTER TABLE `conversation_detail_images_error`
  ADD PRIMARY KEY (`sn`);

--
-- 資料表索引 `conversation_error`
--
ALTER TABLE `conversation_error`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `robot_type` (`robot_type`),
  ADD KEY `exam_sn` (`exam_sn`),
  ADD KEY `mission_sn` (`mission_sn`),
  ADD KEY `item_sn` (`item_sn`),
  ADD KEY `error_code` (`error_code`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `indicator` (`indicator`);

--
-- 資料表索引 `conversation_old`
--
ALTER TABLE `conversation_old`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `exam_sn` (`exam_sn`),
  ADD KEY `conversation_ibfk_2` (`mission_sn`),
  ADD KEY `conversation_ibfk_3` (`item_sn`),
  ADD KEY `conversation_ibfk_1` (`user_id`),
  ADD KEY `robot_type` (`robot_type`),
  ADD KEY `universal_flag` (`completed_flag`),
  ADD KEY `role` (`role`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `indicator` (`indicator`),
  ADD KEY `map_sn_2` (`map_sn`,`indicator`);

--
-- 資料表索引 `conversation_times`
--
ALTER TABLE `conversation_times`
  ADD PRIMARY KEY (`sn`);

--
-- 資料表索引 `cooperation_city_area`
--
ALTER TABLE `cooperation_city_area`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `city_name` (`city_name`),
  ADD KEY `area_code` (`area_code`),
  ADD KEY `area_name` (`area_name`);

--
-- 資料表索引 `cooperation_organization`
--
ALTER TABLE `cooperation_organization`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `seme` (`seme`),
  ADD KEY `organization_id` (`organization_id`),
  ADD KEY `area_code` (`area_code`),
  ADD KEY `co_type` (`co_type`);

--
-- 資料表索引 `course_subject`
--
ALTER TABLE `course_subject`
  ADD PRIMARY KEY (`cs_sn`),
  ADD KEY `type` (`type`),
  ADD KEY `college` (`college`),
  ADD KEY `department` (`department`),
  ADD KEY `subject_id` (`subject_id`),
  ADD KEY `subject_order` (`subject_order`);

--
-- 資料表索引 `cross_school_student`
--
ALTER TABLE `cross_school_student`
  ADD PRIMARY KEY (`cross_school_student_sn`),
  ADD KEY `teacher_id` (`teacher_id`),
  ADD KEY `student_id` (`student_id`);

--
-- 資料表索引 `csrf_security_log`
--
ALTER TABLE `csrf_security_log`
  ADD PRIMARY KEY (`id`),
  ADD KEY `idx_created_at` (`created_at`),
  ADD KEY `idx_event_type` (`event_type`),
  ADD KEY `idx_ip_address` (`ip_address`);

--
-- 資料表索引 `ct_node_mapping`
--
ALTER TABLE `ct_node_mapping`
  ADD PRIMARY KEY (`ct_node_mapping_sn`),
  ADD KEY `ct_node` (`ct_node`);

--
-- 資料表索引 `datashop_data_table`
--
ALTER TABLE `datashop_data_table`
  ADD PRIMARY KEY (`id`),
  ADD KEY `SearchType` (`SearchType`),
  ADD KEY `Subject` (`Subject`);

--
-- 資料表索引 `dbuser`
--
ALTER TABLE `dbuser`
  ADD PRIMARY KEY (`sn`),
  ADD UNIQUE KEY `uk_dbusername` (`dbusername`) USING BTREE,
  ADD UNIQUE KEY `uk_user_id` (`user_id`) USING BTREE;

--
-- 資料表索引 `eduexam_item`
--
ALTER TABLE `eduexam_item`
  ADD PRIMARY KEY (`item_sn`),
  ADD KEY `exam_type` (`exam_type`),
  ADD KEY `item_type` (`item_type`),
  ADD KEY `answer_option_type` (`answer_option_type`),
  ADD KEY `keyboard_sn` (`keyboard_sn`),
  ADD KEY `status` (`status`),
  ADD KEY `updater` (`updater`),
  ADD KEY `map_sn` (`map_sn`);

--
-- 資料表索引 `eduexam_itemgroup`
--
ALTER TABLE `eduexam_itemgroup`
  ADD PRIMARY KEY (`group_sn`),
  ADD KEY `updater` (`updater`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `exam_type` (`exam_type`),
  ADD KEY `status` (`status`);

--
-- 資料表索引 `eduexam_itemgroup_detail`
--
ALTER TABLE `eduexam_itemgroup_detail`
  ADD PRIMARY KEY (`sn`),
  ADD UNIQUE KEY `uk_groupsn_itemsn` (`group_sn`,`item_sn`) USING BTREE,
  ADD UNIQUE KEY `uk_groupsn_itemorder` (`group_sn`,`item_order`) USING BTREE,
  ADD KEY `group_sn` (`group_sn`),
  ADD KEY `updater` (`updater`),
  ADD KEY `item_order` (`item_order`) USING BTREE,
  ADD KEY `item_sn` (`item_sn`) USING BTREE;

--
-- 資料表索引 `eduexam_item_answer_option`
--
ALTER TABLE `eduexam_item_answer_option`
  ADD PRIMARY KEY (`option_sn`),
  ADD UNIQUE KEY `uk_itemsn_optionnumber` (`item_sn`,`option_number`) USING BTREE,
  ADD KEY `option_number` (`option_number`),
  ADD KEY `is_correct` (`is_correct`),
  ADD KEY `updater` (`updater`),
  ADD KEY `item_sn` (`item_sn`) USING BTREE;

--
-- 資料表索引 `eduexam_item_indicator_mapping`
--
ALTER TABLE `eduexam_item_indicator_mapping`
  ADD PRIMARY KEY (`sn`) USING BTREE,
  ADD UNIQUE KEY `uk_itemsn_mapsn_indicateid` (`item_sn`,`map_sn`,`indicate_id`) USING BTREE,
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `indicate_id` (`indicate_id`),
  ADD KEY `updater` (`updater`),
  ADD KEY `item_sn` (`item_sn`) USING BTREE,
  ADD KEY `idx_mapsn_indicateid` (`map_sn`,`indicate_id`) USING BTREE;

--
-- 資料表索引 `eduexam_old_uploaded_filename`
--
ALTER TABLE `eduexam_old_uploaded_filename`
  ADD PRIMARY KEY (`sn`);

--
-- 資料表索引 `eduexam_paper`
--
ALTER TABLE `eduexam_paper`
  ADD PRIMARY KEY (`paper_sn`),
  ADD KEY `updater` (`updater`),
  ADD KEY `exam_type` (`exam_type`),
  ADD KEY `semester` (`year`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `status` (`status`);

--
-- 資料表索引 `eduexam_paper_detail`
--
ALTER TABLE `eduexam_paper_detail`
  ADD PRIMARY KEY (`sn`),
  ADD UNIQUE KEY `uk_papersn_itemsn` (`paper_sn`,`item_sn`) USING BTREE,
  ADD KEY `paper_sn` (`paper_sn`),
  ADD KEY `group_sn` (`group_sn`),
  ADD KEY `updater` (`updater`),
  ADD KEY `item_sn` (`item_sn`) USING BTREE,
  ADD KEY `item_order` (`item_order`),
  ADD KEY `idx_groupsn_itemsn` (`group_sn`,`item_sn`) USING BTREE;

--
-- 資料表索引 `eduexam_record`
--
ALTER TABLE `eduexam_record`
  ADD PRIMARY KEY (`record_sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `mission_sn` (`mission_sn`),
  ADD KEY `is_finished` (`is_finished`),
  ADD KEY `paper_sn` (`paper_sn`),
  ADD KEY `idx_papersn_isfinished_userid` (`paper_sn`,`is_finished`,`user_id`) USING BTREE;

--
-- 資料表索引 `eduexam_record_detail`
--
ALTER TABLE `eduexam_record_detail`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `record_sn` (`record_sn`),
  ADD KEY `option_sn` (`option_sn`),
  ADD KEY `item_sn` (`item_sn`) USING BTREE,
  ADD KEY `group_sn` (`group_sn`),
  ADD KEY `idx_groupsn_itemsn` (`group_sn`,`item_sn`);

--
-- 資料表索引 `egame_exam_record`
--
ALTER TABLE `egame_exam_record`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `egame_id` (`egame_id`) USING BTREE,
  ADD KEY `user_id` (`user_id`),
  ADD KEY `rating` (`rating`);

--
-- 資料表索引 `egame_info`
--
ALTER TABLE `egame_info`
  ADD PRIMARY KEY (`egame_id`) USING BTREE,
  ADD KEY `idx_mapsn_indicateid` (`map_sn`,`indicate_id`),
  ADD KEY `chapter` (`chapter`),
  ADD KEY `stage` (`stage`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `indicate_id` (`indicate_id`),
  ADD KEY `island_name` (`island_id`);

--
-- 資料表索引 `exam_country_student_tmp`
--
ALTER TABLE `exam_country_student_tmp`
  ADD PRIMARY KEY (`stud_sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `uname` (`uname`),
  ADD KEY `organization_id` (`organization_id`),
  ADD KEY `grade` (`grade`),
  ADD KEY `class` (`class`),
  ADD KEY `seme_num` (`seme_num`),
  ADD KEY `seme` (`seme`);

--
-- 資料表索引 `exam_edu_parameter`
--
ALTER TABLE `exam_edu_parameter`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `edu_parameter` (`edu_parameter`),
  ADD KEY `show` (`show`);

--
-- 資料表索引 `exam_item_answer_option_type`
--
ALTER TABLE `exam_item_answer_option_type`
  ADD PRIMARY KEY (`answer_option_type`),
  ADD KEY `answer_option_type_code` (`answer_option_type_code`);

--
-- 資料表索引 `exam_item_type`
--
ALTER TABLE `exam_item_type`
  ADD PRIMARY KEY (`item_type`),
  ADD KEY `item_type_code` (`item_type_code`);

--
-- 資料表索引 `exam_onscreen_keyboard`
--
ALTER TABLE `exam_onscreen_keyboard`
  ADD PRIMARY KEY (`keyboard_sn`),
  ADD KEY `map_sn` (`map_sn`);

--
-- 資料表索引 `exam_paper`
--
ALTER TABLE `exam_paper`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `cs_id` (`cs_id`),
  ADD KEY `paper_vol` (`paper_vol`),
  ADD KEY `item_nums` (`item_nums`),
  ADD KEY `ready` (`ready`);

--
-- 資料表索引 `exam_paper_access`
--
ALTER TABLE `exam_paper_access`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `school_id` (`school_id`),
  ADD KEY `grade` (`grade`),
  ADD KEY `class` (`class`),
  ADD KEY `cs_id` (`cs_id`),
  ADD KEY `paper_vol` (`paper_vol`),
  ADD KEY `semester` (`semester`),
  ADD KEY `exam_paper_id` (`exam_paper_id`),
  ADD KEY `type_id` (`type_id`);

--
-- 資料表索引 `exam_record`
--
ALTER TABLE `exam_record`
  ADD PRIMARY KEY (`exam_sn`),
  ADD KEY `paper_vol` (`paper_vol`),
  ADD KEY `exam_title` (`exam_title`),
  ADD KEY `date` (`date`),
  ADD KEY `TEST` (`date`,`exam_sn`) USING BTREE,
  ADD KEY `cs_id` (`cs_id`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `firm_id` (`firm_id`),
  ADD KEY `degree` (`degree`),
  ADD KEY `type_id` (`type_id`),
  ADD KEY `seme_year` (`seme_year`),
  ADD KEY `reward_mk` (`reward_mk`);

--
-- 資料表索引 `exam_record_aassessment_detail`
--
ALTER TABLE `exam_record_aassessment_detail`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `organization_id` (`organization_id`),
  ADD KEY `exam_range` (`seme`),
  ADD KEY `subject_id` (`seat`),
  ADD KEY `grade` (`grade`),
  ADD KEY `class` (`class`),
  ADD KEY `uname` (`uname`),
  ADD KEY `OpenID_sub` (`OpenID_sub`),
  ADD KEY `rfh_sn` (`rfh_sn`);

--
-- 資料表索引 `exam_record_aassessment_detail_1028`
--
ALTER TABLE `exam_record_aassessment_detail_1028`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `organization_id` (`organization_id`),
  ADD KEY `exam_range` (`seme`),
  ADD KEY `subject_id` (`seat`),
  ADD KEY `grade` (`grade`),
  ADD KEY `class` (`class`),
  ADD KEY `uname` (`uname`),
  ADD KEY `OpenID_sub` (`OpenID_sub`),
  ADD KEY `rfh_sn` (`rfh_sn`),
  ADD KEY `update_time` (`update_time`);

--
-- 資料表索引 `exam_record_bat_detail`
--
ALTER TABLE `exam_record_bat_detail`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `exam_range` (`exam_range`),
  ADD KEY `subject_id` (`subject_id`),
  ADD KEY `city_code` (`city_code`),
  ADD KEY `organization_id` (`organization_id`),
  ADD KEY `grade` (`grade`),
  ADD KEY `class` (`class`),
  ADD KEY `seme_num` (`seme_num`),
  ADD KEY `uname` (`uname`),
  ADD KEY `rank` (`rank`);

--
-- 資料表索引 `exam_record_dynamic`
--
ALTER TABLE `exam_record_dynamic`
  ADD PRIMARY KEY (`exam_sn`),
  ADD KEY `indicator` (`indicator`),
  ADD KEY `exam_title` (`exam_title`),
  ADD KEY `date` (`date`),
  ADD KEY `cs_id` (`cs_id`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `mission_sn` (`mission_sn`),
  ADD KEY `paper_vol` (`paper_vol`),
  ADD KEY `seme_year` (`seme_year`);

--
-- 資料表索引 `exam_record_indicate`
--
ALTER TABLE `exam_record_indicate`
  ADD PRIMARY KEY (`exam_sn`),
  ADD KEY `date` (`date`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `result_sn` (`result_sn`),
  ADD KEY `cs_id` (`cs_id`),
  ADD KEY `seme_year` (`seme_year`),
  ADD KEY `reward_mk` (`reward_mk`);

--
-- 資料表索引 `exam_record_indicate_tmp`
--
ALTER TABLE `exam_record_indicate_tmp`
  ADD PRIMARY KEY (`exam_sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `result_sn` (`result_sn`),
  ADD KEY `cs_id` (`cs_id`),
  ADD KEY `type_id` (`type_id`),
  ADD KEY `seme_year` (`seme_year`);

--
-- 資料表索引 `exam_record_indicator`
--
ALTER TABLE `exam_record_indicator`
  ADD PRIMARY KEY (`exam_sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `paper_sn` (`paper_sn`),
  ADD KEY `mission_sn` (`mission_sn`),
  ADD KEY `date` (`date`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `semester` (`semester`);

--
-- 資料表索引 `exam_record_indicator_bat`
--
ALTER TABLE `exam_record_indicator_bat`
  ADD PRIMARY KEY (`exam_sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `paper_sn` (`paper_sn`),
  ADD KEY `mission_sn` (`mission_sn`),
  ADD KEY `date` (`date`),
  ADD KEY `semester` (`semester`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `reward_mk` (`reward_mk`);

--
-- 資料表索引 `exam_record_indicator_bat_tmp`
--
ALTER TABLE `exam_record_indicator_bat_tmp`
  ADD PRIMARY KEY (`exam_sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `paper_sn` (`paper_sn`),
  ADD KEY `mission_sn` (`mission_sn`),
  ADD KEY `date` (`date`);

--
-- 資料表索引 `exam_record_indicator_cap`
--
ALTER TABLE `exam_record_indicator_cap`
  ADD PRIMARY KEY (`exam_sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `paper_sn` (`paper_sn`),
  ADD KEY `mission_sn` (`mission_sn`),
  ADD KEY `date` (`date`),
  ADD KEY `semester` (`semester`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `reward_mk` (`reward_mk`);

--
-- 資料表索引 `exam_record_indicator_cap_tmp`
--
ALTER TABLE `exam_record_indicator_cap_tmp`
  ADD PRIMARY KEY (`exam_sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `paper_sn` (`paper_sn`),
  ADD KEY `mission_sn` (`mission_sn`);

--
-- 資料表索引 `exam_record_indicator_tmp`
--
ALTER TABLE `exam_record_indicator_tmp`
  ADD PRIMARY KEY (`exam_sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `paper_sn` (`paper_sn`),
  ADD KEY `mission_sn` (`mission_sn`),
  ADD KEY `date` (`date`);

--
-- 資料表索引 `exam_record_interactive`
--
ALTER TABLE `exam_record_interactive`
  ADD PRIMARY KEY (`exam_sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `mission_sn` (`mission_sn`),
  ADD KEY `date` (`update_time`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `correct_rate` (`correct_rate`),
  ADD KEY `subject_id` (`subject_id`);

--
-- 資料表索引 `exam_record_its`
--
ALTER TABLE `exam_record_its`
  ADD PRIMARY KEY (`its_sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `indicate_id` (`indicate_id`);

--
-- 資料表索引 `exam_record_literacy`
--
ALTER TABLE `exam_record_literacy`
  ADD PRIMARY KEY (`exam_sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `mission_sn` (`mission_sn`),
  ADD KEY `date` (`update_time`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `correct_rate` (`correct_rate`),
  ADD KEY `subject_id` (`subject_id`);

--
-- 資料表索引 `exam_record_literacy_interactive`
--
ALTER TABLE `exam_record_literacy_interactive`
  ADD PRIMARY KEY (`exam_sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `mission_sn` (`mission_sn`),
  ADD KEY `date` (`update_time`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `item_li_sn` (`item_li_sn`),
  ADD KEY `start_time` (`start_time`),
  ADD KEY `client_items_idle_time` (`client_items_idle_time`),
  ADD KEY `subject_id` (`subject_id`);

--
-- 資料表索引 `exam_record_literacy_interactive_log`
--
ALTER TABLE `exam_record_literacy_interactive_log`
  ADD PRIMARY KEY (`log_sn`),
  ADD KEY `date` (`update_time`),
  ADD KEY `item_li_sn` (`item_li_sn`),
  ADD KEY `exam_sn` (`exam_sn`);

--
-- 資料表索引 `exam_record_priori`
--
ALTER TABLE `exam_record_priori`
  ADD PRIMARY KEY (`exam_sn`),
  ADD KEY `cp_id` (`cp_id`,`user_id`),
  ADD KEY `grade` (`grade`),
  ADD KEY `idx_grade_cpid` (`grade`,`cp_id`),
  ADD KEY `cp_id_only` (`cp_id`),
  ADD KEY `user_id_only` (`user_id`),
  ADD KEY `exam_user` (`exam_user`),
  ADD KEY `idx_userid_examuser` (`user_id`,`exam_user`),
  ADD KEY `seme_year` (`seme_year`),
  ADD KEY `reward_mk` (`reward_mk`);

--
-- 資料表索引 `exam_record_priori_detail`
--
ALTER TABLE `exam_record_priori_detail`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `organization_id` (`organization_id`),
  ADD KEY `prior_name` (`prior_name`),
  ADD KEY `exam_range` (`exam_range`),
  ADD KEY `subject_id` (`subject_id`),
  ADD KEY `grade` (`grade`),
  ADD KEY `class` (`class`),
  ADD KEY `exam_grade` (`exam_grade`),
  ADD KEY `pass` (`pass`);

--
-- 資料表索引 `exam_record_priori_error`
--
ALTER TABLE `exam_record_priori_error`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `organization_id` (`organization_id`),
  ADD KEY `cp_info_sn` (`cp_info_sn`),
  ADD KEY `exam_sn` (`exam_sn`),
  ADD KEY `exam_user` (`exam_user`),
  ADD KEY `grade` (`grade`),
  ADD KEY `class` (`class`),
  ADD KEY `seme_num` (`seme_num`),
  ADD KEY `date` (`date`),
  ADD KEY `exam_title` (`exam_title`),
  ADD KEY `seme_year` (`seme_year`),
  ADD KEY `is_connect` (`is_connect`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `reward_mk` (`reward_mk`),
  ADD KEY `cp_grade` (`cp_grade`);

--
-- 資料表索引 `exam_record_priori_his`
--
ALTER TABLE `exam_record_priori_his`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `cp_id` (`cp_id`,`user_id`),
  ADD KEY `mission_sn` (`mission_sn`),
  ADD KEY `exam_type` (`exam_type`),
  ADD KEY `exam_sn` (`exam_sn`),
  ADD KEY `seme_year` (`seme_year`),
  ADD KEY `reward_mk` (`reward_mk`),
  ADD KEY `exam_record_priori_his_ibfk_1` (`user_id`);

--
-- 資料表索引 `exam_record_tmp`
--
ALTER TABLE `exam_record_tmp`
  ADD PRIMARY KEY (`exam_sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `cs_id` (`cs_id`),
  ADD KEY `paper_vol` (`paper_vol`),
  ADD KEY `exam_title` (`exam_title`),
  ADD KEY `seme_year` (`seme_year`),
  ADD KEY `degree` (`degree`),
  ADD KEY `type_id` (`type_id`),
  ADD KEY `firm_id` (`firm_id`);

--
-- 資料表索引 `exam_template`
--
ALTER TABLE `exam_template`
  ADD PRIMARY KEY (`template_id`);

--
-- 資料表索引 `exam_type`
--
ALTER TABLE `exam_type`
  ADD PRIMARY KEY (`exam_type`) USING BTREE,
  ADD KEY `exam_type_code` (`exam_type_code`),
  ADD KEY `mission_type` (`mission_type`),
  ADD KEY `series` (`series`);

--
-- 資料表索引 `export_school`
--
ALTER TABLE `export_school`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `org_id` (`org_id`),
  ADD KEY `org_school` (`org_school`),
  ADD KEY `new_id` (`new_id`),
  ADD KEY `new_school` (`new_school`),
  ADD KEY `import_seme` (`import_seme`),
  ADD KEY `export_seme` (`export_seme`),
  ADD KEY `export_date` (`export_date`),
  ADD KEY `exporter` (`exporter`),
  ADD KEY `user_level` (`user_level`),
  ADD KEY `import_date` (`import_date`);

--
-- 資料表索引 `external_res_type`
--
ALTER TABLE `external_res_type`
  ADD PRIMARY KEY (`type_sn`),
  ADD KEY `type_sn` (`type_sn`);

--
-- 資料表索引 `ftp_upload_log`
--
ALTER TABLE `ftp_upload_log`
  ADD PRIMARY KEY (`id`),
  ADD KEY `idx_user_id` (`user_id`),
  ADD KEY `idx_created_at` (`created_at`);

--
-- 資料表索引 `group_check_result`
--
ALTER TABLE `group_check_result`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `check_list_sn` (`check_list_sn`),
  ADD KEY `total_score` (`total_score`),
  ADD KEY `mission_sn` (`mission_sn`);

--
-- 資料表索引 `group_member_score_result`
--
ALTER TABLE `group_member_score_result`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `check_list_sn` (`check_list_sn`),
  ADD KEY `score_user_id` (`score_user_id`),
  ADD KEY `mission_sn` (`mission_sn`);

--
-- 資料表索引 `group_role`
--
ALTER TABLE `group_role`
  ADD PRIMARY KEY (`group_role_sn`),
  ADD KEY `teacher_id` (`teacher_id`);

--
-- 資料表索引 `group_score_ingroup_result`
--
ALTER TABLE `group_score_ingroup_result`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `check_list_sn` (`check_list_sn`),
  ADD KEY `score_user_id` (`score_user_id`),
  ADD KEY `group_id` (`group_id`),
  ADD KEY `mission_sn` (`mission_sn`);

--
-- 資料表索引 `group_score_result`
--
ALTER TABLE `group_score_result`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `check_list_sn` (`check_list_sn`),
  ADD KEY `user_group_id` (`user_group_id`),
  ADD KEY `group_id` (`group_id`),
  ADD KEY `total_score` (`total_score`),
  ADD KEY `mission_sn` (`mission_sn`);

--
-- 資料表索引 `gtm_logs`
--
ALTER TABLE `gtm_logs`
  ADD PRIMARY KEY (`id`),
  ADD KEY `idx_user_id` (`user_id`) COMMENT '使用者索引',
  ADD KEY `idx_event` (`event`) COMMENT '事件索引',
  ADD KEY `idx_timestamp` (`timestamp`) COMMENT '時間戳記索引',
  ADD KEY `idx_created_at` (`created_at`) COMMENT '建立時間索引';

--
-- 資料表索引 `high_level_comment`
--
ALTER TABLE `high_level_comment`
  ADD PRIMARY KEY (`comment_sn`);

--
-- 資料表索引 `high_level_group_ans_list`
--
ALTER TABLE `high_level_group_ans_list`
  ADD PRIMARY KEY (`ans_id`),
  ADD UNIQUE KEY `index_unique_answer` (`mission_sn`,`question_sn`,`group_id`,`sub_question`,`according`);

--
-- 資料表索引 `high_level_joint_discovery`
--
ALTER TABLE `high_level_joint_discovery`
  ADD PRIMARY KEY (`discovery_sn`);

--
-- 資料表索引 `high_level_log`
--
ALTER TABLE `high_level_log`
  ADD PRIMARY KEY (`sn`);

--
-- 資料表索引 `high_level_personal_ans_list`
--
ALTER TABLE `high_level_personal_ans_list`
  ADD PRIMARY KEY (`ans_id`);

--
-- 資料表索引 `high_level_question`
--
ALTER TABLE `high_level_question`
  ADD PRIMARY KEY (`question_sn`);

--
-- 資料表索引 `high_level_question_group`
--
ALTER TABLE `high_level_question_group`
  ADD PRIMARY KEY (`group_id`);

--
-- 資料表索引 `high_level_type`
--
ALTER TABLE `high_level_type`
  ADD PRIMARY KEY (`type_id`);

--
-- 資料表索引 `high_school_dept`
--
ALTER TABLE `high_school_dept`
  ADD PRIMARY KEY (`dept_sn`),
  ADD KEY `dept_code` (`dept_code`) USING BTREE;

--
-- 資料表索引 `high_school_dept_mapping`
--
ALTER TABLE `high_school_dept_mapping`
  ADD PRIMARY KEY (`dept_mapping_sn`),
  ADD UNIQUE KEY `uk_schooltype_groupsn_deptsn` (`school_type`,`group_sn`,`dept_mapping_sn`),
  ADD KEY `dept_sn` (`dept_sn`) USING BTREE,
  ADD KEY `school_type` (`school_type`) USING BTREE,
  ADD KEY `group_sn` (`group_sn`) USING BTREE;

--
-- 資料表索引 `high_school_group`
--
ALTER TABLE `high_school_group`
  ADD PRIMARY KEY (`group_sn`),
  ADD KEY `group_code` (`group_code`) USING BTREE;

--
-- 資料表索引 `high_school_type`
--
ALTER TABLE `high_school_type`
  ADD PRIMARY KEY (`school_type`),
  ADD KEY `school_type_code` (`school_type_code`) USING BTREE;

--
-- 資料表索引 `indicateTest`
--
ALTER TABLE `indicateTest`
  ADD PRIMARY KEY (`sn`),
  ADD UNIQUE KEY `uk_indicate` (`indicate`),
  ADD KEY `class` (`class`),
  ADD KEY `subject` (`subject`);

--
-- 資料表索引 `indicate_test_predata`
--
ALTER TABLE `indicate_test_predata`
  ADD PRIMARY KEY (`sn`),
  ADD UNIQUE KEY `uk_mission_sn` (`mission_sn`);

--
-- 資料表索引 `indicator_info`
--
ALTER TABLE `indicator_info`
  ADD PRIMARY KEY (`indicator_sn`),
  ADD KEY `subject` (`subject`),
  ADD KEY `indicator` (`indicator`);

--
-- 資料表索引 `indicator_priori`
--
ALTER TABLE `indicator_priori`
  ADD PRIMARY KEY (`priori_sn`),
  ADD KEY `subject` (`subject`),
  ADD KEY `priori` (`priori`);

--
-- 資料表索引 `ItemBank_bat_fix_record`
--
ALTER TABLE `ItemBank_bat_fix_record`
  ADD PRIMARY KEY (`fix_record_sn`),
  ADD KEY `item_sn` (`item_sn`),
  ADD KEY `fix_result_index` (`status`,`item_sn`) USING BTREE,
  ADD KEY `fix_result` (`status`) USING BTREE;

--
-- 資料表索引 `ItemBank_fix_record`
--
ALTER TABLE `ItemBank_fix_record`
  ADD PRIMARY KEY (`fix_record_sn`),
  ADD KEY `item_sn` (`item_sn`),
  ADD KEY `fix_result_index` (`status`,`item_sn`) USING BTREE,
  ADD KEY `fix_result` (`status`) USING BTREE,
  ADD KEY `user_id` (`user_id`),
  ADD KEY `item_type` (`item_type`);

--
-- 資料表索引 `item_mission_rate`
--
ALTER TABLE `item_mission_rate`
  ADD PRIMARY KEY (`item_mission_rate_sn`),
  ADD KEY `mission_sn` (`mission_sn`),
  ADD KEY `item_sn` (`item_sn`);

--
-- 資料表索引 `lecturer_list`
--
ALTER TABLE `lecturer_list`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `category` (`category`),
  ADD KEY `create_time` (`create_time`),
  ADD KEY `city` (`city`);

--
-- 資料表索引 `literacy_publisher_mapping`
--
ALTER TABLE `literacy_publisher_mapping`
  ADD PRIMARY KEY (`literacy_publisher_sn`),
  ADD KEY `literacy_item_sn` (`literacy_item_sn`),
  ADD KEY `type` (`type`),
  ADD KEY `grade` (`grade`),
  ADD KEY `seme` (`seme`),
  ADD KEY `publisher_id` (`publisher_id`),
  ADD KEY `unit` (`unit`);

--
-- 資料表索引 `log_exam_bug`
--
ALTER TABLE `log_exam_bug`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `file` (`file`),
  ADD KEY `server_ip` (`server_ip`);

--
-- 資料表索引 `log_exam_record_aassessment_detail`
--
ALTER TABLE `log_exam_record_aassessment_detail`
  ADD PRIMARY KEY (`log_id`),
  ADD KEY `idx_original_sn` (`original_sn`),
  ADD KEY `idx_organization_id` (`organization_id`),
  ADD KEY `idx_user_id` (`user_id`),
  ADD KEY `idx_action_type` (`action_type`),
  ADD KEY `idx_log_date` (`log_date`);

--
-- 資料表索引 `mad_exam_record`
--
ALTER TABLE `mad_exam_record`
  ADD PRIMARY KEY (`exam_sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `cs_id` (`cs_id`),
  ADD KEY `paper_vol` (`paper_vol`),
  ADD KEY `exam_title` (`exam_title`),
  ADD KEY `firm_id` (`firm_id`),
  ADD KEY `item_num` (`item_num`),
  ADD KEY `saveSel` (`saveSel`),
  ADD KEY `formal` (`formal`),
  ADD KEY `type_id` (`type_id`);

--
-- 資料表索引 `mad_item`
--
ALTER TABLE `mad_item`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `sn` (`sn`);

--
-- 資料表索引 `mad_rule`
--
ALTER TABLE `mad_rule`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `mad_type_id` (`mad_type_id`);

--
-- 資料表索引 `mad_type`
--
ALTER TABLE `mad_type`
  ADD PRIMARY KEY (`mad_type_id`);

--
-- 資料表索引 `main_data`
--
ALTER TABLE `main_data`
  ADD PRIMARY KEY (`num`),
  ADD KEY `c_question_num` (`c_question_num`),
  ADD KEY `member_num` (`member_num`);

--
-- 資料表索引 `map_arrows`
--
ALTER TABLE `map_arrows`
  ADD PRIMARY KEY (`arrow_sn`),
  ADD KEY `map_arrows_ibfk_1` (`map_sn`),
  ADD KEY `class` (`class`),
  ADD KEY `arrow_creater` (`arrow_creater`);

--
-- 資料表索引 `map_info`
--
ALTER TABLE `map_info`
  ADD PRIMARY KEY (`map_sn`),
  ADD UNIQUE KEY `uk_map_main_file` (`map_main_file`),
  ADD KEY `display` (`display`),
  ADD KEY `subject_id` (`subject_id`),
  ADD KEY `create_user_id` (`create_user_id`),
  ADD KEY `itembank_group` (`itembank_group`),
  ADD KEY `map_name` (`map_name`);

--
-- 資料表索引 `map_info_0903`
--
ALTER TABLE `map_info_0903`
  ADD PRIMARY KEY (`map_sn`),
  ADD UNIQUE KEY `uk_map_main_file` (`map_main_file`),
  ADD KEY `display` (`display`),
  ADD KEY `subject_id` (`subject_id`),
  ADD KEY `create_user_id` (`create_user_id`),
  ADD KEY `itembank_group` (`itembank_group`),
  ADD KEY `map_name` (`map_name`);

--
-- 資料表索引 `map_node`
--
ALTER TABLE `map_node`
  ADD PRIMARY KEY (`node_sn`),
  ADD KEY `idx_map_sn_indicate_id` (`map_sn`,`indicate_id`);
ALTER TABLE `map_node` ADD FULLTEXT KEY `indicate_name_2` (`indicate_name`);

--
-- 資料表索引 `map_node_0521`
--
ALTER TABLE `map_node_0521`
  ADD PRIMARY KEY (`node_sn`),
  ADD UNIQUE KEY `uk_mapsn_indicateid` (`map_sn`,`indicate_id`) USING BTREE,
  ADD KEY `subject` (`subject`),
  ADD KEY `DA` (`DA`),
  ADD KEY `prac` (`prac`),
  ADD KEY `video` (`video`),
  ADD KEY `teach` (`teach`),
  ADD KEY `indicate_name` (`indicate_name`) USING BTREE,
  ADD KEY `indicate_id` (`indicate_id`),
  ADD KEY `idx_indicateid_indicatename_class` (`indicate_id`,`indicate_name`,`class`) USING BTREE,
  ADD KEY `idx_indicateid_class_mapsn` (`indicate_id`,`class`,`map_sn`),
  ADD KEY `CR` (`CR`),
  ADD KEY `class` (`class`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `idx_mapsn_class_video_indicateid` (`map_sn`,`class`,`video`,`indicate_id`),
  ADD KEY `sub_subject_id` (`sub_subject_id`),
  ADD KEY `idx_mapsn_class_indicateid` (`map_sn`,`class`,`indicate_id`),
  ADD KEY `blank_filling` (`blank_filling`),
  ADD KEY `node_file` (`node_file`),
  ADD KEY `external` (`external`),
  ADD KEY `node_creater` (`node_creater`);

--
-- 資料表索引 `map_node_20240813`
--
ALTER TABLE `map_node_20240813`
  ADD PRIMARY KEY (`node_sn`),
  ADD UNIQUE KEY `uk_mapsn_indicateid` (`map_sn`,`indicate_id`) USING BTREE,
  ADD KEY `subject` (`subject`),
  ADD KEY `DA` (`DA`),
  ADD KEY `prac` (`prac`),
  ADD KEY `video` (`video`),
  ADD KEY `teach` (`teach`),
  ADD KEY `indicate_name` (`indicate_name`) USING BTREE,
  ADD KEY `indicate_id` (`indicate_id`),
  ADD KEY `idx_indicateid_indicatename_class` (`indicate_id`,`indicate_name`,`class`) USING BTREE,
  ADD KEY `idx_indicateid_class_mapsn` (`indicate_id`,`class`,`map_sn`),
  ADD KEY `CR` (`CR`),
  ADD KEY `class` (`class`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `idx_mapsn_class_video_indicateid` (`map_sn`,`class`,`video`,`indicate_id`),
  ADD KEY `sub_subject_id` (`sub_subject_id`),
  ADD KEY `idx_mapsn_class_indicateid` (`map_sn`,`class`,`indicate_id`),
  ADD KEY `blank_filling` (`blank_filling`),
  ADD KEY `node_file` (`node_file`),
  ADD KEY `external` (`external`),
  ADD KEY `node_creater` (`node_creater`);

--
-- 資料表索引 `map_node_20251105`
--
ALTER TABLE `map_node_20251105`
  ADD PRIMARY KEY (`node_sn`),
  ADD UNIQUE KEY `uk_mapsn_indicateid` (`map_sn`,`indicate_id`) USING BTREE,
  ADD KEY `subject` (`subject`),
  ADD KEY `DA` (`DA`),
  ADD KEY `prac` (`prac`),
  ADD KEY `video` (`video`),
  ADD KEY `teach` (`teach`),
  ADD KEY `indicate_name` (`indicate_name`) USING BTREE,
  ADD KEY `indicate_id` (`indicate_id`),
  ADD KEY `idx_indicateid_indicatename_class` (`indicate_id`,`indicate_name`,`class`) USING BTREE,
  ADD KEY `idx_indicateid_class_mapsn` (`indicate_id`,`class`,`map_sn`),
  ADD KEY `CR` (`CR`),
  ADD KEY `class` (`class`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `idx_mapsn_class_video_indicateid` (`map_sn`,`class`,`video`,`indicate_id`),
  ADD KEY `sub_subject_id` (`sub_subject_id`),
  ADD KEY `idx_mapsn_class_indicateid` (`map_sn`,`class`,`indicate_id`),
  ADD KEY `blank_filling` (`blank_filling`),
  ADD KEY `node_file` (`node_file`),
  ADD KEY `external` (`external`),
  ADD KEY `node_creater` (`node_creater`);
ALTER TABLE `map_node_20251105` ADD FULLTEXT KEY `indicate_name_2` (`indicate_name`);

--
-- 資料表索引 `map_node_20251223`
--
ALTER TABLE `map_node_20251223`
  ADD PRIMARY KEY (`node_sn`),
  ADD KEY `idx_map_sn_indicate_id` (`map_sn`,`indicate_id`);
ALTER TABLE `map_node_20251223` ADD FULLTEXT KEY `indicate_name_2` (`indicate_name`);

--
-- 資料表索引 `map_node_bk2`
--
ALTER TABLE `map_node_bk2`
  ADD PRIMARY KEY (`node_sn`),
  ADD UNIQUE KEY `uk_mapsn_indicateid` (`map_sn`,`indicate_id`),
  ADD KEY `subject` (`subject`),
  ADD KEY `DA` (`DA`),
  ADD KEY `prac` (`prac`),
  ADD KEY `video` (`video`),
  ADD KEY `teach` (`teach`),
  ADD KEY `indicate_name` (`indicate_name`) USING BTREE,
  ADD KEY `indicate_id` (`indicate_id`),
  ADD KEY `idx_indicateid_indicatename_class` (`indicate_id`,`indicate_name`,`class`) USING BTREE,
  ADD KEY `idx_indicateid_class_mapsn` (`indicate_id`,`class`,`map_sn`),
  ADD KEY `CR` (`CR`),
  ADD KEY `class` (`class`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `idx_mapsn_class_video_indicateid` (`map_sn`,`class`,`video`,`indicate_id`),
  ADD KEY `sub_subject_id` (`sub_subject_id`),
  ADD KEY `idx_mapsn_class_indicateid` (`map_sn`,`class`,`indicate_id`),
  ADD KEY `blank_filling` (`blank_filling`),
  ADD KEY `node_file` (`node_file`),
  ADD KEY `external` (`external`),
  ADD KEY `node_creater` (`node_creater`);

--
-- 資料表索引 `map_node_external`
--
ALTER TABLE `map_node_external`
  ADD PRIMARY KEY (`res_sn`),
  ADD KEY `indicate_id` (`indicate_id`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `subject_id` (`subject_id`),
  ADD KEY `res_type_name` (`res_type`),
  ADD KEY `public` (`public`),
  ADD KEY `creater` (`creater`),
  ADD KEY `map_sn_2` (`map_sn`,`indicate_id`);

--
-- 資料表索引 `map_node_file`
--
ALTER TABLE `map_node_file`
  ADD PRIMARY KEY (`map_node_file_sn`),
  ADD KEY `map_node_sn` (`map_node_sn`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `indicator` (`indicator`),
  ADD KEY `create_user` (`create_user`),
  ADD KEY `public` (`public`),
  ADD KEY `map_sn_2` (`map_sn`,`indicator`),
  ADD KEY `file_type` (`file_type`);

--
-- 資料表索引 `map_node_grade`
--
ALTER TABLE `map_node_grade`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `subject_id` (`subject_id`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `indicator` (`indicator`),
  ADD KEY `sub_indicator` (`sub_indicator`),
  ADD KEY `idx_subjectid_indicator_subindicator` (`subject_id`,`indicator`,`sub_indicator`),
  ADD KEY `idx_subjectid_indicator` (`subject_id`,`indicator`),
  ADD KEY `idx_mapsn_indicator` (`map_sn`,`indicator`),
  ADD KEY `idx_mapsn_indicator_subindicator` (`map_sn`,`indicator`,`sub_indicator`) USING BTREE,
  ADD KEY `idx_indicator_subindicator` (`indicator`,`sub_indicator`),
  ADD KEY `concat_indicator` (`concat_indicator`);

--
-- 資料表索引 `map_node_mapping`
--
ALTER TABLE `map_node_mapping`
  ADD PRIMARY KEY (`sn`),
  ADD UNIQUE KEY `uk_mapsn_snode` (`map_sn`,`snode_indicate_id`),
  ADD UNIQUE KEY `mapsn_bnode_snode` (`map_sn`,`bnode_indicate_id`,`snode_indicate_id`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `bnode_indicate_id` (`bnode_indicate_id`),
  ADD KEY `snode_indicate_id` (`snode_indicate_id`),
  ADD KEY `map_sn_2` (`map_sn`,`bnode_class`,`bnode_indicate_id`),
  ADD KEY `map_sn_3` (`map_sn`,`snode_class`,`snode_indicate_id`);

--
-- 資料表索引 `map_node_mapping_0521`
--
ALTER TABLE `map_node_mapping_0521`
  ADD PRIMARY KEY (`sn`),
  ADD UNIQUE KEY `uk_mapsn_snode` (`map_sn`,`snode_indicate_id`),
  ADD UNIQUE KEY `mapsn_bnode_snode` (`map_sn`,`bnode_indicate_id`,`snode_indicate_id`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `bnode_indicate_id` (`bnode_indicate_id`),
  ADD KEY `snode_indicate_id` (`snode_indicate_id`),
  ADD KEY `map_sn_2` (`map_sn`,`bnode_class`,`bnode_indicate_id`),
  ADD KEY `map_sn_3` (`map_sn`,`snode_class`,`snode_indicate_id`);

--
-- 資料表索引 `map_node_student_status`
--
ALTER TABLE `map_node_student_status`
  ADD PRIMARY KEY (`status_sn`),
  ADD UNIQUE KEY `uk_map_sn_userid` (`map_sn`,`user_id`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `idx_userid_mapsn` (`user_id`,`map_sn`),
  ADD KEY `recommended_mission` (`recommended_mission`);

--
-- 資料表索引 `math_laboratory_exam_record`
--
ALTER TABLE `math_laboratory_exam_record`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `math_laboratory_ibfk_1` (`user_id`);

--
-- 資料表索引 `message_fileattached`
--
ALTER TABLE `message_fileattached`
  ADD PRIMARY KEY (`file_sn`),
  ADD KEY `message_sn` (`message_sn`),
  ADD KEY `upload_server` (`upload_server`),
  ADD KEY `filetype` (`filetype`),
  ADD KEY `upload_userid` (`upload_userid`),
  ADD KEY `file_backup` (`file_backup`);

--
-- 資料表索引 `message_fileattached_20230316`
--
ALTER TABLE `message_fileattached_20230316`
  ADD PRIMARY KEY (`file_sn`),
  ADD KEY `message_sn` (`message_sn`),
  ADD KEY `upload_server` (`upload_server`),
  ADD KEY `filetype` (`filetype`),
  ADD KEY `upload_userid` (`upload_userid`),
  ADD KEY `file_backup` (`file_backup`);

--
-- 資料表索引 `message_master`
--
ALTER TABLE `message_master`
  ADD PRIMARY KEY (`msg_sn`),
  ADD KEY `touser_id` (`touser_id`),
  ADD KEY `msg_type` (`msg_type`),
  ADD KEY `create_user` (`create_user`),
  ADD KEY `attachefile` (`attachefile`),
  ADD KEY `read_mk` (`read_mk`),
  ADD KEY `delete_flag` (`delete_flag`);

--
-- 資料表索引 `message_response`
--
ALTER TABLE `message_response`
  ADD PRIMARY KEY (`response_sn`),
  ADD KEY `message_sn` (`message_sn`),
  ADD KEY `remsg_create_user` (`remsg_create_user`),
  ADD KEY `res_resmsgto_user` (`res_resmsgto_user`),
  ADD KEY `remsg_delete_flag` (`remsg_delete_flag`);

--
-- 資料表索引 `message_response_20230316`
--
ALTER TABLE `message_response_20230316`
  ADD PRIMARY KEY (`response_sn`),
  ADD KEY `message_sn` (`message_sn`),
  ADD KEY `remsg_create_user` (`remsg_create_user`),
  ADD KEY `res_resmsgto_user` (`res_resmsgto_user`),
  ADD KEY `remsg_delete_flag` (`remsg_delete_flag`);

--
-- 資料表索引 `mimes`
--
ALTER TABLE `mimes`
  ADD PRIMARY KEY (`filetype`);

--
-- 資料表索引 `mission_google_form`
--
ALTER TABLE `mission_google_form`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `form_type` (`form_type`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `city` (`city`),
  ADD KEY `org_name` (`org_name`),
  ADD KEY `grade` (`grade`),
  ADD KEY `class` (`class`),
  ADD KEY `stu_name` (`stu_name`),
  ADD KEY `start_time` (`start_time`),
  ADD KEY `create_time` (`create_time`),
  ADD KEY `seme` (`seme`),
  ADD KEY `stu_sex` (`stu_sex`),
  ADD KEY `mission_sn` (`mission_sn`),
  ADD KEY `org_id` (`org_id`);

--
-- 資料表索引 `mission_google_master`
--
ALTER TABLE `mission_google_master`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `used` (`used`);

--
-- 資料表索引 `mission_group_leader`
--
ALTER TABLE `mission_group_leader`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `group_id` (`group_id`),
  ADD KEY `mission_sn` (`mission_sn`),
  ADD KEY `is_used` (`is_used`);

--
-- 資料表索引 `mission_info`
--
ALTER TABLE `mission_info`
  ADD PRIMARY KEY (`mission_sn`),
  ADD KEY `unable` (`unable`),
  ADD KEY `semester` (`semester`),
  ADD KEY `history_mk` (`history_mk`),
  ADD KEY `teacher_id` (`teacher_id`),
  ADD KEY `exam_type` (`exam_type`),
  ADD KEY `target_type` (`target_type`),
  ADD KEY `cp_id` (`cp_id`),
  ADD KEY `self_practice` (`self_practice`),
  ADD KEY `create_date` (`create_date`),
  ADD KEY `subject_mission_share` (`subject_mission_share`),
  ADD KEY `start_time` (`start_time`),
  ADD KEY `mission_type` (`mission_type`),
  ADD KEY `share` (`share`),
  ADD KEY `copy_record` (`copy_record`),
  ADD KEY `class_type` (`class_type`),
  ADD KEY `subject_id` (`subject_id`),
  ADD KEY `is_auto_finish` (`is_auto_finish`),
  ADD KEY `end_mk` (`end_mk`),
  ADD KEY `can_view_answer` (`can_view_answer`),
  ADD KEY `mid` (`mid`);

--
-- 資料表索引 `mission_result`
--
ALTER TABLE `mission_result`
  ADD PRIMARY KEY (`result_sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `mission_sn` (`mission_sn`),
  ADD KEY `all_pass` (`all_pass`),
  ADD KEY `history_mk` (`history_mk`);

--
-- 資料表索引 `mission_stud_record`
--
ALTER TABLE `mission_stud_record`
  ADD PRIMARY KEY (`stu_mission_sn`),
  ADD KEY `mission_sn` (`mission_sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `semester` (`semester`),
  ADD KEY `is_all_fin` (`is_all_fin`),
  ADD KEY `is_get_extra_coin` (`is_get_extra_coin`);

--
-- 資料表索引 `mission_type`
--
ALTER TABLE `mission_type`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `mission_type_id` (`mission_type_id`) USING BTREE,
  ADD KEY `unable` (`unable`) USING BTREE;

--
-- 資料表索引 `mission_voice_recognition`
--
ALTER TABLE `mission_voice_recognition`
  ADD PRIMARY KEY (`voice_recognition_sn`);

--
-- 資料表索引 `monthly_student_usage`
--
ALTER TABLE `monthly_student_usage`
  ADD PRIMARY KEY (`sn`),
  ADD UNIQUE KEY `uk_org_user_grade_class` (`user_id`) USING BTREE;

--
-- 資料表索引 `monthly_student_usage_0806`
--
ALTER TABLE `monthly_student_usage_0806`
  ADD PRIMARY KEY (`sn`),
  ADD UNIQUE KEY `uk_org_user_grade_class` (`user_id`) USING BTREE;

--
-- 資料表索引 `monthly_student_usage_backup`
--
ALTER TABLE `monthly_student_usage_backup`
  ADD PRIMARY KEY (`sn`),
  ADD UNIQUE KEY `uk_date_org_user_grade_class` (`date`,`organization_id`,`user_id`,`grade`,`class`);

--
-- 資料表索引 `monthly_student_usage_bcakup0804`
--
ALTER TABLE `monthly_student_usage_bcakup0804`
  ADD PRIMARY KEY (`sn`),
  ADD UNIQUE KEY `uk_org_user_grade_class` (`organization_id`,`user_id`,`grade`,`class`) USING BTREE;

--
-- 資料表索引 `monthly_student_usage_rankings`
--
ALTER TABLE `monthly_student_usage_rankings`
  ADD PRIMARY KEY (`sn`);

--
-- 資料表索引 `new_class`
--
ALTER TABLE `new_class`
  ADD PRIMARY KEY (`new_class_sn`),
  ADD KEY `organization_id` (`organization_id`),
  ADD KEY `seme` (`seme`),
  ADD KEY `grade` (`grade`),
  ADD KEY `class_num` (`class_num`),
  ADD KEY `teacher_id` (`teacher_id`);

--
-- 資料表索引 `old_nodes`
--
ALTER TABLE `old_nodes`
  ADD PRIMARY KEY (`id`),
  ADD KEY `uuid` (`uuid`);

--
-- 資料表索引 `old_uploaded_filename`
--
ALTER TABLE `old_uploaded_filename`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `module` (`module`);

--
-- 資料表索引 `onversation_times`
--
ALTER TABLE `onversation_times`
  ADD PRIMARY KEY (`sn`);

--
-- 資料表索引 `openid_api_status_record`
--
ALTER TABLE `openid_api_status_record`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `openid_user_login_record_sn` (`openid_user_login_record_sn`),
  ADD KEY `api_type` (`api_type`),
  ADD KEY `http_status_code` (`http_status_code`);

--
-- 資料表索引 `openid_class_info`
--
ALTER TABLE `openid_class_info`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `openid_user_login_record_sn` (`openid_user_login_record_sn`),
  ADD KEY `grade` (`grade`),
  ADD KEY `class_no` (`class_no`),
  ADD KEY `school_id` (`school_id`),
  ADD KEY `year` (`year`),
  ADD KEY `semester` (`semester`);

--
-- 資料表索引 `openid_titles_info`
--
ALTER TABLE `openid_titles_info`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `openid_user_login_record_sn` (`openid_user_login_record_sn`);

--
-- 資料表索引 `openid_user_data_check_status`
--
ALTER TABLE `openid_user_data_check_status`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `openid_user_login_record_sn` (`openid_user_login_record_sn`),
  ADD KEY `openid_user_data_check_status_discription_sn` (`openid_user_data_check_status_discription_sn`);

--
-- 資料表索引 `openid_user_data_check_status_discription`
--
ALTER TABLE `openid_user_data_check_status_discription`
  ADD PRIMARY KEY (`sn`);

--
-- 資料表索引 `openid_user_info`
--
ALTER TABLE `openid_user_info`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `openid_user_login_record_sn` (`openid_user_login_record_sn`),
  ADD KEY `guid_token` (`guid_token`),
  ADD KEY `name` (`name`),
  ADD KEY `guid` (`guid`),
  ADD KEY `prefferd_name` (`prefferd_name`),
  ADD KEY `school_id` (`school_id`),
  ADD KEY `mentor_grade` (`mentor_grade`),
  ADD KEY `mentor_class` (`mentor_class`),
  ADD KEY `seat_no` (`seat_no`),
  ADD KEY `email` (`email`);

--
-- 資料表索引 `openid_user_login_record`
--
ALTER TABLE `openid_user_login_record`
  ADD PRIMARY KEY (`sn`);

--
-- 資料表索引 `openid_user_login_title`
--
ALTER TABLE `openid_user_login_title`
  ADD PRIMARY KEY (`sn`);

--
-- 資料表索引 `organization`
--
ALTER TABLE `organization`
  ADD PRIMARY KEY (`organization_id`),
  ADD KEY `type` (`type`),
  ADD KEY `used` (`used`),
  ADD KEY `city_code` (`city_code`),
  ADD KEY `idx_used_organizationid_address` (`used`,`organization_id`,`address`) USING BTREE,
  ADD KEY `name` (`name`),
  ADD KEY `class_name_type` (`class_name_type`),
  ADD KEY `openid_trust` (`openid_trust`),
  ADD KEY `location_terrain` (`location_terrain`);

--
-- 資料表索引 `organization_ace_setting`
--
ALTER TABLE `organization_ace_setting`
  ADD PRIMARY KEY (`sn`),
  ADD UNIQUE KEY `uk_organizationid_accesslevel_aceid` (`organization_id`,`access_level`,`module_id`),
  ADD KEY `organization_id` (`organization_id`),
  ADD KEY `access_level` (`access_level`),
  ADD KEY `ace_id` (`module_id`);

--
-- 資料表索引 `organization_Education_Administration`
--
ALTER TABLE `organization_Education_Administration`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `organization_id` (`organization_id`);

--
-- 資料表索引 `organization_exception`
--
ALTER TABLE `organization_exception`
  ADD PRIMARY KEY (`organization_id`),
  ADD KEY `report` (`report`),
  ADD KEY `basical_ability` (`basical_ability`),
  ADD KEY `rank` (`rank`);

--
-- 資料表索引 `organization_project_participation`
--
ALTER TABLE `organization_project_participation`
  ADD PRIMARY KEY (`sn`),
  ADD UNIQUE KEY `uk_organizationid_projectname` (`organization_id`,`project_name`),
  ADD KEY `organization_id` (`organization_id`),
  ADD KEY `project_name` (`project_name`);

--
-- 資料表索引 `organization_subject_group`
--
ALTER TABLE `organization_subject_group`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `level` (`level`),
  ADD KEY `year` (`year`),
  ADD KEY `subject_id` (`subject_id`),
  ADD KEY `organization_id` (`organization_id`);

--
-- 資料表索引 `paper_teacher`
--
ALTER TABLE `paper_teacher`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `teacher_user_id` (`teacher_user_id`);

--
-- 資料表索引 `prac_answer`
--
ALTER TABLE `prac_answer`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `prac_user_ind` (`user_id`,`indicator`) USING BTREE,
  ADD KEY `prac_ind` (`indicator`,`user_id`) USING BTREE,
  ADD KEY `sn` (`sn`,`user_id`) USING BTREE,
  ADD KEY `date` (`date`),
  ADD KEY `indicator` (`indicator`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `indicator_2` (`indicator`,`date`),
  ADD KEY `admin_report_rank` (`user_id`,`date`),
  ADD KEY `mission_sn` (`mission_sn`),
  ADD KEY `reward_mk` (`reward_mk`);

--
-- 資料表索引 `prac_blank_filling_itembank`
--
ALTER TABLE `prac_blank_filling_itembank`
  ADD PRIMARY KEY (`item_sn`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `used` (`used`),
  ADD KEY `indicator` (`indicator`),
  ADD KEY `map_sn_2` (`map_sn`,`indicator`);

--
-- 資料表索引 `prac_blank_filling_itembank_20240813`
--
ALTER TABLE `prac_blank_filling_itembank_20240813`
  ADD PRIMARY KEY (`item_sn`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `used` (`used`),
  ADD KEY `indicator` (`indicator`),
  ADD KEY `map_sn_2` (`map_sn`,`indicator`);

--
-- 資料表索引 `prac_blank_filling_record`
--
ALTER TABLE `prac_blank_filling_record`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `item_sn` (`item_sn`),
  ADD KEY `correct` (`correct`),
  ADD KEY `mission_sn` (`mission_sn`);

--
-- 資料表索引 `prac_options`
--
ALTER TABLE `prac_options`
  ADD PRIMARY KEY (`id`),
  ADD KEY `questions_id` (`questions_id`),
  ADD KEY `correct` (`correct`);

--
-- 資料表索引 `prac_questions`
--
ALTER TABLE `prac_questions`
  ADD PRIMARY KEY (`id`),
  ADD KEY `indicator` (`indicator`,`type`,`public`) USING BTREE,
  ADD KEY `type` (`type`),
  ADD KEY `public` (`public`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `indicator_2` (`indicator`),
  ADD KEY `map_sn_2` (`map_sn`,`indicator`),
  ADD KEY `difficulty` (`difficulty`);

--
-- 資料表索引 `prac_session_data`
--
ALTER TABLE `prac_session_data`
  ADD PRIMARY KEY (`session_id`),
  ADD KEY `user_id` (`user_id`);

--
-- 資料表索引 `prac_single_answer`
--
ALTER TABLE `prac_single_answer`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `prac_user_ind` (`user_id`,`indicator`) USING BTREE,
  ADD KEY `prac_ind` (`indicator`,`user_id`) USING BTREE,
  ADD KEY `sn` (`sn`,`user_id`) USING BTREE,
  ADD KEY `date` (`date`),
  ADD KEY `indicator` (`indicator`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `indicator_2` (`indicator`,`date`),
  ADD KEY `admin_report_rank` (`user_id`,`date`),
  ADD KEY `mission_sn` (`mission_sn`),
  ADD KEY `reward_mk` (`reward_mk`);

--
-- 資料表索引 `priori_itembank`
--
ALTER TABLE `priori_itembank`
  ADD PRIMARY KEY (`priori_item_sn`),
  ADD KEY `exam_range` (`exam_range`),
  ADD KEY `exam_grade` (`exam_grade`),
  ADD KEY `indicator_priori` (`indicator_priori`),
  ADD KEY `subject_name` (`subject_name`);

--
-- 資料表索引 `publisher`
--
ALTER TABLE `publisher`
  ADD PRIMARY KEY (`publisher_id`),
  ADD KEY `publisher` (`publisher`),
  ADD KEY `remark` (`remark`),
  ADD KEY `remark2` (`remark2`),
  ADD KEY `semeyear` (`semeyear`);

--
-- 資料表索引 `publisher_mapping`
--
ALTER TABLE `publisher_mapping`
  ADD PRIMARY KEY (`mapping_sn`),
  ADD UNIQUE KEY `uk_all` (`sems`,`subject_id`,`publisher_id`,`grade`,`unit`,`unit_name`,`indicator`,`sub_indicator`),
  ADD KEY `sems` (`sems`),
  ADD KEY `publisher_id` (`publisher_id`),
  ADD KEY `subject_id` (`subject_id`),
  ADD KEY `grade` (`grade`),
  ADD KEY `indicator` (`indicator`),
  ADD KEY `indicator_2` (`indicator`,`sub_indicator`),
  ADD KEY `sub_indicator` (`sub_indicator`),
  ADD KEY `unit` (`unit`),
  ADD KEY `unit_sub` (`unit_sub`),
  ADD KEY `updater` (`updater`),
  ADD KEY `idx_subjectid_indicator_subindicator` (`subject_id`,`indicator`,`sub_indicator`),
  ADD KEY `idx_subjectid_indicator` (`subject_id`,`indicator`);

--
-- 資料表索引 `publisher_mapping_0521`
--
ALTER TABLE `publisher_mapping_0521`
  ADD PRIMARY KEY (`mapping_sn`),
  ADD UNIQUE KEY `uk_all` (`sems`,`subject_id`,`publisher_id`,`grade`,`unit`,`unit_name`,`indicator`,`sub_indicator`),
  ADD KEY `sems` (`sems`),
  ADD KEY `publisher_id` (`publisher_id`),
  ADD KEY `subject_id` (`subject_id`),
  ADD KEY `grade` (`grade`),
  ADD KEY `indicator` (`indicator`),
  ADD KEY `indicator_2` (`indicator`,`sub_indicator`),
  ADD KEY `sub_indicator` (`sub_indicator`),
  ADD KEY `unit` (`unit`),
  ADD KEY `unit_sub` (`unit_sub`),
  ADD KEY `updater` (`updater`),
  ADD KEY `idx_subjectid_indicator_subindicator` (`subject_id`,`indicator`,`sub_indicator`),
  ADD KEY `idx_subjectid_indicator` (`subject_id`,`indicator`);

--
-- 資料表索引 `publisher_mapping_20240813`
--
ALTER TABLE `publisher_mapping_20240813`
  ADD PRIMARY KEY (`mapping_sn`),
  ADD UNIQUE KEY `uk_all` (`sems`,`subject_id`,`publisher_id`,`grade`,`unit`,`unit_name`,`indicator`,`sub_indicator`),
  ADD KEY `sems` (`sems`),
  ADD KEY `publisher_id` (`publisher_id`),
  ADD KEY `subject_id` (`subject_id`),
  ADD KEY `grade` (`grade`),
  ADD KEY `indicator` (`indicator`),
  ADD KEY `indicator_2` (`indicator`,`sub_indicator`),
  ADD KEY `sub_indicator` (`sub_indicator`),
  ADD KEY `unit` (`unit`),
  ADD KEY `unit_sub` (`unit_sub`),
  ADD KEY `updater` (`updater`),
  ADD KEY `idx_subjectid_indicator_subindicator` (`subject_id`,`indicator`,`sub_indicator`),
  ADD KEY `idx_subjectid_indicator` (`subject_id`,`indicator`);

--
-- 資料表索引 `publisher_mapping_20250207`
--
ALTER TABLE `publisher_mapping_20250207`
  ADD PRIMARY KEY (`mapping_sn`),
  ADD UNIQUE KEY `uk_all` (`sems`,`subject_id`,`publisher_id`,`grade`,`unit`,`unit_name`,`indicator`,`sub_indicator`),
  ADD KEY `sems` (`sems`),
  ADD KEY `publisher_id` (`publisher_id`),
  ADD KEY `subject_id` (`subject_id`),
  ADD KEY `grade` (`grade`),
  ADD KEY `indicator` (`indicator`),
  ADD KEY `indicator_2` (`indicator`,`sub_indicator`),
  ADD KEY `sub_indicator` (`sub_indicator`),
  ADD KEY `unit` (`unit`),
  ADD KEY `unit_sub` (`unit_sub`),
  ADD KEY `updater` (`updater`),
  ADD KEY `idx_subjectid_indicator_subindicator` (`subject_id`,`indicator`,`sub_indicator`),
  ADD KEY `idx_subjectid_indicator` (`subject_id`,`indicator`);

--
-- 資料表索引 `publisher_mapping_backup_20220608`
--
ALTER TABLE `publisher_mapping_backup_20220608`
  ADD PRIMARY KEY (`mapping_sn`),
  ADD KEY `sems` (`sems`),
  ADD KEY `publisher_id` (`publisher_id`),
  ADD KEY `subject_id` (`subject_id`),
  ADD KEY `grade` (`grade`),
  ADD KEY `indicator` (`indicator`),
  ADD KEY `indicator_2` (`indicator`,`sub_indicator`),
  ADD KEY `sub_indicator` (`sub_indicator`),
  ADD KEY `unit` (`unit`);

--
-- 資料表索引 `publisher_mapping_backup_20220628`
--
ALTER TABLE `publisher_mapping_backup_20220628`
  ADD PRIMARY KEY (`mapping_sn`),
  ADD KEY `sems` (`sems`),
  ADD KEY `publisher_id` (`publisher_id`),
  ADD KEY `subject_id` (`subject_id`),
  ADD KEY `grade` (`grade`),
  ADD KEY `indicator` (`indicator`),
  ADD KEY `indicator_2` (`indicator`,`sub_indicator`),
  ADD KEY `sub_indicator` (`sub_indicator`),
  ADD KEY `unit` (`unit`);

--
-- 資料表索引 `publisher_mapping_log`
--
ALTER TABLE `publisher_mapping_log`
  ADD PRIMARY KEY (`sn`);

--
-- 資料表索引 `questionaire_export_permission`
--
ALTER TABLE `questionaire_export_permission`
  ADD PRIMARY KEY (`sn`),
  ADD UNIQUE KEY `uk_accesslevel_formtype` (`access_level`,`form_type`),
  ADD KEY `access_level` (`access_level`),
  ADD KEY `form_type` (`form_type`);

--
-- 資料表索引 `question_main`
--
ALTER TABLE `question_main`
  ADD PRIMARY KEY (`num`),
  ADD KEY `c_question_num` (`c_question_num`),
  ADD KEY `c_answer` (`c_answer`);

--
-- 資料表索引 `remedial_class`
--
ALTER TABLE `remedial_class`
  ADD PRIMARY KEY (`class_sn`),
  ADD KEY `cp_id` (`cp_id`),
  ADD KEY `subject_id` (`subject_id`),
  ADD KEY `seme` (`seme`),
  ADD KEY `create_id` (`create_id`),
  ADD KEY `mission_sn` (`mission_sn`),
  ADD KEY `delete_mark` (`delete_mark`),
  ADD KEY `new_class_sn` (`new_class_sn`);

--
-- 資料表索引 `remedial_ftp_history`
--
ALTER TABLE `remedial_ftp_history`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `file_name` (`file_name`),
  ADD KEY `version` (`version`),
  ADD KEY `status` (`status`),
  ADD KEY `type` (`type`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `file_type` (`file_type`),
  ADD KEY `subject_id` (`subject_id`);

--
-- 資料表索引 `remedial_result_history`
--
ALTER TABLE `remedial_result_history`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `organization_id` (`organization_id`);

--
-- 資料表索引 `remedial_student`
--
ALTER TABLE `remedial_student`
  ADD PRIMARY KEY (`remedial_student_sn`),
  ADD KEY `class_sn` (`class_sn`),
  ADD KEY `new_class_agree` (`new_class_agree`),
  ADD KEY `student_id` (`student_id`);

--
-- 資料表索引 `remedial_teacher`
--
ALTER TABLE `remedial_teacher`
  ADD PRIMARY KEY (`remedial_teacher_sn`),
  ADD KEY `class_sn` (`class_sn`),
  ADD KEY `teacher_id` (`teacher_id`);

--
-- 資料表索引 `report_dailyusage`
--
ALTER TABLE `report_dailyusage`
  ADD PRIMARY KEY (`sn`),
  ADD UNIQUE KEY `uk_datetimelog_organizationid_grade_class` (`datetime_log`,`organization_id`,`grade`,`class`),
  ADD KEY `organization_id` (`organization_id`),
  ADD KEY `grade` (`grade`),
  ADD KEY `class` (`class`),
  ADD KEY `datetime_log` (`datetime_log`);

--
-- 資料表索引 `report_grad_student_dynamic`
--
ALTER TABLE `report_grad_student_dynamic`
  ADD PRIMARY KEY (`sn`),
  ADD UNIQUE KEY `uk_datetimelog_organizationid_mapsn_grade_class` (`datetime_log`,`organization_id`,`map_sn`,`grade`,`class`),
  ADD KEY `organization_id` (`organization_id`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `grade` (`grade`),
  ADD KEY `class` (`class`),
  ADD KEY `datetime_log` (`datetime_log`);

--
-- 資料表索引 `report_grad_student_indicate`
--
ALTER TABLE `report_grad_student_indicate`
  ADD PRIMARY KEY (`sn`),
  ADD UNIQUE KEY `uk_datetimelog_organizationid_mapsn_grade_class` (`datetime_log`,`organization_id`,`map_sn`,`grade`,`class`),
  ADD KEY `organization_id` (`organization_id`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `grade` (`grade`),
  ADD KEY `class` (`class`),
  ADD KEY `datetime_log` (`datetime_log`);

--
-- 資料表索引 `report_grad_student_node`
--
ALTER TABLE `report_grad_student_node`
  ADD PRIMARY KEY (`sn`),
  ADD UNIQUE KEY `uk_datetimelog_organizationid_mapsn_grade_class` (`datetime_log`,`organization_id`,`map_sn`,`grade`,`class`),
  ADD KEY `organization_id` (`organization_id`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `grade` (`grade`),
  ADD KEY `class` (`class`),
  ADD KEY `datetime_log` (`datetime_log`);

--
-- 資料表索引 `report_grad_student_practice`
--
ALTER TABLE `report_grad_student_practice`
  ADD PRIMARY KEY (`sn`),
  ADD UNIQUE KEY `uk_datetimelog_organizationid_mapsn_grade_class` (`datetime_log`,`organization_id`,`map_sn`,`grade`,`class`),
  ADD KEY `organization_id` (`organization_id`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `grade` (`grade`),
  ADD KEY `class` (`class`),
  ADD KEY `datetime_log` (`datetime_log`);

--
-- 資料表索引 `report_grad_student_robot`
--
ALTER TABLE `report_grad_student_robot`
  ADD PRIMARY KEY (`sn`),
  ADD UNIQUE KEY `uk_datetimelog_organizationid_mapsn_grade_class` (`datetime_log`,`organization_id`,`map_sn`,`grade`,`class`),
  ADD KEY `organization_id` (`organization_id`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `grade` (`grade`),
  ADD KEY `class` (`class`),
  ADD KEY `datetime_log` (`datetime_log`);

--
-- 資料表索引 `report_grad_student_unit`
--
ALTER TABLE `report_grad_student_unit`
  ADD PRIMARY KEY (`sn`),
  ADD UNIQUE KEY `uk_datetimelog_organizationid_mapsn_grade_class` (`datetime_log`,`organization_id`,`map_sn`,`grade`,`class`),
  ADD KEY `organization_id` (`organization_id`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `grade` (`grade`),
  ADD KEY `class` (`class`),
  ADD KEY `datetime_log` (`datetime_log`);

--
-- 資料表索引 `report_grad_student_video`
--
ALTER TABLE `report_grad_student_video`
  ADD PRIMARY KEY (`sn`),
  ADD UNIQUE KEY `uk_datetimelog_organizationid_mapsn_grade_class` (`datetime_log`,`organization_id`,`map_sn`,`grade`,`class`),
  ADD KEY `organization_id` (`organization_id`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `grade` (`grade`),
  ADD KEY `class` (`class`),
  ADD KEY `datetime_log` (`datetime_log`);

--
-- 資料表索引 `report_learningeffect_dynamic`
--
ALTER TABLE `report_learningeffect_dynamic`
  ADD PRIMARY KEY (`sn`),
  ADD UNIQUE KEY `uk_datetimelog_organizationid_mapsn_grade_class` (`datetime_log`,`organization_id`,`map_sn`,`grade`,`class`),
  ADD KEY `organization_id` (`organization_id`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `grade` (`grade`),
  ADD KEY `class` (`class`),
  ADD KEY `datetime_log` (`datetime_log`),
  ADD KEY `type` (`type`),
  ADD KEY `name` (`name`),
  ADD KEY `city_area` (`city_area`),
  ADD KEY `postcode` (`postcode`),
  ADD KEY `city_name` (`city_name`);

--
-- 資料表索引 `report_learningeffect_indicate`
--
ALTER TABLE `report_learningeffect_indicate`
  ADD PRIMARY KEY (`sn`),
  ADD UNIQUE KEY `uk_datetimelog_organizationid_mapsn_grade_class` (`datetime_log`,`organization_id`,`map_sn`,`grade`,`class`),
  ADD KEY `organization_id` (`organization_id`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `grade` (`grade`),
  ADD KEY `class` (`class`),
  ADD KEY `datetime_log` (`datetime_log`),
  ADD KEY `city_name` (`city_name`),
  ADD KEY `postcode` (`postcode`),
  ADD KEY `city_area` (`city_area`),
  ADD KEY `name` (`name`),
  ADD KEY `type` (`type`);

--
-- 資料表索引 `report_learningeffect_node`
--
ALTER TABLE `report_learningeffect_node`
  ADD PRIMARY KEY (`sn`),
  ADD UNIQUE KEY `uk_datetimelog_organizationid_mapsn_grade_class` (`datetime_log`,`organization_id`,`map_sn`,`grade`,`class`),
  ADD KEY `organization_id` (`organization_id`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `grade` (`grade`),
  ADD KEY `class` (`class`),
  ADD KEY `datetime_log` (`datetime_log`),
  ADD KEY `city_name` (`city_name`),
  ADD KEY `postcode` (`postcode`),
  ADD KEY `city_area` (`city_area`);

--
-- 資料表索引 `report_learningeffect_practice`
--
ALTER TABLE `report_learningeffect_practice`
  ADD PRIMARY KEY (`sn`),
  ADD UNIQUE KEY `uk_datetimelog_organizationid_mapsn_grade_class` (`datetime_log`,`organization_id`,`map_sn`,`grade`,`class`),
  ADD KEY `organization_id` (`organization_id`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `grade` (`grade`),
  ADD KEY `class` (`class`),
  ADD KEY `datetime_log` (`datetime_log`);

--
-- 資料表索引 `report_learningeffect_robot`
--
ALTER TABLE `report_learningeffect_robot`
  ADD PRIMARY KEY (`sn`),
  ADD UNIQUE KEY `uk_datetimelog_organizationid_mapsn_grade_class` (`datetime_log`,`organization_id`,`map_sn`,`grade`,`class`),
  ADD KEY `organization_id` (`organization_id`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `grade` (`grade`),
  ADD KEY `class` (`class`),
  ADD KEY `datetime_log` (`datetime_log`);

--
-- 資料表索引 `report_learningeffect_unit`
--
ALTER TABLE `report_learningeffect_unit`
  ADD PRIMARY KEY (`sn`),
  ADD UNIQUE KEY `uk_datetimelog_organizationid_mapsn_grade_class` (`datetime_log`,`organization_id`,`map_sn`,`grade`,`class`),
  ADD KEY `organization_id` (`organization_id`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `grade` (`grade`),
  ADD KEY `class` (`class`),
  ADD KEY `datetime_log` (`datetime_log`);

--
-- 資料表索引 `report_learningeffect_video`
--
ALTER TABLE `report_learningeffect_video`
  ADD PRIMARY KEY (`sn`),
  ADD UNIQUE KEY `uk_datetimelog_organizationid_mapsn_grade_class` (`datetime_log`,`organization_id`,`map_sn`,`grade`,`class`),
  ADD KEY `organization_id` (`organization_id`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `grade` (`grade`),
  ADD KEY `class` (`class`),
  ADD KEY `datetime_log` (`datetime_log`),
  ADD KEY `datetime_log_2` (`datetime_log`,`map_sn`),
  ADD KEY `map_sn_2` (`map_sn`,`datetime_log`);

--
-- 資料表索引 `report_workshop_effect`
--
ALTER TABLE `report_workshop_effect`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `organization_id` (`organization_id`);

--
-- 資料表索引 `resource_choose_hint`
--
ALTER TABLE `resource_choose_hint`
  ADD PRIMARY KEY (`id`);

--
-- 資料表索引 `resource_exam_annex`
--
ALTER TABLE `resource_exam_annex`
  ADD PRIMARY KEY (`id`);

--
-- 資料表索引 `resource_filling_answer`
--
ALTER TABLE `resource_filling_answer`
  ADD PRIMARY KEY (`id`);

--
-- 資料表索引 `robot_config`
--
ALTER TABLE `robot_config`
  ADD PRIMARY KEY (`sn`),
  ADD UNIQUE KEY `name` (`name`);

--
-- 資料表索引 `role_carrer_info`
--
ALTER TABLE `role_carrer_info`
  ADD PRIMARY KEY (`carrer_id`);

--
-- 資料表索引 `role_coin_history`
--
ALTER TABLE `role_coin_history`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `action` (`action`),
  ADD KEY `student_id` (`student_id`),
  ADD KEY `mission_sn` (`mission_sn`),
  ADD KEY `rule_name` (`rule_name`),
  ADD KEY `coin_type` (`coin_type`),
  ADD KEY `video_note_sn` (`video_note_sn`),
  ADD KEY `operator` (`operator`) USING BTREE,
  ADD KEY `idx_coin_opr_ts_sid` (`operator`,`timestamp`,`student_id`),
  ADD KEY `idx_rch_operator_timestamp_studentid` (`operator`,`timestamp`,`student_id`);

--
-- 資料表索引 `role_coin_history_20230316`
--
ALTER TABLE `role_coin_history_20230316`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `action` (`action`),
  ADD KEY `student_id` (`student_id`),
  ADD KEY `mission_sn` (`mission_sn`),
  ADD KEY `rule_name` (`rule_name`),
  ADD KEY `coin_type` (`coin_type`),
  ADD KEY `video_note_sn` (`video_note_sn`),
  ADD KEY `operator` (`operator`) USING BTREE;

--
-- 資料表索引 `role_coin_rule`
--
ALTER TABLE `role_coin_rule`
  ADD PRIMARY KEY (`rule_sn`) USING BTREE,
  ADD KEY `reward_name` (`rule_name`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `rule_type` (`rule_type`),
  ADD KEY `display` (`display`);

--
-- 資料表索引 `role_coin_total`
--
ALTER TABLE `role_coin_total`
  ADD PRIMARY KEY (`coin_sn`),
  ADD UNIQUE KEY `uk_studentid_teacherid` (`student_id`,`teacher_id`) USING BTREE,
  ADD KEY `student_id` (`student_id`),
  ADD KEY `teacher_id` (`teacher_id`);

--
-- 資料表索引 `role_food_item`
--
ALTER TABLE `role_food_item`
  ADD PRIMARY KEY (`food_id`),
  ADD KEY `food_type` (`food_type`);

--
-- 資料表索引 `role_info`
--
ALTER TABLE `role_info`
  ADD PRIMARY KEY (`sn`),
  ADD UNIQUE KEY `uk_stud_id` (`stud_id`),
  ADD KEY `carrer_id` (`carrer_id`),
  ADD KEY `organization_id` (`organization_id`),
  ADD KEY `hat` (`hat`),
  ADD KEY `clothes` (`clothes`),
  ADD KEY `shoes` (`shoes`),
  ADD KEY `carrer_equipment` (`carrer_equipment`),
  ADD KEY `coin` (`coin`);

--
-- 資料表索引 `role_prop_item`
--
ALTER TABLE `role_prop_item`
  ADD PRIMARY KEY (`prop_id`),
  ADD KEY `prop_type` (`prop_type`);

--
-- 資料表索引 `role_reward_way`
--
ALTER TABLE `role_reward_way`
  ADD PRIMARY KEY (`reward_sn`),
  ADD KEY `program_id` (`program_id`);

--
-- 資料表索引 `role_reword_card_item`
--
ALTER TABLE `role_reword_card_item`
  ADD PRIMARY KEY (`reward_card_id`),
  ADD KEY `reward_card_type` (`reward_card_type`);

--
-- 資料表索引 `school_class_info`
--
ALTER TABLE `school_class_info`
  ADD PRIMARY KEY (`class_sn`),
  ADD UNIQUE KEY `uk_semester_organizationid_grade_classnumber` (`semester`,`organization_id`,`grade`,`class_number`) USING BTREE,
  ADD KEY `semester` (`semester`),
  ADD KEY `organization_id` (`organization_id`),
  ADD KEY `grade` (`grade`),
  ADD KEY `updater` (`updater`),
  ADD KEY `class_number` (`class_number`),
  ADD KEY `school_type` (`high_school_dept_mapping_sn`) USING BTREE;

--
-- 資料表索引 `seme_grad_student`
--
ALTER TABLE `seme_grad_student`
  ADD PRIMARY KEY (`sn`),
  ADD UNIQUE KEY `uk_seme_uid` (`seme_year`,`uid`),
  ADD UNIQUE KEY `uk_seme_studid` (`seme_year`,`stud_id`) USING BTREE,
  ADD KEY `organization_id` (`organization_id`),
  ADD KEY `next_organization_id` (`next_organization_id`),
  ADD KEY `stud_id` (`stud_id`);

--
-- 資料表索引 `seme_group`
--
ALTER TABLE `seme_group`
  ADD PRIMARY KEY (`group_id`),
  ADD KEY `teacher_id` (`teacher_id`),
  ADD KEY `is_used` (`is_used`);

--
-- 資料表索引 `seme_publisher`
--
ALTER TABLE `seme_publisher`
  ADD PRIMARY KEY (`sn`),
  ADD UNIQUE KEY `uk_sems_organizationid_subjectid_publisherid_grade` (`sems`,`organization_id`,`subject_id`,`publisher_id`,`grade`),
  ADD KEY `sems` (`sems`),
  ADD KEY `publisher_id` (`publisher_id`),
  ADD KEY `organization_id` (`organization_id`),
  ADD KEY `subject_id` (`subject_id`),
  ADD KEY `grade` (`grade`),
  ADD KEY `create_id` (`create_id`);

--
-- 資料表索引 `seme_publisher_tmp`
--
ALTER TABLE `seme_publisher_tmp`
  ADD PRIMARY KEY (`sn`),
  ADD UNIQUE KEY `uk_sems_organizationid_subjectid_publisherid_grade` (`sems`,`organization_id`,`subject_id`,`publisher_id`,`grade`),
  ADD KEY `sems` (`sems`),
  ADD KEY `publisher_id` (`publisher_id`),
  ADD KEY `organization_id` (`organization_id`),
  ADD KEY `subject_id` (`subject_id`),
  ADD KEY `grade` (`grade`);

--
-- 資料表索引 `seme_student`
--
ALTER TABLE `seme_student`
  ADD PRIMARY KEY (`sn`),
  ADD UNIQUE KEY `uk_seme_uid` (`seme_year_seme`,`uid`) USING BTREE,
  ADD UNIQUE KEY `uk_seme_studid` (`seme_year_seme`,`stud_id`) USING BTREE,
  ADD KEY `organization_id` (`organization_id`),
  ADD KEY `CON_userid` (`stud_id`),
  ADD KEY `grade` (`grade`),
  ADD KEY `class` (`class`),
  ADD KEY `seme_year_seme` (`seme_year_seme`),
  ADD KEY `idx_seme_organizationid` (`seme_year_seme`,`organization_id`) USING BTREE,
  ADD KEY `idx_seme_organizationid_grade_class` (`seme_year_seme`,`organization_id`,`grade`,`class`),
  ADD KEY `idx_organizationid_grade_class` (`organization_id`,`grade`,`class`) USING BTREE,
  ADD KEY `idx_seme_student` (`stud_id`,`seme_year_seme`,`seme_num`);

--
-- 資料表索引 `seme_student_rebackschool`
--
ALTER TABLE `seme_student_rebackschool`
  ADD PRIMARY KEY (`sn`),
  ADD UNIQUE KEY `uk_seme_uid` (`seme_year`,`uid`),
  ADD UNIQUE KEY `uk_seme_studid` (`seme_year`,`stud_id`) USING BTREE,
  ADD KEY `organization_id` (`organization_id`),
  ADD KEY `next_organization_id` (`next_organization_id`),
  ADD KEY `stud_id` (`stud_id`),
  ADD KEY `update_id` (`update_id`);

--
-- 資料表索引 `seme_student_tmp`
--
ALTER TABLE `seme_student_tmp`
  ADD PRIMARY KEY (`sn`),
  ADD UNIQUE KEY `uk_seme_uid` (`seme_year_seme`,`uid`),
  ADD UNIQUE KEY `uk_seme_studid` (`seme_year_seme`,`stud_id`) USING BTREE,
  ADD KEY `organization_id` (`organization_id`),
  ADD KEY `CON_userid` (`stud_id`);

--
-- 資料表索引 `seme_teacher_subject`
--
ALTER TABLE `seme_teacher_subject`
  ADD PRIMARY KEY (`sn`) USING BTREE,
  ADD UNIQUE KEY `uk_seme_teacherid_grade_class_subjectmapping` (`seme_year_seme`,`teacher_id`,`grade`,`class`,`subject_mapping`) USING BTREE,
  ADD KEY `seme_year_seme` (`seme_year_seme`),
  ADD KEY `organization_id` (`organization_id`),
  ADD KEY `subject_id` (`subject_id`),
  ADD KEY `teacher_id` (`teacher_id`),
  ADD KEY `grade` (`grade`),
  ADD KEY `class` (`class`),
  ADD KEY `seme_year_seme_3` (`seme_year_seme`,`organization_id`) USING BTREE,
  ADD KEY `subject_mapping` (`subject_mapping`);

--
-- 資料表索引 `seme_teacher_subject_mapping`
--
ALTER TABLE `seme_teacher_subject_mapping`
  ADD PRIMARY KEY (`mapping_sn`),
  ADD UNIQUE KEY `uk_organizationid_subjectname` (`organization_id`,`subject_name`) USING BTREE,
  ADD KEY `organization_id` (`organization_id`),
  ADD KEY `subject_name` (`subject_name`);

--
-- 資料表索引 `seme_teacher_subject_tmp`
--
ALTER TABLE `seme_teacher_subject_tmp`
  ADD PRIMARY KEY (`sn`) USING BTREE,
  ADD UNIQUE KEY `uk_seme_teacherid_grade_class_subjectmapping` (`seme_year_seme`,`teacher_id`,`grade`,`class`,`subject_mapping`) USING BTREE,
  ADD KEY `seme_year_seme` (`seme_year_seme`),
  ADD KEY `organization_id` (`organization_id`),
  ADD KEY `subject_id` (`subject_id`),
  ADD KEY `teacher_id` (`teacher_id`),
  ADD KEY `grade` (`grade`),
  ADD KEY `class` (`class`);

--
-- 資料表索引 `seme_teacher_subject_tmp_0208`
--
ALTER TABLE `seme_teacher_subject_tmp_0208`
  ADD PRIMARY KEY (`sn`) USING BTREE,
  ADD UNIQUE KEY `uk_seme_teacherid_grade_class_subjectid` (`seme_year_seme`,`teacher_id`,`grade`,`class`,`subject_id`),
  ADD KEY `seme_year_seme` (`seme_year_seme`),
  ADD KEY `organization_id` (`organization_id`),
  ADD KEY `subject_id` (`subject_id`),
  ADD KEY `teacher_id` (`teacher_id`),
  ADD KEY `grade` (`grade`),
  ADD KEY `class` (`class`);

--
-- 資料表索引 `srl_calendar`
--
ALTER TABLE `srl_calendar`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `all_day` (`all_day`),
  ADD KEY `mission_sn` (`mission_sn`),
  ADD KEY `is_done` (`is_done`),
  ADD KEY `is_delete` (`is_delete`),
  ADD KEY `eventColor` (`eventColor`);

--
-- 資料表索引 `subject`
--
ALTER TABLE `subject`
  ADD PRIMARY KEY (`subject_id`),
  ADD KEY `display` (`display`),
  ADD KEY `subject_groupings_id` (`subject_groupings_id`),
  ADD KEY `name` (`name`);

--
-- 資料表索引 `subject_0903`
--
ALTER TABLE `subject_0903`
  ADD PRIMARY KEY (`subject_id`),
  ADD KEY `display` (`display`),
  ADD KEY `subject_groupings_id` (`subject_groupings_id`),
  ADD KEY `name` (`name`);

--
-- 資料表索引 `subject_acnt`
--
ALTER TABLE `subject_acnt`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `subject` (`subject`),
  ADD KEY `acnt` (`acnt`);

--
-- 資料表索引 `subject_bk`
--
ALTER TABLE `subject_bk`
  ADD PRIMARY KEY (`subject_id`);

--
-- 資料表索引 `subject_groupings`
--
ALTER TABLE `subject_groupings`
  ADD PRIMARY KEY (`subject_groupings_id`),
  ADD KEY `subject_id` (`subject_id`);

--
-- 資料表索引 `subject_optgroups`
--
ALTER TABLE `subject_optgroups`
  ADD PRIMARY KEY (`subject_optgroups_id`);

--
-- 資料表索引 `sub_subject`
--
ALTER TABLE `sub_subject`
  ADD PRIMARY KEY (`sub_subject_id`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `subject_id` (`subject_id`);

--
-- 資料表索引 `system_user`
--
ALTER TABLE `system_user`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `user_id` (`user_id`);

--
-- 資料表索引 `talper_custom_agent`
--
ALTER TABLE `talper_custom_agent`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `idx_user_id` (`user_id`),
  ADD KEY `idx_creation_time` (`creation_time` DESC),
  ADD KEY `idx_user_creation` (`user_id`,`creation_time` DESC),
  ADD KEY `idx_model_name` (`model_name`),
  ADD KEY `idx_status_user` (`status`,`user_id`);

--
-- 資料表索引 `talper_custom_agent_files`
--
ALTER TABLE `talper_custom_agent_files`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `idx_agent_sn` (`agent_sn`),
  ADD KEY `idx_upload_time` (`upload_time` DESC),
  ADD KEY `idx_agent_upload` (`agent_sn`,`upload_time` DESC),
  ADD KEY `idx_file_path` (`file_path`(255));

--
-- 資料表索引 `talper_custom_agent_test_history`
--
ALTER TABLE `talper_custom_agent_test_history`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `idx_agent_sn` (`agent_sn`),
  ADD KEY `idx_creation_time` (`creation_time` DESC),
  ADD KEY `idx_agent_creation` (`agent_sn`,`creation_time`),
  ADD KEY `idx_role` (`role`);

--
-- 資料表索引 `teacher_class`
--
ALTER TABLE `teacher_class`
  ADD PRIMARY KEY (`teacher_class_sn`),
  ADD KEY `status` (`status`),
  ADD KEY `teacher_id` (`teacher_id`),
  ADD KEY `organization_id` (`organization_id`),
  ADD KEY `seme` (`seme`),
  ADD KEY `create_id` (`create_id`);

--
-- 資料表索引 `teacher_class_member`
--
ALTER TABLE `teacher_class_member`
  ADD PRIMARY KEY (`teacher_class_member_sn`),
  ADD KEY `class_sn` (`class_sn`),
  ADD KEY `teacher_agree` (`teacher_agree`),
  ADD KEY `student_id` (`student_id`);

--
-- 資料表索引 `triangle_class_conclusion`
--
ALTER TABLE `triangle_class_conclusion`
  ADD PRIMARY KEY (`conclusion_sn`);

--
-- 資料表索引 `triangle_conclusion`
--
ALTER TABLE `triangle_conclusion`
  ADD PRIMARY KEY (`conclusion_id`),
  ADD KEY `question_sn` (`question_sn`);

--
-- 資料表索引 `triangle_conclusion_proofs`
--
ALTER TABLE `triangle_conclusion_proofs`
  ADD PRIMARY KEY (`proof_sn`),
  ADD KEY `conclusion_id` (`conclusion_id`);

--
-- 資料表索引 `triangle_proof`
--
ALTER TABLE `triangle_proof`
  ADD PRIMARY KEY (`proof_id`);

--
-- 資料表索引 `triangle_thought`
--
ALTER TABLE `triangle_thought`
  ADD PRIMARY KEY (`thought_id`);

--
-- 資料表索引 `triangle_thought_comment`
--
ALTER TABLE `triangle_thought_comment`
  ADD PRIMARY KEY (`comment_sn`);

--
-- 資料表索引 `triangle_thought_proofs`
--
ALTER TABLE `triangle_thought_proofs`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `thought_id` (`thought_id`),
  ADD KEY `proof_id` (`proof_id`);

--
-- 資料表索引 `users_pwd`
--
ALTER TABLE `users_pwd`
  ADD PRIMARY KEY (`id`);

--
-- 資料表索引 `user_access`
--
ALTER TABLE `user_access`
  ADD PRIMARY KEY (`access_level`),
  ADD KEY `access_title` (`access_title`),
  ADD KEY `block_id` (`block_id`);

--
-- 資料表索引 `user_account_change`
--
ALTER TABLE `user_account_change`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `new_user_account` (`new_user_account`),
  ADD KEY `ori_user_account` (`ori_user_account`);

--
-- 資料表索引 `user_account_history`
--
ALTER TABLE `user_account_history`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `ori_access_level` (`ori_access_level`),
  ADD KEY `ori_organization_id` (`ori_organization_id`);

--
-- 資料表索引 `user_activity`
--
ALTER TABLE `user_activity`
  ADD PRIMARY KEY (`sn`),
  ADD UNIQUE KEY `uk_date_organizationid` (`date`,`organization_id`) USING BTREE,
  ADD KEY `organization_id` (`organization_id`);

--
-- 資料表索引 `user_combine_history`
--
ALTER TABLE `user_combine_history`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `connect_user_id` (`connect_user_id`),
  ADD KEY `by_user_id` (`by_user_id`),
  ADD KEY `type` (`type`);

--
-- 資料表索引 `user_entering_interactive_learning_log`
--
ALTER TABLE `user_entering_interactive_learning_log`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `organization_id` (`organization_id`),
  ADD KEY `its_sn` (`its_sn`);

--
-- 資料表索引 `user_family`
--
ALTER TABLE `user_family`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `fuser_id` (`fuser_id`),
  ADD KEY `organization_id` (`organization_id`),
  ADD KEY `create_userid` (`create_userid`);

--
-- 資料表索引 `user_feedback`
--
ALTER TABLE `user_feedback`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `phone` (`phone`),
  ADD KEY `chk1` (`chk1`),
  ADD KEY `create_user` (`create_user`),
  ADD KEY `chk2` (`chk2`),
  ADD KEY `chk3` (`chk3`),
  ADD KEY `file_backup` (`file_backup`),
  ADD KEY `re_statu` (`re_statu`),
  ADD KEY `re_name` (`re_name`),
  ADD KEY `upload_server` (`upload_server`),
  ADD KEY `create_date` (`create_date`),
  ADD KEY `have_read` (`have_read`),
  ADD KEY `chk3_2` (`chk3`);

--
-- 資料表索引 `user_games_play_time`
--
ALTER TABLE `user_games_play_time`
  ADD PRIMARY KEY (`sn`),
  ADD UNIQUE KEY `uk_apiclientsn_userid_date` (`api_client_sn`,`user_id`,`date`) USING BTREE,
  ADD KEY `user_id` (`user_id`),
  ADD KEY `date` (`date`),
  ADD KEY `api_client_sn` (`api_client_sn`);

--
-- 資料表索引 `user_group`
--
ALTER TABLE `user_group`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `group_id` (`group_id`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `create_date` (`create_date`),
  ADD KEY `organization_id` (`organization_id`),
  ADD KEY `role_sn` (`role_sn`),
  ADD KEY `group_leader` (`group_leader`);

--
-- 資料表索引 `user_group_20230316`
--
ALTER TABLE `user_group_20230316`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `group_id` (`group_id`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `create_date` (`create_date`),
  ADD KEY `organization_id` (`organization_id`),
  ADD KEY `role_sn` (`role_sn`),
  ADD KEY `group_leader` (`group_leader`);

--
-- 資料表索引 `user_info`
--
ALTER TABLE `user_info`
  ADD PRIMARY KEY (`uid`),
  ADD UNIQUE KEY `uk_user_id` (`user_id`) USING BTREE,
  ADD UNIQUE KEY `uk_organizationid_userid` (`organization_id`,`user_id`),
  ADD UNIQUE KEY `uk_user_account` (`user_account`),
  ADD KEY `city_code` (`city_code`),
  ADD KEY `organization_id` (`organization_id`),
  ADD KEY `idx_userid_grade_class` (`user_id`,`grade`,`class`) USING BTREE,
  ADD KEY `idx_organizationid_grade_class` (`organization_id`,`grade`,`class`),
  ADD KEY `user_name` (`uname`) USING BTREE,
  ADD KEY `priori_name` (`priori_name`),
  ADD KEY `user_regdate` (`user_regdate`),
  ADD KEY `update_id` (`update_id`),
  ADD KEY `openid_sub` (`OpenID_sub`),
  ADD KEY `hash_guid` (`hash_guid`),
  ADD KEY `email` (`email`),
  ADD KEY `class_sn` (`class_sn`),
  ADD KEY `sex` (`sex`),
  ADD KEY `used` (`used`);

--
-- 資料表索引 `user_info_temp`
--
ALTER TABLE `user_info_temp`
  ADD PRIMARY KEY (`uid`),
  ADD UNIQUE KEY `uk_user_id` (`user_id`),
  ADD KEY `seme` (`seme`);

--
-- 資料表索引 `user_login_log`
--
ALTER TABLE `user_login_log`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `starttimestamp` (`loginstamp`),
  ADD KEY `organization_id` (`organization_id`) USING BTREE,
  ADD KEY `idx_organizationid_loginstamp` (`organization_id`,`loginstamp`),
  ADD KEY `idx_loginstamp_organizationid` (`loginstamp`,`organization_id`),
  ADD KEY `login_from` (`login_from`),
  ADD KEY `update_time` (`update_time`),
  ADD KEY `update_time_2` (`update_time`);

--
-- 資料表索引 `user_login_log_status`
--
ALTER TABLE `user_login_log_status`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `starttimestamp` (`loginstamp`),
  ADD KEY `login_status` (`login_status`);

--
-- 資料表索引 `user_map_permission`
--
ALTER TABLE `user_map_permission`
  ADD PRIMARY KEY (`sn`),
  ADD UNIQUE KEY `uk_userid_mapsn` (`user_id`,`map_sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `updater` (`updater`);

--
-- 資料表索引 `user_map_permission_log`
--
ALTER TABLE `user_map_permission_log`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `creator` (`updater_id`),
  ADD KEY `action_type` (`action_type`),
  ADD KEY `user_account` (`user_account`),
  ADD KEY `updater_account` (`updater_account`);

--
-- 資料表索引 `user_past_password`
--
ALTER TABLE `user_past_password`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `pass` (`pass`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `modifier` (`modifier`),
  ADD KEY `confirmation` (`confirmation`);

--
-- 資料表索引 `user_person_config`
--
ALTER TABLE `user_person_config`
  ADD PRIMARY KEY (`user_id`),
  ADD KEY `item_card_or_list` (`item_card_or_list`),
  ADD KEY `calendar` (`calendar`);

--
-- 資料表索引 `user_play_game_log`
--
ALTER TABLE `user_play_game_log`
  ADD PRIMARY KEY (`game_log_sn`),
  ADD KEY `api_client_sn` (`api_client_sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `organization_id` (`organization_id`);

--
-- 資料表索引 `user_recovery_record`
--
ALTER TABLE `user_recovery_record`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `confirmation` (`confirmation`),
  ADD KEY `token` (`token`),
  ADD KEY `user_id` (`user_id`);

--
-- 資料表索引 `user_status`
--
ALTER TABLE `user_status`
  ADD PRIMARY KEY (`user_id`),
  ADD KEY `access_level` (`access_level`),
  ADD KEY `last_action_time` (`last_action_time`),
  ADD KEY `unique_id` (`unique_id`),
  ADD KEY `Enabled` (`Enabled`),
  ADD KEY `video_status` (`video_status`),
  ADD KEY `email_verify` (`email_verify`),
  ADD KEY `guide_step` (`guide_step`),
  ADD KEY `idx_user_status_level` (`user_id`,`access_level`);

--
-- 資料表索引 `user_status_log`
--
ALTER TABLE `user_status_log`
  ADD PRIMARY KEY (`log_sn`,`record_time`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `record_time` (`record_time`),
  ADD KEY `action` (`action`),
  ADD KEY `update_id` (`update_id`),
  ADD KEY `organization_id` (`organization_id`),
  ADD KEY `update_id_2` (`update_id`);

--
-- 資料表索引 `user_third_party`
--
ALTER TABLE `user_third_party`
  ADD PRIMARY KEY (`user_id`),
  ADD KEY `create_by_third` (`create_by_third`),
  ADD KEY `cloud_sub` (`cloud_sub`),
  ADD KEY `ck_sub` (`ck_sub`);

--
-- 資料表索引 `video_concept_item`
--
ALTER TABLE `video_concept_item`
  ADD PRIMARY KEY (`video_item_sn`),
  ADD KEY `indicator` (`indicator`),
  ADD KEY `cs_id` (`cs_id`),
  ADD KEY `create_user_id` (`create_user_id`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `item_num` (`item_num`),
  ADD KEY `public` (`public`),
  ADD KEY `end_mk` (`end_mk`),
  ADD KEY `video_version` (`video_version`),
  ADD KEY `map_sn_2` (`map_sn`,`indicator`);

--
-- 資料表索引 `video_concept_item3`
--
ALTER TABLE `video_concept_item3`
  ADD PRIMARY KEY (`video_item_sn`),
  ADD KEY `indicator` (`indicator`),
  ADD KEY `cs_id` (`cs_id`),
  ADD KEY `create_user_id` (`create_user_id`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `item_num` (`item_num`),
  ADD KEY `public` (`public`),
  ADD KEY `end_mk` (`end_mk`),
  ADD KEY `video_version` (`video_version`);

--
-- 資料表索引 `video_concept_item_0730`
--
ALTER TABLE `video_concept_item_0730`
  ADD PRIMARY KEY (`video_item_sn`),
  ADD KEY `indicator` (`indicator`),
  ADD KEY `cs_id` (`cs_id`),
  ADD KEY `create_user_id` (`create_user_id`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `item_num` (`item_num`),
  ADD KEY `public` (`public`),
  ADD KEY `end_mk` (`end_mk`),
  ADD KEY `video_version` (`video_version`),
  ADD KEY `map_sn_2` (`map_sn`,`indicator`);

--
-- 資料表索引 `video_concept_item_20220427`
--
ALTER TABLE `video_concept_item_20220427`
  ADD PRIMARY KEY (`video_item_sn`),
  ADD KEY `indicator` (`indicator`),
  ADD KEY `cs_id` (`cs_id`),
  ADD KEY `create_user_id` (`create_user_id`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `item_num` (`item_num`),
  ADD KEY `public` (`public`),
  ADD KEY `end_mk` (`end_mk`),
  ADD KEY `video_version` (`video_version`);

--
-- 資料表索引 `video_concept_item_20220428`
--
ALTER TABLE `video_concept_item_20220428`
  ADD PRIMARY KEY (`video_item_sn`),
  ADD KEY `indicator` (`indicator`),
  ADD KEY `cs_id` (`cs_id`),
  ADD KEY `create_user_id` (`create_user_id`),
  ADD KEY `map_sn` (`map_sn`),
  ADD KEY `item_num` (`item_num`),
  ADD KEY `public` (`public`),
  ADD KEY `end_mk` (`end_mk`),
  ADD KEY `video_version` (`video_version`);

--
-- 資料表索引 `video_concept_item_plus`
--
ALTER TABLE `video_concept_item_plus`
  ADD PRIMARY KEY (`question_sn`),
  ADD KEY `video_item_sn` (`video_item_sn`),
  ADD KEY `create_user_id` (`create_user_id`),
  ADD KEY `item_num` (`item_num`),
  ADD KEY `video_id` (`video_id`);

--
-- 資料表索引 `video_concept_item_plus_bk`
--
ALTER TABLE `video_concept_item_plus_bk`
  ADD PRIMARY KEY (`question_sn`),
  ADD KEY `CON_video_item_idx` (`video_item_sn`),
  ADD KEY `video_item_sn` (`video_item_sn`),
  ADD KEY `create_user_id` (`create_user_id`),
  ADD KEY `item_num` (`item_num`),
  ADD KEY `video_id` (`video_id`),
  ADD KEY `item_num_2` (`item_num`);

--
-- 資料表索引 `video_concept_stream_error`
--
ALTER TABLE `video_concept_stream_error`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `indicator` (`indicator`);

--
-- 資料表索引 `video_exam_record`
--
ALTER TABLE `video_exam_record`
  ADD PRIMARY KEY (`video_exam_sn`),
  ADD KEY `CON_video_exam_idx` (`question_sn`),
  ADD KEY `indicator` (`indicator`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `review_sn` (`review_sn`),
  ADD KEY `question_sn` (`question_sn`),
  ADD KEY `video_item_sn` (`video_item_sn`),
  ADD KEY `video_id` (`video_id`);

--
-- 資料表索引 `video_exam_record_20230316`
--
ALTER TABLE `video_exam_record_20230316`
  ADD PRIMARY KEY (`video_exam_sn`),
  ADD KEY `CON_video_exam_idx` (`question_sn`),
  ADD KEY `indicator` (`indicator`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `review_sn` (`review_sn`),
  ADD KEY `question_sn` (`question_sn`),
  ADD KEY `video_item_sn` (`video_item_sn`),
  ADD KEY `video_id` (`video_id`);

--
-- 資料表索引 `video_note`
--
ALTER TABLE `video_note`
  ADD PRIMARY KEY (`video_note_sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `video_item_sn` (`video_item_sn`),
  ADD KEY `is_public` (`is_public`),
  ADD KEY `file_backup` (`file_backup`);

--
-- 資料表索引 `video_noteask`
--
ALTER TABLE `video_noteask`
  ADD PRIMARY KEY (`ask_sn`),
  ADD KEY `video_item_sn` (`video_item_sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `group_id` (`group_id`),
  ADD KEY `is_all_open` (`is_all_open`),
  ADD KEY `is_public` (`is_public`),
  ADD KEY `is_show` (`is_show`),
  ADD KEY `hidden_reply` (`hidden_reply`),
  ADD KEY `file_backup` (`file_backup`),
  ADD KEY `grade_class` (`grade_class`),
  ADD KEY `is_chooseBestAns` (`is_chooseBestAns`);

--
-- 資料表索引 `video_noteask_plus`
--
ALTER TABLE `video_noteask_plus`
  ADD PRIMARY KEY (`reply_sn`),
  ADD KEY `CON_video_noteask_idx` (`ask_sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `role_type` (`role_type`),
  ADD KEY `is_bestAnswer` (`is_bestAnswer`),
  ADD KEY `is_show` (`is_show`),
  ADD KEY `ask_sn` (`ask_sn`),
  ADD KEY `file_backup` (`file_backup`),
  ADD KEY `deleter` (`deleter`);

--
-- 資料表索引 `video_noteask_plus_20230316`
--
ALTER TABLE `video_noteask_plus_20230316`
  ADD PRIMARY KEY (`reply_sn`),
  ADD KEY `CON_video_noteask_idx` (`ask_sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `role_type` (`role_type`),
  ADD KEY `is_bestAnswer` (`is_bestAnswer`),
  ADD KEY `is_show` (`is_show`),
  ADD KEY `ask_sn` (`ask_sn`),
  ADD KEY `file_backup` (`file_backup`),
  ADD KEY `deleter` (`deleter`);

--
-- 資料表索引 `video_note_favorite`
--
ALTER TABLE `video_note_favorite`
  ADD PRIMARY KEY (`favorite_sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `video_note_sn` (`video_note_sn`),
  ADD KEY `is_like` (`is_like`),
  ADD KEY `is_favorite` (`is_favorite`);

--
-- 資料表索引 `video_note_favorite_20230316`
--
ALTER TABLE `video_note_favorite_20230316`
  ADD PRIMARY KEY (`favorite_sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `video_note_sn` (`video_note_sn`),
  ADD KEY `is_like` (`is_like`),
  ADD KEY `is_favorite` (`is_favorite`);

--
-- 資料表索引 `video_note_feedback`
--
ALTER TABLE `video_note_feedback`
  ADD PRIMARY KEY (`feedback_sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `video_note_sn` (`video_note_sn`),
  ADD KEY `deleter` (`deleter`),
  ADD KEY `is_delete` (`is_delete`);

--
-- 資料表索引 `video_note_feedback_20230316`
--
ALTER TABLE `video_note_feedback_20230316`
  ADD PRIMARY KEY (`feedback_sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `video_note_sn` (`video_note_sn`),
  ADD KEY `deleter` (`deleter`),
  ADD KEY `is_delete` (`is_delete`);

--
-- 資料表索引 `video_note_recommend`
--
ALTER TABLE `video_note_recommend`
  ADD PRIMARY KEY (`video_recommend_sn`),
  ADD KEY `video_note_sn` (`video_note_sn`),
  ADD KEY `referrer_id` (`referrer_id`);

--
-- 資料表索引 `video_note_recommend_20230316`
--
ALTER TABLE `video_note_recommend_20230316`
  ADD PRIMARY KEY (`video_recommend_sn`),
  ADD KEY `video_note_sn` (`video_note_sn`),
  ADD KEY `referrer_id` (`referrer_id`);

--
-- 資料表索引 `video_review_record`
--
ALTER TABLE `video_review_record`
  ADD PRIMARY KEY (`review_sn`),
  ADD KEY `video_item_sn` (`video_item_sn`) USING BTREE,
  ADD KEY `user_id` (`user_id`),
  ADD KEY `start_time` (`start_time`),
  ADD KEY `idx_starttime_userid` (`start_time`,`user_id`),
  ADD KEY `seme_year` (`seme_year`),
  ADD KEY `mission_sn` (`mission_sn`),
  ADD KEY `idx_userid_starttime` (`user_id`,`start_time`),
  ADD KEY `total_time` (`total_time`),
  ADD KEY `idx_videoitemsn_semeyear_userid` (`video_item_sn`,`seme_year`,`user_id`),
  ADD KEY `reward_mk` (`reward_mk`),
  ADD KEY `idx_userid_videoitemsn` (`user_id`,`video_item_sn`),
  ADD KEY `end_time` (`end_time`),
  ADD KEY `end_time_2` (`end_time`);

--
-- 資料表索引 `video_review_record_plus`
--
ALTER TABLE `video_review_record_plus`
  ADD PRIMARY KEY (`review_plus_sn`),
  ADD KEY `review_sn` (`review_sn`),
  ADD KEY `view_action` (`view_action`);

--
-- 資料表索引 `video_review_record_plus_20230316`
--
ALTER TABLE `video_review_record_plus_20230316`
  ADD PRIMARY KEY (`review_plus_sn`),
  ADD KEY `review_sn` (`review_sn`),
  ADD KEY `view_action` (`view_action`);

--
-- 資料表索引 `video_review_record_tmp`
--
ALTER TABLE `video_review_record_tmp`
  ADD PRIMARY KEY (`review_sn`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `video_id` (`video_id`),
  ADD KEY `video_item_sn` (`video_item_sn`);

--
-- 資料表索引 `website_module`
--
ALTER TABLE `website_module`
  ADD PRIMARY KEY (`module_id`),
  ADD UNIQUE KEY `uk_foldername_subfoldername_filename_uri` (`folder_name`,`sub_folder_name`,`filename`,`specific_uri`) USING BTREE,
  ADD KEY `ace_title` (`module_title`),
  ADD KEY `folder_name` (`folder_name`),
  ADD KEY `sub_folder_name` (`sub_folder_name`);

--
-- 資料表索引 `z_system_identity_log`
--
ALTER TABLE `z_system_identity_log`
  ADD PRIMARY KEY (`sn`),
  ADD KEY `user_loginto` (`user_loginto`),
  ADD KEY `user_loginfrom` (`user_loginfrom`);

--
-- 資料表索引 `z_system_team_feedback_status`
--
ALTER TABLE `z_system_team_feedback_status`
  ADD PRIMARY KEY (`id`),
  ADD KEY `q_id` (`q_id`);

--
-- 資料表索引 `z_system_team_users`
--
ALTER TABLE `z_system_team_users`
  ADD PRIMARY KEY (`id`),
  ADD UNIQUE KEY `uk_useraccount_ip` (`user_account`,`ip`),
  ADD KEY `user_id` (`user_id`),
  ADD KEY `subject_id` (`subject_id`),
  ADD KEY `user_account` (`user_account`),
  ADD KEY `name` (`name`);

--
-- 在傾印的資料表使用自動遞增(AUTO_INCREMENT)
--

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `api_client_config`
--
ALTER TABLE `api_client_config`
  MODIFY `api_client_sn` int UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `api_token_data`
--
ALTER TABLE `api_token_data`
  MODIFY `token_sn` bigint NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `chatroom`
--
ALTER TABLE `chatroom`
  MODIFY `chatroom_id` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `chatroom_member`
--
ALTER TABLE `chatroom_member`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `chat_log`
--
ALTER TABLE `chat_log`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `check_list_table`
--
ALTER TABLE `check_list_table`
  MODIFY `check_list_table_sn` int NOT NULL AUTO_INCREMENT COMMENT '流水號SN';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `city_area`
--
ALTER TABLE `city_area`
  MODIFY `sn` smallint UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `class_name`
--
ALTER TABLE `class_name`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `competency_unit`
--
ALTER TABLE `competency_unit`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `concept_info`
--
ALTER TABLE `concept_info`
  MODIFY `cs_sn` bigint NOT NULL AUTO_INCREMENT COMMENT '概念流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `concept_itemBank`
--
ALTER TABLE `concept_itemBank`
  MODIFY `item_sn` bigint UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '關聯';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `concept_itemBank_0320`
--
ALTER TABLE `concept_itemBank_0320`
  MODIFY `item_sn` bigint UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '關聯';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `concept_itemBank_0521`
--
ALTER TABLE `concept_itemBank_0521`
  MODIFY `item_sn` bigint UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '關聯';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `concept_itemBank_20240904`
--
ALTER TABLE `concept_itemBank_20240904`
  MODIFY `item_sn` bigint UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '關聯';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `concept_itemBank_20250512`
--
ALTER TABLE `concept_itemBank_20250512`
  MODIFY `item_sn` bigint UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '關聯';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `concept_itemBank_20250819`
--
ALTER TABLE `concept_itemBank_20250819`
  MODIFY `item_sn` bigint UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '關聯';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `concept_itemBank_20250827`
--
ALTER TABLE `concept_itemBank_20250827`
  MODIFY `item_sn` bigint UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '關聯';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `concept_itemBank_20251105`
--
ALTER TABLE `concept_itemBank_20251105`
  MODIFY `item_sn` bigint UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '關聯';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `concept_itemBank_20251223`
--
ALTER TABLE `concept_itemBank_20251223`
  MODIFY `item_sn` bigint UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '關聯';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `concept_itemBank_bat`
--
ALTER TABLE `concept_itemBank_bat`
  MODIFY `item_sn` int UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '關聯';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `concept_itemBank_bat_inds`
--
ALTER TABLE `concept_itemBank_bat_inds`
  MODIFY `sn` int UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '關聯';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `concept_itemBank_depiction`
--
ALTER TABLE `concept_itemBank_depiction`
  MODIFY `depiction_sn` int UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `concept_itemBank_depiction_0320`
--
ALTER TABLE `concept_itemBank_depiction_0320`
  MODIFY `depiction_sn` int UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `concept_itemBank_depiction_0710`
--
ALTER TABLE `concept_itemBank_depiction_0710`
  MODIFY `depiction_sn` int UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `concept_itemBank_depiction_20240904`
--
ALTER TABLE `concept_itemBank_depiction_20240904`
  MODIFY `depiction_sn` int UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `concept_itemBank_depiction_20250512`
--
ALTER TABLE `concept_itemBank_depiction_20250512`
  MODIFY `depiction_sn` int UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `concept_itemBank_depiction_20250819`
--
ALTER TABLE `concept_itemBank_depiction_20250819`
  MODIFY `depiction_sn` int UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `concept_itemBank_depiction_20250827`
--
ALTER TABLE `concept_itemBank_depiction_20250827`
  MODIFY `depiction_sn` int UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `concept_itemBank_depiction_20251105`
--
ALTER TABLE `concept_itemBank_depiction_20251105`
  MODIFY `depiction_sn` int UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `concept_itemBank_depiction_20251223`
--
ALTER TABLE `concept_itemBank_depiction_20251223`
  MODIFY `depiction_sn` int UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `concept_itemBank_literacy`
--
ALTER TABLE `concept_itemBank_literacy`
  MODIFY `item_sn` int UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `concept_itemBank_literacy_20250512`
--
ALTER TABLE `concept_itemBank_literacy_20250512`
  MODIFY `item_sn` int UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `concept_itemBank_literacy_20250819`
--
ALTER TABLE `concept_itemBank_literacy_20250819`
  MODIFY `item_sn` int UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `concept_itemBank_literacy_20250827`
--
ALTER TABLE `concept_itemBank_literacy_20250827`
  MODIFY `item_sn` int UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `concept_itemBank_literacy_20251105`
--
ALTER TABLE `concept_itemBank_literacy_20251105`
  MODIFY `item_sn` int UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `concept_itemBank_literacy_20251223`
--
ALTER TABLE `concept_itemBank_literacy_20251223`
  MODIFY `item_sn` int UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `concept_itemBank_literacy_conversation`
--
ALTER TABLE `concept_itemBank_literacy_conversation`
  MODIFY `literacy_conversation_sn` int UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `concept_itemBank_literacy_group`
--
ALTER TABLE `concept_itemBank_literacy_group`
  MODIFY `literacy_group_sn` int UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `concept_itemBank_literacy_interactive`
--
ALTER TABLE `concept_itemBank_literacy_interactive`
  MODIFY `item_li_sn` int UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `concept_itemBank_option`
--
ALTER TABLE `concept_itemBank_option`
  MODIFY `option_sn` int UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `concept_itemBank_option_0320`
--
ALTER TABLE `concept_itemBank_option_0320`
  MODIFY `option_sn` int UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `concept_itemBank_option_0710`
--
ALTER TABLE `concept_itemBank_option_0710`
  MODIFY `option_sn` int UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `concept_item_interactive`
--
ALTER TABLE `concept_item_interactive`
  MODIFY `item_sn` bigint UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '關聯';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `concept_its`
--
ALTER TABLE `concept_its`
  MODIFY `its_sn` int NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `concept_paper`
--
ALTER TABLE `concept_paper`
  MODIFY `paper_sn` bigint UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `concept_priori`
--
ALTER TABLE `concept_priori`
  MODIFY `cp_sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `concept_priori_info`
--
ALTER TABLE `concept_priori_info`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `concept_remedy`
--
ALTER TABLE `concept_remedy`
  MODIFY `sn` int UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `contestexam_item`
--
ALTER TABLE `contestexam_item`
  MODIFY `item_sn` int UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '試題流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `contestexam_itemgroup`
--
ALTER TABLE `contestexam_itemgroup`
  MODIFY `group_sn` int UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '題組流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `contestexam_itemgroup_detail`
--
ALTER TABLE `contestexam_itemgroup_detail`
  MODIFY `sn` int UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `contestexam_item_answer_option`
--
ALTER TABLE `contestexam_item_answer_option`
  MODIFY `option_sn` int UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '選項流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `contestexam_item_indicator_mapping`
--
ALTER TABLE `contestexam_item_indicator_mapping`
  MODIFY `sn` int UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `contestexam_old_uploaded_filename`
--
ALTER TABLE `contestexam_old_uploaded_filename`
  MODIFY `sn` int UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `contestexam_paper`
--
ALTER TABLE `contestexam_paper`
  MODIFY `paper_sn` int UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '卷流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `contestexam_paper_detail`
--
ALTER TABLE `contestexam_paper_detail`
  MODIFY `sn` int UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `contestexam_record`
--
ALTER TABLE `contestexam_record`
  MODIFY `record_sn` int UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '紀錄流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `contestexam_record_connection_log_nouse`
--
ALTER TABLE `contestexam_record_connection_log_nouse`
  MODIFY `log_sn` int NOT NULL AUTO_INCREMENT COMMENT '紀錄流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `contestexam_record_detail`
--
ALTER TABLE `contestexam_record_detail`
  MODIFY `sn` int UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `contest_agreement`
--
ALTER TABLE `contest_agreement`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `contest_certificate`
--
ALTER TABLE `contest_certificate`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `contest_content`
--
ALTER TABLE `contest_content`
  MODIFY `content_sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `contest_dailyreport`
--
ALTER TABLE `contest_dailyreport`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `contest_group`
--
ALTER TABLE `contest_group`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `contest_group_changelog`
--
ALTER TABLE `contest_group_changelog`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `contest_group_comparison`
--
ALTER TABLE `contest_group_comparison`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `contest_info`
--
ALTER TABLE `contest_info`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `contest_itembank`
--
ALTER TABLE `contest_itembank`
  MODIFY `item_li_sn` bigint UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `contest_itembank_2021mathscience`
--
ALTER TABLE `contest_itembank_2021mathscience`
  MODIFY `item_li_sn` bigint UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `contest_itembank_2022math`
--
ALTER TABLE `contest_itembank_2022math`
  MODIFY `item_li_sn` bigint UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `contest_itembank_2023mathscience`
--
ALTER TABLE `contest_itembank_2023mathscience`
  MODIFY `item_li_sn` bigint UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `contest_itembank_group`
--
ALTER TABLE `contest_itembank_group`
  MODIFY `literacy_group_sn` bigint UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `contest_itemBank_literacy`
--
ALTER TABLE `contest_itemBank_literacy`
  MODIFY `item_sn` bigint UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `contest_itembank_summer`
--
ALTER TABLE `contest_itembank_summer`
  MODIFY `item_li_sn` bigint UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `contest_publisher_mapping`
--
ALTER TABLE `contest_publisher_mapping`
  MODIFY `literacy_publisher_sn` int UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `contest_publisher_mapping_2021mathscience`
--
ALTER TABLE `contest_publisher_mapping_2021mathscience`
  MODIFY `literacy_publisher_sn` int UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `contest_publisher_mapping_2022math`
--
ALTER TABLE `contest_publisher_mapping_2022math`
  MODIFY `literacy_publisher_sn` int UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `contest_record`
--
ALTER TABLE `contest_record`
  MODIFY `exam_sn` bigint NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `contest_record_literacy`
--
ALTER TABLE `contest_record_literacy`
  MODIFY `exam_sn` bigint NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `contest_record_respond`
--
ALTER TABLE `contest_record_respond`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `contest_signup`
--
ALTER TABLE `contest_signup`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `contest_signup_20220422`
--
ALTER TABLE `contest_signup_20220422`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `contest_signup_changelog`
--
ALTER TABLE `contest_signup_changelog`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `conversation`
--
ALTER TABLE `conversation`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `conversation_access`
--
ALTER TABLE `conversation_access`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `conversation_detail`
--
ALTER TABLE `conversation_detail`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `conversation_detail_images`
--
ALTER TABLE `conversation_detail_images`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT COMMENT 'cdi.sn';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `conversation_detail_images_error`
--
ALTER TABLE `conversation_detail_images_error`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `conversation_error`
--
ALTER TABLE `conversation_error`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `conversation_old`
--
ALTER TABLE `conversation_old`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `conversation_times`
--
ALTER TABLE `conversation_times`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `cooperation_city_area`
--
ALTER TABLE `cooperation_city_area`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT COMMENT '序';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `cooperation_organization`
--
ALTER TABLE `cooperation_organization`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT COMMENT '序';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `course_subject`
--
ALTER TABLE `course_subject`
  MODIFY `cs_sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `cross_school_student`
--
ALTER TABLE `cross_school_student`
  MODIFY `cross_school_student_sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `csrf_security_log`
--
ALTER TABLE `csrf_security_log`
  MODIFY `id` bigint UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `ct_node_mapping`
--
ALTER TABLE `ct_node_mapping`
  MODIFY `ct_node_mapping_sn` int NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `datashop_data_table`
--
ALTER TABLE `datashop_data_table`
  MODIFY `id` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `dbuser`
--
ALTER TABLE `dbuser`
  MODIFY `sn` int UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `eduexam_item`
--
ALTER TABLE `eduexam_item`
  MODIFY `item_sn` int UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '試題流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `eduexam_itemgroup`
--
ALTER TABLE `eduexam_itemgroup`
  MODIFY `group_sn` int UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '題組流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `eduexam_itemgroup_detail`
--
ALTER TABLE `eduexam_itemgroup_detail`
  MODIFY `sn` int UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `eduexam_item_answer_option`
--
ALTER TABLE `eduexam_item_answer_option`
  MODIFY `option_sn` int UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '選項流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `eduexam_item_indicator_mapping`
--
ALTER TABLE `eduexam_item_indicator_mapping`
  MODIFY `sn` int UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `eduexam_old_uploaded_filename`
--
ALTER TABLE `eduexam_old_uploaded_filename`
  MODIFY `sn` int UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `eduexam_paper`
--
ALTER TABLE `eduexam_paper`
  MODIFY `paper_sn` int UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '卷流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `eduexam_paper_detail`
--
ALTER TABLE `eduexam_paper_detail`
  MODIFY `sn` int UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `eduexam_record`
--
ALTER TABLE `eduexam_record`
  MODIFY `record_sn` int UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '紀錄流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `eduexam_record_detail`
--
ALTER TABLE `eduexam_record_detail`
  MODIFY `sn` int UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `egame_exam_record`
--
ALTER TABLE `egame_exam_record`
  MODIFY `sn` int UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '資料流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `egame_info`
--
ALTER TABLE `egame_info`
  MODIFY `egame_id` int UNSIGNED NOT NULL AUTO_INCREMENT COMMENT 'egame 編號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `exam_country_student_tmp`
--
ALTER TABLE `exam_country_student_tmp`
  MODIFY `stud_sn` int UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `exam_item_answer_option_type`
--
ALTER TABLE `exam_item_answer_option_type`
  MODIFY `answer_option_type` tinyint UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '答案選項類型';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `exam_item_type`
--
ALTER TABLE `exam_item_type`
  MODIFY `item_type` tinyint UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '題型';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `exam_onscreen_keyboard`
--
ALTER TABLE `exam_onscreen_keyboard`
  MODIFY `keyboard_sn` smallint UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '小鍵盤流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `exam_paper`
--
ALTER TABLE `exam_paper`
  MODIFY `sn` bigint NOT NULL AUTO_INCREMENT COMMENT '試卷流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `exam_paper_access`
--
ALTER TABLE `exam_paper_access`
  MODIFY `sn` bigint NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `exam_record`
--
ALTER TABLE `exam_record`
  MODIFY `exam_sn` bigint NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `exam_record_aassessment_detail`
--
ALTER TABLE `exam_record_aassessment_detail`
  MODIFY `sn` int UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `exam_record_aassessment_detail_1028`
--
ALTER TABLE `exam_record_aassessment_detail_1028`
  MODIFY `sn` int UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `exam_record_bat_detail`
--
ALTER TABLE `exam_record_bat_detail`
  MODIFY `sn` int UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `exam_record_dynamic`
--
ALTER TABLE `exam_record_dynamic`
  MODIFY `exam_sn` bigint NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `exam_record_indicate`
--
ALTER TABLE `exam_record_indicate`
  MODIFY `exam_sn` bigint NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `exam_record_indicate_tmp`
--
ALTER TABLE `exam_record_indicate_tmp`
  MODIFY `exam_sn` bigint NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `exam_record_indicator`
--
ALTER TABLE `exam_record_indicator`
  MODIFY `exam_sn` bigint NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `exam_record_indicator_bat`
--
ALTER TABLE `exam_record_indicator_bat`
  MODIFY `exam_sn` bigint NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `exam_record_indicator_bat_tmp`
--
ALTER TABLE `exam_record_indicator_bat_tmp`
  MODIFY `exam_sn` bigint NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `exam_record_indicator_cap`
--
ALTER TABLE `exam_record_indicator_cap`
  MODIFY `exam_sn` bigint NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `exam_record_indicator_cap_tmp`
--
ALTER TABLE `exam_record_indicator_cap_tmp`
  MODIFY `exam_sn` bigint NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `exam_record_indicator_tmp`
--
ALTER TABLE `exam_record_indicator_tmp`
  MODIFY `exam_sn` bigint NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `exam_record_interactive`
--
ALTER TABLE `exam_record_interactive`
  MODIFY `exam_sn` bigint NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `exam_record_its`
--
ALTER TABLE `exam_record_its`
  MODIFY `its_sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `exam_record_literacy`
--
ALTER TABLE `exam_record_literacy`
  MODIFY `exam_sn` bigint NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `exam_record_literacy_interactive`
--
ALTER TABLE `exam_record_literacy_interactive`
  MODIFY `exam_sn` bigint NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `exam_record_literacy_interactive_log`
--
ALTER TABLE `exam_record_literacy_interactive_log`
  MODIFY `log_sn` bigint NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `exam_record_priori`
--
ALTER TABLE `exam_record_priori`
  MODIFY `exam_sn` bigint NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `exam_record_priori_detail`
--
ALTER TABLE `exam_record_priori_detail`
  MODIFY `sn` int UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `exam_record_priori_error`
--
ALTER TABLE `exam_record_priori_error`
  MODIFY `sn` bigint NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `exam_record_priori_his`
--
ALTER TABLE `exam_record_priori_his`
  MODIFY `sn` bigint NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `exam_record_tmp`
--
ALTER TABLE `exam_record_tmp`
  MODIFY `exam_sn` bigint NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `exam_template`
--
ALTER TABLE `exam_template`
  MODIFY `template_id` tinyint UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '測驗之樣版檔代號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `exam_type`
--
ALTER TABLE `exam_type`
  MODIFY `exam_type` tinyint UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `export_school`
--
ALTER TABLE `export_school`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `ftp_upload_log`
--
ALTER TABLE `ftp_upload_log`
  MODIFY `id` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `group_check_result`
--
ALTER TABLE `group_check_result`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `group_member_score_result`
--
ALTER TABLE `group_member_score_result`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `group_role`
--
ALTER TABLE `group_role`
  MODIFY `group_role_sn` int NOT NULL AUTO_INCREMENT COMMENT '小組角色流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `group_score_ingroup_result`
--
ALTER TABLE `group_score_ingroup_result`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `group_score_result`
--
ALTER TABLE `group_score_result`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `gtm_logs`
--
ALTER TABLE `gtm_logs`
  MODIFY `id` bigint NOT NULL AUTO_INCREMENT COMMENT 'primary key';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `high_level_comment`
--
ALTER TABLE `high_level_comment`
  MODIFY `comment_sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `high_level_group_ans_list`
--
ALTER TABLE `high_level_group_ans_list`
  MODIFY `ans_id` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `high_level_joint_discovery`
--
ALTER TABLE `high_level_joint_discovery`
  MODIFY `discovery_sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `high_level_log`
--
ALTER TABLE `high_level_log`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `high_level_personal_ans_list`
--
ALTER TABLE `high_level_personal_ans_list`
  MODIFY `ans_id` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `high_school_dept`
--
ALTER TABLE `high_school_dept`
  MODIFY `dept_sn` int UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '高中職科別流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `high_school_dept_mapping`
--
ALTER TABLE `high_school_dept_mapping`
  MODIFY `dept_mapping_sn` int UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '高中職科別對應流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `high_school_group`
--
ALTER TABLE `high_school_group`
  MODIFY `group_sn` int UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '高中職群別流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `high_school_type`
--
ALTER TABLE `high_school_type`
  MODIFY `school_type` int UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '高中職類型流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `indicateTest`
--
ALTER TABLE `indicateTest`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `indicate_test_predata`
--
ALTER TABLE `indicate_test_predata`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `indicator_info`
--
ALTER TABLE `indicator_info`
  MODIFY `indicator_sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `indicator_priori`
--
ALTER TABLE `indicator_priori`
  MODIFY `priori_sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `ItemBank_bat_fix_record`
--
ALTER TABLE `ItemBank_bat_fix_record`
  MODIFY `fix_record_sn` bigint UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `ItemBank_fix_record`
--
ALTER TABLE `ItemBank_fix_record`
  MODIFY `fix_record_sn` bigint UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `item_mission_rate`
--
ALTER TABLE `item_mission_rate`
  MODIFY `item_mission_rate_sn` bigint NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `lecturer_list`
--
ALTER TABLE `lecturer_list`
  MODIFY `sn` int UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `literacy_publisher_mapping`
--
ALTER TABLE `literacy_publisher_mapping`
  MODIFY `literacy_publisher_sn` int UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `log_exam_bug`
--
ALTER TABLE `log_exam_bug`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `log_exam_record_aassessment_detail`
--
ALTER TABLE `log_exam_record_aassessment_detail`
  MODIFY `log_id` int UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '日誌流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `mad_exam_record`
--
ALTER TABLE `mad_exam_record`
  MODIFY `exam_sn` bigint NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `mad_item`
--
ALTER TABLE `mad_item`
  MODIFY `sn` int UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `mad_rule`
--
ALTER TABLE `mad_rule`
  MODIFY `sn` int UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `mad_type`
--
ALTER TABLE `mad_type`
  MODIFY `mad_type_id` int NOT NULL AUTO_INCREMENT COMMENT '類型id';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `main_data`
--
ALTER TABLE `main_data`
  MODIFY `num` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `map_arrows`
--
ALTER TABLE `map_arrows`
  MODIFY `arrow_sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `map_info`
--
ALTER TABLE `map_info`
  MODIFY `map_sn` int NOT NULL AUTO_INCREMENT COMMENT '星空圖代號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `map_info_0903`
--
ALTER TABLE `map_info_0903`
  MODIFY `map_sn` int NOT NULL AUTO_INCREMENT COMMENT '星空圖代號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `map_node_0521`
--
ALTER TABLE `map_node_0521`
  MODIFY `node_sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `map_node_20240813`
--
ALTER TABLE `map_node_20240813`
  MODIFY `node_sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `map_node_20251105`
--
ALTER TABLE `map_node_20251105`
  MODIFY `node_sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `map_node_bk2`
--
ALTER TABLE `map_node_bk2`
  MODIFY `node_sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `map_node_external`
--
ALTER TABLE `map_node_external`
  MODIFY `res_sn` mediumint UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `map_node_file`
--
ALTER TABLE `map_node_file`
  MODIFY `map_node_file_sn` bigint UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `map_node_grade`
--
ALTER TABLE `map_node_grade`
  MODIFY `sn` int UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `map_node_mapping`
--
ALTER TABLE `map_node_mapping`
  MODIFY `sn` int UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `map_node_mapping_0521`
--
ALTER TABLE `map_node_mapping_0521`
  MODIFY `sn` int UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `map_node_student_status`
--
ALTER TABLE `map_node_student_status`
  MODIFY `status_sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `math_laboratory_exam_record`
--
ALTER TABLE `math_laboratory_exam_record`
  MODIFY `sn` int UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '資料流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `message_fileattached`
--
ALTER TABLE `message_fileattached`
  MODIFY `file_sn` int NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `message_fileattached_20230316`
--
ALTER TABLE `message_fileattached_20230316`
  MODIFY `file_sn` int NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `message_master`
--
ALTER TABLE `message_master`
  MODIFY `msg_sn` int NOT NULL AUTO_INCREMENT COMMENT '訊息id';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `message_response`
--
ALTER TABLE `message_response`
  MODIFY `response_sn` int NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `message_response_20230316`
--
ALTER TABLE `message_response_20230316`
  MODIFY `response_sn` int NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `mission_google_form`
--
ALTER TABLE `mission_google_form`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `mission_google_master`
--
ALTER TABLE `mission_google_master`
  MODIFY `sn` bigint UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `mission_group_leader`
--
ALTER TABLE `mission_group_leader`
  MODIFY `sn` bigint UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `mission_info`
--
ALTER TABLE `mission_info`
  MODIFY `mission_sn` int NOT NULL AUTO_INCREMENT COMMENT '任務 sn ';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `mission_result`
--
ALTER TABLE `mission_result`
  MODIFY `result_sn` int NOT NULL AUTO_INCREMENT COMMENT '任務結果sn';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `mission_stud_record`
--
ALTER TABLE `mission_stud_record`
  MODIFY `stu_mission_sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `mission_type`
--
ALTER TABLE `mission_type`
  MODIFY `sn` tinyint UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `mission_voice_recognition`
--
ALTER TABLE `mission_voice_recognition`
  MODIFY `voice_recognition_sn` int UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `monthly_student_usage`
--
ALTER TABLE `monthly_student_usage`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `monthly_student_usage_0806`
--
ALTER TABLE `monthly_student_usage_0806`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `monthly_student_usage_backup`
--
ALTER TABLE `monthly_student_usage_backup`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `monthly_student_usage_bcakup0804`
--
ALTER TABLE `monthly_student_usage_bcakup0804`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `monthly_student_usage_rankings`
--
ALTER TABLE `monthly_student_usage_rankings`
  MODIFY `sn` bigint NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `new_class`
--
ALTER TABLE `new_class`
  MODIFY `new_class_sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `old_nodes`
--
ALTER TABLE `old_nodes`
  MODIFY `id` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `old_uploaded_filename`
--
ALTER TABLE `old_uploaded_filename`
  MODIFY `sn` int UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `onversation_times`
--
ALTER TABLE `onversation_times`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `openid_api_status_record`
--
ALTER TABLE `openid_api_status_record`
  MODIFY `sn` bigint UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `openid_class_info`
--
ALTER TABLE `openid_class_info`
  MODIFY `sn` bigint UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `openid_titles_info`
--
ALTER TABLE `openid_titles_info`
  MODIFY `sn` bigint UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `openid_user_data_check_status`
--
ALTER TABLE `openid_user_data_check_status`
  MODIFY `sn` bigint UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `openid_user_data_check_status_discription`
--
ALTER TABLE `openid_user_data_check_status_discription`
  MODIFY `sn` bigint UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `openid_user_info`
--
ALTER TABLE `openid_user_info`
  MODIFY `sn` bigint UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `openid_user_login_record`
--
ALTER TABLE `openid_user_login_record`
  MODIFY `sn` bigint UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `openid_user_login_title`
--
ALTER TABLE `openid_user_login_title`
  MODIFY `sn` bigint UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `organization_ace_setting`
--
ALTER TABLE `organization_ace_setting`
  MODIFY `sn` int UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `organization_Education_Administration`
--
ALTER TABLE `organization_Education_Administration`
  MODIFY `sn` int UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `organization_project_participation`
--
ALTER TABLE `organization_project_participation`
  MODIFY `sn` int UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `organization_subject_group`
--
ALTER TABLE `organization_subject_group`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `paper_teacher`
--
ALTER TABLE `paper_teacher`
  MODIFY `sn` bigint NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `prac_answer`
--
ALTER TABLE `prac_answer`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `prac_blank_filling_itembank`
--
ALTER TABLE `prac_blank_filling_itembank`
  MODIFY `item_sn` int UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `prac_blank_filling_itembank_20240813`
--
ALTER TABLE `prac_blank_filling_itembank_20240813`
  MODIFY `item_sn` int UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `prac_blank_filling_record`
--
ALTER TABLE `prac_blank_filling_record`
  MODIFY `sn` int UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `prac_options`
--
ALTER TABLE `prac_options`
  MODIFY `id` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `prac_questions`
--
ALTER TABLE `prac_questions`
  MODIFY `id` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `prac_session_data`
--
ALTER TABLE `prac_session_data`
  MODIFY `session_id` bigint NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `prac_single_answer`
--
ALTER TABLE `prac_single_answer`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `priori_itembank`
--
ALTER TABLE `priori_itembank`
  MODIFY `priori_item_sn` int UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `publisher`
--
ALTER TABLE `publisher`
  MODIFY `publisher_id` smallint UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '出版商編號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `publisher_mapping`
--
ALTER TABLE `publisher_mapping`
  MODIFY `mapping_sn` int NOT NULL AUTO_INCREMENT COMMENT '流水號 (課程列表mid)';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `publisher_mapping_0521`
--
ALTER TABLE `publisher_mapping_0521`
  MODIFY `mapping_sn` int NOT NULL AUTO_INCREMENT COMMENT '流水號 (課程列表mid)';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `publisher_mapping_20240813`
--
ALTER TABLE `publisher_mapping_20240813`
  MODIFY `mapping_sn` int NOT NULL AUTO_INCREMENT COMMENT '流水號 (課程列表mid)';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `publisher_mapping_20250207`
--
ALTER TABLE `publisher_mapping_20250207`
  MODIFY `mapping_sn` int NOT NULL AUTO_INCREMENT COMMENT '流水號 (課程列表mid)';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `publisher_mapping_backup_20220608`
--
ALTER TABLE `publisher_mapping_backup_20220608`
  MODIFY `mapping_sn` int NOT NULL AUTO_INCREMENT COMMENT '流水號 (課程列表mid)';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `publisher_mapping_backup_20220628`
--
ALTER TABLE `publisher_mapping_backup_20220628`
  MODIFY `mapping_sn` int NOT NULL AUTO_INCREMENT COMMENT '流水號 (課程列表mid)';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `publisher_mapping_log`
--
ALTER TABLE `publisher_mapping_log`
  MODIFY `sn` int UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `questionaire_export_permission`
--
ALTER TABLE `questionaire_export_permission`
  MODIFY `sn` int UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `question_main`
--
ALTER TABLE `question_main`
  MODIFY `num` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `remedial_class`
--
ALTER TABLE `remedial_class`
  MODIFY `class_sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `remedial_ftp_history`
--
ALTER TABLE `remedial_ftp_history`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `remedial_result_history`
--
ALTER TABLE `remedial_result_history`
  MODIFY `sn` bigint UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `remedial_student`
--
ALTER TABLE `remedial_student`
  MODIFY `remedial_student_sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `remedial_teacher`
--
ALTER TABLE `remedial_teacher`
  MODIFY `remedial_teacher_sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `report_dailyusage`
--
ALTER TABLE `report_dailyusage`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `report_grad_student_dynamic`
--
ALTER TABLE `report_grad_student_dynamic`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `report_grad_student_indicate`
--
ALTER TABLE `report_grad_student_indicate`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `report_grad_student_node`
--
ALTER TABLE `report_grad_student_node`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `report_grad_student_practice`
--
ALTER TABLE `report_grad_student_practice`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `report_grad_student_robot`
--
ALTER TABLE `report_grad_student_robot`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `report_grad_student_unit`
--
ALTER TABLE `report_grad_student_unit`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `report_grad_student_video`
--
ALTER TABLE `report_grad_student_video`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `report_learningeffect_dynamic`
--
ALTER TABLE `report_learningeffect_dynamic`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `report_learningeffect_indicate`
--
ALTER TABLE `report_learningeffect_indicate`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `report_learningeffect_node`
--
ALTER TABLE `report_learningeffect_node`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `report_learningeffect_practice`
--
ALTER TABLE `report_learningeffect_practice`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `report_learningeffect_robot`
--
ALTER TABLE `report_learningeffect_robot`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `report_learningeffect_unit`
--
ALTER TABLE `report_learningeffect_unit`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `report_learningeffect_video`
--
ALTER TABLE `report_learningeffect_video`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `report_workshop_effect`
--
ALTER TABLE `report_workshop_effect`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `resource_choose_hint`
--
ALTER TABLE `resource_choose_hint`
  MODIFY `id` int NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `resource_exam_annex`
--
ALTER TABLE `resource_exam_annex`
  MODIFY `id` int NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `resource_filling_answer`
--
ALTER TABLE `resource_filling_answer`
  MODIFY `id` int NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `robot_config`
--
ALTER TABLE `robot_config`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `role_coin_history`
--
ALTER TABLE `role_coin_history`
  MODIFY `sn` int UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `role_coin_history_20230316`
--
ALTER TABLE `role_coin_history_20230316`
  MODIFY `sn` int UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `role_coin_rule`
--
ALTER TABLE `role_coin_rule`
  MODIFY `rule_sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `role_coin_total`
--
ALTER TABLE `role_coin_total`
  MODIFY `coin_sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `role_food_item`
--
ALTER TABLE `role_food_item`
  MODIFY `food_id` int NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `role_info`
--
ALTER TABLE `role_info`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `role_prop_item`
--
ALTER TABLE `role_prop_item`
  MODIFY `prop_id` int NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `role_reward_way`
--
ALTER TABLE `role_reward_way`
  MODIFY `reward_sn` smallint NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `role_reword_card_item`
--
ALTER TABLE `role_reword_card_item`
  MODIFY `reward_card_id` int NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `school_class_info`
--
ALTER TABLE `school_class_info`
  MODIFY `class_sn` int UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '學校班級資訊流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `seme_grad_student`
--
ALTER TABLE `seme_grad_student`
  MODIFY `sn` int UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `seme_group`
--
ALTER TABLE `seme_group`
  MODIFY `group_id` int NOT NULL AUTO_INCREMENT COMMENT '小組id';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `seme_publisher`
--
ALTER TABLE `seme_publisher`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `seme_publisher_tmp`
--
ALTER TABLE `seme_publisher_tmp`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `seme_student`
--
ALTER TABLE `seme_student`
  MODIFY `sn` int UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `seme_student_rebackschool`
--
ALTER TABLE `seme_student_rebackschool`
  MODIFY `sn` int UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `seme_student_tmp`
--
ALTER TABLE `seme_student_tmp`
  MODIFY `sn` int UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `seme_teacher_subject`
--
ALTER TABLE `seme_teacher_subject`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `seme_teacher_subject_mapping`
--
ALTER TABLE `seme_teacher_subject_mapping`
  MODIFY `mapping_sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `seme_teacher_subject_tmp`
--
ALTER TABLE `seme_teacher_subject_tmp`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `seme_teacher_subject_tmp_0208`
--
ALTER TABLE `seme_teacher_subject_tmp_0208`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `srl_calendar`
--
ALTER TABLE `srl_calendar`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `subject`
--
ALTER TABLE `subject`
  MODIFY `subject_id` smallint UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '科目代號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `subject_0903`
--
ALTER TABLE `subject_0903`
  MODIFY `subject_id` smallint UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '科目代號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `subject_acnt`
--
ALTER TABLE `subject_acnt`
  MODIFY `sn` int UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `subject_bk`
--
ALTER TABLE `subject_bk`
  MODIFY `subject_id` tinyint UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '科目代號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `subject_groupings`
--
ALTER TABLE `subject_groupings`
  MODIFY `subject_groupings_id` tinyint UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '科別代號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `subject_optgroups`
--
ALTER TABLE `subject_optgroups`
  MODIFY `subject_optgroups_id` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `sub_subject`
--
ALTER TABLE `sub_subject`
  MODIFY `sub_subject_id` tinyint UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '科目代號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `system_user`
--
ALTER TABLE `system_user`
  MODIFY `sn` tinyint UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `talper_custom_agent`
--
ALTER TABLE `talper_custom_agent`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT COMMENT 'agent_sn';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `talper_custom_agent_files`
--
ALTER TABLE `talper_custom_agent_files`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `talper_custom_agent_test_history`
--
ALTER TABLE `talper_custom_agent_test_history`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `teacher_class`
--
ALTER TABLE `teacher_class`
  MODIFY `teacher_class_sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `teacher_class_member`
--
ALTER TABLE `teacher_class_member`
  MODIFY `teacher_class_member_sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `triangle_class_conclusion`
--
ALTER TABLE `triangle_class_conclusion`
  MODIFY `conclusion_sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `triangle_conclusion`
--
ALTER TABLE `triangle_conclusion`
  MODIFY `conclusion_id` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `triangle_conclusion_proofs`
--
ALTER TABLE `triangle_conclusion_proofs`
  MODIFY `proof_sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `triangle_proof`
--
ALTER TABLE `triangle_proof`
  MODIFY `proof_id` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `triangle_thought`
--
ALTER TABLE `triangle_thought`
  MODIFY `thought_id` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `triangle_thought_comment`
--
ALTER TABLE `triangle_thought_comment`
  MODIFY `comment_sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `triangle_thought_proofs`
--
ALTER TABLE `triangle_thought_proofs`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `users_pwd`
--
ALTER TABLE `users_pwd`
  MODIFY `id` int NOT NULL AUTO_INCREMENT COMMENT '密碼流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `user_account_change`
--
ALTER TABLE `user_account_change`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `user_account_history`
--
ALTER TABLE `user_account_history`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `user_activity`
--
ALTER TABLE `user_activity`
  MODIFY `sn` int UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `user_combine_history`
--
ALTER TABLE `user_combine_history`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `user_entering_interactive_learning_log`
--
ALTER TABLE `user_entering_interactive_learning_log`
  MODIFY `sn` bigint UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `user_family`
--
ALTER TABLE `user_family`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `user_feedback`
--
ALTER TABLE `user_feedback`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `user_games_play_time`
--
ALTER TABLE `user_games_play_time`
  MODIFY `sn` int UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `user_group`
--
ALTER TABLE `user_group`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `user_group_20230316`
--
ALTER TABLE `user_group_20230316`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `user_info`
--
ALTER TABLE `user_info`
  MODIFY `uid` int UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `user_info_temp`
--
ALTER TABLE `user_info_temp`
  MODIFY `uid` int UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `user_login_log`
--
ALTER TABLE `user_login_log`
  MODIFY `sn` bigint UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `user_login_log_status`
--
ALTER TABLE `user_login_log_status`
  MODIFY `sn` bigint UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `user_map_permission`
--
ALTER TABLE `user_map_permission`
  MODIFY `sn` smallint UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `user_map_permission_log`
--
ALTER TABLE `user_map_permission_log`
  MODIFY `sn` smallint UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `user_past_password`
--
ALTER TABLE `user_past_password`
  MODIFY `sn` bigint NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `user_play_game_log`
--
ALTER TABLE `user_play_game_log`
  MODIFY `game_log_sn` int UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '流水號';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `user_recovery_record`
--
ALTER TABLE `user_recovery_record`
  MODIFY `sn` bigint NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `user_status_log`
--
ALTER TABLE `user_status_log`
  MODIFY `log_sn` bigint UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '紀錄碼';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `video_concept_item`
--
ALTER TABLE `video_concept_item`
  MODIFY `video_item_sn` int NOT NULL AUTO_INCREMENT COMMENT '影片試題id';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `video_concept_item3`
--
ALTER TABLE `video_concept_item3`
  MODIFY `video_item_sn` int NOT NULL AUTO_INCREMENT COMMENT '影片試題id';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `video_concept_item_0730`
--
ALTER TABLE `video_concept_item_0730`
  MODIFY `video_item_sn` int NOT NULL AUTO_INCREMENT COMMENT '影片試題id';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `video_concept_item_20220427`
--
ALTER TABLE `video_concept_item_20220427`
  MODIFY `video_item_sn` int NOT NULL AUTO_INCREMENT COMMENT '影片試題id';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `video_concept_item_20220428`
--
ALTER TABLE `video_concept_item_20220428`
  MODIFY `video_item_sn` int NOT NULL AUTO_INCREMENT COMMENT '影片試題id';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `video_concept_item_plus`
--
ALTER TABLE `video_concept_item_plus`
  MODIFY `question_sn` int NOT NULL AUTO_INCREMENT COMMENT '問題id';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `video_concept_item_plus_bk`
--
ALTER TABLE `video_concept_item_plus_bk`
  MODIFY `question_sn` int NOT NULL AUTO_INCREMENT COMMENT '問題id';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `video_concept_stream_error`
--
ALTER TABLE `video_concept_stream_error`
  MODIFY `sn` bigint NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `video_exam_record`
--
ALTER TABLE `video_exam_record`
  MODIFY `video_exam_sn` int NOT NULL AUTO_INCREMENT COMMENT '影片問題回答id';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `video_exam_record_20230316`
--
ALTER TABLE `video_exam_record_20230316`
  MODIFY `video_exam_sn` int NOT NULL AUTO_INCREMENT COMMENT '影片問題回答id';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `video_note`
--
ALTER TABLE `video_note`
  MODIFY `video_note_sn` int NOT NULL AUTO_INCREMENT COMMENT '影片筆記id';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `video_noteask`
--
ALTER TABLE `video_noteask`
  MODIFY `ask_sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `video_noteask_plus`
--
ALTER TABLE `video_noteask_plus`
  MODIFY `reply_sn` int NOT NULL AUTO_INCREMENT COMMENT '影片發問回答id';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `video_noteask_plus_20230316`
--
ALTER TABLE `video_noteask_plus_20230316`
  MODIFY `reply_sn` int NOT NULL AUTO_INCREMENT COMMENT '影片發問回答id';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `video_note_favorite`
--
ALTER TABLE `video_note_favorite`
  MODIFY `favorite_sn` int NOT NULL AUTO_INCREMENT COMMENT 'favoriteID';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `video_note_favorite_20230316`
--
ALTER TABLE `video_note_favorite_20230316`
  MODIFY `favorite_sn` int NOT NULL AUTO_INCREMENT COMMENT 'favoriteID';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `video_note_feedback`
--
ALTER TABLE `video_note_feedback`
  MODIFY `feedback_sn` int NOT NULL AUTO_INCREMENT COMMENT '回饋id';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `video_note_feedback_20230316`
--
ALTER TABLE `video_note_feedback_20230316`
  MODIFY `feedback_sn` int NOT NULL AUTO_INCREMENT COMMENT '回饋id';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `video_note_recommend`
--
ALTER TABLE `video_note_recommend`
  MODIFY `video_recommend_sn` int NOT NULL AUTO_INCREMENT COMMENT '推薦筆記id';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `video_note_recommend_20230316`
--
ALTER TABLE `video_note_recommend_20230316`
  MODIFY `video_recommend_sn` int NOT NULL AUTO_INCREMENT COMMENT '推薦筆記id';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `video_review_record`
--
ALTER TABLE `video_review_record`
  MODIFY `review_sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `video_review_record_plus`
--
ALTER TABLE `video_review_record_plus`
  MODIFY `review_plus_sn` int NOT NULL AUTO_INCREMENT COMMENT '瀏覽明細id';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `video_review_record_plus_20230316`
--
ALTER TABLE `video_review_record_plus_20230316`
  MODIFY `review_plus_sn` int NOT NULL AUTO_INCREMENT COMMENT '瀏覽明細id';

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `website_module`
--
ALTER TABLE `website_module`
  MODIFY `module_id` smallint UNSIGNED NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `z_system_identity_log`
--
ALTER TABLE `z_system_identity_log`
  MODIFY `sn` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `z_system_team_feedback_status`
--
ALTER TABLE `z_system_team_feedback_status`
  MODIFY `id` int NOT NULL AUTO_INCREMENT;

--
-- 使用資料表自動遞增(AUTO_INCREMENT) `z_system_team_users`
--
ALTER TABLE `z_system_team_users`
  MODIFY `id` int NOT NULL AUTO_INCREMENT;

--
-- 已傾印資料表的限制式
--

--
-- 資料表的限制式 `api_token_data`
--
ALTER TABLE `api_token_data`
  ADD CONSTRAINT `api_token_data_ibfk_1` FOREIGN KEY (`api_client_sn`) REFERENCES `api_client_config` (`api_client_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `api_token_data_ibfk_2` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `api_token_data_ibfk_3` FOREIGN KEY (`mission_sn`) REFERENCES `mission_info` (`mission_sn`) ON DELETE CASCADE ON UPDATE CASCADE;

--
-- 資料表的限制式 `chatroom`
--
ALTER TABLE `chatroom`
  ADD CONSTRAINT `chatroom_ibfk_1` FOREIGN KEY (`mission_sn`) REFERENCES `mission_info` (`mission_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `chatroom_ibfk_2` FOREIGN KEY (`group_id`) REFERENCES `seme_group` (`group_id`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `chatroom_member`
--
ALTER TABLE `chatroom_member`
  ADD CONSTRAINT `chatroom_member_ibfk_1` FOREIGN KEY (`chatroom_id`) REFERENCES `chatroom` (`chatroom_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `chatroom_member_ibfk_2` FOREIGN KEY (`member`) REFERENCES `user_info` (`user_id`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `chat_log`
--
ALTER TABLE `chat_log`
  ADD CONSTRAINT `chat_log_ibfk_1` FOREIGN KEY (`chatroom_id`) REFERENCES `chatroom` (`chatroom_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `chat_log_ibfk_2` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `check_list_table`
--
ALTER TABLE `check_list_table`
  ADD CONSTRAINT `check_list_table_ibfk_1` FOREIGN KEY (`teacher_id`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL;

--
-- 資料表的限制式 `concept_info`
--
ALTER TABLE `concept_info`
  ADD CONSTRAINT `concept_info_ibfk_2` FOREIGN KEY (`publisher_id`) REFERENCES `publisher` (`publisher_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `concept_info_ibfk_3` FOREIGN KEY (`subject_id`) REFERENCES `subject` (`subject_id`);

--
-- 資料表的限制式 `concept_itemBank`
--
ALTER TABLE `concept_itemBank`
  ADD CONSTRAINT `concept_itemBank_ibfk_2` FOREIGN KEY (`subject_id`) REFERENCES `subject` (`subject_id`);

--
-- 資料表的限制式 `concept_itemBank_bat`
--
ALTER TABLE `concept_itemBank_bat`
  ADD CONSTRAINT `concept_itemBank_bat_ibfk_3` FOREIGN KEY (`subject_id`) REFERENCES `subject` (`subject_id`);

--
-- 資料表的限制式 `concept_itemBank_bat_inds`
--
ALTER TABLE `concept_itemBank_bat_inds`
  ADD CONSTRAINT `concept_itemBank_bat_inds_ibfk_1` FOREIGN KEY (`item_sn`) REFERENCES `concept_itemBank_bat` (`item_sn`) ON DELETE CASCADE ON UPDATE CASCADE;

--
-- 資料表的限制式 `concept_itemBank_depiction_0710`
--
ALTER TABLE `concept_itemBank_depiction_0710`
  ADD CONSTRAINT `concept_itemBank_depiction_0710_ibfk_1` FOREIGN KEY (`item_sn`) REFERENCES `concept_itemBank` (`item_sn`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `concept_itemBank_depiction_0710_ibfk_2` FOREIGN KEY (`updater`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL;

--
-- 資料表的限制式 `concept_itemBank_literacy`
--
ALTER TABLE `concept_itemBank_literacy`
  ADD CONSTRAINT `concept_itemBank_literacy_ibfk_1` FOREIGN KEY (`create_user`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL,
  ADD CONSTRAINT `concept_itemBank_literacy_ibfk_3` FOREIGN KEY (`map_sn`) REFERENCES `map_info` (`map_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `concept_itemBank_literacy_ibfk_4` FOREIGN KEY (`literacy_group_sn`) REFERENCES `concept_itemBank_literacy_group` (`literacy_group_sn`) ON DELETE SET NULL ON UPDATE SET NULL,
  ADD CONSTRAINT `concept_itemBank_literacy_ibfk_5` FOREIGN KEY (`sub_subject_id`) REFERENCES `sub_subject` (`sub_subject_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `concept_itemBank_literacy_ibfk_6` FOREIGN KEY (`subject_id`) REFERENCES `subject` (`subject_id`);

--
-- 資料表的限制式 `concept_itemBank_literacy_group`
--
ALTER TABLE `concept_itemBank_literacy_group`
  ADD CONSTRAINT `concept_itemBank_literacy_group_ibfk_1` FOREIGN KEY (`create_user`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL,
  ADD CONSTRAINT `concept_itemBank_literacy_group_ibfk_3` FOREIGN KEY (`map_sn`) REFERENCES `map_info` (`map_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `concept_itemBank_literacy_group_ibfk_4` FOREIGN KEY (`sub_subject_id`) REFERENCES `sub_subject` (`sub_subject_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `concept_itemBank_literacy_group_ibfk_5` FOREIGN KEY (`subject_id`) REFERENCES `subject` (`subject_id`);

--
-- 資料表的限制式 `concept_itemBank_literacy_interactive`
--
ALTER TABLE `concept_itemBank_literacy_interactive`
  ADD CONSTRAINT `__concept_itemBank_literacy_interactive_ibfk_5` FOREIGN KEY (`subject_id`) REFERENCES `subject` (`subject_id`),
  ADD CONSTRAINT `concept_itemBank_literacy_interactive_ibfk_1` FOREIGN KEY (`api_config_sn`) REFERENCES `api_client_config` (`api_client_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `concept_itemBank_literacy_interactive_ibfk_3` FOREIGN KEY (`map_sn`) REFERENCES `map_info` (`map_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `concept_itemBank_literacy_interactive_ibfk_4` FOREIGN KEY (`sub_subject_id`) REFERENCES `sub_subject` (`sub_subject_id`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `concept_itemBank_option_0710`
--
ALTER TABLE `concept_itemBank_option_0710`
  ADD CONSTRAINT `concept_itemBank_option_0710_ibfk_1` FOREIGN KEY (`item_sn`) REFERENCES `concept_itemBank` (`item_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `concept_itemBank_option_0710_ibfk_2` FOREIGN KEY (`updater`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL;

--
-- 資料表的限制式 `concept_item_interactive`
--
ALTER TABLE `concept_item_interactive`
  ADD CONSTRAINT `concept_item_interactive_ibfk_2` FOREIGN KEY (`create_user_id`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL,
  ADD CONSTRAINT `concept_item_interactive_ibfk_4` FOREIGN KEY (`subject_id`) REFERENCES `subject` (`subject_id`);

--
-- 資料表的限制式 `concept_its`
--
ALTER TABLE `concept_its`
  ADD CONSTRAINT `concept_its_ibfk_1` FOREIGN KEY (`subject_id`) REFERENCES `subject` (`subject_id`);

--
-- 資料表的限制式 `concept_paper`
--
ALTER TABLE `concept_paper`
  ADD CONSTRAINT `concept_paper_ibfk_1` FOREIGN KEY (`publisher`) REFERENCES `publisher` (`publisher_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `concept_paper_ibfk_3` FOREIGN KEY (`ower_id`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL,
  ADD CONSTRAINT `concept_paper_ibfk_4` FOREIGN KEY (`subject`) REFERENCES `subject` (`subject_id`);

--
-- 資料表的限制式 `concept_priori`
--
ALTER TABLE `concept_priori`
  ADD CONSTRAINT `concept_priori_ibfk_2` FOREIGN KEY (`organization_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `concept_priori_ibfk_3` FOREIGN KEY (`create_user`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL,
  ADD CONSTRAINT `concept_priori_ibfk_4` FOREIGN KEY (`cp_info_sn`) REFERENCES `concept_priori_info` (`sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `concept_priori_ibfk_5` FOREIGN KEY (`subject_id`) REFERENCES `subject` (`subject_id`);

--
-- 資料表的限制式 `concept_priori_info`
--
ALTER TABLE `concept_priori_info`
  ADD CONSTRAINT `concept_priori_info_ibfk_2` FOREIGN KEY (`create_user`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL,
  ADD CONSTRAINT `concept_priori_info_ibfk_3` FOREIGN KEY (`subject_id`) REFERENCES `subject` (`subject_id`);

--
-- 資料表的限制式 `contestexam_item`
--
ALTER TABLE `contestexam_item`
  ADD CONSTRAINT `contestexam_item_ibfk_1` FOREIGN KEY (`keyboard_sn`) REFERENCES `exam_onscreen_keyboard` (`keyboard_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `contestexam_item_ibfk_2` FOREIGN KEY (`item_type`) REFERENCES `exam_item_type` (`item_type`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `contestexam_item_ibfk_3` FOREIGN KEY (`exam_type`) REFERENCES `exam_type` (`exam_type`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `contestexam_item_ibfk_4` FOREIGN KEY (`map_sn`) REFERENCES `map_info` (`map_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `contestexam_item_ibfk_5` FOREIGN KEY (`answer_option_type`) REFERENCES `exam_item_answer_option_type` (`answer_option_type`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `contestexam_item_ibfk_6` FOREIGN KEY (`updater`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL;

--
-- 資料表的限制式 `contestexam_itemgroup`
--
ALTER TABLE `contestexam_itemgroup`
  ADD CONSTRAINT `contestexam_itemgroup_ibfk_1` FOREIGN KEY (`map_sn`) REFERENCES `map_info` (`map_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `contestexam_itemgroup_ibfk_2` FOREIGN KEY (`exam_type`) REFERENCES `exam_type` (`exam_type`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `contestexam_itemgroup_ibfk_3` FOREIGN KEY (`updater`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL;

--
-- 資料表的限制式 `contestexam_itemgroup_detail`
--
ALTER TABLE `contestexam_itemgroup_detail`
  ADD CONSTRAINT `contestexam_itemgroup_detail_ibfk_1` FOREIGN KEY (`group_sn`) REFERENCES `contestexam_itemgroup` (`group_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `contestexam_itemgroup_detail_ibfk_2` FOREIGN KEY (`item_sn`) REFERENCES `contestexam_item` (`item_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `contestexam_itemgroup_detail_ibfk_3` FOREIGN KEY (`updater`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL;

--
-- 資料表的限制式 `contestexam_item_answer_option`
--
ALTER TABLE `contestexam_item_answer_option`
  ADD CONSTRAINT `contestexam_item_answer_option_ibfk_1` FOREIGN KEY (`item_sn`) REFERENCES `contestexam_item` (`item_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `contestexam_item_answer_option_ibfk_2` FOREIGN KEY (`updater`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL;

--
-- 資料表的限制式 `contestexam_item_indicator_mapping`
--
ALTER TABLE `contestexam_item_indicator_mapping`
  ADD CONSTRAINT `contestexam_item_indicator_mapping_ibfk_1` FOREIGN KEY (`item_sn`) REFERENCES `contestexam_item` (`item_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `contestexam_item_indicator_mapping_ibfk_3` FOREIGN KEY (`updater`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL;

--
-- 資料表的限制式 `contestexam_paper`
--
ALTER TABLE `contestexam_paper`
  ADD CONSTRAINT `contestexam_paper_ibfk_1` FOREIGN KEY (`map_sn`) REFERENCES `map_info` (`map_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `contestexam_paper_ibfk_2` FOREIGN KEY (`exam_type`) REFERENCES `exam_type` (`exam_type`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `contestexam_paper_ibfk_3` FOREIGN KEY (`updater`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL;

--
-- 資料表的限制式 `contestexam_paper_detail`
--
ALTER TABLE `contestexam_paper_detail`
  ADD CONSTRAINT `contestexam_paper_detail_ibfk_1` FOREIGN KEY (`paper_sn`) REFERENCES `contestexam_paper` (`paper_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `contestexam_paper_detail_ibfk_2` FOREIGN KEY (`group_sn`) REFERENCES `contestexam_itemgroup` (`group_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `contestexam_paper_detail_ibfk_3` FOREIGN KEY (`item_sn`) REFERENCES `contestexam_item` (`item_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `contestexam_paper_detail_ibfk_4` FOREIGN KEY (`updater`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL;

--
-- 資料表的限制式 `contestexam_record`
--
ALTER TABLE `contestexam_record`
  ADD CONSTRAINT `contestexam_record_ibfk_1` FOREIGN KEY (`mission_sn`) REFERENCES `mission_info` (`mission_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `contestexam_record_ibfk_2` FOREIGN KEY (`paper_sn`) REFERENCES `contestexam_paper` (`paper_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `contestexam_record_ibfk_3` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL;

--
-- 資料表的限制式 `contestexam_record_detail`
--
ALTER TABLE `contestexam_record_detail`
  ADD CONSTRAINT `contestexam_record_detail_ibfk_1` FOREIGN KEY (`record_sn`) REFERENCES `contestexam_record` (`record_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `contestexam_record_detail_ibfk_2` FOREIGN KEY (`item_sn`) REFERENCES `contestexam_item` (`item_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `contestexam_record_detail_ibfk_3` FOREIGN KEY (`option_sn`) REFERENCES `contestexam_item_answer_option` (`option_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `contestexam_record_detail_ibfk_4` FOREIGN KEY (`group_sn`) REFERENCES `contestexam_itemgroup` (`group_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `contest_agreement`
--
ALTER TABLE `contest_agreement`
  ADD CONSTRAINT `contest_agreement_ibfk_1` FOREIGN KEY (`contest_sn`) REFERENCES `contest_info` (`sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `contest_agreement_ibfk_2` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE;

--
-- 資料表的限制式 `contest_certificate`
--
ALTER TABLE `contest_certificate`
  ADD CONSTRAINT `contest_certificate_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `contest_certificate_ibfk_2` FOREIGN KEY (`org_id`) REFERENCES `organization` (`organization_id`),
  ADD CONSTRAINT `contest_certificate_ibfk_3` FOREIGN KEY (`contest_sn`) REFERENCES `contest_info` (`sn`);

--
-- 資料表的限制式 `contest_content`
--
ALTER TABLE `contest_content`
  ADD CONSTRAINT `contest_content_ibfk_1` FOREIGN KEY (`contest_sn`) REFERENCES `contest_info` (`sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `contest_content_ibfk_2` FOREIGN KEY (`teacher_id`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL,
  ADD CONSTRAINT `contest_content_ibfk_3` FOREIGN KEY (`group_sn`) REFERENCES `contest_group` (`sn`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `contest_dailyreport`
--
ALTER TABLE `contest_dailyreport`
  ADD CONSTRAINT `contest_dailyreport_ibfk_1` FOREIGN KEY (`org_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `contest_dailyreport_ibfk_2` FOREIGN KEY (`contest_sn`) REFERENCES `contest_info` (`sn`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `contest_group`
--
ALTER TABLE `contest_group`
  ADD CONSTRAINT `contest_group_ibfk_1` FOREIGN KEY (`org_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `contest_group_ibfk_2` FOREIGN KEY (`teacher_id`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL;

--
-- 資料表的限制式 `contest_group_comparison`
--
ALTER TABLE `contest_group_comparison`
  ADD CONSTRAINT `contest_group_comparison_ibfk_1` FOREIGN KEY (`warm_sn`) REFERENCES `contest_group` (`sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `contest_group_comparison_ibfk_2` FOREIGN KEY (`challenge_sn`) REFERENCES `contest_group` (`sn`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `contest_itembank`
--
ALTER TABLE `contest_itembank`
  ADD CONSTRAINT `contest_itembank_ibfk_1` FOREIGN KEY (`contest_sn`) REFERENCES `contest_info` (`sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `contest_itembank_ibfk_3` FOREIGN KEY (`map_sn`) REFERENCES `map_info` (`map_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `contest_itembank_ibfk_4` FOREIGN KEY (`sub_subject_id`) REFERENCES `sub_subject` (`sub_subject_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `contest_itembank_ibfk_5` FOREIGN KEY (`api_config_sn`) REFERENCES `api_client_config` (`api_client_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `contest_itembank_ibfk_6` FOREIGN KEY (`subject_id`) REFERENCES `subject` (`subject_id`);

--
-- 資料表的限制式 `contest_itembank_2021mathscience`
--
ALTER TABLE `contest_itembank_2021mathscience`
  ADD CONSTRAINT `contest_itembank_2021mathscience_ibfk_1` FOREIGN KEY (`contest_sn`) REFERENCES `contest_info` (`sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `contest_itembank_2021mathscience_ibfk_3` FOREIGN KEY (`sub_subject_id`) REFERENCES `sub_subject` (`sub_subject_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `contest_itembank_2021mathscience_ibfk_4` FOREIGN KEY (`map_sn`) REFERENCES `map_info` (`map_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `contest_itembank_2021mathscience_ibfk_5` FOREIGN KEY (`subject_id`) REFERENCES `subject` (`subject_id`);

--
-- 資料表的限制式 `contest_itembank_group`
--
ALTER TABLE `contest_itembank_group`
  ADD CONSTRAINT `contest_itembank_group_ibfk_1` FOREIGN KEY (`contest_sn`) REFERENCES `contest_info` (`sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `contest_itembank_group_ibfk_3` FOREIGN KEY (`map_sn`) REFERENCES `map_info` (`map_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `contest_itembank_group_ibfk_4` FOREIGN KEY (`sub_subject_id`) REFERENCES `sub_subject` (`sub_subject_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `contest_itembank_group_ibfk_5` FOREIGN KEY (`create_user`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL,
  ADD CONSTRAINT `contest_itembank_group_ibfk_6` FOREIGN KEY (`subject_id`) REFERENCES `subject` (`subject_id`);

--
-- 資料表的限制式 `contest_itemBank_literacy`
--
ALTER TABLE `contest_itemBank_literacy`
  ADD CONSTRAINT `contest_itemBank_literacy_ibfk_1` FOREIGN KEY (`contest_sn`) REFERENCES `contest_info` (`sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `contest_itemBank_literacy_ibfk_3` FOREIGN KEY (`map_sn`) REFERENCES `map_info` (`map_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `contest_itemBank_literacy_ibfk_4` FOREIGN KEY (`sub_subject_id`) REFERENCES `sub_subject` (`sub_subject_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `contest_itemBank_literacy_ibfk_5` FOREIGN KEY (`literacy_group_sn`) REFERENCES `contest_itembank_group` (`literacy_group_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `contest_itemBank_literacy_ibfk_6` FOREIGN KEY (`create_user`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL,
  ADD CONSTRAINT `contest_itemBank_literacy_ibfk_7` FOREIGN KEY (`subject_id`) REFERENCES `subject` (`subject_id`);

--
-- 資料表的限制式 `contest_itembank_summer`
--
ALTER TABLE `contest_itembank_summer`
  ADD CONSTRAINT `contest_itembank_summer_ibfk_1` FOREIGN KEY (`contest_sn`) REFERENCES `contest_info` (`sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `contest_itembank_summer_ibfk_3` FOREIGN KEY (`map_sn`) REFERENCES `map_info` (`map_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `contest_itembank_summer_ibfk_4` FOREIGN KEY (`sub_subject_id`) REFERENCES `sub_subject` (`sub_subject_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `contest_itembank_summer_ibfk_5` FOREIGN KEY (`subject_id`) REFERENCES `subject` (`subject_id`);

--
-- 資料表的限制式 `contest_publisher_mapping`
--
ALTER TABLE `contest_publisher_mapping`
  ADD CONSTRAINT `contest_publisher_mapping_ibfk_1` FOREIGN KEY (`publisher_id`) REFERENCES `publisher` (`publisher_id`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `contest_publisher_mapping_2021mathscience`
--
ALTER TABLE `contest_publisher_mapping_2021mathscience`
  ADD CONSTRAINT `contest_publisher_mapping_2021mathscience_ibfk_1` FOREIGN KEY (`publisher_id`) REFERENCES `publisher` (`publisher_id`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `contest_record`
--
ALTER TABLE `contest_record`
  ADD CONSTRAINT `contest_record_ibfk_1` FOREIGN KEY (`contest_info_sn`) REFERENCES `contest_info` (`sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `contest_record_ibfk_2` FOREIGN KEY (`mission_sn`) REFERENCES `mission_info` (`mission_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `contest_record_ibfk_4` FOREIGN KEY (`map_sn`) REFERENCES `map_info` (`map_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `contest_record_ibfk_5` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `contest_record_ibfk_6` FOREIGN KEY (`subject_id`) REFERENCES `subject` (`subject_id`);

--
-- 資料表的限制式 `contest_record_literacy`
--
ALTER TABLE `contest_record_literacy`
  ADD CONSTRAINT `contest_record_literacy_ibfk_1` FOREIGN KEY (`contest_sn`) REFERENCES `contest_info` (`sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `contest_record_literacy_ibfk_2` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE;

--
-- 資料表的限制式 `contest_record_respond`
--
ALTER TABLE `contest_record_respond`
  ADD CONSTRAINT `contest_record_respond_ibfk_1` FOREIGN KEY (`contest_sn`) REFERENCES `contest_info` (`sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `contest_record_respond_ibfk_2` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `contest_record_respond_ibfk_3` FOREIGN KEY (`org_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `contest_signup`
--
ALTER TABLE `contest_signup`
  ADD CONSTRAINT `contest_signup_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `contest_signup_ibfk_2` FOREIGN KEY (`contest_sn`) REFERENCES `contest_info` (`sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `contest_signup_ibfk_3` FOREIGN KEY (`organization_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `contest_signup_ibfk_4` FOREIGN KEY (`group_sn`) REFERENCES `contest_group` (`sn`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `conversation`
--
ALTER TABLE `conversation`
  ADD CONSTRAINT `conversation_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `conversation_ibfk_2` FOREIGN KEY (`mission_sn`) REFERENCES `mission_info` (`mission_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `conversation_ibfk_3` FOREIGN KEY (`item_sn`) REFERENCES `concept_itemBank` (`item_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `conversation_access`
--
ALTER TABLE `conversation_access`
  ADD CONSTRAINT `conversation_access_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE;

--
-- 資料表的限制式 `conversation_detail`
--
ALTER TABLE `conversation_detail`
  ADD CONSTRAINT `conversation_detail_ibfk_1` FOREIGN KEY (`conversation_sn`) REFERENCES `conversation` (`sn`) ON DELETE CASCADE ON UPDATE CASCADE;

--
-- 資料表的限制式 `conversation_old`
--
ALTER TABLE `conversation_old`
  ADD CONSTRAINT `conversation_old_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `conversation_old_ibfk_2` FOREIGN KEY (`mission_sn`) REFERENCES `mission_info` (`mission_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `conversation_old_ibfk_3` FOREIGN KEY (`item_sn`) REFERENCES `concept_itemBank` (`item_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `cooperation_organization`
--
ALTER TABLE `cooperation_organization`
  ADD CONSTRAINT `cooperation_organization_ibfk_1` FOREIGN KEY (`organization_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `course_subject`
--
ALTER TABLE `course_subject`
  ADD CONSTRAINT `course_subject_ibfk_1` FOREIGN KEY (`subject_id`) REFERENCES `subject` (`subject_id`);

--
-- 資料表的限制式 `cross_school_student`
--
ALTER TABLE `cross_school_student`
  ADD CONSTRAINT `cross_school_student_ibfk_1` FOREIGN KEY (`student_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `cross_school_student_ibfk_2` FOREIGN KEY (`teacher_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE;

--
-- 資料表的限制式 `dbuser`
--
ALTER TABLE `dbuser`
  ADD CONSTRAINT `dbuser_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL;

--
-- 資料表的限制式 `eduexam_item`
--
ALTER TABLE `eduexam_item`
  ADD CONSTRAINT `eduexam_item_ibfk_10` FOREIGN KEY (`keyboard_sn`) REFERENCES `exam_onscreen_keyboard` (`keyboard_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `eduexam_item_ibfk_11` FOREIGN KEY (`item_type`) REFERENCES `exam_item_type` (`item_type`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `eduexam_item_ibfk_12` FOREIGN KEY (`exam_type`) REFERENCES `exam_type` (`exam_type`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `eduexam_item_ibfk_5` FOREIGN KEY (`updater`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL,
  ADD CONSTRAINT `eduexam_item_ibfk_6` FOREIGN KEY (`map_sn`) REFERENCES `map_info` (`map_sn`),
  ADD CONSTRAINT `eduexam_item_ibfk_9` FOREIGN KEY (`answer_option_type`) REFERENCES `exam_item_answer_option_type` (`answer_option_type`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `eduexam_itemgroup`
--
ALTER TABLE `eduexam_itemgroup`
  ADD CONSTRAINT `eduexam_itemgroup_ibfk_1` FOREIGN KEY (`map_sn`) REFERENCES `map_info` (`map_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `eduexam_itemgroup_ibfk_3` FOREIGN KEY (`updater`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL,
  ADD CONSTRAINT `eduexam_itemgroup_ibfk_4` FOREIGN KEY (`exam_type`) REFERENCES `exam_type` (`exam_type`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `eduexam_itemgroup_detail`
--
ALTER TABLE `eduexam_itemgroup_detail`
  ADD CONSTRAINT `eduexam_itemgroup_detail_ibfk_1` FOREIGN KEY (`group_sn`) REFERENCES `eduexam_itemgroup` (`group_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `eduexam_itemgroup_detail_ibfk_3` FOREIGN KEY (`updater`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL,
  ADD CONSTRAINT `eduexam_itemgroup_detail_ibfk_4` FOREIGN KEY (`item_sn`) REFERENCES `eduexam_item` (`item_sn`);

--
-- 資料表的限制式 `eduexam_item_answer_option`
--
ALTER TABLE `eduexam_item_answer_option`
  ADD CONSTRAINT `eduexam_item_answer_option_ibfk_1` FOREIGN KEY (`item_sn`) REFERENCES `eduexam_item` (`item_sn`),
  ADD CONSTRAINT `eduexam_item_answer_option_ibfk_2` FOREIGN KEY (`updater`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL;

--
-- 資料表的限制式 `eduexam_item_indicator_mapping`
--
ALTER TABLE `eduexam_item_indicator_mapping`
  ADD CONSTRAINT `eduexam_item_indicator_mapping_ibfk_1` FOREIGN KEY (`item_sn`) REFERENCES `eduexam_item` (`item_sn`),
  ADD CONSTRAINT `eduexam_item_indicator_mapping_ibfk_5` FOREIGN KEY (`updater`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL;

--
-- 資料表的限制式 `eduexam_paper`
--
ALTER TABLE `eduexam_paper`
  ADD CONSTRAINT `eduexam_paper_ibfk_1` FOREIGN KEY (`updater`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL,
  ADD CONSTRAINT `eduexam_paper_ibfk_2` FOREIGN KEY (`map_sn`) REFERENCES `map_info` (`map_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `eduexam_paper_ibfk_3` FOREIGN KEY (`exam_type`) REFERENCES `exam_type` (`exam_type`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `eduexam_paper_detail`
--
ALTER TABLE `eduexam_paper_detail`
  ADD CONSTRAINT `eduexam_paper_detail_ibfk_1` FOREIGN KEY (`paper_sn`) REFERENCES `eduexam_paper` (`paper_sn`),
  ADD CONSTRAINT `eduexam_paper_detail_ibfk_2` FOREIGN KEY (`updater`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL,
  ADD CONSTRAINT `eduexam_paper_detail_ibfk_3` FOREIGN KEY (`group_sn`) REFERENCES `eduexam_itemgroup` (`group_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `eduexam_paper_detail_ibfk_4` FOREIGN KEY (`item_sn`) REFERENCES `eduexam_item` (`item_sn`);

--
-- 資料表的限制式 `eduexam_record`
--
ALTER TABLE `eduexam_record`
  ADD CONSTRAINT `eduexam_record_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL,
  ADD CONSTRAINT `eduexam_record_ibfk_2` FOREIGN KEY (`mission_sn`) REFERENCES `mission_info` (`mission_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `eduexam_record_ibfk_3` FOREIGN KEY (`paper_sn`) REFERENCES `eduexam_paper` (`paper_sn`);

--
-- 資料表的限制式 `eduexam_record_detail`
--
ALTER TABLE `eduexam_record_detail`
  ADD CONSTRAINT `eduexam_record_detail_ibfk_1` FOREIGN KEY (`record_sn`) REFERENCES `eduexam_record` (`record_sn`),
  ADD CONSTRAINT `eduexam_record_detail_ibfk_2` FOREIGN KEY (`item_sn`) REFERENCES `eduexam_item` (`item_sn`),
  ADD CONSTRAINT `eduexam_record_detail_ibfk_3` FOREIGN KEY (`option_sn`) REFERENCES `eduexam_item_answer_option` (`option_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `eduexam_record_detail_ibfk_4` FOREIGN KEY (`group_sn`) REFERENCES `eduexam_itemgroup` (`group_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `egame_exam_record`
--
ALTER TABLE `egame_exam_record`
  ADD CONSTRAINT `egame_exam_record_ibfk_2` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE;

--
-- 資料表的限制式 `exam_country_student_tmp`
--
ALTER TABLE `exam_country_student_tmp`
  ADD CONSTRAINT `exam_country_student_tmp_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `exam_country_student_tmp_ibfk_2` FOREIGN KEY (`organization_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `exam_onscreen_keyboard`
--
ALTER TABLE `exam_onscreen_keyboard`
  ADD CONSTRAINT `exam_onscreen_keyboard_ibfk_1` FOREIGN KEY (`map_sn`) REFERENCES `map_info` (`map_sn`);

--
-- 資料表的限制式 `exam_paper_access`
--
ALTER TABLE `exam_paper_access`
  ADD CONSTRAINT `exam_paper_access_ibfk_1` FOREIGN KEY (`school_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `exam_record`
--
ALTER TABLE `exam_record`
  ADD CONSTRAINT `exam_record_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `exam_record_ibfk_2` FOREIGN KEY (`firm_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `exam_record_aassessment_detail`
--
ALTER TABLE `exam_record_aassessment_detail`
  ADD CONSTRAINT `exam_record_aassessment_detail_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE CASCADE;

--
-- 資料表的限制式 `exam_record_bat_detail`
--
ALTER TABLE `exam_record_bat_detail`
  ADD CONSTRAINT `exam_record_bat_detail_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `exam_record_bat_detail_ibfk_3` FOREIGN KEY (`organization_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `exam_record_bat_detail_ibfk_4` FOREIGN KEY (`subject_id`) REFERENCES `subject` (`subject_id`);

--
-- 資料表的限制式 `exam_record_dynamic`
--
ALTER TABLE `exam_record_dynamic`
  ADD CONSTRAINT `exam_record_dynamic_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `exam_record_dynamic_ibfk_2` FOREIGN KEY (`mission_sn`) REFERENCES `mission_info` (`mission_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `exam_record_indicate`
--
ALTER TABLE `exam_record_indicate`
  ADD CONSTRAINT `exam_record_indicate_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `exam_record_indicate_ibfk_2` FOREIGN KEY (`result_sn`) REFERENCES `mission_info` (`mission_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `exam_record_indicate_tmp`
--
ALTER TABLE `exam_record_indicate_tmp`
  ADD CONSTRAINT `exam_record_indicate_tmp_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `exam_record_indicate_tmp_ibfk_2` FOREIGN KEY (`result_sn`) REFERENCES `mission_info` (`mission_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `exam_record_indicator`
--
ALTER TABLE `exam_record_indicator`
  ADD CONSTRAINT `exam_record_indicator_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `exam_record_indicator_ibfk_2` FOREIGN KEY (`mission_sn`) REFERENCES `mission_info` (`mission_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `exam_record_indicator_ibfk_3` FOREIGN KEY (`paper_sn`) REFERENCES `concept_paper` (`paper_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `exam_record_indicator_ibfk_4` FOREIGN KEY (`map_sn`) REFERENCES `map_info` (`map_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `exam_record_indicator_bat`
--
ALTER TABLE `exam_record_indicator_bat`
  ADD CONSTRAINT `exam_record_indicator_bat_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `exam_record_indicator_bat_ibfk_2` FOREIGN KEY (`paper_sn`) REFERENCES `concept_paper` (`paper_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `exam_record_indicator_bat_ibfk_3` FOREIGN KEY (`map_sn`) REFERENCES `map_info` (`map_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `exam_record_indicator_bat_ibfk_4` FOREIGN KEY (`mission_sn`) REFERENCES `mission_info` (`mission_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `exam_record_indicator_bat_tmp`
--
ALTER TABLE `exam_record_indicator_bat_tmp`
  ADD CONSTRAINT `exam_record_indicator_bat_tmp_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `exam_record_indicator_bat_tmp_ibfk_2` FOREIGN KEY (`paper_sn`) REFERENCES `concept_paper` (`paper_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `exam_record_indicator_bat_tmp_ibfk_3` FOREIGN KEY (`mission_sn`) REFERENCES `mission_info` (`mission_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `exam_record_indicator_cap`
--
ALTER TABLE `exam_record_indicator_cap`
  ADD CONSTRAINT `exam_record_indicator_cap_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `exam_record_indicator_cap_ibfk_2` FOREIGN KEY (`paper_sn`) REFERENCES `concept_paper` (`paper_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `exam_record_indicator_cap_ibfk_3` FOREIGN KEY (`map_sn`) REFERENCES `map_info` (`map_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `exam_record_indicator_cap_ibfk_4` FOREIGN KEY (`mission_sn`) REFERENCES `mission_info` (`mission_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `exam_record_indicator_cap_tmp`
--
ALTER TABLE `exam_record_indicator_cap_tmp`
  ADD CONSTRAINT `exam_record_indicator_cap_tmp_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `exam_record_indicator_cap_tmp_ibfk_2` FOREIGN KEY (`paper_sn`) REFERENCES `concept_paper` (`paper_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `exam_record_indicator_cap_tmp_ibfk_3` FOREIGN KEY (`mission_sn`) REFERENCES `mission_info` (`mission_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `exam_record_indicator_tmp`
--
ALTER TABLE `exam_record_indicator_tmp`
  ADD CONSTRAINT `exam_record_indicator_tmp_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `exam_record_indicator_tmp_ibfk_2` FOREIGN KEY (`paper_sn`) REFERENCES `concept_paper` (`paper_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `exam_record_indicator_tmp_ibfk_3` FOREIGN KEY (`mission_sn`) REFERENCES `mission_info` (`mission_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `exam_record_interactive`
--
ALTER TABLE `exam_record_interactive`
  ADD CONSTRAINT `exam_record_interactive_ibfk_1` FOREIGN KEY (`mission_sn`) REFERENCES `mission_info` (`mission_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `exam_record_interactive_ibfk_3` FOREIGN KEY (`map_sn`) REFERENCES `map_info` (`map_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `exam_record_interactive_ibfk_4` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `exam_record_interactive_ibfk_5` FOREIGN KEY (`subject_id`) REFERENCES `subject` (`subject_id`);

--
-- 資料表的限制式 `exam_record_its`
--
ALTER TABLE `exam_record_its`
  ADD CONSTRAINT `exam_record_its_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE;

--
-- 資料表的限制式 `exam_record_literacy`
--
ALTER TABLE `exam_record_literacy`
  ADD CONSTRAINT `exam_record_literacy_ibfk_1` FOREIGN KEY (`mission_sn`) REFERENCES `mission_info` (`mission_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `exam_record_literacy_ibfk_3` FOREIGN KEY (`map_sn`) REFERENCES `map_info` (`map_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `exam_record_literacy_ibfk_7` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `exam_record_literacy_ibfk_8` FOREIGN KEY (`subject_id`) REFERENCES `subject` (`subject_id`);

--
-- 資料表的限制式 `exam_record_priori_detail`
--
ALTER TABLE `exam_record_priori_detail`
  ADD CONSTRAINT `exam_record_priori_detail_ibfk_3` FOREIGN KEY (`organization_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `exam_record_priori_detail_ibfk_4` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL,
  ADD CONSTRAINT `exam_record_priori_detail_ibfk_5` FOREIGN KEY (`subject_id`) REFERENCES `subject` (`subject_id`);

--
-- 資料表的限制式 `exam_record_priori_error`
--
ALTER TABLE `exam_record_priori_error`
  ADD CONSTRAINT `exam_record_priori_error_ibfk_1` FOREIGN KEY (`cp_info_sn`) REFERENCES `concept_priori_info` (`sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `exam_record_priori_error_ibfk_2` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL,
  ADD CONSTRAINT `exam_record_priori_error_ibfk_3` FOREIGN KEY (`exam_sn`) REFERENCES `exam_record_priori` (`exam_sn`) ON DELETE SET NULL ON UPDATE SET NULL;

--
-- 資料表的限制式 `exam_record_priori_his`
--
ALTER TABLE `exam_record_priori_his`
  ADD CONSTRAINT `exam_record_priori_his_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `exam_record_priori_his_ibfk_2` FOREIGN KEY (`mission_sn`) REFERENCES `mission_info` (`mission_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `exam_record_priori_his_ibfk_3` FOREIGN KEY (`exam_sn`) REFERENCES `exam_record_indicator_bat` (`exam_sn`) ON DELETE CASCADE ON UPDATE CASCADE;

--
-- 資料表的限制式 `exam_record_tmp`
--
ALTER TABLE `exam_record_tmp`
  ADD CONSTRAINT `exam_record_tmp_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `exam_record_tmp_ibfk_2` FOREIGN KEY (`firm_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `exam_type`
--
ALTER TABLE `exam_type`
  ADD CONSTRAINT `exam_type_ibfk_1` FOREIGN KEY (`mission_type`) REFERENCES `mission_type` (`mission_type_id`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `export_school`
--
ALTER TABLE `export_school`
  ADD CONSTRAINT `export_school_ibfk_1` FOREIGN KEY (`org_id`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL,
  ADD CONSTRAINT `export_school_ibfk_2` FOREIGN KEY (`new_id`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL,
  ADD CONSTRAINT `export_school_ibfk_3` FOREIGN KEY (`org_school`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `export_school_ibfk_4` FOREIGN KEY (`new_school`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `export_school_ibfk_5` FOREIGN KEY (`exporter`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL,
  ADD CONSTRAINT `export_school_ibfk_6` FOREIGN KEY (`user_level`) REFERENCES `user_access` (`access_level`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `group_check_result`
--
ALTER TABLE `group_check_result`
  ADD CONSTRAINT `group_check_result_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `group_check_result_ibfk_2` FOREIGN KEY (`check_list_sn`) REFERENCES `check_list_table` (`check_list_table_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `group_check_result_ibfk_3` FOREIGN KEY (`mission_sn`) REFERENCES `mission_info` (`mission_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `group_member_score_result`
--
ALTER TABLE `group_member_score_result`
  ADD CONSTRAINT `group_member_score_result_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `group_member_score_result_ibfk_2` FOREIGN KEY (`score_user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `group_member_score_result_ibfk_3` FOREIGN KEY (`check_list_sn`) REFERENCES `check_list_table` (`check_list_table_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `group_member_score_result_ibfk_4` FOREIGN KEY (`mission_sn`) REFERENCES `mission_info` (`mission_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `group_role`
--
ALTER TABLE `group_role`
  ADD CONSTRAINT `group_role_ibfk_1` FOREIGN KEY (`teacher_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE;

--
-- 資料表的限制式 `group_score_ingroup_result`
--
ALTER TABLE `group_score_ingroup_result`
  ADD CONSTRAINT `group_score_ingroup_result_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `group_score_ingroup_result_ibfk_2` FOREIGN KEY (`score_user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `group_score_ingroup_result_ibfk_3` FOREIGN KEY (`check_list_sn`) REFERENCES `check_list_table` (`check_list_table_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `group_score_ingroup_result_ibfk_4` FOREIGN KEY (`mission_sn`) REFERENCES `mission_info` (`mission_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `group_score_ingroup_result_ibfk_5` FOREIGN KEY (`group_id`) REFERENCES `seme_group` (`group_id`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `group_score_result`
--
ALTER TABLE `group_score_result`
  ADD CONSTRAINT `group_score_result_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `group_score_result_ibfk_2` FOREIGN KEY (`check_list_sn`) REFERENCES `check_list_table` (`check_list_table_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `group_score_result_ibfk_3` FOREIGN KEY (`user_group_id`) REFERENCES `seme_group` (`group_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `group_score_result_ibfk_4` FOREIGN KEY (`group_id`) REFERENCES `seme_group` (`group_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `group_score_result_ibfk_5` FOREIGN KEY (`mission_sn`) REFERENCES `mission_info` (`mission_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `high_school_dept_mapping`
--
ALTER TABLE `high_school_dept_mapping`
  ADD CONSTRAINT `high_school_dept_mapping_ibfk_1` FOREIGN KEY (`school_type`) REFERENCES `high_school_type` (`school_type`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `high_school_dept_mapping_ibfk_2` FOREIGN KEY (`group_sn`) REFERENCES `high_school_group` (`group_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `high_school_dept_mapping_ibfk_3` FOREIGN KEY (`dept_sn`) REFERENCES `high_school_dept` (`dept_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `indicateTest`
--
ALTER TABLE `indicateTest`
  ADD CONSTRAINT `indicateTest_ibfk_1` FOREIGN KEY (`subject`) REFERENCES `subject` (`subject_id`);

--
-- 資料表的限制式 `indicate_test_predata`
--
ALTER TABLE `indicate_test_predata`
  ADD CONSTRAINT `indicate_test_predata_ibfk_1` FOREIGN KEY (`mission_sn`) REFERENCES `mission_info` (`mission_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `indicator_info`
--
ALTER TABLE `indicator_info`
  ADD CONSTRAINT `indicator_info_ibfk_1` FOREIGN KEY (`subject`) REFERENCES `subject` (`subject_id`);

--
-- 資料表的限制式 `indicator_priori`
--
ALTER TABLE `indicator_priori`
  ADD CONSTRAINT `indicator_priori_ibfk_1` FOREIGN KEY (`subject`) REFERENCES `subject` (`subject_id`);

--
-- 資料表的限制式 `ItemBank_fix_record`
--
ALTER TABLE `ItemBank_fix_record`
  ADD CONSTRAINT `ItemBank_fix_record_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL,
  ADD CONSTRAINT `ItemBank_fix_record_ibfk_2` FOREIGN KEY (`item_type`) REFERENCES `mission_type` (`mission_type_id`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `item_mission_rate`
--
ALTER TABLE `item_mission_rate`
  ADD CONSTRAINT `item_mission_rate_ibfk_1` FOREIGN KEY (`mission_sn`) REFERENCES `mission_info` (`mission_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `literacy_publisher_mapping`
--
ALTER TABLE `literacy_publisher_mapping`
  ADD CONSTRAINT `literacy_publisher_mapping_ibfk_1` FOREIGN KEY (`publisher_id`) REFERENCES `publisher` (`publisher_id`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `log_exam_bug`
--
ALTER TABLE `log_exam_bug`
  ADD CONSTRAINT `log_exam_bug_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE;

--
-- 資料表的限制式 `mad_exam_record`
--
ALTER TABLE `mad_exam_record`
  ADD CONSTRAINT `mad_exam_record_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `mad_exam_record_ibfk_2` FOREIGN KEY (`firm_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `map_arrows`
--
ALTER TABLE `map_arrows`
  ADD CONSTRAINT `map_arrows_ibfk_1` FOREIGN KEY (`map_sn`) REFERENCES `map_info` (`map_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `map_arrows_ibfk_2` FOREIGN KEY (`arrow_creater`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL;

--
-- 資料表的限制式 `map_info`
--
ALTER TABLE `map_info`
  ADD CONSTRAINT `map_info_ibfk_2` FOREIGN KEY (`create_user_id`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL,
  ADD CONSTRAINT `map_info_ibfk_3` FOREIGN KEY (`subject_id`) REFERENCES `subject` (`subject_id`);

--
-- 資料表的限制式 `map_node_bk2`
--
ALTER TABLE `map_node_bk2`
  ADD CONSTRAINT `map_node_bk2_ibfk_1` FOREIGN KEY (`map_sn`) REFERENCES `map_info` (`map_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `map_node_bk2_ibfk_3` FOREIGN KEY (`node_creater`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL,
  ADD CONSTRAINT `map_node_bk2_ibfk_4` FOREIGN KEY (`sub_subject_id`) REFERENCES `sub_subject` (`sub_subject_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `map_node_bk2_ibfk_5` FOREIGN KEY (`subject`) REFERENCES `subject` (`subject_id`);

--
-- 資料表的限制式 `map_node_external`
--
ALTER TABLE `map_node_external`
  ADD CONSTRAINT `map_node_external_ibfk_3` FOREIGN KEY (`creater`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL,
  ADD CONSTRAINT `map_node_external_ibfk_4` FOREIGN KEY (`subject_id`) REFERENCES `subject` (`subject_id`);

--
-- 資料表的限制式 `map_node_file`
--
ALTER TABLE `map_node_file`
  ADD CONSTRAINT `map_node_file_ibfk_3` FOREIGN KEY (`create_user`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL;

--
-- 資料表的限制式 `map_node_grade`
--
ALTER TABLE `map_node_grade`
  ADD CONSTRAINT `map_node_grade_ibfk_1` FOREIGN KEY (`subject_id`) REFERENCES `subject` (`subject_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `map_node_grade_ibfk_2` FOREIGN KEY (`map_sn`) REFERENCES `map_info` (`map_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `map_node_student_status`
--
ALTER TABLE `map_node_student_status`
  ADD CONSTRAINT `map_node_student_status_ibfk_1` FOREIGN KEY (`map_sn`) REFERENCES `map_info` (`map_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `map_node_student_status_ibfk_2` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `map_node_student_status_ibfk_3` FOREIGN KEY (`recommended_mission`) REFERENCES `mission_info` (`mission_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `math_laboratory_exam_record`
--
ALTER TABLE `math_laboratory_exam_record`
  ADD CONSTRAINT `math_laboratory_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE RESTRICT ON UPDATE CASCADE;

--
-- 資料表的限制式 `message_fileattached`
--
ALTER TABLE `message_fileattached`
  ADD CONSTRAINT `message_fileattached_ibfk_2` FOREIGN KEY (`upload_userid`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL,
  ADD CONSTRAINT `message_fileattached_ibfk_3` FOREIGN KEY (`message_sn`) REFERENCES `message_master` (`msg_sn`) ON DELETE CASCADE ON UPDATE CASCADE;

--
-- 資料表的限制式 `message_master`
--
ALTER TABLE `message_master`
  ADD CONSTRAINT `message_master_ibfk_1` FOREIGN KEY (`touser_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `message_master_ibfk_2` FOREIGN KEY (`create_user`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE;

--
-- 資料表的限制式 `message_response`
--
ALTER TABLE `message_response`
  ADD CONSTRAINT `message_response_ibfk_1` FOREIGN KEY (`message_sn`) REFERENCES `message_master` (`msg_sn`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `message_response_ibfk_2` FOREIGN KEY (`remsg_create_user`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `message_response_ibfk_3` FOREIGN KEY (`res_resmsgto_user`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE;

--
-- 資料表的限制式 `mission_google_form`
--
ALTER TABLE `mission_google_form`
  ADD CONSTRAINT `mission_google_form_ibfk_1` FOREIGN KEY (`form_type`) REFERENCES `mission_google_master` (`sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `mission_google_form_ibfk_2` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL,
  ADD CONSTRAINT `mission_google_form_ibfk_3` FOREIGN KEY (`mission_sn`) REFERENCES `mission_info` (`mission_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `mission_google_form_ibfk_4` FOREIGN KEY (`org_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `mission_group_leader`
--
ALTER TABLE `mission_group_leader`
  ADD CONSTRAINT `mission_group_leader_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `mission_group_leader_ibfk_2` FOREIGN KEY (`group_id`) REFERENCES `seme_group` (`group_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `mission_group_leader_ibfk_3` FOREIGN KEY (`mission_sn`) REFERENCES `mission_info` (`mission_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `mission_info`
--
ALTER TABLE `mission_info`
  ADD CONSTRAINT `mission_info_ibfk_2` FOREIGN KEY (`teacher_id`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL,
  ADD CONSTRAINT `mission_info_ibfk_4` FOREIGN KEY (`mission_type`) REFERENCES `mission_type` (`mission_type_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `mission_info_ibfk_5` FOREIGN KEY (`copy_record`) REFERENCES `mission_info` (`mission_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `mission_info_ibfk_6` FOREIGN KEY (`subject_id`) REFERENCES `subject` (`subject_id`),
  ADD CONSTRAINT `mission_info_ibfk_7` FOREIGN KEY (`mid`) REFERENCES `publisher_mapping` (`mapping_sn`) ON DELETE SET NULL ON UPDATE SET NULL;

--
-- 資料表的限制式 `mission_result`
--
ALTER TABLE `mission_result`
  ADD CONSTRAINT `mission_result_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `mission_result_ibfk_2` FOREIGN KEY (`mission_sn`) REFERENCES `mission_info` (`mission_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `mission_stud_record`
--
ALTER TABLE `mission_stud_record`
  ADD CONSTRAINT `mission_stud_record_ibfk_1` FOREIGN KEY (`mission_sn`) REFERENCES `mission_info` (`mission_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `mission_stud_record_ibfk_2` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE;

--
-- 資料表的限制式 `new_class`
--
ALTER TABLE `new_class`
  ADD CONSTRAINT `new_class_ibfk_1` FOREIGN KEY (`organization_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `new_class_ibfk_2` FOREIGN KEY (`teacher_id`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL;

--
-- 資料表的限制式 `openid_api_status_record`
--
ALTER TABLE `openid_api_status_record`
  ADD CONSTRAINT `openid_api_status_record_ibfk_1` FOREIGN KEY (`openid_user_login_record_sn`) REFERENCES `openid_user_login_record` (`sn`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `openid_class_info`
--
ALTER TABLE `openid_class_info`
  ADD CONSTRAINT `openid_class_info_ibfk_1` FOREIGN KEY (`openid_user_login_record_sn`) REFERENCES `openid_user_login_record` (`sn`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `openid_titles_info`
--
ALTER TABLE `openid_titles_info`
  ADD CONSTRAINT `openid_titles_info_ibfk_1` FOREIGN KEY (`openid_user_login_record_sn`) REFERENCES `openid_user_login_record` (`sn`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `openid_user_data_check_status`
--
ALTER TABLE `openid_user_data_check_status`
  ADD CONSTRAINT `openid_user_data_check_status_ibfk_1` FOREIGN KEY (`openid_user_login_record_sn`) REFERENCES `openid_user_login_record` (`sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `openid_user_data_check_status_ibfk_2` FOREIGN KEY (`openid_user_data_check_status_discription_sn`) REFERENCES `openid_user_data_check_status_discription` (`sn`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `openid_user_info`
--
ALTER TABLE `openid_user_info`
  ADD CONSTRAINT `openid_user_info_ibfk_1` FOREIGN KEY (`openid_user_login_record_sn`) REFERENCES `openid_user_login_record` (`sn`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `organization_ace_setting`
--
ALTER TABLE `organization_ace_setting`
  ADD CONSTRAINT `organization_ace_setting_ibfk_1` FOREIGN KEY (`organization_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `organization_ace_setting_ibfk_2` FOREIGN KEY (`access_level`) REFERENCES `user_access` (`access_level`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `organization_ace_setting_ibfk_3` FOREIGN KEY (`module_id`) REFERENCES `website_module` (`module_id`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `organization_Education_Administration`
--
ALTER TABLE `organization_Education_Administration`
  ADD CONSTRAINT `organization_Education_Administration_ibfk_1` FOREIGN KEY (`organization_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `organization_exception`
--
ALTER TABLE `organization_exception`
  ADD CONSTRAINT `organization_exception_ibfk_1` FOREIGN KEY (`organization_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `organization_project_participation`
--
ALTER TABLE `organization_project_participation`
  ADD CONSTRAINT `organization_project_participation_ibfk_1` FOREIGN KEY (`organization_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `organization_subject_group`
--
ALTER TABLE `organization_subject_group`
  ADD CONSTRAINT `organization_subject_group_ibfk_1` FOREIGN KEY (`organization_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `paper_teacher`
--
ALTER TABLE `paper_teacher`
  ADD CONSTRAINT `paper_teacher_ibfk_1` FOREIGN KEY (`map_sn`) REFERENCES `map_info` (`map_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `paper_teacher_ibfk_2` FOREIGN KEY (`teacher_user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE;

--
-- 資料表的限制式 `prac_answer`
--
ALTER TABLE `prac_answer`
  ADD CONSTRAINT `prac_answer_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `prac_answer_ibfk_2` FOREIGN KEY (`mission_sn`) REFERENCES `mission_info` (`mission_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `prac_blank_filling_record`
--
ALTER TABLE `prac_blank_filling_record`
  ADD CONSTRAINT `prac_blank_filling_record_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `prac_blank_filling_record_ibfk_2` FOREIGN KEY (`item_sn`) REFERENCES `prac_blank_filling_itembank` (`item_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `prac_blank_filling_record_ibfk_3` FOREIGN KEY (`mission_sn`) REFERENCES `mission_info` (`mission_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `prac_options`
--
ALTER TABLE `prac_options`
  ADD CONSTRAINT `prac_options_ibfk_1` FOREIGN KEY (`questions_id`) REFERENCES `prac_questions` (`id`) ON DELETE CASCADE ON UPDATE CASCADE;

--
-- 資料表的限制式 `prac_session_data`
--
ALTER TABLE `prac_session_data`
  ADD CONSTRAINT `prac_session_data_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE;

--
-- 資料表的限制式 `publisher_mapping`
--
ALTER TABLE `publisher_mapping`
  ADD CONSTRAINT `publisher_mapping_ibfk_1` FOREIGN KEY (`publisher_id`) REFERENCES `publisher` (`publisher_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `publisher_mapping_ibfk_3` FOREIGN KEY (`updater`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL,
  ADD CONSTRAINT `publisher_mapping_ibfk_4` FOREIGN KEY (`subject_id`) REFERENCES `subject` (`subject_id`);

--
-- 資料表的限制式 `questionaire_export_permission`
--
ALTER TABLE `questionaire_export_permission`
  ADD CONSTRAINT `questionaire_export_permission_ibfk_1` FOREIGN KEY (`access_level`) REFERENCES `user_access` (`access_level`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `questionaire_export_permission_ibfk_2` FOREIGN KEY (`form_type`) REFERENCES `mission_google_master` (`sn`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `remedial_class`
--
ALTER TABLE `remedial_class`
  ADD CONSTRAINT `remedial_class_ibfk_1` FOREIGN KEY (`mission_sn`) REFERENCES `mission_info` (`mission_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `remedial_class_ibfk_2` FOREIGN KEY (`new_class_sn`) REFERENCES `new_class` (`new_class_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `remedial_class_ibfk_4` FOREIGN KEY (`create_id`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL,
  ADD CONSTRAINT `remedial_class_ibfk_5` FOREIGN KEY (`subject_id`) REFERENCES `subject` (`subject_id`);

--
-- 資料表的限制式 `remedial_ftp_history`
--
ALTER TABLE `remedial_ftp_history`
  ADD CONSTRAINT `remedial_ftp_history_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL,
  ADD CONSTRAINT `remedial_ftp_history_ibfk_2` FOREIGN KEY (`subject_id`) REFERENCES `subject` (`subject_id`);

--
-- 資料表的限制式 `remedial_result_history`
--
ALTER TABLE `remedial_result_history`
  ADD CONSTRAINT `remedial_result_history_ibfk_1` FOREIGN KEY (`organization_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `remedial_student`
--
ALTER TABLE `remedial_student`
  ADD CONSTRAINT `remedial_student_ibfk_1` FOREIGN KEY (`class_sn`) REFERENCES `remedial_class` (`class_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `remedial_student_ibfk_2` FOREIGN KEY (`student_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `remedial_student_ibfk_3` FOREIGN KEY (`new_class_agree`) REFERENCES `new_class` (`new_class_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `remedial_teacher`
--
ALTER TABLE `remedial_teacher`
  ADD CONSTRAINT `remedial_teacher_ibfk_1` FOREIGN KEY (`class_sn`) REFERENCES `remedial_class` (`class_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `remedial_teacher_ibfk_2` FOREIGN KEY (`teacher_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE;

--
-- 資料表的限制式 `report_dailyusage`
--
ALTER TABLE `report_dailyusage`
  ADD CONSTRAINT `report_dailyusage_ibfk_1` FOREIGN KEY (`organization_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `report_grad_student_node`
--
ALTER TABLE `report_grad_student_node`
  ADD CONSTRAINT `report_grad_student_node_ibfk_1` FOREIGN KEY (`organization_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `report_grad_student_node_ibfk_2` FOREIGN KEY (`map_sn`) REFERENCES `map_info` (`map_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `report_grad_student_practice`
--
ALTER TABLE `report_grad_student_practice`
  ADD CONSTRAINT `report_grad_student_practice_ibfk_1` FOREIGN KEY (`organization_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `report_grad_student_practice_ibfk_2` FOREIGN KEY (`map_sn`) REFERENCES `map_info` (`map_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `report_grad_student_unit`
--
ALTER TABLE `report_grad_student_unit`
  ADD CONSTRAINT `report_grad_student_unit_ibfk_1` FOREIGN KEY (`organization_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `report_grad_student_unit_ibfk_2` FOREIGN KEY (`map_sn`) REFERENCES `map_info` (`map_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `report_grad_student_video`
--
ALTER TABLE `report_grad_student_video`
  ADD CONSTRAINT `report_grad_student_video_ibfk_1` FOREIGN KEY (`organization_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `report_grad_student_video_ibfk_2` FOREIGN KEY (`map_sn`) REFERENCES `map_info` (`map_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `report_learningeffect_dynamic`
--
ALTER TABLE `report_learningeffect_dynamic`
  ADD CONSTRAINT `report_learningeffect_dynamic_ibfk_1` FOREIGN KEY (`organization_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `report_learningeffect_dynamic_ibfk_2` FOREIGN KEY (`map_sn`) REFERENCES `map_info` (`map_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `report_learningeffect_indicate`
--
ALTER TABLE `report_learningeffect_indicate`
  ADD CONSTRAINT `report_learningeffect_indicate_ibfk_1` FOREIGN KEY (`organization_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `report_learningeffect_indicate_ibfk_2` FOREIGN KEY (`map_sn`) REFERENCES `map_info` (`map_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `report_learningeffect_node`
--
ALTER TABLE `report_learningeffect_node`
  ADD CONSTRAINT `report_learningeffect_node_ibfk_1` FOREIGN KEY (`organization_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `report_learningeffect_node_ibfk_2` FOREIGN KEY (`map_sn`) REFERENCES `map_info` (`map_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `report_learningeffect_practice`
--
ALTER TABLE `report_learningeffect_practice`
  ADD CONSTRAINT `report_learningeffect_practice_ibfk_1` FOREIGN KEY (`organization_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `report_learningeffect_practice_ibfk_2` FOREIGN KEY (`map_sn`) REFERENCES `map_info` (`map_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `report_learningeffect_unit`
--
ALTER TABLE `report_learningeffect_unit`
  ADD CONSTRAINT `report_learningeffect_unit_ibfk_1` FOREIGN KEY (`organization_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `report_learningeffect_unit_ibfk_2` FOREIGN KEY (`map_sn`) REFERENCES `map_info` (`map_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `report_learningeffect_video`
--
ALTER TABLE `report_learningeffect_video`
  ADD CONSTRAINT `report_learningeffect_video_ibfk_1` FOREIGN KEY (`organization_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `report_learningeffect_video_ibfk_2` FOREIGN KEY (`map_sn`) REFERENCES `map_info` (`map_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `report_workshop_effect`
--
ALTER TABLE `report_workshop_effect`
  ADD CONSTRAINT `report_workshop_effect_ibfk_1` FOREIGN KEY (`organization_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `role_coin_history`
--
ALTER TABLE `role_coin_history`
  ADD CONSTRAINT `role_coin_history_ibfk_1` FOREIGN KEY (`operator`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `role_coin_history_ibfk_2` FOREIGN KEY (`student_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `role_coin_history_ibfk_3` FOREIGN KEY (`mission_sn`) REFERENCES `mission_info` (`mission_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `role_coin_history_ibfk_4` FOREIGN KEY (`video_note_sn`) REFERENCES `video_note` (`video_note_sn`) ON DELETE SET NULL ON UPDATE SET NULL;

--
-- 資料表的限制式 `role_coin_rule`
--
ALTER TABLE `role_coin_rule`
  ADD CONSTRAINT `role_coin_rule_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE;

--
-- 資料表的限制式 `role_coin_total`
--
ALTER TABLE `role_coin_total`
  ADD CONSTRAINT `role_coin_total_ibfk_1` FOREIGN KEY (`student_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `role_coin_total_ibfk_2` FOREIGN KEY (`teacher_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE;

--
-- 資料表的限制式 `role_info`
--
ALTER TABLE `role_info`
  ADD CONSTRAINT `role_info_ibfk_1` FOREIGN KEY (`organization_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `role_info_ibfk_2` FOREIGN KEY (`stud_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `role_info_ibfk_3` FOREIGN KEY (`carrer_id`) REFERENCES `role_carrer_info` (`carrer_id`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `school_class_info`
--
ALTER TABLE `school_class_info`
  ADD CONSTRAINT `school_class_info_ibfk_4` FOREIGN KEY (`organization_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `school_class_info_ibfk_5` FOREIGN KEY (`high_school_dept_mapping_sn`) REFERENCES `high_school_dept_mapping` (`dept_mapping_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `school_class_info_ibfk_6` FOREIGN KEY (`updater`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL;

--
-- 資料表的限制式 `seme_grad_student`
--
ALTER TABLE `seme_grad_student`
  ADD CONSTRAINT `seme_grad_student_ibfk_1` FOREIGN KEY (`organization_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `seme_grad_student_ibfk_2` FOREIGN KEY (`stud_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `seme_grad_student_ibfk_3` FOREIGN KEY (`next_organization_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `seme_group`
--
ALTER TABLE `seme_group`
  ADD CONSTRAINT `seme_group_ibfk_1` FOREIGN KEY (`teacher_id`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL;

--
-- 資料表的限制式 `seme_publisher`
--
ALTER TABLE `seme_publisher`
  ADD CONSTRAINT `seme_publisher_ibfk_1` FOREIGN KEY (`organization_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `seme_publisher_ibfk_3` FOREIGN KEY (`publisher_id`) REFERENCES `publisher` (`publisher_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `seme_publisher_ibfk_4` FOREIGN KEY (`create_id`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL,
  ADD CONSTRAINT `seme_publisher_ibfk_5` FOREIGN KEY (`subject_id`) REFERENCES `subject` (`subject_id`);

--
-- 資料表的限制式 `seme_publisher_tmp`
--
ALTER TABLE `seme_publisher_tmp`
  ADD CONSTRAINT `seme_publisher_tmp_ibfk_1` FOREIGN KEY (`organization_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `seme_publisher_tmp_ibfk_3` FOREIGN KEY (`publisher_id`) REFERENCES `publisher` (`publisher_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `seme_publisher_tmp_ibfk_4` FOREIGN KEY (`subject_id`) REFERENCES `subject` (`subject_id`);

--
-- 資料表的限制式 `seme_student`
--
ALTER TABLE `seme_student`
  ADD CONSTRAINT `seme_student_ibfk_1` FOREIGN KEY (`organization_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `seme_student_ibfk_2` FOREIGN KEY (`stud_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE;

--
-- 資料表的限制式 `seme_student_rebackschool`
--
ALTER TABLE `seme_student_rebackschool`
  ADD CONSTRAINT `seme_student_rebackschool_ibfk_1` FOREIGN KEY (`organization_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `seme_student_rebackschool_ibfk_2` FOREIGN KEY (`next_organization_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `seme_student_rebackschool_ibfk_3` FOREIGN KEY (`stud_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `seme_student_rebackschool_ibfk_4` FOREIGN KEY (`update_id`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL;

--
-- 資料表的限制式 `seme_student_tmp`
--
ALTER TABLE `seme_student_tmp`
  ADD CONSTRAINT `seme_student_tmp_ibfk_1` FOREIGN KEY (`organization_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `seme_student_tmp_ibfk_2` FOREIGN KEY (`stud_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE;

--
-- 資料表的限制式 `seme_teacher_subject`
--
ALTER TABLE `seme_teacher_subject`
  ADD CONSTRAINT `seme_teacher_subject_ibfk_1` FOREIGN KEY (`organization_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `seme_teacher_subject_ibfk_2` FOREIGN KEY (`teacher_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `seme_teacher_subject_ibfk_4` FOREIGN KEY (`subject_mapping`) REFERENCES `seme_teacher_subject_mapping` (`mapping_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `seme_teacher_subject_ibfk_5` FOREIGN KEY (`subject_id`) REFERENCES `subject` (`subject_id`);

--
-- 資料表的限制式 `seme_teacher_subject_mapping`
--
ALTER TABLE `seme_teacher_subject_mapping`
  ADD CONSTRAINT `seme_teacher_subject_mapping_ibfk_1` FOREIGN KEY (`organization_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `seme_teacher_subject_tmp`
--
ALTER TABLE `seme_teacher_subject_tmp`
  ADD CONSTRAINT `seme_teacher_subject_tmp_ibfk_1` FOREIGN KEY (`organization_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `seme_teacher_subject_tmp_ibfk_2` FOREIGN KEY (`teacher_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `seme_teacher_subject_tmp_ibfk_3` FOREIGN KEY (`subject_id`) REFERENCES `subject` (`subject_id`);

--
-- 資料表的限制式 `srl_calendar`
--
ALTER TABLE `srl_calendar`
  ADD CONSTRAINT `srl_calendar_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `srl_calendar_ibfk_2` FOREIGN KEY (`mission_sn`) REFERENCES `mission_info` (`mission_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `subject`
--
ALTER TABLE `subject`
  ADD CONSTRAINT `subject_ibfk_1` FOREIGN KEY (`subject_groupings_id`) REFERENCES `subject_groupings` (`subject_groupings_id`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `subject_groupings`
--
ALTER TABLE `subject_groupings`
  ADD CONSTRAINT `subject_groupings_ibfk_1` FOREIGN KEY (`subject_id`) REFERENCES `subject` (`subject_id`);

--
-- 資料表的限制式 `sub_subject`
--
ALTER TABLE `sub_subject`
  ADD CONSTRAINT `sub_subject_ibfk_1` FOREIGN KEY (`map_sn`) REFERENCES `map_info` (`map_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `sub_subject_ibfk_2` FOREIGN KEY (`subject_id`) REFERENCES `subject` (`subject_id`);

--
-- 資料表的限制式 `system_user`
--
ALTER TABLE `system_user`
  ADD CONSTRAINT `system_user_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `teacher_class`
--
ALTER TABLE `teacher_class`
  ADD CONSTRAINT `teacher_class_ibfk_1` FOREIGN KEY (`organization_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `teacher_class_ibfk_2` FOREIGN KEY (`teacher_id`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL,
  ADD CONSTRAINT `teacher_class_ibfk_3` FOREIGN KEY (`create_id`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL;

--
-- 資料表的限制式 `teacher_class_member`
--
ALTER TABLE `teacher_class_member`
  ADD CONSTRAINT `teacher_class_member_ibfk_1` FOREIGN KEY (`class_sn`) REFERENCES `teacher_class` (`teacher_class_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `teacher_class_member_ibfk_2` FOREIGN KEY (`student_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE;

--
-- 資料表的限制式 `triangle_conclusion`
--
ALTER TABLE `triangle_conclusion`
  ADD CONSTRAINT `triangle_conclusion_ibfk_1` FOREIGN KEY (`question_sn`) REFERENCES `high_level_question` (`question_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `triangle_conclusion_proofs`
--
ALTER TABLE `triangle_conclusion_proofs`
  ADD CONSTRAINT `triangle_conclusion_proofs_ibfk_1` FOREIGN KEY (`conclusion_id`) REFERENCES `triangle_conclusion` (`conclusion_id`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `user_account_history`
--
ALTER TABLE `user_account_history`
  ADD CONSTRAINT `user_account_history_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `user_account_history_ibfk_2` FOREIGN KEY (`ori_access_level`) REFERENCES `user_access` (`access_level`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `user_account_history_ibfk_3` FOREIGN KEY (`ori_organization_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `user_activity`
--
ALTER TABLE `user_activity`
  ADD CONSTRAINT `user_activity_ibfk_1` FOREIGN KEY (`organization_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `user_combine_history`
--
ALTER TABLE `user_combine_history`
  ADD CONSTRAINT `user_combine_history_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `user_combine_history_ibfk_2` FOREIGN KEY (`connect_user_id`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL,
  ADD CONSTRAINT `user_combine_history_ibfk_3` FOREIGN KEY (`by_user_id`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL;

--
-- 資料表的限制式 `user_entering_interactive_learning_log`
--
ALTER TABLE `user_entering_interactive_learning_log`
  ADD CONSTRAINT `user_entering_interactive_learning_log_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `user_entering_interactive_learning_log_ibfk_2` FOREIGN KEY (`organization_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `user_family`
--
ALTER TABLE `user_family`
  ADD CONSTRAINT `user_family_ibfk_1` FOREIGN KEY (`organization_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `user_family_ibfk_2` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `user_family_ibfk_3` FOREIGN KEY (`fuser_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `user_family_ibfk_4` FOREIGN KEY (`create_userid`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL;

--
-- 資料表的限制式 `user_feedback`
--
ALTER TABLE `user_feedback`
  ADD CONSTRAINT `user_feedback_ibfk_1` FOREIGN KEY (`create_user`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL,
  ADD CONSTRAINT `user_feedback_ibfk_2` FOREIGN KEY (`chk3`) REFERENCES `subject` (`subject_id`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `user_games_play_time`
--
ALTER TABLE `user_games_play_time`
  ADD CONSTRAINT `user_games_play_time_ibfk_1` FOREIGN KEY (`api_client_sn`) REFERENCES `api_client_config` (`api_client_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `user_games_play_time_ibfk_2` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE;

--
-- 資料表的限制式 `user_group`
--
ALTER TABLE `user_group`
  ADD CONSTRAINT `user_group_ibfk_1` FOREIGN KEY (`organization_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `user_group_ibfk_2` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `user_group_ibfk_3` FOREIGN KEY (`group_id`) REFERENCES `seme_group` (`group_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `user_group_ibfk_4` FOREIGN KEY (`role_sn`) REFERENCES `group_role` (`group_role_sn`) ON DELETE SET NULL ON UPDATE SET NULL;

--
-- 資料表的限制式 `user_info`
--
ALTER TABLE `user_info`
  ADD CONSTRAINT `user_info_ibfk_1` FOREIGN KEY (`class_sn`) REFERENCES `school_class_info` (`class_sn`) ON DELETE SET NULL ON UPDATE SET NULL,
  ADD CONSTRAINT `user_info_ibfk_2` FOREIGN KEY (`organization_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `user_info_ibfk_3` FOREIGN KEY (`update_id`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL;

--
-- 資料表的限制式 `user_login_log`
--
ALTER TABLE `user_login_log`
  ADD CONSTRAINT `__user_login_log_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `__user_login_log_ibfk_2` FOREIGN KEY (`organization_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `user_map_permission`
--
ALTER TABLE `user_map_permission`
  ADD CONSTRAINT `user_map_permission_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `user_map_permission_ibfk_2` FOREIGN KEY (`map_sn`) REFERENCES `map_info` (`map_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `user_map_permission_ibfk_3` FOREIGN KEY (`updater`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL;

--
-- 資料表的限制式 `user_past_password`
--
ALTER TABLE `user_past_password`
  ADD CONSTRAINT `user_past_password_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `user_past_password_ibfk_2` FOREIGN KEY (`modifier`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL;

--
-- 資料表的限制式 `user_person_config`
--
ALTER TABLE `user_person_config`
  ADD CONSTRAINT `user_person_config_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE;

--
-- 資料表的限制式 `user_play_game_log`
--
ALTER TABLE `user_play_game_log`
  ADD CONSTRAINT `user_play_game_log_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE RESTRICT ON UPDATE CASCADE,
  ADD CONSTRAINT `user_play_game_log_ibfk_2` FOREIGN KEY (`organization_id`) REFERENCES `organization` (`organization_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `user_play_game_log_ibfk_3` FOREIGN KEY (`api_client_sn`) REFERENCES `api_client_config` (`api_client_sn`) ON DELETE RESTRICT ON UPDATE CASCADE;

--
-- 資料表的限制式 `user_recovery_record`
--
ALTER TABLE `user_recovery_record`
  ADD CONSTRAINT `user_recovery_record_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE;

--
-- 資料表的限制式 `user_status`
--
ALTER TABLE `user_status`
  ADD CONSTRAINT `CON_userid` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `user_status_ibfk_1` FOREIGN KEY (`access_level`) REFERENCES `user_access` (`access_level`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `user_third_party`
--
ALTER TABLE `user_third_party`
  ADD CONSTRAINT `user_third_party_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE;

--
-- 資料表的限制式 `video_concept_item`
--
ALTER TABLE `video_concept_item`
  ADD CONSTRAINT `video_concept_item_ibfk_2` FOREIGN KEY (`create_user_id`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL;

--
-- 資料表的限制式 `video_concept_item_plus`
--
ALTER TABLE `video_concept_item_plus`
  ADD CONSTRAINT `CON_video_item` FOREIGN KEY (`video_item_sn`) REFERENCES `video_concept_item` (`video_item_sn`),
  ADD CONSTRAINT `video_concept_item_plus_ibfk_1` FOREIGN KEY (`create_user_id`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL;

--
-- 資料表的限制式 `video_concept_stream_error`
--
ALTER TABLE `video_concept_stream_error`
  ADD CONSTRAINT `video_concept_stream_error_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE;

--
-- 資料表的限制式 `video_exam_record`
--
ALTER TABLE `video_exam_record`
  ADD CONSTRAINT `CON_video_exam` FOREIGN KEY (`question_sn`) REFERENCES `video_concept_item_plus` (`question_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `video_exam_record_ibfk_1` FOREIGN KEY (`review_sn`) REFERENCES `video_review_record` (`review_sn`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `video_exam_record_ibfk_2` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `video_exam_record_ibfk_3` FOREIGN KEY (`video_item_sn`) REFERENCES `video_concept_item` (`video_item_sn`) ON DELETE CASCADE ON UPDATE CASCADE;

--
-- 資料表的限制式 `video_note`
--
ALTER TABLE `video_note`
  ADD CONSTRAINT `video_note_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `video_note_ibfk_2` FOREIGN KEY (`video_item_sn`) REFERENCES `video_concept_item` (`video_item_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `video_noteask`
--
ALTER TABLE `video_noteask`
  ADD CONSTRAINT `video_noteask_ibfk_1` FOREIGN KEY (`video_item_sn`) REFERENCES `video_concept_item` (`video_item_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `video_noteask_ibfk_2` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `video_noteask_ibfk_3` FOREIGN KEY (`group_id`) REFERENCES `seme_group` (`group_id`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `video_noteask_plus`
--
ALTER TABLE `video_noteask_plus`
  ADD CONSTRAINT `CON_video_noteask` FOREIGN KEY (`ask_sn`) REFERENCES `video_noteask` (`ask_sn`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `video_noteask_plus_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `video_noteask_plus_ibfk_2` FOREIGN KEY (`deleter`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL;

--
-- 資料表的限制式 `video_note_favorite`
--
ALTER TABLE `video_note_favorite`
  ADD CONSTRAINT `video_note_favorite_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `video_note_favorite_ibfk_2` FOREIGN KEY (`video_note_sn`) REFERENCES `video_note` (`video_note_sn`) ON DELETE CASCADE ON UPDATE CASCADE;

--
-- 資料表的限制式 `video_note_feedback`
--
ALTER TABLE `video_note_feedback`
  ADD CONSTRAINT `video_note_feedback_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `video_note_feedback_ibfk_2` FOREIGN KEY (`video_note_sn`) REFERENCES `video_note` (`video_note_sn`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `video_note_feedback_ibfk_3` FOREIGN KEY (`deleter`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL;

--
-- 資料表的限制式 `video_note_recommend`
--
ALTER TABLE `video_note_recommend`
  ADD CONSTRAINT `video_note_recommend_ibfk_1` FOREIGN KEY (`video_note_sn`) REFERENCES `video_note` (`video_note_sn`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `video_note_recommend_ibfk_2` FOREIGN KEY (`referrer_id`) REFERENCES `user_info` (`user_id`) ON DELETE SET NULL ON UPDATE SET NULL;

--
-- 資料表的限制式 `video_review_record`
--
ALTER TABLE `video_review_record`
  ADD CONSTRAINT `video_review_record_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `video_review_record_ibfk_2` FOREIGN KEY (`video_item_sn`) REFERENCES `video_concept_item` (`video_item_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  ADD CONSTRAINT `video_review_record_ibfk_3` FOREIGN KEY (`mission_sn`) REFERENCES `mission_info` (`mission_sn`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `video_review_record_plus`
--
ALTER TABLE `video_review_record_plus`
  ADD CONSTRAINT `video_review_record_plus_ibfk_1` FOREIGN KEY (`review_sn`) REFERENCES `video_review_record` (`review_sn`) ON DELETE CASCADE ON UPDATE CASCADE;

--
-- 資料表的限制式 `z_system_identity_log`
--
ALTER TABLE `z_system_identity_log`
  ADD CONSTRAINT `z_system_identity_log_ibfk_1` FOREIGN KEY (`user_loginto`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `z_system_identity_log_ibfk_2` FOREIGN KEY (`user_loginfrom`) REFERENCES `user_info` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE;

--
-- 資料表的限制式 `z_system_team_feedback_status`
--
ALTER TABLE `z_system_team_feedback_status`
  ADD CONSTRAINT `z_system_team_feedback_status_ibfk_1` FOREIGN KEY (`q_id`) REFERENCES `user_feedback` (`sn`) ON DELETE RESTRICT ON UPDATE RESTRICT;

--
-- 資料表的限制式 `z_system_team_users`
--
ALTER TABLE `z_system_team_users`
  ADD CONSTRAINT `z_system_team_users_ibfk_1` FOREIGN KEY (`user_account`) REFERENCES `user_info` (`user_account`) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT `z_system_team_users_ibfk_2` FOREIGN KEY (`subject_id`) REFERENCES `subject` (`subject_id`) ON DELETE RESTRICT ON UPDATE RESTRICT;
COMMIT;

/*!40101 SET CHARACTER_SET_CLIENT=@OLD_CHARACTER_SET_CLIENT */;
/*!40101 SET CHARACTER_SET_RESULTS=@OLD_CHARACTER_SET_RESULTS */;
/*!40101 SET COLLATION_CONNECTION=@OLD_COLLATION_CONNECTION */;
