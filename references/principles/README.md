# Principles

This folder is intended to hold one Markdown file per principle for deep treatment of each of devour's twelve principles.

**The twelve principle files are forthcoming.** They are not included in the v0.1 release.

For now:

- **Principle citations and source mapping:** [`../principles-map.md`](../principles-map.md) ... maps each principle to its primary and secondary sources across the four-layer lineage.
- **Failure modes per principle:** [`../anti-patterns.md`](../anti-patterns.md) ... 3-5 named anti-patterns per principle, with descriptions and fix directions.
- **Positive exemplars per principle:** [`../exemplars.md`](../exemplars.md) ... named products with specific moves to study.
- **Full lineage:** [`../lineage.md`](../lineage.md) ... the four-layer model and how each layer feeds the spine.

---

## Planned structure

When the per-principle files are written, each will follow this structure:

```
# Principle #N: [Name]

> [The principle statement, one sentence]

## Statement

[The full principle in 2-3 sentences, expanded beyond the one-line version]

## Sources

[Primary and secondary citations from the four-layer lineage]

## What this catches

[The classes of problems this principle is designed to identify]

## Tactics

[The positive patterns that satisfy this principle, with code examples]

## Anti-patterns

[The failure modes specific to this principle, cross-referenced to anti-patterns.md]

## Exemplars

[Named products that demonstrate this principle in production, cross-referenced to exemplars.md]

## Relationship to other principles

[How this principle interacts with or depends on others in the spine]
```

---

## Contribution

The per-principle files are a high-priority contribution target. If you want to contribute to devour, writing a thorough treatment of a single principle... with citations, code examples, and real exemplars... is the most valuable thing you can add.

Guidelines for contributing a principle file:

1. Follow the structure above.
2. Cite the lineage explicitly by layer and name. The principle has a four-layer ancestry; trace it.
3. Code examples should be idiomatic React/TypeScript. No pseudocode for the tactical sections.
4. Exemplars must be real, named, publicly accessible products. No hypotheticals.
5. Anti-patterns must name the failure mode, not just describe the bad code. The name is what makes it memorable and citable.

Open a pull request with the principle file, and reference the principle number in the PR title.

---

## See also

- [`../principles-map.md`](../principles-map.md)
- [`../anti-patterns.md`](../anti-patterns.md)
- [`../exemplars.md`](../exemplars.md)
- [`../lineage.md`](../lineage.md)
- [`../../skills/devour/SKILL.md`](../../skills/devour/SKILL.md)
