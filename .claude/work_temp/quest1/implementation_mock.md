# 任務觀看權限擴展 — 實作樣態預覽 v2

> 已確認架構：後端獨立新檔，前端改原檔最小幅度修改。
> case 內只呼叫 function，邏輯定義在外面，保持閱讀性與語意化。

---

## 檔案總覽

| 檔案 | 動作 | 說明 |
|------|------|------|
| `modules/assignMission/prodb_mission_share.php` | **新建** | 分享相關所有 AJAX action |
| `modules/assignMission/assign_mission.php` | 改 | MISSION_AUTHORITY HTML + JS + 送出參數 |
| `modules/assignMission/assign_mission_prodb.php` | 小改 | INSERT 後加幾行寫關聯表 |
| `modules/dashboard/modules_teacher_get_mission.php` | 小改 | 科任指派查詢條件替換 |

---

## 1. 資料庫 DDL

```sql
CREATE TABLE mission_share_staff (
    sn            INT AUTO_INCREMENT PRIMARY KEY,
    mission_sn    INT NOT NULL,
    staff_id    VARCHAR(60) NOT NULL,
    created_at    DATETIME DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_mission_sn (mission_sn),
    INDEX idx_staff_id (staff_id),
    UNIQUE KEY uk_mission_staff (mission_sn, staff_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

---

## 2. 後端新檔 — prodb_mission_share.php

```php
<?php
/**
 * 任務觀看權限分享管理
 * 透過 turnpage.php router 進入
 * encodeAjaxUrlBase64('assignMission', 'prodb_mission_share')
 */

$action = $_POST['act'] ?? '';

switch ($action) {
    case 'get_shareable_staff':
        getShareableStaff($dbh_slave);
        break;

    case 'get_shared_staff':
        getSharedStaff($dbh_slave);
        break;

    case 'update_share_staff':
        updateShareStaff($dbh);
        break;
}

// ========== 查詢可授權人員 ==========
function getShareableStaff($dbh_slave) {
    $orgId = xss_filter($_POST['organization_id']);
    $grade = intval($_POST['grade']);
    $classNum = intval($_POST['class_num']);
    $seme = xss_filter($_POST['seme_year_seme']);
    $currentUserId = $_SESSION['user_data']->user_id;

    $result = [
        'mentor' => [],
        'subject_teacher' => [],
        'admin' => []
    ];

    // 1. 該班級任導師（user_info.grade/class 匹配）
    $sql = "SELECT ui.user_id, ui.uname
            FROM user_info ui
            JOIN user_status us ON ui.user_id = us.user_id
            WHERE ui.organization_id = :org_id
              AND ui.grade = :grade
              AND ui.class = :class_num
              AND us.access_level IN (21, 25)
              AND ui.used = 1
              AND ui.user_id != :current_user";
    $stmt = $dbh_slave->prepare($sql);
    $stmt->execute([
        ':org_id' => $orgId,
        ':grade' => $grade,
        ':class_num' => $classNum,
        ':current_user' => $currentUserId
    ]);
    $result['mentor'] = $stmt->fetchAll(PDO::FETCH_ASSOC);

    // 2. 該班科任教師（seme_teacher_subject）
    $sql = "SELECT DISTINCT sts.teacher_id AS user_id, ui.uname, stsm.subject_name
            FROM seme_teacher_subject sts
            JOIN seme_teacher_subject_mapping stsm ON sts.subject_mapping = stsm.mapping_sn
            JOIN user_info ui ON sts.teacher_id = ui.user_id
            WHERE sts.organization_id = :org_id
              AND sts.grade = :grade
              AND sts.class = :class_num
              AND sts.seme_year_seme = :seme
              AND ui.used = 1
              AND sts.teacher_id != :current_user";
    $stmt = $dbh_slave->prepare($sql);
    $stmt->execute([
        ':org_id' => $orgId,
        ':grade' => $grade,
        ':class_num' => $classNum,
        ':seme' => $seme,
        ':current_user' => $currentUserId
    ]);
    $result['subject_teacher'] = $stmt->fetchAll(PDO::FETCH_ASSOC);

    // 3. 該校主任 + 校長
    $sql = "SELECT ui.user_id, ui.uname, us.access_level
            FROM user_info ui
            JOIN user_status us ON ui.user_id = us.user_id
            WHERE ui.organization_id = :org_id
              AND us.access_level IN (31, 32)
              AND ui.used = 1
              AND ui.user_id != :current_user";
    $stmt = $dbh_slave->prepare($sql);
    $stmt->execute([
        ':org_id' => $orgId,
        ':current_user' => $currentUserId
    ]);
    $result['admin'] = $stmt->fetchAll(PDO::FETCH_ASSOC);

    echo json_encode($result);
}

