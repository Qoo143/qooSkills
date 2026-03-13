### 需開發目標 php 列表

1. 任務儀表板 - modules_student.php
2. 星空圖 - chart.php
3. 排行榜 - rankingList.php - 跳轉回舊頁面不用寫api
4. 報告/測驗報告 - missionReport.php
5. 報告/學習紀錄 - video_personal_file.php
6. 討論/筆記 - learning_note.php
7. 討論/題問 - learning_ask_student.php
8. 討論/討論區 - eZDiscus教育雲功能不實作 - 跳轉回舊頁面不用寫api
9. 學習扶助/科技化評量 - query_prioriData.php
10. 學習扶助/縣市學力檢測 - query_prioriData_basicalAbility.php
11. 學習扶助/大考專區 - query_prioriData_new.php
----
12. 執行任務區域(知識結構的影片、練習題、動態練習、e度)
13. 其他任務執行區域，有些是外包可以直接跳轉(素養、單元等等要再確認)

### TODO
- [ ] 課程包任務同步 (mission_type=12) — 原始碼 `mission_action_general.php` L671-730，查清單時 curl 外部 API 更新完成狀態。建議改成獨立端點 `POST /student/missions/sync-course` 或等外包提供 Webhook

