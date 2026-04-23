---
name: devour
description: "Lineage-aware design engineering review. Reviews changed code or a target component/page/flow against a stable spine of design principles drawn from Dieter Rams, Edward Tufte, Don Norman, Bret Victor, Bill Buxton, Loren Brichter, Andy Matuschak, the Linear team, Rauno Freiberg, Emil Kowalski, and others. Outputs specific, citable findings with fixes. Use when the user wants principled design review, polish that traces back to a source, or to raise the craft bar on a specific surface."
argument-hint: "[target file, component, page, or pattern] [--terse]"
user-invocable: true
license: Apache 2.0. See NOTICE.md for full attribution to the design lineage this skill stands on.
---

> Good design is thorough down to the last detail.
> ... Dieter Rams, principle 8

Devour is a principles-driven, lineage-aware design engineering review. It is opinionated, citation-heavy, and built for designer-engineers who want a second pair of eyes calibrated to a higher bar. Every finding traces back to a named principle and a real source. Nothing is left at "this looks better."

This skill does not invent. It compresses a body of work spanning 40+ years of design thought into a usable review heuristic.

---

## When to use

- The user wants design review on a specific component, page, or flow ("review this dialog," "look at the onboarding").
- The user just shipped or is about to ship and wants a craft pass.
- The user mentions one of: polish, taste, craft, design engineering, motion review, micro-interactions, principles review.
- The user references the lineage by name: Rams, Tufte, Norman, Rauno, Emil, Sonner, Vaul, cmdk, Linear, Arc, Vercel.

