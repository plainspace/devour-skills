---
name: devour-brief
description: "Per-target design brief for devour reviews. Articulates a specific target (audience, register, comparable products, motion and density delta from project defaults, success criterion) before any design or review begins. Output is a Target line plus a 3-sentence rationale, saved to .devour/briefs/ and referenceable by /devour --brief <path>. Use standalone before designing, or as the front of a review."
argument-hint: "[--repo <absolute-path>] [target file, component, page, or prose description] [--register <brand|product>] [--force]"
user-invocable: true
license: Apache 2.0. See NOTICE.md for full attribution to the design lineage this skill stands on.
---

> The brief is the smallest artifact that can fail well. Write it down before you make anything you'd rather not throw away.

Devour-brief is a per-target design brief. It runs in 4-6 questions and writes one file: a Target line (machine-parseable, quotable) and a 3-sentence rationale (the why). It is the layer between `DEVOUR.md` (project-wide context, written once) and a specific design or review (target-specific, written each time the target changes).

A brief makes two things possible:

1. **Designing toward a target you can name.** Without a brief, a designer working in a new register defaults to whatever feels right. What feels right is usually what's familiar, which is usually what they last shipped, which is usually wrong for this target. A brief locks the target before the first pixel.
2. **Reviewing against a target, not just principles.** Devour's spine catches principle violations. A brief catches *target* violations: principle 9 (reduce decoration) is a generic rule; "principle 9 for an editorial-magazine register where Tufte's data-ink ratio is load-bearing" is a target. Findings that quote the brief are sharper than findings that quote only the principle.

---

## When to use

- **Before designing something new** in a register you don't usually work in (a kids' app when you mostly do B2B SaaS; a finance dashboard when you mostly do marketing pages; an editorial layout when you mostly do product UI).
- **Before iterating on something** that doesn't feel right but you can't articulate why. The brief surfaces the target you've been designing past without naming.
- **Before a devour review** when the surface is unusual for the project (the marketing page in a product-register repo; the dashboard in a brand-register repo; a one-off campaign micro-site).
- **As a handoff artifact** to another designer, an AI design tool (impeccable, Open Design, claude-design), or a contractor. The brief is what you'd write on a Post-it before sending the link.

Do NOT use for routine work where `DEVOUR.md` already calibrates the project well and the target sits inside the project's normal register. The brief adds friction; spend it where it earns its keep.

---

## Relationship to other devour skills

- **`devour-teach`** runs once per repo. It writes `DEVOUR.md` (project context).
- **`devour-brief`** runs per target. It writes `.devour/briefs/<filename>.md` (target context). The brief inherits from `DEVOUR.md` and may override its fields for this specific target.
- **`devour`, `devour-motion`, `devour-micro`, `devour-state`** consume both. They read `DEVOUR.md` for project defaults, read `--brief <path>` if passed for target-specific overrides, and quote the Target line in every finding's footer.

The brief never modifies `DEVOUR.md`. They are independent artifacts at different scopes.

---

## Process

### Step 0a ... Resolve target repo (`--repo`) and parse known flags

Read `$ARGUMENTS`. If `--repo <path>` is present:

1. Extract `<path>` as the value.
2. Strip `--repo <path>` from `$ARGUMENTS` before passing to later steps.
3. Validate: the path must exist and be a directory. If it does not: stop and report `--repo path does not exist: <path>. Aborting.`
4. Set `$REPO` = the absolute `<path>`.

If `--repo` is not present: set `$REPO` = current working directory.

Parse other known flags before later steps use them:

- `--register <value>` ... extract value. Must be exactly `brand` or `product` (case-insensitive). If invalid, fail with `--register must be 'brand' or 'product', got '<value>'. Aborting.` Store as `$CLI_REGISTER` for use in Step 2.
- `--force` ... set `$FORCE = true`. Allow overwriting an existing brief file with the same target slug from the same UTC day.

Strip both from `$ARGUMENTS`.

For the rest of this skill:

- All file reads use `$REPO` as the root.
- `DEVOUR.md` reads from `$REPO/DEVOUR.md`.
- `.devour/briefs/` writes to `$REPO/.devour/briefs/`.
- If the target argument is a relative path, resolve it against `$REPO`.

### Step 0b ... Read `DEVOUR.md` for inheritance

Check `$REPO/DEVOUR.md`. Three cases:

