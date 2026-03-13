# 任務觀看權限分享 V2 — 設計規格

## 1. 背景與現狀

### 1.1 現有功能（subject_mission_share）

目前指派任務時有一個 toggle 開關「任務觀看權限 — 開放此任務權限給導師」：

- **前端**：`#ShareMentor` checkbox，開啟時送 `ShareMentor=1`
- **後端**：寫入 `mission_info.subject_mission_share = 1`
- **查看端**：`modules_teacher_get_mission.php` 透過以下邏輯讓導師看到任務：
  - 查 `seme_teacher_subject` 找出導師帶的班（`subject_name = '導師班'`）
  - JOIN `seme_student` 找到該班學生
  - JOIN `mission_stud_record` + `mission_info` 找出 `subject_mission_share=1` 的任務

### 1.2 現有限制

| 限制 | 說明 |
|------|------|
| 只對導師開放 | 只有科任教師表中登記為「導師班」的教師才能看到 |
| 無法指定個人 | 不能選擇「只讓某位老師看」或「讓主任也能看」 |
| 班級導師限定 | 透過學生班級反查導師，非該班導師看不到 |
| 分組/跨校無解 | 小組學生可跨班甚至跨校，無法用班級反查邏輯處理 |

### 1.3 寫入 subject_mission_share 的位置（assign_mission_prodb.php，共 6 處）

| 路徑 | 行號 | 條件 | 值 |
|------|------|------|-----|
| 問卷 + 多班級 | 2205-2209 | `normal_class && share_mentor=1` 才寫 1 | 0 或 1 |
| 問卷 + 個人/小組 | 2296 | 直接用 `$share_mentor` | 0 或 1 |
| 非問卷 + 多班級 | 2360-2368 | `normal_class && share_mentor=1` 才寫 1 | 0 或 1 |
| 非問卷 + 個人/小組 | 2468 | 直接用 `$share_mentor` | 0 或 1 |
| 轉派 + 多班級 | 2676 | 直接用 POST 值 | 0 或 1 |
| 轉派 + 單一 | 2736 | ⚠ 疑似 bug：綁了 `autofinish` 的值 | 待確認 |

> ⚠ Line 2736：`subject_mission_share` 綁的是 `$_POST['mission_assignshare_autofinish']`，
> 而非 `$_POST['mission_assignshare_shareMentor']`，疑似 copy-paste bug。

### 1.4 前端觸發指派的入口（共 2 處）

| 入口 | 位置 | 說明 |
|------|------|------|
| 一般指派 | `assign_mission.php:5819` | `$.ajax` → `act: 'assignMissions'`，送 `ShareMentor` |
| 轉派 | `assign_mission.php:4125` | `$.ajax` → `act: 'assignShareMission'`，送 `mission_assignshare_shareMentor` |

### 1.5 指派對象的排列組合

前端有兩個維度決定指派對象：

**維度 1 — mission_target_type**（對象類別）：
- `C` — 班級（整班指派）
- `I` — 個人（勾選個別學生）
- `G` — 小組

**維度 2 — mission_class_type**（班級類別，僅 C 和 I 有）：
- `normal_class` — 一般班級（有 org_id, grade, class）
- `remedial_class` — 學習扶助班級
- `teacher_class` — 自組班級（學生可跨校）

**後端對應**：

| target_type | class_type | $bMutliClass | mission_info 筆數 | 備註 |
|-------------|------------|-------------|-------------------|------|
| C | normal_class | true | 每班一筆 | 最常見 |
| C | remedial_class | true | 每班一筆 | |
| C | teacher_class | true | 每班一筆 | 學生可跨校 |
| I | normal_class | false | 一筆 | |
| I | remedial_class | false | 一筆 | |
| I | teacher_class | false | 一筆 | 學生可跨校 |
| G | (null) | false | 一筆 | 學生可跨班跨校 |

---

## 2. V2 新設計

### 2.1 核心概念

**原有 ShareMentor toggle 完全保留不動**。

新增一個獨立的「指定分享人員」欄位，讓指派者透過 **Modal 選擇** 要授權觀看的人。
以手動選擇取代自動反查，避免跨班/跨校/分組的複雜邏輯。

### 2.2 與 ShareMentor 的關係

| ShareMentor | 指定分享人員 | 結果 |
|-------------|-------------|------|
| 關 | 空 | 只有指派者自己能看（現有行為） |
| 開 | 空 | 導師可看（現有行為不變） |
| 關 | 有選人 | 被選中的人可看（新功能） |
| 開 | 有選人 | 導師 + 被選中的人都可看（聯集） |

