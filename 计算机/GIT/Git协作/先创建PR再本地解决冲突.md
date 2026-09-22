```mermaid
flowchart TD
    subgraph WORK["蓝色主分支 / 橙色 aa / 紫色 bb"]
        direction TD

        O["① 共同起点 · 7a96d1d<br/>本地 master 与 origin/master 已同步<br/>test.txt 为空"]
        A0["在本地 aa 提交 a · b9ea1c9<br/>文件内容：a0"]
        A1["在本地 aa 提交 a1 · abeab63<br/>文件内容：a0、a1"]
        PA["② PR #12 已创建，等待合并<br/>远程 aa → 远程 master"]
        B0["在本地 bb 提交 b0 · fddb8b3<br/>文件内容：b0"]
        B1["在本地 bb 提交 b1 · 0cb2d33<br/>文件内容：b0、b1"]
        PB["③ PR #13 已创建，等待合并<br/>远程 bb → 远程 master"]
        READY["两个 PR 都已创建，尚未合并<br/>此时远程 master 仍在 7a96d1d"]
        P12["④ aa 的 PR #12 已合并<br/>远程 master 更新到 c25f6a4<br/>文件内容：a0、a1"]
        X{{"PR #13 与远程 master 有冲突<br/>bb 的 b0、b1 与 master 的 a0、a1 冲突<br/>需先在本地 bb 整合双方改动"}}
        FETCH["⑤ 获取远程 master 的最新状态<br/>git fetch origin master<br/>origin/master 更新到 c25f6a4"]
        M["⑥ 在本地 bb 解决冲突并提交<br/>生成合并提交 e5bfff3，尚未推送<br/>文件内容（三行）：b0、b1a0、a1"]
        UPDATE["⑦ 远程 bb 更新到 e5bfff3<br/>原 PR #13 自动更新，无需重新创建"]
        P13["⑧ bb 的 PR #13 已合并<br/>远程 master 更新到 f87dde7<br/>文件内容（三行）：b0、b1a0、a1"]
    end

    subgraph FINISH["同步本地 master，完成流程"]
        direction TD
        SYNC["切回本地 master，拉取并快进同步<br/>本地 master 与 origin/master 均指向 f87dde7"]
    end

    O -->|"创建 aa"| A0
    A0 --> A1
    A1 -->|"git push origin aa，然后创建 PR"| PA
    O -->|"创建 bb"| B0
    B0 --> B1
    B1 -->|"git push origin bb，然后创建 PR"| PB
    PA --> READY
    PB --> READY
    READY -->|"在 GitHub 先合并 PR #12"| P12
    PB --> X
    P12 -->|"远程 master 已加入 aa 的改动"| X
    X -->|"回到本地处理"| FETCH
    FETCH -->|"在本地 bb 执行 git merge origin/master"| M
    B1 -->|"保留 bb 原有改动"| M
    M -->|"git push origin bb"| UPDATE
    UPDATE -->|"在 GitHub 合并原 PR #13"| P13
    P12 --> P13
    P13 --> SYNC

    classDef mainNode fill:#eff6ff,stroke:#2563eb,color:#172554,stroke-width:2px;
    classDef aaNode fill:#fff7ed,stroke:#c2410c,color:#7c2d12,stroke-width:2px;
    classDef bbNode fill:#f5f3ff,stroke:#7c3aed,color:#4c1d95,stroke-width:2px;
    classDef actionNode fill:#f8fafc,stroke:#64748b,color:#0f172a,stroke-width:1.5px;

    class O,P12,FETCH,P13 mainNode;
    class A0,A1,PA aaNode;
    class B0,B1,PB,X,M,UPDATE bbNode;
    class READY,SYNC actionNode;

    linkStyle 0,1,2,6 stroke:#c2410c,stroke-width:3px;
    linkStyle 3,4,5,7,9,13,14,15 stroke:#7c3aed,stroke-width:3px;
    linkStyle 8,10,12,16 stroke:#2563eb,stroke-width:3px;
    linkStyle 11,17 stroke:#64748b,stroke-width:2px;
```

| 步骤 | 在哪里操作 | 做了什么 | 结果与分支状态 |
| --- | --- | --- | --- |
| ① 共同起点，分别开发 | 本地 aa、bb | 从 `7a96d1dee55a8c711000873247550d1421318fd7` 创建 aa、bb，各提交两次 | aa 到 `abeab63`，bb 到 `0cb2d33`；两边尚未互相合并 |
| ② 创建 aa 的 PR | 本地推送；GitHub 创建 PR | 推送 aa，创建 [PR #12](https://github.com/dcdmm/Script/pull/12)：远程 aa → master | aa、origin/aa 与远程 aa 均在 `abeab63`；创建 PR 不会修改主分支 |
| ③ 创建 bb 的 PR | 本地推送；GitHub 创建 PR | 推送 bb，创建 [PR #13](https://github.com/dcdmm/Script/pull/13)：远程 bb → master | bb、origin/bb 与远程 bb 均在 `0cb2d33`；此时 PR #12 也尚未合并，远程 master 分支仍在 `7a96d1d` |
| ④ 合并 aa | GitHub | 合并 PR #12，生成 `c25f6a4` | 远程 master 更新到 `c25f6a4`，与 PR #13 有冲突；本地 master、origin/master 仍在 `7a96d1d`，bb 仍在 `0cb2d33` |
| ⑤ 获取最新主分支 | 本地 | 执行 `git fetch origin master`，获取远程主分支的最新状态 | origin/master 更新到 `c25f6a4`；本地 master 仍在 `7a96d1d`，工作分支尚未因此合并 |
| ⑥ 解决冲突并提交 | 本地 bb | 在本地 bb 执行 `git merge origin/master`，合入远程 master 的最新提交 `c25f6a4`，解决冲突后提交 | 生成合并提交 e5bfff3，其两个父提交分别是 bb 分支原先指向的 0cb2d33 和远程主分支指向的 c25f6a4；此时远程 bb 和 origin/bb 尚未接收新提交 |
| ⑦ 更新已有 PR | 本地推送；GitHub 自动更新 PR | 推送 bb 的合并提交 | bb、origin/bb 与远程 bb 均在 `e5bfff3`；原 PR #13 自动包含新提交，无需再建 PR |
| ⑧ 合并 bb | GitHub | 合并原 PR #13，生成 `f87dde7` | 远程 master 接收 bb 的整合结果；远程 aa 仍在 `abeab63`，远程 bb 仍在 `e5bfff3` |
