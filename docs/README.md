# docs/ · 文档地图与索引

> 本目录参照 `ReBaseAgent/docs/` 的分层惯例组织，但**提交范围更严**：**面向使用者／维护者的文档**
> （guide / architecture / development / product）与**工程过程资产**（research / engineering / collab /
> competition / reviews）分开；前者入库，后者仅本机保留。`product/` 承载产品定稿与已定决定，属维护者
> 必读，故纳入入库层（`research/`、`engineering/`、`collab/` 等纯过程资产不在仓库内）。
> 权威性排序：**源码 / `openspec/changes/archive/` > 本目录**；判断项目状态以源码与归档 change 为准。
>
> **提交边界（2026-10-18 决定）**：仓库**只提交面向使用者／维护者的层**——`guide/`、`architecture/`、
> `development/`、`product/`，以及本文件；`product/` 承载产品定稿与已定决定，属维护者必读，故纳入入库层。
> 工程过程资产**仅本机保留、不入库**（`.gitignore` 忽略 `research/`、`engineering/`、`collab/`、
> `drafts/`、`reviews/`）；原型代码、合成样例与运行证据同样只在本机
> （`/prototypes/`、`/output/`）。交接索引 `../HANDOFF.md` 也只在本机。

---

## 面向使用者／维护者（入库）

| 目录 | 内容 |
|---|---|
| [`architecture/`](architecture/prototype-overview.md) | **架构总览**：原型组件划分、信任边界、fail-closed 启动顺序、停止与证据结构 |
| [`development/`](development/project-status.md) | **开发与维护**：项目状态、开发工作流与约定、环境前提与一次性权限供给 |
| [`product/`](product/product-design.md) | **产品定稿与已定决定**：统一设计方案（设计权威文档）、产品决定记录（提权、目录布局与授权分区、数据落点、C 盘偏好） |
| `guide/` | 预留：面向产品使用者的入门文档（产品尚无可用形态，暂空） |

## 工程过程资产（仅本机保留、不入库）

> 这些是调研、探讨、规划、报告与会话记录，不是面向使用者／维护者的文档；需要共享时应先整理进正式文档或 OpenSpec change。

| 目录 | 内容 |
|---|---|
| `research/` | 调研与实测结论：Windows 隔离原语与工具兼容性（`windows-isolation-primitives.md`） |
| `engineering/` | 执行过程资产：`notes/`（坑与规避）、`plans/`（后续计划）、`reports/`（运行报告） |
| `collab/` | 会话与讨论记录：`sessions/`（含产品大框架讨论纪要） |
| `drafts/` | 个人讨论与设计草稿的本地保留区 |
| `reviews/` | 预留：评审过程记录与截图，**整体本机保留、不入库** |

## 变更历史

- 2026-10-18 首次建立：随 `validate-windows-execution-and-file-rollback` 的 apply 阶段建立分层结构；
  同日把原在 `drafts/` 的两份文档（统一设计方案、产品大框架讨论纪要）整理为正式文档并移入
  `product/product-design.md` 与 `collab/sessions/2026-10-18-product-framework-discussion.md`；
  确立提交边界：**只提交 guide／architecture／development／product**，其余工程过程资产本机保留。
