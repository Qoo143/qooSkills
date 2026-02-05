# 自動創建帳號

## 觸發條件

程式碼位置: `prodb_choiceIdentity.php:838-920`

```php
$bIsAutoCreateAccount = !$bHasAccount && (
    isRole($sUserLoginTitle, USER_TITLE_CITY_MANAGEMENT) ||      // 教育局人員
    isRole($sUserLoginTitle, USER_TITLE_SCHOOL_MANAGEMENT) ||    // 校管人員
    ($bOpenIdTrust && isStudentRole($sUserLoginTitle))           // 信任學校學生
);
```

三個條件:

1. 帳號查詢 4 層全部找不到 (`!$bHasAccount`)
2. 身分為教育局人員、校管人員、或信任學校學生
3. 教師和非信任學校學生不會自動創建

---

## 不同身分的創建規則

| 身分 | OpenID 信任 | 年班資料 | 創建內容 | 原因 |
|:-----|:------------|:---------|:---------|:-----|
| 教育局人員 | 不限 | 不需要 | grade=0, class=0,不建立學期資料 | 管理人員無需年班 |
| 校管人員 | 不限 | 不需要 | grade=0, class=0,不建立學期資料 | 管理人員無需年班 |
| 信任學校學生 | ✓ 必須 | 有效 | 使用 OpenID 完整資料 (年班、學期、座號) | 資料可信,可自動創建 |
| 信任學校學生 | ✓ 必須 | 無效 | 使用基本資料,grade/class 使用查詢轉換值 | 資料不完整,設基本值 |
| 非信任學校學生 | ✗ | 不限 | 不自動創建 | 資料可信度低,需手動註冊 |
| 教師 | 不限 | 不限 | 不自動創建 | 任課資料複雜,需手動註冊 |

---

## 創建流程

### 步驟 1: 產生帳號 ID

```php
$iNextUid = getNextUserInfoAI();  // 取得 AUTO_INCREMENT
$sNewUserid = $sOrgCode . "-user" . $iNextUid;  // 例: 093501-user12345
```

`getNextUserInfoAI()` 查詢 `information_schema.tables` 的 AUTO_INCREMENT 值 (`function_addUser.php:25-47`)。

### 步驟 2: 產生隨機密碼

```php
$sPassWord = substr(md5(random_int(0, 2147483647)), 0, 9);
$sHashPassWord = password_hash(string2hash($sPassWord, false), PASSWORD_DEFAULT);
```

### 步驟 3: 準備 user_info 資料

信任學校學生 + 年班有效:

```php
$arrUserInfoInsertData = exportCreateUserAccountUserInfoByOpenIDUserInfo(
    $oOpenIDUserInfo, $sOrgCode, $sNewUserid, $sHashPassWord,
    $arrOpenIDStudentClassInfo['year']
);
$arrSemeInsertData = exportCreateSemeStudentInfo(
    $oOpenIDUserInfo, $sOrgCode, $sNewUserid, $sCurrentSemeYearSeme
);
```

信任學校學生 + 年班無效:

```php
$arrUserInfoInsertData['grade'] = $iSearchStudentGrade;  // 查詢轉換後的值
$arrUserInfoInsertData['class'] = $iSearchStudentClass;
$arrSemeInsertData[0]['seme_num'] = $bIsSeatNumberValid ? $arrOpenIDUserInfo['seat_no'] : 0;
```

校管/局管:

```php
$arrUserInfoInsertData = exportCreateUserAccountUserInfoByOpenIDUserInfo(
    $oOpenIDUserInfo, $sOrgCode, $sNewUserid, $sHashPassWord,
    getNowSemeYear(), false  // 第6參數 false = 不帶年班
);
$arrSemeInsertData = [];  // 不建立學期資料
```

### 步驟 4: 準備 user_status 資料

```php
$arrUserStatusInsertData = exportCreateUserStatusInfo(
    $sNewUserid,
    $iUserAccessLevel,
    date("Y-m-d", strtotime('+10 year'))  // 帳號有效期 10 年
);
```

### 步驟 5: Transaction 保護建立帳號

