# Series: Vault as OS

How I turned an Obsidian vault into an automated knowledge system powered by AI agents.

The system today runs seven automated agents on schedule, routes every daily note capture to its destination without manual sorting, coordinates an 11-persona AI team across 14 projects, and compresses its own knowledge base weekly to stay lean as it grows.

This series documents how each layer was built, what broke along the way, and what the architecture looks like now.

> **System:** Built around an Obsidian vault. See [Anmoll Wadhwa](https://github.com/Anmoll-W) for context on the full toolchain.

## Posts in This Series

0. [Three Claude Tools, One Vault: The Architecture Behind the System](../posts/three-claude-tools-one-vault.md)
   The starting point: how three Claude interfaces (mobile, desktop, code) share one vault as a common memory layer. Covers the context cascade, folder-level skills, and the inbox processing pattern.

1. [From Manual to Automatic: How I Built a Vault OS That Runs Itself](../posts/vault-os-that-runs-itself.md)
   The automation layer: three LaunchAgents running morning briefings, inbox processing, and weekly reviews on schedule. Covers why CronCreate failed and why macOS LaunchAgents were the right answer.

2. [Building an 11-Agent AI Team: From Isolated Agents to Coordinated Personas](../posts/building-an-ai-agent-team.md)
   The coordination layer: replacing isolated agents with a flat team that shares a knowledge brain. Covers the shared team brain architecture, auto-invocation via BOOT, and how Harper prevents agent proliferation.

3. [How I Retired Notion in One Session](../posts/how-i-retired-notion.md)
   The consolidation: migrating all Notion content to the vault in one session, deleting tasks files, and fixing 23 stale path references. Why everything that Claude Code needs to read must live as Markdown.

4. [From Identity Files to Persona Stubs: Redesigning the Vault Agent System](../posts/persona-layer-architecture.md)
   The v3 redesign: replacing 12 full identity files loaded on every session with a stub layer plus on-demand depth loading. Covers the five approaches evaluated and why Approach E won.

5. [The Eval Agent: Adding a Quality Gate to an AI Workflow](../posts/the-eval-agent.md)
   The evaluation layer: adding a 13th persona whose only job is to stress-test output. Covers the rubric system, why producing agents cannot eval their own work, and the blind-spot question pattern.

6. [Scheduling Claude: Expanding a Vault OS from 3 Automated Routines to 7](../posts/second-automation-layer.md)
   The second automation layer: expanding from three scheduled agents to seven, adding Chrome-gated analytics sync, daily task migration, Monday content dispatch, and quality gates on code review and blog publishing. Covers the sleep-gap bug, guard clause patterns, and why monthly operations belong inside weekly runners.

7. [My AI Agents Had Identity. They Needed Methodology.](../posts/wiring-claude-skills-to-agents.md)
   The skills layer: installing six external Claude skills from GitHub and wiring them into the agent system. Covers the auto-invoke vs. situational distinction, how each of the three stateless agents changed, and why the QA agent had been starting from the wrong assumption.

8. [My AI Agents Were Running. They Just Weren't Working.](../posts/self-correcting-agents.md)
   The self-correction layer: what happens when you audit a 13-agent system and find five aren't doing their jobs. Covers the difference between a protocol and a reminder, how self-correcting behaviors are baked into agent identity, and why "operational" is a claim that needs evidence.

9. [Agents That Do Not Learn: Rebuilding the Self-Improvement Layer from First Principles](../posts/agents-that-dont-learn.md)
   The compounding layer: how verbal self-reflection (Reflexion Block), a user profile mined from 60 sessions, memory decay flags, path-scoped rules, and explicit agent frontmatter turned a system that accumulated sessions into one that accumulates intelligence.

10. [The Agents Were Ready. The Coordination Was Not.](../posts/skill-chaining-agent-orchestration.md)
    The orchestration layer: how 13 isolated agents got a coordination contract. Covers the central-vs-declarative-vs-signal architecture decision, the two-orchestrator design (live + auto), the shared skills registry, Vera's pre-mortem that caught five production failure modes before implementation, and the Handoff contract that makes agent-to-agent chaining mechanical rather than hopeful.

11. [The Vault Was Organized. The Files Were Not.](../posts/vault-structural-drift.md)
    The hygiene layer: a nightly automation that scans every Projects/ folder for misplaced files, moves them to their correct locations (patching wikilinks first), creates missing index.md stubs, and appends a report to a safety log. Covers the auto-move vs alert-only decision, the wikilink patching problem, and the three-layer visibility chain.

12. [The Boot Sequence Was in the Docs. So Was Every Skipped Step.](../posts/claude-code-boot-sequence-as-infrastructure.md)
    The enforcement layer: analyzing 104 Claude Code sessions to find three recurring failure patterns (skipped boot, shallow audits, wrong format), then moving the session boot sequence from CLAUDE.md documentation into a SessionStart hook that fires on every session. Covers the five new workflow rules, five slash commands, and the difference between instructions that remind and infrastructure that enforces.

13. [The Dashboard Was Lists. The Hub Is a Board.](../posts/kanban-board-as-project-hub.md)
    The navigation layer: replacing a vertical Dataview dashboard with a Kanban board as the vault's central hub. Covers the status-vs-project columns decision, colored project tags, wikilinked cards as navigation pointers, the dual-layer task system (board for strategic, daily notes for operational), and how the weekly-review runner prunes the board automatically every Sunday.

14. [AI Runners That Remember](../posts/ai-runners-that-remember.md)
    The memory layer: six scheduled runners with persistent cross-run memory. Covers the three-piece system — a per-runner memory store with a `## Strategy for next run` behavioral handoff, a spaced repetition queue that surfaces past mistakes at decision time, and a 2am sleeptime consolidator that adds strength signals to patterns.md. All three are implementable with file I/O and the Claude Code CLI, no SDK required.

15. [The Eval Layer Caught Me Violating My Own Rules](../posts/the-eval-layer-caught-me.md)
    The discipline layer: installing Karpathy's four AI coding principles across thirty CLAUDE.md files at three levels (global, workspace, project) — and watching the eval agent catch a naming inconsistency, a duplicate file, and Tier B bloat that violated Simplicity First. Covers the cascade-vs-gradient decision, why discipline scales differently than rules, and why every system that makes decisions needs an eval layer that catches its own drift.

16. [We Designed a Multi-Model Router. Then We Asked One Question.](../posts/right-model-wrong-problem.md)
    The routing layer (that didn't get built): a full session of multi-model architecture research, a Vera decision eval, a Gemini adversarial BLOCK overturned by a 15-minute CLI test — and the one question that reduced the whole system to two runner modifications and one new file.

17. [The Claims Were There. The Sources Were Not.](../posts/claims-without-sources.md)
    The provenance layer: reading PM Brain OS, identifying the 20% the vault was missing, and what a Vera decision eval caught before it shipped. Covers conservative-floor provenance tagging, hypothesis lifecycle tracking vault-wide (four projects), the 7d/30d staleness system, and the /prep ritual that surfaces hypothesis-decision conflicts before stakeholder meetings.

18. [Four Agents Went RED for Three Weeks. I Cut the Team in Half.](../posts/four-agents-went-red.md)
    The consolidation: thirteen personas down to six after three consecutive RED weekly evals exposed memory-based rules as non-load-bearing. Covers the Brain Protocol (token-bounded retrieval, ReAct loop, mandatory gate blocks, validated memory writes), the skills routing matrix that replaced sole-ownership, and the audit pattern that catches stale references the original sweep missed.

19. [Upgrading My AI Agent Roster, and the Baseline I Forgot to Save](../posts/personas-180-of-180-and-no-baseline.md)
    The depth upgrade on top of the six-persona roster: dissent protocol, Anmoll-watchlist, banned-phrase list, per-persona refusal catalogs and failure-mode lists, and a 6-axis structural eval. Score: 180/180. Catch: no pre-upgrade snapshot existed, so the behavioral delta is unmeasurable. Ships a 47-line `persona-snapshot.sh` so the next upgrade can be A/B tested.

## What Is Coming

- The knowledge compression system: how the vault keeps its own files lean with weekly archival and monthly summarization

15. [How I Taught My Vault to Read YouTube](../posts/youtube-to-vault-pipeline.md)
    A yt-dlp + Whisper pipeline that turns any YouTube URL into a vault note. Captions-first (under five seconds for most videos), Whisper fallback for audio-only transcription, Claude summarises the transcript. Auto-invoke wired to all six vault personas via the skills registry — any agent can trigger it without manual instruction. Three P0 bugs caught by Vera before ship: VTT metadata leaking into transcripts, stale output file poisoning the next run, and a one-character language selector mismatch that silently bypassed captions on English videos.

---

*[All posts](../README.md) · [Anmoll Wadhwa](https://github.com/Anmoll-W)*
