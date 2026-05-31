# Graph Report - /Users/aw/blog  (2026-05-14)

## Corpus Check
- 48 files · ~62,837 words
- Verdict: corpus is large enough that graph structure adds value.

## Summary
- 271 nodes · 351 edges · 11 communities detected
- Extraction: 93% EXTRACTED · 7% INFERRED · 0% AMBIGUOUS · INFERRED: 24 edges (avg confidence: 0.77)
- Token cost: 0 input · 0 output

## God Nodes (most connected - your core abstractions)
1. `My AI Agents Had Identity. They Needed Methodology.` - 23 edges
2. `Building an 11-Agent AI Team` - 15 edges
3. `When the AI Fix Is Wrong` - 14 edges
4. `Persona Layer Architecture (v3)` - 14 edges
5. `The Eval Agent (Vera)` - 13 edges
6. `Vault OS That Runs Itself` - 13 edges
7. `The Eval Layer Caught Me Violating My Own Rules` - 12 edges
8. `Agents That Do Not Learn` - 12 edges
9. `Second Automation Layer (3 to 7 routines)` - 11 edges
10. `Blog Index (Learning in Public)` - 10 edges

## Surprising Connections (you probably didn't know these)
- `Blog Index (Learning in Public)` --references--> `Series: Vault as OS`  [EXTRACTED]
  README.md → series/vault-as-os.md
- `Blog Index (Learning in Public)` --references--> `Series: GitHub is Your Portfolio`  [EXTRACTED]
  README.md → series/github-is-your-portfolio.md
- `Blog Publishing Protocol` --references--> `Blog Index (Learning in Public)`  [EXTRACTED]
  CLAUDE.md → README.md
- `GitHub Profile README Auto-Push` --references--> `Anmoll Wadhwa (GitHub Profile)`  [EXTRACTED]
  CLAUDE.md → README.md
- `Series: Vault as OS` --references--> `Anmoll Wadhwa (GitHub Profile)`  [EXTRACTED]
  series/vault-as-os.md → README.md

## Hyperedges (group relationships)
- **Silent Failures Pattern (no error, clean build, invisible until prod)** — concept_silent_no_op, concept_claude_extra_text_after_json, concept_loaded_plist_not_running, concept_playfair_turbopack_font_failure, concept_telegram_dm_constraint [EXTRACTED 0.90]
- **Vault as Shared Context Across Three Claude Surfaces** — concept_obsidian_vault, concept_context_cascade, concept_inbox_processing_pattern, concept_claude_dispatch_desktop_code [EXTRACTED 0.90]
- **Vault OS Memory Layer (cross-run persistence)** — concept_runner_memory_store, concept_strategy_for_next_run, concept_srs_queue, concept_sleeptime_consolidator [EXTRACTED 0.90]
- **Silent Failure Pattern Across Layers** — telegram_privacy_mode_post, missing_viewport_tag_post, when_the_ai_fix_is_wrong_post, building_internal_dashboard_silent_bugs [EXTRACTED 0.95]
- **Enforcement via Infrastructure (not documentation)** — claude_code_boot_sequence_session_start_hook, vault_structural_drift_auto_move, second_automation_layer_launchagent_pattern [EXTRACTED 0.90]
- **Dashboard-to-Digest Delivery Pattern** — dashboard_to_digest_post, google_analytics_daily_digest_post, gmail_support_system_post, building_internal_dashboard_post [EXTRACTED 0.90]
- **Vera eval layer applied across content, skills, discipline, and routing decisions** — the_eval_agent, the_eval_layer_caught_me, cherry_picking_cost_post, right_model_wrong_problem, multi_llm_gate [EXTRACTED 0.90]
- **Vault OS substrate: consolidation, automation, persona architecture** — how_i_retired_notion, vault_os, persona_layer, the_eval_agent [EXTRACTED 0.88]
- **Agent team evolution: identity -> methodology -> coordination -> self-correction -> learning** — persona_layer, wiring_claude_skills_to_agents, skill_chaining_agent_orchestration, self_correcting_agents, agents_that_dont_learn [EXTRACTED 0.92]

## Communities

### Community 0 - "Agent Learning & Memory"
Cohesion: 0.05
Nodes (48): anmoll-profile.md, Memory Decay 14-day staleness flag, Numbered Steps vs Paragraph Bullets (rationale), Path-Scoped Rules (.claude/rules/), Reflexion Block protocol, Reflexion paper (NeurIPS 2023 Shinn et al.), Agents That Do Not Learn, Persona Layer Architecture (v3) (+40 more)

### Community 1 - "Dashboard & Digest Pipeline"
Cohesion: 0.06
Nodes (45): Building Internal Support Dashboard, Rationale: Audit inherited scaffold assumptions, Four Silent Setup Bugs, SQLite-to-Postgres Migration, Supabase Two-Connection-String Pattern, Support Digest Cron Pipeline, Rationale: Deletion as a Product Decision, HelpScout v2 API Failure Modes (+37 more)

### Community 2 - "AI Agent Team Architecture"
Cohesion: 0.06
Nodes (42): BOOT as Routing Layer, Flat Team Structure (no hierarchy), Harper's 3-Evidence Rule, Building an 11-Agent AI Team, Rationale: Flat over hierarchical for <15 agents, Shared Team Brain, Deprecated Skill Names PreToolUse Hook, Rationale: Documentation vs Infrastructure Enforcement (+34 more)