兩者是 **OR** 關係，不互斥。

### 2.3 DB 表（已建立）

```sql
CREATE TABLE mission_share_staff (
    sn          INT AUTO_INCREMENT PRIMARY KEY,
    mission_sn  INT NOT NULL,
    staff_id    VARCHAR(60) NOT NULL,
    created_at  DATETIME DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_mission_sn (mission_sn),
    INDEX idx_staff_id (staff_id),
    UNIQUE KEY uk_mission_staff (mission_sn, staff_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

---

## 3. 前端設計 — 方案 B：Modal 選擇

### 3.1 UI 佈局

在現有 `MISSION_AUTHORITY`（ShareMentor toggle）**下方**新增區塊：

```
┌─ 更多設定 ─────────────────────────────────────────┐
│                                                     │
│  [任務觀看權限]                                      │
│  ☑ 開放此任務權限給導師            ← 原有 toggle     │
│                                                     │
│  [指定分享人員]                     ← 新增區塊       │
│  已選擇：王大明、李小華             [+ 選擇人員]      │
│                                                     │
└─────────────────────────────────────────────────────┘
```

- 無人選擇時顯示「尚未指定」
- 有人選擇時顯示已選人員名稱列表（tag 形式，可個別移除）
- 點「+ 選擇人員」開啟 Modal

### 3.2 Modal 互動流程

```
┌─── 選擇分享人員 ──────────────────────────────────┐
│                                                    │
│  學校：[▼ ○○國小 (本校)        ]                   │
│  班級：[▼ 請選擇班級            ]  ← 選校後載入     │
│                                                    │
│  ┌─ 查詢結果 ──────────────────────────────────┐  │
│  │                                              │  │
│  │  ── 校級人員 ──                              │  │
│  │  ☐ 張主任（主任）                             │  │
│  │  ☐ 林校長（校長）                             │  │
│  │                                              │  │
│  │  ── 三年一班 教師 ──          ← 選班後出現    │  │
│  │  ☐ 王大明（級任導師）                         │  │
│  │  ☐ 李小華（科任教師 — 數學）                   │  │
│  │  ☐ 陳老師（科任教師 — 英語）                   │  │
│  │                                              │  │
│  └──────────────────────────────────────────────┘  │
│                                                    │
│                        [取消]  [確定]               │
└────────────────────────────────────────────────────┘
```

**互動細節**：

1. **開啟 Modal 時**：
   - 學校預設本校（`$_SESSION['user_data']->organization_id`）
   - 校級人員（主任/校長）**立即載入**，不需選班
   - 班級下拉選單載入該校當學期的班級列表

2. **選擇班級後**：
   - 查詢該班的級任導師 + 科任教師
   - 顯示於校級人員下方
   - 可切換班級重新查詢，已勾選的人員保留

3. **確定後**：
   - 勾選的人員加入已選列表（tag 形式）
   - Modal 關閉
   - 可再次開啟 Modal 繼續選人（已選的自動打勾）

4. **學校下拉選單**：
   - 預設只有本校
   - 未來若需跨校，可擴充為查詢所有學校（或輸入學校名稱搜尋）
   - V1 先做本校即可，跨校需求較少

### 3.3 前端觸發時機與各情境處理

新增欄位在**所有指派類型**下都可用，不受 mission_target_type 或 mission_class_type 影響：

| 情境 | ShareMentor toggle | 指定分享人員 | 說明 |
|------|-------------------|-------------|------|
| 一般班級 (normal_class) | 可用 | 可用 | 最常見 |
| 學習扶助班級 (remedial_class) | 可用 | 可用 | |
| 自組班級 (teacher_class) | 可用 | 可用 | 跨校時特別有用 |
| 小組 (G) | 可用 | 可用 | 跨班跨校時特別有用 |
| 個人 (I) | 可用 | 可用 | |
| 轉派 (assignshare) | 可用 | 可用 | |

> 指定分享人員是獨立功能，不依賴指派對象的類型，所以所有情境都通用。

### 3.4 涉及頁面（共 3 處 UI）

| 頁面 | 位置 | 說明 |
|------|------|------|
| `assign_mission.php` | `MISSION_AUTHORITY` 下方 | 一般指派的「指定分享人員」 |
| `assign_mission.php` | 轉派 Modal 的 `assignshare__share_mentor` 下方 | 轉派的「指定分享人員」 |
| `modules_mission_edit.php` | `#mission_share_mentor` 下方 | 編輯頁載入 + 修改 |

