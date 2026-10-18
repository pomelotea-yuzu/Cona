# 环境前提与一次性权限供给

本机前提的完整记录：`output/windows-execution-validation/<run-id>/environment.json`（`aclProbe` 字段）。
**执行类用例必须先满足本文件的前提**，否则文件型载荷在卷根 `lstat` 处 `EPERM`。

## 1. 为什么需要一次提升权限（本机实测）

普通用户做不到的两类操作：

1. **卷根读属性／遍历**：Node／Git 按路径逐段解析会 `stat` 卷根（如 `D:\`）；卷根 DACL 普通用户改不了
   （`SetNamedSecurityInfo` 返回 rc=5）。
2. **运行时安装目录**：`D:\nodejs` 的 2454 个子项 `Access is denied`（只有目录根本身可改 DACL）；
   `C:\Program Files\Git` 连目录本身都是 rc=5。

实测对照：宿主**自有**目录（实验目录）授予容器身份**不需要**提升权限（rc=0）。

## 2. 一次性供给（已在本机执行）

```powershell
# 管理员 PowerShell
powershell -NoProfile -ExecutionPolicy Bypass -File prototypes/windows-execution/executor/provision-elevated.ps1
# 回收（完整撤销本次授予）
powershell -NoProfile -ExecutionPolicy Bypass -File prototypes/windows-execution/executor/provision-elevated.ps1 -Revoke
```

脚本做三件事，均可完整回收：

| 目标 | 授予 | 说明 |
|---|---|---|
| `D:\` 卷根 | 容器身份 `(X,RA)`，**不继承** | 只允许路径解析穿过，不扩大 `D:\` 下其它目录的暴露 |
| `D:\nodejs` | 递归只读 `(RX)` | Node／pnpm 运行时只读 |
| `C:\Program Files\Git` | 递归只读 `(RX)` | Git 运行时只读（产品形态应改为随包运行时，见产品决定） |

要点：**先单独授予目标本身，再递归子项**——实测 `/T` 可能不覆盖根目录，而缺根目录会阻断遍历导致整个目录不可用。

## 3. 容器身份与 ACL 探测

- 容器身份**跨运行固定**为 `cona.prototype.windows`，因此一次性供给不必重复执行。
- 执行器提供两个维护模式（诊断用，不参与正常请求）：
  - `--probe-acl <dir> --container-name <name>`：**只读取 DACL**，报告 `existingContainerAceCount`；
    **例行运行只用这个模式**，避免误删已供给授权。
  - `--probe-acl-write <dir> --container-name <name>`：尝试授予并记录分阶段 rc，仅在"本次新增 ACE"时回退；
    只用于宿主自有对照目录，用来记录"未提升时能否授权"的证据。
- 判据提醒：`setDaclRc` 描述的是**宿主进程本次能否改写该 DACL**（未提升时为 5），
  **不等于**"容器是否已获得访问"；后者看 `existingContainerAceCount > 0`。

## 4. 运行期与结束时的卫生核对

- 驱动每次运行会：为实验目录与仓库目录链授予容器 ACE → 结束时回收 → 清扫历史遗留容器 SID；
  并在结束时删除容器身份档案（原型行为；产品形态应保留该档案）。
- 结束核对：
  1. `icacls D:\Cona`、`icacls d:\Cona\output` 上容器 SID 数应为 0；
  2. 持久供给（`D:\`、`D:\nodejs`、`C:\Program Files\Git`）应仍存在，回收用上面 `-Revoke`；
  3. `%LOCALAPPDATA%\Packages` 下 `cona.*` 应为 0；
  4. **不要**让任何流程去改写卷根 DACL（非提升环境必 rc=5）。

## 5. 已知工具链缺口

| 项 | 状态 |
|---|---|
| MSVC C++ 工具集 | **缺失**（`VC\Tools\MSVC` 下无 `cl.exe`）；本轮以 mingw-w64 gcc 8.1.0 替代，仅服务接口验证 |
| Windows SDK | 已安装（10.0.22621.0 / 10.0.26100.0） |
| Node / pnpm / Git / openspec | Node 24.19.0、pnpm 11.21.0、Git 2.49.0、openspec 1.13.1 |
| 依赖缓存与联网要求 | **未知**（E02 受阻，未执行依赖安装） |
