# 任務觀看權限擴展 — 需求規格書 v2

## 一、現況分析

### 現有機制
- 科任老師指派任務時，可勾選「開放此任務權限給導師」checkbox
- 後端將 `mission_info.subject_mission_share` 設為 `0` 或 `1`（tinyint）
- Dashboard 的「科任指派」篩選器透過 `seme_teacher_subject` 查找 `subject_name = '導師班'` 的教師，再 JOIN `mission_info.subject_mission_share = 1` 來顯示
- **限制**：只能分享給「該班的班導師」，且只是一個開關，無法指定分享給誰

### 導師 vs 科任的識別方式

| 身分 | 識別方式 | 資料表 |
|------|----------|--------|
| 級任導師（班導師） | `user_info.grade > 0 AND user_info.class > 0` | `user_info` |
| 科任教師 | 在 `seme_teacher_subject` 有該班的科目記錄 | `seme_teacher_subject` + `seme_teacher_subject_mapping` |
| 主任 | `access_level = 31` | `user_status` |
| 校長 | `access_level = 32` | `user_status` |

> 導師與科任的 `access_level` 都是 21 或 25，差別在 `user_info.grade/class` 是否有值。
> 科任老師的任教班級定義在 `seme_teacher_subject` 表。
> 參考：`modules/schoolReport/setTeachClass.php` 的查詢邏輯。

### 涉及檔案
| 檔案 | 角色 |
|------|------|
| `modules/assignMission/assign_mission.php` | 前端 UI（指派任務表單） |
| `modules/assignMission/assign_mission_prodb.php` | 後端處理（寫入 DB） |
| `modules/dashboard/modules_teacher_get_mission.php` | Dashboard 查詢任務邏輯 |
| `include/config.php` | 角色常數定義 |
| `modules/schoolReport/setTeachClass.php` | 科任老師管理（參考邏輯） |

### 角色代碼（config.php）
| 常數 | 代碼 | 說明 |
|------|------|------|
| `USER_TEACHER` | 21 | 教師（含導師與科任） |
| `USER_LECTURER` | 25 | 講師（含導師與科任） |
| `USER_SCHOOL_DIRECTOR` | 31 | 主任 |
| `USER_SCHOOL_PRINCIPAL` | 32 | 校長 |

---

## 二、需求目標

### 核心需求
> 科任老師指派任務時，能將任務觀看權限開放給**同校**的特定教職人員，被授權的人能在 Dashboard 上看到該任務。

### 授權對象範圍（已確認）
限制為**同校**，不跨校：

1. **該班級的級任導師** — 透過 `user_info.grade/class` 匹配目標班級
2. **該班級的科任教師** — 透過 `seme_teacher_subject` 查詢教該班的其他老師
3. **該校主任** — `access_level = 31`，同 `organization_id`
4. **該校校長** — `access_level = 32`，同 `organization_id`

### 範圍限定（已確認）
- 不跨校
- 目前無批次指派需求
- 需支援事後編輯（增減授權對象）
- Dashboard 呈現方式暫不異動
- 共享任務流程（assignshare）暫不調整

---

## 三、方案設計

### 3.1 資料庫變更

**新增關聯表 `mission_share_staff`**

```sql
CREATE TABLE mission_share_staff (
    sn            INT AUTO_INCREMENT PRIMARY KEY,
    mission_sn    INT NOT NULL,          -- 對應 mission_info.mission_sn
    staff_id      VARCHAR(60) NOT NULL,  -- 被授權觀看的教職人員 user_id
    created_at    DATETIME DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_mission_sn (mission_sn),
    INDEX idx_staff_id (staff_id),
    UNIQUE KEY uk_mission_staff (mission_sn, staff_id)
);
```

**`subject_mission_share` 欄位保留作為旗標**，向下相容邏輯：

| `subject_mission_share` | `mission_share_staff` 有記錄 | 行為 |
|---|---|---|
| 0 | — | 不分享，無人可查 |
| 1 | 有記錄 | 新邏輯：依關聯表決定誰可查 |
| 1 | 無記錄 | **舊資料**：退回原邏輯，僅該班導師可查 |

### 3.2 前端變更（assign_mission.php）

**取代原本的單一 checkbox**，改為：

1. 保留「任務觀看權限」區塊標題
2. 保留「開啟分享」toggle 開關，開啟後展開人員選擇區
3. 人員選擇區依角色分組顯示（勾選清單）：
   - **級任導師**：列出該班導師姓名
   - **科任教師**：列出教該班的其他科任老師姓名
   - **主任**：列出同校所有主任姓名
   - **校長**：列出同校校長姓名
4. 提供「全選」按鈕

**觸發時機**：前端在選擇目標班級後，AJAX 查詢可授權人員清單。

