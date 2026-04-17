<!-- source_session: 2026-04-17_ghostty-terminal-setup -->

# I Spent an Afternoon Making My Terminal Feel Alive

*2026-04-18 · developer-tools, ghostty, claude-code, terminal, workflow*

I had been using Mac's default terminal for years. It worked. That was the problem.

When a tool "works," you stop noticing it. You stop asking whether it should work better. You just open it, type, and leave. But the terminal is not a tool you visit occasionally — it is the environment you live in when you build. I was opening it dozens of times a day and never once questioning what it looked like, how it behaved, or whether it was doing anything for me beyond accepting input.

That stops being fine at some point. This is about when it stopped being fine.

---

## The Warp Decision

Everyone building with AI tools in 2026 has seen Warp. It is a beautiful product — modern UI, block-based output, GPU-accelerated, and a built-in AI layer that can suggest commands, explain errors, and run multi-step tasks from natural language.

I installed it. I uninstalled it the same afternoon.

Not because it was bad. Because its primary value proposition — the AI layer — was something I already had in a better form. I run Claude Code for everything: writing code, debugging, working across files, running shell tasks. Adding Warp's AI on top of Claude Code was not compounding value. It was redundancy I would pay for.

The product decision here is not "Warp is bad." It is: **before adding a tool, audit whether you already have the capability it sells.** Warp made sense for developers without an agentic AI workflow. For me, the terminal needed to be the shell. The intelligence was already handled elsewhere.

That decision pointed toward Ghostty.

---

## What Ghostty Is

Ghostty was built by Mitchell Hashimoto — the same person who built Terraform and Vagrant. Open source, free, GPU-accelerated via Metal on Mac, no account required:

```bash
brew install --cask ghostty
```

Split panes, native macOS tabs, scrollback search, full Nerd Font support, zsh shell integration. Fast. Native feel. Gets out of the way.

But getting out of the way is table stakes. The more interesting question was what I could build on top of it.

---

## The Part That Was Actually Fun

Ghostty supports custom GLSL shaders — GPU-rendered animations running continuously behind your terminal content. The community has built 33 of them: a galaxy spinning in the background, matrix rain, fireworks, a starfield, a cursor that leaves a fire trail, underwater ripples, CRT scanlines, lava, snow.

These run at 60fps on Metal. Text stays completely readable on top. The terminal just feels alive.

Ghostty also ships with over 300 built-in themes.

So I wrote a script that runs every time a new terminal window opens. It picks a random theme from the full list. Then a random shader from the 33. Writes both to the config and prints two lines:

```
🎨  Theme:  Kanagawa Wave
✨  Shader: sparks-from-fire
```

Every session is a surprise. Here is the full script:

```zsh
#!/bin/zsh
GHOSTTY_BIN="/Applications/Ghostty.app/Contents/MacOS/ghostty"
CONFIG="$HOME/.config/ghostty/config"
SHADERS_DIR="$HOME/.config/ghostty/shaders"

THEME=$($GHOSTTY_BIN +list-themes 2>/dev/null \
  | sed 's/ (resources)//' \
  | sort -R \
  | head -1)

SHADER=$(ls "$SHADERS_DIR"/*.glsl | sort -R | head -1)
SHADER_NAME=$(basename "$SHADER" .glsl)

sed -i '' "s|^theme = .*|theme = $THEME|" "$CONFIG"
sed -i '' "s|^custom-shader = .*|custom-shader = $SHADER|" "$CONFIG"

echo "🎨  Theme:  $THEME"
echo "✨  Shader: $SHADER_NAME"
```

Hook it into `.zshrc` so it only fires on the first shell of a new window:

```zsh
if [[ "$TERM_PROGRAM" == "ghostty" && "$SHLVL" == "1" ]]; then
  ~/.config/ghostty/random-theme.sh
fi
```

Shaders from [this community repo](https://github.com/0xhckr/ghostty-shaders). Clone to `~/.config/ghostty/shaders/`.

---

## The Relatable Part

There is a version of this afternoon that does not happen. The version where "it works" is enough and you close the tab and move on.

I have done that for years. Not just with the terminal — with every tool that was good enough. The calendar app, the note-taking system, the file manager. All of them were fine. None of them were mine.

The terminal is where I spend most of my working hours. It is open in the background of almost every session. I had never once asked whether it should feel like something.

---

## What I Learned

**Technical:** Ghostty's config is one plain text file. Custom shaders are GLSL fragments that take five minutes to wire up. The random selection logic is ten lines of shell. None of this required deep knowledge — it required giving yourself permission to look.

**Decision:** "Good enough" is a silent tax. You pay it in the small, cumulative friction of using tools you tolerate rather than tools you chose. The terminal was not broken. But I was spending thousands of hours in an environment I had never invested one afternoon in designing. That asymmetry is worth noticing.

If you build with AI tools, your terminal is your cockpit. Make it yours.

---

## Related

- [From Manual to Automatic: How I Built a Vault OS That Runs Itself](vault-os-that-runs-itself.md) — the same principle applied to a knowledge system: if you spend time in an environment, build it to work for you
- [My AI Agents Had Identity. They Needed Methodology.](wiring-claude-skills-to-agents.md) — Claude Code as the AI layer that made Warp's AI redundant; how your agent stack shapes every tool decision downstream
