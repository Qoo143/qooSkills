# 因材網 Claude Code 配置

專業化的 Claude Code 配置，用於 API-V3 開發與遷移工作流程。

## 目錄結構

```
.claude/
├── README.md                    # 配置說明（本檔案）
├── agents/                      # 獨立的 Agent 配置
│   ├── api-mapping-table.md    # 建立對照表 Agent
│   ├── api-write.md            # 撰寫程式碼 Agent
│   ├── api-review.md           # 代碼審查 Agent
│   └── api-verify-mapping.md   # 驗證對照表 Agent
├── commands/                    # 簡化的命令配置
│   ├── api-mapping-table.md    # /api-mapping-table
│   ├── api-write.md            # /api-write
│   ├── api-review.md           # /api-review
│   └── api-verify-mapping.md   # /api-verify-mapping
└── skills/                      # 技能知識庫
    └── api-v3/                  # API-V3 架構規範
        ├── skill.md
        ├── routes.md
        └── api-mapping/         # 對照表存放目錄
            └── api-mapping-modules_student.md
```

## Agents vs Commands vs Skills

### Agents（獨立執行單元）

位置：`.claude/agents/`

**特點**：
- 有獨立的工具集（tools）
- 可指定執行模型（model: sonnet/opus/haiku）
- 包含詳細的執行流程和檢查清單
- 自動化執行複雜的多步驟任務

**格式**：
```yaml
---
name: agent-name
description: Agent 功能說明
tools: Read, Write, Grep, Glob, Bash
model: sonnet
---

# Agent 執行指令
...
```

**現有 Agents**：
1. `api-mapping-table` - 分析舊 PHP 檔案，建立對照表（Sonnet）
2. `api-write` - 按照 DAO → Service → Controller 順序撰寫程式碼（Sonnet）
3. `api-review` - 審查程式碼品質、安全性、架構規範（Sonnet）
4. `api-verify-mapping` - 驗證對照表與實作的完整性（Sonnet）

### Commands（簡化調用）

位置：`.claude/commands/`

**特點**：
- 簡化的命令說明
- 可能調用對應的 Agent
- 作為快速參考指南

**使用方式**：
- `/api-mapping-table` - 建立對照表
- `/api-write` - 撰寫程式碼
- `/api-review` - 代碼審查
- `/api-verify-mapping` - 驗證對照表

### Skills（知識庫）

位置：`.claude/skills/`

**特點**：
- 參考文檔和規範
- Claude 會自動讀取並應用
- 提供架構指南、最佳實踐

**現有 Skills**：
1. `api-v3` - API-V3 架構規範、分層職責、開發範例

> **資料庫查詢**：已改用 MCP MySQL Server（`.mcp.json`）直連資料庫，可即時查詢 schema 和資料，不再需要靜態 skill 文件。

## API-V3 開發工作流程

### 完整流程

```
1. 建立對照表
   ↓
2. 驗證對照表
   ↓
3. 撰寫程式碼
   ↓
4. 代碼審查
```

### 1. 建立對照表

**觸發**：「請建立 modules_student.php 的 API 對照表」

**Agent**：`api-mapping-table`（Sonnet）

**執行內容**：
- 讀取並分析舊 PHP 檔案
- 識別所有功能端點（函式或邏輯區塊）
- 分析參數、SQL、權限、回傳格式
- 規劃 Controller 方法對應
- 建立對照表於 `.claude/skills/api-v3/api-mapping/`

**輸出**：
- 對照表檔案：`api-mapping-{filename}.md`
- 分析摘要：端點數、Controller 建議、安全性問題

### 2. 驗證對照表（可選）

**觸發**：「驗證 api-mapping-modules_student.md 對照表」

**Agent**：`api-verify-mapping`（Sonnet）

**執行內容**：
- 深度比對舊代碼與對照表
- 檢查參數、SQL、權限邏輯是否遺漏
- 檢查 Controller 實作（如已存在）
- 生成驗證報告

**輸出**：
- 驗證報告（CRITICAL/WARNING/SUGGESTION）
- 對照表更新建議

### 3. 撰寫程式碼

**觸發**：「撰寫 TodoController 的 list() 方法」

