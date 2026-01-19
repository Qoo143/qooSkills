# 任務報告 (MissionReport) 測試

> 對應 Postman: `ADLAPI/v3 > 任務報告`

## 狀態
- [x] 已上架
- [ ] 測試案例已建立
- [ ] 測試通過

## API 清單

| API | 說明 | 測試狀態 |
|-----|------|---------|
| `POST /mission-report/list` | 學生個人任務報告清單 | ⬜ 待建立 |
| `POST /mission-report/detail` | 任務報告詳情 | ⬜ 待建立 |

---

## POST /mission-report/list - 學生個人任務報告清單

### 案例 1: 正常請求

```bash
curl -X POST "{{BASE_URL}}/ADLAPI/v3/mission-report/list" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "{{TEST_USER_ID}}",
    "school_id": "{{TEST_SCHOOL_ID}}"
  }'
```

**預期結果:**
- Status: `200`
- Response 包含任務報告列表

---

### 案例 2: 分頁查詢

```bash
curl -X POST "{{BASE_URL}}/ADLAPI/v3/mission-report/list" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "{{TEST_USER_ID}}",
    "school_id": "{{TEST_SCHOOL_ID}}",
    "page": 1,
    "page_size": 10
  }'
```

**預期結果:**
- Status: `200`
- Response 包含分頁資訊及任務列表

---

## POST /mission-report/detail - 任務報告詳情

### 案例 1: 正常請求

```bash
curl -X POST "{{BASE_URL}}/ADLAPI/v3/mission-report/detail" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "mission_id": "{{TEST_MISSION_ID}}",
    "user_id": "{{TEST_USER_ID}}"
  }'
```

**預期結果:**
- Status: `200`
- Response 包含任務詳細資訊

---

### 案例 2: 不存在的任務 ID

```bash
curl -X POST "{{BASE_URL}}/ADLAPI/v3/mission-report/detail" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "mission_id": "non_existent_id",
    "user_id": "{{TEST_USER_ID}}"
  }'
```

**預期結果:**
- Status: `404`
- 錯誤訊息: Mission not found
