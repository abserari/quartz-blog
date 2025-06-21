---
title: 🎁首页
enableToc: false
---
[[obsidian]] + [[Quartz]] Publish Web HomePage

Hello👋，本页面提供了一个网络访问我的笔记的途径，使用 [[Hugo]] 搭建（Quartz），笔记使用 Obsidian 编写。

## 双链的阅读建议
- 通过鼠标悬浮预览进行上下文不中断的阅读。
- 通过底部图和双向链接找到更多感兴趣的。
- 想查找直接使用搜索按钮

也可以查看这个导航：[标签](/tags/)


目前的写作工作流：
### 写
在任意地方都可以写，打开一个 [[obsidian]] 目标文件夹即可。

### 发布到网页 notes.abser.top
1. 通过 abserari/quartz 这个 repo，直接使用 obsidian-git 插件，push 到 quartz 库中。
2. 最后通过 git 提交到 github 触发 action 自动构建
3. 构建使用 [[hugo-extended]]  [[hugo-obsidian]] 工具

### 同步功能
然后使用 [remotely save](https://github.com/remotely-save/remotely-save) 插件同步，
