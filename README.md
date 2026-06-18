# 知识中枢（Knowledge Hub）

团队共享的「知识中枢」仓库骨架。各接入工程的开发者把本地攒的知识卡片同步到这里，全团队可检索、可复用。

> 本仓库是**检索聚合镜像**，不是知识真相源（single-source-of-truth）。每条知识的正文真相留在各自工程的本地 git 仓库；中枢只存其派生卡片（索引投影 / 去项目化经验全文）的最终一致副本。

## 目录结构

```
.
├── README.md                 # 本说明
├── schema-version.yaml       # 卡片格式版本号（格式升级时据此迁移）
├── raw/                      # 原始区：各工程 sync 上来的卡片直接落这
│   ├── project-cards/        #   项目特定卡（索引投影，正文留各工程仓库）
│   │   └── <repo-slug>/      #     按来源仓库分文件夹，一张卡一个 .yaml 文件
│   └── general-knowledge/    #   去项目化通用经验（自包含全文）
│       ├── fix-issue/        #     一条经验一个 .md 文件
│       ├── learnings/
│       └── decisions/
├── published/                # 已发布区：经治理筛选的卡片进这（读取方只读此区）
│   ├── project-cards/
│   └── general-knowledge/
├── _state/                   # 搬运程序状态：u4-watermark.yaml（SHA 增量水位）
└── _diagnostics/             # 搬运程序诊断：abstraction-pending.jsonl（存疑卡，append-only）
```

> `_state/` 与 `_diagnostics/` 由搬运程序（curate-hub-knowledge / U4）写入；不在 `raw/` 路径下，不会被增量 diff 回拽。

## 两区职责

- **`raw/`（原始区）**：同步程序的写入目标。任何接入工程的开发者推送知识卡片时，卡片落在 `raw/` 下对应路径。一卡一文件、按来源分片，并发推送几乎不冲突。
- **`published/`（已发布区）**：由搬运程序 `curate-hub-knowledge`（U4，集中维护者单实例跑）从 `raw/` 中挑选、去重、抽象通用化后写入。检索方（如项目管理侧）只读 `published/`。

> 初次启用时 `published/` 为空，需治理程序就位后才有内容；这是预期的过渡态，不是故障。

## 初始化（一次性 · 手动）

1. 在团队 GitLab 上新建一个空仓库，权限设为：成员可 clone / commit / push。
2. 把本骨架内容拷入并推送到默认分支：
   ```bash
   git init && git add . && git commit -m "init knowledge hub" && git push -u origin main
   ```
3. 在每台开发机配置同步端点（同步程序按此配置 push）：见同步程序文档中的 `hub-endpoint.yaml` 说明。

## 卡片格式

- `raw/project-cards/<repo-slug>/<unit-slug>.yaml`：索引投影卡。字段含 `unit_id` / `source`（来源回链）/ `tags`（三轴标签，检索键）/ `summary`（一段话概要）/ `local_ref`。**不含正文全文、不含源码**。
- `raw/general-knowledge/<category>/<slug>.md`：去项目化经验全文。frontmatter 含 `title` / `category` / `tags` / `source_repo`（+ 可选 `source_unit`，溯源元数据，供搬运程序在卡判存疑/不可抽象时降级落 `published/project-cards/<source_repo>/`）；正文为剥离了项目耦合的通用经验（现象 / 适用条件 / 规避或操作 / 原理）。溯源是元数据、正文仍去项目化，二者不冲突。

写入采用「确定路径即 upsert」：同一卡片身份对应同一文件路径，重复同步覆盖该文件，git 天然完成「有则更新、无则新增」。
