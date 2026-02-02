# API 路由表

| 模組 | API | 說明 |
|------|-----|------|
| UserData | `POST /user-data/load` | 取得使用者資料 |
| Mission | `POST /mission/types` | 取得任務類型清單 |
| Mission | `POST /mission/list` | 查詢任務清單 |
| Mission | `POST /mission/detail` | 查詢單一任務詳情 |
| LearningRecord | `POST /learning-record/personal` | 取得個人學習紀錄 |
| Announcement | `POST /announcement/list` | 取得公告清單 |
| Todo | `POST /todo/list` | 查詢待辦事項清單 |
| Todo | `POST /todo/detail` | 查詢單筆待辦事項 |
| Todo | `POST /todo/create` | 新增待辦事項 |
| Todo | `POST /todo/update` | 更新待辦事項 |
| Todo | `POST /todo/delete` | 刪除待辦事項 (軟刪除) |
| Knowledge | `POST /knowledge/node` | 取得單一節點詳情及學習資源 |
| Knowledge | `POST /knowledge/unit` | 取得單元節點列表 |
| Knowledge | `POST /knowledge/list` | 取得科目群組出版商及單元列表 |
| Knowledge | `POST /knowledge/related-nodes` | 取得上下位節點 |
| KsLearning | `POST /ks-learning/practice` | 取得練習題題目 |
| KsLearning | `POST /ks-learning/practice/submit` | 提交練習題答案 |
| KsLearning | `POST /ks-learning/assessment` | 取得動態評量題目 |
| KsLearning | `POST /ks-learning/assessment/submit` | 提交動態評量答案 |
| KsLearning | `POST /ks-learning/video` | 開始觀看影片 |
| KsLearning | `POST /ks-learning/video/record` | 更新影片觀看紀錄 |
| KsLearning | `POST /ks-learning/checkpoint` | 取得檢核點題目 |
| KsLearning | `POST /ks-learning/checkpoint/submit` | 提交檢核點答案 |
| Literacy | `POST /literacy/list` | 取得素養題目列表 |
| Literacy | `POST /literacy/detail` | 取得題目詳情 |
| Literacy | `POST /literacy/filters` | 取得篩選選項 |
| EduExam | `POST /eduexam/types` | 取得考試類型 |
| EduExam | `POST /eduexam/papers` | 取得試卷列表 |
| EduExam | `POST /eduexam/detail` | 取得試卷詳情 |
| EduExam | `POST /eduexam/filters` | 取得篩選選項 |
| Questionnaire | `POST /questionnaire/types` | 取得問卷類型列表 |
| Questionnaire | `POST /questionnaire/detail` | 取得問卷詳情 |
| Questionnaire | `POST /questionnaire/status` | 檢查問卷填寫狀態 |
| Questionnaire | `POST /questionnaire/history` | 取得問卷填寫歷史 |
| SRL | `POST /srl/calendar` | 取得行事曆事件 |
| SRL | `POST /srl/notes` | 取得學習筆記列表 |
| SRL | `POST /srl/note-detail` | 取得筆記詳情 |
| SRL | `POST /srl/checklist` | 取得檢核表列表 |
| SRL | `POST /srl/questions` | 取得學生提問列表 |
| MissionReport | `POST /mission-report/list` | 學生個人任務報告清單 |
| MissionReport | `POST /mission-report/detail` | 任務報告詳情 |
| Mission | `POST /mission/group-report` | 小組進度報告 |
| PersonalConfig | `POST /personal-config/get` | 取得個人設定 |
| PersonalConfig | `PUT /personal-config/update` | 更新個人設定 |
