# OpenID 相關資料表

## 一般資料表

OpenID 登入流程會查詢和寫入以下資料表:

### organization

- 用途: 查詢學校是否為 OpenID 信任學校
- 查詢時機: 每次登入時檢查 `isOpenIDTrust($sOrgCode)`
- 關鍵欄位: `organization_id`, `openid_trust`
- 操作: 只查詢

### user_info

- 用途: 儲存所有使用者的基本資料
- 查詢時機: 四層帳號查詢、帳號創建
- 寫入時機: 創建帳號、補綁 OpenID_sub 或 hash_guid、資料同步、更新 openid_data
- 關鍵欄位:
  - `user_id` - 帳號 ID (如 093501-user12345)
  - `OpenID_sub` - OpenID 唯一識別碼
  - `hash_guid` - SHA256 雜湊後的身分證字號
  - `name` - 使用者姓名
  - `grade` - 年級
  - `class` - 班級
  - `openid_data` - OpenID 年班 JSON 快照 (用於快速登入比對)
  - `used` - 帳號狀態 (1=啟用, 0=停用, 2=刪除)
  - `organization_id` - 學校代碼

openid_data 寫入時機:
- 在一般登入 (LOGIN_SELECTED_IDENTITY) 的年班比對前寫入
- `prodb_choiceIdentity.php:936-949`

grade/class 欄位說明:
- `user_info.grade` / `user_info.class`: 儲存學生當前年班,用於快速查詢
- `seme_student.grade` / `seme_student.class`: 儲存學生在特定學期的年班
- 同步時會同時更新兩個表的年班欄位

### user_status

- 用途: 使用者權限等級和到期日
- 查詢時機: 查詢 `access_level` 確認使用者身分
- 寫入時機: 創建帳號
- 關鍵欄位: `user_id`, `access_level`, `expiration_date`

### export_school

- 用途: 記錄已經轉學的學生
- 查詢時機: 第三層查詢 (排除已轉學帳號)
- 關鍵欄位: `user_id`, `organization_id`

### seme_grad_student

- 用途: 記錄已經畢業的學生
- 查詢時機: 第四層查詢 (排除畢業生)
- 關鍵欄位: `user_id`, `seme_year_seme`

### seme_student

- 用途: 學生學期資料 (年班座號)
- 查詢時機: 學生帳號查詢 (年班+姓名比對)
- 寫入時機: 創建學生帳號、學生資料同步
- 關鍵欄位: `user_id`, `seme_year_seme`, `organization_id`, `grade`, `class`, `seatno`

### seme_teacher_subject

- 用途: 教師任課資料 (班級 x 科目)
- 查詢時機: 教師帳號查詢
- 寫入時機: 教師資料同步 (`replaceSemeTeacherSubject`)
- 操作: REPLACE INTO (覆蓋式寫入)
- 關鍵欄位: `teacher_id`, `seme_year_seme`, `organization_id`, `grade`, `class`, `subject_mapping_sn`

### seme_teacher_subject_mapping

- 用途: 儲存科目名稱和序號的對應關係
- 查詢時機: 教師同步時查科目代碼
- 關鍵欄位: `sn`, `subject_name`, `organization_id`

### information_schema.tables

- 用途: 查詢 `user_info` 的 `AUTO_INCREMENT` 目前值,用來產生新的 uid 序號
- 查詢時機: 創建帳號前取得下一個 uid
- 相關函式: `getNextUserInfoAI()` (`function_addUser.php:25`)

---

## Log 資料表

OpenID 登入過程會記錄詳細的 Log 資料供除錯和分析:

### openid_user_login_record

- 用途: 登入主記錄 (主鍵表)
- 記錄時機: 每次登入 (時機 1 和時機 2)
- 關鍵欄位:
  - `sn` (PK, AUTO_INCREMENT)
  - `sub` (OpenID_sub)
  - `access_token`
  - `error_status` (1=正確, 2=有錯誤)
  - `login_time`

### openid_user_info

- 用途: 使用者資料快照
- 記錄時機: 每次登入 1 筆
- 外鍵: `openid_user_login_record_sn`
- 關鍵欄位: `name`, `guid`, `guid_token`, `prefferd_name`, `school_id`, `mentor_grade`, `mentor_class`, `email`

### openid_class_info

- 用途: 班級資料快照
- 記錄時機: 每次登入多筆 (教師有多個任課班級)
- 外鍵: `openid_user_login_record_sn`
- 關鍵欄位: `year`, `semester`, `grade`, `class`, `organization_id`

### openid_titles_info

- 用途: 身分資料快照
- 記錄時機: 每次登入多筆 (多重身分或多校身分)
- 外鍵: `openid_user_login_record_sn`
- 關鍵欄位: `organization_id`, `title`

### openid_user_login_title

- 用途: 使用者選擇的登入身分
- 記錄時機: 僅時機 2 記錄 1 筆
- 外鍵: `openid_user_login_record_sn`
- 關鍵欄位: `school_id`, `titles`

### openid_user_data_check_status

- 用途: 資料驗證錯誤記錄
- 記錄時機: OpenID 資料驗證時,有錯誤時多筆
- 外鍵: `openid_user_login_record_sn`
- 關鍵欄位: `check_status_sn`

### openid_user_data_check_status_discription

- 用途: 錯誤代碼說明表 (靜態參照表)
- 定義: 60+ 種驗證錯誤類型
- 操作: 只查詢不新增
- 關鍵欄位: `sn`, `description`

### openid_api_status_record

- 用途: API 請求狀態 (HTTP 狀態碼和回應時間)
- 記錄時機: 每次登入 5 筆 (5 個 API)
- 外鍵: `openid_user_login_record_sn`
- 關鍵欄位: `api_type`, `http_status`, `response_time_ms`

---

## 資料表關聯圖

```
openid_user_login_record (主表, sn=PK)
  |-- openid_user_info (1:1)
  |-- openid_class_info (1:N)
  |-- openid_titles_info (1:N)
  |-- openid_user_login_title (1:0..1, 僅時機 2)
  |-- openid_user_data_check_status (1:N)
  +-- openid_api_status_record (1:5)

openid_user_data_check_status_discription (獨立參照表)

user_info (核心使用者表)
  |-- user_status (1:1)
  |-- seme_student (1:N, 學生)
  |-- seme_teacher_subject (1:N, 教師)
  |-- export_school (1:N, 轉出)
  +-- seme_grad_student (1:N, 畢業)
```

## 相關文件

- [Log 系統](./log-system.md) - Log 記錄機制
- [帳號查詢](./account-query.md) - 查詢使用的資料表
- [帳號創建](./account-create.md) - 寫入使用的資料表
- [資料同步](./data-sync.md) - 同步更新的資料表
