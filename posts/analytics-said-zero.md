<!-- source_session: 2026-07-28_product-blog-batch -->

# Analytics said zero. The order system said otherwise.

*2026-07-28 · analytics, silent-failures, saas, debugging · Silent Failures*

Whether to keep investing in a market usually comes down to one question: is anyone actually buying. For four months in a row, the answer a regional pricing page on a SaaS product I work on gave in Google Analytics was no. Zero key events, zero transactions, every month, on the same page. By the time it showed up in a consolidated health report, it read less like a data quirk and more like a verdict: this market was not converting, and a go or no-go call on it was going to be made on that basis.

I want to be honest about how easy that number was to believe. It was consistent. It was not a one-off blip that could be explained by a tracking outage on a single day. It repeated across four consecutive reporting periods, on the same page, in the same analytics property. When a metric behaves that consistently, the instinct is to trust it and move on to explaining it, not to question whether it is measuring what you think it is measuring. The report did exactly that. It concluded, in writing, that this was the fourth straight month of zero real conversion signal for that market.

The catch started on a walkthrough call, not in a scheduled analysis. A colleague going through the report with me stopped on that section and did something the report itself had not done: instead of accepting the analytics number, he manually looked at the order system's admin panel and found real completed orders on several specific dates inside that exact window, dates the analytics platform had reported as having zero transactions. A follow-up pull the next day, this time using the order system's own built-in filter for the same payment gateway and date range rather than manual scrolling, confirmed it: multiple completed orders sat inside the window GA4 had reported as empty.

That is the moment the finding flipped. It was not that the market had no demand. It was that two systems measuring the same thing over the same period disagreed, and only one of them was being trusted by default.

The likely mechanism made sense once we looked for it, though it remained an inference from the pattern, not something we instrumented and proved directly. The order system was the ground truth: it recorded a payment as completed because money had actually moved through the gateway. The analytics platform was recording a different thing entirely: whether a browser session carrying an analytics client identifier reached a conversion event. If a customer's checkout flow involved a redirect through the payment gateway and that redirect did not preserve the analytics client identifier on the way back, the completed order would exist in the order system with no corresponding event in analytics. The order was real. The attribution link back to the session that produced it was probably broken the same way. In the roughly five-week window we actually checked, "zero signal" was not zero customers. It was a redirect likely dropping the thread that connects a sale to the session that generated it.

What made this dangerous is exactly what made it believable in the first place: analytics platforms do not fail loudly. A broken attribution link does not throw an error or flag itself as suspicious. It just reports a clean, confident zero, and a zero looks like data rather than like a gap. The report had, in fact, generated the ground-truth comparison data elsewhere in the same document, from the order system, but had never filtered that section by the same market to check it against the "zero real sales" claim before publishing it. The two pieces of evidence that would have caught this sat in the same report without ever being cross-referenced against each other.

The fix we adopted was narrow on purpose. It is not "audit everything" or "distrust analytics." It is a single standing rule: no claim of zero real conversion signal from analytics ships without a cross-check against the order system, filtered to the exact same segment and date window, first. If the order system agrees there were no completed transactions, the zero is real and a market-exit decision can be made on it. If it disagrees, the decision on the table changes entirely, from "wind this market down" to "fix the checkout flow and look again," and those two calls have very different costs to get wrong.

It is a cheap rule to apply and an expensive one to skip. The cross-check takes minutes because the order system already has the same date and segment filters available. Skipping it risks a report telling a team that a market is dead when the actual problem is a technical seam in a payment redirect.

The lesson I keep coming back to is that a zero from a single-source system is not evidence of absence, it is evidence of what that one measurement pipeline was able to see. Any metric that depends on a client-side signal surviving an external redirect, a third-party domain hop, or a cross-system handoff has a failure mode that produces a clean, confident, wrong zero, not a visible error. The discipline that catches it is not smarter analysis of the number you already have. It is refusing to let a single system's absence of a signal stand as proof of the underlying event's absence until a second, independently-sourced system has been asked the same question in the same terms.

## Related

- [Google Analytics Was Fine. Opening It Was Not.](google-analytics-daily-digest.md): another case where the GA4 data itself was not the villain. The process wrapped around reading it was
- [launchd and iCloud: The Silent Block That Stopped Every Scheduled Agent](launchd-icloud-silent-block.md): the same "the system reports nothing wrong because it cannot see the failure" pattern, in a scheduling layer instead of an analytics pipeline
- [The Metric Did Not Improve. The Denominator Changed.](the-denominator-changed.md): a companion case from the same reporting stack. A metric trusted at face value because a second system was never consulted to check it

---

*This post was distilled from a working session in my Obsidian vault. I build products with AI tools and write about the systems behind the work. [All posts](../README.md)*
