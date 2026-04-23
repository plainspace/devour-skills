---
name: devour-teach
description: "Project context setup for devour reviews. Run once per repo before the first devour pass. Reads the project, asks 5-7 questions, synthesizes answers into a Devour Context block, and writes it to .devour-context.md or the project CLAUDE.md. Without context, devour produces generic findings; with context, it applies the right principles at the right weight for this specific product."
argument-hint: ""
user-invocable: true
license: Apache 2.0. See NOTICE.md for full attribution to the design lineage this skill stands on.
---

> Different products live in different parts of the spine. A marketing page and a productivity tool share the same twelve principles, but weight them very differently.

Devour's principles are stable. Their relative importance is not. A landing page rarely fails #4 (reversibility) but often fails #9 (decoration). A productivity tool rarely fails #12 (type system) but often fails #3 (intent). A creative tool values #2 (physics) and #5 (sequence) highly; a dashboard values #10 (density) and #9 (information) highly.

Without context, devour produces findings that are technically correct but practically wrong for the product. You get a reversibility audit on a marketing page and a decoration audit on an offline-first productivity tool.

Run `/devour-teach` once per repo before any other devour skill. The output is a `Devour Context` block that all subsequent devour invocations will read before reviewing.

---

## When to use

- First time devour is run on any project.
- A project has significantly changed product direction, target audience, or surface type since the last `/devour-teach` run.
- The existing `Devour Context` feels misaligned with what the project actually is.
- Another devour skill (`devour`, `devour-motion`, `devour-micro`, `devour-state`) stopped and said "Devour needs project context first."

---

## Process

### Step 1 ... Read existing project files

Before asking any questions, read the project to understand what you are already looking at. Do not ask questions you can infer from the code.

Read in this order (stop when you have enough context to ask meaningful questions, not before):

1. `package.json` ... what is this project? what dependencies are installed? (React version, Next.js, Framer Motion, animation libraries, UI library, form library, routing)
2. `README.md` ... how does the project describe itself?
3. `.claude/CLAUDE.md` or `CLAUDE.md` ... any existing project instructions; don't overwrite, but note what's there
4. Any obvious design system or token files: `tailwind.config.*`, `tokens.css`, `theme.ts`, `design-tokens.*`
5. A sample of the actual source files: two or three representative page or component files

From this reading, form a preliminary hypothesis about the product type. You will refine it with questions.

---

### Step 2 ... Ask the seven questions

After reading, ask the user a focused set of seven questions. Do not ask questions you can already answer from the code.

**Default mode: progressive.** Ask one question at a time, wait for the answer, then ask the next. This lets the user reconsider an earlier answer if a later question reveals it was wrong, and avoids the "wall of questions" intimidation that hurts adoption.

**Quick mode (advanced users):** If the user invoked `/devour-teach --quick` or explicitly said "ask them all at once," present all seven in a single numbered block.

**The seven questions:**

**1. Product type.** What category best describes this product?
- (A) Marketing / landing pages ... primary goal is acquisition or conversion
- (B) Productivity tool ... users do repeated work; speed and density matter
- (C) Creative tool ... users make things; craft and expressiveness matter
- (D) Dashboard / analytics ... users read data; density and hierarchy matter
- (E) Content platform ... users read or browse; typography and navigation matter
- (F) E-commerce ... users evaluate and purchase; trust and reversibility matter
- (G) Other (describe)

**2. Target audience.** Who uses this product?
- (A) Consumers / general public
- (B) Developers
- (C) Designers
- (D) Operations / SRE / enterprise power users
- (E) Executives / decision-makers
- (F) Mixed (describe the primary segment)

**3. Tone target.** What is the intended emotional register?
- (A) Calm and considered (Arc, Linear, Vercel)
- (B) Playful and expressive (NotBoring, Daylight)
- (C) Bold and direct (Stripe, Figma)
- (D) Minimal and receding (Things 3, Bear)
- (E) Dense and information-first (Bloomberg, Linear in power-user mode)
- (F) Ambient and spatial (MercuryOS direction)
- (G) Other (describe)

