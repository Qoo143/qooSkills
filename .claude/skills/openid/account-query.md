# 帳號查詢與登入決策

## 帳號查詢的 4 層結構

程式碼位置: `prodb_choiceIdentity.php:518-810`

帳號查詢在 `LOGIN_SELECTED_IDENTITY` 中執行,所有查詢會先全部執行完畢,再依優先順序判斷結果。

```
[第1層] 啟用帳號查詢 (used=1, 非轉出)
  |-- 優先順序 1: OpenID_sub
  |-- 優先順序 2: hash_guid (身分證雜湊)
  +-- 優先順序 3: 年班+姓名
       | 找不到
[第2層] 停用帳號查詢 (used=0)
  |-- 優先順序 1: OpenID_sub
  |-- 優先順序 2: hash_guid
  +-- 優先順序 3: 年班+姓名
       | 找不到
[第3層] 轉出帳號查詢 (export_school)
  |-- 優先順序 1: OpenID_sub
  +-- 優先順序 2: hash_guid
       | 找不到
[第4層] 同校同名查詢 (僅學生)
  +-- 學校+姓名+啟用&停用+非轉學+非畢業
```

---

## 第一層:啟用帳號查詢

三種查詢方式依序定義,查詢全部執行完後再判斷優先順序:

### 查詢 1: hash_guid (身分證雜湊)

```php
// prodb_choiceIdentity.php:534-539
if (isStudentRole($sUserLoginTitle) || isRoleInTeacherGroup($sUserLoginTitle)) {
    $arrEnabledUserInfoByGUID = getUserInfoByHashGuid(
        $arrLoginUserInfo['guid'], $sOrgCode,
        $arrLoginUserInfo['name'], $arrQueryUserAccessLevelList, true
    );
} else {
    $arrEnabledUserInfoByGUID = [];
}
```

- 適用:學生、教師
- 校管/局管不使用此查詢

### 查詢 2: OpenID_sub

```php
// prodb_choiceIdentity.php:543-563
if (isStudentRole($sUserLoginTitle) || isRoleInTeacherGroup($sUserLoginTitle)) {
    $arrEabledUserInfoByOpenSubList = getUserInfoByOpenIDSub(
        $arrLoginUserInfo['sub'], $sOrgCode, $arrQueryUserAccessLevelList, true
    );
} elseif (isRole($sUserLoginTitle, USER_TITLE_SCHOOL_MANAGEMENT)) {
    $arrEabledUserInfoByOpenSubList = getUserSchoolAdminInfo($arrLoginUserInfo['sub']);
} elseif (isRole($sUserLoginTitle, USER_TITLE_CITY_MANAGEMENT)) {
    // 教育局人員:先取得城市資訊再查詢
    $arrEabledUserInfoByOpenSubList = getUserCityManagementInfo($sCityMA, $sOrgCode);
}
```

- 適用:所有身分 (各身分使用不同查詢函式)
- 校管使用 `getUserSchoolAdminInfo()`
- 局管使用 `getUserCityManagementInfo()`

### 查詢 3: 年班+姓名

```php
// prodb_choiceIdentity.php:567-626
// 學生:查詢 seme_student (含/不含座號,取聯集)
if (isStudentRole($sUserLoginTitle)) {
    $arrEnabledUserInfoByClassInfo = getUserInfoByStudentClassInfo(...);
}
// 教師:查詢 user_info + seme_teacher_subject (多班級聯合查詢)
elseif (isRoleInTeacherGroup($sUserLoginTitle)) {
    // 為每個學校加上 0年0班 此學期的資料,增加找到帳號的機會
    // 同時查詢 getUserInfoByCalssInfo() 和 getUserInfoBySemeTeacherClassInfo()
    // 最後去重複: uniqueMultiDimArray($arr, "user_id")
}
```

