# GitHub Skills Search

一个 Codex Skill，用于在 GitHub 上发现并排名 AI Agent Skills 项目。

## 功能

- 按任意关键词搜索 GitHub Skills（如科学写作、数据分析、工具开发、视频制作等）
- 多查询去重合并，覆盖更全
- Top 30 排名表：Stars + ⭐/天增速 + 创建日期 + Topics + 最近活跃度
- 🔥 Rising Stars 检测：近 1 月创建且增速超过中位线的潜力项目
- 研究相关性过滤（宽松/严格两档）
- Stars 质量门槛（≥ 10）
- 会话级缓存（同一关键词 5 分钟内免重复 API 调用）
- 用户反馈机制：可标记不相关项目，本会话内自动排除
- 排序切换：Stars / ⭐/天增速 / 最近更新 / 最新创建
- ✅ **内置自审机制**：执行完毕后自动加载审计清单，逐条核查 10 步 30+ 项条件，发现遗漏立即补充

## 安装

```bash
git clone https://github.com/gggghhhhhhhhhhhhhhh/github-skills-search.git `
  "$env:USERPROFILE\.codex\skills\github-skills-search"
```

## 使用方式

在对话中触发：

> **"使用 $github-skills-search 搜索工具开发相关的 Skills"**
> **"用 GitHub Skills Search 帮我搜一下数据分析领域"**
> **"Use $github-skills-search to find video production agent skills"**

### 交互流程

1. 你输入一个关键词（任意领域），或不指定走默认搜索
2. Skill 调 GitHub API 多查询搜索 → 去重 → 过滤 → 计算指标
3. 输出 Top 30 Markdown 表格 + 🔥 Rising Stars Spotlight
4. 你可以切换排序或标记不相关结果
5. ✅ **自动运行内置审计**：核查所有条件是否满足，不满足则补充修正

### 自审机制

Skill 内置了 `references/audit-checklist.md` 审计清单，覆盖 10 个步骤 30+ 项检查点：

- 每一步执行后自动核查，确保无遗漏
- 从"是否问了关键词"到"是否提供了深挖选项"全覆盖
- 发现缺失条件立即修复，保证输出完整规范

## License

MIT
