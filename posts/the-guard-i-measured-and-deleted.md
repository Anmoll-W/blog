<!-- source_session: 2026-07-25_substack-bulk-upload -->

# The Guard I Built, Measured, and Deleted

*2026-08-09 · automation, ai-coding, tooling, safety*

I had 151 posts sitting as drafts on a publication, plus one already published that the input file was wrongly carrying as a draft, with cover images and infographics rendered and waiting on disk. There is no supported way to set a draft's cover image or embed an image programmatically. Doing it by hand in the editor is fine for five posts. It does not survive 151.

So I built a command line tool with Claude to do it. Zero dependencies, Node 18.3 or later, one CSV in, images inserted into drafts. The interesting part of the build was not the automation. It was the check I was most confident about, which I measured and then deleted the same day.

## What the tool actually talks to

The publication dashboard calls three undocumented endpoints on the publication's own subdomain. One uploads a single image and returns a content delivery network URL. One fetches a draft, including the body as a stringified rich text document. One replaces that body.

That third one is the whole risk model. It is a full document replace, not a patch. You read the body, you modify it in memory, and you write the entire thing back. Get it wrong and the previous version of the post is gone.

Two platform facts cost real time to establish. An image only renders if the image node is wrapped in a captioned image node. A bare image node is accepted by the write and then silently stripped, so the post looks correct in the payload and empty in the editor. And the cover is derived from the first captioned image node in the body. There is a topImage attribute that looks exactly like the thing that selects a cover. It is not. I verified that against a live published post that had a populated cover and contained zero topImage nodes. Prepending the image as the first node is what does it.

Wrong facts about a platform outlive the code that contains them, which is why both of those live in the repository as comments rather than in my head.

## Every guard fails closed

The tool writes to live content, so every check refuses by default. A response missing a field the tool relies on skips the row rather than treating the absence as permission.

It writes only when the interface reports that the post is explicitly not published. An earlier version tested whether the post was published, which meant an undefined field read as false and passed the check. That is failing open, and it took a real row to notice.

It skips a draft with a schedule attached, and it checks the schedules array rather than the post date field, because the post date stays null until a post actually sends. Testing the date alone was a genuine bug and not an academic one.

It is idempotent. Any draft that already leads with an image block is left alone, and the check deliberately does not compare the image source, so a draft where a different lead image was placed on purpose is not second guessed.

It writes the pre-write body to disk before the write, not after. A full document replace has no undo.

After writing, it fetches the draft again and asserts three things: the node count grew by exactly the number of nodes inserted, the lead image source is the URL that was just uploaded, and every pre-existing node survived unchanged. Any failure names the backup file to restore from.

And three consecutive failures aborts the batch. An expired session or a changed interface fails every row identically, and firing 152 doomed requests at a host sitting behind a content delivery firewall is its own way to earn a real block.

## The guard I deleted

The one thing none of those checks cover is whether the right image is going on the right post. Each of them validates the mechanics of the write. None validates the meaning of it. The draft identifier is inherited on trust from whatever join produced the input file, and a wrong but valid identifier writes successfully and verifies successfully, in green, with a backup file.

The obvious fix is to compare the live post title against the slug in the filename. I built that. Then I ran it against all 152 rows before trusting it.

It flagged 111 correct rows as mismatches.

The reason is not subtle in hindsight. The titles had been rewritten editorially and share almost no vocabulary with the slugs they came from. The draft whose slug reads breadboarding and fat marker sketches is titled "Every Button Has a Shadow. Nobody Asked if a Screen Was the Right Answer." Nothing in a string comparison recovers that.

A guard that is wrong 73 percent of the time gets switched off within a day of being turned on. That is bad on its own. What makes it worse is that it also trains the operator to skim past the guards that are right, and the rest of the tool is built out of those. So it was deleted, and the mapping is printed instead: a read only preflight pass that fetches every draft and prints the live title next to the cover filename it is about to receive, next to whether the row would be written and why not. No uploads, no writes.