---

## 4. 後端設計

### 4.1 新增後端 API — `prodb_mission_share.php`

透過 `turnpage.php` 路由：`encodeAjaxUrlBase64('assignMission', 'prodb_mission_share')`

| Action | 用途 | 輸入參數 | 輸出 |
|--------|------|---------|------|
| `get_class_list` | Modal 載入班級下拉選單 | `org_id`, `seme` | 班級列表 `[{class_sn, class_name}]` |
| `get_staff_by_class` | 選班後查教師 | `org_id`, `grade`, `class_num`, `seme` | 級任 + 科任列表 |
| `get_school_admins` | Modal 開啟時查校級人員 | `org_id` | 主任 + 校長列表 |
| `get_shared_staff` | 編輯頁載入已授權人員 | `mission_sn` (hashed) | 已授權 user_id 列表 |
| `update_share_staff` | 編輯頁更新授權人員 | `mission_sn` (hashed), `staff_ids` (JSON) | success/error |

### 4.2 SQL 查詢

**get_school_admins** — 校級人員（選校即查，不需選班）：
```sql
SELECT ui.user_id, ui.uname, us.access_level
FROM user_info ui
JOIN user_status us ON ui.user_id = us.user_id
WHERE ui.organization_id = :org_id
  AND us.access_level IN (USER_SCHOOL_ADMIN_GROUP)  -- 31, 32
  AND ui.used = 1
  AND ui.user_id != :current_user
```

**get_staff_by_class** — 該班教師（級任 + 科任合併回傳）：
```sql
-- 級任導師
SELECT ui.user_id, ui.uname, us.access_level, NULL AS subject_name
FROM user_info ui
JOIN user_status us ON ui.user_id = us.user_id
WHERE ui.organization_id = :org_id
  AND ui.grade = :grade
  AND ui.class = :class_num
  AND us.access_level IN (USER_TEACHER, USER_LECTURER)  -- 21, 25
  AND ui.used = 1
  AND ui.user_id != :current_user

UNION

-- 科任教師
SELECT DISTINCT sts.teacher_id AS user_id, ui.uname, us.access_level, stsm.subject_name
FROM seme_teacher_subject sts
JOIN seme_teacher_subject_mapping stsm ON sts.subject_mapping = stsm.mapping_sn
JOIN user_info ui ON sts.teacher_id = ui.user_id
JOIN user_status us ON ui.user_id = us.user_id
WHERE sts.organization_id = :org_id
  AND sts.grade = :grade
  AND sts.class = :class_num
  AND sts.seme_year_seme = :seme
  AND ui.used = 1
  AND sts.teacher_id != :current_user
```

**get_class_list** — 班級下拉選單：
```sql
-- 可複用 TeacherClasses 或直接查 seme_student 取得該校當學期的年班組合
SELECT DISTINCT ss.grade, ss.class,
       CONCAT(ss.grade, '年', ss.class, '班') AS class_name
FROM seme_student ss
WHERE ss.organization_id = :org_id
  AND ss.seme_year_seme = :seme
ORDER BY ss.grade, ss.class
```

> 注意：get_class_list 查的是該校所有班級，不限於目前指派的對象班級。
> 因為指派者可能想分享給不在指派對象班級的老師。

### 4.3 指派時儲存（assign_mission_prodb.php）

在每處 `lastInsertId()` 之後加入儲存邏輯：

```php
$missionSn = $dbh->lastInsertId();

// 儲存指定分享人員
$shareStaffIds = parseShareStaffIds(); // 解析前端 JSON
if (!empty($shareStaffIds)) {
    saveMissionShareStaff($missionSn, $shareStaffIds);
}
```

**需要加入的位置**（與 1.3 的 6 處對應）：

| 路徑 | lastInsertId 行號 | 說明 |
|------|-------------------|------|
| 問卷 + 多班級 | ~2240 | foreach 每班的 INSERT 後 |
| 問卷 + 個人/小組 | ~2300 | 單筆 INSERT 後 |
| 非問卷 + 多班級 | ~2410 | foreach 每班的 INSERT 後 |
| 非問卷 + 個人/小組 | ~2470 | 單筆 INSERT 後 |
| 轉派 + 多班級 | ~2710 | foreach 每班的 INSERT 後 |
| 轉派 + 單一 | ~2770 | 單筆 INSERT 後 |

