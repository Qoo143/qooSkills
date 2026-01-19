# 學習紀錄 (LearningRecord) 測試

> 對應 Postman: `ADLAPI/v3 > 學習紀錄`

## 狀態
- [x] 已上架
- [ ] 測試案例已建立
- [ ] 測試通過

## API 清單

| API | 說明 | 測試狀態 |
|-----|------|---------|
| `POST /learning-record/personal` | 取得個人學習紀錄 | ⬜ 待建立 |

---

## POST /learning-record/personal - 取得個人學習紀錄

### 案例 1: 正常請求

```bash
curl -X POST "{{BASE_URL}}/ADLAPI/v3/learning-record/personal" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "{{TEST_USER_ID}}",
    "school_id": "{{TEST_SCHOOL_ID}}"
  }'
```

**預期結果:**
- Status: `200`
- Response 包含學習紀錄列表

---

### 案例 2: 指定日期範圍

```bash
curl -X POST "{{BASE_URL}}/ADLAPI/v3/learning-record/personal" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "{{TEST_USER_ID}}",
    "school_id": "{{TEST_SCHOOL_ID}}",
    "start_date": "2024-01-01",
    "end_date": "2024-12-31"
  }'
```

**預期結果:**
- Status: `200`
- Response 包含指定日期範圍內的學習紀錄

---

### 案例 3: 缺少必要參數

```bash
curl -X POST "{{BASE_URL}}/ADLAPI/v3/learning-record/personal" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{}'
```

**預期結果:**
- Status: `400`
- 錯誤訊息: user_id is required
