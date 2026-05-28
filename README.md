# GitHub Skills Search

快速了解 GitHub 上某个领域相关的 Skill 项目概况。只需输入一个关键词，即可获取两份排名表。

## 功能

- **输入关键词**，搜索 GitHub 上该领域的 Skill 项目
- **总表**：按 Star 总数排名 Top 30（质量门槛 ≥ 10⭐）
- **新发布Skill表**：近 1 个月内发布的项目 Top 30（质量门槛 ≥ 10⭐）
- 数据指标：发布天数、Star 总数、平均每天 Star 数、最近更新日期、项目地址
- 每张表均**保存为 .md 文件**并在**对话框同步显示**
- 排序切换：Star总数 / 平均⭐/天 / 最近更新 / 最新发布
- 用户反馈：可标记不相关项目，本会话内自动排除
- 🔍 **内置审查 Skill**：执行完毕后自动审计，确保每一步完整执行

## 安装

```bash
git clone https://github.com/gggghhhhhhhhhhhhhhh/github-skills-search.git `
  "$env:USERPROFILE\.codex\skills\github-skills-search"
```

## 使用方式

> **"使用 $github-skills-search 搜索视频制作"**
> **"用 GitHub Skills Search 搜一下数据分析"**

### 交互流程

1. 你输入一个关键词
2. 搜索 GitHub → 去重 → 过滤（≥ 10⭐）→ 计算指标
3. 输出 **总表**（All-time Top 30）+ **新发布Skill表**（近 1 月 Top 30）
4. 每张表同步保存为 .md 文件
5. 可切换排序、标记不相关项目
6. 🔍 自动运行审查 Skill，验证所有步骤

## 文件结构

```
github-skills-search/
├── SKILL.md             # 主 Skill（10 步工作流）
├── audit/
│   └── SKILL.md         # 🔍 审查 Skill
├── agents/openai.yaml
└── README.md
```

## License

MIT
