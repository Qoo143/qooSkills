# 使用者資訊 (UserData) 測試

> 對應 Postman: `ADLAPI/v3 > 使用者資訊`

## 狀態
- [x] 已上架
- [ ] 測試案例已建立
- [ ] 測試通過

## API 清單

| API | 說明 | 測試狀態 |
|-----|------|---------|
| `POST /user-data/load` | 取得使用者資料 | ⬜ 待建立 |

---

## POST /user-data/load - 取得使用者資料

### 案例 1: 正常請求

```bash
curl -X POST "{{BASE_URL}}/ADLAPI/v3/user-data/load" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "{{TEST_USER_ID}}"
  }'
```

**預期結果:**
- Status: `200`
- Response 包含使用者基本資料

---

### 案例 2: 無效 Token

```bash
curl -X POST "{{BASE_URL}}/ADLAPI/v3/user-data/load" \
  -H "Authorization: Bearer invalid_token" \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "{{TEST_USER_ID}}"
  }'
```

**預期結果:**
- Status: `401`
- 錯誤訊息: Unauthorized

---

### 案例 3: 缺少 user_id

```bash
curl -X POST "{{BASE_URL}}/ADLAPI/v3/user-data/load" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{}'
```

**預期結果:**
- Status: `400`
- 錯誤訊息: user_id is required