- 學生:使用 `getUserInfoByStudentClassInfo()`,同時查含座號和不含座號,取聯集
- 教師:使用 `getUserInfoByCalssInfo()` + `getUserInfoBySemeTeacherClassInfo()`,遍歷所有任課班級查詢
- 教師查詢時會額外加入 0 年 0 班的搜尋條件,增加找到帳號機會

---

## 優先順序判斷邏輯

所有查詢執行完畢後,按以下順序判斷:

```php
// prodb_choiceIdentity.php:720-809
if ($iUserOpenSubEnabledAccountCount == 1) {
    // OpenID_sub 找到唯一帳號 --> 直接登入
    $bHasAccount = true;
    $bRequireUpdatedHashGuid = true;  // 需補綁身分證
} elseif ($iUserOpenSubEnabledAccountCount > 1) {
    // OpenID_sub 找到多筆 --> 顯示選單
    $bHasDuplicateAcceessLevelAccount = true;
} elseif ($iEnabledUserGUIDCount == 1) {
    if (!isEmptyHashGUID($arrEnabledUserInfoByGUID[0]['hash_guid'])) {
        // hash_guid 找到且有效 --> 直接登入
        $bHasAccount = true;
        $bRequireUpdatedOpenIDSub = true;  // 需補綁 OpenID_sub
    } else {
        // hash_guid 為空 --> 可能有帳號
        $bMayHasAccount = true;
    }
} elseif ($iEnabledUserInfoByClassInfoCount == 1) {
    // 年班+姓名找到唯一帳號 --> 直接登入
    $bHasAccount = true;
    $bRequireUpdatedOpenIDSub = true;   // 需補綁 OpenID_sub
    $bRequireUpdatedHashGuid = true;    // 需補綁身分證
} elseif ($iEnabledUserInfoByClassInfoCount > 1) {
    // 年班+姓名找到多筆 --> 可能有帳號
    $bMayHasAccount = true;
} elseif (/* 第1層全部找不到 */) {
    // 進入第2層、第3層、第4層判斷...
}
```

### 優先順序整理

| 優先順序 | 查詢方式 | 找到 1 筆 | 找到多筆 | 補綁動作 |
|:---------|:---------|:----------|:---------|:---------|
| 1 | OpenID_sub | 直接登入 | 顯示選單 | 補綁 hash_guid |
| 2 | hash_guid | 直接登入 (需非空) | - | 補綁 OpenID_sub |
| 3 | 年班+姓名 | 直接登入 | 可能有帳號 | 補綁 OpenID_sub + hash_guid |

---

## 第二層:停用帳號查詢

當第一層完全找不到時進入。查詢方式與第一層相同,但 `$bUsed = false`。

找到停用帳號時:

```php
// prodb_choiceIdentity.php:775-778
if (!$bIsAccountEnabled) {
    responseFailure("目前查看系統您的帳號已經停用,請聯絡校管或客服人員啟用帳號");
    die();
}
```

## 第三層:轉出帳號查詢

當第二層也找不到時,檢查 `export_school` 表:

```php
// prodb_choiceIdentity.php:675-689
$arrExportUserInfoByOpenIDSubList = getExportSchoolUserInfoByOpenIDSub(...);
$arrExportUserInfoByHashGuidList = getExportSchoolUserInfoByHashGuid(...);
```

找到轉出帳號時:

```php
// prodb_choiceIdentity.php:800-803
if ($bIsAccountExportSchool) {
    responseFailure("目前查看系統您的因材網帳號已經轉出,請聯絡校管或因材網客服人員確認帳號狀態");
    die();
}
```

## 第四層:同校同名查詢

僅限學生身分,防止重複創建帳號:

