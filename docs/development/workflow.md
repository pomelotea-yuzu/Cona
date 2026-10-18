# 开发工作流与约定

适用范围：本仓库当前阶段的原型工作与 OpenSpec 流程。更新：2026-10-18。

## 1. 仓库与提交约定

- 仓库只保留 **OpenSpec 产物**（`.gitignore` + `openspec/`）与 **docs 面向使用者／维护者的层**
  （`docs/README.md` + `docs/guide/` + `docs/architecture/` + `docs/development/` + `docs/product/`）。
- **仅本机保留、不入库**的内容：
  - 原型代码、合成样例与运行证据（`.gitignore` 忽略 `/prototypes/` 与 `/output/`）；
  - 工程过程资产：`docs/research/`、`docs/engineering/`、`docs/collab/`、`docs/drafts/`、`docs/reviews/`，
    以及交接索引 `HANDOFF.md`。
- 需要共享的过程结论，应先整理进正式文档（`docs/architecture|development|product`）或 OpenSpec change。
  2026-10-18 已把「统一设计方案」与「产品大框架讨论纪要」两份从 `drafts/` 整理为正式文档并移入
  `product/product-design.md` 与 `collab/sessions/2026-10-18-product-framework-discussion.md`；
  其中设计稿随 `product/` 入库，讨论纪要仍仅本机保留。
- **未经用户明确指示，不提交、不推送**；提交内容由用户严格控制。远程：`origin`=GitHub、`gitee`=Gitee。

## 2. 原型构建与运行（本机）

```powershell
# 构建原生执行器（必须先满足一次性供给前提，见 environment-and-provisioning.md）
powershell -NoProfile -ExecutionPolicy Bypass -File prototypes/windows-execution/executor/build.ps1

# 运行全部用例（E + F + P），或裁剪
node prototypes/windows-execution/driver/run.mjs
node prototypes/windows-execution/driver/run.mjs --cases=E01,F01,P02 --keep
```

- 构建脚本与 PowerShell 脚本必须保存为 **UTF-8 with BOM**，否则 PowerShell 5.1 按 ANSI 读会导致语法错误。
- `--keep` 保留实验目录（默认运行结束时清理），便于排查。
- 参考：文件路径解析失败时优先查你是否漏授了祖先链某一级（见调研文档）。

## 3. 证据结构

每次运行生成 `output/windows-execution-validation/<run-id>/`：

| 文件 | 内容 |
|---|---|
| `environment.json` | 环境与工具链、权限前提、ACL 探测（`aclProbe`）、授权供给记录 |
| `results.json` | 逐用例 `caseId`／`status`／`expected`／`actual`／`evidencePaths`，以及断言与汇总 |
| `logs/`、`results/` | 执行器 stdout/stderr 与原始 `key=value` 事实（含资源口径） |
| `acl-cleanup.json` | 授权回收、历史遗留清扫、容器档案删除记录 |

用例状态只有四态：`pass` / `fail` / `blocked` / `untested`；**未测与受阻不得当作通过**。

## 4. OpenSpec 流程

- 提案产物：`proposal.md`、`design.md`、`specs/`、`tasks.md`、`validation-matrix.md`；评审记录 `review.md` 与
  `review-response.md` 随 change 保留，不覆盖原始评审结论。
- 每个勾选必须有**代码、运行证据或明确研究结论**；未实现的工作保持未勾选。
- 校验：`openspec.cmd validate <change> --strict --no-interactive`。

## 5. 纪律（沿用 change 硬规则）

- 不因负例全被拒绝就宣称安全，也不因功能收益或资源开销小就掩盖失败。
- **不以放宽隔离冒充通过**（例如改用完整用户权限、放行宽泛网络）。若必须放宽，把新增暴露面写入结论。
- 证据本机保留、结论入库；生成物不直接当提案正文。
- 每次实验后核对两项卫生：
  1. **ACL**：`D:\Cona`、`output` 上容器 SID 数为 0；持久供给（`D:\`、`D:\nodejs`、`Git`）应仍在。
  2. **APPDATA**：`%LOCALAPPDATA%\Packages` 下 `cona.*` 为 0（用 `--delete-profile` 逐个清理）。

## 6. 未验证前不得当作结论

原型里存在若干"想法"（例如把容器档案目录用 junction 重定向到非 C 盘）。这类未验证项必须显式标注，
不得在后续文档中当作可行方案使用。
