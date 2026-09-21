```mermaid
flowchart TD
    subgraph WORK["蓝色 master / 橙色 aa / 紫色 bb"]
        direction TD

        O["① 共同起点 · 0a558dd<br/>本地 master 与 origin/master 已同步<br/>test.txt 为空"]
        A0["在本地 aa 提交 a0 · c4376d2<br/>文件内容：a0"]
        A1["在本地 aa 提交 a1 · 7e5545c<br/>文件内容：a0、a1"]
        PA["② PR #16 已创建，等待合并<br/>远程 aa → 远程 master"]
        B0["在本地 bb 提交 b0 · c7666cd<br/>文件内容：b0"]
        B1["在本地 bb 提交 b1 · 59b7fd3<br/>文件内容：b0、b1"]
        PB["③ PR #17 已创建，等待合并<br/>远程 bb → 远程 master"]
        READY["两个 PR 都已创建，尚未合并<br/>远程 master 仍在 0a558dd"]
        P16["④ aa 的 PR #16 已合并<br/>远程 master 更新到 3977c37<br/>文件内容：a0、a1"]
        X{{"⑤ PR #17 与远程 master 有冲突<br/>在 GitHub 点击 Resolve conflicts<br/>保留 b0、b1、a0、a1，去掉冲突标记"}}
        M["⑥ 远程 bb 更新到 0c3d1ee<br/>文件内容：b0、b1、a0、a1<br/>原 PR #17 自动更新，远程 master 仍在 3977c37"]
        P17["⑦ bb 的 PR #17 已合并<br/>远程 master 更新到 8920474<br/>文件内容：b0、b1、a0、a1"]
    end

    subgraph FINISH["同步本地 master，完成流程"]
        direction TD
        S["切回本地 master，拉取并快进同步<br/>本地 master 与 origin/master 均指向 8920474"]
    end

    O -->|"创建 aa"| A0
    A0 --> A1
    A1 -->|"git push origin aa，然后创建 PR"| PA
    O -->|"创建 bb"| B0
    B0 --> B1
    B1 -->|"git push origin bb，然后创建 PR"| PB
    PA --> READY
    PB --> READY
    READY -->|"在 GitHub 先合并 PR #16"| P16
    PB --> X
    P16 -->|"远程 master 的 a0、a1 与 bb 的 b0、b1 冲突"| X
    X -->|"Mark as resolved → Commit merge<br/>将远程 master 合入远程 bb"| M
    M -->|"Merge pull request：合并原 PR #17<br/>将远程 bb 合入远程 master"| P17
    P16 --> P17
    P17 --> S

    classDef mainNode fill:#eff6ff,stroke:#2563eb,color:#172554,stroke-width:2px;
    classDef aaNode fill:#fff7ed,stroke:#c2410c,color:#7c2d12,stroke-width:2px;
    classDef bbNode fill:#f5f3ff,stroke:#7c3aed,color:#4c1d95,stroke-width:2px;
    classDef actionNode fill:#f8fafc,stroke:#64748b,color:#0f172a,stroke-width:1.5px;

    class O,P16,P17 mainNode;
    class A0,A1,PA aaNode;
    class B0,B1,PB,X,M bbNode;
    class READY,S actionNode;

    linkStyle 0,1,2,6 stroke:#c2410c,stroke-width:3px;
    linkStyle 3,4,5,7,9,11,12 stroke:#7c3aed,stroke-width:3px;
    linkStyle 8,10,13 stroke:#2563eb,stroke-width:3px;
    linkStyle 14 stroke:#64748b,stroke-width:2px;
```

| 步骤 | 在哪里操作 | 做了什么 | 完成后的状态 |
| --- | --- | --- | --- |
| ① 从同一起点分别开发 | 本地 aa、bb | 从 `0a558dd2a578f3c0b5537e1f87c830c015f29118` 创建 aa、bb，各提交两次；| aa 到 `7e5545c`，内容为 a0、a1；bb 到 `59b7fd3`，内容为 b0、b1 |
| ② 创建 aa 的 PR | 本地推送；GitHub 创建 PR | 推送 aa，创建 [PR #16](https://github.com/dcdmm/Script/pull/16)：远程 aa → master | aa、origin/aa 与远程 aa 均在 `7e5545c`；主分支尚未改变 |
| ③ 创建 bb 的 PR | 本地推送；GitHub 创建 PR | 推送 bb，创建 [PR #17](https://github.com/dcdmm/Script/pull/17)：远程 bb → master | bb、origin/bb 与远程 bb 均在 `59b7fd3`；两个 PR 都已创建，远程 master 仍在 `0a558dd` |
| ④ 合并 aa 的 PR | GitHub | 合并 PR #16 | 远程 master 更新到 `3977c37`，包含 a0、a1；bb 的 PR #17 与更新后的主分支有冲突 |
| ⑤ 在Github处理冲突 | GitHub 的 PR #17 页面 | 点击 `Resolve conflicts`，编辑冲突文件，保留 b0、b1、a0、a1，去掉冲突标记 | 在Github中确定合并结果；此时尚未将 bb 的 PR 合入 master |
| ⑥ 保存Github上的冲突解决结果 | GitHub 的远程 bb | 点击 `Mark as resolved`，再点击 `Commit merge`，将远程 master 合入远程 bb | GitHub 生成 `0c3d1ee`，远程 bb 更新，原 PR #17 自动更新；远程 master 仍在 `3977c37`。本地 bb 和 origin/bb 仍在 `59b7fd3`，不会自动同步 |
| ⑦ 合并 bb 原来的 PR | GitHub | 点击 `Merge pull request`，完成 PR #17 的合并 | 远程 master 更新到 `8920474`，包含双方内容；远程 bb 仍在 `0c3d1ee`。这一步才把 bb 的整合结果放入主分支 |
