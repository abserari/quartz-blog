---
id: claude-code-task-tool
title: Claude Code Task工具使用指南
date: 2025-06-19
modified: 2025-06-19
tags: [Claude, MCP, 工具, AI, 并行处理]
type: note
status: active
ai-summary: Claude Code的Task工具能够创建并行子代理处理复杂任务
---

# Claude Code Task工具使用指南

## 工具概述

Claude Code可以作为MCP服务器运行（`claude mcp serve`），其中Task工具是一个强大的功能，允许创建子代理来并行处理复杂问题。

## 核心特性

### 子代理机制
- 子代理拥有与主代理相同的工具访问权限
- 唯一限制：子代理不能再生成其他子任务
- 子代理完成后会向主代理报告结果

### 并行处理能力
- 可以同时运行多个子任务
- 保持主任务的上下文窗口有序
- 让Claude保持专注于主要目标

## 实践案例

### 多角色分析示例

```bash
> Read files in the current directory to deduct a pattern for building Tailwind Plus components. 
> You should spawn 4 sub-tasks with slightly different priorities:
> - Design color expert
> - Accessibility expert  
> - Mobile/responsive expert
> - Overall style expert

# 执行结果
⏺ Task(Design Color Expert Analysis)
  ⎿ Done (24 tool uses · 41.5k tokens · 3m 4.4s)

⏺ Task(Accessibility Expert Analysis)
  ⎿ Done (15 tool uses · 38.0k tokens · 2m 0.0s)

⏺ Task(Mobile/Responsive Expert Analysis)
  ⎿ Done (14 tool uses · 45.5k tokens · 2m 1.2s)

⏺ Task(Overall Style Expert Analysis)
  ⎿ Done (23 tool uses · 58.7k tokens · 2m 22.0s)
```

## 使用场景

### 适用情况
1. **复杂问题分解** - 需要多个专业视角
2. **大规模分析** - 需要并行处理提高效率
3. **质量保证** - 通过多个角度验证方案
4. **探索性任务** - 需要广泛调研

### 最佳实践
1. **早期使用** - 在对话或任务早期使用子代理
2. **保留上下文** - 避免主任务上下文污染
3. **明确角色** - 为每个子任务定义清晰的角色和优先级
4. **结果整合** - 主代理负责综合各子任务的结果

## 技术优势

- **效率提升**：并行处理显著减少总体时间
- **质量保证**：多角度分析提高解决方案质量
- **上下文管理**：保持主任务上下文清晰
- **灵活扩展**：可根据需要调整子任务数量

## 参考资源
- [How I use Claude Code](https://spiess.dev/blog/how-i-use-claude-code)
- [[AI工作流-探索规划编码提交]]
- [[AI工作流核心理念]]

---
*创建时间：2025-06-19*