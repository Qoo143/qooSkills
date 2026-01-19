# 問卷 (Questionnaire) 測試

> 對應 Postman: `ADLAPI/v3 > 任務 > 問卷 (未上架+未完成)`

## 狀態
- [ ] 已上架
- [ ] 測試案例已建立
- [ ] 測試通過

## API 清單

| API | 說明 | 測試狀態 |
|-----|------|---------|
| `POST /questionnaire/types` | 取得問卷類型列表 | ⬜ 待建立 |
| `POST /questionnaire/detail` | 取得問卷詳情 | ⬜ 待建立 |
| `POST /questionnaire/status` | 檢查問卷填寫狀態 | ⬜ 待建立 |
| `POST /questionnaire/history` | 取得問卷填寫歷史 | ⬜ 待建立 |

---

## POST /questionnaire/types - 取得問卷類型列表

### 案例 1: 正常請求

```bash
curl -X POST "{{BASE_URL}}/ADLAPI/v3/questionnaire/types" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{}'
```

**預期結果:**
- Status: `200`
- Response 包含問卷類型列表

---

## POST /questionnaire/detail - 取得問卷詳情

### 案例 1: 正常請求

```bash
curl -X POST "{{BASE_URL}}/ADLAPI/v3/questionnaire/detail" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "questionnaire_id": "{{TEST_QUESTIONNAIRE_ID}}"
  }'
```

**預期結果:**
- Status: `200`
- Response 包含問卷詳細內容及題目

---

## POST /questionnaire/status - 檢查問卷填寫狀態

### 案例 1: 正常請求

```bash
curl -X POST "{{BASE_URL}}/ADLAPI/v3/questionnaire/status" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "questionnaire_id": "{{TEST_QUESTIONNAIRE_ID}}",
    "user_id": "{{TEST_USER_ID}}"
  }'
```

**預期結果:**
- Status: `200`
- Response 包含填寫狀態（已完成/未完成/進行中）

---

## POST /questionnaire/history - 取得問卷填寫歷史

### 案例 1: 正常請求

```bash
curl -X POST "{{BASE_URL}}/ADLAPI/v3/questionnaire/history" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "{{TEST_USER_ID}}"
  }'
```

**預期結果:**
- Status: `200`
- Response 包含使用者的問卷填寫歷史紀錄