```php
$arrCreateAccountResult = createUserAccount(
    $arrUserInfoInsertData,      // user_info 資料
    $arrUserStatusInsertData,    // user_status 資料
    $arrSemeInsertData,          // 學期資料 (學生) 或空陣列 (管理員)
    $arrSubjectClassInfoList     // 科目資料 (空陣列)
);

if (!$arrCreateAccountResult['status']) {
    responseFailure($arrCreateAccountResult['message']);
}
```

### 步驟 6: 取得完整帳號資訊

```php
$arrLoginAccountInfo = getUserAccountInfo($sNewUserid);
```

---

## Transaction 保護機制

程式碼位置: `function_addUser.php:88-166`

```php
function createUserAccount($arrUserInfo, $arrStatusInfo, $arrSemeInfoList = [], $arrRelation = []) {
    global $dbh;

    try {
        $dbh->beginTransaction();

        // 1. INSERT user_info
        insertUserInfo($arrUserInfo);
        $iUID = $dbh->lastInsertId();

        // 2. INSERT user_status
        insertUserStatus($arrStatusInfo);

        // 3a. 學生: INSERT seme_student
        if (in_array($arrStatusInfo['access_level'], USER_STUDENT_GROUP)) {
            foreach ($arrSemeInfoList as $arrSemeInfo) {
                insertSemeStudent($arrSemeInfo);
            }
        }
        // 3b. 教師: REPLACE seme_teacher_subject (多筆)
        elseif (in_array($arrStatusInfo['access_level'], USER_TEACHER_GROUP)) {
            foreach ($arrSubjectMappingSNList as $iSubjectMappingSN) {
                foreach ($arrSemeInfoList as $arrSemeInfo) {
                    replaceSemeTeacherSubject(...);
                }
            }
        }

        $dbh->commit();  // 全部成功才提交
        $bResult = true;

    } catch (Exception $e) {
        $dbh->rollBack();  // 任何失敗都回滾
        $bResult = false;
        $sMessage = $e->getMessage();
    }

    return ['status' => $bResult, 'message' => $sMessage];
}
```

Transaction 確保:

- 資料一致性 (全有或全無)
- 避免產生「只有 user_info 沒有 user_status」的錯誤狀態
- 異常時自動回滾所有操作

---

## 建立帳號相關函式

定義於 `function_addUser.php`:

| 函式 | 行號 | 用途 |
|:-----|:-----|:-----|
| `getNextUserInfoAI()` | 25 | 取得 user_info AUTO_INCREMENT 值 |
| `isUserHasAcountByHashGuid()` | 49 | 以 hash_guid 檢查帳號是否存在 |
| `isUserHasAccountByClassInfo()` | 62 | 以年班姓名檢查帳號是否存在 |
| `createUserAccount()` | 88 | Transaction 建立帳號主函式 |
| `isCreateUserInfoValid()` | 168 | 驗證 user_info 資料完整性 |
| `insertUserInfo()` | 219 | INSERT user_info |
| `insertUserStatus()` | 300 | INSERT user_status |
| `insertSemeStudent()` | 348 | INSERT seme_student |
| `insertTeacherSubjectMapping()` | 399 | INSERT 教師科目對應 |
| `getSubjectMappingInfoBySubjectName()` | 425 | 查詢科目代碼 |
| `getTeacherSubjectList()` | 453 | 取得教師科目清單 |
| `replaceSemeTeacherSubject()` | 493 | REPLACE 教師任課資料 |

定義於 `function_OpenID.php`:

| 函式 | 行號 | 用途 |
|:-----|:-----|:-----|
| `exportCreateUserAccountUserInfoByOpenIDUserInfo()` | 630 | 匯出 user_info INSERT 資料 |
| `exportCreateUserStatusInfo()` | 656 | 匯出 user_status INSERT 資料 |
| `exportCreateSemeStudentInfo()` | 667 | 匯出 seme_student INSERT 資料 |
| `exportCreateSemeTeacherSubjectInfo()` | 686 | 匯出教師任課 INSERT 資料 |

## 相關文件

- [帳號查詢](./account-query.md) - 查無帳號時才觸發自動創建
- [資料同步](./data-sync.md) - 已有帳號時的同步機制
- [資料庫](./database.md) - user_info、user_status、seme_student 表結構
