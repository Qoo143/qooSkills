# 公告 (Announcement) 測試

> 對應 Postman: `ADLAPI/v3 > 公告(未上架)`

## 狀態
- [ ] 已上架
- [ ] 測試案例已建立
- [ ] 測試通過

## API 清單

| API | 說明 | 測試狀態 |
|-----|------|---------|
| `POST /announcement/list` | 取得公告清單 | ⬜ 待建立 |

---

## POST /announcement/list - 取得公告清單

### 案例 1: 正常請求

```bash
curl -X POST "{{BASE_URL}}/ADLAPI/v3/announcement/list" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "{{TEST_USER_ID}}",
    "school_id": "{{TEST_SCHOOL_ID}}"
  }'
```

**預期結果:**
- Status: `200`
- Response 包含公告列表

---

### 案例 2: 分頁查詢

```bash
curl -X POST "{{BASE_URL}}/ADLAPI/v3/announcement/list" \
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
- Response 包含分頁資訊及公告列表

---

### 案例 3: 篩選特定類型公告

```bash
curl -X POST "{{BASE_URL}}/ADLAPI/v3/announcement/list" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "{{TEST_USER_ID}}",
    "school_id": "{{TEST_SCHOOL_ID}}",
    "type": "system"
  }'
```

**預期結果:**
- Status: `200`
- Response 只包含系統公告
