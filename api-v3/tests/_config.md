# 測試環境配置

執行 cURL 測試時，請將 `{{變數名}}` 替換為以下對應的值。

## 環境變數

### 伺服器設定

| 變數 | 值 | 說明 |
|------|---|------|
| `{{BASE_URL}}` | `https://adl-aial.edu.tw` | API 伺服器位址 |

### 測試用使用者

| 變數 | 值 | 說明 |
|------|---|------|
| `{{TEST_USER_ID}}` | `190041-s0903001` | 測試學生帳號 |
| `{{TEST_SCHOOL_ID}}` | `123456` | 測試學校代碼 |
| `{{TEST_CLASS_ID}}` | `class001` | 測試班級 ID |
| `{{TEST_TEACHER_ID}}` | `test_teacher_001` | 測試教師帳號 |

### 測試用資料 ID

| 變數 | 值 | 說明 |
|------|---|------|
| `{{TEST_MISSION_ID}}` | `` | 測試任務 ID |
| `{{TEST_NODE_ID}}` | `` | 測試知識節點 ID |
| `{{TEST_VIDEO_ID}}` | `` | 測試影片 ID |
| `{{TEST_PAPER_ID}}` | `` | 測試試卷 ID |

---

## Token 認證（自動取得）

執行 API 測試前，AI 助手會自動執行以下步驟取得 Token：

### Step 1: 取得 Token

```bash
curl -X POST "{{BASE_URL}}/ADLAPI/v2/token" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "data={{AUTH_DATA}}"
```

### AUTH_DATA 編碼值

> ⚠️ 此值已包含 user_id、clientid、clientsecret 等資訊，經 base64 編碼

```
{{AUTH_DATA}} = MWMzODNjZDMwYjdjMjk4YWI1MDI5M2FkZmVjYjdiMThFZFVBZExZVG8wT250ek9qYzZJblZ6WlhKZmFXUWlPM002TVRRNklqRTVNREEwTVMxek1Ea3dNekF4SWp0ek9qZzZJbUpsYUdGMmFXOXlJanRPTzNNNk9Eb2lZMnhwWlc1MGFXUWlPM002TXpJNklrUXljVnBJY1doVk5FUnJkVkoxVUhCWWREaDZaamh0UmsxcVJVRXljVXM1SWp0ek9qRXlPaUpqYkdsbGJuUnpaV055WlhRaU8zTTZOalE2SW1WNmJscFpjM2hMZERRNWVuaHVNR1J3VTFoR2VWZHhhekpYYjFWWVNtaEZWRzE0VEVoRU5XeDBaamQzTmxSbWFHcE9XbEJNT1hCSVlrdHBhRVp6Y1dZaU8zMD1FZFVBZEw5RTY3REU1QTgzQkFGQTUwNkVGNTQ0NzMzQTQyMUE1OA===9E67DE5A83BAFA506EF544733A421A58==
```

### Step 2: 解析回應取得 accesstoken

回應範例：
```json
{
  "status": "OK",
  "accesstoken": "a1b2c3d4e5f6..."
}
```

### Step 3: 使用 token 執行 API 測試

取得的 `accesstoken` 會自動帶入後續 API 請求的 body 中。

---

## 測試執行流程

當您說「幫我測試 xxx API」時，AI 助手會：

1. **自動取得 Token** - 執行 Step 1 的 cURL
2. **解析 accesstoken** - 從回應中提取 token
3. **執行測試 API** - 將 token 帶入請求
4. **回報結果** - 顯示狀態碼和回應內容

---

## 環境切換

### 測試環境 (預設)
```
BASE_URL = https://adl-aial.edu.tw
```

### 正式環境
```
BASE_URL = https://adl.edu.tw
```

> ⚠️ **注意**: 請勿在正式環境執行破壞性測試（如 DELETE、UPDATE）

---

## 手動取得 Token（備用）

如果自動取得失敗，可手動：

1. 使用瀏覽器登入因材網
2. 開啟開發者工具 (F12)
3. 在 Network 中找到任一 API 請求
4. 複製 Request Body 中的 `accesstoken` 值
