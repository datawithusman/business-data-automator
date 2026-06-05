# Business Data Automator

A Claude Code skill for developers who turn messy client spreadsheets into real systems.

Built for freelancers, consultants, and small agencies working with non-tech business owners. The kind of client who runs their entire operation on Excel and calls you when things break.

[![Version](https://img.shields.io/github/v/release/datawithusman/business-data-automator?color=blue&label=version)](https://github.com/datawithusman/business-data-automator/releases)
[![License: MIT](https://img.shields.io/github/license/datawithusman/business-data-automator?color=green)](LICENSE)
[![Stars](https://img.shields.io/github/stars/datawithusman/business-data-automator?style=social)](https://github.com/datawithusman/business-data-automator/stargazers)
[![Skill Type](https://img.shields.io/badge/type-Claude%20Code%20Skill-6C63FF)](#install)
[![Last Updated](https://img.shields.io/github/last-commit/datawithusman/business-data-automator?label=last%20update)](https://github.com/datawithusman/business-data-automator/commits/main)

## What this skill does

When you load this skill in Claude Code (or Cursor, OpenCode, Windsurf), the agent learns how to:

- Profile messy Excel files from real businesses
- Clean mixed date formats, currency stored as text, and merged cells
- Design simple schemas that match how the business actually thinks
- Pick the right dashboard stack (Metabase, Streamlit, or Next.js)
- Build the 3 automations that small business owners actually use
- Price and scope automation work
- Avoid the anti-patterns that kill these projects

## Why this skill exists

Most developers can write pandas. Most developers cannot turn a 40-sheet Excel file used by a delivery fleet into a working dashboard in 2 weeks.

That gap is where the money is. Small businesses in MENA, South Asia, Africa, and Latin America are running on Excel. Their owners have budget but cannot find developers who understand their work.

This skill teaches Claude (or any coding agent) the patterns that work in this market. It is built from real client work, not from tutorials.

## Install

### Claude Code

```bash
# Add to your project
git clone https://github.com/datawithusman/business-data-automator.git
cp business-data-automator/SKILL.md .claude/skills/business-data-automator.md
```

Or install globally:

```bash
mkdir -p ~/.claude/skills
curl -sL https://raw.githubusercontent.com/datawithusman/business-data-automator/main/SKILL.md \
  -o ~/.claude/skills/business-data-automator.md
```

### Cursor

Save `SKILL.md` as `.cursor/rules/business-data-automator.mdc` in any project where you do client work.

### OpenCode / Windsurf / others

Copy the contents of `SKILL.md` into your tool's rules or system prompt file. The format is plain markdown with YAML frontmatter.

## How to use

After installing, just describe what you are working on:

```
I have a client running a delivery fleet in Riyadh.
They track 200 riders on Excel. Sheets include:
- Deliveries (date, rider, client, status)
- Payments (week, rider, amount)
- Issues (date, rider, type)

They want a dashboard and daily KPI email.
```

Claude will use this skill to:

1. Ask the right profiling questions
2. Propose a schema based on the business type
3. Write the cleaning code with tested patterns
4. Pick a dashboard stack that fits their budget
5. Build the daily KPI email automation

## What is inside

The SKILL.md file covers:

| Section | What you learn |
|---|---|
| Data profiling | How to read a messy Excel file before writing code |
| Cleaning patterns | Tested pandas functions for dates, currency, categories |
| Schema design | How to listen to a client and design tables they understand |
| Dashboard architecture | 3 stack options with real trade-offs |
| Automation recipes | Daily KPI email, late alerts, weekly Excel sync |
| Client types | Fleet, retail, clinic, construction playbooks |
| Pricing | Real ranges for 2026 freelance market |
| Anti-patterns | What kills these projects before they start |

## Who this is for

- Freelancers doing data work for small businesses
- Consultants moving clients from Excel to real systems
- Agency owners selling automation services
- Developers building internal tools for non-tech teams
- Anyone who has been handed a 50-sheet Excel file and asked to "make it work"

## Who this is NOT for

- Enterprise data engineering (use dbt + Snowflake + Airflow instead)
- Real-time streaming pipelines (use Kafka + Flink)
- Data science and ML projects (different skill)
- Building a generic SaaS product (this is for client work)

## About the author

Muhammad Usman. I run [Data With Usman](https://github.com/datawithusman), a data automation agency for non-tech B2B businesses. Before that I managed operations for a 200-rider delivery fleet serving Talabat, where I learned these patterns the hard way.

I also teach Python as a Stanford Code in Place Section Leader and build AI systems at Nobel AI.

- GitHub: [@datawithusman](https://github.com/datawithusman)
- Site: [datawithusman.com](https://datawithusman.com)

## Contributing

If you have a cleaning pattern, dashboard recipe, or client-type playbook that is not covered here, open a pull request. Keep additions short and based on real work, not theory.

See [CONTRIBUTING.md](CONTRIBUTING.md) for details.

## License

MIT. Use it for client work, paid projects, your own agency. No attribution required (but appreciated).

## Other skills in this series

- [business-data-automator](https://github.com/datawithusman/business-data-automator) - This repo
- [rag-production-setup](https://github.com/datawithusman/rag-production-setup) - Production RAG systems with FastAPI + LangChain
- [ml-pipeline-builder](https://github.com/datawithusman/ml-pipeline-builder) - End-to-end ML pipelines from data to API