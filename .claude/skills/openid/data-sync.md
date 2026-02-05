# 資料同步機制

## 快速登入 vs 一般登入的同步差異

資料同步邏輯在快速登入和一般登入中基本相同,但有以下關鍵差異:

| 項目 | 快速登入 (QUICK_LOGIN) | 一般登入 (LOGIN_SELECTED_IDENTITY) |
|:-----|:----------------------|:----------------------------------|
| 執行位置 | prodb_choiceIdentity.php:349-460 | prodb_choiceIdentity.php:960-1031 |
| 同步邏輯 | 相同 | 相同 |
| 失敗處理 | `error_log` --> 繼續登入 | `responseFailure()` --> 阻止登入 |
| 設計理由 | openid_data 已驗證一致,優先使用者體驗 | 資料可能不一致,嚴格檢查 |

快速登入同步的定位:

- 快速登入的同步是補救機制
- 正常狀況下,一般登入的同步會被更常使用
- 因為有變化 openid_data 也會變,觸發降級到一般登入
- 快速登入同步只是彌補「一般登入同步失敗,但 openid_data 已寫入更新」的狀況

---

## 同步觸發條件

### 快速登入同步條件

程式碼位置: `prodb_choiceIdentity.php:350-359`

基本條件:

```php
$bBasicSyncCondition = $bOpenIdTrust && $bIsClassInfoValid;
```

| 條件層級 | 條件 | 檢查方式 | 適用身分 | 不滿足時 |
|:---------|:-----|:---------|:---------|:---------|
| 外層 | 信任學校 | `isOpenIDTrust($sOrgCode)` | 學生、教師 | 不同步 |
| 外層 | 年班有效 | `empty(array_intersect(INVALID_CLASSINFO_ERROR_LIST, ...))` | 學生、教師 | 不同步 |
| 學生內層 | OpenID 學期=當前學期 | `$sOpenIDYearSemester == $sNowYearSemester` | 僅學生 | 不同步 |
| 教師內層 | 任課資料非空 | `!empty($arrClassInfoList)` | 僅教師 | 跳過同步 |
| 教師內層 | 本校任課非空 | `!empty($arrSemeInsertData)` | 僅教師 | 跳過同步 |

### 一般登入同步條件 (3 個條件)

程式碼位置: `prodb_choiceIdentity.php:960`

```php
if ($bOpenIdTrust && $bIsSelectedNowSemester && !$bIsAutoCreateAccount) {
    // 進入同步邏輯,內部會再檢查 $bIsClassInfoValid
}
```

| 條件 | 檢查方式 | 不滿足時 |
|:-----|:---------|:---------|
| 信任學校 | `isOpenIDTrust($sOrgCode)` | 不同步 |
| 當前學期 | `$sSlectedYearSemester == $sNowYearSemester` | 不同步 |
| 非自動創建 | `!$bIsAutoCreateAccount` | 不同步 (新帳號已使用 OpenID 資料) |
| (內部) 年班有效 | `$bIsClassInfoValid` | 不執行同步函式 |

---

## 學生資料同步

### 快速登入

程式碼位置: `prodb_choiceIdentity.php:352-400`

```php
if ($bBasicSyncCondition && isStudentRole($sUserLoginTitle)) {
    // 學期驗證:防止 OpenID 回傳非當前學期資料
    $sOpenIDYearSemester = $arrMentorClassInfo['year'] .
                          substr($arrMentorClassInfo['semester'], -1);
    $bIsOpenIDCurrentSemester = $sOpenIDYearSemester == $sNowYearSemester;

    if ($bIsOpenIDCurrentSemester) {
        // 格式轉換:非高中需將中文轉數字
        if (!isHighSchool($sOrgCode)) {
            $iSearchStudentGrade = transformClassName2Int($arrMentorClassInfo['grade']);
            $iSearchStudentClass = transformClassName2Int($arrMentorClassInfo['class']);
        }

        try {
            if ($bIsSeatNumberValid) {
                updateStudentSemesterInfo(...);      // 含座號
            } else {
                updateStudentSemesterInfoWithoutSeatno(...);  // 不含座號
            }
        } catch (Exception $e) {
            // 快速登入:同步失敗只記錄,不阻止登入
            error_log("Quick Login - Student sync failed: " . $e->getMessage());
        }
    }
}
```

