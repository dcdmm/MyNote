```mermaid
flowchart TD
    subgraph HISTORY["蓝色主分支 / 橙色 aa / 紫色 bb"]
        direction TD

        O["① 共同起点 · 58723a2<br/>本地 master 与 origin/master 已同步<br/>test.txt 为空"]
        A0["在 aa 提交 a0 · 8885010<br/>文件内容：a0"]
        A1["在 aa 提交 a1 · 743e2b5<br/>文件内容：a0、a1"]
        RA["aa 推送完成<br/>远程 aa 与本地 origin/aa 指向 743e2b5<br/>本地 aa 也仍指向 743e2b5"]
        B0["在 bb 提交 b0 · db08dae<br/>文件内容：b0"]
        B1["在 bb 提交 b1 · b57328b<br/>文件内容：b0、b1"]
        P6["② aa 的 PR #6 已合并<br/>远程 master 更新到 5f10f1f<br/>文件内容：a0、a1"]
        FM["③ 获取远程主分支的最新状态<br/>git fetch origin<br/>本地 origin/master 更新到 5f10f1f"]
        X{{"在本地 bb 执行 git merge origin/master<br/>bb 的 b0、b1 与主分支的 a0、a1 冲突<br/>等待手动解决"}}
        M["④ 在 bb 解决冲突并完成提交<br/>生成合并提交 0d0202f<br/>文件内容：a0、a1、b0、b1"]
        RB["bb 推送完成<br/>远程 bb 与本地 origin/bb 指向 0d0202f<br/>本地 bb 也仍指向 0d0202f"]
        P7["⑤ bb 的 PR #7 已合并<br/>远程 master 更新到 a445e6b<br/>文件内容：a0、a1、b0、b1"]

        O -->|"创建 aa"| A0
        A0 --> A1
        A1 -->|"git push origin aa"| RA
        O -->|"创建 bb"| B0
        B0 --> B1
        O --> P6
        RA -->|"PR #6：远程 aa → 远程master"| P6
        P6 --> FM
        B1 -->|"bb 原有内容：b0、b1"| X
        FM -->|"从 origin/master 引入 a0、a1"| X
        X -->|"手动保留双方内容，再 add、commit"| M
        M -->|"git push origin bb"| RB
        P6 --> P7
        RB -->|"PR #7：远程 bb → 远程master"| P7
    end

    subgraph FINISH["同步本地 master，完成流程"]
        direction TD
        S["⑥ 切回 master，拉取远程更新并完成快进同步<br/>本地 master 与 origin/master 均指向 a445e6b<br/>流程完成"]
    end

    P7 --> S

    classDef mainNode fill:#eff6ff,stroke:#2563eb,color:#172554,stroke-width:2px;
    classDef aaNode fill:#fff7ed,stroke:#c2410c,color:#7c2d12,stroke-width:2px;
    classDef bbNode fill:#f5f3ff,stroke:#7c3aed,color:#4c1d95,stroke-width:2px;
    classDef actionNode fill:#f8fafc,stroke:#64748b,color:#0f172a,stroke-width:1.5px;

    class O,P6,FM,P7 mainNode;
    class A0,A1,RA aaNode;
    class B0,B1,X,M,RB bbNode;
    class S actionNode;

    linkStyle 0,1,2,6 stroke:#c2410c,stroke-width:3px;
    linkStyle 3,4,8,10,11,13 stroke:#7c3aed,stroke-width:3px;
    linkStyle 5,7,9,12 stroke:#2563eb,stroke-width:3px;
    linkStyle 14 stroke:#64748b,stroke-width:2px;
```

| 步骤 | 在哪里操作 | 做了什么 | 完成后的状态 |
| --- | --- | --- | --- |
| ① 从同一起点分别开发 | 本地 aa、bb | 从 `58723a2` 创建两个分支，分别修改、提交 | aa 有 a0、a1；bb 有 b0、b1；主分支仍在起点 |
| ② aa 通过 PR 进入主分支 | 本地推送 aa；GitHub 合并 PR | `git push origin aa`，创建并合并 [PR #6](https://github.com/dcdmm/Script/pull/6)：远程 aa → 远程master | 推送后 aa、origin/aa 和远程 aa 均在 `743e2b5`；PR 合并只把远程 master 更新到 `5f10f1f`，本地 master、origin/master 和 bb 不会因此自动更新 |
| ③ 在 bb 同步主分支，遇到冲突 | 本地 bb | `git switch bb` → `git fetch origin` → `git merge origin/master` | fetch 将 origin/master 更新到 `5f10f1f`，不移动本地 master；合并时双方在同一位置添加不同内容，出现冲突，等待解决 |
| ④ 解决冲突并提交 | 本地 bb | 将文件整理成 a0、a1、b0、b1，去掉冲突标记，执行 `git add test.txt` 和 `git commit` | 仅本地 bb 更新到 `0d0202f`；origin/bb 和远程 bb 尚未接收此提交；origin/master 与远程 master 仍在 `5f10f1f` |
| ⑤ bb 通过 PR 进入主分支 | 本地推送 bb；GitHub 合并 PR | `git push origin bb`，创建并合并 [PR #7](https://github.com/dcdmm/Script/pull/7)：远程 bb → master | 推送后 bb、origin/bb 和远程 bb 均在 `0d0202f`；PR 合并把远程 master 更新到 `a445e6b`，本地 origin/master 仍需获取更新 |
| ⑥ 同步本地主分支，完成流程 | 本地 master | 切回 master，拉取远程更新，完成快进同步 | 本地 master 与 origin/master 都指向 `a445e6b`，不产生新提交 |