// ========== 查詢已授權人員（編輯時載入用） ==========
function getSharedStaff($dbh_slave) {
    $missionSn = intval($_POST['mission_sn']);

    $sql = "SELECT staff_id FROM mission_share_staff WHERE mission_sn = :mission_sn";
    $stmt = $dbh_slave->prepare($sql);
    $stmt->execute([':mission_sn' => $missionSn]);
    $sharedList = $stmt->fetchAll(PDO::FETCH_COLUMN);

    echo json_encode(['shared_staff_ids' => $sharedList]);
}

// ========== 更新授權人員（事後編輯） ==========
function updateShareStaff($dbh) {
    $missionSn = intval($_POST['mission_sn']);
    $shareMentor = intval($_POST['ShareMentor'] ?? 0);
    $shareStaffIds = json_decode($_POST['shareStaffIds'] ?? '[]', true);

    if (!is_array($shareStaffIds)) {
        $shareStaffIds = [];
    }
    $shareStaffIds = array_map('xss_filter', $shareStaffIds);

    $dbh->beginTransaction();
    try {
        // 1. 刪除舊關聯
        $stmt = $dbh->prepare("DELETE FROM mission_share_staff WHERE mission_sn = :mission_sn");
        $stmt->execute([':mission_sn' => $missionSn]);

        // 2. 插入新關聯
        if ($shareMentor == 1 && !empty($shareStaffIds)) {
            insertShareStaff($dbh, $missionSn, $shareStaffIds);
        }

        // 3. 更新旗標
        $stmtFlag = $dbh->prepare(
            "UPDATE mission_info SET subject_mission_share = :flag WHERE mission_sn = :mission_sn"
        );
        $stmtFlag->execute([
            ':flag' => ($shareMentor == 1) ? 1 : 0,
            ':mission_sn' => $missionSn
        ]);

        $dbh->commit();
        echo json_encode(['status' => 'success']);
    } catch (Exception $e) {
        $dbh->rollBack();
        echo json_encode(['status' => 'error', 'message' => $e->getMessage()]);
    }
}

// ========== 共用：批次寫入關聯表 ==========
function insertShareStaff($dbh, $missionSn, $teacherIds) {
    $stmt = $dbh->prepare(
        "INSERT INTO mission_share_staff (mission_sn, staff_id) VALUES (:mission_sn, :staff_id)"
    );
    foreach ($teacherIds as $tid) {
        $stmt->execute([
            ':mission_sn' => $missionSn,
            ':staff_id' => $tid
        ]);
    }
}
```

---

## 3. 前端改動 — assign_mission.php

### 3.1 新增 router 變數（頁面上方 PHP 區）

```php
<?php
$shareRouter = encodeAjaxUrlBase64('assignMission', 'prodb_mission_share');
?>
```

### 3.2 修改 MISSION_AUTHORITY 區塊（約 line 661）

```javascript
const MISSION_AUTHORITY = `
    <div class="switch--knob more-settings" id="mission_share_mentor">
        <div class="more-settings--textBox">
            <h2 class="more-settings--title">任務觀看權限</h2>
            <p class="more-settings--text">開放此任務權限給其他教職人員</p>
        </div>
        <input type="checkbox" class="switch--checkbox" name="ShareMentor" id="ShareMentor">
        <label class="switch--btn" for="ShareMentor">開放</label>
    </div>
    <div id="share_staff_list" style="display:none; margin-top:10px; padding:10px; border:1px solid #ddd; border-radius:8px;">
        <div style="margin-bottom:8px;">
            <label><input type="checkbox" id="share_select_all"> 全選</label>
        </div>
        <div id="share_group_mentor">
            <h3 style="font-size:14px; color:#666; margin:8px 0 4px;">級任導師</h3>
            <div id="share_mentor_checkboxes"></div>
        </div>
        <div id="share_group_subject">
            <h3 style="font-size:14px; color:#666; margin:8px 0 4px;">科任教師</h3>
            <div id="share_subject_checkboxes"></div>
        </div>
        <div id="share_group_admin">
            <h3 style="font-size:14px; color:#666; margin:8px 0 4px;">主任 / 校長</h3>
            <div id="share_admin_checkboxes"></div>
        </div>
    </div>`;
