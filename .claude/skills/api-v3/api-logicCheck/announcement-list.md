# API 邏輯檢查: Announcement - List

## 基本資訊

| 項目 | 內容 |
|------|------|
| **檢查日期** | 2026-02-11 |
| **檢查者** | AI Assistant |
| **API 端點** | `POST /announcement/list` |
| **舊程式檔案** | `modulesNews_data.js` + `modules/srl/calender_todolist.php` |
| **Controller** | `AnnouncementController::list()` |
| **Service** | `AnnouncementService::getAnnouncements()` |
| **DAO** | `UserDao::getUserStatus()` |

---

## 1. SQL 查詢比對

### 舊程式 (UserDao.php L38-46)
```sql
SELECT access_level 
FROM user_status 
WHERE user_id = :user_id
```

### 新 API (UserDao.php L38-46)
```sql
SELECT access_level 
FROM user_status 
WHERE user_id = :user_id
```

**✅ SQL 完全一致**

---

## 2. 參數驗證比對

### 舊程式
- **calender_todolist.php (L3-16)**:
  - 通過 `$_SESSION['user_data']` 取得 `access_level`
  - 通過 if-else 判斷身分:
    - `USER_TEACHER_GROUP` → `'teacher'`
    - `USER_SCHOOL_ADMIN_GROUP` → `'principal'`
    - `USER_STUDENT_GROUP` → `'student'`
    - `USER_PARENTS_GROUP` → `'parent'`
    - `USER_SCHOOL_ADMIN` → `'schoolAdmin'`

### 新 API
- **AnnouncementController (L24-26)**:
  ```php
  $this->guard(['method' => 'POST', 'token' => true]);
  $userId = $this->request->getUserId();
  ```
- **AnnouncementService::getUserIdentity() (L62-72)**:
  ```php
  $userStatus = $this->userDao->getUserStatus($userId);
  $accessLevel = intval($userStatus['access_level']);
  return UserHelper::getIdentityByAccessLevel($accessLevel);
  ```

**✅ 邏輯一致** - 都通過 `access_level` 判斷身分,新 API 使用 Helper 方法封裝

---

## 3. 業務邏輯流程比對

### 舊程式流程
1. **身分判斷** (calender_todolist.php L3-16)
2. **取得公告資料** (modulesNews_data.js L18-903)
   - 資料結構: `newsData[identity][]`
   - 欄位: `startdate`, `enddate`, `icon`, `newsTitle`, `newsDetail`
3. **篩選有效期間公告** (calender_todolist.php L163-237)
   - `gatherNewsData(userIdentity)` 函數
   - 包含 `userIdentity` 和 `'all'` 兩類
   - 依據 `startdate` 和 `enddate` 過濾
4. **排序** (calender_todolist.php L167-169)
   - 按 `startdate` 倒序 (最新在前)
5. **顯示** (calender_todolist.php L192)
   - append 到 `.newsListWrap`

### 新 API 流程
1. **身分判斷** (AnnouncementService L22, L62-72)
   ```php
   $identity = $this->getUserIdentity($userId);
   ```
2. **取得公告資料** (AnnouncementService L24, L74-131)
   ```php
   $allNews = $this->getNewsData();
   ```
   - 資料結構完全相同於 modulesNews_data.js
3. **篩選有效期間公告** (AnnouncementService L28-51)
   ```php
   $targetIdentities = [$identity, 'all'];
   foreach ($targetIdentities as $id) {
       foreach ($allNews[$id] as $news) {
           $startDate = strtotime($news['startdate']);
           $endDate = strtotime($news['enddate']);
           if ($now >= $startDate && $now <= $endDate) {
               $result[] = [...]
           }
       }
   }
   ```
4. **排序** (AnnouncementService L53-56)
   ```php
   usort($result, function ($a, $b) {
       return strtotime($b['start_date']) - strtotime($a['start_date']);
   });
   ```
5. **返回 JSON** (AnnouncementController L28)

**✅ 邏輯完全一致**

---

## 4. 回應格式比對

### 舊程式
- **前端 JS 動態生成 HTML** (calender_todolist.php L192)
  ```javascript
  $(".newsListWrap").append('<div class="newsList" ...>' + assembleData[i].newsTitle + ...)
  ```

### 新 API
- **JSON 格式**
  ```json
  {
    "success": true,
    "message": "Announcements loaded successfully",
    "data": {
      "announcements": [
        {
          "start_date": "2024/08/01",
          "end_date": "2025/12/01",
          "icon": "",
          "title": "規律用眼3010",
          "content": "<p><b>...</b></p>"
        }
      ]
    }
  }
  ```

**📝 差異說明**:
- 舊程式: 伺服器端渲染 HTML
- 新 API: 返回結構化 JSON 數據
- **欄位對應**:
  - `newsTitle` → `title`
  - `newsDetail` → `content`
  - `startdate` → `start_date`
  - `enddate` → `end_date`

---

