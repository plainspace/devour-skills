<!-- markdownlint-disable MD024 -->
<!-- Duplicate ### subheadings (What it is / Why it fails / Fix direction / Related / Origin) are intentional: every entry follows the same structure so readers can scan and compare. -->

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

## Line-citation accuracy: name what's at the line

### What it is

A finding cites `file:line`, the line number is correct, but the symptom paragraph names the wrong identifier or element at that line. The reviewer read the file, saw a numeric pattern (a `radius="full"` value, an inline style, a magic number), tagged it with the surrounding component name they remembered from earlier in the file, and moved on. The line is right; the name is wrong. The fix-implementer reads the symptom, looks at the line, and finds something different than the symptom describes ... at which point they either (a) lose trust in the entire review, (b) apply the fix to the wrong thing, or (c) spend ten minutes reverse-engineering what the reviewer actually meant.

The shape: the reviewer saw `radius="full"` at line 249, recognized "Avatar" as the dominant component in the file, named the finding "Avatar `radius='full'` at line 249." Reality at line 249 was a HeroUI `Button`, not an `Avatar`. Avatars elsewhere in the file uniformly used `radius="lg"`. The drift was real (a Button radius inconsistent with surrounding interactive-element radii), but it was named wrong and the principle was misframed.

### Why it fails

**`[code-confirmed]` is a claim about reading.** The tag means: "I opened the file, I read the line, I am naming what's on it." If the reviewer can't name the element type or identifier at the cited line, they didn't read it ... they remembered it. Findings sourced from memory of the file rather than the line itself misframe the symptom roughly half the time on long files (>500 lines), more on files with multiple component types interleaved.

**Fix-implementers trust line citations more than narrative.** A finding paragraph with hedged language ("the avatar shape might drift") gets re-read; a finding with a specific line number ("Avatar at comment.tsx:249") gets acted on. The line number is the load-bearing claim. If it doesn't match what's at the line, the entire finding's credibility collapses ... including any unrelated findings in the same review.

**The misframe is invisible to the reviewer.** Devour can't catch this on its own; the line numbers are syntactically valid and the finding format reads correct. Only re-opening the file with the symptom in hand surfaces the mismatch. Discipline at write-time is the only defense.

### Fix direction

**When a finding cites file:line, the symptom paragraph must name the actual identifier or element at that line.** Specifically: the JSX element tag name as it appears in the source (`<Button>`, `<Avatar>`, `<motion.div>`, `<Drawer>`), the function name (`handleScroll`, `useAlbumPlaybackGate`), or the variable name (`isAuthed`, `lastScrollTop`). The conceptual category alone ("Avatar," "the kebab," "the date kicker") is not enough ... that's a remembered category, not a re-read fact. Name the source-level identifier so the fix-implementer can match the symptom to the line without re-deriving the reviewer's intent.

**The `[code-confirmed]` tag means: "I read the line AND I named the source-level identifier at the line."** Before tagging `[code-confirmed]`, ask the explicit question: "Did I just open the file at this line, or am I remembering it from earlier in the review?" If the answer is "remembering," re-open and re-read. The reviewer's remembered category may be correct (the line really is the element they think it is), but more often on long files (>500 lines) or files with multiple component types interleaved, the remembered category is one-element-off from what's actually at the line. Re-reading is the only defense.

**A finding without a verifiable line citation is still a finding ... it just becomes a register-level observation, not a code-level one.** Re-frame as: "the post-detail chassis drifts toward HeroUI defaults vs the editorial primitives at `community-page.tsx:923-948` (ArtistLikedStamp pattern)" rather than a false-precision claim like "Avatar `radius='full'` at line 249." The register-level observation is honest about its scope and forces the fix-implementer to do the line-level identification, which is appropriate when the reviewer didn't.

**Concrete reframe template.**