### Community 3 - "Bug Investigations & Fixes"
Cohesion: 0.11
Nodes (30): Artifact Check (vs Error Check), Claude Appends Explanatory Text After JSON, Extract JSON By Bracket Position, grammY webhookCallback Empty 200 Behavior, In-Group Inline Keyboards (Preference Collection), launchd EPERM From iCloud-Synced Scripts, A Loaded Plist Is Not Evidence of a Running Agent, Module-Level SDK Initialization Build Failure (+22 more)

### Community 4 - "Vault & Workflow Patterns"
Cohesion: 0.1
Nodes (23): Active Column Cap of Three, Anthropic memory_20250818 Memory Tool, Cards as Wikilink Pointers, Three Claude Surfaces (Dispatch / Desktop / Code), Context Cascade (root + per-folder CLAUDE.md), Inbox Processing Pattern, Status Columns Over Project Columns, Letta Sleep-Time Compute (arxiv 2504.13171) (+15 more)

### Community 5 - "Multi-LLM Routing & Cost"
Cohesion: 0.1
Nodes (21): Cache Discipline rule, Cherry-Picking the Viral Cost-Cut Post, /effort per prompt rule, Quota Tier vs Price Tier, Subagent Model Routing, Three-Question Adoption Framework (rationale), Dispatcher / launch agent watcher, Four Review Modes (code/content/decision/research) (+13 more)

### Community 6 - "Terminal & Portfolio Setup"
Cohesion: 0.11
Nodes (20): Random Theme + Shader Script, 0xhckr/ghostty-shaders, Ghostty Terminal Setup, Warp vs Ghostty Decision (rationale), GitHub as Your Portfolio, Decision Log tool (archived), Stack as Positioning Statement, Tech Fluency Companion tool (archived) (+12 more)

### Community 7 - "Blog Publishing Protocol"
Cohesion: 0.18
Nodes (12): Blog Publishing Protocol, Bidirectional Cross-Linking Rule, Gemini Review Content Gate, GitHub Profile README Auto-Push, Slug-Only Filename Rule, Fixed-Width Markdown Constraint, Narrative Story Profile Format, Reject GitHub Stats Widgets (Vanity Metrics) (+4 more)

### Community 8 - "Parallel Agents & Spec Execution"
Cohesion: 0.18
Nodes (12): Rationale: Budget Day 3 for integration bugs, Parallel Agent Workstreams (8-day to 4-day), Shared Domain Constraints in CLAUDE.md, Tool Pruning Rule (addition requires subtraction), Git Worktree Per Agent Pattern, Live Testing Revealed Broken Bot (referenced), PM Problem Space Research (referenced), Pre-Model Verbatim Paste Check (+4 more)

### Community 9 - "Plugin Audit & Analytics"
Cohesion: 0.2
Nodes (10): 80/20 Audit Strategy, Support Category Distribution as Audit Entry, False Positive Discipline (rationale), Auditing a Plugin Against Support Data, Silent PHP Bug Pattern, Formula Hooks Kill the Metric They Optimize For, Performance Rule Expiry Condition, Maya marketing agent (+2 more)

### Community 10 - "PM Research & Archetypes"
Cohesion: 0.29
Nodes (7): Independent Research Rebuild From Scratch, PM Learning Companion Problem Statement, Six PM Archetypes (Mapper, Climber, Sprinter, Specialist, Coach, Scanner), Anchor Paid Tier on Sprinter Archetype, Research Before Building: PM Problem Space, Rationale: Independent Rebuild Grounds Agreement, Rationale: Deadlines Convert Better Than Goals

## Knowledge Gaps
- **170 isolated node(s):** `prompt-generator-skill`, `Slug-Only Filename Rule`, `Bidirectional Cross-Linking Rule`, `Gemini Review Content Gate`, `Rationale: Design Tested Only By Builder Is Not Tested` (+165 more)
  These have ≤1 connection - possible missing edges or undocumented components.

## Suggested Questions
_Questions this graph is uniquely positioned to answer:_

- **Why does `Building an 11-Agent AI Team` connect `AI Agent Team Architecture` to `Parallel Agents & Spec Execution`, `Dashboard & Digest Pipeline`?**
  _High betweenness centrality (0.050) - this node is a cross-community bridge._
- **Why does `launchd and iCloud Silent Block (referenced)` connect `Dashboard & Digest Pipeline` to `AI Agent Team Architecture`?**
  _High betweenness centrality (0.048) - this node is a cross-community bridge._
- **What connects `prompt-generator-skill`, `Slug-Only Filename Rule`, `Bidirectional Cross-Linking Rule` to the rest of the system?**
  _170 weakly-connected nodes found - possible documentation gaps or missing edges._
- **Should `Agent Learning & Memory` be split into smaller, more focused modules?**
  _Cohesion score 0.05 - nodes in this community are weakly interconnected._
- **Should `Dashboard & Digest Pipeline` be split into smaller, more focused modules?**
  _Cohesion score 0.06 - nodes in this community are weakly interconnected._
- **Should `AI Agent Team Architecture` be split into smaller, more focused modules?**
  _Cohesion score 0.06 - nodes in this community are weakly interconnected._
- **Should `Bug Investigations & Fixes` be split into smaller, more focused modules?**
  _Cohesion score 0.11 - nodes in this community are weakly interconnected._