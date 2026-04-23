---
name: devour-motion
description: "Deep motion review against principles #1 (honest motion), #2 (physics over duration), and #5 (sequence carries meaning). Use when the user wants an animation pass, a motion-specific craft review, or when motion in a component feels wrong but the reason isn't named. Traces findings back to the lineage: Emil Kowalski, Rauno Freiberg, Loren Brichter, Chaudhri/Ording at Apple, Dieter Rams."
argument-hint: "[target] [--terse]"
user-invocable: true
license: Apache 2.0. See NOTICE.md for full attribution to the design lineage this skill stands on.
---

> Real movement has mass. Eased durations have none.
> ... after Rauno Freiberg, Devouring Details, "Simulating Physics"

Devour-motion is a focused review of three motion principles: honest motion (#1), physics over duration (#2), and sequence as meaning (#5). It runs deeper on these three than `devour` does in a full-spine pass. Use it after the main review when motion findings warrant detail, or run it first when the user's concern is specifically about how things move.

Motion cannot be fully reviewed from static code. The findings here will be as good as the code allows, but the spring curves that look right on paper can still feel wrong at 120Hz. State this to the user when relevant.

---

## When to use

- The user says "make this motion feel right," "the animation feels off," or "can you do an animation pass."
- A prior `devour` run produced 🔴 or 🟡 findings in principles #1, #2, or #5 and the user wants depth.
- The user is building a component where motion is central: drawer, dialog, toast, sheet, list, carousel, command menu, drag-and-drop.
- The user has Framer Motion, react-spring, or CSS `@property` animations in the target and wants them reviewed.
- The user references motion work from the lineage: Sonner, Vaul, NotBoring, Brichter, Rauno's spring configs.

For a broader craft review, use [`devour`](../devour/SKILL.md). For micro-interaction review (hover, touch targets, metaphor mismatches), use [`devour-micro`](../devour-micro/SKILL.md).

---

## MANDATORY PREPARATION

This skill requires project context established by `/devour-teach`.

**If the project has not run `/devour-teach` yet:**

1. STOP. Do not proceed.
2. Tell the user: "Devour needs project context first. Running `/devour-teach` to set up."
3. Invoke `devour-teach`. Follow it through to completion.
4. Return here.

**If context exists:**

Read the `Devour Context` block from `.devour-context.md`, `.claude/CLAUDE.md`, or the loaded project instructions. Check the **Motion appetite** and **Principle weighting** fields before reviewing. A product with "none/static" motion appetite should have most of principle #1's findings treated as 🟢 OPPORTUNITIES, not 🔴 BREAKS ... unless motion is present and wrong, which is always a break regardless of appetite.

---

## Process

**Always execute this process from scratch on each invocation.** If prior devour-motion output exists in session memory, ignore it. Re-read targets, re-run browser-MCP detection, re-verify findings. Never reproduce, paraphrase, or replay cached output. If the user asks to "re-run," "run again," or "check again," they are asking for a fresh execution of the full process.

### Step 0 ... Check for --terse flag

Read `$ARGUMENTS`. If `--terse` is present, set output mode to **terse**. Strip `--terse` before passing arguments to Step 1. Terse mode keeps the same rigor and the same three-principle scope but strips teaching prose from each finding. The APPLY? prompt and INTERACTIONS BETWEEN FINDINGS block are unchanged in both modes.

### Step 1 ... Establish target and environment

If `$ARGUMENTS` is provided (after stripping `--terse`), the target is that file, component, or pattern. Read it in full.

If `$ARGUMENTS` is empty:
- Default to changed files in the current branch (`git diff main..HEAD --name-only`, filtered to `.tsx`, `.jsx`, `.css`, `.scss`).
- If no changes, ask the user what to review.

**Motion cannot be fully reviewed from static code.** A 200ms `ease-in-out` looks fine in source but may feel hesitant on real hardware. A spring config that math-checks may overshoot at 60Hz. The skill must drive the actual UI when possible.

#### Step 1a ... Detect a browser-driving MCP

Before reading any code, check whether any browser-driving MCP is available in your tool set. Devour-motion does not require a specific browser MCP; any tool family that lets you open pages, navigate, and evaluate scripts will work.

Common browser MCPs to look for, by tool-name prefix:

- `mcp__chrome-devtools__*` (chrome-devtools-mcp ... most common)
- `mcp__playwright__*` (Playwright MCP)
- `mcp__browser__*` or `mcp__browser-mcp__*` (BrowserMCP)
- `mcp__browserbase__*` (Browserbase)
- `mcp__puppeteer__*` (Puppeteer MCP)

The minimum capabilities devour-motion needs are: open a URL, evaluate JavaScript on the page, and take a DOM snapshot (accessibility tree or equivalent). Screenshots are optional and should be used sparingly. Use `evaluate_script` to read timing functions, computed transforms, `performance.now()` timestamps, and animation state directly from the DOM. Different MCPs name these differently. Identify the relevant tools by capability, not by exact name.

**DOM-first rule:** motion findings are timing-based, and timing is a number. Read the number with `evaluate_script`; do not infer it from a screenshot. Measure animation duration by capturing `performance.now()` before/after the animation completes, not by comparing image frames. Read `getComputedStyle(el).transitionDuration` and `transitionTimingFunction` directly. Screenshots are appropriate only for compositing artifacts or visual-perception-of-motion judgments that the DOM cannot express.

**If NO browser-driving MCP is available:**

Tell the user once, plainly: "I don't have a browser-driving MCP available in this session, so this will be a code-only review. Motion findings about felt timing and runtime behavior need browser verification. To enable that, install any browser MCP (chrome-devtools, Playwright, BrowserMCP, etc.) and re-run."

Then proceed with code-only review. Mark the output `Reviewed: code only`.

**If a browser MCP IS available:**

Try the equivalent of `list_pages` first. Three cases:

1. **A page matching the dev server is already open** (look for `localhost`, `127.0.0.1`, or a known dev URL from `package.json`'s `dev` script): use it. Select/focus it.
2. **No matching page is open, but you can find the dev server URL** (read `package.json`, look for `next dev`, `vite`, `pnpm dev`, port hints; default to `http://localhost:3000` for Next/Vite, `http://localhost:5173` for Vite, `http://localhost:5174` for Astro): open it.
3. **No dev server detectable**: ask the user once: "What URL is your dev server on?" If the user doesn't have one running, fall back to code-only and mark accordingly.

Mark the output `Reviewed: code + browser (<MCP name>)` once you have a live page.

#### Step 1b ... Read the code

After environment is established, read the target file(s) in full. Specific patterns to look for immediately:

- Any `transition:` or `animation:` in CSS or inline styles
- Any Framer Motion imports (`motion`, `AnimatePresence`, `useSpring`, `useMotion*`)
- Any react-spring imports (`useSpring`, `useTransition`, `useTrail`, `animated.*`)
- Any `@property` or `linear()` easing in CSS
- Any keyframes definitions in `globals.css` or equivalent
- Any `nth-child` patterns (often hardcoded stagger ceilings)

#### Step 1c ... Drive the UI for findings that need it

For each finding that depends on felt experience rather than code structure, drive the page. Examples:

- **Sheet timing or dialog entry feel** ... navigate to the page that triggers it, click the trigger, observe. Use `evaluate_script` to inspect computed timing functions (`getComputedStyle(el).transitionDuration`, `transitionTimingFunction`) and measure actual elapsed time via `performance.now()` bracketing the trigger. A screenshot is a last resort if the animation cannot be reasoned about from computed styles.
- **Hover commit delays** ... evaluate JavaScript to dispatch `mouseenter`/`mouseleave` events on target elements, measure response.
- **Stagger sequences** ... navigate to the relevant grid. Use `evaluate_script` to observe each item's animation state directly: read `animation-delay`, `transition-delay`, or inline motion props on each item; trigger the entrance (scroll into view, filter change, etc.) and bracket with `performance.now()` to confirm whether all items animate or only the first N. This is faster and more precise than reading screenshots at multiple timestamps.
- **Filter-change re-animation** (the bug devour catches): apply a filter, watch whether existing cards re-trigger their entrance animation. This is the canonical case for browser-required verification.
- **Reduced-motion behavior** ... evaluate JavaScript to set `prefers-reduced-motion: reduce` via CSS or media-query emulation, re-trigger interactions.

Annotate findings with `[code-confirmed]` or `[browser-confirmed]` so the user knows which findings have been verified at runtime.

**Rules for emitting findings when browser MCP is available:**

- If verification **confirms** the issue: emit the finding tagged `[browser-confirmed]`.
- If verification **shows no issue**: do NOT emit the finding. The code-based hypothesis was wrong; observation overrides it.
- Do NOT emit findings that say "verify in browser first" or "test this in browser." The skill has browser access; it does that work itself.

---

### Step 2 ... Apply principles #1, #2, and #5

For each principle, scan the target for both **anti-pattern matches** (the failure mode) and **tactic gaps** (where the right approach is missing entirely).

---

#### Principle #1 ... Honest motion

> If you animate it, it must communicate something the static state could not.

**Source:** Dieter Rams principle 6 ("Good design is honest"), Don Norman on feedback as affordance, Emil Kowalski's "You Don't Need Animations" (emilkowal.ski).

**The question to ask for every animation:** "If this motion were removed, would the user lose information?" If no, the motion is decorative and likely wrong.

**Anti-patterns to catch:**

- `transition: opacity 200ms` on a static element that isn't changing state ... the fade is decorative, communicates nothing
- Entrance animation on content that the user didn't trigger ... a card that fades in on page load when the user didn't navigate to it
- Loading skeleton with shimmer that runs longer than the actual load time ... the shimmer outlives its informational purpose and becomes decoration
- `@keyframes pulse` on a badge that is not changing state (suggesting pending activity when there is none)
- Animated counters counting up from zero to a number that was always known at render time ... the count communicates nothing that the final number doesn't, it just performs motion
- Motion that has no end state ... continuous ambient animation that never resolves, suggesting perpetual activity

**Tactic gaps to catch:**

- A form submission that shows nothing between click and response ... no motion to communicate work-in-progress, user gets no feedback that intent was registered
- A list item that appears or disappears without any motion to communicate causality ... the item is just gone
- Navigation that transitions instantly with no motion to communicate spatial relationship

**The "You Don't Need Animations" check:**

Apply Emil Kowalski's test: for each animation, ask whether you actually need it. Not whether it's nice to have, whether the user *needs* it to understand what just happened or what is about to happen. If the answer is no, the motion should be removed, not refined.

---

#### Principle #2 ... Physics over duration

> Real movement has mass, springs, and damping. Eased durations betray themselves as animation.

**Source:** Imran Chaudhri and Bas Ording (original iPhone rubber-banding), Loren Brichter (pull-to-refresh), Rauno Freiberg (Devouring Details, "Simulating Physics"), Emil Kowalski (Animations on the Web, spring chapters).

**The core idea:** A tween is a function of time. A spring is a function of force. Users feel the difference, even when they can't name it. When you move a thing (a drawer, a dialog, a list item dragged from one position to another), that thing should behave as if it has mass and encounters resistance. A tween slides; a spring *arrives*.

**Anti-patterns to catch:**

Any `transition: transform Xms ease-*` on a movable element:
```css
/* catches these patterns */
transition: transform 300ms ease-out;
transition: transform 200ms ease-in-out;
transition: transform 400ms cubic-bezier(0.4, 0, 0.2, 1);
```

These are linear-time tweens dressed in easing clothing. They still run for a fixed duration regardless of the user's gesture velocity. A spring's duration varies with input velocity; a tween's doesn't.

Any `transition: all Xms` on an interactive element ... "all" prevents targeted physics; the geometry and the color should not move at the same speed.

Linear easing (`transition: transform 200ms linear`) on anything that moves spatially. Linear is always wrong for spatial movement. It is mechanical, not physical.

A tween-based transition between two modal states (dialog entering and exiting) where a spring would close the loop naturally:
```tsx
// Anti-pattern: tween
<Dialog className="transition-transform duration-300 ease-out" />

// What to suggest instead: spring via Framer Motion
<motion.div
  initial={{ scale: 0.95, opacity: 0 }}
  animate={{ scale: 1, opacity: 1 }}
  exit={{ scale: 0.95, opacity: 0 }}
  transition={{ type: "spring", stiffness: 400, damping: 30 }}
/>
```

**Tactic gaps to catch:**

Draggable elements using CSS transition instead of spring physics on release:
```tsx
// Missing: spring snap-back
const dragControls = useDragControls()
// Should have spring-based constraints and release behavior
```

Vaul and Sonner as canonical spring exemplars: if the target is a drawer or toast, compare its motion to Vaul/Sonner behavior. Vaul uses spring physics for the drag-and-release; Sonner uses spring physics for toast enter/exit. Any drawer or toast that feels different from these is worth flagging.

**Framer Motion spring config reference:**
```tsx
// Responsive snap (dialogs, dropdowns)
transition={{ type: "spring", stiffness: 400, damping: 30 }}

// Heavy drag (drawers, sheets)
transition={{ type: "spring", stiffness: 300, damping: 40, mass: 1.2 }}

// Light, playful (NotBoring-style)
transition={{ type: "spring", stiffness: 500, damping: 25 }}
```

**CSS `@property` + `linear()` for spring-like easing:**
```css
/* Spring approximation without JS */
@property --translateY {
  syntax: '<length>';
  inherits: false;
  initial-value: 0px;
}
transition: --translateY 600ms linear(
  0, 0.009, 0.035 2.1%, 0.141%, 0.281 6.7%, 0.723 12.9%,
  0.938 16.7%, 1.04, 1.098 24.3%, 1.121, 1.124, 1.121,
  1.098 29.3%, 1.04, 1 35.5%, 0.949, 0.929 41.6%, 0.938,
  0.962 52.7%, 0.988 61.8%, 1
);
```

---

#### Principle #5 ... Sequence carries meaning

> Stagger only when order matters. Otherwise, simultaneity.

**Source:** Edward Tufte on data-ink ratio (every element earns its place, sequence earns its order), Bill Verplank's do/feel/know model, Rauno Freiberg (Devouring Details, "Motion Choreography").

**The core idea:** When you stagger animations, you communicate to the user that the items appeared in that order for a reason. If there is no reason for the order, the stagger is a lie. It tells the user a causal story that doesn't exist.

Conversely, when two things are causally linked, simultaneous animation hides the relationship. If pressing a button causes a panel to expand, those two things should animate in a way that shows the connection, not at the same moment in identical ways.

**Anti-patterns to catch:**

Staggered list entrance where the order is arbitrary:
```tsx
// Anti-pattern: stagger on arbitrary-ordered list
{users.map((user, i) => (
  <motion.div
    initial={{ opacity: 0, y: 20 }}
    animate={{ opacity: 1, y: 0 }}
    transition={{ delay: i * 0.05 }}
    key={user.id}
  >
```
If users are sorted alphabetically or by ID, the stagger implies rank that doesn't exist. Ask: why does the first user "arrive" before the last? If there's no answer, the stagger is decoration.

Simultaneous animation where causality matters:
```tsx
// Anti-pattern: everything enters at once when parent caused children
<motion.div animate={{ opacity: 1 }}>
  <motion.div animate={{ opacity: 1 }}>child 1</motion.div>
  <motion.div animate={{ opacity: 1 }}>child 2</motion.div>
</motion.div>
// Parent and children entering together hides the parent-child relationship
```

Missing follow-through on chained interactions. If clicking a row expands a detail panel below it, the row and the panel should have motion that tells the causal story: row indicates selection, *then* panel enters. If both animate simultaneously and identically, the user has to infer the relationship.

**Tactic gaps to catch:**

Staggered list where the order *does* matter (ranked items, priority queue, step-by-step instructions) but there is no stagger ... adding stagger here would communicate the ordering.

A tab switch where both the outgoing and incoming content animate simultaneously and identically ... they could animate in sequence (outgoing exits, incoming enters) to communicate the transition through space.

**Linear's filter-pill animation** is the canonical positive exemplar: when a filter is added, existing pills shift first, then the new pill enters. The sequence tells the causal story: "your new filter joined the existing ones."

---

### Step 3 ... Write findings

Each finding follows the standard devour format:

```
[#N PRINCIPLE NAME] - <severity>
File: <path>:<line range>
Symptom:
  <one or two sentences describing the observed code/behavior>
Principle:
  <one sentence: the principle, what it requires here>
Tactic:
  <the specific change, with code>
Reference:
  <citation: lineage source + canonical exemplar>
```

**Severity scale:**

- **🔴 BREAKS** ... the motion principle is violated in a way that is actively wrong: a drawer using CSS transition instead of spring physics; a stagger on ordered content that communicates false causality; decorative motion that fires on user action and blocks interaction.
- **🟡 DRIFTS** ... the motion is not catastrophically wrong but is drifting: easing curves that are close to correct but not quite spring behavior; a stagger that's mostly harmless but unprincipled; honest motion that could be slightly more informative.
- **🟢 OPPORTUNITY** ... the motion is absent where it would add real information, or is correct but a higher-craft version is available: spring physics available that would improve the feel; stagger that would add meaning to an ordered list.

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

The header block, severity groupings, INTERACTIONS BETWEEN FINDINGS block, MOTION SUMMARY block, and APPLY? block all remain in terse mode, unchanged.

**Verbose output format (default):**

```
═══════════════════════════════════════════════════
DEVOUR MOTION REVIEW: <target>
Context: <motion appetite + principle weighting from Devour Context>
═══════════════════════════════════════════════════

🔴 BREAKS (N findings)
───────────────────────
<findings>

🟡 DRIFTS (N findings)
───────────────────────
<findings>

🟢 OPPORTUNITIES (N findings)
─────────────────────────────
<findings>

═══════════════════════════════════════════════════
INTERACTIONS BETWEEN FINDINGS (only if any)
───────────────────────────────────────────────────
- Finding N and Finding M are partially in tension. <Brief explanation
  of how fixing one affects the other, and what structural change
  resolves both at once if applicable.>
═══════════════════════════════════════════════════

═══════════════════════════════════════════════════
MOTION SUMMARY
N breaks · N drifts · N opportunities
Principles reviewed: #1 (honest motion), #2 (physics), #5 (sequence)
Reviewed: code only | code + browser (<MCP name>)
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

After printing the review, **always print the APPLY? block as the final lines of output.** Do not skip it.

If code-only (no browser MCP found or no dev server), append above the APPLY? block: "Recommend re-running with a browser MCP and dev server for full verification. Spring feel and felt timing cannot be fully confirmed from code."

**Before printing the SUMMARY block, check for inter-finding conflicts.** Two findings can be in tension when fixing one weakens the other, or when both share a root cause that requires a single structural change to resolve. If any conflicts exist, name them in the INTERACTIONS BETWEEN FINDINGS block (between the findings and the SUMMARY). If none, skip the block entirely.

When applying:

- **Show the diff** before each file change. Brief, just the hunks.
- **Apply 🔴 BREAKS without further confirmation** if the user picked option 1, 2, or 3.
- **Ask once per 🟡 DRIFT or 🟢 OPPORTUNITY** that involves a real taste call (e.g., "extending stagger ceiling to 20 ... is this the right max for your grids?"). Skip the ask if the fix is mechanical.
- **After all fixes are applied, ask if the user wants to commit.** Do not auto-commit.

---

## Voice

Specific, calm, citation-heavy. State the principle. Name the source. Recommend the tactic. Leave the call to the user.

Do not say "this looks better." Do not say "modern best practice." Do not say "feels off" without naming why. If you can't cite a principle, the finding doesn't belong in this review.

---

## See also

- [`../devour/SKILL.md`](../devour/SKILL.md) ... full-spine review
- [`../../references/principles-map.md`](../../references/principles-map.md) ... source citations for #1, #2, #5
- [`../../references/exemplars.md`](../../references/exemplars.md) ... Sonner, Vaul, NotBoring iOS, iOS rubber-banding
- [`../../references/anti-patterns.md`](../../references/anti-patterns.md) ... full anti-pattern catalog for motion principles
- [`../../references/reading-list.md`](../../references/reading-list.md) ... Emil Kowalski's courses and essays
