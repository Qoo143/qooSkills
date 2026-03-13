# 任務分享人員 V2 — 實作說明（按數據流向）

## 整體架構圖

```
┌─────────────────────────────────────────────────────────────────┐
│                        資料庫                                    │
│  mission_info.subject_mission_share  ←── 原有「開放導師」toggle   │
│  mission_share_staff (mission_sn, staff_id)  ←── 新增「指定人員」 │
│                                                                  │
│  兩者為 OR 關係，任一成立即可觀看                                  │
└──────────────────────────┬──────────────────────────────────────┘
                           │
          ┌────────────────┼────────────────┐
          ▼                ▼                ▼
      ① 指派時寫入    ② 編輯頁修改     ③ 查看端讀取
```

---

## ① 指派時寫入（Create Flow）

### 前端：assign_mission.php

**新增 PHP 變數**（供 JS 使用）：

```
routerShare → encodeAjaxUrlBase64('assignMission', 'prodb_mission_share')
currentOrgId → $_SESSION['user_data']->organization_id
currentSeme → getYearSeme()
```

**新增 UI 元件**：

- `SHARE_STAFF_SELECTOR` 常數 — 插入每種任務類型的 MORE_SETTING 組合中
  - 位置：在 `MISSION_AUTHORITY`（導師觀看 toggle）與 `SHARED_MISSION`（共享任務 radio）之間
  - 涵蓋 16 種任務類型 + 4 處動態 `append()`，共 20 處
- `#shareStaffModal` — 獨立的 Modal 彈窗（放在 `</section>` 外面）
  - 班級下拉選單 + 校級人員 checkbox 區 + 班級教師 checkbox 區

**JS 互動邏輯**：

```
使用者點「+ 選擇人員」
  → 暫存目前選擇（cancelable）
  → loadClassList()    → POST get_class_list    → 填入 <select>
  → loadSchoolAdmins() → POST get_school_admins → 填入校級 checkbox

使用者選班級
  → POST get_staff_by_class → 填入班級教師 checkbox（級任 + 科任）

使用者點「確定」
  → 從 Modal checkbox 收集選中人員 → shareStaffSelected[]
  → renderShareStaffTags() 顯示 tag

使用者點「取消」
  → 還原暫存的 shareStaffTempSelected
```

**提交時**：

```javascript
// 一般指派（~line 5933）
shareStaffIds: JSON.stringify(getShareStaffIds())

// 轉派（~line 4223）
shareStaffIds: JSON.stringify(getShareStaffIds())
```

> 用 `JSON.stringify` 是因為 jQuery `$.post` 會把陣列轉成 `key[]=val` 格式，後端用 `json_decode` 更可靠。

---

### 後端 API：prodb_mission_share.php（新增檔案）

透過 `turnpage.php` 路由進入，提供 5 個 action：

| Action | 用途 | 讀/寫 | DB 連線 |
|--------|------|-------|---------|
| `get_class_list` | Modal 班級下拉 | 讀 | `$dbh_slave` |
| `get_school_admins` | Modal 校級人員 | 讀 | `$dbh_slave` |
| `get_staff_by_class` | Modal 班級教師 | 讀 | `$dbh_slave` |
| `get_shared_staff` | 編輯頁載入已授權 | 讀 | `$dbh_slave` |
| `update_share_staff` | 編輯頁更新授權 | 寫 | `$dbh` |

**共用函式**（供指派 prodb 和本檔共用）：

```php
insertShareStaff($missionSn, $staffIds)   // 批次 INSERT
parseShareStaffIds()                       // 解析前端 JSON 字串
```

---

### 後端儲存：assign_mission_prodb.php（修改）

**解析位置**（2 處，因函式作用域不同）：

| 位置 | 原因 |
|------|------|
| 全域（~line 2068） | 一般指派的 6 條路徑都在此作用域 |
| `assignShareMission()` 內（~line 2591） | 轉派是獨立函式，有自己的作用域 |

**儲存位置**（6 處 `lastInsertId()` 之後）：

```
問卷 + 多班級      → 每班的 INSERT 後
問卷 + 個人/小組   → 單筆 INSERT 後
非問卷 + 多班級    → 每班的 INSERT 後
非問卷 + 個人/小組 → 單筆 INSERT 後
轉派 + 多班級      → 每班的 INSERT 後
轉派 + 單一        → 單筆 INSERT 後
```

每處邏輯相同：

```php
$iGetInsertMissionSn = $dbh->lastInsertId();
// ... 原有邏輯 ...
if (!empty($shareStaffIds)) {
    saveMissionShareStaff($iGetInsertMissionSn, $shareStaffIds);
}
```

**`saveMissionShareStaff` 函式**（檔案末尾新增）：
引入 `prodb_mission_share.php` 的 `insertShareStaff()` 來執行實際 INSERT。

> 多班級時每筆 mission_info 都存同一份 shareStaffIds，因為指定分享人員是指派者手動選的，不按班級過濾。

---

## ② 編輯頁修改（Update Flow）

### 前端：modules_mission_edit.php（修改）

**頁面載入**：

```
getMissionData()
  → makeMissionData()
    → loadSharedStaff()
      → POST get_shared_staff (mission_sn)
      → 填入 shareStaffSelected[]
      → renderShareStaffTags()
      → 若 is_legacy=true，顯示舊版提示
```

> `is_legacy` 判斷：`subject_mission_share=1` 但 `mission_share_staff` 無記錄 → 舊版資料

