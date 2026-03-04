# Agent Smith Matrix

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

> 递归自相似多智能体系统 —— 通过目录隔离协议实现无冲突的并行任务分解与执行

**[English](README.md)** | 简体中文

---

## 简介

**Agent Smith Matrix**（史密斯矩阵）是一个通用的多智能体协作框架，通过严格的目录隔离协议，让多个 AI Agent 能够并行工作，无冲突地协作完成复杂任务。

核心设计理念：
- **自相似性**：每个智能体（史密斯）都遵循相同的协议，可递归创建子智能体
- **无冲突并行**：通过目录隔离确保多 Agent 同时工作时互不干扰
- **任务分解**：自动将复杂任务拆分为可并行的子任务
- **平台无关**：可在任何支持多智能体的系统或平台上运行

---

## 快速开始

### 安装

**方式一：Claude Code Skill（推荐）**

将 `smith-matrix` 目录复制到 Claude Code 的 skills 目录：

```bash
# macOS / Linux
cp -r smith-matrix ~/.claude/skills/

# Windows (PowerShell)
Copy-Item -Recurse smith-matrix $env:USERPROFILE\.claude\skills\
```

**方式二：独立使用**

直接复制 `.smith-matrix/` 目录结构到你的项目：

```bash
cp -r .smith-matrix-template ./my-project/.smith-matrix
```

### 使用方法

1. **初始化矩阵**

   在你的工作目录创建 `.smith-matrix/` 结构：
   ```bash
   mkdir -p .smith-matrix/{inbox,smiths,results}
   mkdir -p .smith-matrix/smiths/smith-root/{private,outbox,children}
   ```

2. **定义任务**

   在 `inbox/` 目录创建任务文件：
   ```markdown
   # task-001.md
   ## 任务：AI Agent 市场研究

   ### 目标
   全面了解 AI Agent 市场现状

   ### 子任务
   1. 市场趋势分析
   2. 主要厂商调研
   3. 技术发展追踪
   4. 应用场景研究
   ```

3. **启动执行**

   根智能体读取任务，决定直接执行或分解创建子智能体

---

## 核心概念

### 史密斯 (Smith)

自相似的智能体单元，每个史密斯拥有：
- 唯一 ID（如 `smith-root`、`smith-001`）
- 层级标识（Level 0 为根，逐级递增）
- 父史密斯引用（根史密斯无父）

### 目录隔离协议

```
.smith-matrix/
├── inbox/                 # 任务队列（父写子读）
├── smiths/
│   ├── smith-root/        # 根智能体
│   │   ├── smith.md       # 智能体定义（提示词）
│   │   ├── private/       # 私有工作区
│   │   ├── outbox/        # 结果输出
│   │   └── children/      # 子智能体目录
│   └── smith-001/         # 子智能体
│       ├── smith.md
│       ├── private/
│       ├── outbox/
│       └── children/
└── results/
    └── final.md           # 最终结果
```

**访问控制规则**：

| 目录 | 权限 | 说明 |
|------|------|------|
| `private/` | 只写自己 | 草稿、思考、临时文件 |
| `outbox/` | 只写自己 | 最终结果输出 |
| `children/` | 只写自己 | 创建子智能体（父权限） |
| `inbox/` | 父写子读 | 任务分发队列 |

### 执行流程

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
结束                  创建子智能体
                              ↓
                        等待子结果
                              ↓
                        汇总结果
                              ↓
                        写入 outbox/
                              ↓
                        结束
```

---

## 平台集成

### Claude Code

本仓库已包含完整的 Claude Code Skill 配置：

- **Skill 入口**：`smith-matrix/SKILL.md`
- **触发词**："创建多智能体系统"、"设置智能体矩阵"、"分解任务并行执行"
- **自动初始化**：触发后自动创建 `.smith-matrix/` 目录结构

### 其他平台

Smith Matrix 是一个开放协议，可以在以下平台实现：

- **AutoGen** - 使用 UserProxyAgent + AssistantAgent 组合
- **LangGraph** - 作为状态机工作流实现
- **CrewAI** - 作为 Crew + Agents 结构
- **自定义系统** - 任何支持目录读写和多进程的环境

---

## 示例场景

### 市场研究

将复杂的 AI Agent 市场研究分解为 4 个并行子任务：
1. 市场趋势分析
2. 主要厂商调研
3. 技术发展追踪
4. 应用场景研究

→ [查看完整示例](./smith-matrix/examples/market-research.md)

### 代码审查

将大规模代码审查分解为模块级别并行处理：
1. 数据层审查
2. 业务逻辑层审查
3. API 接口层审查
4. 前端组件审查

### 内容创作

分布式协作完成内容项目：
1. 大纲设计
2. 章节撰写（多个作者并行）
3. 编辑校对
4. 格式统一

---

## 最佳实践

### 任务分解原则

1. **粒度控制**：每个子任务应该在 1-4 小时内可完成
2. **独立性优先**：子任务之间应尽量低耦合，减少依赖
3. **明确接口**：每个任务都应有清晰的输入定义和输出格式
4. **终局思维**：避免无限分解，设定最大层级（建议不超过 3 层）

### 结果汇总技巧

1. **交叉验证**：检查子任务结果之间的一致性
2. **矛盾处理**：发现矛盾时深入分析原因，给出合理解释
3. **增量汇总**：子任务完成后立即部分汇总，避免最后堆积
4. **可追溯性**：汇总结果中引用各子任务的输出路径

---

## 项目结构

```
smith-matrix/
├── SKILL.md              # Claude Code Skill 定义
├── smith.md              # 史密斯核心提示词模板
├── examples/             # 使用示例
│   ├── market-research.md
│   └── code-refactor.md
├── references/           # 参考资料
│   ├── concepts.md       # 核心概念详解
│   ├── protocol.md       # 协议规范
│   └── best-practices.md # 最佳实践
└── templates/            # 文件模板
```

---

## 协议规范

### 智能体定义文件 (smith.md)

```yaml
---
smith_id: smith-001
parent_id: smith-root
level: 1
created_at: 2026-03-05
---

# 史密斯 {SMITH_ID}

## 身份
- ID: {SMITH_ID}
- 父级: {PARENT_ID}
- 层级: {LEVEL}

## 任务
读取 inbox/task-{ID}.md 并执行

## 约束
- 只写入自己的 private/ 和 outbox/
- 可创建 children/ 下的子智能体
- 必须在完成时输出到 outbox/result.md
```

### 结果输出格式 (outbox/result.md)

```markdown
# 结果: {任务标题}

## 摘要
一句话总结执行结果。

## 详细结果
...

## 子任务引用（如有）
- smith-xxx: 负责 ...
- smith-yyy: 负责 ...

## 完成状态
- [x] 已完成
- 完成时间: 2026-03-05 12:00:00
```

---

## 贡献

欢迎提交 Issue 和 Pull Request。

### 扩展想法

- [ ] 可视化监控面板
- [ ] 结果版本控制
- [ ] 任务优先级队列
- [ ] 跨矩阵协作协议

---

## 许可证

[MIT](LICENSE) © 2026 Chen Yijun