**事後編輯**：任務編輯頁面也要能修改已授權的人員（載入現有關聯 → 允許增減 → 儲存）。

### 3.3 後端變更（assign_mission_prodb.php）

#### 新增 AJAX action：`get_shareable_staff`

- **輸入**：`organization_id`, `grade`, `class`, `seme_year_seme`
- **輸出**：JSON 分組清單

```json
{
  "mentor": [{"user_id": "xxx", "uname": "王老師", "role": "級任導師"}],
  "subject_teacher": [{"user_id": "yyy", "uname": "李老師", "role": "科任教師（數學）"}],
  "admin": [{"user_id": "zzz", "uname": "張主任", "role": "主任"}, {"user_id": "www", "uname": "陳校長", "role": "校長"}]
}
```

**查詢邏輯**：

```sql
-- 1. 該班級任導師
SELECT ui.user_id, ui.uname
FROM user_info ui
JOIN user_status us ON ui.user_id = us.user_id
WHERE ui.organization_id = :org_id
  AND ui.grade = :grade AND ui.class = :class
  AND us.access_level IN (21, 25)
  AND ui.used = 1;

-- 2. 該班科任教師
SELECT DISTINCT sts.teacher_id AS user_id, ui.uname, stsm.subject_name
FROM seme_teacher_subject sts
JOIN seme_teacher_subject_mapping stsm ON sts.subject_mapping = stsm.mapping_sn
JOIN user_info ui ON sts.teacher_id = ui.user_id
WHERE sts.organization_id = :org_id
  AND sts.grade = :grade AND sts.class = :class
  AND sts.seme_year_seme = :seme
  AND ui.used = 1;

-- 3. 該校主任 + 校長
SELECT ui.user_id, ui.uname, us.access_level
FROM user_info ui
JOIN user_status us ON ui.user_id = us.user_id
WHERE ui.organization_id = :org_id
  AND us.access_level IN (31, 32)
  AND ui.used = 1;
```

- 排除任務指派者自己

#### 修改儲存邏輯

**新增任務時**：
1. `subject_mission_share` 依是否有勾選人員設 0/1
2. 勾選的人員 ID 批次 INSERT INTO `mission_share_staff`

**編輯任務時**：
1. 先 DELETE 該 mission_sn 的所有舊關聯
2. 重新 INSERT 新勾選的人員 ID
3. 更新 `subject_mission_share` 旗標

### 3.4 Dashboard 變更（modules_teacher_get_mission.php）

**修改「科任指派」查詢邏輯**：

```sql
-- 新邏輯（概念）
JOIN (
    -- 新資料：查關聯表
    SELECT mission_sn FROM mission_share_staff
    WHERE staff_id = :current_user_id

    UNION

    -- 舊資料向下相容：subject_mission_share=1 且關聯表無記錄 → 該班導師可查
    SELECT mi.mission_sn
    FROM mission_info mi
    WHERE mi.subject_mission_share = 1
      AND mi.mission_sn NOT IN (SELECT mission_sn FROM mission_share_staff)
      AND :current_user_is_mentor_of_target_class  -- 原有導師判斷邏輯
) viewable ON mission_info.mission_sn = viewable.mission_sn
```

這樣：
- 新資料走關聯表，精確控制誰能看
- 舊資料（有旗標但無關聯記錄）退回原邏輯，只有班導師可看
- 不影響現有 Dashboard 的呈現方式

---

## 四、實作步驟

| 順序 | 項目 | 說明 |
|------|------|------|
| 1 | 建立 `mission_share_staff` 表 | DDL + 索引 |
| 2 | 後端新增 `get_shareable_staff` | 查詢可授權人員清單 API |
| 3 | 後端修改儲存邏輯 | 新增/編輯時寫入關聯表 |
| 4 | 前端改造「任務觀看權限」UI | toggle + 人員勾選清單 |
| 5 | 前端接上事後編輯邏輯 | 載入現有授權 → 允許增減 |
| 6 | Dashboard 修改查詢邏輯 | 關聯表查詢 + 舊資料相容 |

---

## 五、影響範圍

| 項目 | 影響程度 | 說明 |
|------|----------|------|
| 資料庫 | 新增 1 張表 | `mission_info` 欄位不變 |
| assign_mission.php | 中度改動 | UI 區塊替換 + AJAX |
| assign_mission_prodb.php | 中度改動 | 新增 action + 修改儲存 |
| modules_teacher_get_mission.php | 小幅改動 | 查詢條件替換，保留舊資料相容 |
| 現有資料 | 不受影響 | 舊資料透過旗標邏輯自動相容 |
| 共享任務流程（assignshare） | 不動 | 暫不調整 |