### 一般登入

程式碼位置: `prodb_choiceIdentity.php:962-973`

```php
if (isStudentRole($sUserLoginTitle)) {
    if ($bIsClassInfoValid) {
        if ($bIsSeatNumberValid) {
            if (!updateStudentSemesterInfo(...)) {
                // 一般登入:同步失敗阻止登入
                responseFailure("學生年班資料與OpenID資料比對有差異,自動更新年班失敗,請聯絡因材網客服");
            }
        } else {
            if (!updateStudentSemesterInfoWithoutSeatno(...)) {
                responseFailure("學生年班資料與OpenID資料比對有差異,自動更新年班失敗,請聯絡因材網客服");
            }
        }
    }
}
```

### 同步函式說明

定義於 `function_user_info.php`:

`updateStudentSemesterInfo()` (行 761-780):
- 同時更新 `user_info` 的 grade/class 和 `seme_student` 的年班座號
- 使用 `replaceSemeStudent()` (REPLACE INTO) 確保紀錄存在

`updateStudentSemesterInfoWithoutSeatno()` (行 782-805):
- 同時更新 `user_info` 的 grade/class 和 `seme_student` 的年班
- 不更新座號欄位
- 使用 `updateSemeStudent()` (UPDATE) 只更新年班

### 學期驗證的重要性

```
問題情境:暑假期間

時間:2024/8/20 (暑假)
OpenID 學期:1131 (113學年第1學期) <-- 下學期
因材網學期:1122 (112學年第2學期) <-- 當前學期

如果不驗證:
  OpenID 的「四年級」會覆蓋因材網的「三年級」
  --> 學生還在三年級暑假,但系統顯示四年級

正確做法:
  比對學期 --> 不同 --> 不同步
  --> 保護當前學期資料不被覆蓋
```

---

## 教師資料同步

教師資料同步使用 Relation API 的 `$arrRelation['relation']` 資料。

### 快速登入

程式碼位置: `prodb_choiceIdentity.php:404-458`

```php
if ($bBasicSyncCondition && isRoleInTeacherGroup($sUserLoginTitle)) {
    $arrClassInfoList = $arrRelation['relation'];  // Relation API 資料

    if (!empty($arrClassInfoList)) {
        // 過濾出本校任課班級
        $arrSemeInsertData = [];
        foreach ($arrClassInfoList as $arrClassInfo) {
            if (strval($arrClassInfo['schoolid']) == $sOrgCode) {
                $sOidcSemester = $arrClassInfo['year'] .
                                substr($arrClassInfo['semester'], -1);
                $arrSemeInsertData[] = [
                    'seme_year_seme' => $sOidcSemester,
                    'organization_id' => $arrClassInfo['schoolid'],
                    'teacher_id' => $arrLoginAccountInfo['user_id'],
                    'grade' => $arrClassInfo['grade'],
                    'class' => $arrClassInfo['classno']
                ];
            }
        }

        if (!empty($arrSemeInsertData)) {
            try {
                $dbh->beginTransaction();

                $arrSubjectMappingSNList = getTeacherSubjectList(
                    $arrRelation['relation'],
                    $arrLoginAccountInfo['organization_id']
                );

                // 雙層迴圈:科目 x 班級
                foreach ($arrSubjectMappingSNList as $iSubjectMappingSN) {
                    foreach ($arrSemeInsertData as $arrSemeInfo) {
                        replaceSemeTeacherSubject($arrTeacherSubjectUserInfo, $iSubjectMappingSN, ...);
                    }
                }

                $dbh->commit();
            } catch (Exception $e) {
                $dbh->rollback();
                // 快速登入:同步失敗只記錄,不阻止登入
                error_log("Quick Login - Teacher sync failed: " . $e->getMessage());
            }
        }
    }
}
```

### 一般登入

