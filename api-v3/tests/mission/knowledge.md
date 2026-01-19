# 知識結構 (Knowledge & KsLearning) 測試

> 對應 Postman: `ADLAPI/v3 > 任務 > 知識結構 (未上架+未完成)`

## 狀態
- [ ] 已上架
- [ ] 測試案例已建立
- [ ] 測試通過

## API 清單

### Knowledge - 知識結構

| API | 說明 | 測試狀態 |
|-----|------|---------|
| `POST /knowledge/node` | 取得單一節點詳情及學習資源 | ⬜ 待建立 |
| `POST /knowledge/unit` | 取得單元節點列表 | ⬜ 待建立 |
| `POST /knowledge/list` | 取得科目群組出版商及單元列表 | ⬜ 待建立 |
| `POST /knowledge/related-nodes` | 取得上下位節點 | ⬜ 待建立 |

### KsLearning - 知識點學習

| API | 說明 | 測試狀態 |
|-----|------|---------|
| `POST /ks-learning/practice` | 取得練習題題目 | ⬜ 待建立 |
| `POST /ks-learning/practice/submit` | 提交練習題答案 | ⬜ 待建立 |
| `POST /ks-learning/assessment` | 取得動態評量題目 | ⬜ 待建立 |
| `POST /ks-learning/assessment/submit` | 提交動態評量答案 | ⬜ 待建立 |
| `POST /ks-learning/video` | 開始觀看影片 | ⬜ 待建立 |
| `POST /ks-learning/video/record` | 更新影片觀看紀錄 | ⬜ 待建立 |
| `POST /ks-learning/checkpoint` | 取得檢核點題目 | ⬜ 待建立 |
| `POST /ks-learning/checkpoint/submit` | 提交檢核點答案 | ⬜ 待建立 |

---

# Knowledge API

## POST /knowledge/node - 取得單一節點詳情及學習資源

### 案例 1: 正常請求

```bash
curl -X POST "{{BASE_URL}}/ADLAPI/v3/knowledge/node" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "node_id": "{{TEST_NODE_ID}}"
  }'
```

**預期結果:**
- Status: `200`
- Response 包含節點詳情及相關學習資源

---

## POST /knowledge/unit - 取得單元節點列表

### 案例 1: 正常請求

```bash
curl -X POST "{{BASE_URL}}/ADLAPI/v3/knowledge/unit" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "unit_id": "{{TEST_UNIT_ID}}"
  }'
```

**預期結果:**
- Status: `200`
- Response 包含單元下的知識節點列表

---

## POST /knowledge/list - 取得科目群組出版商及單元列表

### 案例 1: 正常請求

```bash
curl -X POST "{{BASE_URL}}/ADLAPI/v3/knowledge/list" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "subject": "math",
    "grade": 7
  }'
```

**預期結果:**
- Status: `200`
- Response 包含科目的出版商及單元列表

---

## POST /knowledge/related-nodes - 取得上下位節點

### 案例 1: 正常請求

```bash
curl -X POST "{{BASE_URL}}/ADLAPI/v3/knowledge/related-nodes" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "node_id": "{{TEST_NODE_ID}}"
  }'
```

**預期結果:**
- Status: `200`
- Response 包含先備節點及延伸節點

---

# KsLearning API

## POST /ks-learning/practice - 取得練習題題目

### 案例 1: 正常請求

```bash
curl -X POST "{{BASE_URL}}/ADLAPI/v3/ks-learning/practice" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "node_id": "{{TEST_NODE_ID}}",
    "user_id": "{{TEST_USER_ID}}"
  }'
```

**預期結果:**
- Status: `200`
- Response 包含練習題題目

---

## POST /ks-learning/practice/submit - 提交練習題答案

### 案例 1: 正常提交

```bash
curl -X POST "{{BASE_URL}}/ADLAPI/v3/ks-learning/practice/submit" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "practice_id": "{{TEST_PRACTICE_ID}}",
    "user_id": "{{TEST_USER_ID}}",
    "answers": [
      {"question_id": "q1", "answer": "A"},
      {"question_id": "q2", "answer": "B"}
    ]
  }'
```

**預期結果:**
- Status: `200`
- Response 包含批改結果

---

## POST /ks-learning/video - 開始觀看影片

### 案例 1: 正常請求

```bash
curl -X POST "{{BASE_URL}}/ADLAPI/v3/ks-learning/video" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "video_id": "{{TEST_VIDEO_ID}}",
    "user_id": "{{TEST_USER_ID}}"
  }'
```

**預期結果:**
- Status: `200`
- Response 包含影片資訊及播放 URL

---

## POST /ks-learning/video/record - 更新影片觀看紀錄

### 案例 1: 正常請求

```bash
curl -X POST "{{BASE_URL}}/ADLAPI/v3/ks-learning/video/record" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "video_id": "{{TEST_VIDEO_ID}}",
    "user_id": "{{TEST_USER_ID}}",
    "watch_duration": 120,
    "is_completed": false
  }'
```

**預期結果:**
- Status: `200`
- 觀看紀錄已更新

---

## POST /ks-learning/assessment - 取得動態評量題目

### 案例 1: 正常請求

```bash
curl -X POST "{{BASE_URL}}/ADLAPI/v3/ks-learning/assessment" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "node_id": "{{TEST_NODE_ID}}",
    "user_id": "{{TEST_USER_ID}}"
  }'
```

**預期結果:**
- Status: `200`
- Response 包含動態評量題目

---

## POST /ks-learning/checkpoint - 取得檢核點題目

### 案例 1: 正常請求

```bash
curl -X POST "{{BASE_URL}}/ADLAPI/v3/ks-learning/checkpoint" \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "video_id": "{{TEST_VIDEO_ID}}",
    "checkpoint_time": 60
  }'
```

**預期結果:**
- Status: `200`
- Response 包含檢核點題目
