<!-- source_session: 2026-05-30_youtube-to-vault-pipeline -->

# How I Taught My Vault to Read YouTube

*2026-05-30 · vault, ai, automation, tools · Vault as OS*

There is a category of learning that only happens on YouTube. Conference talks. Long-form technical walkthroughs. Founders explaining how they built something. These are not things you find in documentation or blog posts. They exist on video, and that means — unless you do something deliberate — they exist nowhere in your knowledge system.

I had no good answer for this. I would watch something, take a few notes in a scratch pad, and lose most of it within a week. The knowledge was not making it into the vault.

The fix took one script and one skill file.

---

## Why yt-dlp and Whisper

yt-dlp is a command-line video downloader with a specific capability that mattered here: it can pull auto-generated captions directly, without downloading the video file. For most YouTube content with English captions, that is fast — the caption file arrives in a few seconds, no audio download needed.

Whisper, from OpenAI, is a speech-to-text model that runs locally. When captions are not available — or when they are in the wrong language — Whisper transcribes the audio file.

Two other tools on my list were Ollama and n8n. Ollama runs language models locally; I already have that covered through Hermes and OpenRouter. n8n is a visual workflow orchestrator; my vault already runs on scheduled Claude routines and LaunchAgents. Both were redundant. yt-dlp and Whisper were not.

---

## What I Built

The pipeline is two files.

**The script** lives at `~/.claude/scripts/youtube-to-vault.sh`. It takes a YouTube URL and an optional tag. The flow is:

1. Clear any output from previous runs before starting — this prevents a failed run from poisoning the next one
2. Fetch the video title via yt-dlp
3. Try to pull auto-generated captions (`en.*` to catch `en-US`, `en-GB`, and other English variants)
4. If captions exist, clean the VTT format: strip timestamps, metadata lines, and duplicate phrases from the sliding caption window
5. If no captions, download the audio and run Whisper with the base model
6. Write the result to `/tmp/yt_output_{video-id}.md` with a frontmatter block containing the title, URL, tag, and transcription method — using the video ID in the filename prevents concurrent agent runs from clobbering each other

**The skill file** lives at `~/.claude/skills/youtube-to-vault/SKILL.md`. It teaches every vault agent when to invoke this pipeline, how to read the output, and how to format the vault entry. The agent — not the script — generates the summary. The script handles the mechanical work; the agent handles the interpretation.

The output lands in today's daily note, not a fixed folder. A YouTube video is not always learning material. Sometimes it is research. Sometimes it is consulting context. The tag at invoke time (`#learn`, `#research`, `#consulting`, `#finance`) determines how it is routed. Daily notes are already the capture hub for everything else.

---

## The Bugs Vera Found

Before I declared this done, I ran Vera — the QA persona in my vault — across the full implementation. She found three P0 issues.

**VTT metadata leaking into transcripts.** YouTube's VTT caption format includes lines like `Kind: captions` and `Language: en` that survive basic header stripping. My original filter removed `WEBVTT` at the top but left these behind. Every caption-sourced transcript would have started with `Kind: captions Language: en` before the actual content.

**Stale output file poisoning.** The script writes to a temporary path keyed to the video ID (`/tmp/yt_output_{video-id}.md`). The original used a fixed path, `/tmp/yt_output.md`. If a run failed mid-way, the file from the previous successful run remained at that path — a failed run on video B would cause the agent to summarise video A into today's note. Two fixes applied: the output file is deleted at script start, and the video ID is used in the filename to prevent concurrent agent runs from clobbering each other.

**`en` matching only `en`, not `en-US`.** The original script used `"en"` as the language selector — which did not match `en-US` or `en-GB` tagged captions. The result: the script would silently find no captions, fall through to Whisper, and spend several minutes transcribing audio that had perfectly good captions available. The fix is `"en.*"` — one character change that catches all English variants.

All three were in the script. None required a redesign. But all three would have produced incorrect vault entries in ways that would have been hard to notice after the fact.

---

## How Agents Know to Use It

Once the script and skill existed, I wired them to the vault's agent system. I added an auto-invoke rule to the vault's root `CLAUDE.md`:

> Any YouTube URL shared with intent to capture ("transcribe this", "save this", "take notes on this", "watch this") → invoke `youtube-to-vault` skill. All personas must route here — never transcribe manually.

I also added an entry to `Knowledge/skills-registry.md`, which is the file both orchestrators in the vault read to dispatch tasks. The skill is now a first-class capability any of the six vault personas can invoke without me explaining the pipeline each time.

---

## What This Changes

The friction of capturing a YouTube video used to be high enough that I rarely did it deliberately. Watch, take a few notes, move on. Now the friction is: paste the URL, name the tag, confirm. The agent handles everything else — download, transcription, summarisation, vault entry.

The more important shift is that YouTube content is now in the same system as everything else. The daily note becomes the landing zone regardless of whether the source is a video, an article, a meeting, or a product decision. One system, one search surface, one place to review what you captured last week.

---

## The Pattern Under the Build

The system prompt for this build was not "automate YouTube transcription." It was "where does learning go when you watch something important, and why does it not make it to the vault?" The answer was not a missing tool. It was a missing bridge between a passive medium and an active knowledge system.

The script is the bridge. The skill is the instruction manual the agents use to cross it.

---

## Related

- [From Manual to Automatic: How I Built a Vault OS That Runs Itself](vault-os-that-runs-itself.md) — the LaunchAgent automation layer this pipeline plugs into
- [Three Claude Tools, One Vault: The Architecture Behind the System](three-claude-tools-one-vault.md) — the vault architecture that makes skills and agents work together
- [Building an 11-Agent AI Team: From Isolated Agents to Coordinated Personas](building-an-ai-agent-team.md) — how the six vault personas that now use this skill were designed

---

*This post was distilled from a working session in my Obsidian vault. I build products with AI tools and write about the systems behind the work. [All posts](../README.md)*

<!-- FACT SOURCES
- "three P0 bugs" — verified: ~/.claude/scripts/youtube-to-vault.sh lines 17 (rm -f), 40 (en.*), 51-52 (Kind:/Language: filters)
- "six vault personas" — verified: Aw Vault/CLAUDE.md Persona Routing table: Sage, Alex, Maya, Priya, Vera, Rex
- Script path — verified: /Users/aw/.claude/scripts/youtube-to-vault.sh (exists, chmod +x)
- Skill path — verified: /Users/aw/.claude/skills/youtube-to-vault/SKILL.md (exists)
- Auto-invoke rule — verified: Aw Vault/CLAUDE.md line 99 verbatim
- skills-registry entry — verified: Aw Vault/Knowledge/skills-registry.md id: youtube-to-vault block
-->

