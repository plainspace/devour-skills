# Examples

This folder will hold annotated before/after diffs from real codebases that ran devour.

**For the v0.1 release, this folder is intentionally empty.** Examples come from real PRs, not made-up scenarios.

---

## Why deferred

Fabricated examples are worse than no examples. A made-up before/after diff can demonstrate a principle in isolation, but it cannot demonstrate the judgment calls that arise when principles conflict, when context changes the severity, or when the right fix is not the obvious one. That judgment only shows up in real code, under real constraints.

The examples folder will be populated as the skill is used in production work. The first entries will come from the author's own codebases, with identifying details removed.

---

## Planned format

Each example will be a single `.md` file. The format:

```markdown
# [Component or feature name]

**Product type:** [from Devour Context]
**Principles engaged:** #N, #N
**Severity:** 🔴 BREAKS | 🟡 DRIFTS | 🟢 OPPORTUNITY

## Context

[One paragraph: what the component does, what the product is, why this finding matters]

## Finding

[The finding block from the devour output, verbatim]

## Before

```tsx
// The original code
```

## After

```tsx
// The fixed code
```

## Notes

[Any judgment calls made in the fix, trade-offs acknowledged, things that were considered and
rejected]
```

---

## Contributing examples

If you run devour on a real codebase and have a finding worth sharing, a pull request to this folder is welcome. Requirements:

1. Real code, real product (with permission and identifying details removed if needed).
2. The before/after must be a real change that was made, not a hypothetical.
3. Follow the format above.
4. The finding must cite a principle from the spine.

One good real example is worth more than ten made-up ones.

---

## See also

- [`../skills/devour/SKILL.md`](../skills/devour/SKILL.md) ... the main skill
- [`../references/anti-patterns.md`](../references/anti-patterns.md) ... the failure modes these examples will illustrate
- [`../references/exemplars.md`](../references/exemplars.md) ... the positive exemplars from the lineage
