---
name: competitive-intel-brief
description: >
  Use this skill EVERY TIME Jason asks for competitive intelligence, competitor news, or a weekly
  brief. Tracks Sophos, CrowdStrike, ReliaQuest, Microsoft Security, Palo Alto Networks, Fortinet,
  Arctic Wolf, SentinelOne, Rapid7, Darktrace, Proofpoint, and Mimecast. Trigger on: "run my
  competitive intel brief", "pull my weekly brief", "comp intel", "competitor news", "it's Monday
  brief me", "what's new with [company]", "competitive landscape update", "what did I miss in
  cybersecurity", "Sophos news", "Proofpoint news", "Mimecast news", "Darktrace update",
  "tell me about CrowdStrike", "what's Arctic Wolf up to", "prep me for my call", or any request
  about recent activity at any tracked company — including single-company lookups and pre-call prep.
  Produces a Notion page with an executive summary and per-company sections: product announcements,
  pricing, leadership, wins/losses, analyst mentions, channel changes, competitive plays, earnings.
compatibility: >
  Requires web_search tool, Exa MCP (https://mcp.exa.ai/mcp), Tavily MCP (configured via MCP
  settings — API key must not be hardcoded here), and Notion MCP. Sub-agent architecture
  requires Claude Code or Cowork. See CLAUDE.AI FALLBACK for single-agent operation.
---

# Competitive Intelligence Brief Skill

## Purpose

