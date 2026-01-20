# 資源服務新增臺史博影片連結

## 任務資訊

| 項目 | 內容 |
|------|------|
| 日期 | 2026-01-20 |
| 需求 | 在操作手冊 > 教材介紹 > 資源服務，新增臺史博介紹影片連結 |
| 頁面 URL | https://adl.edu.tw/HomePage/webinfo/?id=4 |
| 影片連結 | https://youtu.be/SMEKB63O190?si=3ueeBlnmzLuKBqNL |
| 狀態 | ✅ 已分析 |

---

## 程式位置

### 資料來源

| 項目 | 說明 |
|------|------|
| API 檔案 | `HomePage/apis/webInfo/Get_WebInfo.php` |
| 資料表 | `WebInfo` |
| 查詢條件 | `id = 4` |

### 修改位置

內容儲存於資料庫 `WebInfo` 資料表的 HTML 欄位中。

**「資源服務」區塊位置**：約在 HTML 內容的第 1113-1173 行區域

現有項目（依序）：
1. 教育雲電子書
2. 國圖到你家
3. 藝術教育網
4. 本土數位教材專區
5. 高中自主學習網
6. Cool English
7. 臺灣客語辭典
8. 科宇宙悠遊學

---

## 修改內容

需在「科宇宙悠遊學」之後新增：

```html
<a
    href="https://youtu.be/SMEKB63O190?si=3ueeBlnmzLuKBqNL"
    class="list-group-item"
    target="_blank"
    rel="noreferrer noopenner"
    title="在新視窗打開鏈結">臺史博
</a>
```

---

## 修改方式

由於內容存於資料庫，需透過以下方式修改：
1. **資料庫直接修改**：UPDATE WebInfo 資料表 id=4 的 HTML 內容欄位
2. **後台管理介面**：若有 CMS 後台，可透過後台編輯

---

## 複製檔案參考

- 原始 HTML 內容備份：`webinfo-copy/0120-test.php`

---

## 完成確認

- [x] 程式位置已定位（資料庫 WebInfo 表）
- [x] 修改位置已確認（資源服務區塊，第 1113-1173 行）
- [x] 新增內容已準備（臺史博連結 HTML）
- [ ] 資料庫已更新
- [ ] 頁面已驗證
