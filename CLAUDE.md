# Sophos Weekly Competitive Intelligence Brief

This repository powers a scheduled remote agent that produces Jason Cobb's weekly competitive
intelligence brief for Sophos enterprise sales territory (TN, AL, MS + national strategic accounts).

## How to run

When invoked, read `SKILL.md` and execute the full competitive intelligence brief workflow.
The agent will research all 12 tracked companies and publish a structured Notion page.

## Required MCP connectors

- **Exa** (`https://mcp.exa.ai/mcp`) — blog/newsroom/analyst content retrieval
- **Tavily** — time-sensitive news, earnings, leadership changes
- **Notion** — output destination for the weekly brief page

## Trigger prompt

> Run the weekly competitive intelligence brief as defined in SKILL.md. Research all 12 companies
> for the past 7 days, synthesize the executive summary, and publish the Notion page. Return the
> Notion URL when complete.
