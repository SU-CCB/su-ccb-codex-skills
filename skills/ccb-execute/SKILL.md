---
name: ccb-execute
description: >
  Execute CCB implementation, explore, or consult tasks from Claude based on spec/docs,
  validate the result, and return a compact receipt.
  Use when Claude sends a CCB task with execute/explore/consult mode.
metadata:
  short-description: CCB 执行、协商与回执
---

# CCB 执行

## 触发条件
- Claude 通过 ask 派发了实施任务、勘探任务或协商请求。
- 当前任务已有 spec 或明确的文档路径。
- 需要按 CCB 回执合同或协商契约返回结果。

## 输入
- Claude 提供的 ask 指令（含 mode 标记）。
- `docs/.ccb/specs/active/*.md`。
- 按需读取的架构、需求、计划、模块与经验文档。
- 当前项目代码与配置。

## 三模式行为

### mode: execute（执行模式，默认）
1. 先读 spec，再读相关文档，再看现有实现，最后判断边界。
2. 默认采用半开放实施模式，对实现负责，但不擅自更改高影响决策。
3. 做最小充分改动，允许局部整理与有限重构。
4. 验证是交付的一部分，明确跑了什么、结果如何、哪些未覆盖。
5. 按 `receipt-contract` 生成精简回执。

### mode: explore（勘探模式）
1. 不写代码，返回现状、风险、建议切分与待拍板问题。
2. 如遇高影响决策、任务模糊、范围扩大、设计冲突等情况，立即回抛给 Claude。

### mode: consult（协商模式，v0.2 新增）
1. **只读/分析/推理，不修改任何文件。**
2. 读取 Claude 指定的代码和文档，分析现状。
3. 遵循 evidence-before-conclusion：先列 findings，再给 recommendation。
4. 按 `consult-contract` 格式回复，包含 `analysis_depth_hint` / `hint_reason` / `hint_confidence`。
5. 详见 `references/consult-contract.md`。

## Superpowers 协调

不同模式下 Superpowers 的适用边界不同：

| Superpowers Skill | 执行模式 | 协商模式 |
|---|---|---|
| `executing-plans` | ✅ 适用 | ❌ 不适用 |
| `systematic-debugging` | ✅ 适用 | ⚠️ 仅故障分析咨询 |
| `verification-before-completion` | ✅ 适用 | ❌ 不适用（替代为 evidence-before-conclusion） |
| `receiving-code-review` | ✅ 返工时 | ❌ 不适用 |

详见 `references/superpowers-integration.md`。

## 输出格式

### 执行/勘探模式
- 结果摘要。
- 修改文件或勘探输出。
- 验证结果。
- 风险点。
- 建议补文档项。

### 协商模式
- mode / status / understanding / findings / options / recommendation / rationale / risks / assumptions / open_questions / analysis_depth_hint / hint_reason / hint_confidence

## 停止条件
- 执行/勘探：目标已完成 + 已验证 + 已输出回执。
- 协商：已按 consult-contract 格式回复。

## 升级条件（回抛给 Claude）
- 需求不清楚，无法确定正确做法。
- 现有设计与代码冲突，无法判断以哪个为准。
- 外部契约、数据结构、跨模块依赖、状态流或事务流需要变化。
- 改动范围明显扩大，或继续实施会引入更大风险。
- 协商模式下：`status=blocked` 或 `status=insufficient-context`。
