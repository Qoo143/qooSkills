---
name: adl_database
description: >
  資料庫結構參考。當詢問資料表、欄位、schema、
  CREATE TABLE、資料模型、FK、關聯 相關問題時觸發。
---

# 資料庫結構

## 使用方式

每個資料表的原始 CREATE TABLE 語句存放於 `tables/` 目錄。

## 資料表清單

| 資料表 | 說明 | 檔案 |
|:-------|:-----|:-----|
| api_client_config | API 客戶端設定 | [tables/api_client_config.md](./tables/api_client_config.md) |
| api_token_data | API Token 資料 | [tables/api_token_data.md](./tables/api_token_data.md) |
| eduexam_paper | 大考考古題試題卷 | [tables/eduexam_paper.md](./tables/eduexam_paper.md) |
| eduexam_record | 大考考古題測驗紀錄 | [tables/eduexam_record.md](./tables/eduexam_record.md) |
| exam_record_indicate | 縱貫測驗結果 | [tables/exam_record_indicate.md](./tables/exam_record_indicate.md) |
| exam_record_indicator | 單元測驗結果 | [tables/exam_record_indicator.md](./tables/exam_record_indicator.md) |
| exam_record_literacy_interactive | 互動式素養題測驗結果 | [tables/exam_record_literacy_interactive.md](./tables/exam_record_literacy_interactive.md) |
| export_school | 轉出入登記 | [tables/export_school.md](./tables/export_school.md) |
| mission_google_form | Google 表單問卷回應 | [tables/mission_google_form.md](./tables/mission_google_form.md) |
| mission_info | 任務資訊 | [tables/mission_info.md](./tables/mission_info.md) |
| mission_stud_record | 學生任務狀態 | [tables/mission_stud_record.md](./tables/mission_stud_record.md) |
| mission_type | 任務類型 | [tables/mission_type.md](./tables/mission_type.md) |
| remedial_student | 補救教學學生 | [tables/remedial_student.md](./tables/remedial_student.md) |
| seme_student | 學生學期資料 | [tables/seme_student.md](./tables/seme_student.md) |
| srl_calendar | 自主學習行事曆 | [tables/srl_calendar.md](./tables/srl_calendar.md) |
| teacher_class_member | 教師班級成員 | [tables/teacher_class_member.md](./tables/teacher_class_member.md) |
| user_info | 使用者基本資料 | [tables/user_info.md](./tables/user_info.md) |
| user_status | 使用者狀態 | [tables/user_status.md](./tables/user_status.md) |

## 檔案結構

每個 `tables/*.md` 檔案格式如下：

```markdown
# {table_name}

{一行說明}

## CREATE TABLE

```sql
CREATE TABLE ...
```
```

從 CREATE TABLE 可解析：欄位定義、PRIMARY KEY、INDEX、FOREIGN KEY。

