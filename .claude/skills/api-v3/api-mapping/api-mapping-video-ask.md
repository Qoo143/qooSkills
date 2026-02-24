# API Mapping: 學習提問 (Video Ask)

**建立日期**: 2026-02-11
**來源檔案**: modules/srl/prodb_learning_ask.php

---

## 概述

學習提問模組允許學生在觀看影片時提出問題，教師和其他學生可以回覆。支援小組功能，可以在小組內進行討論。

---

## 資料表

| 資料表 | 說明 |
|--------|------|
| video_noteask | 提問主表 |
| video_noteask_plus | 提問回覆 |
| video_concept_item | 影片概念關聯 |
| seme_group | 小組資訊 |
| map_node | 知識節點 |
| map_info | 知識地圖 |
| user_info | 使用者資訊 |

---

## Controller: VideoAskController

### 端點列表

| 端點 | 方法 | Action 對照 | 角色 | 說明 |
|------|------|-------------|------|------|
| POST /video-ask/list | list() | getQuestions4Student/getQuestions4Teacher | 學生/教師 | 取得提問列表 |
| POST /video-ask/detail | detail() | - | 全部 | 提問詳情（含回覆） |
| POST /video-ask/create | create() | - | 學生/教師 | 建立提問 |
| POST /video-ask/update | update() | - | 提問擁有者 | 更新提問 |
| POST /video-ask/delete | delete() | - | 提問擁有者 | 刪除提問 |
| POST /video-ask/reply | reply() | - | 全部 | 回覆提問 |
| POST /video-ask/reply/update | replyUpdate() | - | 回覆擁有者 | 更新回覆 |
| POST /video-ask/reply/delete | replyDelete() | - | 回覆擁有者 | 刪除回覆 |
| POST /video-ask/toggle-solved | toggleSolved() | - | 提問擁有者/教師 | 標記已解決 |

---

## 端點詳細規格

### POST /video-ask/list

取得提問列表（我的提問、學生提問、小組提問）。

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
| type | string | 選填 | 類型：my(我的)/class(班級)/group(小組) |

**Response**
```json
{
  "status": "success",
  "data": {
    "my_questions": [...],
    "class_questions": [...],
    "group_questions": [...]
  }
}
```

**提問物件結構**
```json
{
  "ask_sn": 123,
  "user_id": "STU001",
  "uname": "王小明",
  "class_full_name": "三年一班",
  "access_level": 1,
  "question_content": "這題為什麼是這樣解？",
  "stop_time": 125,
  "create_time": "2024-03-15 14:30",
  "indicator": "N-1-1",
  "indicate_name": "整數的運算",
  "subject_id": 2,
  "subject_name": "數學",
  "reply_time": "2024-03-15 15:00",
  "reply_count": 3,
  "is_solved": 0,
  "group_id": null,
  "group_nm": null,
  "file_path": null
}
```

---

### POST /video-ask/detail

取得提問詳情（含所有回覆）。

**Request**
```json
{
  "ask_sn": 123
}
```

**Response**
```json
{
  "status": "success",
  "data": {
    "question": {
      "ask_sn": 123,
      "user_id": "STU001",
      "uname": "王小明",
      "question_content": "這題為什麼是這樣解？",
      "stop_time": 125,
      "create_time": "2024-03-15 14:30",
      "indicator": "N-1-1",
      "is_solved": 0
    },
    "replies": [
      {
        "reply_sn": 456,
        "user_id": "TCH001",
        "uname": "李老師",
        "access_level": 21,
        "reply_content": "這題的關鍵是...",
        "reply_time": "2024-03-15 15:00"
      }
    ]
  }
}
```

---

### POST /video-ask/create

建立提問。

**Request**
```json
{
  "video_item_sn": 789,
  "stop_time": 125,
  "question_content": "這題為什麼是這樣解？",
  "group_id": null,
  "file_path": null
}
```

