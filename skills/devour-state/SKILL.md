---
name: devour-state
description: "Deep review of state-handling principles #4 (reversibility is craft) and #7 (preserve user state across boundaries). Use when optimistic UI feels unsafe, when users lose work on navigation, when error paths are missing or silent, or when a feature involving async operations needs a state lifecycle audit. Traces findings back to Emil Kowalski (Sonner), the Linear team, Loren Brichter, Andy Matuschak, Bret Victor, Don Norman."
argument-hint: "[target file or component]"
user-invocable: true
license: Apache 2.0. See NOTICE.md for full attribution to the design lineage this skill stands on.
---

> Optimism without a return address is a user experience bug.
> ... after Emil Kowalski, Sonner, and Don Norman's gulf of evaluation

Devour-state is a focused review of the two principles that govern how software handles the gap between what the user did and what the system confirmed: reversibility (#4) and state preservation (#7). These are the principles that separate software that feels *trustworthy* from software that feels slippery.

Most state bugs are not engineering bugs. They are design bugs where the loading state was considered and the error state was not. The submit button got a spinner. The spinner never became an error message. The user tried again. The action ran twice.

---

## When to use

- A component or flow involves optimistic UI (update the UI before server confirmation).
- A form, editor, or filter interface exists and the user might navigate away.
- There are toast notifications or banners in the codebase with no corresponding failure variants.
- A prior `devour` run produced 🔴 or 🟡 findings in principles #4 or #7.
- The user explicitly says: "this feels unsafe," "users are losing their work," "the error handling isn't complete," "our retry doesn't work."
- You are reviewing a flow that has: submit buttons, async mutations, navigation transitions, filter/sort state, tabs, pagination, or draft-like editing.

For motion review (spring physics, animation honesty), use [`devour-motion`](../devour-motion/SKILL.md). For micro-interaction review (hover delays, touch targets, component choice), use [`devour-micro`](../devour-micro/SKILL.md). For a full-spine pass, use [`devour`](../devour/SKILL.md).

---

## MANDATORY PREPARATION

This skill requires project context established by `devour:teach`.

**If the project has not run `devour:teach` yet:**

1. STOP. Do not proceed.
2. Tell the user: "Devour needs project context first. Running `devour:teach` to set up."
3. Invoke `devour:teach`. Follow it through to completion.
4. Return here.

**If context exists:**

Read the `Devour Context` block. Check **Principle weighting**. For a productivity tool, e-commerce, or any tool where users create or modify data, principles #4 and #7 should be high. For a marketing site or content browser, they may be low. Weight the findings accordingly.

---

## Process

### Step 1 ... Establish target

If `$ARGUMENTS` is provided, read the target file(s) in full.

If `$ARGUMENTS` is empty:
- Default to changed files in the current branch filtered to `.tsx`, `.jsx`, `.ts`.
- If no changes, ask the user.

Look for immediately:
- `mutation`, `useMutation`, `mutate`, `mutateAsync` ... TanStack Query / SWR mutations
- `useState` paired with async operations
- `toast()`, `toast.success()`, `toast.error()` ... are all three variants present?
- `onSubmit`, `handleSubmit` ... form submission handlers; look for error handling
- `useRouter().push()` or `navigate()` ... navigation that could lose form state
- `useEffect` with router dependency ... scroll restoration patterns
- `useSearchParams`, `useParams` ... URL-based state that might not persist

---

### Step 2 ... Apply principles #4 and #7

---

#### Principle #4 ... Reversibility is craft

> Every optimistic state needs a believable error path.

**Source:** Dieter Rams, principle 6 ("Good design is honest," Layer 1). Don Norman, gulf of evaluation and feedback (Layer 2). Bret Victor, "Magic Ink" on state visibility (Layer 2). Emil Kowalski, Sonner's lifecycle design (Layer 4).

**The core idea:** Optimistic UI is a trust mechanism. The interface promises the user that their action will succeed before the server confirms it. When you make that promise, you take on a debt: you must show the user what happens if the promise fails. The "Saved!" toast that never shows an error is not optimistic UI; it is a lie that the server might expose.

Dieter Rams principle 6 ("Good design is honest") is the load-bearing claim here. An interface that shows success but cannot show failure is not honest. It is performing success without encoding the full meaning of the operation.

**Anti-patterns to catch:**

`toast.success()` with no corresponding `toast.error()`:
```tsx
// Anti-pattern: incomplete lifecycle
const handleSave = async () => {
  try {
    await saveDocument(data)
    toast.success("Saved!")
  } catch (e) {
    console.error(e) // Error is logged, not shown to user
  }
}
```

`toast.promise()` used correctly is the exemplar. One call, three states:
```tsx
// Correct: toast.promise() forces you to define all three states
toast.promise(saveDocument(data), {
  loading: "Saving...",
  success: "Saved",
  error: "Failed to save. Try again.",
})
```

Silent optimistic rollback:
```tsx
// Anti-pattern: UI updates, server fails, UI reverts with no user signal
const handleToggle = () => {
  setEnabled(!enabled) // optimistic update
  try {
    await updateSetting({ enabled: !enabled })
  } catch (e) {
    setEnabled(enabled) // silent rollback
    // No toast, no banner, no indication anything failed
  }
}
```

Submit button that enters spinner and stays there on network failure:
```tsx
// Anti-pattern: incomplete button lifecycle
const [loading, setLoading] = useState(false)

const handleSubmit = async () => {
  setLoading(true)
  await submitForm(data)
  setLoading(false)
  // If submitForm throws, setLoading(false) never runs
  // Button stays in spinner state permanently
}

// Should be:
const handleSubmit = async () => {
  setLoading(true)
  try {
    await submitForm(data)
    setLoading(false)
    toast.success("Submitted.")
  } catch (e) {
    setLoading(false)
    toast.error("Submission failed. Try again.")
  }
}
```

Delete confirmation that doesn't show what was deleted. "Are you sure?" is not a confirmation; "Delete 'Q4 Report'?" is:
```tsx
// Anti-pattern: generic confirmation
<AlertDialog>
  <AlertDialogDescription>
    Are you sure? This cannot be undone.
  </AlertDialogDescription>

// Better: name the thing being deleted
<AlertDialog>
  <AlertDialogDescription>
    Delete "{document.title}"? This cannot be undone.
  </AlertDialogDescription>
```

**TanStack Query optimistic update with rollback** is the correct implementation model:
```tsx
const mutation = useMutation({
  mutationFn: updateItem,
  onMutate: async (newItem) => {
    // Cancel outgoing refetches
    await queryClient.cancelQueries({ queryKey: ['items', newItem.id] })
    // Snapshot current value
    const previousItem = queryClient.getQueryData(['items', newItem.id])
    // Optimistically update
    queryClient.setQueryData(['items', newItem.id], newItem)
    // Return snapshot for rollback
    return { previousItem }
  },
  onError: (err, newItem, context) => {
    // Rollback with signal
    queryClient.setQueryData(['items', newItem.id], context.previousItem)
    toast.error("Update failed. Your changes have been reverted.")
  },
  onSettled: () => {
    queryClient.invalidateQueries({ queryKey: ['items'] })
  },
})
```

The `onError` callback is the reversibility moment. If it exists but doesn't surface anything to the user, reversibility has been implemented in the data layer and dropped in the UI layer.

**Loading → success → error lifecycle check:**

For every async operation in the target, verify all three states exist and are surfaced:
- **Loading:** button spinner, skeleton, or "Saving..." signal
- **Success:** toast, banner, inline confirmation, or state change
- **Error:** toast, banner, inline error, or recovery path

Missing any of these is a 🔴 BREAKS finding.

---

#### Principle #7 ... Preserve user state across boundaries

> Loading must not lose your scroll, your selection, your draft.

**Source:** Bruce Tognazzini, state preservation as first principle (Layer 2). Andy Matuschak, working memory as a design constraint (Layer 3). Loren Brichter, Tweetie's state continuity across navigation (Layer 3). Linear team, offline-first state model (Layer 3).

**The core idea:** Every navigation event, filter change, tab switch, or page load is a boundary. Each boundary is an opportunity to lose state that the user has accumulated. Working memory is limited; the interface should not ask users to hold things in their head that it could hold itself.

Andy Matuschak's framing is precise: if you navigate away from a half-filled form and back, the form should be exactly as you left it. The interface, not your memory, is the externalized store.

**Anti-patterns to catch:**

Scroll reset on navigation:
```tsx
// Anti-pattern: page scrolls to top on every navigation
// In Next.js App Router, this is the default behavior unless addressed

// Common pattern that loses scroll:
useEffect(() => {
  window.scrollTo(0, 0)
}, [pathname])
// This is sometimes appropriate; often it is not
```

Lost form state on refresh or navigation:
```tsx
// Anti-pattern: form state lives only in React state
const [title, setTitle] = useState("")
// Browser back → forward: title is gone
// Accidental refresh: title is gone
// Navigate away and back: title is gone
```

`react-hook-form` with no persistence layer for drafts. For any form where the user might spend significant time, consider `localStorage` persistence:
```tsx
// Pattern to suggest:
const form = useForm({
  defaultValues: () => {
    const saved = localStorage.getItem('form-draft')
    return saved ? JSON.parse(saved) : defaultValues
  }
})

// Auto-save on change:
useEffect(() => {
  const subscription = form.watch((values) => {
    localStorage.setItem('form-draft', JSON.stringify(values))
  })
  return () => subscription.unsubscribe()
}, [form.watch])
```

Selection lost on filter change:
```tsx
// Anti-pattern: selected items cleared when filter updates
const [selected, setSelected] = useState<Set<string>>(new Set())
const [filter, setFilter] = useState("")

const filteredItems = items.filter(item => item.name.includes(filter))

const handleFilterChange = (newFilter: string) => {
  setFilter(newFilter)
  setSelected(new Set()) // Anti-pattern: clears selection on filter change
}
```

Tab state not persisted to URL:
```tsx
// Anti-pattern: active tab lives only in component state
const [activeTab, setActiveTab] = useState("overview")
// Share the URL? The recipient sees a different tab.
// Refresh? Tab resets to default.

// Better: tab state in URL params
const searchParams = useSearchParams()
const activeTab = searchParams.get("tab") ?? "overview"
```

Modal state not preserved when reopened. If a wizard-style modal has multiple steps and the user accidentally closes it, they should be able to return to where they were.

**Linear's offline-first model** is the canonical exemplar for #7. Open Linear on a plane. Navigate through issues. Create a new issue. Close the app. Reopen it. Everything is exactly as you left it. The state boundary (network loss, app close) is invisible to the user. This is the aspirational end state; for most web apps, a partial implementation (URL state, localStorage drafts) captures most of the value.

**Loren Brichter's Tweetie** is the original exemplar: tab state, scroll position, and draft text were all preserved across background/foreground transitions. This was novel in 2008 and became the iOS standard. The principle has not changed.

**State preservation checklist:**

For each boundary in the target (navigation event, filter change, tab switch, modal close/reopen, browser refresh), verify:
- Scroll position: preserved or deliberately reset?
- Form draft: persisted or cleared?
- Active selection: preserved through filter changes?
- Active tab: URL-driven or component-state-only?
- Sort/filter state: URL-driven or reset on navigation?

Any "cleared" or "reset on navigation" that is not deliberate is a finding.

---

### Step 3 ... Write findings

```
[#N PRINCIPLE NAME] - <severity>
File: <path>:<line range>
Symptom:
  <one or two sentences describing the observed code>
Principle:
  <one sentence: the principle, what it requires here>
Tactic:
  <the specific change, with code>
Reference:
  <citation: lineage source + canonical exemplar>
```

**Severity scale:**

- **🔴 BREAKS** ... the principle violation causes real user harm: a submit button stuck in spinner state on failure; a form that loses all user input on navigation; a silent rollback where the user never knows their action failed; an optimistic update with no error path at all.
- **🟡 DRIFTS** ... the principle is not catastrophically violated but the surface is slipping: toast exists but the error variant is generic ("Something went wrong") rather than actionable; scroll position resets in a case where most users won't notice but some will; tab state is component-only rather than URL-driven.
- **🟢 OPPORTUNITY** ... the principle is met minimally but a more complete implementation would substantially improve trust: add `toast.promise()` where you have separate loading/success/error calls; persist filter state to URL; add auto-save draft to a long-form editor.

**Output format:**

```
═══════════════════════════════════════════════════
DEVOUR STATE REVIEW: <target>
Context: <principle weighting from Devour Context>
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
STATE SUMMARY
N breaks · N drifts · N opportunities
Principles reviewed: #4 (reversibility), #7 (state preservation)
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

After printing the review, **always print the APPLY? block as the final lines of output.** Do not skip it. State findings often involve the highest-value fixes in a devour pass; do not bury the apply prompt.

When applying:

- **Show the diff** before each file change. Brief, just the hunks.
- **Apply 🔴 BREAKS without further confirmation** if the user picked option 1, 2, or 3. State breaks (missing error paths, lost user state) almost always need the named fix.
- **Ask once per 🟡 DRIFT or 🟢 OPPORTUNITY** that involves a real taste call (e.g., "persisting filter state to URL ... is this navigable enough to want it deep-linkable?"). Skip the ask if the fix is mechanical.
- **After all fixes are applied, ask if the user wants to commit.** Do not auto-commit.

---

## Voice

Specific, calm, citation-heavy. The findings in this review are often the most important ones in a devour pass, because state failures cost users real work. Name the failure precisely. Cite the principle. Show the fix with code.

Do not say "this could be better." Do not say "you might want to add error handling." Say what is missing, which principle it violates, what the user experiences when it's missing, and how to fix it.

---

## See also

- [`../devour/SKILL.md`](../devour/SKILL.md) ... full-spine review
- [`../../references/principles-map.md`](../../references/principles-map.md) ... source citations for #4, #7
- [`../../references/exemplars.md`](../../references/exemplars.md) ... Sonner toast lifecycle, Linear offline-first, Tweetie state preservation
- [`../../references/anti-patterns.md`](../../references/anti-patterns.md) ... full anti-pattern catalog for state principles
- [`../../references/reading-list.md`](../../references/reading-list.md) ... Bret Victor's "Magic Ink," Andy Matuschak's writings
