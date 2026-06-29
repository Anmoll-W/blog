# Series: Hermes as a PA

Building Hermes — an always-on body for my Obsidian vault. A Telegram personal assistant and a set of scheduled jobs running on a small rented Linux server, built one hardened problem at a time.

The vault is the brain and Claude Code is the hands, but both only work when I am at the keyboard. This series documents giving them a body that acts while I am away: safely, reliably, and one wave at a time. Each wave solves a complete slice of the problem and is reviewed by two adversarial AI personas before I trust it.

> **System:** Built around an Obsidian vault plus the NousResearch Hermes agent on a Hetzner server, routed through OpenRouter. See [Anmoll Wadhwa](https://github.com/Anmoll-W) for the full toolchain.

## Posts in This Series

1. [Hermes, Wave 1: Giving My Vault an Always-On Body](../posts/hermes-the-foundation.md)
   The foundation: a safe data boundary (a deny-by-default synced slice of the vault), a single-writer rule that stops sync conflicts, a fail-loud harness so no scheduled job dies silently, and dead-man's-switch watchdogs. Plus the first two features you can feel — a morning digest and a reminder loop you can text.

2. [Hermes, Wave 2: The Tests Passed. The Code Was Broken Anyway.](../posts/hermes-wave-2-tests-that-lie.md)
   Consolidating a sprawl of scattered Mac automation into a handful of hardened, always-on jobs on the server — and the verification discipline that made it trustworthy. A two-reviewer adversarial gate caught five defects that green tests completely missed: a deploy tool that could not deploy its own code, a backup that would have wiped the target on failure, an idempotency ledger that could split and spam, a consolidated job that double-sent the morning digest, and two tests that passed against deliberately-broken code. Built and verified, going live after the current soak.

3. [Hermes, Wave 3: A Machine That Drafts From My Notes But Cannot Post](../posts/hermes-wave-3-the-machine-that-drafts.md)
   The always-on server starts to write: it drafts LinkedIn posts from existing vault signals onto a review bench, for two surfaces, and — by construction — cannot post a single one. A machine that ingests your own notes is a prompt-injection surface, and the adversarial review caught it: a note that could break out of its content wrapper into instruction space, a crafted identifier that could forge an "approved" status past the human gate, and collisions that could silently drop content — all with green tests. Safety comes from a human-approve gate the code cannot bypass, privacy-pinned no-log routing, and someone actively trying to break both.

4. [Hermes, Wave 4: Finance Intel From a Server That Cannot See My Finances](../posts/hermes-wave-4-finance-intel-off-server.md)
   Finance intel from a server deliberately blindfolded to all finance data. The math runs on the Mac that can see the accounts; only a sanitized verdict plus one headline number crosses to the always-on server, which delivers it and — enforced in code — can never trade or move money. Reviewed through three lenses (build, adversarial-security, finance-domain); the finance reviewer caught two numbers that lied comfortingly. Deployed; no soak yet.

**Companion — [How My Hermes Agent Works, From a PM's Point of View](../posts/building-a-personal-ai-ops-layer.md):** the map above the wave posts — the always-on-body problem, the three-layer architecture, guardrails as product constraints, and the value-first roadmap, with the PM takeaways behind the build.

**Conclusion — [Why I Shut Down Hermes — a Multi-Agent AI System I Built Myself](../posts/why-i-shut-down-hermes.md):** the honest accounting of the shutdown. Maintenance overhead exceeded value, the reflector loop never shipped, and Mac-side runners with Claude Code delivered the core value without the distributed layer. The foundation held — the overhead did not.
