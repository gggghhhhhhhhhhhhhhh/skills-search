---
name: skills-search-audit
description: >
  Quality assurance audit for the skills-search skill.
  Use after skills-search completes its 10-step workflow to verify
  every condition is met. Load this skill when the main skill finishes execution
  to check for missing or incomplete steps. If any step fails, re-execute it.
---

# Audit — skills-search

## Instructions

After the main search skill finishes Steps 1-10, load this audit skill.
Go through each checklist item below. For any `❌` item, re-execute the affected step immediately.

---

## Checklist

### Step 1: Ask keyword
- [ ] Did you ask the user for a keyword?
- [ ] If no keyword, did you warn about token cost?

### Step 2: Cache
- [ ] Did you check `$script:SkillsSearchCache` before API calls?
- [ ] After API calls, did you save results to cache?

### Step 3: API calls
- [ ] Did you use `$ProgressPreference = "SilentlyContinue"`?
- [ ] Did you set `-TimeoutSec 15`?
- [ ] Did you include header `@{"Accept"="application/vnd.github.v3+json"}`?
- [ ] Did you deduplicate by `id`?
- [ ] Did you handle 403 rate limit (60s pause + retry)?

### Step 4: Quality gate
- [ ] Did you apply `stars >= 10` filter?

### Step 5: Metrics
- [ ] Did you compute: 发布天数, star总数, 平均⭐/天, 最近更新, 项目地址?
- [ ] Is `stars/day = stars / Max(1, days since creation)`?

### Step 6: Table A — 总表
- [ ] Is the table sorted by star总数 descending?
- [ ] Does the table contain exactly 30 items (or fewer if not enough results)?
- [ ] Is the table saved as a .md file?
- [ ] Is the table also displayed in the dialog?
- [ ] Header format: Rank | Project | 发布天数 | Star总数 | 平均⭐/天 | 最近更新 | 项目地址

### Step 7: Table B — 新发布Skill表
- [ ] Is it filtered to repos created within last 30 days?
- [ ] Does it have stars >= 10?
- [ ] Is it sorted by star总数 descending?
- [ ] Is it saved as a .md file?
- [ ] Is it also displayed in the dialog?
- [ ] Is it clearly labeled as "新发布Skill表"?

### Step 8: Sort toggle
- [ ] Did you offer sort options: [1]Star总数 [2]平均⭐/天 [3]最近更新 [4]最新发布?
- [ ] Does re-sorting use cached data (no extra API calls)?

### Step 9: User feedback
- [ ] Did you ask "有不相关项目？"?
- [ ] Is `$script:ExcludedRepos` tracking excluded items?

### Step 10: Audit itself
- [ ] Did you load this audit skill and run through all checks?

---

## Quick Result

| Step | Check | Status |
|------|-------|--------|
| 1 | Asked keyword? | ☐ |
| 2 | Cache used/saved? | ☐ |
| 3 | API correct? | ☐ |
| 4 | Quality gate? | ☐ |
| 5 | Metrics computed? | ☐ |
| 6 | 总表 saved + displayed? | ☐ |
| 7 | 新发布表 saved + displayed? | ☐ |
| 8 | Sort offered? | ☐ |
| 9 | Feedback asked? | ☐ |
| 10 | Audit completed? | ☐ |

**All ✅** → Execution complete.
**Any ☐** → Re-run affected step.
