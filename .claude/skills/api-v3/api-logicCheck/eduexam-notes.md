# EduExamController 檢查初步發現

## 新 API 端點（9個）
1. types - 考試類型列表
2. papers - 試卷列表
3. detail - 試卷詳情
4. filters - 篩選選項
5. start - 開始測驗
6. submit - 提交答案
7. finish - 交卷
8. result - 測驗結果
9. history - 測驗歷史

## 舊程式檔案發現
- `query_prioriData_new.php` - 主要是前端 UI (HTML/CSS/Vue)
- `answer.php` - 作答頁面 UI

## 待確認
需要找到舊程式的實際後端 API 邏輯（可能在 JS 或其他 PHP 檔案中）