| 參數 | 類型 | 必填 | 說明 |
|------|------|------|------|
| video_item_sn | int | 必填 | 影片項目 SN |
| stop_time | int | 選填 | 影片暫停時間點（秒） |
| question_content | string | 必填 | 提問內容 |
| group_id | int | 選填 | 小組 ID |
| file_path | string | 選填 | 附件路徑 |

**Response**
```json
{
  "status": "success",
  "data": {
    "ask_sn": 124
  }
}
```

---

### POST /video-ask/reply

回覆提問。

**Request**
```json
{
  "ask_sn": 123,
  "reply_content": "這題的關鍵是..."
}
```

| 參數 | 類型 | 必填 | 說明 |
|------|------|------|------|
| ask_sn | int | 必填 | 提問 SN |
| reply_content | string | 必填 | 回覆內容 |

**Response**
```json
{
  "status": "success",
  "data": {
    "reply_sn": 457,
    "reply_time": "2024-03-15 16:00"
  }
}
```

---

### POST /video-ask/toggle-solved

標記提問為已解決或未解決。

**Request**
```json
{
  "ask_sn": 123
}
```

**Response**
```json
{
  "status": "success",
  "data": {
    "is_solved": true
  }
}
```

---

## DAO: VideoAskDao

### 方法列表

| 方法 | 說明 |
|------|------|
| getMyQuestions($userId, $startDate, $endDate) | 取得我的提問 |
| getClassQuestions($teacherId, $semester, $startDate, $endDate) | 取得班級學生提問 |
| getGroupQuestions($userId, $groupId, $startDate, $endDate) | 取得小組提問 |
| getQuestionById($askSn) | 取得提問詳情 |
| getReplies($askSn) | 取得回覆列表 |
| createQuestion($data) | 建立提問 |
| updateQuestion($askSn, $data) | 更新提問 |
| deleteQuestion($askSn) | 刪除提問 (is_show = 0) |
| createReply($askSn, $userId, $content) | 建立回覆 |
| updateReply($replySn, $content) | 更新回覆 |
| deleteReply($replySn) | 刪除回覆 (is_show = 0) |
| toggleSolved($askSn, $isSolved) | 切換已解決狀態 |

---

## Service: VideoAskService

### 方法列表

| 方法 | 說明 |
|------|------|
| getList($userId, $isTeacher, $params) | 取得提問列表 |
| getDetail($askSn, $userId) | 取得提問詳情 |
| create($userId, $data) | 建立提問 |
| update($askSn, $userId, $data) | 更新提問 |
| delete($askSn, $userId) | 刪除提問 |
| reply($askSn, $userId, $content) | 回覆提問 |
| updateReply($replySn, $userId, $content) | 更新回覆 |
| deleteReply($replySn, $userId) | 刪除回覆 |
| toggleSolved($askSn, $userId, $isTeacher) | 切換已解決 |

---

## 舊版 Action 對照

| 舊 Action | 新端點 | 說明 |
|-----------|--------|------|
| getQuestions4Student | /video-ask/list | 學生提問列表 |
| getQuestions4Teacher | /video-ask/list | 教師查看提問 |
| saveSession | - | 已整合到其他端點 |

---

## 注意事項

1. **顯示狀態 (is_show)**：
   - `0` = 已刪除/隱藏
   - `1` = 顯示中

2. **已解決狀態 (is_solved)**：
   - `0` = 未解決
   - `1` = 已解決

3. **小組功能**：
   - 提問可以指定 `group_id` 關聯到 `seme_group`
   - 小組內的成員可以看到和回覆該提問

4. **影片概念關聯**：
   - 提問透過 `video_item_sn` 關聯到 `video_concept_item`
   - 再透過 `indicator` + `map_sn` 關聯到 `map_node`

5. **檔案上傳**：
   - 提問支援上傳圖片附件
   - 檔案路徑存於 `file_path`，伺服器位置存於 `upload_server`

6. **年級班級格式**：
   - `grade_class` 欄位格式為 `{grade}_{class}`
   - 例如：`3_1` 表示三年一班
