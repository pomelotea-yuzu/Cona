# 原型架构总览（validate-windows-execution-and-file-rollback）

本文件描述 M0 验证原型的组件划分、信任边界与启动顺序。原型只服务接口验证，**不代表正式产品架构**，
也不锁定正式执行器的语言与工具链。代码位于本机 `prototypes/windows-execution/`（不入库）。

## 组件划分

| 组件 | 位置 | 职责 |
|---|---|---|
| 原生执行器 | `executor/cona_exec.c` → `bin/cona-exec.exe` | 授权核验 → 受限资源与环境 → 挂起创建进程 → Job 归属 → 隔离核验 → 恢复执行；故障注入、心跳、停止核查、观察窗口、ACL 探测维护模式、资源口径回报 |
| 构建脚本 | `executor/build.ps1` | mingw-w64 gcc 构建（本机 MSVC C++ 工具集缺失，以其替代） |
| 一次性供给脚本 | `executor/provision-elevated.ps1` | 提升权限授予容器身份所需访问；`-Revoke` 完整回收 |
| 驱动 | `driver/run.mjs` | 环境报告、实验环境供给、ACL 授予/回收、用例调度、证据写出、清理 |
| 用例实现 | `driver/cases.mjs`（E 执行与隔离）、`cases-files.mjs`（F 文件版本与交付）、`cases-measure.mjs`（P 安装与资源） | 按验证矩阵组织正反例 |
| 版本存储 | `driver/lib/versions.mjs` | 完整清单 + 内容寻址副本、字节核验、一致性检查、上限与失败注入、回退 |
| 交付与恢复 | `driver/lib/delivery.mjs` | 三方计划（导入基线 × 当前原项目 × 成果版本）、目标恢复点、逐项安全替换、重开核查、交付回退 |
| 操作账本 | `driver/lib/ledger.mjs` | 逐项持久进度，区分已完成／未完成／冲突／未知 |
| 合成样例 | `fixtures/payloads/` | 受限进程载荷（正常、越界、路径别名、后台停止、网络、Git）与真实子进程载荷（文件改动） |

## 信任边界

- **可信宿主**：驱动及其持有的环境报告、假凭据、版本存储、交付账本、日志。
- **受限进程（AppContainer）**：只能访问当前样例的工作副本、容器自有 profile 与临时区、明确列出的只读运行时；
  对模拟原项目、其他任务、宿主假凭据、备份无访问权。
- **受保护数据**：版本存储与账本位于宿主区，容器不可读写；交付与恢复只经可信服务执行，受限进程只提出请求。
- **目录分区**：只读运行时区（RX）与可写数据区（F）在授权上分离；原型即按此实现（产品决定见
  [`../product/product-decisions.md`](../product/product-decisions.md)）。

## 启动顺序（fail-closed）

1. 验证授权与路径身份（无授权／已失效／范围外／任务身份不匹配均在启动前拒绝）；
2. 建立受限资源与环境（显式环境列表，不继承宿主秘密；AppContainer 必需的 profile 变量指向容器自有目录）；
3. 创建受限且**挂起**的进程；
4. 完成 Job 归属（禁止 breakaway、kill-on-close）；
5. 隔离核验（容器令牌、Job 归属）；
6. 恢复执行。

任一步失败都销毁未运行载荷并给出原因；**不存在"以当前用户完整权限继续运行"的兜底分支**。

## 停止与终态

停止顺序为"先发停止请求 → 到时限后终止受管理进程 → 核查真实进程身份与终态 → 按固定观察窗口持续观察"。
观察参数在运行前固定（默认心跳 100ms、采样 200ms，核查终态后至少 10 秒且不少于 20 个心跳周期），
用于样例证据可比性，**不是停止时延承诺**。

## 证据结构

每次运行输出到本机 `output/windows-execution-validation/<run-id>/`：
`environment.json`（环境与权限、ACL 探测）、`results.json`（逐用例状态与断言）、`logs/`、`results/`（执行器原始事实）、
`acl-cleanup.json`（授权回收与清扫记录）。
