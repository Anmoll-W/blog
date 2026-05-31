# Series: Hermes as a PA

Building Hermes — an always-on body for my Obsidian vault. A Telegram personal assistant and a set of scheduled jobs running on a small rented Linux server, built one hardened problem at a time.

The vault is the brain and Claude Code is the hands, but both only work when I am at the keyboard. This series documents giving them a body that acts while I am away: safely, reliably, and one wave at a time. Each wave solves a complete slice of the problem and is reviewed by two adversarial AI personas before I trust it.

> **System:** Built around an Obsidian vault plus the NousResearch Hermes agent on a Hetzner server, routed through OpenRouter. See [Anmoll Wadhwa](https://github.com/Anmoll-W) for the full toolchain.

## Posts in This Series

1. [Hermes, Wave 1: Giving My Vault an Always-On Body](../posts/hermes-the-foundation.md)
   The foundation: a safe data boundary (a deny-by-default synced slice of the vault), a single-writer rule that stops sync conflicts, a fail-loud harness so no scheduled job dies silently, and dead-man's-switch watchdogs. Plus the first two features you can feel — a morning digest and a reminder loop you can text.