```php
// prodb_choiceIdentity.php:695-700, 805-808
if (isStudentRole($sUserLoginTitle)) {
    $arrUserInfoBySchoolAndName = getUserInfoBySchoolAndName(
        $sOrgCode, $arrLoginUserInfo['name'], $arrQueryUserAccessLevelList
    );
}
// 若找到同名學生,設為「可能有帳號」,前端顯示綁定選項
if (isStudentRole($sUserLoginTitle) && $iUserInfoBySchoolAndNameCount >= 1) {
    $bMayHasAccount = true;
}
```

---

## 台南 OpenID 臨時方案

由於台南 OpenID 不會傳送多重身分 (例如同時是教師+主任),因此使用寬鬆查詢邏輯。

程式碼位置: `function_OpenID.php:437-450`

```php
// K 版函數 - 寬鬆比對 (台南 OpenID 臨時方案)
function setQueryUserAccessLevelListForK($sRole) {
    if (isRole($sRole, "學生")) {
        return [USER_STUDENT];  // 1
    } elseif (isRoleInTeacherGroup($sRole)) {
        // 只要是教師群,就允許查詢所有教師群帳號
        return [
            USER_TEACHER,           // 21
            USER_LECTURER,          // 24
            USER_SCHOOL_DIRECTOR,   // 25
            USER_SCHOOL_PRINCIPAL   // 27
        ];
    }
    return [0];
}
```

### 嚴格版對比

```php
// 原版函數 - 嚴格比對
function setQueryUserAccessLevelList($sRole) {
    // 教師只查 [21, 24]
    // 主任只查 [25]
    // 各自分開查詢
}
```

### 實際影響

```
情境: OpenID 傳來「教師」,但因材網帳號是「主任」
K 版結果: 查得到 (查詢範圍包含 21, 24, 25, 27)
原版結果: 查不到 (只查 21, 24)
```

使用位置:

```php
// prodb_choiceIdentity.php:527
$arrQueryUserAccessLevelList = setQueryUserAccessLevelListForK($sUserLoginTitle);
```

> 注意:等台南 OpenID 修正多重身分傳輸後,需改回 `setQueryUserAccessLevelList()` 嚴格模式

---

## 查詢結果處理總覽

| 查詢結果 | 系統行為 | 後續處理 |
|:---------|:---------|:---------|
| 找到 1 筆啟用帳號 | 直接登入 | 執行資料同步、補綁識別碼 |
| 找到多筆啟用帳號 | 顯示選單 | 使用者選擇 --> SET_ENABLED_ACCOUNT |
| 找到停用帳號 | 阻止登入 | 顯示「帳號已停用,請聯絡管理員」 |
| 找到轉出帳號 | 阻止登入 | 顯示「帳號已轉出,請聯絡管理員」 |
| 同校同名 (學生) | 可能有帳號 | 顯示「創建新帳號」或「綁定舊帳號」選項 |
| 完全查無 | 檢查自動創建條件 | 自動創建 or 手動註冊 |

## 狀態旗標說明

程式碼定義的旗標 (`prodb_choiceIdentity.php:704-717`):

| 旗標 | 意義 | 後續動作 |
|:-----|:-----|:---------|
| `$bHasAccount` | 確認找到帳號 | 執行登入流程 |
| `$bMayHasAccount` | 可能有帳號 | 前端顯示選擇/綁定介面 |
| `$bHasDuplicateAcceessLevelAccount` | 同身分有多個帳號 | 前端顯示帳號選單 |
| `$bIsAccountEnabled` | 帳號是否啟用 | false 時阻止登入 |
| `$bIsAccountExportSchool` | 帳號是否轉出 | true 時阻止登入 |
| `$bRequireUpdatedHashGuid` | 需要補綁身分證 | 更新 user_info.hash_guid |
| `$bRequireUpdatedOpenIDSub` | 需要補綁 OpenID_sub | 更新 user_info.OpenID_sub |

## 相關文件

- [登入流程](./login-flow.md) - 完整登入流程
- [帳號創建](./account-create.md) - 查無帳號時的自動創建
- [資料同步](./data-sync.md) - 找到帳號後的同步動作
