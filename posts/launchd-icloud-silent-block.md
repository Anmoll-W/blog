---
date: 2026-04-14
tags: [engineering, silent-failures, macos, launchd, automation]
series: silent-failures
---
<!-- source_session: 2026-04-14_vault-automation-audit-and-fix -->

# launchd and iCloud: The Silent Block That Stopped Every Scheduled Agent

The plist was loaded. The script was executable. The schedule was correct. `launchctl list` showed the job was registered. And the agent was doing nothing at all.

I had four automations scheduled through macOS launchd, each invoking a Claude CLI session against my Obsidian vault: a morning briefing, an inbox processor, a weekly review, and a LinkedIn analyst. The system was supposed to run itself. I had written about that system. I had told people it was running.

Then I went to look at the outputs and realised I had been assuming more than I could prove.

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

## Related

- [Three Cascading Bugs: Module-Level SDK, Scroll Overflow, Invisible Font](three-cascading-bugs.md) — another silent failure class where a clean build was not evidence the page worked
- [Missing Viewport Tag: The Silent Root of All Mobile Failures](missing-viewport-tag.md) — a rendering-layer failure that produced zero error output, same "nothing to react to" category
- [From Manual to Automatic: How I Built a Vault OS That Runs Itself](vault-os-that-runs-itself.md) — the automation layer whose runners this audit was fixing
- [Building an 11-Agent AI Team: From Isolated Agents to Coordinated Personas](building-an-ai-agent-team.md) — the agent team whose members these launchd schedules are supposed to invoke

---
*This post was distilled from a working session in my Obsidian vault. I build products with AI tools and write about the systems behind the work. [All posts](../README.md)*
