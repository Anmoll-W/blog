<!-- source_session: 2026-04-14_vault-automation-audit-and-fix -->

# launchd and iCloud: The Silent Block That Stopped Every Scheduled Agent

*2026-04-14 · engineering, silent-failures, macos, launchd · Silent Failures*

I had been telling people the vault runs itself. The morning brief, the inbox processor, the weekly review, a LinkedIn analyst that surfaces what the feed is missing — all scheduled, all unattended, all working. I had written about this system. I had described it out loud as running.

When I went to actually check the outputs, the agent that mattered most for distribution had never produced a single artifact. Not once. The plist was loaded, the schedule was correct, and the system had been silently failing for as long as I had been describing it as working.

## The Audit

The first question was the obvious one: which of these agents is actually producing artifacts? I opened the logs directory and the recent session folder and started matching schedules to outputs.

The morning briefing had some outputs. The inbox processor had some. The weekly review had some. The LinkedIn analyst had none. Not a single report in the reports folder. Not a single log line. No evidence it had ever run.

I pulled up the plist for the analyst and read the ProgramArguments. It pointed at a shell script at a path I did not recognise. I went looking for the script. It did not exist. It had never existed. I had written the plist at some earlier point, registered it with launchctl, and never written the runner it was supposed to invoke. launchd had been firing the schedule, failing silently at exec, and moving on. There was nothing to watch because there was nothing to see.

That was the first finding. The second took longer.

## The Second Finding

The three agents that had produced some output were not reliable either. When I ran one of them manually with `launchctl start` to verify, I got "Operation not permitted" in the log. The file was there. `ls -l` showed it as executable. I could run it from the terminal by typing its path. launchd could not.

EPERM at exec is a message that sends debugging in the wrong direction. It reads like a permission bit problem, so you check chmod. chmod is fine. Then it reads like an ownership problem, so you check owner. Owner is fine. Then you start looking at System Integrity Protection and code signing and Transparency Consent and Control, which is where I spent a chunk of the session.

The actual cause was that every runner script lived inside my Obsidian vault, which is synced through iCloud Drive. That choice had felt natural at the time. The scripts belonged to the vault, so they lived with the vault. Files in iCloud-synced folders carry extended attributes and sync state that a regular filesystem does not. When launchd tries to exec one, the exec path is strict enough that the attempt fails and returns EPERM. A shell running interactively has enough privilege to read through the iCloud layer. launchd does not.

I could not find a clean diagnostic that said "this is an iCloud issue." What I could find was that the moment I copied the script to a local path outside iCloud and repointed the plist, it ran immediately.

## The Fix

Three changes, all of them mechanical once the cause was clear.

First, move every runner script out of the vault to `~/.claude/vault-runners/`, which is on local disk. Strip any lingering extended attributes with `xattr -c`. Make the scripts executable. Move the log destinations out of iCloud as well, so the logs themselves stop being subject to the same rule.

Second, update every launchd plist ProgramArguments to the new path. Update StandardOutPath and StandardErrorPath to point at the local log directory. Unload the old plists and load the new ones.

Third, and this is the check I had been skipping, run every agent manually with `launchctl start` once, immediately after loading. Then verify the expected artifact actually appeared in the expected location. Not just "the job exited zero." The artifact.

All four ran. The morning briefing wrote its section into today's daily note. The inbox processor wrote its routing summary. The weekly review wrote its session file. The LinkedIn analyst, which I had to build from scratch in the same session, wrote its first weekly report and surfaced a 43 day repost drought I had not noticed by looking at the feed manually.

## What I Learned

Do not place executable scripts inside iCloud-synced folders if launchd has to run them. The error when it fails is EPERM, which does not point at iCloud, and the fix is not configurable. It is positional. Move the file.

A loaded plist is not evidence that the agent can run. `launchctl list` only proves the plist parsed and registered. It proves nothing about whether the executable at ProgramArguments[0] exists, is reachable, or is allowed to exec in its current filesystem context. Every new LaunchAgent needs a manual `launchctl start` and an artifact check the first time it is loaded, not just "it is registered, we are done."

Silent success is almost as dangerous as silent failure. Three of my four agents were producing some output, so I assumed the system was healthy. The one producing no output was invisible because there was nothing to react to. Scheduled jobs need explicit absence-of-artifact checks, not just error watching. If a weekly report is supposed to land every Sunday, something in the system has to notice when it does not, because launchd will not.

The absence of a file is not a signal unless you are looking for it. I am adding the watchdog this week.

**"The system runs itself" is a claim, not a status — and claims require evidence.** I had been treating "running" as a description of the architecture rather than a verified state of the system. The LinkedIn analyst had a plist, a schedule, and a purpose, and it had never run once. Going forward: every system I describe as autonomous needs an explicit artifact check before I describe it that way out loud. Registration is not running. A launchctl list is not proof. The artifact is proof.

---

**[2026-04-14](../README.md#all-posts)** · [![engineering](https://img.shields.io/badge/engineering-586069?style=flat-square&logoColor=white)](../README.md#all-posts) [![silent-failures](https://img.shields.io/badge/silent--failures-d73a4a?style=flat-square&logoColor=white)](../README.md#all-posts) [![macos](https://img.shields.io/badge/macos-586069?style=flat-square&logoColor=white)](../README.md#all-posts) [![launchd](https://img.shields.io/badge/launchd-586069?style=flat-square&logoColor=white)](../README.md#all-posts) [![automation](https://img.shields.io/badge/automation-2ea44f?style=flat-square&logoColor=white)](../README.md#all-posts)

**Series:** [Silent Failures →](../series/silent-failures.md)

## Related

- [The Health Check Was Reporting to a File Nobody Read](a-stub-a-dead-model-and-a-health-log.md): what rotted in the seams after these jobs were relocated. A placeholder left as a no-op, a stale model pin, and a health log disconnected from its reader.
- [Three Cascading Bugs: Module-Level SDK, Scroll Overflow, Invisible Font](three-cascading-bugs.md) — another silent failure class where a clean build was not evidence the page worked
- [Missing Viewport Tag: The Silent Root of All Mobile Failures](missing-viewport-tag.md) — a rendering-layer failure that produced zero error output, same "nothing to react to" category
- [From Manual to Automatic: How I Built a Vault OS That Runs Itself](vault-os-that-runs-itself.md) — the automation layer whose runners this audit was fixing
- [Building an 11-Agent AI Team: From Isolated Agents to Coordinated Personas](building-an-ai-agent-team.md) — the agent team whose members these launchd schedules are supposed to invoke
- [Scheduling Claude: Expanding a Vault OS from 3 Automated Routines to 7](second-automation-layer.md) — what was built after this audit: four more agents, Chrome guard clauses, and a sleep-gap fix that came directly from what was learned here
- [I Was Burning 18 Claude Sessions a Day. Here Is What Found Them.](token-burn-audit.md) — a second-generation audit of the same system: done-file contract failures, Perl alarm timing on macOS, and zombie agents that survived their own self-disable logic

---
*This post was distilled from a working session in my Obsidian vault. I build products with AI tools and write about the systems behind the work. [All posts](../README.md)*