1. **`DEVOUR.md` exists:** parse it. Extract `## Register`, `## Audience`, `## Brand voice`, `## Anti-references`, `## Principle weighting (default)`, `## Motion appetite`, `## Density target`, `## Reference exemplars`, `## Specific things to watch for`. These become **defaults** for the brief; the user can override any of them in Step 2.

2. **`DEVOUR.md` is missing AND `$REPO/.devour-context.md` exists** (legacy v0.3 format): tell the user once: "Found legacy `.devour-context.md` at `$REPO/.devour-context.md`. The brief format expects `DEVOUR.md` (v0.4+). The legacy file is not read for inheritance. To migrate, run `/devour-teach --force` to write a fresh `DEVOUR.md`. Proceeding with this brief as self-contained." Continue without inheritance.

3. **Both missing:** the brief proceeds without inheritance. Tell the user once: "No `DEVOUR.md` found at `$REPO/DEVOUR.md`. The brief will be self-contained. Run `/devour-teach` if you'd like project-wide context to inherit from on future briefs." Continue.

A brief without an inherited `DEVOUR.md` is valid (useful for one-off work, prototypes, scratch repos). A brief that inherits from `DEVOUR.md` is sharper because the inherited fields don't need to be re-asked.

**Built-in defaults when no inheritance is available** (cases 2 and 3 above):