**UI 結構**：

```
moresettingWrap
  ├── 自動完成任務（原有）
  ├── 允許快轉影片（原有）
  ├── 任務觀看權限 #mission_share_mentor（原有 toggle）
  └── 指定分享人員 #share_staff_section（新增）
       ├── tag 列表 #shareStaffTags
       └── 「+ 選擇人員」按鈕 → 開啟 #shareStaffModal
```

**Modal 互動**：與指派頁完全相同的 JS 邏輯（loadClassList、loadSchoolAdmins、class change handler）。

**儲存**：

```javascript
// saveMission() 的 AJAX data 加入：
shareStaffIds: JSON.stringify(getShareStaffIds())
```

### 後端：modules_mission_edit_prodb.php（修改）

在 `$update2->execute()` 成功後：

```php
// 1. 解析前端 JSON
$shareStaffIds = json_decode($_POST['shareStaffIds'], true);

// 2. DELETE 舊關聯
DELETE FROM mission_share_staff WHERE mission_sn = :mission_sn

// 3. INSERT 新關聯
INSERT INTO mission_share_staff (mission_sn, staff_id) VALUES (...)
```

> 不動 `subject_mission_share` 旗標，那是 ShareMentor toggle 管的。

---

## ③ 查看端讀取（Read Flow）

### 後端：modules_teacher_get_mission.php（修改）

**新增常數**：

```php
define('SHARE_STAFF_ASSIGN', 5);  // 其他指派
```

**查詢邏輯**（在 `setAssignorTypeString()` 加入新分支）：

```
使用者在 dashboard 選「其他指派」(value=5)
  → setAssignorTypeString(5, ...)
  → 設定 JOIN：
      JOIN mission_share_staff mss
        ON mission_info.mission_sn = mss.mission_sn
       AND mss.staff_id = '{當前使用者 user_id}'
  → 主查詢自動套用 → 只回傳被分享給自己的任務
```

### 前端：modules_dashboard.php（修改）

篩選下拉選單新增選項：

```html
<li data-value="5">其他指派</li>
```

---

## 異動檔案清單

| 檔案 | 異動類型 | 主要修改 |
|------|---------|---------|
| `modules/assignMission/prodb_mission_share.php` | **新增** | 5 個 AJAX action + 2 個共用函式 |
| `modules/assignMission/assign_mission.php` | 修改 | CSS + Modal + JS + 20 處 MORE_SETTING + AJAX data |
| `modules/assignMission/assign_mission_prodb.php` | 修改 | 2 處解析 + 6 處儲存 + 1 個函式 |
| `modules/dashboard/modules_mission_edit.php` | 修改 | CSS + UI 區塊 + Modal + JS + AJAX data |
| `modules/dashboard/modules_mission_edit_prodb.php` | 修改 | execute 後 DELETE+INSERT |
| `modules/dashboard/modules_teacher_get_mission.php` | 修改 | 新常數 + JOIN 分支 |
| `modules/dashboard/modules_dashboard.php` | 修改 | 下拉選單加 1 項 |

---

## 耦合分析與可能的改善方向

### 目前的耦合點

| 耦合點 | 說明 | 嚴重度 |
|--------|------|--------|
| **JS 重複** | `assign_mission.php` 和 `modules_mission_edit.php` 有近乎相同的 Modal JS（~200 行） | 中 |
| **CSS 重複** | 兩個檔案各寫一份相同的 `.share-staff-*` CSS | 低 |
| **6 處 saveMissionShareStaff** | 散布在 `assign_mission_prodb.php` 的 6 個分支中 | 中 |
| **prodb 函式引用** | `saveMissionShareStaff()` 透過 `require_once` 引入 `prodb_mission_share.php`，但該檔的 switch-case 會因 `$_POST['act']` 為空而跳過 | 低 |

### 改善方案（可視需要採納）

**1. JS 抽共用檔案** — 降低重複度

```
scripts/share_staff_modal.js
  ├── initShareStaffModal(routerShare, orgId, seme)
  ├── loadSharedStaff(missionSn)   // 編輯頁用
  ├── getShareStaffIds()
  └── renderShareStaffTags()
```

兩個頁面只需 `<script src="scripts/share_staff_modal.js">` + 呼叫 init。

> 優點：修一處即全域生效
> 代價：新增一個 JS 檔，需確保載入順序

**2. CSS 抽共用檔案**

將 `.share-staff-*` CSS 獨立成 `css/share_staff.css`，兩頁面各 `<link>` 引入。

> 優點：單一來源
> 代價：多一個 HTTP 請求（可接受）

**3. 儲存邏輯集中化** — 降低 6 處散布風險

在 `assign_mission_prodb.php` 的 6 處 `lastInsertId` 後，目前是直接呼叫函式。
如果未來要加更多「指派後處理」（如通知、log），可考慮用 event hook 模式：

```php
// 指派後統一呼叫
function onMissionCreated($missionSn, $postData) {
    saveMissionShareStaff($missionSn, $postData);
    // 未來可加：sendNotification(), writeLog(), etc.
}
```

> 但目前只有 1 個 hook，抽象化反而過度設計。等有第 2 個需求時再重構即可。

### 結論

目前的實作**符合專案既有模式**（每個頁面自包含 CSS/JS、prodb 處理自己的邏輯）。
最值得做的改善是 **JS 抽共用檔案**（減少 ~200 行重複），其餘可在未來需求擴充時再處理。