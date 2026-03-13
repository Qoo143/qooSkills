# quest1 — 任務觀看權限 數據流

## 全局總覽

```mermaid
flowchart LR
    subgraph 新增任務
        A1[assign_mission.php]
        A2[assign_mission_prodb.php]
        A3[prodb_mission_share.php]
    end
    subgraph 編輯任務
        B1[modules_mission_edit.php]
        B2[modules_mission_edit_prodb.php]
        B3[prodb_mission_share.php]
    end
    subgraph Dashboard
        C1[modules_dashboard.php]
        C2[modules_teacher_get_mission.php]
    end
    DB[(mission_share_staff)]
    A1 -->|shareStaffIds| A2
    A1 -->|get_shareable_staff| A3
    A2 -->|INSERT| DB
    B1 -->|get_shared_staff<br>get_shareable_staff| B3
    B3 -->|SELECT| DB
    B1 -->|PATCH| B2
    B1 -->|update_share_staff| B3
    B3 -->|DELETE + INSERT| DB
    C2 -->|SELECT UNION| DB
```

---

## 新增任務：完整數據流

```mermaid
flowchart TD
    U[老師操作] --> S1

    subgraph 前端 assign_mission.php
        S1[勾選 ShareMentor] --> S2[getSelectedClassInfo<br>取 hashed class_sn + 學期]
        S2 --> S3[fetchAndRenderShareStaff]
    end

    S3 -->|POST| S4

    subgraph 後端 prodb_mission_share.php
        S4[get_shareable_staff]
        S4 --> S4a[hash2string 解碼 class_sn]
        S4a --> S4b[parseClassSn<br>拆出 org_id / grade / class]
        S4b --> S4c[queryMentorsByClass]
        S4b --> S4d[querySubjectTeachersByClass]
        S4b --> S4e[querySchoolAdmins]
        S4c --> S4f[加 role 欄位]
        S4d --> S4f
        S4e --> S4f
    end

    S4f -->|JSON 回傳<br>user_id + uname + role| S5

    subgraph 前端渲染
        S5[renderShareStaffList]
        S5 --> S5a[buildShareCheckboxGroup<br>顯示 uname + role]
        S5a --> S6[老師勾選人員]
    end

    S6 --> S7[送出任務<br>getShareStaffIds 收集 user_id]
    S7 -->|POST shareStaffIds| S8

    subgraph 後端 assign_mission_prodb.php
        S8[INSERT mission_info] --> S9[lastInsertId 取 mission_sn]
        S9 --> S10{share_mentor == 1<br>且 staffIds 非空?}
        S10 -->|是| S11[saveMissionShareStaff]
    end

    S11 -->|INSERT| DB[(mission_share_staff<br>mission_sn + staff_id)]
```

---

## 編輯任務：完整數據流

```mermaid
flowchart TD
    U[老師進入編輯頁面] --> E1

    subgraph 前端 modules_mission_edit.php
        E1[makeMissionData 載入任務資料]
        E1 --> E2{subject_mission_share == 1?}
        E2 -->|否| E2a[不載入分享清單]
        E2 -->|是| E3[loadShareStaffForEdit]
    end

    E3 -->|POST mission_sn| E4

    subgraph 後端 prodb_mission_share.php — 查已授權
        E4[get_shared_staff]
        E4 --> E4a[SELECT staff_id<br>FROM mission_share_staff]
        E4 --> E4b[SELECT subject_mission_share<br>FROM mission_info]
        E4a --> E4c{關聯表有記錄?}
        E4b --> E4c
        E4c -->|flag=1 且無記錄| E4d[is_legacy = true]
        E4c -->|flag=1 且有記錄| E4e[is_legacy = false]
    end

    E4d -->|JSON| E5
    E4e -->|JSON| E5

    subgraph 前端判斷 + 查可授權人員
        E5{is_legacy?}
        E5 -->|true 舊資料| E5a[查 get_shareable_staff<br>自動勾選該班導師]
        E5 -->|false 新資料| E5b[查 get_shareable_staff<br>依 shared_staff_ids 打勾]
    end

    E5a --> E6[renderShareStaffList]
    E5b --> E6

    E6 --> E7[老師修改勾選 → 按儲存]
    E7 -->|POST PATCH| E8[modules_mission_edit_prodb.php<br>更新 mission_info]
    E8 -->|成功| E9

    E9 -->|POST| E10

    subgraph 後端 prodb_mission_share.php — 更新
        E10[update_share_staff]
        E10 --> E10a[BEGIN TRANSACTION]
        E10a --> E10b[DELETE FROM mission_share_staff<br>WHERE mission_sn = ?]
        E10b --> E10c{ShareMentor == 1<br>且 staffIds 非空?}
        E10c -->|是| E10d[insertShareStaff<br>INSERT mission_share_staff]
        E10c -->|否| E10e[不寫入]
        E10d --> E10f[UPDATE mission_info<br>SET subject_mission_share = flag]
        E10e --> E10f
        E10f --> E10g[COMMIT]
    end

    E10g -->|成功| E11[前端顯示「儲存成功」]
```

---

## Dashboard 查詢：數據流

```mermaid
flowchart TD
    U[老師登入 Dashboard<br>點「其他教師指派」] --> D1

    subgraph modules_teacher_get_mission.php
        D1[組合 UNION 查詢]
        D1 --> D2[Part 1 — 新資料]
        D1 --> D3[Part 2 — 舊資料相容]

        D2 --> D2a[SELECT DISTINCT mission_sn<br>FROM mission_share_staff<br>WHERE staff_id = 當前使用者]

        D3 --> D3a[原導師班邏輯<br>seme_teacher_subject<br>subject_name = 導師班]
        D3a --> D3b[JOIN mission_info<br>subject_mission_share = 1]
        D3b --> D3c[WHERE mission_sn NOT IN<br>SELECT mission_sn<br>FROM mission_share_staff]
    end

    D2a --> D4[UNION 合併]
    D3c --> D4
    D4 --> D5[JOIN mission_info<br>取任務清單顯示]
```

---

## 資料表欄位

```mermaid
erDiagram
    mission_info {
        int mission_sn PK
        varchar teacher_id "指派者"
        tinyint subject_mission_share "0=不分享 1=分享"
    }
    mission_share_staff {
        int sn PK
        int mission_sn FK
        varchar staff_id "被授權者 user_id"
        datetime created_at
    }
    mission_info ||--o{ mission_share_staff : "1:N"
```
