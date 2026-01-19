# 任務共用 (Mission Common) 測試

> 對應 Postman: `ADLAPI/v3 > 任務 (未上架+未完成)`

## 狀態
- [ ] 已上架
- [ ] 測試案例已建立
- [ ] 測試通過

## 說明

此檔案包含任務模組的共用 API，這些 API 適用於所有任務類型（素養導向、考古題、問卷、SRL表單、知識結構）。

## API 清單

| API | 說明 | 測試狀態 |
|-----|------|---------|
| `POST /mission/types` | 取得任務類型清單 | ⬜ 待建立 |
| `POST /mission/list` | 查詢任務清單 | ⬜ 待建立 |
| `POST /mission/detail` | 查詢單一任務詳情 | ⬜ 待建立 |

---

## POST /mission/types - 取得任務類型清單

### 案例 1: 正常請求

```bash
curl -X POST "{{BASE_URL}}/ADLAPI/v3/mission/types" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{}'
```

**預期結果:**
- Status: `200`
- Response 包含任務類型列表:
  - 素養導向
  - 考古題
  - 問卷
  - SRL表單
  - 知識結構

---

## POST /mission/list - 查詢任務清單

### 案例 1: 查詢所有任務

```bash
curl -X POST "{{BASE_URL}}/ADLAPI/v3/mission/list" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "{{TEST_USER_ID}}",
    "school_id": "{{TEST_SCHOOL_ID}}"
  }'
```

**預期結果:**
- Status: `200`
- Response 包含所有任務列表

---

### 案例 2: 篩選特定類型任務

```bash
curl -X POST "{{BASE_URL}}/ADLAPI/v3/mission/list" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "{{TEST_USER_ID}}",
    "school_id": "{{TEST_SCHOOL_ID}}",
    "type": "literacy"
  }'
```

**預期結果:**
- Status: `200`
- Response 只包含素養導向類型的任務

---

### 案例 3: 篩選任務狀態

```bash
curl -X POST "{{BASE_URL}}/ADLAPI/v3/mission/list" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "{{TEST_USER_ID}}",
    "school_id": "{{TEST_SCHOOL_ID}}",
    "status": "in_progress"
  }'
```

**預期結果:**
- Status: `200`
- Response 只包含進行中的任務

---

## POST /mission/detail - 查詢單一任務詳情

### 案例 1: 正常請求

```bash
curl -X POST "{{BASE_URL}}/ADLAPI/v3/mission/detail" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "mission_id": "{{TEST_MISSION_ID}}"
  }'
```

**預期結果:**
- Status: `200`
- Response 包含任務詳細資訊

---

### 案例 2: 不存在的任務

```bash
curl -X POST "{{BASE_URL}}/ADLAPI/v3/mission/detail" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "mission_id": "non_existent_id"
  }'
```

**預期結果:**
- Status: `404`
- 錯誤訊息: Mission not found
