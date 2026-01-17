---
name: writing-plans
description: "當有多步驟任務的規格或需求時使用 - 在開始編碼之前先創建詳細的實施計劃"
---

# 撰寫實施計劃

## 概述

撰寫全面的實施計劃，假設工程師對代碼庫沒有任何上下文。記錄他們需要知道的一切：每個任務要觸碰哪些文件、代碼、測試、可能需要檢查的文檔、如何測試。把整個計劃分成小任務。DRY。YAGNI。TDD。頻繁提交。

假設執行者是熟練的開發者，但對工具集或問題領域幾乎一無所知。

## 計劃保存位置

`docs/plans/YYYY-MM-DD-<功能名稱>.md`

## 任務粒度

**每步是一個動作（2-5 分鐘）：**
- 「寫失敗的測試」 - 一步
- 「運行確保它失敗」 - 一步
- 「實現最小代碼使測試通過」 - 一步
- 「運行測試確保它們通過」 - 一步
- 「提交」 - 一步

## 計劃文檔結構

```markdown
# [功能名稱] 實施計劃

**目標：** [一句話描述這將構建什麼]

**架構：** [2-3 句關於方法]

**技術棧：** [關鍵技術/庫]

---

### 任務 N：[組件名稱]

**文件：**
- 創建：`exact/path/to/file.py`
- 修改：`exact/path/to/existing.py:123-145`
- 測試：`tests/exact/path/to/test.py`

**步驟 1：寫失敗的測試**

```python
def test_specific_behavior():
    result = function(input)
    assert result == expected
```

**步驟 2：運行測試驗證它失敗**

運行：`pytest tests/path/test.py::test_name -v`
預期：FAIL，「function not defined」

**步驟 3：寫最小實現**

```python
def function(input):
    return expected
```

**步驟 4：運行測試驗證它通過**

運行：`pytest tests/path/test.py::test_name -v`
預期：PASS

**步驟 5：提交**

```bash
git add tests/path/test.py src/path/file.py
git commit -m "feat: add specific feature"
```
```

## 記住

- 總是精確的文件路徑
- 計劃中要有完整代碼（不要「添加驗證」）
- 精確的命令和預期輸出
- 引用相關技能（如 `test-driven-development`）
- DRY、YAGNI、TDD、頻繁提交