> 多班級時每筆 mission_info 都存同一份 shareStaffIds。
> 指定分享人員是指派者手動選的，不需要按班級過濾。

### 4.4 查看端（modules_teacher_get_mission.php）

現有 `subject_mission_share=1` 的 JOIN 邏輯（line 1138-1162）**不動**。

新增 UNION 分支，讓被指定的人也能看到任務：

```sql
-- 原有子查詢的 ) 前加入 UNION
UNION
SELECT DISTINCT mss.mission_sn
FROM mission_share_staff mss
JOIN mission_info mi ON mss.mission_sn = mi.mission_sn
  AND mi.unable = 1
WHERE mss.staff_id = :current_user_id
```

### 4.5 編輯頁（modules_mission_edit.php）

- 頁面載入時呼叫 `get_shared_staff` 取得已授權人員
- 顯示為 tag 列表 + Modal（同指派頁）
- 修改後呼叫 `update_share_staff` 存回

`update_share_staff` 邏輯（transaction）：
```
1. DELETE FROM mission_share_staff WHERE mission_sn = :mission_sn
2. INSERT 新的 staff_id 列表
3. 不動 subject_mission_share 旗標（那是 ShareMentor 管的）
```

---

## 5. 資料流總覽

```
指派頁 (assign_mission.php)
  │
  ├─ ShareMentor toggle ──→ mission_info.subject_mission_share = 0/1    (不動)
  │
  └─ 指定分享人員 Modal ──→ mission_share_staff (mission_sn, staff_id)  (新增)
       │
       ├─ get_school_admins  → 校級人員（主任/校長）
       ├─ get_class_list     → 班級下拉
       └─ get_staff_by_class → 該班教師（級任+科任）

查看端 (modules_teacher_get_mission.php)
  │
  ├─ 原有邏輯：subject_mission_share=1 → 導師班反查                     (不動)
  │
  └─ 新增 UNION：mission_share_staff.staff_id = 當前用戶               (新增)

編輯頁 (modules_mission_edit.php)
  │
  ├─ ShareMentor toggle：讀取 subject_mission_share                    (不動)
  │
  └─ 指定分享人員：get_shared_staff → Modal 修改 → update_share_staff  (新增)
```

---

## 6. 前端 AJAX data 傳遞變更

### 6.1 一般指派（assign_mission.php:5825）

現有 data 物件新增一個欄位：

```javascript
data: {
    // ... 現有欄位全部保留 ...
    ShareMentor: ShareMentor,
    shareStaffIds: JSON.stringify(shareStaffIds)   // ← 新增
}
```

### 6.2 轉派（assign_mission.php:4129）

```javascript
data: {
    // ... 現有欄位全部保留 ...
    mission_assignshare_shareMentor: mission_assignshare_shareMentor,
    shareStaffIds: JSON.stringify(shareStaffIds)   // ← 新增
}
```

> `shareStaffIds` 是 JS 陣列，例如 `["190041-t001", "190041-t002"]`。
> 用 `JSON.stringify` 是因為 jQuery $.post 會把陣列拆成 `key[]=val` 格式，
> 但後端用 `json_decode` 接收，需要 JSON 字串。

---

## 7. 決策記錄

| 項目 | 決策 |
|------|------|
| 跨校支援 | V1 先做本校，學校下拉預設且僅有本校 |
| Line 2736 bug | 本次不修正，維持現狀 |
| Dashboard 分類 | 新增一個 tab「其他指派」 |
| 分享人數上限 | 不設上限 |
| 班級列表資料來源 | 直接查 `seme_student` 取該校當學期所有年班組合 |

---

## 8. 影響範圍

| 檔案 | 動作 | 變更內容 |
|------|------|---------|
| `prodb_mission_share.php` | **新建** | 5 個 AJAX action |
| `assign_mission.php` | 修改 | 新增 Modal HTML + JS（2 處 UI + AJAX data） |
| `assign_mission_prodb.php` | 修改 | 6 處 lastInsertId 後加 saveMissionShareStaff |
| `modules_mission_edit.php` | 修改 | 新增 Modal + 載入/儲存邏輯 |
| `modules_teacher_get_mission.php` | 修改 | UNION 查 mission_share_staff |
| `modules_dashboard.php` | 可能修改 | tab 文字調整（若需要） |
| DB | 已建立 | `mission_share_staff` 表 |