- Before (bad): `Avatar radius='full' at line 249 mixed with radius='lg' elsewhere.`
- After (good): `HeroUI Button radius='full' at comment.tsx:249 (collapse-toggle Button); UserAvatars elsewhere uniformly use radius='lg'. Drift: Button radius doesn't match other interactive-element radii in the editorial chassis.`

The "after" version names what's actually at the line (HeroUI Button, collapse-toggle), establishes the comparison surface (UserAvatars elsewhere), and frames the drift correctly (interactive-element radii in editorial chassis, not avatar shape). The fix-implementer can act on this without re-deriving the reviewer's intent.

### Related

- **`[code-confirmed]` is also a claim about reading.** See the finding-annotation rules in each review skill's Step 3.
- **Speculative fixing without observation** (above) is the fix-implementer mirror of this: applying a tactic without observing whether the tactic landed where the model said it would. Line-citation accuracy is the upstream version: naming what's at the line so the implementer doesn't have to derive it.

### Origin

This entry was added after the Sleeve chrome rebuild devour run on `(with-cname)+/p.$slug.tsx` (2026-05-04, run file `2026-05-05T010621-devour-frontend-app-routes-with-cname-p-slug.md`). Finding F1 cluster (f) cited `comment.tsx:249` for "Avatar `radius='full'` mixed with `radius='lg'` elsewhere." CC verification revealed line 249 was a HeroUI Button (the collapse-toggle), not an Avatar; UserAvatars in the file uniformly used `radius="lg"`. The line was right; the name was wrong; the principle ("avatar shape consistency") was misframed; the actual drift ("interactive-element radius consistency in editorial chassis") was missed.

---

## Tactic-as-candidate: verify semantic equivalence before asserting a swap

### What it is

A finding proposes "swap component A for component B," where A and B share visual register or appearance but differ in semantic intent. The reviewer saw component A producing a wrong-feeling output, recognized component B as the editorial-register equivalent, and asserted the swap as the tactic. The fix-implementer applies the swap, ships a finding-derived change, and only later (or never) discovers that A and B were doing different jobs ... the swap shipped a wrong meaning to users, or removed a needed signal entirely.

