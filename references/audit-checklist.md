# Audit Checklist — github-skills-search

This is a built-in quality assurance checklist for the github-skills-search skill.
After the search-skill completes its 10-step workflow, load this checklist and verify
every condition. If any condition is not met, supplement the missing step immediately.

## Audit Instructions

1. Load this audit checklist at the end of search-skill execution.
2. Go through each condition below one by one.
3. If a condition is **not met** or **partially met**, stop and fix it before proceeding.
4. Only mark "✅ Pass" when all items are verified.

---

## 10-Step Audit Checklist

### Step 1: Ask for search domain
- [ ] Did the agent ask the user for a keyword before making API calls?
- [ ] If user did NOT provide a keyword, was a warning about token cost displayed?
- [ ] The question should be: "请问你要搜索哪个领域的 Skills？请提供一个关键词..."

### Step 2: Session cache
- [ ] Did the agent check `$script:SkillsSearchCache` before calling APIs?
- [ ] If cached and within 5 minutes, were results reused without API calls?
- [ ] After new API calls, were results stored in the cache variable?

### Step 3: Determine search queries
- [ ] **User specified a domain**: Were only targeted queries used? (2-3 max: `<domain>+skill`, `<domain>+research`, maybe `<domain>+agent`)
- [ ] **No domain**: Were exactly 5 broad queries used?
- [ ] Broad queries must include: `research+skill`, `research+agent+skills`, `academic+research+skills`, `scientific+research+workflow+skill`, `科研+skills`
- [ ] Are queries kept to ≤ 5 total to stay within rate limits?

### Step 4: Call GitHub Search API
- [ ] Is `$ProgressPreference = "SilentlyContinue"` set before API calls?
- [ ] Is `-TimeoutSec 15` set on every request?
- [ ] Is the header `@{"Accept"="application/vnd.github.v3+json"}` included?
- [ ] Are results deduplicated by `id` using a hashtable?
- [ ] If 403 returned, was a 60s pause + retry attempted?

### Step 5: Quality gate
- [ ] Is `stargazers_count -ge 10` filter applied?
- [ ] Are filtered-out items silently excluded (not shown to user)?

### Step 6: Research filter
- [ ] Is keyword-based filtering applied on `description` and `topics`?
- [ ] Are the research keywords used: `research, science, academic, scientific, paper, study, publication, 实验室, 科研, 学术, 论文, 研究`?
- [ ] Default mode is **loose** (>= 1 keyword match)?
- [ ] If filtering removes too many results, does the agent ask user to switch to **strict** mode (>= 2 keyword matches)?

### Step 7: Calculate metrics
- [ ] For each repo, `stars/day = stars / Max(1, (now - created_at).TotalDays)`?
- [ ] Is `stars/day` rounded to 2 decimal places?
- [ ] Is the **median** (not mean) stars/day of Top 30 calculated as the Rising Star reference line?
- [ ] Is `oneMonthAgo = now.AddMonths(-1)` correctly set?
- [ ] Are "创建日期" and "最近更新" extracted from `created_at` and `updated_at`?
- [ ] Is `lastCommitDays` derived from `updated_at` as a proxy for activity?

### Step 8: Output results

#### 8A: Table format
- [ ] Is the output printed as a **Markdown table** (not Format-Table or plain text)?
- [ ] Does the table include ALL of these columns: Rank, Project (with link), 创建日期, 最近更新, Lang, Stars, Stars/day, 活跃, Topics?
- [ ] Are 🆕 markers added for repos created within last 30 days?
- [ ] No verbose description list below the table? (descriptions only shown on deep-dive)

#### 8B: Rising Stars Spotlight
- [ ] Is the median stars/day value displayed?
- [ ] Are repos with (created < 30 days AND stars/day > median) highlighted with 🔥?
- [ ] If no repos qualify, are the top 3 closest candidates shown with their stars/day vs median?

#### 8C: Sort toggle
- [ ] Are sorting options offered to the user? [1]Stars [2]Stars/day [3]Last update [4]Newest
- [ ] Does re-sorting use cached data (no extra API calls)?

### Step 9: User feedback loop
- [ ] Did the agent ask "这些结果中是否有不相关的项目？"?
- [ ] Is `$script:ExcludedRepos` hashtable tracking excluded repo IDs?
- [ ] On subsequent searches with same keyword, are excluded repos filtered out?

### Step 10: Deep-dive offer
- [ ] Did the agent offer to: open a repo URL, fetch README summary, show language breakdown, or refine search?
- [ ] Is at least one of these options presented to the user?

---

## Quick Summary

| Step | Check | Status |
|------|-------|--------|
| 1 | Asked for domain? | ☐ |
| 2 | Cache checked/saved? | ☐ |
| 3 | Queries correct (targeted vs broad)? | ☐ |
| 4 | API: headers, timeout, dedup, rate limit? | ☐ |
| 5 | Quality gate (>=10 stars)? | ☐ |
| 6 | Research filter (loose/strict)? | ☐ |
| 7 | Metrics correct (stars/day, median)? | ☐ |
| 8A | Table: Markdown + all columns + no verbose desc list? | ☐ |
| 8B | Rising Stars displayed? | ☐ |
| 8C | Sort toggle offered? | ☐ |
| 9 | Feedback loop asked? | ☐ |
| 10 | Deep-dive offered? | ☐ |

## Result

- **All ✅**: Execution is complete and correct.
- **Any ☐**: Fix the missing condition immediately, then re-run the affected step.