```

### 3.3 新增 JS 邏輯

```javascript
var shareRouter = '<?php echo $shareRouter; ?>';

// ShareMentor toggle → 查詢人員
$(document).on('change', '#ShareMentor', function() {
    if ($(this).prop('checked')) {
        let orgId = '<?php echo $_SESSION["user_data"]->organization_id; ?>';
        let targetClass = /* 取得目前已選擇的班級 target_id */;
        let parts = targetClass.split('-');
        let grade = parts[1];
        let classNum = parseInt(parts[2]);
        let seme = '<?php echo $sYearSeme; ?>';

        $.post('../../turnpage.php', {
            router: shareRouter,
            act: 'get_shareable_staff',
            organization_id: orgId,
            grade: grade,
            class_num: classNum,
            seme_year_seme: seme
        }, function(response) {
            let data = JSON.parse(response);
            renderShareStaffList(data);
            $('#share_staff_list').slideDown();
        });
    } else {
        $('#share_staff_list').slideUp();
        $('#share_staff_list input[type="checkbox"]').prop('checked', false);
    }
});

// 全選
$(document).on('change', '#share_select_all', function() {
    $('#share_staff_list input.share-staff-cb').prop('checked', $(this).prop('checked'));
});

// 渲染人員清單
function renderShareStaffList(data, preChecked) {
    preChecked = preChecked || [];

    let mentorHtml = buildCheckboxGroup(data.mentor, preChecked, function(t) {
        return t.uname;
    });
    $('#share_mentor_checkboxes').html(mentorHtml);

    let subjectHtml = buildCheckboxGroup(data.subject_teacher, preChecked, function(t) {
        return t.uname + ' (' + t.subject_name + ')';
    });
    $('#share_subject_checkboxes').html(subjectHtml);

    let adminHtml = buildCheckboxGroup(data.admin, preChecked, function(t) {
        let role = t.access_level == 32 ? '校長' : '主任';
        return t.uname + ' (' + role + ')';
    });
    $('#share_admin_checkboxes').html(adminHtml);
}

// 共用：產生 checkbox HTML
function buildCheckboxGroup(list, preChecked, labelFn) {
    if (!list || list.length === 0) {
        return '<span style="color:#999;">無</span>';
    }
    let html = '';
    list.forEach(function(t) {
        let checked = preChecked.includes(t.user_id) ? 'checked' : '';
        html += '<label style="display:inline-block; margin:4px 12px 4px 0;">' +
            '<input type="checkbox" class="share-staff-cb" value="' + t.user_id + '" ' + checked + '> ' +
            labelFn(t) +
            '</label>';
    });
    return html;
}
```

### 3.4 修改送出資料收集（約 line 5517）

```javascript
let ShareMentor = $("#ShareMentor").prop('checked');
let shareStaffIds = [];
if (ShareMentor) {
    ShareMentor = 1;
    $('input.share-staff-cb:checked').each(function() {
        shareStaffIds.push($(this).val());
    });
} else {
    ShareMentor = 0;
}

// AJAX data 中（約 line 5839）
data: {
    // ...原有欄位...
    ShareMentor: ShareMentor,
    shareStaffIds: JSON.stringify(shareStaffIds),
    // ...
}
```

---

## 4. 後端小改 — assign_mission_prodb.php

### 4.1 接收新參數（約 line 2068 後新增）

```php
$shareStaffIds = [];
if (isset($_POST['shareStaffIds']) && $_POST['shareStaffIds'] != '') {
    $shareStaffIds = json_decode($_POST['shareStaffIds'], true);
    if (!is_array($shareStaffIds)) {
        $shareStaffIds = [];
    }
    $shareStaffIds = array_map('xss_filter', $shareStaffIds);
}
```

### 4.2 INSERT mission_info 後寫關聯（約 line 2232 後新增）

```php
$iGetInsertMissionSn = $dbh->lastInsertId();

