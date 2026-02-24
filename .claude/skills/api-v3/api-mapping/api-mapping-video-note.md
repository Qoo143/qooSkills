# API Mapping: 學習筆記 (Video Note)

**建立日期**: 2026-02-11
**來源檔案**: modules/srl/prodb_learning_note.php

---

## 概述

學習筆記模組允許學生在觀看影片時記錄筆記，並支援按讚、收藏、回饋、推薦等社交互動功能。教師可以查看班級學生的筆記並給予代幣獎勵。

---

## 資料表

| 資料表 | 說明 |
|--------|------|
| video_note | 筆記主表 |
| video_note_favorite | 按讚/收藏紀錄 |
| video_note_feedback | 回饋紀錄 |
| video_note_recommend | 教師推薦紀錄 |
| video_concept_item | 影片概念關聯 |
| role_coin_history | 代幣紀錄 |
| map_node | 知識節點 |
| map_info | 知識地圖 |

---

## Controller: VideoNoteController

### 端點列表

| 端點 | 方法 | Action 對照 | 角色 | 說明 |
|------|------|-------------|------|------|
| POST /video-note/list | list() | getNotes4Student/getNotes4Teacher | 學生/教師 | 取得筆記列表 |
| POST /video-note/detail | detail() | - | 全部 | 筆記詳情 |
| POST /video-note/create | create() | - | 學生 | 建立筆記 |
| POST /video-note/update | update() | - | 筆記擁有者 | 更新筆記 |
| POST /video-note/delete | delete() | deleteNotes | 筆記擁有者 | 刪除筆記 |
| POST /video-note/toggle-like | toggleLike() | updateLike | 全部 | 按讚/取消 |
| POST /video-note/toggle-favorite | toggleFavorite() | updateFavorite | 全部 | 收藏/取消 |
| POST /video-note/toggle-display | toggleDisplay() | displayNote | 筆記擁有者 | 公開/隱藏 |
| POST /video-note/feedback/list | feedbackList() | searchFeedback | 全部 | 回饋列表 |
| POST /video-note/feedback/create | feedbackCreate() | sentFeedback | 全部 | 寫入回饋 |
| POST /video-note/feedback/delete | feedbackDelete() | deleteFeedback | 回饋擁有者 | 刪除回饋 |
| POST /video-note/recommend | recommend() | recommendNotes | 教師 | 推薦筆記 |
| POST /video-note/give-coins | giveCoins() | giveCoins | 教師 | 給予代幣 |
| POST /video-note/recommended-list | recommendedList() | getRecommendNotes | 全部 | 取得推薦筆記 |

---

## 端點詳細規格

### POST /video-note/list

取得筆記列表（我的筆記、我的收藏、班級筆記）。

**Request**
```json
{
  "start_time": "2024-01-01",
  "end_time": "2024-12-31",
  "type": "my"
}
```

| 參數 | 類型 | 必填 | 說明 |
|------|------|------|------|
| start_time | date | 必填 | 開始日期 |
| end_time | date | 必填 | 結束日期 |
| type | string | 選填 | 類型：my(我的)/favorite(收藏)/class(班級) |

**Response**
```json
{
  "status": "success",
  "data": {
    "my_notes": [...],
    "favorite_notes": [...],
    "class_notes": [...]
  }
}
```

**筆記物件結構**
```json
{
  "video_note_sn": 123,
  "user_id": "STU001",
  "uname": "王小明",
  "class_full_name": "三年一班",
  "access_level": 1,
  "content": "筆記內容...",
  "stop_time": 125,
  "is_public": 1,
  "create_time": "2024-03-15 14:30",
  "indicator": "N-1-1",
  "indicate_name": "整數的運算",
  "subject_id": 2,
  "count_recommend": 3,
  "count_like": 15,
  "count_favorite": 8,
  "count_feedback": 2,
  "is_like": 1,
  "is_favorite": 0,
  "has_coin": 0
}
```

---

### POST /video-note/toggle-like

按讚或取消按讚。

**Request**
```json
{
  "video_note_sn": 123
}
```

**Response**
```json
{
  "status": "success",
  "data": {
    "is_like": true,
    "count_like": 16
  }
}
```

---

### POST /video-note/toggle-favorite

收藏或取消收藏。

**Request**
```json
{
  "video_note_sn": 123
}
```

**Response**
```json
{
  "status": "success",
  "data": {
    "is_favorite": true,
    "count_favorite": 9
  }
}
```

---

### POST /video-note/feedback/list

取得筆記回饋列表。

**Request**
```json
{
  "video_note_sn": 123
}
```

**Response**
```json
{
  "status": "success",
  "data": {
    "feedbacks": [
      {
        "feedback_sn": 456,
        "user_id": "TCH001",
        "uname": "李老師",
        "content": "寫得很好！",
        "create_time": "2024-03-16 09:00"
      }
    ]
  }
}
```

