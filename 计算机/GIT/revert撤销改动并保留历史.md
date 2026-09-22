```mermaid
flowchart TD
    subgraph BACKGROUND
        direction TD

        O["共同起点 · e278964<br/>test.txt 为空"]
        A0["master：添加 a0 · 47e4949<br/>test.txt：a0<br/>合并前，当前分支 master 指向此提交"]
        B0["xx：添加 b0 · 4f41cfb<br/>test.txt：b0"]
        B1["xx：添加 b1 · c22b8b8<br/>test.txt：b0、b1"]
        B2["xx：随后添加 b2 · a8003a9<br/>test.txt：b0、b1、b2<br/>合并前，被合并分支 xx 指向此提交"]
        M["当前在 master，执行 git merge xx<br/>解决冲突，生成合并提交 2318068<br/>父编号 1：master 的 47e4949<br/>父编号 2：xx 的 a8003a9<br/>test.txt：a0、b0、b1、b2"]

        O -->|"master 开发"| A0
        O -->|"xx 开发"| B0
        B0 --> B1
        B1 --> B2
        A0 -->|"第一父提交：合并前当前分支的最新提交"| M
        B2 -->|"第二父提交：被合并分支的最新提交"| M
    end

    subgraph REVERT
        direction TD

        R1["① 撤销普通提交<br/>git revert c22b8b8<br/>撤销目标：添加 b1 的提交<br/>本例解决冲突后，只移除 b1<br/>新增提交 b7625ac<br/>test.txt：a0、b0、b2"]
        R2["② 撤销一次 revert<br/>git revert b7625ac<br/>撤销目标：① 产生的撤销提交<br/>反向撤销，再次加入 b1<br/>新增提交 0f0c7cb<br/>test.txt：a0、b0、b1、b2"]
        R3["③ 撤销合并提交<br/>git revert -m 1 2318068<br/>撤销目标：合并 xx 的提交<br/>-m 1：以父编号 1 的 47e4949 为比较基准<br/>该基准是合并前的 master，内容只有 a0<br/>合并相对它新增了 b0、b1、b2，因此移除这些内容<br/>新增提交 aeb69e9<br/>test.txt：a0"]

        R1 --> R2
        R2 --> R3
    end

    F["三次 revert 都新增提交，原提交仍保留在历史中<br/>最终 master 指向 aeb69e9，文件内容只剩 a0"]

    M -->|"从合并后的 master 开始执行"| R1
    R3 --> F

    classDef backgroundNode fill:#f8fafc,stroke:#94a3b8,color:#475569,stroke-width:1px;
    classDef targetNode fill:#fff7ed,stroke:#c2410c,color:#7c2d12,stroke-width:2px;
    classDef revertNode fill:#fef2f2,stroke:#dc2626,color:#7f1d1d,stroke-width:2px;
    classDef restoreNode fill:#f0fdf4,stroke:#16a34a,color:#14532d,stroke-width:2px;
    classDef mergeRevertNode fill:#f5f3ff,stroke:#7c3aed,color:#4c1d95,stroke-width:2px;
    classDef resultNode fill:#eff6ff,stroke:#2563eb,color:#172554,stroke-width:2px;

    class O,A0,B0,B2 backgroundNode;
    class B1 targetNode;
    class R1 revertNode;
    class R2 restoreNode;
    class R3 mergeRevertNode;
    class M,F resultNode;

    linkStyle 0,1,2,3,4,5 stroke:#94a3b8,stroke-width:1.5px;
    linkStyle 6 stroke:#16a34a,stroke-width:3px;
    linkStyle 7 stroke:#7c3aed,stroke-width:3px;
    linkStyle 8 stroke:#dc2626,stroke-width:3px;
    linkStyle 9 stroke:#2563eb,stroke-width:3px;
```

| 对比项 | ① 撤销普通提交 | ② 撤销一次 revert | ③ 撤销合并提交 |
| --- | --- | --- | --- |
| 撤销目标 | `c22b8b8`：添加 b1 | `b7625ac`：第①步产生的撤销提交 | `2318068`：合并 xx |
| 命令 | `git revert c22b8b8` | `git revert b7625ac` | `git revert -m 1 2318068` |
| 父提交的含义 | — | — | 本次在 master 执行 `git merge xx`：第一父提交是合并前 master 的 `47e4949`；第二父提交是被合并的 xx 的 `a8003a9`。编号由本次合并方向决定，与提交时间先后无关 |
| 本次改动 | 移除 b1，保留 a0、b0 和后续添加的 b2 | 反向撤销第①步，重新加入 b1 | 撤销合并相对第一父提交引入的改动，移除 b0、b1、b2 |
| 执行前的 test.txt | a0、b0、b1、b2 | a0、b0、b2 | a0、b0、b1、b2 |
| 执行后的 test.txt | a0、b0、b2 | a0、b0、b1、b2 | a0 |
| 新增提交 | `b7625ac` | `0f0c7cb` | `aeb69e9` |
| 操作要点 | 本例有冲突；整理为 a0、b0、b2，执行 `git add test.txt`、`git revert --continue` | 目标是撤销提交 `b7625ac`，从而恢复 b1 | `-m 1` 中的 `1` 是父提交编号：以合并前 master 的 `47e4949`（只有 a0）为比较基准，撤销合并相对它新增的 b0、b1、b2 |
| 历史变化 | 原提交 `c22b8b8` 仍保留 | 原撤销提交 `b7625ac` 仍保留 | 原合并提交 `2318068` 仍保留，master 向前更新到 `aeb69e9` |