This skill produces a weekly competitive intelligence brief for Jason — a Senior Enterprise Strategic
Accounts Manager at Sophos covering Tennessee, Alabama, and Mississippi, plus national strategic
accounts including Under Armour, Church & Dwight, Ricoh, and Greenberg Traurig. It tracks Sophos
(Jason's own company) plus eleven competitors, synthesizing the past 7 days of news into an
actionable, scannable Notion page he can review before calls and territory planning.

Run cadence: weekly, typically at the start of the week. Jason may also request single-company
deep dives or mid-week updates — the same research protocol applies, just scoped to the companies
requested.

---

## ARCHITECTURE OVERVIEW

This skill uses a **two-phase sub-agent architecture** designed for Claude Code or Cowork:

```
Main Agent (Orchestrator)
│
├── Phase 1: Spawn 12 parallel Research Sub-Agents (one per company)
│   └── Each returns a structured findings object (see RESEARCH SUB-AGENT OUTPUT FORMAT)
│
└── Phase 2: Main agent synthesizes Executive Summary and creates Notion page
    ├── Extended thinking for cross-company pattern recognition
    ├── Check/create "Competitive Intel Briefs" Notion database (if first run)
    └── Create brief page with formatted content → returns Notion URL to Jason
```

**Why this architecture:**
- **Parallelization**: 12 companies x 3+ tool calls each = 36-50+ sequential tool calls if done
  serially. Parallel research sub-agents reduce wall-clock time dramatically.
- **Context isolation**: Each research sub-agent works within a focused, company-specific context
  window rather than accumulating all 12 companies' raw search results in one massive thread.
- **Simplified output**: The main agent writes the Notion page directly via MCP calls — no
  sub-agent needed for output, since Notion writes are straightforward API operations.
- **Exec summary quality**: The main agent retains synthesis. Cross-company pattern recognition
  benefits from having all findings in a single context with extended thinking.

---

## CLAUDE.AI FALLBACK

If running in Claude.ai (no sub-agents available), execute sequentially:
1. Research all 12 companies one at a time using the research tool set
2. Synthesize the executive summary using extended thinking
3. Assemble the full brief content
4. Create the Notion page in the same session

All other instructions (research protocol, output spec, formatting rules) apply identically.
The only difference is sequential vs. parallel execution. Expect significantly longer run time.

---

## COMPANIES TRACKED

| # | Company | Ticker / Notes | Priority |
|---|---------|----------------|----------|
| 1 | **Sophos** | Private — **Jason's company**. Endpoint, XDR, MDR (Sophos + Secureworks Taegis) | Own company |
| 2 | CrowdStrike | CRWD — primary endpoint/XDR competitor | High |
| 3 | ReliaQuest | Private — MDR/XDR competitor, strong in Southeast | High |
| 4 | Microsoft Security | MSFT — Defender/Sentinel, bundling play | High |
| 5 | Palo Alto Networks | PANW — platform consolidation narrative | High |
| 6 | Fortinet | FTNT — firewall/SASE, moving into XDR | Medium |
| 7 | Arctic Wolf | Private — MDR competitor, mid-market and up | Medium |
| 8 | SentinelOne | S — endpoint/XDR, Purple AI narrative | Medium |
| 9 | Rapid7 | RPD — MDR/VM, InsightIDR | Medium |
| 10 | Darktrace | DARK (LSE) — AI-driven network detection & response, email security | Watch |
| 11 | Proofpoint | Private (PE-owned) — email security, threat intelligence, data loss prevention | Watch |
| 12 | Mimecast | Private (PE-owned) — email security, cyber resilience, archiving | Watch |

**Important**: Sophos is listed first because it is Jason's own company. The Sophos section uses a
slightly different subsection structure than the competitor sections — see OUTPUT SPECIFICATION.

**Priority guide for exec summary**: When multiple companies have notable news in the same week,
weight High-priority competitors more heavily in the executive summary. Watch-tier companies
(Darktrace, Proofpoint, Mimecast) should surface in the exec summary only if the development is
directly relevant to Jason's territory or active deals.

**Context for companies 10-12**: Darktrace, Proofpoint, and Mimecast are tracked as Jason expands
Sophos coverage into email and network security territory. While not direct endpoint/XDR competitors,
they appear in enterprise security budget conversations and platform consolidation discussions.
Track them for: product expansions that overlap with Sophos, channel/partner moves in Jason's
territory (TN, AL, MS) and national strategic accounts, and any bundling plays that could crowd
out Sophos in a deal.

**Private companies** (Sophos, ReliaQuest, Arctic Wolf, Proofpoint, Mimecast): Do not include
SEC Filings & Earnings Highlights sections for these companies. Omit the section header entirely.

---

## EXECUTION REQUIREMENTS

### Research Tool Set (used by all Research Sub-Agents and in Claude.ai sequential mode)

Each company requires the following tools. Use all four as appropriate — do not substitute one for
another or skip a tool category.

**Tool roles:**

- **`web_search`** — First-pass broad sweep. Run 1-2 short queries per company:
  - `[company name] cybersecurity news [current month] [current year]`
  - `[company name] announcement [current year]`

- **Exa MCP** (`https://mcp.exa.ai/mcp`) — High-quality long-form content from company-owned
  sources (blogs, newsrooms, press releases) and analyst/industry coverage. Use for:
  - Company blog posts and newsroom announcements (especially private companies: Sophos, ReliaQuest,
    Arctic Wolf, Proofpoint, Mimecast — these often don't get trade press pickup for days)
  - Analyst mentions: Gartner, Forrester, IDC, SE Labs, MITRE coverage
  - Query pattern: `[company name] blog OR newsroom OR press release [current month] [current year]`
  - Query pattern: `[company name] Gartner OR Forrester OR analyst [current year]`
  - Known blog/newsroom URLs to target when direct retrieval is needed:
    - Sophos: sophos.com/en-us/press-office and news.sophos.com
    - CrowdStrike: crowdstrike.com/blog
    - ReliaQuest: reliaquest.com/blog
    - Microsoft Security: microsoft.com/en-us/security/blog
    - Palo Alto Networks: paloaltonetworks.com/blog
    - Fortinet: fortinet.com/blog
    - Arctic Wolf: arcticwolf.com/resources/blog
    - SentinelOne: sentinelone.com/blog
    - Rapid7: rapid7.com/blog
    - Darktrace: darktrace.com/blog
    - Proofpoint: proofpoint.com/us/blog and proofpoint.com/us/newsroom
    - Mimecast: mimecast.com/blog and mimecast.com/about/press-releases

- **Tavily MCP** (configured via MCP settings — use the tool as connected, do not hardcode the API
  key) — Time-sensitive, high-signal news retrieval. Use for:
  - Earnings reports and financial results (public companies: CRWD, MSFT, PANW, FTNT, S, RPD, DARK)
  - Leadership changes (C-suite hires, departures, reorgs)
  - Product launch announcements covered by trade press
  - Competitive plays and campaigns targeting Sophos or Sophos customers
  - Query pattern: `[company name] earnings OR results [current quarter] [current year]`
  - Query pattern: `[company name] CEO OR leadership OR hire OR departure [current month] [current year]`
  - Query pattern: `[company name] product launch OR release [current month] [current year]`
  - Query pattern: `[company name] vs Sophos OR competing OR competitive [current year]`

- **`web_fetch`** — Reserved for when a specific URL has been identified but the search snippet is
  insufficient. Load the full page of a press release, earnings transcript, or article.
  Do not use as a substitute for search. If a page is paywalled or returns an error, note the
  source in plain text without a link: `(Publication Name, Date — paywalled)`.

**Minimum coverage per company**: At least one `web_search`, one Exa, and one Tavily query.
Use additional queries wherever results are thin or ambiguous.

**Fallback queries** (use when results are thin):
- `[company name] [specific product e.g. 'Falcon' or 'Taegis' or 'Intercept X'] update`
- `[company name] channel partner program [current year]`
- `[company name] SEC filing OR investor [current year]` (public companies only)

**Recency**: Prioritize results from the last 7 days. If nothing newsworthy happened for a given
company in the past week, say so explicitly rather than padding with old news.

**No Fabrication**: Never invent news, quotes, statistics, or analyst opinions. A "nothing notable
this week" entry is far more valuable than fabricated filler.

---

## WORKFLOW

### Phase 1 — Parallel Research (Sub-Agent Mode)

**Step 1: Calculate the date range**
Determine today's date. The lookback window is 7 calendar days (e.g., if today is April 17, the
window is April 10-17). Pass this date range to every research sub-agent.

**Step 2: Spawn 12 Research Sub-Agents in parallel**
Spawn one sub-agent per company. Each sub-agent receives:
- The company name, ticker/notes, and Priority from the COMPANIES TRACKED table above
- The 7-day date range
- The full Research Tool Set above
- The RESEARCH SUB-AGENT OUTPUT FORMAT below
- The applicable subsection structure (Sophos vs. competitor — see OUTPUT SPECIFICATION)
- The "so what" framing requirement: each finding must end with Jason's sales angle

Wait for all 12 sub-agents to return before proceeding to Phase 2. If a sub-agent has not returned
after 3 minutes, mark that company as "Research unavailable this week" and proceed with the
remaining findings. Do not omit the company from the brief — include a placeholder noting the
research could not be completed.

If a sub-agent returns a malformed or missing response, treat it the same as a timeout: note the
company as unavailable and continue. Do not retry.

#### RESEARCH SUB-AGENT OUTPUT FORMAT

Each research sub-agent must return a JSON object in this exact structure:

```json
{
  "company": "CrowdStrike",
  "date_range": "April 10-17, 2026",
  "is_sophos": false,
  "subsections": {
    "product_announcements": [
      {
        "headline": "Falcon AI Runtime Protection targets AI workloads",
        "body": "**Falcon AI Runtime Protection** — New capability targeting AI workload security in cloud environments. Direct pressure on Sophos Cloud Workload Protection in enterprise accounts with AI/ML infrastructure. [SiliconANGLE, Apr 10](https://siliconangle.com/2026/04/10/example)"
      }
    ],
    "pricing_packaging": [],
    "leadership_changes": [],
    "wins_losses": [],
    "analyst_mentions": [],
    "channel_changes": [],
    "competitive_plays": [],
    "earnings_highlights": []
  },
  "quiet_week": false,
  "quiet_note": null
}
```

**Notes:**
- `body`: Standard Markdown string. Use `**bold**` for key phrases and takeaways. Embed the source
  link inline as `[Publication Name, Date](URL)`. Do not use HTML or XML tags. Keep each body
  under 400 words to respect Notion's block size limits.
- `quiet_week`: set `true` if no newsworthy activity found. Set `quiet_note` to a one-line
  summary (e.g., "No notable public activity detected this week.").
- All subsection arrays must be present even if empty — use `[]`, not `null`.
- `is_sophos`: `true` only for the Sophos object. Tells the main agent to use Sophos-specific
  subsection labels (see OUTPUT SPECIFICATION).
- `competitive_plays` field: For competitor objects, populate this with competitive plays,
  campaigns, or messaging specifically targeting Sophos or Sophos customers. For the Sophos object
  (`is_sophos: true`), populate this with Sophos's own competitive positioning updates — new
  battlecards, messaging changes, win/loss themes, or go-to-market shifts that affect how Jason
  sells against the field.

---

### Phase 2 — Executive Summary + Notion Page Creation (Main Agent)

**Step 3: Synthesize with extended thinking**
Once all available findings objects have been received (all 12, or all that returned within the
timeout window), activate extended thinking. Review all findings and identify the 3-7 most impactful
developments across all companies with a Sophos-sales lens:
- What creates urgency or risk in Jason's active deals?
- What gives Jason a displacement or expansion angle?
- What is the dominant competitive theme this week?
- Weight High-priority competitors more heavily (see COMPANIES TRACKED Priority column).

Write 3-7 executive summary bullets in this typed format:

```json
"executive_summary_bullets": [
  {
    "type": "threat",
    "text": "**CrowdStrike launches MDR-lite tier** targeting mid-market — direct pressure on Sophos MDR pricing in sub-1000 seat deals. [CrowdStrike Blog, Apr 10](https://example.com)"
  },
  {
    "type": "opportunity",
    "text": "**Mimecast DMARC enforcement gap reported** — customers complaining publicly; Sophos Email is positioned to displace in upcoming renewal cycles. [Reddit/MSP, Apr 9](https://example.com)"
  },
  {
    "type": "sophos",
    "text": "**Sophos UTM EOL migration offer extended** — use as a renewal conversation opener with any account still running SG appliances. [Sophos Newsroom, Apr 10](https://example.com)"
  }
]
```

Type mapping to Notion callout icons (applied when assembling page content):
- `"threat"` -> `> 🔴 **[THREAT]**`
- `"opportunity"` -> `> 🟢 **[OPPORTUNITY]**`
- `"sophos"` -> `> 🔵 **[SOPHOS]**`

**Step 4: Set up the Notion database (first-run only)**
Search for the "Competitive Intel Briefs" database in the Notion workspace:
```
notion-search query="Competitive Intel Briefs"
```

If the database does not exist, create it at workspace level using the Notion MCP's
`notion-create-database` call. Define the following properties (use the Notion MCP property type
names exactly):

| Property Name | Notion Type | Notes |
|---|---|---|
| Brief Title | title | Primary title field |
| Week Of | date | Start date of the 7-day window |
| Threat Count | number | Count of "threat" exec summary bullets |
| Opportunity Count | number | Count of "opportunity" exec summary bullets |
| Sophos Updates | checkbox | true if any Sophos subsection has at least one finding |
| Companies With News | multi_select | One option per company name (see list below) |

Multi-select options for "Companies With News" with suggested colors:
Sophos (blue), CrowdStrike (red), ReliaQuest (orange), Microsoft Security (gray),
Palo Alto Networks (purple), Fortinet (green), Arctic Wolf (blue), SentinelOne (yellow),
Rapid7 (pink), Darktrace (brown), Proofpoint (orange), Mimecast (gray)

This schema gives Jason a filterable archive: he can see at a glance which weeks had the most
threats or opportunities, filter by company, and quickly check whether Sophos had updates.

**Notion MCP unavailable — fallback:** If the Notion MCP is not connected or returns an error,
assemble the full brief as a Markdown file and save it to the outputs folder with the filename
`comp-intel-brief-[date-range].md`. Inform Jason that Notion was unavailable and provide the
file link instead.

**Step 5: Assemble and create the Notion page**
Create a new page in the database. Set these properties:
- `Brief Title`: e.g., `"Comp Intel Brief — Apr 10-17, 2026"`
- `Week Of`: start date of the window in ISO format (e.g., `"2026-04-10"`)
- `Threat Count`: number of `"threat"` bullets in the executive summary
- `Opportunity Count`: number of `"opportunity"` bullets in the executive summary
- `Sophos Updates`: `true` if any Sophos subsection has at least one finding, `false` otherwise
- `Companies With News`: list of company names that had at least one non-empty subsection

See OUTPUT SPECIFICATION for the full Notion page content format.

**Step 6: Quality check**
Before presenting the URL to Jason, verify:
1. **Completeness**: All 12 companies present in the page? Sophos listed first?
2. **Executive summary**: 3-7 bullets? Correct tag types applied?
3. **Quiet companies**: Companies with no news show "No notable activity this week." — not blank.
4. **Private companies**: No earnings section heading for Sophos, ReliaQuest, Arctic Wolf,
   Proofpoint, or Mimecast.
5. **Properties**: Database properties populated correctly?

Fix any issues found, then present the Notion page URL to Jason.

---

## OUTPUT SPECIFICATION

### Notion Page: Comp Intel Brief — [Date Range]

Page content is written in Notion-flavored Markdown. Follow this exact structure.

---

#### Page Header

```
> 🛡️ **Weekly Competitive Intelligence Brief** | [Date Range] | Jason Cobb · Sophos Enterprise Sales
```

Followed immediately by a horizontal rule: `---`

---

#### Section 1: Executive Summary

```
## 🎯 Executive Summary — Top Moves This Week

> 🔴 **[THREAT]** **[bold company + key development]** — [impact on Jason's territory/deals]. [[Publication, Date]](url)

> 🟢 **[OPPORTUNITY]** **[bold key development]** — [Jason's displacement or expansion angle]. [[Publication, Date]](url)

> 🔵 **[SOPHOS]** **[bold Sophos development]** — [how Jason uses this in a selling conversation]. [[Publication, Date]](url)
```

Rules:
- 3-7 bullets total. Use a mix of types appropriate to the week's news.
- Each bullet is its own callout block (separate `>` line with a blank line between each).
- Bold the company name and key development. End every bullet with Jason's action angle.
- If it was a genuinely quiet week with nothing notable, say so plainly in a single neutral bullet.

---

#### Section 2: Sophos (Jason's Company)

```
---

## 🏢 Sophos *(Your Company)*

### 📦 Product & Feature Announcements
[findings or "No notable activity this week."]

### 💰 Pricing & Packaging Changes
[findings or "No notable activity this week."]

### 👥 Leadership & Organizational Changes
[findings or "No notable activity this week."]

### 📊 Wins, Losses & Customer Moves
[findings or "No notable activity this week."]

### 🔍 Analyst & Industry Mentions
[findings or "No notable activity this week."]

### 🤝 Channel & Partner Program Changes
[findings or "No notable activity this week."]

### ⚔️ Competitive Positioning Updates
[findings or "No notable activity this week."]
```

Note: Sophos is a private company. Do not include an SEC Filings & Earnings Highlights section
for Sophos under any circumstances.

---

#### Sections 3-13: Competitor Sections

Order: CrowdStrike, ReliaQuest, Microsoft Security, Palo Alto Networks, Fortinet, Arctic Wolf,
SentinelOne, Rapid7, Darktrace, Proofpoint, Mimecast.

Each competitor section uses this template:

```
---

## ⚔️ [Company Name]
*[Ticker / notes from COMPANIES TRACKED table]*

### 📦 Product & Feature Announcements
[findings or "No notable activity this week."]

### 💰 Pricing & Packaging Changes
[findings or "No notable activity this week."]

### 👥 Leadership & Organizational Changes
[findings or "No notable activity this week."]

### 📊 Wins, Losses & Customer Moves
[findings or "No notable activity this week."]

### 🔍 Analyst & Industry Mentions
[findings or "No notable activity this week."]

### 🤝 Channel & Partner Program Changes
[findings or "No notable activity this week."]

### 🎯 Competitive Plays Targeting Sophos
[findings or "No notable activity this week."]

### 📈 SEC Filings & Earnings Highlights
[findings or "No notable activity this week."]
```

**Private company earnings rule**: For ReliaQuest, Arctic Wolf, Proofpoint, and Mimecast —
omit the `### 📈 SEC Filings & Earnings Highlights` heading entirely. Do not include it with
a placeholder. These companies do not file with the SEC and have no earnings releases to track.

If a sub-agent returned `quiet_week: true` for a company, render its entire section as:

```
---

## ⚔️ [Company Name]
*[Ticker / notes]*

*No notable public activity detected this week.*
```

---

#### Formatting within each subsection

Each finding should be:
- 1-3 sentences, plain direct language
- Lead with the news, end with Jason's "so what" sales angle
- **Bold** the most important phrase or takeaway
- Source as inline Markdown link: `[Publication Name, Date](URL)`
- If no URL available (paywalled or not found): plain text `(Publication Name, Date — paywalled)`
- Keep each finding under 400 words to stay within Notion's block size limits

Example finding:

```
**Falcon AI Runtime Protection** — New capability targeting AI workload security in cloud environments. Direct pressure on Sophos Cloud Workload Protection in enterprise accounts with active AI/ML build-outs. [SiliconANGLE, Apr 10](https://example.com)
```

---

## HANDLING THIN NEWS WEEKS

- **Never pad**: Do not stretch old news or minor updates to fill space.
- **Say it plainly**: "No notable product announcements this week." is a perfectly valid entry.
- **Aggregate quiet competitors**: If 3+ competitors had essentially no news, the exec summary
  can note: "Quiet week for Arctic Wolf, ReliaQuest, and Rapid7 — no significant public activity."
- **Focus Jason's attention**: The report is most valuable when it highlights what changed and
  what didn't — both are useful intel for a seller.
- **Sophos can be quiet too**: Report it honestly if Sophos had no notable public news.

---

## ERROR HANDLING

**Sub-agent timeout or failure**: If a research sub-agent does not return within 3 minutes or
returns a malformed response, include a placeholder in that company's section:
```
*Research unavailable for this company this week. Please check manually.*
```
Do not omit the company from the brief. Do not retry the sub-agent.

**All search tools return no results for a company**: Mark the company `quiet_week: true` with
`quiet_note: "No search results found this week — manual check recommended."` This is different
from a confirmed quiet week; flag it so Jason knows the gap is a data issue, not confirmed silence.

**Notion MCP error**: Fall back to Markdown output as described in Step 4. Always deliver
something — never return empty-handed.

**Paywalled source**: If a key article is behind a paywall and web_fetch cannot retrieve it,
cite it in plain text without a link: `(Publication Name, Date — paywalled)`. Do not fabricate
the content.

**API rate limits**: If a search tool returns a rate limit error mid-research, wait 10 seconds
and retry once. If the retry fails, note the affected company as research-limited and continue.

---

## EVALS

Test prompts and expected outputs are stored in `evals/evals.json`. Cases covered:

1. **Weekly brief** — Standard Monday morning request
2. **Single-company deep dive** — "What's going on with CrowdStrike this week?"
3. **Pre-call prep** — "I have a call tomorrow, prep me on the competitive landscape"
4. **Thin news week** — Week with minimal industry activity
5. **Mid-week check-in** — Request on a Wednesday with a shorter lookback window
