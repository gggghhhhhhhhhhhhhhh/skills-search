---
name: skills-search
description: >
  Search GitHub for AI agent Skills (skills, plugins, MCP tools) by keyword.
  Use when the user wants to: (1) get a quick overview of Skills projects on GitHub for any domain,
  (2) see two ranking tables — all-time Top 30 and recent-month Top 30,
  (3) discover trending Skills projects with star growth metrics.
  Before running, always ask the user which keyword they want to search.
  If the user provides no keyword, default to a broad search but warn
  that this consumes more tokens due to larger result volume.

---
# Skills Search

## Workflow

### Step 1: Ask for the search keyword

Before making any API calls, ask the user:

> "请问你要搜索哪个领域的 Skills？请提供一个关键词（如：科学写作、数据分析、工具开发、视频制作等）。如果不指定，则默认搜索全部领域（约 5 次 API 调用，消耗更多 token）。"

Wait for the user's response:
- **If they provide a keyword** -> use it for targeted search.
- **If not** -> warn about token cost, then proceed with default broad search.

### Step 2: Session cache

Before calling the API, check `$script:SkillsSearchCache`. If the same keyword was searched within the last 5 minutes, reuse cached results and skip API calls.

```powershell
if ($script:SkillsSearchCache -and $script:SkillsSearchCache.Keyword -eq $keyword -and $script:SkillsSearchCache.Time -gt (Get-Date).AddMinutes(-5)) {
    $sortedAll = $script:SkillsSearchCache.ResultsAll
    $sortedNew = $script:SkillsSearchCache.ResultsNew
}
```

After API calls, store results:

```powershell
$script:SkillsSearchCache = @{
    Keyword   = $keyword
    Time      = Get-Date
    ResultsAll = $sortedAll   # Top 30 all-time
    ResultsNew = $sortedNew   # Top 30 recent-month
}
```

### Step 3: Call GitHub Search API

Use 2-3 queries for the user's keyword:

```powershell
$queries = @("$keyword+skill", "$keyword+research", "$keyword+agent")
```

For each query:

```powershell
$url = "https://api.github.com/search/repositories?q=$query&sort=stars&order=desc&per_page=30"
$ProgressPreference = "SilentlyContinue"
$response = Invoke-WebRequest -Uri $url -UseBasicParsing -TimeoutSec 15 -Headers @{"Accept"="application/vnd.github.v3+json"}
$data = $response.Content | ConvertFrom-Json
```

**Rate limit:** 60 req/h unauthenticated. Keep ≤ 5 queries. If 403, pause 60s and retry once.

Deduplicate by `id`:

```powershell
$allItems = @{}
foreach ($item in $data.items) { if (-not $allItems.ContainsKey($item.id)) { $allItems[$item.id] = $item } }
```

### Step 4: Apply quality gate

Filter repos with `stargazers_count -ge 10`.

### Step 5: Compute metrics

For each repo:

```powershell
$created = [DateTime]$item.created_at
$updated = [DateTime]$item.updated_at
$daysOld = [Math]::Max(1, [int]($now - $created).TotalDays)
$starsPerDay = [Math]::Round($item.stargazers_count / $daysOld, 2)
```

Metrics: **发布天数** (days since creation), **star总数** (total stars), **平均每天star数** (stars/day), **最近更新日期** (last update), **项目地址** (URL).

### Step 6: Generate Table A — 总表 (All-time Top 30)

Filter: stars >= 10. Sort by **star总数** descending. Take top 30.

Save as CSV:

```powershell
$csvPath = "$(Get-Location)\skills-search-总表-$keyword.csv"
"Rank,Project,发布天数,Star总数,平均⭐/天,最近更新,项目地址" | Out-File $csvPath -Encoding utf8
$i=0; foreach ($item in $sortedAll) { $i++; $c=[DateTime]$item.created_at; $u=[DateTime]$item.updated_at
  $days=[Math]::Floor(($now-$c).TotalDays); $d2=[Math]::Max(1,$days); $spd=[Math]::Round($item.stargazers_count/$d2,2)
  "$i,$($item.full_name),$days 天,$($item.stargazers_count),$spd,$($u.ToString('yyyy-MM-dd')),$($item.html_url)" | Out-File $csvPath -Encoding utf8 -Append }
```

Then show the same table in dialog:

```
📄 **总表已保存至**: `$csvPath`

| Rank | Project | 发布天数 | Star总数 | 平均⭐/天 | 最近更新 | 项目地址 |
```

### Step 7: Generate Table B — 新发布Skill表 (Recent-month Top 30)

Filter: stars >= 10 AND created within last 30 days. Sort by **star总数** descending. Take top 30.

Save as CSV:

```powershell
$csvPath2 = "$(Get-Location)\skills-search-新发布表-$keyword.csv"
"Rank,Project,发布天数,Star总数,平均⭐/天,最近更新,项目地址" | Out-File $csvPath2 -Encoding utf8
$i=0; foreach ($item in $sortedNew) { $i++; $c=[DateTime]$item.created_at; $u=[DateTime]$item.updated_at
  $days=[Math]::Floor(($now-$c).TotalDays); $d2=[Math]::Max(1,$days); $spd=[Math]::Round($item.stargazers_count/$d2,2)
  "$i,$($item.full_name),$days 天,$($item.stargazers_count),$spd,$($u.ToString('yyyy-MM-dd')),$($item.html_url)" | Out-File $csvPath2 -Encoding utf8 -Append }
```

Then show in dialog.

**Note:** Both tables are saved as .csv files AND shown in the dialog.### Step 8: Sort toggle

After displaying both tables, offer:

> "📌 排序切换: [1]Star总数 [2]平均⭐/天 [3]最近更新 [4]最新发布"

Re-sort the **cached data** without extra API calls.

### Step 9: User feedback

> "📌 有不相关项目？告诉我序号，本会话内自动排除。"

Track in `$script:ExcludedRepos` by repo id.

### Step 10: Load and run audit

After all steps complete, load the **audit skill** to verify execution:

```powershell
# Load audit checklist
$auditPath = "$PSScriptRoot\audit\SKILL.csv"
if (Test-Path $auditPath) {
    # Follow audit instructions to verify all steps
}
```

The audit skill is located at the `audit/` subdirectory within this skill. Load it and follow its instructions to verify every step was executed correctly and completely. If any step is missing or incomplete, re-execute it before finishing.

## Notes

- Both tables are output in the dialog AND saved as .csv files.
- Always respect the quality gate (stars >= 10).
- Cache results by keyword within a session.
- Use CSV table format for all output.
