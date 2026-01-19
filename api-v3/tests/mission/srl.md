# SRL表單 (Self-Regulated Learning) 測試

> 對應 Postman: `ADLAPI/v3 > 任務 > SRL表單 (未上架+未完成)`

## 狀態
- [ ] 已上架
- [ ] 測試案例已建立
- [ ] 測試通過

## API 清單

| API | 說明 | 測試狀態 |
|-----|------|---------|
| `POST /srl/calendar` | 取得行事曆事件 | ⬜ 待建立 |
| `POST /srl/notes` | 取得學習筆記列表 | ⬜ 待建立 |
| `POST /srl/note-detail` | 取得筆記詳情 | ⬜ 待建立 |
| `POST /srl/checklist` | 取得檢核表列表 | ⬜ 待建立 |
| `POST /srl/questions` | 取得學生提問列表 | ⬜ 待建立 |

---

## POST /srl/calendar - 取得行事曆事件

### 案例 1: 正常請求

```bash
curl -X POST "{{BASE_URL}}/ADLAPI/v3/srl/calendar" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "{{TEST_USER_ID}}",
    "start_date": "2024-01-01",
    "end_date": "2024-12-31"
  }'
```

**預期結果:**
- Status: `200`
- Response 包含指定日期範圍內的行事曆事件

---

## POST /srl/notes - 取得學習筆記列表

### 案例 1: 正常請求

```bash
curl -X POST "{{BASE_URL}}/ADLAPI/v3/srl/notes" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "{{TEST_USER_ID}}"
  }'
```

**預期結果:**
- Status: `200`
- Response 包含學習筆記列表

---

## POST /srl/note-detail - 取得筆記詳情

### 案例 1: 正常請求

```bash
curl -X POST "{{BASE_URL}}/ADLAPI/v3/srl/note-detail" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "note_id": "{{TEST_NOTE_ID}}"
  }'
```

**預期結果:**
- Status: `200`
- Response 包含筆記詳細內容

---

## POST /srl/checklist - 取得檢核表列表

### 案例 1: 正常請求

```bash
curl -X POST "{{BASE_URL}}/ADLAPI/v3/srl/checklist" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "{{TEST_USER_ID}}"
  }'
```

**預期結果:**
- Status: `200`
- Response 包含檢核表列表

---

## POST /srl/questions - 取得學生提問列表

### 案例 1: 正常請求

```bash
curl -X POST "{{BASE_URL}}/ADLAPI/v3/srl/questions" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "{{TEST_USER_ID}}"
  }'
```

**預期結果:**
- Status: `200`
- Response 包含學生提問列表
