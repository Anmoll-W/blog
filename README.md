# Learning in Public

Writing about systems, silent bugs, and shipping products with AI tools.

These posts come from real working sessions. Every post is distilled from something that actually happened: a bug that took too long to find, a decision that needed to be made, a system that needed to be rebuilt. Not tutorial content. Specific things that happened and what I learned from them.

<details>
<summary><b>Contents</b></summary>

- [Series](#series)
- [All Posts](#all-posts)
- [About](#about)
- [See Also](#see-also)

</details>

---

## Series

Series are groups of posts that build on each other. Start here if you want context before reading individual posts.

| Series | Posts | What It Covers |
|--------|-------|----------------|
| [Hermes as a PA](series/hermes-as-a-pa.md) | 1 | Giving the vault an always-on body — a Telegram assistant and scheduled jobs on a rented server, built one hardened problem at a time |
| [Vault as OS](series/vault-as-os.md) | 16 | Building an Obsidian vault into an automated knowledge system with AI personas, scheduled agents, and enforced session behavior |
| [Silent Failures](series/silent-failures.md) | 8 | Bugs that pass tests and fail in production — Claude API, Telegram, Next.js, WordPress, launchd |
| [GitHub is Your Portfolio](series/github-is-your-portfolio.md) | 3 | Why I archived a portfolio site and rebuilt my presence around GitHub — including Wikipedia-style READMEs |
| [Building ChalotripBot](series/building-chalotripbot.md) | 2 | From broken prototype to shipped product — live test failures and a parallel agent sprint |

---

## All Posts

*Newest first*

| Date | Title | Series | Tags |
|------|-------|--------|------|
| 2026-06-12 | [Every Status Was Green. Three of Them Were Lying.](posts/every-status-was-green.md) | Vault as OS | vault, ai, systems, debugging |
| 2026-06-10 | [Append-Only Is Where Lessons Go to Die](posts/append-only-is-where-lessons-go-to-die.md) | Vault as OS | vault, ai, systems |
| 2026-06-06 | [Your Agents Are Only as Good as the Context You Program Them With](posts/context-is-the-program.md) | Vault as OS | vault, ai-agents, systems, debugging |
| 2026-06-02 | [How My Hermes Agent Works — and What It Does for Me, From a PM's Point of View](posts/building-a-personal-ai-ops-layer.md) | Hermes as a PA | ai, systems, automation, vault |
| 2026-06-02 | [Hermes, Wave 4: Finance Intel From a Server That Cannot See My Finances](posts/hermes-wave-4-finance-intel-off-server.md) | Hermes as a PA | ai, systems, automation, finance |
| 2026-06-01 | [Hermes, Wave 3: A Machine That Drafts From My Notes But Cannot Post](posts/hermes-wave-3-the-machine-that-drafts.md) | Hermes as a PA | ai, systems, automation, vault |
| 2026-06-01 | [Hermes, Wave 2: The Tests Passed. The Code Was Broken Anyway.](posts/hermes-wave-2-tests-that-lie.md) | Hermes as a PA | ai, systems, automation, vault |
| 2026-05-31 | [Hermes, Wave 1: Giving My Vault an Always-On Body](posts/hermes-the-foundation.md) | Hermes as a PA | ai, systems, automation, vault |
| 2026-05-30 | [How I Taught My Vault to Read YouTube](posts/youtube-to-vault-pipeline.md) | Vault as OS | vault, ai, automation, tools |
| 2026-05-27 | [AI Credits Are Infrastructure. Start Treating Them That Way.](posts/ai-credits-are-infrastructure.md) | — | ai-tools, product-management, cost-thinking |
| 2026-05-25 | [Upgrading My AI Agent Roster, and the Baseline I Forgot to Save](posts/personas-180-of-180-and-no-baseline.md) | Vault as OS | vault, ai, systems, evals |
| 2026-05-24 | [Four Agents Went RED for Three Weeks. I Cut the Team in Half.](posts/four-agents-went-red.md) | Vault as OS | vault, ai-agents, personas, systems |
| 2026-05-20 | [The Claims Were There. The Sources Were Not.](posts/claims-without-sources.md) | Vault as OS | vault, knowledge-management, ai-agents, pm, obsidian |
| 2026-05-18 | [I Was Burning 18 Claude Sessions a Day. Here Is What Found Them.](posts/token-burn-audit.md) | Silent Failures | claude-code, automation, token-efficiency, silent-failures, launchagent |
| 2026-05-11 | [We Designed a Multi-Model Router. Then We Asked One Question.](posts/right-model-wrong-problem.md) | Vault as OS | ai-agents, multi-model, decision-frameworks, claude-code, gemini |
| 2026-05-10 | [Two Models, One Pipeline: Building a Multi-LLM Peer Review Gate That Caught Its Own Silent Bug](posts/multi-llm-peer-review-gate.md) | — | ai-agents, claude-code, gemini, multi-llm, eval |
| 2026-04-27 | [I Cherry-Picked a Viral Cost-Cut Post](posts/cherry-picking-the-cost-post.md) | — | claude-code, token-efficiency, decision-frameworks, ai-tools |
| 2026-04-27 | [The Eval Layer Caught Me Violating My Own Rules](posts/the-eval-layer-caught-me.md) | Vault as OS | ai, vault, claude-code, discipline, eval |
| 2026-04-23 | [AI Runners That Remember](posts/ai-runners-that-remember.md) | Vault as OS | automation, ai-agents, claude-code, memory, vault-os |
| 2026-04-22 | [Google Analytics Was Fine. Opening It Was Not.](posts/google-analytics-daily-digest.md) | — | engineering, product, ai, analytics |
| 2026-04-22 | [The Dashboard Was Lists. The Hub Is a Board.](posts/kanban-board-as-project-hub.md) | Vault as OS | vault, obsidian, kanban, systems, productivity |
| 2026-04-21 | [The Boot Sequence Was in the Docs. So Was Every Skipped Step.](posts/claude-code-boot-sequence-as-infrastructure.md) | Vault as OS | claude-code, automation, vault, ai-tools |
| 2026-04-21 | [The Vault Was Organized. The Files Were Not.](posts/vault-structural-drift.md) | Vault as OS | vault, automation, ai, obsidian, systems |
| 2026-04-19 | [The Agents Were Ready. The Coordination Was Not.](posts/skill-chaining-agent-orchestration.md) | Vault as OS | claude, obsidian, agents, orchestration, vault-os |
| 2026-04-19 | [Agents That Do Not Learn: Rebuilding the Self-Improvement Layer from First Principles](posts/agents-that-dont-learn.md) | Vault as OS | ai-agents, claude, agent-architecture, obsidian, self-improvement |
| 2026-04-18 | [I Spent an Afternoon Making My Terminal Feel Alive](posts/ghostty-terminal-alive.md) | — | developer-tools, ghostty, claude-code, terminal, workflow |
| 2026-04-17 | [We Were Paying for a Support Tool. Gmail Already Did the Job.](posts/gmail-support-system-without-paid-tools.md) | — | product, engineering, operations, saas |
| 2026-04-16 | [My AI Agents Were Running. They Just Weren't Working.](posts/self-correcting-agents.md) | Vault as OS | vault, ai, systems, agents, product |
| 2026-04-16 | [My AI Agents Had Identity. They Needed Methodology.](posts/wiring-claude-skills-to-agents.md) | Vault as OS | vault, ai, systems, agents, skills |
| 2026-04-15 | [Scheduling Claude: Expanding a Vault OS from 3 Automated Routines to 7](posts/second-automation-layer.md) | Vault as OS | vault, automation, ai, macos, launchagent, systems |
| 2026-04-14 | [launchd and iCloud: The Silent Block That Stopped Every Scheduled Agent](posts/launchd-icloud-silent-block.md) | Silent Failures | engineering, silent-failures, macos, launchd |
| 2026-04-13 | [Nobody Was Logging In: How I Deleted a Support Dashboard and Built a Cron Job Instead](posts/dashboard-to-digest.md) | — | engineering, product, ai, helpscout |
| 2026-04-13 | [The Eval Agent: Adding a Quality Gate to an AI Workflow](posts/the-eval-agent.md) | Vault as OS | vault, ai, systems |
| 2026-04-12 | [From Identity Files to Persona Stubs: Redesigning the Vault Agent System](posts/persona-layer-architecture.md) | Vault as OS | vault, ai, systems |
| 2026-04-10 | [Building an 11-Agent AI Team: From Isolated Agents to Coordinated Personas](posts/building-an-ai-agent-team.md) | Vault as OS | ai-agents, personas, vault |
| 2026-04-10 | [Missing Viewport Tag: The Silent Root of All Mobile Failures](posts/missing-viewport-tag.md) | Silent Failures | css, mobile, silent-bugs |
| 2026-04-10 | [Formula Hooks Kill the Metric They Optimize For](posts/formula-hooks-kill-metrics.md) | — | ai, systems, content |
| 2026-04-10 | [From Problem Space to Working Codebase in One Day](posts/spec-to-codebase-one-day.md) | — | engineering, product, ai |
| 2026-04-10 | [Why I Rebuilt My GitHub Profile as a Wikipedia Article](posts/wikipedia-style-readme.md) | GitHub is Your Portfolio | github, portfolio |
| 2026-04-09 | [When the AI Fix is Wrong: What Senior Review Catches](posts/when-the-ai-fix-is-wrong.md) | Silent Failures | code-review, ai-coding |
| 2026-04-09 | [Research Before Building: How I Map a Problem Space from Scratch](posts/pm-problem-space-research.md) | — | product, ai, pm |
| 2026-04-08 | [How I Retired Notion in One Session](posts/how-i-retired-notion.md) | Vault as OS | vault, systems, obsidian |
| 2026-04-08 | [GitHub as Your Portfolio: Why I Archived a Full Portfolio Site for a README](posts/github-as-your-portfolio.md) | GitHub is Your Portfolio | github, portfolio |
| 2026-04-08 | [Building the GitHub Profile README: Design Decisions and the One Constraint That Shaped Everything](posts/github-profile-rebuild.md) | GitHub is Your Portfolio | github, portfolio |
| 2026-04-08 | [How to Audit a Production Codebase Against Its Own Support Data](posts/auditing-plugin-against-support-data.md) | — | engineering, debugging |
| 2026-04-08 | [From 70 Reported Issues to 9 Root Causes: A Production Bug Sprint](posts/support-tickets-root-causes.md) | — | engineering, debugging |
| 2026-04-05 | [From Manual to Automatic: How I Built a Vault OS That Runs Itself](posts/vault-os-that-runs-itself.md) | Vault as OS | vault, automation |
| 2026-04-04 | [Claude Appends Text After JSON: A Silent Bug Across 8 API Call Sites](posts/claude-appends-text-after-json.md) | Silent Failures | claude-api, json, debugging |
| 2026-04-04 | [Telegram Bots Cannot DM Users Who Have Not Pressed Start](posts/telegram-bots-cant-dm.md) | Silent Failures | telegram, silent-bugs |
| 2026-04-01 | [Building an Internal Support Dashboard: From Broken Scaffold to Live Data](posts/building-internal-dashboard.md) | — | engineering, supabase, nextjs |
| 2026-03-31 | [Collapsing an 8-Day Build Into 4 Days With Parallel Agent Workstreams](posts/parallel-agents-ai-stack.md) | Building ChalotripBot | ai, engineering, product |
| 2026-03-30 | [Live Testing Revealed the Bot Was Fundamentally Broken](posts/live-testing-revealed-broken-bot.md) | Building ChalotripBot | engineering, testing |
| 2026-03-30 | [Telegram Privacy Mode: The Silent Setting That Broke Natural Language](posts/telegram-privacy-mode.md) | Silent Failures | telegram, silent-bugs |
| 2026-03-29 | [Three Cascading Bugs: Module-Level SDK, Scroll Overflow, Invisible Font](posts/three-cascading-bugs.md) | Silent Failures | engineering, silent-failures |
| 2026-03-25 | [Three Claude Tools, One Vault: The Architecture Behind the System](posts/three-claude-tools-one-vault.md) | Vault as OS | vault, ai, systems |

---

## About

Product manager. In 2024 I started building products with AI tools — by 2025 that became the primary way I work. I use Obsidian to manage everything: notes, sessions, decisions, agent personas, and automated workflows.

---

## See Also

- [Anmoll Wadhwa](https://github.com/Anmoll-W) — main profile, projects directory, full career
- [chalo-trip-bot](https://github.com/Anmoll-W/chalo-trip-bot) — AI Telegram bot; 6 posts in this blog cover its build and bugs 🔒
- [linkwhisper-plugin-ui](https://github.com/Anmoll-W/linkwhisper-plugin-ui) — WordPress plugin prototype; 4 posts cover work done with this codebase 🔒
- [linkwhisper-support-dash](https://github.com/Anmoll-W/linkwhisper-support-dash) — support dashboard; 3 posts cover the build and analysis sprint 🔒
- [prompt-generator-skill](https://github.com/Anmoll-W/prompt-generator-skill) — Claude Code skill for structured prompt generation; used for all Claude API calls in ChalotripBot

## Linked From

- [Anmoll Wadhwa](https://github.com/Anmoll-W) — linked in Writing section and Projects Directory
- [chalo-trip-bot](https://github.com/Anmoll-W/chalo-trip-bot) — linked in See Also
- [linkwhisper-plugin-ui](https://github.com/Anmoll-W/linkwhisper-plugin-ui) — linked in See Also
- [linkwhisper-support-dash](https://github.com/Anmoll-W/linkwhisper-support-dash) — linked in See Also
- [prompt-generator-skill](https://github.com/Anmoll-W/prompt-generator-skill) — linked in See Also

---

![Category: AI](https://img.shields.io/badge/Category-AI-7b2ff7?style=flat-square)
![Category: Engineering](https://img.shields.io/badge/Category-Engineering-24292f?style=flat-square)
![Category: Product](https://img.shields.io/badge/Category-Product-0a66c2?style=flat-square)
![Learning in Public](https://img.shields.io/badge/Learning%20in%20Public-2ea44f?style=flat-square)