**4. Primary surface type.** What is the dominant UI pattern?
- (A) Long-scroll pages (landing, editorial, docs)
- (B) App-like multi-screen with navigation (SaaS dashboard, mobile app)
- (C) Dense data tables or lists (issue tracker, data grid, log viewer)
- (D) Creative canvas or editor (design tool, rich text, code editor)
- (E) Conversational (chat, support, onboarding wizard)
- (F) Mixed (describe the dominant one)

**5. Motion appetite.** What level of animation is appropriate for this product?
- (A) None / static (no animation except functional loading states)
- (B) Restrained / functional (only where motion communicates state change)
- (C) Considered (deliberate motion that adds craft without calling attention to itself)
- (D) Expressive (motion is part of the product personality)

**6. Density target.** How dense should the primary working surfaces be?
- (A) Spacious (Notion, Craft ... generous whitespace, relaxed reading)
- (B) Balanced (Vercel dashboard, GitHub ... functional density without oppression)
- (C) Dense (Linear, VS Code ... maximum information per viewport)
- (D) Very dense (Bloomberg Terminal, Datadog ... power-user specific)

**7. Reference exemplars.** Name 3-5 products that represent the quality bar or aesthetic direction you want for this product. Be specific. "Apple" is too broad; "Linear" or "Arc" or "Vercel dashboard" is useful.

---

### Step 3 ... Synthesize into a Devour Context block

Take the answers, combine them with what you observed in Step 1, and write a `Devour Context` block.

**Principle weighting guide by product type:**

Use this table as the starting point, then adjust based on the specific answers.

**Marketing / landing pages:**
- High: #1 (honest motion), #9 (reduce decoration), #12 (type system)
- Medium: #8 (affordances), #11 (metaphor), #5 (sequence)
- Low: #4 (reversibility), #7 (state preservation)
- N/A: rarely #6 on desktop-only sites

**Productivity tools:**
- High: #3 (intent commit), #4 (reversibility), #6 (ergonomics), #7 (state preservation)
- Medium: #9 (decoration), #10 (density), #8 (affordances)
- Low: #1 (motion ... functional only), #12 (type rarely breaks)
- Adjust if mobile: raise #6 significantly

**Creative tools:**
- High: #2 (physics), #5 (sequence), #11 (metaphor), #4 (reversibility)
- Medium: #8 (affordances), #6 (ergonomics), #1 (honest motion)
- Low: #12 (type system, unless typography is core to the product), #10 (density)

**Dashboards / analytics:**
- High: #9 (decoration), #10 (density), #12 (type system), #8 (affordances)
- Medium: #7 (state preservation), #6 (ergonomics), #3 (intent)
- Low: #2 (physics), #5 (sequence), #1 (motion)

**Content platforms:**
- High: #12 (type system), #11 (metaphor), #9 (decoration)
- Medium: #8 (affordances), #1 (motion), #10 (density)
- Low: #4 (reversibility), #7 (state), #2 (physics)

**E-commerce:**
- High: #4 (reversibility), #7 (state), #6 (ergonomics), #1 (honest motion)
- Medium: #3 (intent), #8 (affordances), #11 (metaphor)
- Low: #2 (physics), #5 (sequence), #10 (density)

---

### Step 4 ... Write the context block

Write the complete `Devour Context` block in this format:

```markdown
## Devour Context

**Product type:** <type from Q1>
**Target audience:** <audience from Q2>
**Tone:** <tone from Q3>
**Primary surface:** <surface from Q4>
**Motion appetite:** <appetite from Q5>
**Density target:** <density from Q6>
**Reference exemplars:** <list from Q7>

**Principle weighting** (which principles apply hardest to this product):
- High: #N, #N, #N (the principles that should produce 🔴 BREAKS, not just 🟢 OPPORTUNITIES)
- Medium: #N, #N, #N (the principles that should produce 🟡 DRIFTS if violated)
- Low: #N, #N (violations worth noting but not worth stopping for)
- N/A: #N (genuinely does not apply to this product's surface type)

**Specific things to watch for in this codebase:**
- <observation from reading the code in Step 1>
- <observation from reading the code in Step 1>
- <2-4 product-type-specific patterns based on the answers>
```

