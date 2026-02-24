# API Mapping: 學習扶助 (Remedy Test)

**建立日期**: 2026-02-11
**來源檔案**:
- modules/remedyTest/query_prioriData.php (科技化評量 - 學生端)
- modules/remedyTest/query_prioriData_basicalAbility.php (縣市學力檢測 - 學生端)
- modules/remedyTest/prodb_get_priori_data.php (教師端資料查詢)
- modules/remedyTest/print_prioriData_bNode.php (節點資料處理)

---

## 概述

學習扶助模組整合「科技化評量」和「縣市學力檢測」兩種測驗類型，使用相同的資料表結構，透過 `exam_type` 欄位區分。

### 測驗類型

| exam_type | 名稱 | 說明 |
|-----------|------|------|
| `''` (空字串) | 科技化評量 | 學習扶助診斷測驗 |
| `'A'` | 縣市學力檢測 | 縣市學力檢測 |
| `'B'` | 學力檢測考古題 | 考古題練習 |

---

## 資料表

### 主要資料表

| 資料表 | 說明 |
|--------|------|
| exam_record_priori | 學生測驗紀錄 |
| concept_priori | 測驗概念/範圍定義 |
| user_info | 使用者資訊 |
| subject | 科目對照 |

### 補救班級相關

| 資料表 | 說明 |
|--------|------|
| remedial_class | 補救班級 |
| remedial_student | 補救班級學生 |
| remedial_teacher | 補救班級教師 |

---

## Controller: RemedyTestController

### 學生端端點

| 端點 | 方法 | Action 對照 | 說明 |
|------|------|-------------|------|
| POST /remedy-test/filters | filters() | - | 取得篩選選項（日期、科目） |
| POST /remedy-test/records | records() | - | 取得測驗紀錄列表 |
| POST /remedy-test/result | result() | - | 取得測驗結果（節點精熟狀態） |
| POST /remedy-test/self-mission | selfMission() | assignSelfMission | 學生自派診斷任務 |

### 教師端端點

| 端點 | 方法 | Action 對照 | 說明 |
|------|------|-------------|------|
| POST /remedy-test/students | students() | find_student | 查詢補救學生列表 |
| POST /remedy-test/grades | grades() | find_grade | 取得年級列表 |
| POST /remedy-test/classes | classes() | find_class | 取得班級列表 |
| POST /remedy-test/teachers | teachers() | find_teacher | 取得教師列表 |
| POST /remedy-test/subjects | subjects() | find_subject | 取得科目列表 |
| POST /remedy-test/dates | dates() | find_date | 取得測驗日期列表 |

---

## 端點詳細規格

### POST /remedy-test/filters

取得學生的測驗篩選選項。

**Request**
```json
{
  "exam_type": ""
}
```

| 參數 | 類型 | 必填 | 說明 |
|------|------|------|------|
| exam_type | string | 選填 | 測驗類型（空字串/A/B） |

**Response**
```json
{
  "status": "success",
  "data": {
    "exam_ranges": [
      {
        "exam_range": "112年4月",
        "subjects": [
          {
            "subject_id": 1,
            "subject_name": "國語文",
            "import_date": "2024-04-15"
          }
        ]
      }
    ]
  }
}
```

---

### POST /remedy-test/records

取得學生的測驗紀錄。

**Request**
```json
{
  "exam_type": "",
  "exam_range": "112年4月",
  "subject_id": 1
}
```

| 參數 | 類型 | 必填 | 說明 |
|------|------|------|------|
| exam_type | string | 選填 | 測驗類型 |
| exam_range | string | 選填 | 測驗範圍/日期 |
| subject_id | int | 選填 | 科目 ID |

**Response**
```json
{
  "status": "success",
  "data": {
    "records": [
      {
        "exam_sn": 12345,
        "cp_id": "CP001",
        "subject_id": 1,
        "subject_name": "國語文",
        "exam_range": "112年4月",
        "date": "2024-04-15"
      }
    ]
  }
}
```

---

### POST /remedy-test/result

取得測驗結果（節點精熟狀態）。

**Request**
```json
{
  "exam_sn": 12345,
  "cp_id": "CP001"
}
```

| 參數 | 類型 | 必填 | 說明 |
|------|------|------|------|
| exam_sn | int | 必填 | 測驗 SN |
| cp_id | string | 必填 | 概念 ID |