The shape: a `Sparkle` Badge on the post-author byline (semantically: "this account is the campaign owner / artist") was flagged as drift because the editorial register uses `ArtistLikedStamp` (semantically: "this account liked this thing"). The reviewer named the tactic as "swap Sparkle Badge for ArtistLikedStamp." The two components share visual register (small absolutely-positioned glyph on an avatar) but encode different signals. The right tactic was REMOVE the Badge (the campaignOwner signal was already conveyed by the post's authorship; the Badge was decorative drift), not swap it.

### Why it fails

**Visual register is not semantic equivalence.** Two components that look similar can encode different facts about the data model. ArtistLikedStamp answers "who liked this?"; Sparkle-on-author answers "is this the artist?"; a swap conflates the two and ships either a wrong signal (the artist's own posts now appear to be "liked by the artist") or a removed signal (the campaignOwner indicator is gone with no replacement).

**Asserted tactics get applied.** A devour finding that says "Tactic: swap A for B" reads as a directive. Impeccable runs, the swap happens, the finding closes. Whereas a finding that says "Tactic candidate: A and B share visual register but differ in semantic intent ... verify before applying" forces the implementer to check the data model, which is exactly the right move when semantic equivalence is unverified.

**The reviewer doesn't always have the data-model context.** Devour reads code, but a single component's role in the data graph (what props it derives from, what fact about the user/content it encodes) often lives in adjacent files, GraphQL types, or product docs that the reviewer didn't load into context. Asserting swap tactics across that knowledge gap produces fixes that look right and ship wrong.

### Fix direction

**When proposing a tactic that involves swapping component A for component B, verify A and B share semantic intent, not just visual register.** Semantic equivalence means: A and B encode the same fact about the data model, drawn from the same kind of source, conveyed for the same purpose. If A encodes "campaignOwner" and B encodes "likedByArtist," they are not semantically equivalent ... regardless of how similar they look.

**If semantic equivalence is unclear from the code in front of the reviewer, flag the tactic as a candidate, not an assertion.** Use the explicit phrasing `Tactic candidate: ... verify semantics before applying.` Name what would need to be verified (the data model source for A vs B, the user-facing meaning of each, the consequence of the swap if semantics differ) so the implementer knows what verification means.

**REMOVE is a valid tactic candidate.** When the reviewer flags a component as drift but isn't sure what should replace it, "REMOVE" is often the right candidate: if the existing user differentiation pattern (e.g., a `__typename` branch in adjacent components, a separate badge for the same fact rendered elsewhere) already covers the signal, the component is decorative and should be removed rather than swapped. Surfacing REMOVE as an option forces the implementer to check whether the signal is needed at all.

**Concrete reframe template.**

- Before (bad): `Wrong metaphor; editorial register uses ArtistLikedStamp (heart, not sparkle). Tactic: swap Sparkle Badge for ArtistLikedStamp.`
- After (good): `Wrong visual register: Badge uses bg-slvPrimary which competes with primary CTAs. Semantic intent: campaignOwner ("authored by artist"), distinct from ArtistLikedStamp's "liked by artist." Tactic candidate: REMOVE the Badge if existing user differentiation (e.g. PostCommentUser __typename branch) already covers the campaignOwner signal, OR find an editorial-register equivalent if the campaignOwner signal is genuinely needed. Verify semantics before applying.`

The "after" version names the actual visual problem (slvPrimary competing with CTAs), names the semantic distinction (campaignOwner vs likedByArtist), surfaces both candidate paths (REMOVE vs swap-to-equivalent), and forces the verification step.

### Related

- **Principle #11 (match the metaphor to the medium)** is the principle-level claim this discipline operationalizes. The principle says metaphor must match medium; this discipline says: when proposing a metaphor swap, verify the two metaphors encode the same semantic before asserting the swap.
- **Speculative fixing without observation** (above) is the runtime mirror: applying a tactic without observing whether the tactic landed correctly. This entry is the design-time mirror: asserting a tactic without verifying the semantic equivalence that justifies it.

### Origin

This entry was added in the same Sleeve chrome rebuild review as the line-citation entry above (run file `2026-05-05T010621-devour-frontend-app-routes-with-cname-p-slug.md`, F1 cluster (c)). The finding asserted "swap Sparkle Badge for ArtistLikedStamp" as the tactic. Verification revealed the two components encoded different signals (campaignOwner vs likedByArtist); the right move was REMOVE the Badge entirely (the campaignOwner signal was already conveyed by post authorship). Asserting the swap without verifying semantic equivalence would have shipped a wrong meaning ("the artist liked their own post") to users.

---

## Composing devour with impeccable

Devour and [impeccable](https://impeccable.style) are designed for different jobs. Impeccable has 23 commands: 21 MAKE design, 2 EVALUATE (`critique` and `audit`). Devour does design review with citations... it competes with `/impeccable critique` on the citation axis, not with the makers.

Use `/impeccable critique` for heuristic-scored UX review. Use devour when you want every finding to trace to a named source from the design lineage.

The two compose on the execution side: run devour to identify principle violations, pick a matching `/impeccable <subcommand>` to apply the fix. Devour names the principle; impeccable executes the change.

Recommended flow for a serious craft pass:

1. **Set up project context once per repo:**
   - Run `/devour-teach` to create `DEVOUR.md` at repo root. devour-teach opportunistically reads `PRODUCT.md` and `DESIGN.md` if they exist (impeccable's files), and pre-fills register, audience, and brand voice from them. For multi-surface projects (e.g., marketing site + product app in one repo), pass `--surfaces` to declare per-path-prefix overrides.

2. **Review with devour:**
   - `/devour` for a full-spine review.
   - `/devour <prose description>` to route by keyword.
   - `/devour-motion`, `/devour-micro`, `/devour-state` for focused depth.
   - `--register brand` or `--register product` to override the project default for a single run.
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
