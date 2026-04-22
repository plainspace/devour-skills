# Examples

Real before/after pairs from real codebases that ran devour. Each example is a self-contained folder with the original code, the devour finding that drove the change, the resulting code, and a writeup explaining why the change matters.

**No synthetic examples.** Everything here came from a working codebase. If devour didn't find it, it doesn't belong here.

---

## Why examples matter

Principles in the abstract are easy to nod along with. The hard work is recognizing them in real code, applying them under real constraints, and noticing where the tactic doesn't fit perfectly and you have to make a judgment call.

These examples exist to:

1. **Show what devour findings look like in practice.** Verbatim output, not cleaned-up.
2. **Show that the principle catches a thing the developer wouldn't have caught otherwise.** If the change was obvious, it's a bad example.
3. **Show where the tactic needed to be adapted.** Real engineering involves taste, not formula.

---

## Format

Each example is a numbered folder: `NNNN-<codebase>-<short-title>/`

```
0001-fifthset-event-card-hover/
├── README.md             ← the writeup (template below)
├── before.tsx            ← file snapshot before the change
├── after.tsx             ← file snapshot after the change
├── devour-finding.md     ← verbatim devour output that drove the change
└── screenshot-before.png ← optional, only for visual or motion findings
```

Numbers are sequential, never reused, never reordered. They are sort keys, not priority indicators.

### Writeup template

Each example's `README.md` follows this structure. Copy this template when adding a new example.

```markdown
# Example NNNN: <short title>

**Codebase:** <name and one-line tech description>
**Date:** <YYYY-MM-DD>
**Devour skill used:** <devour | devour:motion | devour:micro | devour:state>
**Principle(s) engaged:** <#N (name), #N (name)>
**Severity:** <🔴 BREAKS | 🟡 DRIFTS | 🟢 OPPORTUNITY>

## The before

<Two or three sentences describing what the code did, with file:line references where relevant.>

## The devour finding

<Verbatim copy of the devour output for this finding. Include the principle citation, the tactic, and the reference. Do not edit it for clarity ... if it was unclear, that is information.>

## The after

<Two or three sentences describing what changed and why. Reference the new file structure if non-obvious.>

## What this example demonstrates

<One paragraph. What about this finding is generalizable. Why this is a useful exemplar for someone learning the principle. What pattern this case represents.>

## Notes

<Optional. Judgment calls made. Places where the recommendation did not fit perfectly. Where you departed from the tactic and why. Honest engineering notes.>
```

---

## What makes a good example

Good examples have at least three of:

- **A finding the developer wouldn't have caught otherwise.** Devour earned its keep.
- **A clean before/after diff.** The change is small enough to reason about in one sitting.
- **A principle that wasn't obvious from the symptom.** "Pulse animation on a static badge" is a good case for #1; "this button is too small" is too obvious to teach.
- **A judgment call worth explaining.** Where you adapted the tactic to the specific context.
- **A reference to the lineage.** The exemplar (Sonner, Vaul, Linear, etc.) made the fix obvious.

Bad examples:

- Renaming a variable
- A pure refactor with no devour finding
- Following the recommendation literally with no thinking applied
- "Made it look better" without principle citation
- Anything that could have been caught by `polish`, ESLint, or a linter

---

## Numbering and authorship

The first authored example will be `0001`. Examples are added by the author (initially Jared Volpe) as the skill is used in production work. Outside contributions are welcome but require following the template above and citing the lineage explicitly.

If an example turns out to be wrong (the principle didn't apply, the fix was bad, the lineage citation was misleading), don't delete it. Add a `RETROSPECTIVE.md` to the folder explaining what was wrong. Mistakes are useful.

---

## Visual and motion examples

For findings that are primarily visual or motion-related, capture artifacts:

- **Visual** ... PNG screenshot before, PNG after. Keep small (under 500KB each).
- **Motion** ... 3-5 second `.gif` or `.mp4` capture. Compress aggressively. Note in the writeup that motion examples require running the code to fully appreciate.

These are optional but strongly preferred for any finding under principles #1, #2, or #5.

---

## Status

**v0.1:** intentionally empty. Examples are added as devour gets used in production work, not invented for the launch. The first example is expected to come from a fifthset (Next.js events directory) test pass.

---

## See also

- [`../skills/devour/SKILL.md`](../skills/devour/SKILL.md) ... the main skill
- [`../references/anti-patterns.md`](../references/anti-patterns.md) ... the failure modes these examples will illustrate
- [`../references/exemplars.md`](../references/exemplars.md) ... the positive exemplars from the lineage