**Agent**：`api-write`（Sonnet）

**執行內容**：
- 讀取對照表和舊代碼
- 按照 DAO → Service → Controller 順序開發
- 遵循架構規範和最佳實踐
- 更新對照表狀態

**輸出**：
- DAO: `ADLAPI/v3/App/Dao/XxxDao.php`
- Service: `ADLAPI/v3/App/Service/XxxService.php`
- Controller: `ADLAPI/v3/App/Controller/XxxController.php`
- 測試指令

### 4. 代碼審查

**觸發**：「審查 TodoController 的程式碼」

**Agent**：`api-review`（Sonnet）

**執行內容**：
- 檢查分層職責（Controller/Service/DAO）
- 檢查安全性（SQL 注入、XSS、權限）
- 檢查錯誤處理
- 檢查程式碼品質

**輸出**：
- 審查報告（CRITICAL/WARNING/SUGGESTION）
- 具體修正建議

## 使用範例

### 範例 1：完整流程

```
用戶：請幫我遷移 modules/dashboard/modules_student.php 的功能

Claude：
1. 使用 api-mapping-table agent 分析舊檔案
2. 建立對照表，發現 5 個端點
3. 詢問：「對照表已建立，是否要驗證對照表？」

用戶：是的

Claude：
1. 使用 api-verify-mapping agent 驗證
2. 發現遺漏 2 個參數和 1 個權限邏輯
3. 更新對照表

用戶：開始撰寫 list() 方法

Claude：
1. 使用 api-write agent
2. 撰寫 TodoDao::getList()
3. 撰寫 TodoService::getList()
4. 撰寫 TodoController::list()
5. 更新對照表狀態為 ✅

用戶：審查程式碼

Claude：
1. 使用 api-review agent
2. 檢查發現 1 個 WARNING（缺少註解）
3. 生成審查報告
```

### 範例 2：單獨使用

```
用戶：只需要建立對照表，不要撰寫程式碼

Claude：
使用 api-mapping-table agent，完成後不繼續執行其他步驟
```

## 模型選擇

所有 Agents 都配置為使用 **Sonnet** 模型：
- **原因**：需要深度理解業務邏輯、精確生成代碼
- **Token 優化**：Agents 只讀取必要的文件
- **成本控制**：可以在 agent 配置中改為 `model: haiku`（簡單任務）

## 自訂配置

### 修改 Agent 模型

編輯 `.claude/agents/xxx.md`：
```yaml
---
name: agent-name
model: haiku  # 改為 haiku（快速、便宜）
---
```

### 新增 Agent

1. 在 `.claude/agents/` 建立新檔案
2. 使用標準格式（參考現有 agents）
3. 在 `.claude/commands/` 建立對應的簡化命令

### 更新 Skills

直接編輯 `.claude/skills/api-v3/skill.md`

## 參考資料

### 架構規範
- `.claude/skills/api-v3/skill.md` - API-V3 完整架構指南
- `CLAUDE.md` - 專案級指令（XSS 防護要求）

### 資料庫
- MCP MySQL Server（`.mcp.json`）- 即時查詢資料庫 schema 與資料

### 範例代碼
- `ADLAPI/v3/App/Controller/` - 現有 Controller
- `ADLAPI/v3/App/Service/` - 現有 Service
- `ADLAPI/v3/App/Dao/` - 現有 DAO

## 故障排除

### Q: Agent 沒有被觸發？
A: 確認檔案格式正確，frontmatter 包含 `name`, `description`, `tools`, `model`

### Q: 想要更詳細的日誌？
A: 在對話中要求：「請詳細說明每個步驟的執行過程」

### Q: Agent 執行結果不符預期？
A: 提供更具體的指令，例如：「請按照對照表 api-mapping-modules_student.md 的第 3 個端點撰寫程式碼」

## 更新日誌

### 2026-02-02
- ✅ 建立專業化的 Agents 配置（參考 everything-claude-code）
- ✅ 建立 Commands 簡化調用
- ✅ 整合 Skills 知識庫
- ✅ 所有 Agents 配置為 Sonnet 模型
- ✅ 清理舊的 workflows 目錄