程式碼位置: `prodb_choiceIdentity.php:974-1029`

邏輯與快速登入相同,差異在錯誤處理:

```php
try {
    $dbh->beginTransaction();
    // ... 相同的同步邏輯 ...
    $bReplaceSubjectResult = true;
} catch (Exception $e) {
    $bReplaceSubjectResult = false;
    $sRelaceSubjectErrorMessage = $e->getMessage();
} finally {
    if ($bReplaceSubjectResult) {
        $dbh->commit();
    } else {
        $dbh->rollback();
        // 一般登入:同步失敗阻止登入
        responseFailure($sRelaceSubjectErrorMessage);
    }
}
```

### 教師同步為何需要 Transaction

```
範例:王老師任教 5 個班級 x 2 個科目 = 10 筆記錄

不用交易的風險:
  第 1 次 REPLACE:成功
  第 2 次 REPLACE:成功
  第 3 次 REPLACE:成功
  第 4 次 REPLACE:失敗  <-- 網路中斷
  第 5-10 次:未執行

  結果:只更新了 3/10,資料不一致

使用交易:
  全部成功 --> commit (10/10)
  任何失敗 --> rollback (0/10)
```

---

## openid_data 寫入時機

在一般登入的年班比對前更新 `openid_data`:

```php
// prodb_choiceIdentity.php:936-949
$sOpenIDData = json_encode($arrOpenIDClassList);
try {
    $update = $dbh->prepare(
        "UPDATE user_info SET openid_data = :openid_data WHERE user_id = :user_id"
    );
    $update->bindValue(':openid_data', $sOpenIDData, PDO::PARAM_STR);
    $update->bindValue(':user_id', $arrLoginAccountInfo['user_id'], PDO::PARAM_STR);
    $update->execute();
} catch (PDOException $e) {
    error_log("PDO Error: " . $e->getMessage());
}
```

此操作在同步之前執行,因此:
- 即使同步失敗,openid_data 已更新
- 下次登入時快速登入的 openid_data 比對會通過
- 快速登入的同步成為補救機制

---

## 非信任學校年班不一致處理

程式碼位置: `prodb_choiceIdentity.php:951-957`

```php
$bIsClassInfoDiffrence = ($bIsSelectedNowSemester &&
    ($arrLoginAccountInfo['grade'] != $iSearchStudentGrade ||
     $arrLoginAccountInfo['class'] != $iSearchStudentClass));

if (!$bOpenIdTrust && $bIsClassInfoDiffrence && isStudentRole($sUserLoginTitle)) {
    setSessionModifiedClassOpenIDUserID($arrLoginAccountInfo['user_id']);
    setSessionOpenIDLoginMode(LOGIN_MODE_MODIFIED_CLASSINFO);
    echo json_encode(['status' => RESPONSE_CLASSINFO_COMPARE_DIFFRENT]);
    die();
}
```

- 僅影響非信任學校的學生
- 年班不一致時不自動同步,而是要求使用者手動確認
- 前端顯示年班選擇介面,使用者確認後觸發 `SET_USER_CLASS_INFO`

---

## 補綁識別碼

同步完成後,會補綁缺少的識別碼:

```php
// prodb_choiceIdentity.php:1033-1039
if ($bRequireUpdatedHashGuid) {
    updateUserGuid($arrLoginAccountInfo['user_id'], $arrLoginUserInfo['guid']);
}

if ($bRequireUpdatedOpenIDSub) {
    bindSub($arrLoginAccountInfo, $arrLoginUserInfo['sub']);
}
```

補綁發生在帳號查詢時以其他方式找到帳號的情況:
- 用 OpenID_sub 找到 --> 需補綁 hash_guid
- 用 hash_guid 找到 --> 需補綁 OpenID_sub
- 用年班姓名找到 --> 兩者都需補綁

## 相關文件

- [登入流程](./login-flow.md) - 同步在流程中的位置
- [帳號查詢](./account-query.md) - 查詢結果決定是否同步
- [錯誤處理](./error-handling.md) - 同步失敗的處理方式
