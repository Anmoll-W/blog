<!-- source_session: 2026-04-17_gmail-support-system -->

# We Were Paying for a Support Tool. Gmail Already Did the Job.

*2026-04-17 · product, engineering, operations, saas*

We were paying for a shared inbox tool to manage support for a SaaS product receiving 6 to 10 tickets a day. The tool had shared views, assignment workflows, and collision detection. It was well-designed. It was also solving a problem we did not actually have at our volume.

The decision that created this situation was common: someone picked a professional support tool at the start, when the product had no users, on the assumption that it would scale. It did not need to scale. Six to ten tickets a day is not a volume problem. It is a routing and response problem. The routing and response problem has a free solution. We were not using it.

This post covers what we built when we stopped paying for the tool.

## What We Actually Needed

Before replacing anything, the question worth asking was: what job is the support inbox doing?

The answer had three parts. First, every incoming ticket needed to land with the right person — billing questions should not go to the developer, and technical errors should not go to the billing specialist. Second, the team needed to respond from a single unified address so customers never saw multiple senders. Third, common issues needed pre-written responses, because the same ten questions arrive over and over and re-typing them is a waste.

None of those requirements mandate a paid tool. They mandate filters, delegation, and templates. Gmail has all three.

The decision we made was to treat Gmail not as a basic email client, but as a configurable triage system. That reframe changed what we looked for when we opened the settings.

## The Build

**Auto-routing filters.** Seven filters now process every incoming ticket the moment it lands in the support inbox. Each filter matches on keywords and sender patterns — subject lines containing "charge" or "billing," body text mentioning specific error codes, emails from the payment processor domain — and applies a label and a forward rule. A ticket tagged Billing/Refund goes to the person who handles billing. A ticket tagged Technical goes to the developer. A ticket tagged Account/License goes to the activation team. No manual reading required to route it. The ticket arrives labeled and forwarded before anyone has opened the inbox.

This is the part people underestimate. Gmail filters are not just for personal inbox management. They are programmable routing rules that run on every incoming message. The only cost is the time to write them once. After that, the routing runs without touching.

**Gmail delegation.** Five team members can now read, compose, and reply from the shared support address. Customers see one address. The team sees one inbox. Replies from any team member appear to come from support. There is no confusion about who owns what channel, and no customer ever receives a reply from a personal address. Delegation is a single settings change that most Gmail users have never looked at.

**Canned response templates.** Ten templates cover the highest-frequency issues: surprise charges, cancellation requests, refund policy, license activation steps, AI feature credits setup, account transfer, billing cycle questions, technical error acknowledgments, and two escalation paths for issues that need a human decision. The templates are not form letters. Each one has a blank where the responding team member fills in the specific detail — the transaction date, the specific error code, the exact next step. The structure is fixed. The personalization is intentional. The structure removes the blank-page delay. The personalisation slot removes the robotic feel. A template that requires no thought also produces responses that feel like they required no thought.

## What This Replaced

The paid shared inbox tool handled the same three functions: routing (through manually assigning each ticket to a team member), unified address (through the shared inbox), and common responses (through saved replies). The difference is that those functions cost money in the paid tool and cost nothing in Gmail.

The honest tradeoff: the paid tool had better collision detection — two team members could see when the other was drafting a reply. Gmail does not have this natively. Our fix was a Slack message convention: when someone picks up a ticket, they drop the ticket subject into a shared Slack channel. It is manual. It takes three seconds. For six to ten tickets a day, it is more than sufficient.

The other tradeoff: reporting. The paid tool had built-in volume charts and response time metrics. We do not have that now. We also did not use it when we had it. That was the harder truth to admit.

## What I Learned

**Technical lesson.** Gmail filters accept multi-condition logic — sender domain, subject keywords, body text, and combinations of these. A filter that says "if the sender domain is [payment processor] AND the subject contains 'dispute'" is a one-line routing rule that previously required a paid tool to configure. The feature has been in Gmail for years. Most people have only ever used it to archive newsletters.

**Decision lesson.** The paid tool got selected before there was any data on ticket volume or type distribution. The reasoning was reasonable: we are building a serious product, we should use serious tools. But "serious" was doing work it had no business doing in that sentence. Serious tools match the problem, not the self-image. The right question before any tooling decision is: what is the actual job this tool needs to do, at actual volume, and what is the simplest thing that does that job? Starting there would have saved money from day one.

The system we have now is not a workaround. It is the right tool for the problem we actually have. Gmail is more configurable than most people give it credit for. At 6 to 10 tickets a day, it is the right answer.

If you want to replicate this: Gmail delegation is under Settings → See all settings → Accounts → Grant access to your account. The filters are under Settings → Filters and Blocked Addresses. Neither requires any plugin or third-party tool.

---

**[2026-04-17](../README.md#all-posts)** · [![product](https://img.shields.io/badge/product-0366d6?style=flat-square&logoColor=white)](../README.md#all-posts) [![engineering](https://img.shields.io/badge/engineering-586069?style=flat-square&logoColor=white)](../README.md#all-posts) [![operations](https://img.shields.io/badge/operations-2ea44f?style=flat-square&logoColor=white)](../README.md#all-posts) [![saas](https://img.shields.io/badge/saas-e4606d?style=flat-square&logoColor=white)](../README.md#all-posts)

## Related

- [Nobody Was Logging In: How I Deleted a Support Dashboard and Built a Cron Job Instead](dashboard-to-digest.md) — the same product: a paid support tool replaced with a simpler system, and the decision logic that made the replacement the right call
- [From 70 Reported Issues to 9 Root Causes: A Production Bug Sprint](support-tickets-root-causes.md) — what the support ticket data actually contained, which directly informed the routing categories we built
- [How to Audit a Production Codebase Against Its Own Support Data](auditing-plugin-against-support-data.md) — using support ticket patterns to inform engineering priorities, upstream of the triage system
- [Research Before Building: How I Map a Problem Space from Scratch](pm-problem-space-research.md) — the problem-framing approach that would have caught the over-tooling decision before money was spent

---
*This post was distilled from a working session in my Obsidian vault. I build products with AI tools and write about the systems behind the work. [All posts](../README.md)*
