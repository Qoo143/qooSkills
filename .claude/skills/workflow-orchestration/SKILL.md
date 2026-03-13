---
name: workflow-orchestration
description: 【必讀】所有開發任務的核心工作流程與原則。任何涉及寫程式、修 bug、code review、新增功能、重構、檔案編輯的任務都必須先載入此 skill，並檢閱 tasks/lessons.md 中的經驗教訓避免重複犯錯。
---

# 工作流編排 (Workflow Orchestration)

## 1. 預設規劃模式 (Plan Node Default)

- 針對任何非瑣碎任務（3 個步驟以上或涉及架構決策），必須進入規劃模式
- 如果執行過程出錯，立即停止並重新規劃——不要強行推進
- 規劃模式不僅用於構建，也應處理驗證步驟
- 事先撰寫詳細的規格說明（Specs），以減少歧義

## 2. 子代理策略 (Subagent Strategy)

- 靈活使用子代理（Subagents），以保持主上下文視窗（Main Context Window）乾淨
- 將研究、探索和並行分析任務交由子代理處理
- 對於複雜問題，透過子代理投入更多運算資源
- 每個子代理專注於單一任務，以確保執行精準

## 3. 自我完善循環 (Self-Improvement Loop)

- 在收到使用者的任何糾正後：立即將模式更新至 tasks/lessons.md
- 為自己撰寫規則，以防止重複犯錯
- 無情地迭代這些經驗教訓，直到錯誤率下降
- 在專案階段開始時，複習相關課程經驗

## 4. 完成前驗證 (Verification Before Done)

- 在證明其有效之前，絕不將任務標記為完成
- 必要時，比對原本行為與修改後行為的差異（Diff）
- 問自己：「一位**資深主任工程師（Staff Engineer）**會批准這個改動嗎？」
- 執行測試、檢查日誌，並展示其正確性

## 5. 追求優雅（平衡） (Demand Elegance - Balanced)

- 針對非瑣碎的更動：停下來思考「是否有更優雅的方式？」
- 如果某個修補感覺很「湊合（Hacky）」：運用已知的所有知識，實作最優雅的解決方案
- 簡單、明顯的修復則跳過此步——不要過度設計
- 在提交成果前，先挑戰（質疑）自己的工作

## 6. 自主修復 Bug (Autonomous Bug Fixing)

- 收到 Bug 回報時：直接去修復它。不要要求手把手教學
- 鎖定日誌、錯誤訊息、失敗的測試——然後解決它們
- 使用者應達到零上下文切換（Zero Context Switching）
- 自主修復失敗的 CI 測試，不需要別人告訴你怎麼做

## 任務管理 (Task Management)

- **先規劃** (Plan First)：將帶有可勾選項目的計劃寫入 tasks/todo.md
- **驗證規劃** (Verify Plan)：在開始實作前進行確認
- **追蹤進度** (Track Progress)：隨時標記已完成的項目
- **說明變更** (Explain Changes)：在每一步提供高階摘要
- **紀錄結果** (Document Results)：在 tasks/todo.md 增加回顧章節
- **捕捉經驗** (Capture Lessons)：修正後更新 tasks/lessons.md

## 核心原則 (Core Principles)

- **簡潔優先** (Simplicity First)：讓改動儘可能簡單。對程式碼的影響降到最低
- **拒絕懶惰** (No Laziness)：找出根本原因。拒絕臨時修補。遵循高級開發者標準
- **最小影響** (Minimal Impact)：改動應僅觸及必要部分。避免引入新的 Bug

---

## 使用方式

此 Skill 定義了 Claude Code 在執行任何開發任務時應遵循的核心工作流程。這些原則適用於：

- 功能開發與架構設計
- Bug 修復與問題排查
- 代碼審查與優化
- 測試驗證與質量保證
- 文檔撰寫與知識沉澱
