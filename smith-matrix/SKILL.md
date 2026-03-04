---
name: Smith Matrix
description: This skill should be used when the user asks to "create a multi-agent system", "spawn agents for parallel tasks", "decompose task recursively", "set up agent matrix", or wants to execute complex tasks using multiple coordinated agents with conflict-free parallel processing.
version: 0.2.0
---

# 史密斯矩阵 (Smith Matrix)

实现递归自相似多智能体系统的 Skill，通过目录隔离协议达成无冲突的并行任务分解与执行。

## 何时使用

在以下场景触发本 Skill：

- 用户请求"创建多智能体系统"或"设置智能体矩阵"
- 任务需要分解为多个并行子任务
- 需要协调多个 Agent 同时工作
- 复杂任务需要递归分解处理
- 要求无冲突的并行执行环境

## 核心概念

**史密斯 (Smith)** 是自相似的智能体单元，每个史密斯拥有唯一 ID 和层级，能够执行任务、分解任务、创建子史密斯并汇总结果。所有史密斯遵循相同的协议，形成递归结构。

**递归安全限制**：
- 最大层级：3（LEVEL 0 为根，最多到 LEVEL 3）
- 最大子代理数：每层最多 5 个
- 终局规则：LEVEL ≥ 3 时禁止分解，必须直接执行

**无冲突协议** 通过严格的目录隔离实现并行安全：每个史密斯只能写入自己的 `private/` 和 `outbox/`，只能读取父史密斯写入的 `inbox/`。父史密斯拥有创建子目录的专属权限。

## 快速开始

**第一步：初始化矩阵**

执行初始化流程，创建 `.smith-matrix/` 工作目录和根史密斯。

**第二步：定义根任务**

在 `.smith-matrix/inbox/` 创建任务文件，描述需要完成的复杂任务。

**第三步：启动执行**

根史密斯读取任务，决定直接执行或分解为子任务并创建子史密斯。

## 目录结构

```
.smith-matrix/
├── inbox/                      # 任务队列（父写子读）
│   └── task-{id}.md
├── smiths/
│   ├── smith-root/             # 根史密斯
│   │   ├── smith.md            # 史密斯定义（只读）
│   │   ├── private/            # 私有工作区
│   │   ├── outbox/             # 结果输出
│   │   │   └── result.md
│   │   └── children/           # 子史密斯目录
│   │       └── smith-001/
│   └── smith-001/
│       ├── smith.md
│       ├── private/
│       ├── outbox/
│       └── children/
└── results/
    └── final.md                # 最终结果
```

## 初始化矩阵

当用户请求初始化 Smith Matrix 时，执行以下步骤：

**1. 创建目录结构 `.smith-matrix/`**

创建基础目录框架：
- `.smith-matrix/inbox/` —— 任务队列
- `.smith-matrix/smiths/` —— 史密斯目录
- `.smith-matrix/results/` —— 最终结果

**2. 读取 `smith.md` 模板**

从 `smith-matrix/smith.md` 读取史密斯定义模板。

**3. 替换占位符**

替换模板中的变量：
- `{SMITH_ID}` → `smith-root`
- `{PARENT_ID}` → `none`
- `{LEVEL}` → `0`

**4. 写入根史密斯定义**

将替换后的内容写入 `.smith-matrix/smiths/smith-root/smith.md`。

**5. 创建任务文件**

根据用户提供的任务描述，在 `.smith-matrix/inbox/` 创建任务文件。

**6. 生成启动指南**

在 `.smith-matrix/smiths/smith-root/private/START_HERE.md` 生成启动指南，包含：
- 当前身份确认
- 任务文件位置
- 执行流程说明
- 输出要求

## 创建子史密斯

当父史密斯需要创建子史密斯时，执行以下步骤：

**1. 确定新史密斯 ID**

按序号递增生成 ID，如 `smith-001`、`smith-002`。

**2. 创建子目录**

创建 `.smith-matrix/smiths/{smith-id}/` 目录结构。

**3. 读取模板并替换**

读取 `smith.md` 模板，替换占位符：
- `{SMITH_ID}` → 新 ID（如 `smith-001`）
- `{PARENT_ID}` → 当前史密斯 ID
- `{LEVEL}` → 当前层级 + 1

**4. 写入子史密斯定义**

将替换后的内容写入子目录的 `smith.md`。

**5. 创建子目录结构**

创建以下子目录：
- `private/` —— 私有工作区
- `outbox/` —— 结果输出
- `children/` —— 子史密斯容器

**6. 生成子史密斯启动指南**

在 `private/START_HERE.md` 生成启动指南，包含：
- 子史密斯身份确认
- 父史密斯引用
- 任务文件位置
- 执行约束说明

## 执行流程

```
读取 inbox/ 任务
    ↓
分析任务复杂度
    ↓
┌─────────────┴─────────────┐
↓                           ↓
可直接完成              需要分解
    ↓                           ↓
执行任务              设计子任务
    ↓                           ↓
写入 outbox/          创建 inbox/ 子任务
    ↓                           ↓
结束                  创建子史密斯
                              ↓
                        等待子结果
                              ↓
                        汇总结果
                              ↓
                        写入 outbox/
                              ↓
                        结束
```

## 写约束协议

**允许写入**：
- 自己的 `private/` —— 草稿、思考、临时文件
- 自己的 `outbox/result.md` —— 最终结果
- 自己的 `children/` —— 创建子史密斯（父权限）
- `inbox/` —— 创建子任务（父权限）

**禁止写入**：
- 其他史密斯的 `private/` 或 `outbox/`
- 父史密斯的目录

## 参考资料

- [核心概念详解](./references/concepts.md) —— 史密斯的定义、目录隔离机制、递归分解策略
- [协议规范](./references/protocol.md) —— 详细的通信协议和约束规则
- [最佳实践](./references/best-practices.md) —— 任务分解原则、结果汇总技巧

## 示例

- [市场研究示例](./examples/market-research.md) —— 展示如何将复杂的市场研究任务分解为 4 个并行子任务
- [代码重构示例](./examples/code-refactor.md) —— 展示如何并行处理大规模代码重构

## 安装

将此目录复制到 Claude Code skills 目录：

```bash
cp -r smith-matrix ~/.claude/skills/
```
