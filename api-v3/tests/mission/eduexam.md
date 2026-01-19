# 考古題 (EduExam) 測試

> 對應 Postman: `ADLAPI/v3 > 任務 > 考古題 (未上架+未完成)`

## 狀態
- [ ] 已上架
- [ ] 測試案例已建立
- [ ] 測試通過

## API 清單

| API | 說明 | 測試狀態 |
|-----|------|---------|
| `POST /eduexam/types` | 取得考試類型 | ⬜ 待建立 |
| `POST /eduexam/papers` | 取得試卷列表 | ⬜ 待建立 |
| `POST /eduexam/detail` | 取得試卷詳情 | ⬜ 待建立 |
| `POST /eduexam/filters` | 取得篩選選項 | ⬜ 待建立 |

---

## POST /eduexam/types - 取得考試類型

### 案例 1: 正常請求

```bash
curl -X POST "{{BASE_URL}}/ADLAPI/v3/eduexam/types" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{}'
```

**預期結果:**
- Status: `200`
- Response 包含考試類型列表（會考、學測、指考等）

---

## POST /eduexam/papers - 取得試卷列表

### 案例 1: 正常請求

```bash
curl -X POST "{{BASE_URL}}/ADLAPI/v3/eduexam/papers" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "{{TEST_USER_ID}}"
  }'
```

**預期結果:**
- Status: `200`
- Response 包含試卷列表

---

### 案例 2: 篩選年度與類型

```bash
curl -X POST "{{BASE_URL}}/ADLAPI/v3/eduexam/papers" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "{{TEST_USER_ID}}",
    "year": 2024,
    "exam_type": "cap"
  }'
```

**預期結果:**
- Status: `200`
- Response 包含 2024 年會考試卷

---

## POST /eduexam/detail - 取得試卷詳情

### 案例 1: 正常請求

```bash
curl -X POST "{{BASE_URL}}/ADLAPI/v3/eduexam/detail" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "paper_id": "{{TEST_PAPER_ID}}"
  }'
```

**預期結果:**
- Status: `200`
- Response 包含試卷詳細內容及題目

---

## POST /eduexam/filters - 取得篩選選項

### 案例 1: 正常請求

```bash
curl -X POST "{{BASE_URL}}/ADLAPI/v3/eduexam/filters" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{}'
```

**預期結果:**
- Status: `200`
- Response 包含可用的篩選選項（年度、考試類型、科目等）
