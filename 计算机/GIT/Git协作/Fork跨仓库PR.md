```mermaid
flowchart TD
    subgraph MAIN["主仓库 dcdmm/Script · 蓝色 master"]
        direction TD
        O["① 主仓库起点 · 6e95e33<br/>test.txt 为空"]
        A0["在 master 提交 a0 · 734215c<br/>文件内容：a0"]
        A1["在 master 提交 a1 · a894076<br/>文件内容：a0、a1"]
        A2["在 master 提交 a2 · eecee1f<br/>文件内容：a0、a1、a2"]
    end

    subgraph FORK["另一个账号 dcdmmgo · 紫色 Fork 工作分支 xx"]
        direction TD
        F["② dcdmmgo 将项目 Fork 到自己的账号<br/>得到仓库 dcdmmgo/Script，xx 从 734215c 分出<br/>文件内容：a0"]
        B0["在 xx 提交 b0 · 3953e61<br/>文件内容：a0、b0"]
        B1["在 xx 提交 b1 · 926027a<br/>文件内容：a0、b0、b1"]
        PUSH["③ xx 推送完成<br/>dcdmmgo/Script 的远程 xx 更新到 926027a<br/>文件内容：a0、b0、b1"]
    end

    subgraph PR["PR #19：先在 xx 解决冲突，再合并回主仓库"]
        direction TD
        OPEN["④ xx 的 PR #19 已创建，等待合并<br/>dcdmmgo/Script:xx → dcdmm/Script:master"]
        X{{"xx 的 b0、b1 与主仓库 master 的 a1、a2 冲突<br/>等待手动解决"}}
        FETCH["从主仓库 dcdmm/Script 获取更新<br/>本地 upstream/master 记录更新到 eecee1f<br/>本地 xx 仍在 926027a"]
        M["⑤ 在本地 xx 解决冲突并完成提交<br/>生成合并提交 a6ca320<br/>文件内容：a0、b0、b1、a1、a2"]
        UPDATE["xx 推送完成<br/>dcdmmgo/Script 的远程 xx 更新到 a6ca320<br/>原 PR #19 自动更新，主仓库 master 仍在 eecee1f"]
        MERGE["⑥ xx 的 PR #19 已合并<br/>主仓库 master 更新到 7f10e1d<br/>文件内容：a0、b0、b1、a1、a2"]
    end

    subgraph FINISH["回到 dcdmm/Script 的本地仓库，同步 master"]
        direction TD
        SYNC["切回 master，拉取主仓库更新并完成快进同步<br/>本地 master 与 origin/master 均指向 7f10e1d<br/>流程完成"]
    end

    O --> A0
    A0 --> A1
    A1 --> A2
    A0 -->|"Fork 协作，xx 从此处分出"| F
    F --> B0
    B0 --> B1
    B1 -->|"git push -u origin xx"| PUSH
    PUSH -->|"dcdmmgo 创建跨仓库 PR #19"| OPEN
    OPEN --> X
    A2 --> X
    X -->|"贡献者本地：git switch xx；git fetch upstream"| FETCH
    B1 -->|"xx 原有内容：a0、b0、b1"| M
    FETCH -->|"在本地 xx 执行 git merge upstream/master<br/>手动解决冲突，再 git add test.txt、git commit"| M
    M -->|"git push origin xx"| UPDATE
    UPDATE -->|"dcdmm 合并 PR #19：Fork 的 xx → 主仓库 master"| MERGE
    A2 --> MERGE
    MERGE -->|"git switch master；git pull --tags origin master"| SYNC
    A2 --> FETCH

    classDef mainNode fill:#eff6ff,stroke:#2563eb,color:#172554,stroke-width:2px;
    classDef forkNode fill:#f5f3ff,stroke:#7c3aed,color:#4c1d95,stroke-width:2px;
    classDef actionNode fill:#f8fafc,stroke:#64748b,color:#0f172a,stroke-width:1.5px;

    class O,A0,A1,A2,FETCH,MERGE mainNode;
    class F,B0,B1,PUSH,X,M,UPDATE forkNode;
    class OPEN,SYNC actionNode;

    linkStyle 0,1,2,9,12,15,17 stroke:#2563eb,stroke-width:3px;
    linkStyle 3,4,5,6,11,13,14 stroke:#7c3aed,stroke-width:3px;
    linkStyle 7,8,10,16 stroke:#64748b,stroke-width:2px;
```

| 步骤 | 在哪里操作 | 做了什么 | 完成后的状态 |
| --- | --- | --- | --- |
| ① 主仓库开发 | `dcdmm/Script:master` | 从 `6e95e33` 的空文件开始，依次提交 a0、a1、a2 | master 到达 `eecee1f`，文件内容为 a0、a1、a2 |
| ② Fork 并开发 xx | 贡献者的 Fork：`dcdmmgo/Script` | Fork 项目；xx 从 `734215c` 分出，依次提交 b0、b1 | xx 到达 `926027a`，文件内容为 a0、b0、b1 |
| ③ 推送 xx 到自己的仓库 | dcdmmgo 的本地 xx | 执行 `git push -u origin xx`，推送到 `dcdmmgo/Script` | 远程 xx 指向 `926027a`；主仓库 master 尚未接收 b0、b1 |
| ④ 创建 PR #19 | GitHub | `dcdmmgo` 创建 [PR #19](https://github.com/dcdmm/Script/pull/19)：`dcdmmgo/Script:xx` → `dcdmm/Script:master` | 两边在 a0 后追加了不同内容，PR 有冲突，暂时不能合并 |
| ⑤ 获取更新、解决冲突并推送 | 贡献者本地 xx | `git switch xx` → `git fetch upstream` → `git merge upstream/master`<br/>编辑 test.txt，删除冲突标记，保留 a0、b0、b1、a1、a2 五行<br/>执行 `git add test.txt` → `git commit` → `git push origin xx` | `upstream/master` 是本地保存的主仓库 master 位置记录；fetch 将它更新到 `eecee1f`，不会修改 xx 的文件；merge 才把这个提交合入 xx。解决冲突后生成 `a6ca320`，父提交依次为 `926027a`、`eecee1f`；推送后原 PR 自动更新，主仓库 master 仍在 `eecee1f` |
| ⑥ 将 xx 的 PR 合入主仓库 | GitHub，`dcdmm/Script:master` | `dcdmm` 合并原 PR #19：Fork 的 xx → 主仓库 master | master 更新到 `7f10e1d`，父提交依次为 `eecee1f`、`a6ca320`；文件内容为 a0、b0、b1、a1、a2；Fork 的远程 xx 仍在 `a6ca320` |