**Response**
```json
{
  "status": "success",
  "data": {
    "nodes": [
      {
        "node_id": "N-1-1",
        "node_name": "整數的運算",
        "status": "pass",
        "exam_result": "O",
        "adl_status": 2
      }
    ],
    "legend": {
      "O": "所有試題通過",
      "X": "所有試題未通過",
      "△": "部分試題未通過",
      "N": "尚未有測驗資料"
    }
  }
}
```

---

### POST /remedy-test/self-mission

學生自派診斷任務。

**Request**
```json
{
  "mission_nm": "進階診斷 - 整數運算",
  "node": "N-1-1",
  "map_sn": 3,
  "self_exam_type": 1
}
```

| 參數 | 類型 | 必填 | 說明 |
|------|------|------|------|
| mission_nm | string | 必填 | 任務名稱 |
| node | string | 必填 | 節點代號或試卷 SN |
| map_sn | int | 必填 | 地圖 SN |
| self_exam_type | int | 選填 | 自我測驗類型（0=全測, 1=適性省題） |

**Response**
```json
{
  "status": "success",
  "data": {
    "mission_sn": 67890,
    "paper_sn": null
  }
}
```

---

### POST /remedy-test/students (教師端)

查詢補救學生列表。

**Request**
```json
{
  "exam_type": "",
  "exam_range": "112年4月",
  "subject_id": 1,
  "grade": 3,
  "class": 1
}
```

**Response**
```json
{
  "status": "success",
  "data": {
    "students": [
      {
        "user_id": "STU001",
        "uname": "王小明",
        "grade": 3,
        "class": 1,
        "class_full_name": "三年一班",
        "cp_id": "CP001",
        "exam_range": "112年4月"
      }
    ]
  }
}
```

---

## DAO: RemedyTestDao

### 方法列表

| 方法 | 說明 |
|------|------|
| getExamFilters($userId, $examType) | 取得篩選選項 |
| getExamRecords($userId, $params) | 取得測驗紀錄 |
| getExamResult($userId, $examSn, $cpId) | 取得測驗結果 |
| getStudentsByFilter($params) | 查詢補救學生 |
| getGradesByFilter($params) | 取得年級列表 |
| getClassesByFilter($params) | 取得班級列表 |
| getTeachersByOrg($orgId) | 取得教師列表 |
| getSubjectsByOrg($orgId) | 取得科目列表 |
| getDatesByFilter($params) | 取得測驗日期 |
| createSelfMission($userId, $data) | 建立自派任務 |

---

## Service: RemedyTestService

### 方法列表

| 方法 | 說明 |
|------|------|
| getFilters($userId, $examType) | 取得篩選選項 |
| getRecords($userId, $params) | 取得測驗紀錄 |
| getResult($userId, $examSn, $cpId) | 取得測驗結果 |
| getStudents($teacherId, $params) | 查詢補救學生 |
| getGrades($teacherId, $params) | 取得年級列表 |
| getClasses($teacherId, $params) | 取得班級列表 |
| getTeachers($teacherId) | 取得教師列表 |
| getSubjects($teacherId) | 取得科目列表 |
| getDates($teacherId, $params) | 取得測驗日期 |
| createSelfMission($userId, $data) | 建立自派任務 |

---

## 注意事項

1. **exam_type 區分**：
   - 空字串 `''` = 科技化評量
   - `'A'` = 縣市學力檢測
   - `'B'` = 學力檢測考古題

2. **節點狀態圖例**：
   - `O` = 所有試題通過
   - `X` = 所有試題未通過
   - `△` = 部分試題未通過
   - `N` = 尚未有測驗資料

3. **因材網指標狀態**：
   - `-1` = 未測驗
   - `0` = 待補救
   - `2` = 精熟

4. **自派任務類型**：
   - mission_type = 4: 國語/英語單元卷
   - mission_type = 5: 數學縱貫

---

## 舊版 Action 對照

| 舊 Action | 新端點 |
|-----------|--------|
| query_prioriData.php | /remedy-test/filters + /remedy-test/records + /remedy-test/result |
| query_prioriData_basicalAbility.php | 同上（exam_type='A'） |
| find_student | /remedy-test/students |
| find_grade | /remedy-test/grades |
| find_class | /remedy-test/classes |
| find_teacher | /remedy-test/teachers |
| find_subject | /remedy-test/subjects |
| find_date | /remedy-test/dates |
| assignSelfMission | /remedy-test/self-mission |