The "Specific things to watch for" section should include at least one observation derived from actually reading the code. Examples:
- "Framer Motion is installed and used; spring config will be reviewable in detail"
- "shadcn/ui is the component library; watch for Tooltip `delayDuration={0}` patterns"
- "This is a mobile app; principle #6 findings should assume 44pt targets"
- "react-hook-form is installed but localStorage persistence is not; draft preservation is worth reviewing"
- "No toast library is installed; reversibility lifecycle relies on custom UI, which will need review"

---

### Step 5 ... Write to a file

**Default: write to `.devour-context.md` in the project root.**

This is the open-source-friendly default. `.devour-context.md` is a portable, tool-agnostic file that any AI assistant (Claude Code, Cursor, Cline, Continue, Codex, Aider, Goose, ...) can be pointed at. It also makes "this repo has run devour" a discoverable signal in the file tree.

Create `.devour-context.md` with this structure:

```markdown
# Devour Context

This file is read by the devour skill family before running reviews on this project.
Generated by `/devour-teach` on <date>. Update if the product direction changes significantly.

If your AI assistant doesn't auto-load this file, you can manually include it:
- Claude Code: append the contents below to your CLAUDE.md, or reference this file from there
- Cursor: append to .cursorrules or reference from .cursor/rules/
- Other tools: include via your tool's project-instructions mechanism

---

## Devour Context

...
```

Tell the user: "Created `.devour-context.md` in the project root. Add it to version control. If you want it auto-loaded by Claude Code, append its contents to your CLAUDE.md or reference it from there."

**Optional: auto-link from CLAUDE.md if it exists.**

If `.claude/CLAUDE.md` or `CLAUDE.md` exists in the project root, ask the user: "I've created `.devour-context.md`. Want me to add a one-line reference in your CLAUDE.md so Claude Code picks it up automatically?"

If yes, append a single line to the appropriate CLAUDE.md:

```markdown
**Devour context:** see [.devour-context.md](.devour-context.md) for the design review context for this project.
```

Do not overwrite or duplicate context into CLAUDE.md. The single source of truth is `.devour-context.md`. The CLAUDE.md line is just a pointer.

---

### Step 6 ... Confirm and hand off

After writing the context, summarize in 3-5 sentences:
- What kind of product you've set up context for
- Which principles you've weighted high and why
- What specific patterns to watch for in this codebase

Then offer: "Run `/devour` to start a full-spine review, or `/devour-motion`, `/devour-micro`, or `/devour-state` for a focused pass."

---

## Example output

```
I've read the project files. This looks like a B2B productivity SaaS with a React/Next.js
stack, shadcn/ui components, TanStack Query for data fetching, and Framer Motion
for animations.

A few questions before I set up context:

1. Product type: ...
[questions 1-7 as numbered list]

---

(After answers)

Writing Devour Context to .devour-context.md ...

Done. Context summary:
- Productivity tool, targeting operations/SRE users who prioritize speed and density
- Principles #3, #4, #6, and #7 weighted high: this audience expects intent-aware
  hover behavior, complete error paths on every async operation, and keyboard-first ergonomics
- Noted: shadcn Tooltip is imported in 12 components; delayDuration will be worth checking
- Noted: several useMutation calls observed; error variants will be a specific focus

Run /devour to start, or /devour-state for an immediate pass on the mutation error paths.
```

---

## What the context enables

Without a Devour Context, the review applies all twelve principles at equal weight. The result is technically correct and practically noisy: six findings about motion on a tool where the motion appetite is "none/static," and no findings about state preservation on a tool where every async operation is a potential data-loss event.

With a Devour Context, the review front-loads the principles that matter, calibrates severity, and skips (or lowers) principles that are not relevant to this product's surface. The findings are sharper, shorter, and more actionable.

---

## See also

- [`../devour/SKILL.md`](../devour/SKILL.md) ... start here after running teach
- [`../../references/lineage.md`](../../references/lineage.md) ... the four-layer model
- [`../../references/principles-map.md`](../../references/principles-map.md) ... per-principle source citations
- [`../../references/anti-patterns.md`](../../references/anti-patterns.md) ... what each principle is protecting against
