---
name: github-skills-search
description: >
  Search GitHub for AI agent Skills (skills, plugins, MCP tools) related to a specific research domain.
  Use when the user wants to: (1) discover trending or rising-star Skills projects on GitHub,
  (2) find research-related Skills repos for academic writing, scientific computing, literature review,
  bioinformatics, medical research, or any other domain; (3) get a formatted ranking table of repos
  with creation date, stars, stars/day, and rising-star indicators.
  Before running, always ask the user which domain/keyword they want to search.
  If the user provides no specific domain, default to a broad search but warn
  that this consumes more tokens due to larger result volume.
---

# GitHub Skills Search

## Workflow

### Step 1: Ask for the search domain

Before making any API calls, ask the user:

> "请问你要搜索哪个领域的 Skills？请提供一个关键词（如：single-cell、bioinformatics、medical、NLP、CV、ecology 等）。如果不指定，则默认搜索全部领域（约 5 次 API 调用，消耗更多 token）。"

Wait for the user's response:
- **If they provide a keyword** -> use it for targeted search (2-3 queries, ~800-1200 tokens for output).
- **If not** -> warn: "默认搜索会使用 5 个查询词，匹配 6000+ 仓库，消耗约 2000-3000 tokens。建议尽量提供关键词。" Then proceed with default.

### Step 2: Session cache

Before calling the API, check if a session-level cache variable exists (e.g., `$script:SkillsSearchCache`).
If the same keyword was searched within the last 5 minutes, reuse the cached results and skip API calls.

```powershell
if ($script:SkillsSearchCache -and $script:SkillsSearchCache.Keyword -eq $keyword -and $script:SkillsSearchCache.Time -gt (Get-Date).AddMinutes(-5)) {
    Write-Output "使用缓存结果。"
    $items = $script:SkillsSearchCache.Results
}
```

After API calls, store results in cache:

```powershell
$script:SkillsSearchCache = @{
    Keyword = $keyword
    Time    = Get-Date
    Results = $allResults
}
```

### Step 3: Determine search queries

Based on user input:

- **User specified a domain** -> Use 2-3 targeted queries:
  - `"<domain>+skill"`
  - `"<domain>+research"` (catches repos without "skill" in name)
  - If results < 50: `"<domain>+agent"` (broader net)

- **User specified no domain (default)** -> Use 5 broad queries:
  - `"research+skill"`
  - `"research+agent+skills"`
  - `"academic+research+skills"`
  - `"scientific+research+workflow+skill"`
  - `"科研+skills"`

Each query returns max 30 results. 5 queries = up to 150 raw items deduplicated.

### Step 4: Call GitHub Search API

For each query:

```powershell
$url = "https://api.github.com/search/repositories?q=$query&sort=stars&order=desc&per_page=30"
$ProgressPreference = "SilentlyContinue"
$response = Invoke-WebRequest -Uri $url -UseBasicParsing -TimeoutSec 15 -Headers @{"Accept"="application/vnd.github.v3+json"}
$data = $response.Content | ConvertFrom-Json
```

**Rate limit handling:** GitHub API allows 60 requests/hour unauthenticated. Keep queries <= 5. If 403 returned, pause 60s and retry once.

Deduplicate by `id` using a hashtable:

```powershell
$allItems = @{}
foreach ($item in $data.items) { if (-not $allItems.ContainsKey($item.id)) { $allItems[$item.id] = $item } }
```

### Step 5: Apply quality gate

Filter repos with `stargazers_count -ge 10`. This removes noise from trivial/unproven projects.
Keep the item in filtered list for metrics calculation but mark it as "below threshold" for display.

### Step 6: Apply research filter

Apply keyword-based filtering on `description` and `topics` to exclude non-research repos:

```powershell
$researchKeywords = @("research", "science", "academic", "scientific", "paper", "study", "publication",
                      "实验室", "科研", "学术", "论文", "研究")
$isResearchRepo = $researchKeywords | Where-Object {
    ($item.description -and $item.description -match $_) -or
    ($item.topics -and $item.topics -match $_)
}
```

