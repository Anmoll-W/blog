<!-- source_session: 2026-07-19_layover-launch -->

# Layover: Guess Where You Woke Up

*2026-07-19 · product, travel, security, shipping*

Layover went live today at layover.certifiedlost.com. It is a daily browser game with one premise: you wake up in an unknown city and have to figure out where you are before you run out of clues. No map, no country list to scroll through, just what the place feels like at that moment, revealed one piece at a time.

## The shape of the game

Every puzzle opens on a single clue, always the vaguest one on purpose: what the city sounds and smells like in that first waking moment, deliberately written so it could describe several cities at once. You get one guess. Guess wrong and a second clue unlocks, this time about cost of living relative to the rest of the country. Wrong again, and food. Then street life. Then culture. The sixth and last clue is the giveaway, something specific enough that most players will place the city immediately, if they have made it that far without solving it already.

Six clues, six guesses, and a warmth signal after each wrong one, telling you whether your last guess landed closer to the answer than the one before it or further away. Guess a city on the wrong continent twice in a row and the game flags it, so you are not left guessing blind about how far off you are. You can also give up and reveal the answer once you have seen at least three clues, trading the win for closure.

The puzzles are pulled from a bundle of roughly a thousand cities and rotate on a fixed daily schedule, one city per day, the same puzzle for everyone playing that day, the way Wordle popularized. Today, launch day, is city zero.

## Why a geography game, and why now

Every existing daily geography game on the market tests the same skill. Can you recognize a country's silhouette, its flag, its borders on a blank map. That is map knowledge. Layover is deliberately testing something else, what a place feels like to actually be there, the sensory and economic texture of a city rather than its outline. It is a different genre wearing the same daily puzzle clothes.

That distinction is also the reason it sits under Certified Lost rather than as a standalone brand. Certified Lost already runs a Telegram bot that helps groups plan real trips together. Layover is not that product, but it shares the same instinct: build things around the specific, unglamorous texture of travel, the parts a map cannot show you. Waking up somewhere unfamiliar and having to work out where you are is not a metaphor here so much as the entire mechanic. The name of the brand is the premise of the game.

## The bug that shipped on launch day

The first version of Layover stored every puzzle as a flat file bundled straight into the website, the same static approach Wordle popularized. No server, no database, just JSON shipped to the browser. That works well for a puzzle that has already unlocked. It does not work for puzzles that have not unlocked yet, because a bundled file is a bundled file. Anyone who opened their browser's developer tools before a future day's puzzle went live could read that day's answer and every one of its clues directly out of the page source, days ahead of when they were supposed to see any of it.

That got fixed today, on launch day itself, before the first real puzzle was live to the public. The fix moves the answer and the unrevealed clues off the browser entirely. A small server now holds each day's puzzle in memory and only ever sends the browser the pieces of it a player has actually earned by guessing: the current clue, and nothing past it. Every guess gets checked on that server, not in the browser, so there is no longer a copy of the answer sitting anywhere a player could inspect it early. The gating logic was also changed to fail closed by default, so a misconfigured deploy cannot accidentally unlock every future puzzle at once instead of just today's.

It is a small architectural change with an unglamorous name, server side validation, but the practical effect is the one that matters for a daily puzzle: the day's answer genuinely does not exist in a player's browser until the round is over. For a game whose entire loop depends on nobody being able to peek ahead, that guarantee was worth rebuilding the storage layer for, on the same day the game went live.

## Related

- [One Sheet I Can Trust](one-sheet-i-can-trust.md): another shipped system where the real vulnerability was not caught by functional tests, only found once the question changed from does it work to how would someone abuse it.
- [Live Testing Revealed the Bot Was Fundamentally Broken](live-testing-revealed-broken-bot.md): the earlier chapter of the Certified Lost brand's shipping story, and a reminder that the failures a single-account build never sees are the ones real players find first.
