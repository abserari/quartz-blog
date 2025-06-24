# Claude Code 使用指南

> 本文档整理自 Anthropic 官方的 Claude Code 最佳实践，提供了使用这个代理式编码工具的详细指南和技巧。

## 设计理念
Claude Code 是有意设计为低级别和无偏见的，提供接近原始模型访问的能力，而不强制特定的工作流程。这种设计理念创造了一个灵活、可定制、可脚本化和安全的强大工具。

虽然功能强大，但这种灵活性对于新接触代理式编码工具的工程师来说存在学习曲线——至少在他们开发出自己的最佳实践之前。

## 核心最佳实践

### 1. MCP (Model Context Protocol) 配置

#### 在项目中配置 MCP
在签入的 `.mcp.json` 文件中配置（对代码库中的任何人都可用）：
```json
{
  "servers": {
    "puppeteer": {
      "command": "npx",
      "args": ["@modelcontextprotocol/server-puppeteer"]
    },
    "sentry": {
      "command": "npx", 
      "args": ["mcp-server-sentry", "--auth-token", "YOUR_TOKEN"]
    }
  }
}
```

这样，在你的仓库中工作的每个工程师都可以开箱即用地使用这些工具。

#### 调试 MCP
使用 MCP 时，可以使用 `--mcp-debug` 标志启动 Claude 来帮助识别配置问题。

### 2. 自定义斜杠命令

对于重复的工作流程（调试循环、日志分析等），将提示模板存储在 `.claude/commands` 文件夹中的 Markdown 文件中。当你输入 `/` 时，这些命令会通过斜杠命令菜单可用。

#### 示例：自动修复 GitHub Issue
```markdown
Please analyze and fix the GitHub issue: $ARGUMENTS
```

自定义斜杠命令可以包含特殊关键字 `$ARGUMENTS` 来从命令调用中传递参数。

### 3. 多 Claude 实例协作

#### 代码编写与审查分离
一个简单但有效的方法是让一个 Claude 编写代码，而另一个进行审查或测试：
- 类似于与多个工程师合作
- 有时拥有独立的上下文是有益的

#### 测试驱动开发
- 让一个 Claude 编写测试
- 让另一个 Claude 编写代码以通过测试
- 可以通过给它们单独的工作草稿本来让 Claude 实例相互通信

### 4. 并行工作流程

不要等待 Claude 完成每个步骤，Anthropic 的许多工程师采用的方法是：
- 为多个独立任务并行启动多个 Claude 实例
- 这种方法对于多个独立任务特别有效
- 提供了多个检出的轻量级替代方案

### 5. Git Worktrees 使用

Git worktrees 允许你从同一仓库将多个分支检出到单独的目录中：
- 适合需要在多个分支上同时工作的场景
- 避免频繁切换分支的开销
- 每个 Claude 实例可以在独立的 worktree 中工作

## 项目结构建议

```
project/
├── .mcp.json              # MCP 服务器配置
├── .claude/
│   └── commands/          # 自定义斜杠命令
│       ├── debug.md       # 调试命令模板
│       ├── analyze-logs.md # 日志分析模板
│       └── fix-issue.md   # Issue 修复模板
└── src/                   # 源代码
```

## 工作流程优化

### 1. 初始设置
- 配置项目特定的 MCP 服务器
- 创建常用的斜杠命令模板
- 设置必要的 Git worktrees

### 2. 日常使用
- 使用斜杠命令快速启动常见任务
- 对复杂任务使用多个 Claude 实例
- 利用 MCP 服务器进行外部工具集成

### 3. 团队协作
- 将 `.mcp.json` 和命令模板签入版本控制
- 共享最佳实践和常用模板
- 标准化团队工作流程

## 高级技巧

### 1. 脚本化和自动化
Claude Code 的低级别设计使其易于脚本化：
- 可以创建自动化工作流程
- 集成到 CI/CD 管道中
- 构建自定义工具链

### 2. 安全考虑
- Claude Code 设计为安全的强大工具
- 始终审查生成的代码
- 在生产环境中谨慎使用

### 3. 性能优化
- 为独立任务使用并行实例
- 避免在单个实例中处理过多上下文
- 合理使用 worktrees 减少切换开销

## 常见模式

### 1. 调试循环
```markdown
# .claude/commands/debug.md
Analyze the error in $ARGUMENTS and provide:
1. Root cause analysis
2. Suggested fix
3. Steps to prevent similar issues
```

### 2. 代码审查
```markdown
# .claude/commands/review.md
Review the code changes in $ARGUMENTS for:
- Code quality
- Potential bugs
- Performance implications
- Security concerns
```

### 3. 测试生成
```markdown
# .claude/commands/generate-tests.md
Generate comprehensive tests for $ARGUMENTS including:
- Unit tests
- Edge cases
- Integration scenarios
```

## 与其他工具集成

### 1. Puppeteer 集成
用于自动化网页测试和爬虫任务：
```json
{
  "servers": {
    "puppeteer": {
      "command": "npx",
      "args": ["@modelcontextprotocol/server-puppeteer"]
    }
  }
}
```

### 2. Sentry 集成
用于错误跟踪和监控：
```json
{
  "servers": {
    "sentry": {
      "command": "npx",
      "args": ["mcp-server-sentry", "--auth-token", "YOUR_TOKEN"]
    }
  }
}
```

## 故障排除

### 常见问题

#### 1. MCP 服务器连接失败
- 使用 `--mcp-debug` 标志启动 Claude
- 检查 `.mcp.json` 配置文件语法
- 确保所需的 npm 包已安装

#### 2. 斜杠命令不显示
- 确保命令文件在 `.claude/commands/` 目录中
- 文件必须是 Markdown 格式（.md 扩展名）
- 重启 Claude Code 以加载新命令

#### 3. 多实例协作问题
- 确保每个实例有独立的工作目录
- 使用明确的文件命名避免冲突
- 考虑使用 Git worktrees

## 最佳实践总结

1. **保持简单**：从基本配置开始，逐步添加功能
2. **版本控制**：将配置和命令模板纳入 Git 管理
3. **团队标准化**：建立团队共享的工作流程和模板
4. **持续优化**：根据使用经验不断改进配置和流程
5. **安全第一**：始终审查生成的代码，特别是在生产环境中

## 总结
Claude Code 的强大之处在于其灵活性和可定制性。通过遵循这些最佳实践，工程师可以：
- 提高工作效率
- 标准化团队工作流程
- 充分利用 AI 辅助编码的优势

记住，这些建议只是起点。鼓励你进行实验，找到最适合你的方法！

## 更多资源
- [Claude Code 官方文档](https://claude.ai/code)
- [MCP 协议文档](https://modelcontextprotocol.io/)
- [Anthropic Cookbook](https://github.com/anthropics/anthropic-cookbook)
