---
name: devour-micro
description: "Deep micro-interaction review against principles #3 (commit on intent, not on contact), #6 (the fingertip and the cursor are not the same), and #11 (match the metaphor to the medium). Use when hover behavior feels jittery, touch targets feel wrong, or UI components are using the wrong interaction pattern for their context. Traces findings back to Rauno Freiberg, Bruce Tognazzini, Bill Buxton, Don Norman, the original iPhone team."
argument-hint: "[target] [--terse]"
user-invocable: true
license: Apache 2.0. See NOTICE.md for full attribution to the design lineage this skill stands on.
---

> The interface should know whether you are touching it or passing through.
> ... after Rauno Freiberg, Devouring Details, "Inferring Intent"

Devour-micro is a focused review of three principles that govern small-surface interaction behavior: commit on intent (#3), ergonomics (#6), and metaphor matching (#11). These are the principles most commonly violated in incremental feature development, where each individual addition seems reasonable and the cumulative effect is a surface that is jittery, difficult to use, or built from the wrong components for the job.

---

## When to use

- Hover behavior is flickery or triggering when it shouldn't.
- Touch targets feel small or imprecise on mobile.
- A component was chosen because it was available, not because it fits the interaction model (modals for single edits, toasts for persistent errors, pagination where scroll fits).
- A prior `devour` run produced 🔴 or 🟡 findings in principles #3, #6, or #11 and the user wants depth.
- The user is building: tooltips, nav menus, hover cards, modals, dialogs, bottom sheets, drawers, toasts, banners, popovers, inline edits, command menus, or any component where touch/click affordance and interaction pattern matter.
- The user references exemplars from the lineage: Linear nav, iOS keyboard, Notion slash menu, cmdk, Vaul, Raycast.

For motion-specific review (spring physics, honest animation, sequence), use [`devour-motion`](../devour-motion/SKILL.md). For state and reversibility review, use [`devour-state`](../devour-state/SKILL.md). For a full-spine pass, use [`devour`](../devour/SKILL.md).

---

## MANDATORY PREPARATION

This skill requires project context established by `/devour-teach`.

**If the project has not run `/devour-teach` yet:**

1. STOP. Do not proceed.
2. Tell the user: "Devour needs project context first. Running `/devour-teach` to set up."
3. Invoke `devour-teach`. Follow it through to completion.
4. Return here.

**If context exists:**

Read the `Devour Context` block. Check **Primary surface** and **Principle weighting**. For a productivity tool, #3 and #6 weight highest. For a mobile-first app, #6 is most critical. For a content platform, #11 is most likely to surface mismatches.

---

## Process

**Always execute this process from scratch on each invocation.** If prior devour-micro output exists in session memory, ignore it. Re-read targets, re-run browser-MCP detection, re-verify findings. Never reproduce, paraphrase, or replay cached output. If the user asks to "re-run," "run again," or "check again," they are asking for a fresh execution of the full process.

### Step 0 ... Check for --terse flag

Read `$ARGUMENTS`. If `--terse` is present, set output mode to **terse**. Strip `--terse` before passing arguments to Step 1. Terse mode keeps the same rigor and the same three-principle scope but strips teaching prose from each finding. The APPLY? prompt and INTERACTIONS BETWEEN FINDINGS block are unchanged in both modes.

### Step 1 ... Establish target

If `$ARGUMENTS` is provided (after stripping `--terse`), the target is that file, component, or pattern. Read it.

If `$ARGUMENTS` is empty:
- Default to changed files in the current branch filtered to design-relevant files.
- If no changes, ask the user.

Look for immediately:
- `onMouseEnter`, `onMouseLeave`, `onHoverStart`, `onHoverEnd`, `useHover` ... intent commit patterns
- `onClick`, `onPointerDown`, `onTouchStart` ... touch interaction patterns
- Touch target sizing: any `w-X h-X` on clickable elements, `size:`, `p-X` on buttons
- Component usage: `<Dialog>`, `<Modal>`, `<Drawer>`, `<Sheet>`, `<Toast>`, `<Popover>`, `<Tooltip>`, `<Pagination>` ... check if the right component is being used for the job

#### Step 1a ... Detect a browser-driving MCP

Before reading any code, check whether any browser-driving MCP is available in your tool set. Devour-micro does not require a specific browser MCP; any tool family that lets you open pages, navigate, evaluate scripts, and (ideally) take screenshots will work.

Common browser MCPs to look for, by tool-name prefix:

- `mcp__chrome-devtools__*` (chrome-devtools-mcp ... most common)
- `mcp__playwright__*` (Playwright MCP)
- `mcp__browser__*` or `mcp__browser-mcp__*` (BrowserMCP)
- `mcp__browserbase__*` (Browserbase)
- `mcp__puppeteer__*` (Puppeteer MCP)

The minimum capabilities devour-micro needs are: open a URL, evaluate JavaScript on the page, and (preferably) take a screenshot or DOM snapshot. Different MCPs name these differently. Identify the relevant tools by capability, not by exact name.

**If NO browser-driving MCP is available:**

Tell the user once, plainly: "I don't have a browser-driving MCP available in this session, so this will be a code-only review. Almost every finding in this skill is runtime-dependent ... hover delays, touch target hit areas, and modal/overlay behavior all need browser verification. To enable that, install any browser MCP (chrome-devtools, Playwright, BrowserMCP, etc.) and re-run."

Then proceed with code-only review. Mark the output `Reviewed: code only`.

**If a browser MCP IS available:**

Try the equivalent of `list_pages` first. Three cases:

1. **A page matching the dev server is already open** (look for `localhost`, `127.0.0.1`, or a known dev URL from `package.json`'s `dev` script): use it. Select/focus it.
2. **No matching page is open, but you can find the dev server URL** (read `package.json`, look for `next dev`, `vite`, `pnpm dev`, port hints; default to `http://localhost:3000` for Next/Vite, `http://localhost:5173` for Vite, `http://localhost:5174` for Astro): open it.
3. **No dev server detectable**: ask the user once: "What URL is your dev server on?" If the user doesn't have one running, fall back to code-only and mark accordingly.

Mark the output `Reviewed: code + browser (<MCP name>)` once you have a live page.

**When a browser MCP IS available and a dev server is running:**

For any finding that depends on runtime behavior, **do the browser verification before emitting the finding**. For devour-micro, this covers nearly every finding:

- **Hover delay / intent commit** (principle #3): dispatch `mouseenter`/`mouseleave` events via JavaScript, measure how quickly the UI responds. Observe whether tooltips, hover cards, or nav sub-menus fire on cursor pass-through.
- **Touch target hit areas** (principle #6): inspect computed dimensions on clickable elements. Evaluate against the 44px standard. Check whether padding expands the tap area beyond the visual bounds.
- **Modal / overlay / metaphor behavior** (principle #11): navigate to the surface that triggers each component. Open it. Observe behavior at runtime ... dismiss behavior, snap points, spring feel, whether the metaphor fits the data shape.

Rules:

- If verification **confirms** the issue: emit the finding tagged `[browser-confirmed]`.
- If verification **shows no issue**: do NOT emit the finding. The code-based hypothesis was wrong; observation overrides it.
- Do NOT emit findings that say "verify in browser first" or "test this in browser." The skill has browser access; it does that work itself.

**When a browser MCP IS available but NO dev server is running:**

Ask the user once: "I have browser MCP access but no dev server. Should I try to detect a running instance, or should I proceed with code-only review?" If they provide a URL, use it. Otherwise proceed code-only.

---

### Step 2 ... Apply principles #3, #6, and #11

---

#### Principle #3 ... Commit on intent, not on contact

> Distinguish hover-passing-through from hover-with-intent.

**Source:** Bruce Tognazzini, First Principles of Interaction Design, the prediction principle (asktog.com, Layer 2). Rauno Freiberg, Devouring Details, "Inferring Intent" chapter (Layer 4). Linear team, hover delay patterns (Layer 3).

**The core idea:** The cursor passes over many elements on the way to its actual target. Triggering state on every element the cursor contacts while in transit creates visual noise, breaks spatial coherence, and makes interfaces feel reactive in the wrong way. The interface should predict whether the user *intends* to interact with an element, not just whether they're physically over it.

The canonical test: move the cursor from the top-left of the screen to an element in the center. Count how many tooltips, hover cards, or state changes fire in transit. More than zero is a failure.

**Anti-patterns to catch:**

Instant `onMouseEnter` tooltip with no commit delay:
```tsx
// Anti-pattern: fires the moment cursor enters
<Tooltip>
  <TooltipTrigger onMouseEnter={() => setOpen(true)} />
</Tooltip>

// Or in shadcn/Radix:
<TooltipProvider delayDuration={0}> {/* delayDuration={0} is the anti-pattern */}
```

Instant dismiss on `mouseLeave`:
```tsx
// Anti-pattern: disappears the moment cursor exits, punishes slight overshoot
onMouseLeave={() => setOpen(false)}
// Should have a closing delay of ~100ms to allow cursor to reach the tooltip
```

Nav items that activate on `mouseEnter` without an intent delay:
```tsx
// Anti-pattern: nav sub-menu opens immediately
onMouseEnter={() => setActiveNav(item.id)}
// No delay means hovering over the nav bar en route to something else activates menu items
```

Hover cards or preview panels that open instantly on cursor contact. These interrupt reading while the user is simply scanning.

**Tactic patterns to implement:**

A simple, robust intent delay with cleanup:
```tsx
const intentTimerRef = useRef<NodeJS.Timeout>()

const handleMouseEnter = () => {
  intentTimerRef.current = setTimeout(() => {
    setOpen(true)
  }, 80) // 80ms is Rauno's recommended baseline; Linear uses ~100ms
}

const handleMouseLeave = () => {
  clearTimeout(intentTimerRef.current)
  // Optional: brief close delay for tooltip reachability
  setTimeout(() => setOpen(false), 100)
}
```

For Radix UI / shadcn Tooltip, the correct configuration:
```tsx
<TooltipProvider delayDuration={400}> {/* default: 700ms; 400ms is a good balance */}
  <Tooltip>
    <TooltipTrigger asChild>...</TooltipTrigger>
    <TooltipContent>...</TooltipContent>
  </Tooltip>
</TooltipProvider>
```

**Linear nav hover** is the canonical exemplar. The brief commit delay (~80-100ms) prevents hover flicker as the cursor passes across nav items en route to its destination, while still feeling responsive when the user actually pauses on an item.

---

#### Principle #6 ... The fingertip and the cursor are not the same

> Touch targets, hit boxes, ergonomic distance, Fitts's law.

**Source:** Bill Buxton, Sketching User Experiences, input-first thinking (Layer 2). Bruce Tognazzini, Fitts's law applications (Layer 2). Imran Chaudhri/Bas Ording, original iPhone interaction design, 44pt standard (Layer 3). Rauno Freiberg, Devouring Details, "Ergonomic Interactions" chapter (Layer 4).

**Fitts's law:** The time to acquire a target is a function of target size and distance. Small targets far from the cursor take longer to hit. Obvious in theory; routinely ignored in practice.

**The fingertip mismatch:** The mouse cursor is a single pixel. A fingertip is approximately 10-15mm in contact diameter. These require different design choices. A 24px close button is a precise mouse click. It is a frustrating mobile interaction.

**Anti-patterns to catch:**

Touch targets below 44pt (44px on 1x screens, 88px on 2x):
```tsx
// Anti-pattern: icon button too small for touch
<button className="w-6 h-6 flex items-center justify-center">
  <XIcon />
</button>
// Should be minimum w-11 h-11 (44px) on touch surfaces

// Anti-pattern: close button in a dialog header with tight padding
<DialogHeader className="flex justify-between items-center p-3">
  <button className="p-1"><XIcon className="w-4 h-4" /></button>
```

Hit boxes that match visual bounds but should be larger:
```tsx
// Anti-pattern: interactive element with no hit zone expansion
<Link className="text-sm text-blue-500 underline">View details</Link>
// Text links are tiny tap targets. Should have explicit padding/min-height.

// Better:
<Link className="text-sm text-blue-500 underline py-2 px-1 -mx-1 inline-block">
```

Close buttons in ergonomically difficult positions. On mobile, corners are *harder* to reach than center-screen, not easier. Bottom-sheet close actions placed in the top-right corner require the user to reach across the screen.

Inputs that don't expand hit zones to include the label:
```tsx
// Anti-pattern: label and input as separate tap targets
<label>Email</label>
<input type="email" />

// Should be: label wraps input, entire row is tappable
<label className="flex flex-col gap-1 cursor-pointer">
  <span>Email</span>
  <input type="email" className="..." />
</label>
```

Inline icon actions on list rows with no visual hit zone:
```tsx
// Anti-pattern: three-dot menu icon with 16px padding
<button className="p-1"><MoreIcon className="w-4 h-4" /></button>
// The visible element is tiny; the tap area should be large regardless of visibility
```

**iOS keyboard as the canonical exemplar:** Each key is visually small but has a hit zone that grows during touch-down to match what the finger expects. The visual boundary and the interactive boundary are intentionally different. This is the correct model for touch interfaces.

**Tactic gaps:**

Desktop-designed components deployed to mobile views without touch target adjustment. Check if the project has a mobile breakpoint and whether touch targets change at that breakpoint.

Actions that require cursor precision on a surface that may be used on touch devices. Resize handles, drag handles, and inline edit triggers are all common offenders.

---

#### Principle #11 ... Match the metaphor to the medium

> Do not paginate what should scroll. Do not modal what should be inline. Do not toast what should be persistent.

**Source:** Bill Buxton, medium-shapes-the-message thesis (Layer 2). Don Norman, mapping (Layer 2). Rauno Freiberg, Devouring Details, "Interaction Metaphors" chapter (Layer 4). Loren Brichter's pull-to-refresh as paradigm-creating example (Layer 3).

**The core idea:** Every UI component embodies a metaphor about how interaction works. A modal says "stop everything, handle this first." A toast says "this just happened; it will pass." Pagination says "this content has chapters." When the component's metaphor doesn't match the data's nature or the user's task, friction accumulates. The user does the right thing in the wrong context, over and over, until they stop doing the right thing.

**Anti-patterns to catch:**

**Modal for a single-field edit:**
```tsx
// Anti-pattern: modal for inline rename
<Dialog>
  <DialogContent>
    <DialogHeader><DialogTitle>Rename project</DialogTitle></DialogHeader>
    <input type="text" defaultValue={project.name} />
    <DialogFooter>
      <Button>Cancel</Button>
      <Button>Save</Button>
    </DialogFooter>
  </DialogContent>
</Dialog>
// This should be an inline edit on the name element itself
```

**Pagination where infinite scroll fits the data shape:**
```tsx
// Anti-pattern: paginated news feed
<Feed items={feedItems} />
<Pagination current={page} total={totalPages} />
// A feed is not a document with chapters; pagination imposes structure
// the data doesn't have
```

**Toast for a persistent or actionable error:**
```tsx
// Anti-pattern: toast for a system-level error that requires user action
toast.error("Your session has expired. Please log in again.")
// A session expiry requires navigation; a toast auto-dismisses
// Use a persistent banner or a redirect
```

**Popover used as a modal:**
```tsx
// Anti-pattern: popover with a form inside it
<Popover>
  <PopoverContent>
    <form>...</form> {/* Form in a popover: if it has required fields
                        and a submit, it's a modal masquerading as a popover */}
  </PopoverContent>
</Popover>
```

**Bottom sheet on desktop:**
```tsx
// Anti-pattern: Vaul drawer used as primary detail surface on desktop
<Drawer>
  <DrawerContent> {/* Bottom sheet is a mobile metaphor.
                     On desktop, this is a panel, dialog, or sidebar */}
    <DetailPanel />
  </DrawerContent>
</Drawer>
```

**Tooltip on mobile:**
```tsx
// Anti-pattern: tooltip used as the only affordance for an action on mobile
<Tooltip content="Delete">
  <IconButton><TrashIcon /></IconButton>
</Tooltip>
// Tooltips require hover. Mobile has no hover. The label is invisible.
```

**Tactic gaps:**

Long-scrolling content with no jump links or table of contents ... the content should have navigation if it warrants pagination, but often warrants neither.

An inline edit surface that opens a modal instead of editing in place. If Notion does it inline, ask why you need a modal.

**Notion slash menu** as the canonical metaphor-match exemplar: the action surface is inline in the document, not a modal that interrupts. The writing metaphor and the command surface are the same thing. When you press `/`, you are still in the document. This is the correct model for document-style editing.

**Vaul drawer** as the correct mobile metaphor: bottom sheet for mobile, with proper snap points and spring physics. The sheet behaves like a physical surface the user lifts. Do not use it on desktop without explicit, considered justification.

**cmdk** as the canonical command surface: keyboard-navigable, intent-aware (hover commit delays), appears on explicit invocation, not on cursor contact. The metaphor is a palette, not a menu.

---

### Step 3 ... Write findings

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

- **🔴 BREAKS** ... the principle violation actively hurts usability: touch target is unreachable; tooltip fires on every cursor pass; modal is used where inline edit is the right answer and the user has to interrupt their flow to complete a trivial task.
- **🟡 DRIFTS** ... not catastrophic but the surface is drifting: hover delays are slightly too short; a touch target is 36px when 44px is the standard; a popover is getting close to modal territory.
- **🟢 OPPORTUNITY** ... the principle is not violated but a higher-craft move is available: intent delay could be tuned; touch targets could be expanded without visual change; a metaphor mismatch that affects a secondary surface rather than a primary task.

**Finding annotation:** Each finding should be tagged immediately after the severity marker:

- `[code-confirmed]` ... finding was verified from static code alone
- `[browser-confirmed]` ... finding was verified by driving the live UI via browser MCP

If no browser MCP is available, all findings are `[code-confirmed]`. If browser MCP is available, findings touching hover intent (#3), touch target sizing (#6), and overlay/metaphor behavior (#11) should be `[browser-confirmed]` by actually running the check before emitting the finding. Devour-micro findings are more runtime-dependent than any other skill in the family ... code-only review on these three principles is structurally limited.

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

The header block, severity groupings, INTERACTIONS BETWEEN FINDINGS block, MICRO SUMMARY block, and APPLY? block all remain in terse mode, unchanged.

**Verbose output format (default):**

```
═══════════════════════════════════════════════════
DEVOUR MICRO REVIEW: <target>
Context: <primary surface + principle weighting from Devour Context>
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
MICRO SUMMARY
N breaks · N drifts · N opportunities
Principles reviewed: #3 (intent), #6 (ergonomics), #11 (metaphor)
Reviewed: code only | code + browser (<MCP name>)
═══════════════════════════════════════════════════

═══════════════════════════════════════════════════
APPLY?
  1. Apply all 🔴 BREAKS (N findings)
  2. Apply all 🔴 + 🟡 (N findings)
  3. Apply everything (N findings)
  4. Cherry-pick ... tell me which (e.g., "1, 3, and 5" or specific finding name)
  5. Review only ... apply nothing
═══════════════════════════════════════════════════
```

After printing the review, **always print the APPLY? block as the final lines of output.** Do not skip it.

If code-only (no browser MCP found or no dev server), append above the APPLY? block: "Reviewed code only. Most findings in this skill are runtime-dependent; hover delays, touch target hit areas, and overlay behavior need browser verification before you trust them. Re-run with a browser MCP + dev server to confirm."

**Before printing the SUMMARY block, check for inter-finding conflicts.** Two findings can be in tension when fixing one weakens the other, or when both share a root cause that requires a single structural change to resolve. If any conflicts exist, name them in the INTERACTIONS BETWEEN FINDINGS block (between the findings and the SUMMARY). If none, skip the block entirely.

When applying:

- **Show the diff** before each file change. Brief, just the hunks.
- **Apply 🔴 BREAKS without further confirmation** if the user picked option 1, 2, or 3.
- **Ask once per 🟡 DRIFT or 🟢 OPPORTUNITY** that involves a real taste call. Skip the ask if the fix is mechanical.
- **After all fixes are applied, ask if the user wants to commit.** Do not auto-commit.

---

## Voice

Specific, calm, citation-heavy. Name the principle. Cite the source. Describe the fix in code. Leave the final call to the user.

Do not say "this component is wrong for this use case" without naming which principle it violates and which component would be right. Do not say "bad UX." Do not say "users will struggle." Say what the principle requires and what the code currently does instead.

---

## See also

- [`../devour/SKILL.md`](../devour/SKILL.md) ... full-spine review
- [`../../references/principles-map.md`](../../references/principles-map.md) ... source citations for #3, #6, #11
- [`../../references/exemplars.md`](../../references/exemplars.md) ... Linear nav, iOS keyboard, Notion slash menu, cmdk, Vaul, Raycast
- [`../../references/anti-patterns.md`](../../references/anti-patterns.md) ... full anti-pattern catalog for micro-interaction principles
- [`../../references/reading-list.md`](../../references/reading-list.md) ... Rauno's "Inferring Intent," Tognazzini's First Principles
