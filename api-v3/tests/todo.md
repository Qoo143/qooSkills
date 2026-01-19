# 代辦事項 (Todo) 測試

> 對應 Postman: `ADLAPI/v3 > 代辦事項(未上架)`

## 狀態
- [ ] 已上架
- [ ] 測試案例已建立
- [ ] 測試通過

## API 清單

| API | 說明 | 測試狀態 |
|-----|------|---------|
| `POST /todo/list` | 查詢待辦事項清單 | ⬜ 待建立 |
| `POST /todo/detail` | 查詢單筆待辦事項 | ⬜ 待建立 |
| `POST /todo/create` | 新增待辦事項 | ⬜ 待建立 |
| `POST /todo/update` | 更新待辦事項 | ⬜ 待建立 |
| `POST /todo/delete` | 刪除待辦事項 (軟刪除) | ⬜ 待建立 |

---

## POST /todo/list - 查詢待辦事項清單

### 案例 1: 正常請求

```bash
curl -X POST "{{BASE_URL}}/ADLAPI/v3/todo/list" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "{{TEST_USER_ID}}"
  }'
```

**預期結果:**
- Status: `200`
- Response 包含待辦事項列表

---

## POST /todo/detail - 查詢單筆待辦事項

### 案例 1: 正常請求

```bash
curl -X POST "{{BASE_URL}}/ADLAPI/v3/todo/detail" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "todo_id": "{{TEST_TODO_ID}}"
  }'
```

**預期結果:**
- Status: `200`
- Response 包含待辦事項詳細資訊

---

## POST /todo/create - 新增待辦事項

### 案例 1: 正常新增

```bash
curl -X POST "{{BASE_URL}}/ADLAPI/v3/todo/create" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "{{TEST_USER_ID}}",
    "title": "測試待辦事項",
    "description": "這是一個測試",
    "due_date": "2024-12-31"
  }'
```

**預期結果:**
- Status: `201`
- Response 包含新建立的待辦事項 ID

---

## POST /todo/update - 更新待辦事項

### 案例 1: 正常更新

```bash
curl -X POST "{{BASE_URL}}/ADLAPI/v3/todo/update" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "todo_id": "{{TEST_TODO_ID}}",
    "title": "更新後的標題",
    "is_completed": true
  }'
```

**預期結果:**
- Status: `200`
- Response 包含更新後的待辦事項

---

## POST /todo/delete - 刪除待辦事項

### 案例 1: 正常刪除 (軟刪除)

```bash
curl -X POST "{{BASE_URL}}/ADLAPI/v3/todo/delete" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "todo_id": "{{TEST_TODO_ID}}"
  }'
```

**預期結果:**
- Status: `200`
- 待辦事項被標記為已刪除