That preflight is now the only check on the highest severity defect in the tool, and reading it is a human job. The documentation says so in both places a person is likely to look.

## What the preflight found

The read only sweep of all 152 rows reported 149 that would be written and three that would not. Each of the three is a different guard doing its job.

One row pointed at a post that was already published. That was a defect the input file had been carrying, and it is precisely why the published check now fails closed rather than open. One row pointed at a draft that already led with an image, so the idempotency check left it alone. One row had no cover path at all, which the generator that built the file had already warned about loudly instead of silently dropping the row.

The same sweep produced the other useful number. The platform rate limits these endpoints and tells you nothing about it: no retry header, no rate limit headers, just a bare 429. Fired back to back, the first 49 requests returned 200 and everything after that was intermittent, roughly one success in three. That reads like a bucket of about 50 that refills slowly. So the client self throttles to one request every 1200 milliseconds by default and backs off five seconds, then fifteen, then forty five. The backoffs print, because a run that silently pauses for forty five seconds looks exactly like a hung process and gets killed halfway through a batch.

A 403 is deliberately never retried. A dead session is a real answer, and retrying it four times per row turns a rate limit problem into a blocked account.

## The failure that shaped the layout

The first version wrote the cover and the infographic as one leading block. That opens a post with two full width graphics stacked back to back before a single word of the article, and that is how the drafts were written on the first pass, before the layout was repaired.

The fix is in the tool now. The cover leads. The infographic is placed before the first heading that has body text ahead of it, and with no such heading it goes last, because appending is recoverable and guessing a slot in the middle of a document is not.

## What I am carrying forward

A safety check is not free because it is cheap to write. It has to be measured against real data before it is trusted, and the measurement has to happen before it ships, not after it starts crying wolf. My confidence in the title check was entirely unearned, and the only reason it did not survive is that running it against 152 live rows took less time than arguing about it.

The second thing is smaller and matters more day to day. When an automated check cannot be made correct, the honest move is to print the evidence and name the human who has to read it, not to ship a weaker version of the check and let a green result stand in for a decision nobody made.

The tool still has open items. Alt text is written as null on every image, on a channel where search visibility is the point of publishing. That is a known gap, logged rather than quietly carried.

## Related

- [The Claims Were There. The Sources Were Not.](claims-without-sources.md): the same problem one layer up. Output that is confidently shaped and structurally unverifiable until something forces the check
- [Every Status Was Green. Three of Them Were Lying.](every-status-was-green.md): a green result standing in for a verification that never ran, which is exactly what a wrong but valid draft identifier produces here
- [I ran the stats hoping to prove it worked. It did not, and that was the deliverable.](null-result-was-the-deliverable.md): the companion case for measuring your own idea before you commit to it, and accepting the answer when the measurement says no
- [When the AI Fix is Wrong: What Senior Review Catches That Pattern Matching Misses](when-the-ai-fix-is-wrong.md): the same gap between a fix that passes and a fix that is correct
- [My Agents Were Calling Skills That Did Not Exist](agents-calling-skills-that-do-not-exist.md): the case where measuring a guard argued for keeping it and changing what it checked, rather than deleting it. Its first two versions manufactured their own false positives before the measurement was worth anything
- [A Real Problem Is Not a Reason to Build](a-real-problem-is-not-a-reason-to-build.md): the same measure-your-own-work discipline pointed at a feature rather than a guard, where the usage numbers argued for retiring the thing their author had just proposed relocating
- [The Detector Scored Who Wrote It, Not How It Was Written](who-wrote-it-not-how-it-was-written.md): the same habit taken to an AI detector, where ten credits of controlled tests overturned the plan I walked in with and pointed at a different lever entirely

---

*This post was distilled from a working session in my Obsidian vault. I build products with AI tools and write about the systems behind the work. [All posts](../README.md)*
