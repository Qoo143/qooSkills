# 素養導向 (Literacy) 測試

> 對應 Postman: `ADLAPI/v3 > 任務 > 素養導向 (未上架+未完成)`

## 狀態
- [ ] 已上架
- [ ] 測試案例已建立
- [ ] 測試通過

## API 清單

| API | 說明 | 測試狀態 |
|-----|------|---------|
| `POST /literacy/list` | 取得素養題目列表 | ⬜ 待建立 |
| `POST /literacy/detail` | 取得題目詳情 | ⬜ 待建立 |
| `POST /literacy/filters` | 取得篩選選項 | ⬜ 待建立 |

---

## POST /literacy/list - 取得素養題目列表

### 案例 1: 正常請求

```bash
curl -X POST "{{BASE_URL}}/ADLAPI/v3/literacy/list" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "{{TEST_USER_ID}}"
  }'
```

**預期結果:**
- Status: `200`
- Response 包含素養題目列表

---

### 案例 2: 分頁與篩選

```bash
curl -X POST "{{BASE_URL}}/ADLAPI/v3/literacy/list" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "{{TEST_USER_ID}}",
    "page": 1,
    "page_size": 20,
    "subject": "math"
  }'
```

**預期結果:**
- Status: `200`
- Response 包含篩選後的題目列表

---

## POST /literacy/detail - 取得題目詳情

### 案例 1: 正常請求

```bash
curl -X POST "{{BASE_URL}}/ADLAPI/v3/literacy/detail" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "question_id": "{{TEST_LITERACY_ID}}"
  }'
```

**預期結果:**
- Status: `200`
- Response 包含題目詳細內容

---

## POST /literacy/filters - 取得篩選選項

### 案例 1: 正常請求

```bash
curl -X POST "{{BASE_URL}}/ADLAPI/v3/literacy/filters" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{}'
```

**預期結果:**
- Status: `200`
- Response 包含可用的篩選選項（科目、年級等）