// === 新增：寫入任務分享教師關聯 ===
if ($share_mentor == 1 && !empty($shareStaffIds)) {
    // 引用 prodb_mission_share.php 的 insertShareStaff()
    // 或直接在此處寫入（因為 prodb 檔案不會被 include，需要獨立寫）
    $sqlInsertShare = "INSERT INTO mission_share_staff (mission_sn, staff_id) VALUES (:mission_sn, :staff_id)";
    $stmtShare = $dbh->prepare($sqlInsertShare);
    foreach ($shareStaffIds as $tid) {
        $stmtShare->execute([
            ':mission_sn' => $iGetInsertMissionSn,
            ':staff_id' => $tid
        ]);
    }
}
```

> 注意：`assign_mission_prodb.php` 中有多處 INSERT mission_info（不同任務類型），每處 `lastInsertId()` 後都要加。

---

## 5. Dashboard 小改 — modules_teacher_get_mission.php

### 修改「科任指派」查詢（約 line 1120-1163）

```php
} elseif ($iAssignorType == SUBJECT_TEACHER_ASSIGN) {

    $sCurrentUserId = array2sqlIN($arrUserInfo['user_id']);

    $sSqlUnion = '';
    if (strval($sSeme) === getYearSeme()) {
        $sSqlUnion =
            " UNION
              SELECT '" . $sSeme . "' seme_year_seme, organization_id, grade, class
              FROM user_info
              WHERE user_id IN ($sCurrentUserId) ";
    }

    $arrResult['iAssignorType']['join_table'] =
        " JOIN (
            -- 新資料：mission_share_staff 有明確授權
            SELECT DISTINCT mission_sn
            FROM mission_share_staff
            WHERE staff_id IN ($sCurrentUserId)

            UNION

            -- 舊資料相容：subject_mission_share=1 且關聯表無記錄
            SELECT DISTINCT mi.mission_sn
            FROM (
                SELECT sts.seme_year_seme, sts.organization_id, sts.grade, sts.class
                FROM seme_teacher_subject sts
                JOIN seme_teacher_subject_mapping stsm ON sts.subject_mapping = stsm.mapping_sn
                AND stsm.subject_name = '導師班'
                WHERE sts.seme_year_seme = '" . $sSeme . "'
                AND sts.teacher_id IN ($sCurrentUserId)
                $sSqlUnion
            ) tmp_class
            JOIN seme_student ss
                ON ss.seme_year_seme = tmp_class.seme_year_seme
                AND ss.organization_id = tmp_class.organization_id
                AND ss.grade = tmp_class.grade
                AND ss.class = tmp_class.class
            JOIN mission_stud_record msr ON ss.stud_id = msr.user_id
            JOIN mission_info mi ON msr.mission_sn = mi.mission_sn
                AND mi.unable = 1
                AND mi.subject_mission_share = 1
                AND CONVERT(ss.seme_year_seme, UNSIGNED) = mi.semester
            WHERE mi.mission_sn NOT IN (
                SELECT mission_sn FROM mission_share_staff
            )
        ) tmp ON mission_info.mission_sn = tmp.mission_sn";

    $arrResult['iAssignorType']['has_join_table'] = 1;
    $arrResult['iAssignorType']['has_bind_value'] = 0;
    $arrResult['iAssignorType']['has_query_string'] = 0;
}
```

---

## 6. 資料流程圖

```
【指派任務】

前端選班級 → toggle「開放」
    ↓
AJAX → prodb_mission_share.php?act=get_shareable_staff
    ↓  （走獨立 router）
回傳分組人員 → 前端渲染勾選清單
    ↓
送出任務 → assign_mission_prodb.php（原有流程）
    ↓  ShareMentor=1, shareStaffIds=["id1","id2"]
INSERT mission_info → lastInsertId()
    ↓
INSERT mission_share_staff（逐筆）


【Dashboard 查詢】

登入者 → 「科任指派」
    ↓
UNION：查 mission_share_staff(新) + 舊資料相容邏輯
    ↓
有授權 → 顯示任務


【事後編輯】

載入任務 → prodb_mission_share.php?act=get_shared_staff（已授權清單）
         → prodb_mission_share.php?act=get_shareable_staff（可授權清單）
    ↓
已授權的預設勾選 → 修改後
    ↓
prodb_mission_share.php?act=update_share_staff
    → transaction { DELETE → INSERT → UPDATE 旗標 }
```
