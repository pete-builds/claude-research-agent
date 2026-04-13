# Claude Research Agent

A custom skill for [Claude Code](https://claude.com/claude-code) that turns the AI into a disciplined, citation-grounded research tool.

Invoke it with `/research <topic>` and it produces a fully sourced, structured report, saved locally and optionally pushed to a public GitHub repo.

**Example output:** [Claude Code Source Code Leak](https://pete-builds.github.io/research-reports/claude-code-source-code-leak.html)

---

## The Problem It Solves

Most AI research tools confidently fill gaps with plausible-sounding content. This agent doesn't. It adds value not by knowing more, but by being disciplined about what it claims.

Every factual statement requires a source. If it can't cite something, it won't say it.

---

## How It Works

When you run `/research <topic>`, the skill spawns a dedicated subagent with a strict rule set:

- **No hallucinations**: Every claim is traceable to a source: a URL fetched and verified, not just surfaced by search.
- **Inline citations**: Every factual statement is tagged `[source: url]` as it's written.
- **Confidence levels**: Findings are labeled High / Medium / Low confidence so you know what's solid vs. inferred.
- **Update vs. Fresh mode**: If a report on the topic already exists, it reads what's there and only searches for what's new, using roughly 30-40% of the tokens a full run costs.
- **Structured output**: Every report follows a consistent format: Current Status, Timeline (reverse chronological), Technical Analysis, Confidence Assessment, Open Questions, and a categorized Sources section.
- **Self-audit**: After drafting, the agent reviews every claim and removes or flags anything it can't support.

---

## Installation

1. Create the skill directory in your Claude Code workspace:

```bash
mkdir -p .claude/skills/research
```

2. Copy the skill file:

```bash
# For the portable version (no self-hosted tools required):
cp SKILL-portable.md .claude/skills/research/SKILL.md

# For the full version (requires SearXNG + threat intel MCP):
cp SKILL.md .claude/skills/research/SKILL.md
```

3. That's it. Run `/research <your topic>` in Claude Code.

---

## Two Versions

### `SKILL-portable.md` — Works out of the box

Uses Claude Code's built-in `WebSearch` and `WebFetch` tools. No additional setup required. Works for any Claude Code user immediately.

The portable version:
- Uses `WebSearch` for discovery, `WebFetch` for source verification
- Saves reports to `./research-reports/` in your current working directory in two formats:
  - **Markdown** (`.md`) for editing, version control, and update cycles
  - **HTML** (`.html`) for reading, sharing, and printing. Self-contained, no external dependencies, works offline, supports dark mode.
- No GitHub account or publishing step required. Share the HTML file however you want: email, shared drive, Slack, Teams, or just open it in a browser.

### `SKILL.md` — Full version

This is the production version of the skill. It references two custom MCP servers that are not included in this repo:

- **[mcp-searxng](https://github.com/pete-builds/mcp-searxng)**: Wraps a self-hosted [SearXNG](https://github.com/searxng/searxng) instance and exposes it as Claude Code tools (`search_deep`, `search_news`, `search_tech`). SearXNG aggregates results from multiple search engines, deduplicates by URL, and scores results by how many engines found them.
- **[mcp-threatintel](https://github.com/pete-builds/mcp-threatintel)**: Provides IOC lookups, CVE checks, breach monitoring, dark web search, and OTX pulse scanning. Useful for security research.

`SKILL.md` is published here as a reference for what's possible when you pair Claude Code with custom search infrastructure. If you want to build a similar setup, start with SearXNG's Docker image and an MCP server that wraps its search API.

**If you don't have these MCP servers, use `SKILL-portable.md` instead.** It produces the same report format using only built-in Claude Code tools.

---

## Customizing the Full Version

If you use `SKILL.md`, update these sections for your environment:

**Report output path** (near the bottom of the file):
```
Write to `/your/path/research-reports/<slug>.md`
```

**GitHub auto-publish** (Step 3 in the Report Output section):
```bash
git clone https://github.com/YOUR-USERNAME/YOUR-REPO.git research-reports-sync
git config user.name "YOUR-USERNAME"
git config user.email "YOUR-USERNAME@users.noreply.github.com"
```

**Remove or skip** the threat intel sections if you don't have that MCP server configured.

---

## Report Format

Every report is generated in the same structure and saved as both `.md` and `.html`:

```
---
title:
date:
updated:
summary:
---

## Current Status       <- 5-6 bullet points on the current state
## Table of Contents
## Findings
  ### 1. What Happened
  ### 2. Technical Analysis
  ### 3. Timeline        <- Reverse chronological
  ### 4. Discovery
  ### 5. Response
  ### 6. [Topic-specific sections]
## Confidence Assessment
  ### High Confidence
  ### Medium Confidence
## Open Questions        <- Updated each cycle in Update Mode
## Sources               <- Grouped by category, links verified
## Update History
## How This Report Was Generated
```

The HTML version is a single self-contained file: no JavaScript, no CDN links, no external dependencies. It supports light and dark mode, renders cleanly on any screen size, and can be opened directly from the filesystem or shared as an attachment.

---

## Update Mode

If you run `/research` on a topic you've already researched, the agent enters Update Mode automatically:

1. Reads the existing report and notes what's already covered
2. Targets the highest-value sources from the last run directly (cheap, targeted)
3. Runs a news search scoped to the time since the last update
4. Appends new findings without rewriting what hasn't changed
5. Updates the `updated:` timestamp, Current Status bullets, Timeline, and Open Questions

This keeps recurring research cheap. A full fresh report might use 50k tokens. An update run typically uses 15-20k.

---

## Example Topics

```bash
/research the xz utils supply chain attack
/research claude code source code leak fallout
/research anthropic model spec changes
/research critical infrastructure ransomware trends 2026
```

---

## Credits

Built by [Pete Stergion](https://github.com/pete-builds) for use with [Claude Code](https://claude.com/claude-code).

SearXNG: [searxng/searxng](https://github.com/searxng/searxng)