If filtering removes too many results, offer user a choice:
> "当前宽松过滤后剩余 N 个项目。需要切换到严格模式（要求 topic 或 description 明确包含 research/academic/科研 等关键词）吗？"

- **Loose mode** (default): repo matches at least 1 research keyword
- **Strict mode**: repo description or topics contain >= 2 research keywords

### Step 7: Calculate metrics

```powershell
$now = Get-Date
$oneMonthAgo = $now.AddMonths(-1)

# For each repo
$created = [DateTime]$item.created_at
$daysOld = [Math]::Max(1, [int]($now - $created).TotalDays)
$starsPerDay = [Math]::Round($item.stargazers_count / $daysOld, 2)

# Also track recent commit activity (if available via GET /repos/{owner}/{repo}/commits?per_page=1)
# Extract last_commit_date from item.updated_at as proxy
$lastCommitDays = [Math]::Floor(($now - [DateTime]$item.updated_at).TotalDays)

# Topics display
$topicsStr = if ($item.topics -and $item.topics.Count -gt 0) {
    $item.topics[0..[Math]::Min(4, $item.topics.Count-1)] -join ", "
} else { "N/A" }
```

- **Stars/day** = stars / Max(1, days since creation)
- **Activity score**: days since last update (lower = more active). Use `updated_at` as proxy.
- **Rising Star flag**: created within last 30 days AND stars/day > median stars/day of Top 30
- **Median** (not mean) is used as reference line because extreme values (e.g. ECC with 1509/day) would skew the mean.

### Step 8: Output results

Print results as **Markdown tables** for best readability:

#### Section A: Top 30 by total Stars

```powershell
Write-Output "| Rank | Project | Language | Stars | Stars/day | Last Commit | Topics |"
Write-Output "|------|---------|----------|-------|-----------|-------------|--------|"
# only output stars/day for the first 3 as summary
```

Columns: Rank | Project (with link) | Language | Stars | Stars/day | Last Commit (days ago) | Rising Star? | Topics (first 5)

Truncate description to 150 chars. Only include Language if non-null.

#### Section B: Rising Stars Spotlight

Only repos meeting: created < 30 days AND stars/day > median stars/day.

```powershell
$medianSpd = ($sorted | ForEach-Object {
    $c=[DateTime]$_.created_at; $d=[Math]::Max(1,($now-$c).TotalDays); $_.stargazers_count/$d
} | Sort-Object)[[Math]::Floor($sorted.Count/2)]
```

If no repos qualify, state clearly and list the top 3 closest candidates with their stars/day vs median.

#### Section C: Sort toggle offer

After displaying default view, offer the user:

> "需要切换排序方式吗？可选： [1] Stars 总量（当前）[2] Stars/天增速 [3] 最近更新 [4] 最新创建"

If user picks an alternative sort, re-sort the existing cached data and re-display without API calls.

### Step 9: User feedback loop (quality validation)

After displaying results, ask:

> "这些结果中是否有不相关的项目？如果有，请告诉我项目名称或序号，我会记住偏好并过滤掉类似项目。"

Track user's feedback in `$script:ExcludedRepos` hashtable (by repo id or owner/name pattern):

```powershell
if (-not $script:ExcludedRepos) { $script:ExcludedRepos = @{} }
$script:ExcludedRepos[$item.id] = $true
```

On subsequent searches with the same keyword in the same session, exclude these repos automatically.

### Step 10: Offer to deep-dive

After displaying results, offer:
- Open any repo URL
- Fetch README summary for a specific repo
- Show language breakdown or contributor stats for a repo
- Refine the search with a different keyword

## Notes

- GitHub API is publicly accessible - no authentication needed for basic searches.
- Prefer PowerShell for API calls (Windows environment).
- Always respect the quality gate (stars >= 10) unless user explicitly asks to see low-star projects.
- Cache results by keyword within a session to avoid redundant API calls.
- Use Markdown table format for output for better human readability.
