# Methodology

Notes on how to use devour well. Not about what to review. About how to review.

The spine (the 12 principles) is stable. The tactics (from Layer 4) evolve. The exemplars accumulate. This file catches a third thing: **disciplines of the reviewer**. Habits that make findings credible, fixes that actually fix, and the failure modes of review itself.

New entries go here when real use exposes a review-discipline failure mode, not a design-principle one. The distinction: anti-patterns live in `anti-patterns.md` when they describe failures in the reviewed work. They live here when they describe failures in the *act of reviewing*.

---

## Speculative fixing without observation

### What it is

A reviewer applies a finding's suggested fix, the fix doesn't resolve the symptom, and the reviewer applies a second speculative fix ... then a third ... without once observing the actual runtime state (DOM, console, network, process, data). Three fixes in, the bug is still there and the reviewer is now three steps further from understanding the real cause than when they started.

The shape is recognizable: `try A → didn't work → try B → didn't work → try C → didn't work → *now* inspect`. By the fourth attempt, the reviewer has accumulated unused code changes, a cluttered git history, and a weaker understanding of the system than they started with.

### Why it fails

**The fix is a claim.** When a finding says "add `requestAnimationFrame` to defer navigation," that claim rests on an underlying model of what the system is doing. If the symptom persists after the fix, one of two things is true: the model was wrong, or the fix didn't land where the reviewer thought it did. Either answer requires observation to establish. Speculative fixing skips the observation step and pattern-matches instead.

**Pattern-matching compounds errors.** Each speculative fix is based on the same flawed model that produced the previous speculative fix. The second fix is not an independent attempt; it inherits the assumptions of the first. Three speculative fixes in a row are three rounds of the same assumption, not three genuinely different hypotheses.

**The reviewer loses trust in their own tools.** If you ship three fixes and the bug persists, the question isn't "what do I try next?" It is "what am I not seeing?" That question is answered by inspection, not by imagination.

### Fix direction

**After one failed fix, stop and observe.** Not after three. One failed attempt is already the signal that the model behind the fix is wrong. Run the app. Open the console. Inspect the DOM. Log the actual state. Compare observed state to the model the fix was based on. Adjust the model before writing more code.

**For portal-rendered UI specifically,** the observation is usually: what is the actual `data-state` attribute on the overlay after the action? Is it what React thinks it is? If not, React state and DOM state are out of sync... that's the real bug, not the thing the first fix targeted.

**For navigation-related bugs specifically,** the observation is: is the component that owns the state still mounted after the navigation? If not, cleanup of that state will never run. No amount of cleanup ordering will fix unmount-during-cleanup.

**For any bug:** the first failed fix is the cheapest signal that the model is wrong. Take it seriously.

### Related

- **Principle #4** (reversibility is craft) applies at the review level too... a fix that doesn't verifiably reverse the symptom is not a reversible claim, which makes it not craft.
- Every devour skill should be run with a browser-driving MCP available (chrome-devtools, Playwright, Puppeteer). The observation step is what the browser MCP exists for.
- The finding format in review output names the symptom, principle, tactic, and reference. The implicit fifth field is *verification*: did the tactic actually resolve the symptom? Terse mode doesn't include a verification field, but the discipline applies in both terse and verbose modes.

### Origin

This entry was added after a real debugging session in which the reviewer applied three speculative fixes (`requestAnimationFrame` deferral, `setTimeout` deferral, hoisting the parent component into root layout) to a close-on-navigate bug before inspecting the DOM. The actual cause (one component containing a portal overlay was rendered three times, opening three parallel dialogs, only one of which received the close signal) would have been visible in seconds via a single DOM query. The three fixes were reverted; the real fix was an architectural change informed by one minute of observation.

---

## Composing devour with impeccable

Devour and [impeccable](https://impeccable.style) are designed for different jobs. Impeccable has 23 commands: 21 MAKE design, 2 EVALUATE (`critique` and `audit`). Devour does design review with citations... it competes with `/impeccable critique` on the citation axis, not with the makers.

Use `/impeccable critique` for heuristic-scored UX review. Use devour when you want every finding to trace to a named source from the design lineage.

The two compose on the execution side: run devour to identify principle violations, pick a matching `/impeccable <subcommand>` to apply the fix. Devour names the principle; impeccable executes the change.

Recommended flow for a serious craft pass:

1. **Set up project context once per repo:**
   - Run `/impeccable teach` to create `PRODUCT.md` and `DESIGN.md` (audience, register, brand voice, anti-references, design system tokens).
   - Run `/devour-teach` to create `.devour-context.md` (principle weighting, motion appetite, density preference).

2. **Review with devour:**
   - `/devour` for a full-spine review.
   - `/devour-motion`, `/devour-micro`, `/devour-state` for focused depth.
   - Every finding cites a principle and a source. Devour writes a run file to `.devour/runs/`.

3. **For findings you decide to apply, pick the matching `/impeccable` subcommand:**
   - Principle #9 (reduce decoration) → `/impeccable distill`
   - Principle #12 (type is a system) → `/impeccable typeset`
   - Principle #1 / #2 (motion) → `/impeccable animate`
   - Principle #10 (density) → `/impeccable layout`
   - Principle #4 / #7 (state handling) → `/impeccable harden`
   - Principle #8 (affordances) → `/impeccable polish`

4. **Commit with a message that names the principle and the impeccable command used.** Example: `Apply devour #9 via /impeccable distill on StatCard`.

Devour alone tells you what's wrong and why. Impeccable alone tells you what to make. Devour + impeccable closes the loop from principle to code.

**Not required:** devour works standalone. If you don't have impeccable installed, findings still cite principles and sources; the tactic fix is yours to execute however you want.