## 5. 錯誤處理比對

### 舊程式
- **無明確錯誤處理**
- 如果 `user_status` 查詢失敗或沒有資料,默認為 `'student'` (calender_todolist.php L66-67)

### 新 API
- **getUserIdentity() 有預設值處理** (AnnouncementService L66-68)
  ```php
  if ($userStatus === null || !isset($userStatus['access_level'])) {
      return 'student';
  }
  ```
- **Controller 層有 guard 驗證** (AnnouncementController L24)
  - 驗證 HTTP method 和 token

**✅ 新 API 錯誤處理更完善**

---

## 6. 重複邏輯識別

### 發現的重複邏輯

#### 6.1 **身分判斷邏輯**
- **位置**:
  - `calender_todolist.php` L3-16
  - `AnnouncementService::getUserIdentity()` L62-72
- **建議**: 已通過 `UserHelper::getIdentityByAccessLevel()` 統一處理 ✅

#### 6.2 **公告資料維護**
- **位置**:
  - `modulesNews_data.js` (前端 903 行)
  - `AnnouncementService::getNewsData()` (後端 L74-131)
- **問題**: 公告資料在前端和後端各維護一份
- **建議**: 
  - ⚠️ **需要建立共用資料來源**
  - 方案 1: 將公告資料移到資料庫 `news` 表
  - 方案 2: 建立單一配置檔,前後端共同讀取
  - 方案 3: 後端提供 API,前端調用後端 API

#### 6.3 **時間驗證邏輯**
- **位置**:
  - `calender_todolist.php` L174-190 (前端)
  - `AnnouncementService::getAnnouncements()` L37-49 (後端)
- **建議**: 已在後端統一處理,前端僅負責顯示 ✅

---

## 7. 改進建議

### 7.1 資料來源統一 (🔴 高優先級)
**問題**: 公告資料在 `modulesNews_data.js` 和 `AnnouncementService` 中各維護一份

**建議方案**:
```php
// 建議: 將公告資料移到資料庫
CREATE TABLE `system_announcements` (
  `id` INT PRIMARY KEY AUTO_INCREMENT,
  `target_identity` VARCHAR(20),  -- teacher/student/parent/all
  `start_date` DATETIME,
  `end_date` DATETIME,
  `icon` VARCHAR(10),
  `title` VARCHAR(255),
  `content` TEXT,
  `priority` INT DEFAULT 0,
  `created_at` TIMESTAMP
);

// 對應 DAO 方法
class AnnouncementDao {
    public function getActiveAnnouncements($identity, $now) {
        $sql = "SELECT * FROM system_announcements
                WHERE (target_identity = :identity OR target_identity = 'all')
                  AND start_date <= :now
                  AND end_date >= :now
                ORDER BY start_date DESC";
    }
}
```

**優點**:
- 公告資料統一管理,避免重複維護
- 可通過管理介面動態新增/修改公告
- 無需重新部署程式碼

### 7.2 Helper 方法提取 (⚪ 中優先級)
**位置**: `AnnouncementService::getUserIdentity()` L62-72

**建議**: 此方法已提取為通用 Helper ✅

### 7.3 快取機制 (⚪ 低優先級)
**問題**: 每次請求都要重新篩選公告

**建議**:
```php
// 在 AnnouncementService 中加入快取
public function getAnnouncements($userId) {
    $identity = $this->getUserIdentity($userId);
    $cacheKey = "announcements:{$identity}:" . date('Ymd');
    
    // 嘗試從快取取得
    $cached = $this->cache->get($cacheKey);
    if ($cached !== null) {
        return $cached;
    }
    
    // 原有邏輯...
    
    // 儲存快取 (1小時)
    $this->cache->set($cacheKey, $result, 3600);
    return $result;
}
```

### 7.4 回應格式標準化 (⚪ 低優先級)
**建議**: 欄位命名統一使用 snake_case,與資料庫命名一致 ✅ (已實現)

---

## 8. 檢查結論

| 檢查項目 | 狀態 |
|---------|------|
| SQL 查詢一致性 | ✅ 完全一致 |
| 參數驗證 | ✅ 一致,新 API 更完善 |
| 業務邏輯流程 | ✅ 完全一致 |
| 回應格式 | 📝 前端 HTML vs 後端 JSON (符合預期) |
| 錯誤處理 | ✅ 新 API 更完善 |
| 重複邏輯處理 | ⚠️ 公告資料雙重維護需改進 |

### 整體評估: **✅ 邏輯一致,有改進空間**

### 關鍵差異
1. **舊程式**: 伺服器端渲染 HTML
2. **新 API**: 返回 JSON,前後端分離

### 必須處理的問題
1. 🔴 **公告資料雙重維護** - 建議移到資料庫統一管理

### 可選的優化
1. ⚪ 加入快取機制
2. ⚪ 提取時間驗證為共用 Helper

---

**檢查完成時間**: 2026-02-11 16:10:00