For *general* AI-aesthetic-removal polish, the [`impeccable`](https://github.com/pbakaus/impeccable) family of skills (`polish`, `animate`, `delight`, `bolder`, `harden`) is the right tool. Devour is for principles review with citations.

For accessibility audits, a checklist tool (`rams`, `web-design-guidelines`, `a11y-debugging`) is faster and more comprehensive than devour for that specific purpose. Devour will flag obvious a11y failures as part of principle #6 (ergonomics) but is not an a11y audit.

---

## MANDATORY PREPARATION

Before reviewing anything, devour requires project context. Different products live in different parts of the spine ... a marketing page values principles 9 and 12 most; a productivity app values 3, 4, and 7 most; a creative tool values 2 and 5 most. Without context, devour produces generic findings.

**If the project has not run `/devour-teach` yet:**

1. STOP. Do not proceed with review.
2. Tell the user: "Devour needs project context first. Running `/devour-teach` to set up."
3. Invoke `devour-teach`. Follow it through to completion.
4. Then return to this skill.

**If the project has already run `/devour-teach`:**

A `Devour Context` block will exist in `.devour-context.md` (the canonical location) or, if the consumer has linked it, in their loaded project instructions (`.claude/CLAUDE.md`, `.cursorrules`, etc.). Read it before doing review. The context names which principles weight highest for this product and which exemplars are most relevant.

---

## Process

### Step 0 ... Check for --terse flag

Read `$ARGUMENTS`. If `--terse` is present, set output mode to **terse**. Strip `--terse` from the arguments before passing them to Step 1. Terse mode keeps the same rigor and the same spine but strips teaching prose from each finding. The APPLY? prompt and INTERACTIONS BETWEEN FINDINGS block are unchanged in both modes.

### Step 1 ... Establish target

If `$ARGUMENTS` is provided (after stripping `--terse`), the target is that file, component, pattern, or URL. Read it.

If `$ARGUMENTS` is empty:
- Default to **changed files in the current branch** (`git diff main..HEAD --name-only`, filtered to design-relevant files: `.tsx`, `.jsx`, `.vue`, `.svelte`, `.css`, `.scss`, `.html`, `.astro`).
- If the current branch has no changes, ask the user what to review.

For each target file, you will need:
- The current code
- Any related design system tokens or shared components
- Any visible production state (running dev server URL, screenshot, deployed preview)

**If the user has a running dev server**, prefer driving it via the `chrome-devtools` MCP to see actual behavior. Static code review misses motion and interaction issues that the spine specifically catches. State this explicitly in the output ("Reviewed code only; recommend re-running with dev server for motion findings").

### Step 2 ... Apply the spine

The spine is twelve principles. Each principle has tactics (positive patterns) and anti-patterns (failure modes). The full treatment lives in `references/principles/<principle-N>.md` ... read these as you encounter findings.

For each principle, scan the target for both:
- **Anti-pattern matches** ... the failure mode the principle is designed to catch
- **Tactic gaps** ... places where the right tactic is missing

The twelve principles, in spine order:

1. **Honest motion** ... if you animate it, it must communicate something the static state could not
2. **Physics over duration** ... real movement has mass, springs, damping; eased durations betray themselves
3. **Commit on intent, not on contact** ... distinguish hover-passing-through from hover-with-intent
4. **Reversibility is craft** ... every optimistic state needs a believable error path
5. **Sequence carries meaning** ... stagger only when order matters
6. **The fingertip and the cursor are not the same** ... Fitts, hit boxes, ergonomic distance
7. **Preserve user state across boundaries** ... loading must not lose your scroll, your selection, your draft
8. **Make affordances visible without making them loud** ... signifiers should be discoverable, not declarative
9. **Reduce decoration, increase information** ... every element earns its pixels
10. **Density is a craft choice, not a default** ... choose intentionally
11. **Match the metaphor to the medium** ... do not paginate what should scroll
12. **Type is a system, not a decision per element** ... scale, leading, weight, optical sizing follow rules

Citation map for each principle is in [`references/principles-map.md`](../../references/principles-map.md).

**Weighting:** Apply the project's `Devour Context` to weight which principles to look hardest for. A landing page rarely fails #4 (reversibility) but often fails #9 (decoration). A productivity tool rarely fails #12 (type system) but often fails #3 (intent). Don't waste the user's attention on principles that don't apply to their context.

### Step 3 ... Write findings

Each finding follows this structure:

```
[#N PRINCIPLE NAME] - <severity>
File: <path>:<line range>
Symptom:
  <one or two sentences describing what you observe in the code/UI>
Principle:
  <one sentence stating the principle and what it requires here>
Tactic:
  <the specific change that would address this>
  ```<code or pseudo-code showing the fix>```
Reference:
  <citation to references/principles/<file>.md, plus the canonical exemplar from the lineage>
```

**Severity scale:**

- **🔴 BREAKS** ... the principle is violated in a way that visibly hurts the user. Reversibility error path missing. Touch target unreachable. State lost on navigation.
- **🟡 DRIFTS** ... the principle is not catastrophically violated but the surface is drifting toward sloppiness. Hover commits too fast. Type scale has crept to four sizes. Decoration is creeping.
- **🟢 OPPORTUNITY** ... the principle is not violated but there's a higher-craft move available. Spring physics where you have eased duration. Stagger where current motion is simultaneous (or vice versa).

### Step 4 ... Look for inter-finding conflicts

Before writing the final output, scan the findings for interactions. Two findings can be in tension when:

- **Fixing one weakens the other** (e.g., extending a stagger ceiling fixes the cutoff but leaves the broader re-animation problem unresolved)
- **Both have the same root cause** and a single structural change resolves both, but it's bigger than either finding individually
- **Tactics conflict** (one says "add motion here," another says "remove motion there")
- **Findings cluster around an architectural pattern** (multiple violations from the same misuse)

When you find an interaction, surface it explicitly in a dedicated block (see Step 5 output format). Don't quietly apply structural changes at apply-time without the user knowing they were the only path. Honest review names trade-offs before they happen.

### Step 5 ... Output format

**Default (verbose):** Each finding includes symptom, principle explanation, tactic with code, and reference with full exemplar prose. Use this unless `--terse` was set.

**Terse mode (`--terse`):** Each finding is compressed to six lines. No exemplar paragraph, no extended explanation. Same severity markers, same principle and source citations, same APPLY? and INTERACTIONS blocks.

Terse finding format:
```
🔴 / 🟡 / 🟢  File: <path>:<line>
Symptom: <one sentence>
Principle: #N <name>
Source: <Designer>, "<Work>"
Tactic: <one sentence>
```

The header block, severity groupings, INTERACTIONS BETWEEN FINDINGS block, SUMMARY block, and APPLY? block all remain in terse mode, unchanged.

**Verbose output format (default):**

```
═══════════════════════════════════════════════════
DEVOUR REVIEW: <target>
Context: <project context summary in one line>
═══════════════════════════════════════════════════

🔴 BREAKS (N findings)
───────────────────────
<finding 1>

<finding 2>

🟡 DRIFTS (N findings)
───────────────────────
<finding 3>

🟢 OPPORTUNITIES (N findings)
─────────────────────────────
<finding 4>

═══════════════════════════════════════════════════
INTERACTIONS BETWEEN FINDINGS (only if any)
───────────────────────────────────────────────────
- Finding N and Finding M are partially in tension. <Brief explanation
  of how fixing one affects the other, and what structural change
  resolves both at once if applicable.>
═══════════════════════════════════════════════════

═══════════════════════════════════════════════════
SUMMARY
N breaks · N drifts · N opportunities
Principles most engaged: <#1, #4, #8>
Principles checked but clean: <#2, #3, #6, #7, #9, #10, #11, #12>
Principles not applicable to this surface: <#5>
═══════════════════════════════════════════════════

═══════════════════════════════════════════════════
APPLY?
  1. Apply all 🔴 BREAKS (N findings)
  2. Apply all 🔴 + 🟡 (N findings)
  3. Apply everything (N findings)
  4. Cherry-pick ... tell me which (e.g., "1, 3, and 5" or "the featured badge fix")
  5. Review only ... apply nothing
═══════════════════════════════════════════════════
```

After printing the review, **always print the APPLY? block as the final lines of output.** Do not skip it. The block names the user's options explicitly so they don't have to guess what's possible.

When applying:

- **Show the diff** before each file change. Brief, just the hunks.
- **Apply 🔴 BREAKS without further confirmation** if the user picked option 1, 2, or 3. The principle violation was named; the tactic was specific; the user opted in.
- **Ask once per 🟡 DRIFT or 🟢 OPPORTUNITY** that involves a real taste call (e.g., "the stagger ceiling extends to 20 ... is this the right max for your grids?"). Don't ask if the fix is mechanical.
- **After all fixes are applied, ask if the user wants to commit.** Do not auto-commit.

---

## Sub-skills (the family)

For deeper passes on specific principles, devour has focused sub-skills. Invoke when the user signals depth on a specific axis:

- [`/devour-motion`](../devour-motion/SKILL.md) ... principles #1, #2, #5 (honest motion, physics, sequence)
- [`/devour-micro`](../devour-micro/SKILL.md) ... principles #3, #6, #11 (intent, ergonomics, metaphor)
- [`/devour-state`](../devour-state/SKILL.md) ... principles #4, #7 (reversibility, state preservation)
- [`/devour-teach`](../devour-teach/SKILL.md) ... project context setup (run once per repo)

---

## Anti-fashion stance

Devour is built to age slowly. The spine draws primarily from Layers 1-3 of the lineage, which are decades or centuries old. Layer 4 tactics (Rauno's spring stiffness numbers, Emil's specific easing curves, the latest cmdk pattern) appear only as exemplars, not as principles.

When a Layer 4 tactic becomes outdated ... and it will, every five years or so ... the spine survives. The references folder gets new exemplars. The principles do not change.

This means devour will sometimes give findings that are *unfashionable*. A particular animation pattern might be trendy on Twitter and still fail principle #1 (honest motion). Devour calls that out. The user makes the final call.

---

## Voice

Devour's voice is:

- **Specific** ... cite line numbers, name the principle, point to the source
- **Calm** ... no hype, no superlatives, no emoji-heavy enthusiasm
- **Opinionated but not bossy** ... state the principle, recommend the tactic, leave the call to the user
- **Honest about uncertainty** ... if motion can't be reviewed from code alone, say so
- **Generous with attribution** ... cite the lineage by name, every time

Avoid:

- "This looks better" without principle citation
- "Modern best practice" framing (devour cares about durable practice, not modern practice)
- "Industry standard" hand-waving
- Stacking findings on top of each other without distinguishing severity

---

## Citations are not optional

If a finding cannot be traced to a principle in the spine, it does not belong in the review. The whole skill is built on the claim that good design review is principled, not preferential.

If you find yourself wanting to flag something the spine doesn't cover, two paths:

1. The thing belongs to a different skill (accessibility checklist, performance audit, copy edit). Recommend that skill.
2. The thing reveals a gap in the spine. Note it as a meta-observation at the end of the review and move on. The spine evolves slowly and on purpose; one missing case isn't enough to justify changing it.

---

## See also

- [`references/lineage.md`](../../references/lineage.md) ... full lineage, four layers
- [`references/principles-map.md`](../../references/principles-map.md) ... per-principle source citations
- [`references/exemplars.md`](../../references/exemplars.md) ... named products with specific moves
- [`references/anti-patterns.md`](../../references/anti-patterns.md) ... failure modes catalog
- [`references/reading-list.md`](../../references/reading-list.md) ... ordered reading list
- [`examples/`](../../examples/) ... annotated before/after diffs from real codebases