---

### POST /video-note/recommend

教師推薦筆記。

**Request**
```json
{
  "video_note_sn": 123
}
```

**Response**
```json
{
  "status": "success",
  "data": {
    "is_recommended": true,
    "count_recommend": 4
  }
}
```

---

### POST /video-note/give-coins

教師給予學生代幣。

**Request**
```json
{
  "video_note_sn": 123,
  "coin_type": 1,
  "amount": 5
}
```

| 參數 | 類型 | 必填 | 說明 |
|------|------|------|------|
| video_note_sn | int | 必填 | 筆記 SN |
| coin_type | int | 選填 | 代幣類型 (預設 1) |
| amount | int | 選填 | 代幣數量 (預設 1) |

**Response**
```json
{
  "status": "success",
  "data": {
    "message": "已給予 5 個代幣"
  }
}
```

---

## DAO: VideoNoteDao

### 方法列表

| 方法 | 說明 |
|------|------|
| getMyNotes($userId, $startDate, $endDate) | 取得我的筆記 |
| getMyFavorites($userId, $startDate, $endDate) | 取得我的收藏 |
| getClassNotes($teacherId, $semester, $startDate, $endDate) | 取得班級筆記 |
| getNoteById($noteSn) | 取得筆記詳情 |
| createNote($data) | 建立筆記 |
| updateNote($noteSn, $data) | 更新筆記 |
| deleteNote($noteSn) | 刪除筆記 |
| toggleLike($noteSn, $userId) | 切換按讚狀態 |
| toggleFavorite($noteSn, $userId) | 切換收藏狀態 |
| toggleDisplay($noteSn, $isPublic) | 切換公開狀態 |
| getFeedbacks($noteSn) | 取得回饋列表 |
| createFeedback($noteSn, $userId, $content) | 建立回饋 |
| deleteFeedback($feedbackSn) | 刪除回饋 |
| addRecommend($noteSn, $teacherId) | 新增推薦 |
| removeRecommend($noteSn, $teacherId) | 移除推薦 |
| giveCoins($noteSn, $teacherId, $coinType, $amount) | 給予代幣 |
| getRecommendedNotes($videoItemSn) | 取得推薦筆記 |

---

## Service: VideoNoteService

### 方法列表

| 方法 | 說明 |
|------|------|
| getList($userId, $isTeacher, $params) | 取得筆記列表 |
| getDetail($noteSn, $userId) | 取得筆記詳情 |
| create($userId, $data) | 建立筆記 |
| update($noteSn, $userId, $data) | 更新筆記 |
| delete($noteSn, $userId) | 刪除筆記 |
| toggleLike($noteSn, $userId) | 切換按讚 |
| toggleFavorite($noteSn, $userId) | 切換收藏 |
| toggleDisplay($noteSn, $userId) | 切換公開 |
| getFeedbackList($noteSn) | 取得回饋列表 |
| createFeedback($noteSn, $userId, $content) | 建立回饋 |
| deleteFeedback($feedbackSn, $userId) | 刪除回饋 |
| recommend($noteSn, $teacherId) | 推薦筆記 |
| giveCoins($noteSn, $teacherId, $coinType, $amount) | 給予代幣 |
| getRecommendedList($videoItemSn) | 取得推薦列表 |

---

## 舊版 Action 對照

| 舊 Action | 新端點 | 說明 |
|-----------|--------|------|
| getNotes4Student | /video-note/list | 學生筆記列表 |
| getNotes4Teacher | /video-note/list | 教師筆記列表 |
| updateLike | /video-note/toggle-like | 按讚切換 |
| updateFavorite | /video-note/toggle-favorite | 收藏切換 |
| searchFeedback | /video-note/feedback/list | 回饋列表 |
| sentFeedback | /video-note/feedback/create | 新增回饋 |
| deleteFeedback | /video-note/feedback/delete | 刪除回饋 |
| displayNote | /video-note/toggle-display | 公開切換 |
| giveCoins | /video-note/give-coins | 給予代幣 |
| recommendNotes | /video-note/recommend | 推薦筆記 |
| deleteNotes | /video-note/delete | 刪除筆記 |
| getTeacherRecommend | /video-note/recommended-list | 推薦筆記列表 |
| getRecommendNotes | /video-note/recommended-list | 推薦筆記列表 |

---

## 注意事項

1. **公開狀態 (is_public)**：
   - `0` = 私密（只有本人可見）
   - `1` = 公開（班級可見）

2. **影片概念關聯**：
   - 筆記透過 `video_item_sn` 關聯到 `video_concept_item`
   - 再透過 `indicator` + `map_sn` 關聯到 `map_node`

3. **代幣系統**：
   - 教師給代幣會記錄到 `role_coin_history`
   - 每個教師對同一筆記只能給一次代幣