- `register` ... ask explicitly in Step 2 question 0 (added below). No silent default; the register frames every other answer and must be intentional.
- `motion appetite` ... ask absolute value in Step 2 question 4 (instead of "delta from inherited").
- `density target` ... ask absolute value in Step 2 question 4 (instead of "delta from inherited").
- `audience`, `brand voice`, `anti-references`, `reference exemplars` ... no defaults; ask the user as if `DEVOUR.md` did not exist for those fields.
- `principle weighting` ... not asked in the brief (briefs don't override weighting; that's `DEVOUR.md`'s job).

### Step 0c ... Establish target

If `$ARGUMENTS` contains a path-like string (matches the path heuristic from `devour/SKILL.md` Step 0e: starts with `/`, `.`, `src/`, `components/`, etc., or ends in `.tsx`, `.jsx`, etc., or contains `/` with no whitespace):

- Set `$TARGET_KIND = file`.
- Set `$TARGET = <the resolved absolute path>`.
- Read the file briefly (first 200 lines or whole file if shorter) so you can refer to what it actually does in Step 2 questions.

If `$ARGUMENTS` is prose:

- Set `$TARGET_KIND = prose`.
- Set `$TARGET = <the prose, verbatim>`.
- Use the prose as the seed for Step 2 question 1.

If `$ARGUMENTS` is empty:

- Ask once: "What's the target for this brief? A file path (e.g., `app/routes/dashboard.tsx`), a surface description (e.g., `the new pricing page`), or a prose statement of intent (e.g., `a kids' app onboarding flow that doesn't feel infantilizing`)."
- Wait for the answer. Re-classify per the path-vs-prose check above.

### Step 1 ... Apply per-surface match if relevant

If `DEVOUR.md` has a `## Per-surface overrides` section AND `$TARGET_KIND = file` AND `$TARGET` matches one of the declared surface prefixes (first match wins), use that surface's defaults instead of the top-level `DEVOUR.md` defaults for inheritance.

If no per-surface match or `$TARGET_KIND = prose`, use top-level defaults.

If `--register <value>` was passed, it overrides whatever the inheritance produced for register. Other inherited fields (motion appetite, density target, weighting) still apply unless the user changes them in Step 2.

### Step 2 ... Ask brief questions

Ask these questions one at a time. Show inherited defaults from `DEVOUR.md` (if present) and let the user accept or override. If a question's answer is fully determined by inheritance and the inheritance is high-confidence (e.g., register is unambiguous from per-surface match), skip the question and tell the user once that you're using the inherited value.

**Question 0 ... Register (asked only when there is no inheritance and no `--register` CLI flag).**

If `DEVOUR.md` is missing AND `--register` was not passed, ask: "What register is this target? `brand` (marketing, landing, portfolio, editorial) or `product` (app UI, admin, dashboard, productivity tool)?"

Skip this question if register is already established via inheritance from `DEVOUR.md` or via the `--register` CLI flag. Register is the frame for every other answer; do not assume it.

**Question 1 ... Target description (always asked).**

What is this target, and what's it for?

If `$TARGET_KIND = file`, pre-fill from the file's apparent purpose: "I see `<filename>` ... it looks like a `<dialog | route | component | page>` for `<inferred purpose>`. Is that right? Add anything I missed."

If `$TARGET_KIND = prose`, echo the prose back: "You said: '<prose>'. Anything to add or sharpen?"

Goal: get one or two sentences that name the target and its job.

**Question 2 ... Audience narrowing (always asked, even if `DEVOUR.md` has a project-wide audience).**

Who specifically is this target for? Project-wide audience is one thing; per-target audience is sharper.

Inherit from `DEVOUR.md` `## Audience` as the starting point. Ask: "Project audience is `<inherited>`. Is the audience for THIS target the same, narrower, or different? If narrower, who specifically?"

Examples of useful narrowing:

- Project audience = "B2B SaaS users." Brief audience = "engineering managers in their first manager role, evaluating tools mid-flight in a Slack thread."
- Project audience = "consumers." Brief audience = "moms aged 30-45 buying a kids' app at 9pm after the kids are in bed, on iPhone, in dark mode."
- Project audience = "developers." Brief audience = "developers comparing this against three other tools on a Tuesday afternoon, opened the docs in five tabs."

The narrowing is the point. Generic audiences produce generic design.

**Question 3 ... Comparable products (always asked, two parts).**

(a) Name 2-4 products this target should feel adjacent to. Not copy. Adjacent. The reader should be able to imagine the family resemblance.

Inherit from `DEVOUR.md` `## Reference exemplars` as candidates if any. Ask: "Project exemplars include `<inherited list>`. For THIS target, which 2-4 products should it feel adjacent to? Be specific: not 'Apple' but 'Apple Notes app' or 'iOS Settings'."

(b) Name 1-3 products this target should NOT feel like, even if the comparison is tempting. The anti-references catch the lazy default.

Inherit from `DEVOUR.md` `## Anti-references` as candidates. Ask: "Project anti-references include `<inherited list>`. For THIS target, what should it explicitly NOT feel like? Things people might assume but you're rejecting."

If the user can't name an anti-reference, prompt: "What's the obvious comparison everyone would reach for that you want to avoid? That's the anti-reference."

**Question 4 ... Motion and density (always asked).**

Two cases based on whether `DEVOUR.md` is providing inheritance.

**Case A ... `DEVOUR.md` provides inherited defaults.** Ask delta:

Project defaults from `DEVOUR.md`: motion `<inherited>`, density `<inherited>`.

Ask: "For THIS target, are motion and density the same as the project defaults, or different?"

If different, ask the user to state the delta in one sentence each:

- "Motion: I want it to feel `<more restrained | more expressive | the same>` than the project default because `<reason>`."
- "Density: I want it to feel `<more spacious | denser | the same>` than the project default because `<reason>`."

If the same, accept inheritance and continue.

**Case B ... no inheritance (`DEVOUR.md` is missing).** Ask absolute values:

- Motion: "How much motion does this target want? `none` (no animation except functional loading), `restrained` (motion only where it communicates state change), `considered` (deliberate motion that adds craft without calling attention to itself), or `expressive` (motion is part of the personality)?"
- Density: "How dense should this target feel? `spacious` (Notion / Craft, generous whitespace), `balanced` (Vercel dashboard / GitHub, functional density), `dense` (Linear / VS Code, max info per viewport), or `very-dense` (Bloomberg Terminal, power users)?"

In both cases, the resolved motion and density values land in the Target line in Step 3.

**Question 5 ... Success criterion (always asked).**

The hardest question. What does success look like? Not "good design" ... a specific testable observation.

Examples of useful success criteria:

- "A new user lands on this page and books a demo within 90 seconds without scrolling past the hero."
- "An engineering manager sees the dashboard and can name three things to do next within 10 seconds."
- "A reader finishes the editorial spread and remembers the lead photo, the pull quote, and the byline."
- "A first-time user opens the app and the first screen is the only place they pause; everything after feels obvious."

If the user gives a vague answer ("it feels great"), push back once: "Can you turn that into something an outside observer could verify? What would you watch them do?"

If they still can't, accept the vague answer and note it as a soft criterion. Don't force a hard one when there isn't one.

**Question 6 ... Specific things to watch for (asked only if relevant).**

If `DEVOUR.md` has `## Specific things to watch for`, ask: "Project-wide watch list includes `<inherited>`. Anything specific to THIS target? Things that would be 🔴 BREAK if violated?"

If `DEVOUR.md` is missing or has no watch list, skip this question. The brief is meant to be light.

### Step 3 ... Compose the brief

Compose two outputs:

**(a) The Target line.** Strict template, single line, machine-parseable. Format:

```
Target: <register>-<surface-type> for <audience-narrow>; adjacent-to <ref1, ref2[, ref3, ref4]>; not <anti-ref1[, anti-ref2, anti-ref3]>; motion-<level>; density-<level>; success-when <criterion>
```

Field rules:

- `<register>` = `brand` or `product` (one word).
- `<surface-type>` = inferred or asked: `landing`, `dashboard`, `dialog`, `onboarding`, `editorial`, `pricing`, `docs`, `settings`, `email`, `carousel`, `prototype`, `spread`, `flow`, `component`, `route`, etc. One word, lowercase.
- `<audience-narrow>` = the answer to Question 2, compressed to a noun phrase under 12 words.
- `<ref1, ref2, ...>` = the answer to Question 3a, comma-separated, 2-4 items.
- `<anti-ref1, ...>` = the answer to Question 3b, comma-separated, 1-3 items.
- `<level>` for motion = one of `none`, `restrained`, `considered`, `expressive`. Map from the inherited or delta-adjusted motion appetite.
- `<level>` for density = one of `spacious`, `balanced`, `dense`, `very-dense`. Map from the inherited or delta-adjusted density target.
- `<criterion>` = the answer to Question 5, compressed to a single observation under 25 words. If the criterion is soft, prefix with `(soft)`.

Example Target line:

```
Target: brand-editorial for moms-aged-30-to-45-buying-a-kids-app-at-9pm; adjacent-to The New York Times, Frame, Brockmann's Grid Systems; not Disney+, Cocomelon, ABCmouse; motion-restrained; density-balanced; success-when reader remembers lead photo, pull quote, and byline 24 hours later
```

It's long. It's not pretty. It's quotable, parseable, and one line. That's the point.

**(b) The Rationale.** Three sentences, prose. Each sentence does one job:

- **Sentence 1 ... Target.** What this target is, who it's for, and what it's competing against (in attention, in convention). One sentence.
- **Sentence 2 ... Tension.** The non-obvious choice the brief makes ... the thing the designer might do by default that this brief is rejecting. One sentence.
- **Sentence 3 ... Test.** How the designer (or reviewer) will know if it's working. One sentence. Echoes the success criterion in plain language.

Example rationale:

```
This is an editorial spread for a kids' app onboarding flow, aimed at moms making a paid-app decision late at night, competing not with other kids' apps but with the mom's working memory of the next thing she has to do. The default move is to make this look like a kids' app (bright primaries, rounded sans, mascot illustration); the brief rejects all of that and asks for an editorial register because the buyer is the mom, not the kid. Success is the mom showing the page to her partner the next morning and being able to articulate what the app is in one sentence.
```

The Target line and the Rationale are both written. Target line is the enforceable artifact (quoted in every review finding's footer when `--brief` is passed). Rationale is the useful artifact (gives the next person ... or AI tool ... enough to act on).

### Step 4 ... Write the brief file

Write the brief to `$REPO/.devour/briefs/<YYYY-MM-DDTHHMMSS>-<target-slug>.md` (UTC timestamp, ISO-8601-like, no colons in filename).

`<target-slug>` derivation:

- For file targets: slugify the relative path from `$REPO`. Replace `/` with `-`, lowercase, strip unsafe chars, cap at 40 chars.
- For prose targets: take the first 5-7 meaningful words from the prose, lowercase, hyphenate. Cap at 40 chars.

Directory creation: if `$REPO/.devour/` or `$REPO/.devour/briefs/` does not exist, create the nested structure.

**File template:**

```markdown
---
created: <ISO-8601 UTC timestamp>
skill: devour-brief
target: <human-readable target description>
target-kind: <file|prose>
target-path: <absolute path if target-kind=file, else null>
repo: <$REPO absolute path>
context-file: <path to DEVOUR.md if read, else null>
register: <brand|product>
inherited-from: <DEVOUR.md path if inheritance applied, else null>
per-surface-match: <surface prefix if matched, else null>
---

# Devour brief: <target description>

## Target line

<the Target line from Step 3a, single line>

## Rationale

<the 3-sentence rationale from Step 3b>

## Inputs

### Target description
<answer to Question 1>

### Audience
<answer to Question 2>

### Comparable products (adjacent to)
<answer to Question 3a, as bullet list>

### Comparable products (NOT)
<answer to Question 3b, as bullet list>

### Motion delta
<answer to Question 4 motion part, or "same as project default">

### Density delta
<answer to Question 4 density part, or "same as project default">

### Success criterion
<answer to Question 5>

### Specific things to watch for (target-specific)
<answer to Question 6 if asked, else "none">

## How to use this brief

To run a devour review against this target with the brief loaded:

```
/devour --repo <$REPO> --brief .devour/briefs/<this-filename>.md <target>
```

The Target line will be quoted in the run header. Each finding will get a "Why this matters here" footer that ties the principle back to this brief's target.

For other tools (impeccable, AI design tools, contractors): paste the Target line and Rationale into the brief field. They are sufficient context for someone unfamiliar with the project.
```

**Force handling.** If a brief file already exists at the same path (same target slug, same UTC day):

- Without `--force`: tell the user the file exists and print its path. Stop.
- With `--force`: overwrite. Confirm once before doing so.

### Step 5 ... Print the brief and next-step prompt

Print to the conversation:

```
═══════════════════════════════════════════════════
DEVOUR BRIEF
File: <path>
═══════════════════════════════════════════════════

TARGET LINE:
<the Target line>

RATIONALE:
<the 3-sentence rationale>

═══════════════════════════════════════════════════

NEXT?
  1. Run /devour with this brief loaded (full-spine review)
  2. Run /devour-motion with this brief (motion-only review)
  3. Run /devour-micro with this brief (micro-interaction-only review)
  4. Run /devour-state with this brief (state-only review)
  5. Stop here ... use the brief as a design input for impeccable, Figma, or another tool
═══════════════════════════════════════════════════
```

If the user picks 1-4, dispatch the chosen review skill with `--brief <path>` and the same `$REPO` and target as the brief.

If the user picks 5, stop. The brief is the artifact.

If the user gives no answer, default to 5 (stop). Do not assume the user wants a review.

---

## Inheritance rules (precedence)

When a field could be set from multiple sources, this is the precedence (highest wins):

1. **CLI flag** (e.g., `--register <value>`).
2. **User answer in Step 2** (the user explicitly accepted or overrode the inherited value).
3. **Per-surface override** in `DEVOUR.md` (matched by path prefix in Step 1).
4. **Top-level `DEVOUR.md`** field.
5. **Built-in default** (used only if `DEVOUR.md` is missing entirely).

When inheritance applies, never silently override. Always tell the user what's being inherited and let them confirm or change.

---

## What `devour-brief` does NOT do

- **Does not write to `DEVOUR.md`.** The brief is target-scoped; `DEVOUR.md` is project-scoped. They live separately.
- **Does not run a review.** The brief is the brief. Reviews run via the review skills with `--brief <path>`.
- **Does not generate design.** The brief is for the designer or the AI tool that will generate the design.
- **Does not enforce.** A brief is a target you can fail. Failing the brief is honest; pretending a target was different than what was articulated is not.
- **Does not auto-invoke.** If a review skill is run without `--brief`, it falls back to `DEVOUR.md`-only context. The brief is opt-in.

---

## Voice

Devour-brief's voice is:

- **Direct** ... questions are short and answerable. No leading prompts.
- **Calm** ... no hype. The brief is a small artifact; treat it that way.
- **Honest about ambiguity** ... if the user can't name a success criterion, accept that and note it. Don't force precision that isn't there.
- **Generous with inheritance** ... never re-ask what `DEVOUR.md` already answered, unless the per-target answer differs.

Avoid:

- Treating the brief as a discovery interview. It's 4-6 questions, not 20.
- Forcing a Target line when the inputs aren't there. If the user can't articulate the target, the brief should say so plainly, not paper it over.
- Quoting the brief back at the user as if they didn't write it.

---

## See also

- [`../devour-teach/SKILL.md`](../devour-teach/SKILL.md) ... project-wide context (run once per repo)
- [`../devour/SKILL.md`](../devour/SKILL.md) ... full-spine review (consumes briefs via `--brief <path>`)
- [`../devour-motion/SKILL.md`](../devour-motion/SKILL.md) ... motion review (consumes briefs)
- [`../devour-micro/SKILL.md`](../devour-micro/SKILL.md) ... micro-interaction review (consumes briefs)
- [`../devour-state/SKILL.md`](../devour-state/SKILL.md) ... state review (consumes briefs)
- [`../../references/principles-map.md`](../../references/principles-map.md) ... per-principle source citations
