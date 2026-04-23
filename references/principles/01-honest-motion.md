# Principle 1 ... Honest motion

> Good design is honest.
> ... Dieter Rams, principle 6

## What it means

If you animate something, the animation must communicate information the static state could not. Motion that performs quality without doing informational work is decoration that moves, which is worse than decoration that doesn't. Remove any animation where its absence costs the user no understanding.

## Why it matters

Rams's principle 6 establishes that design cannot misrepresent its purpose or capabilities. Don Norman's feedback model extends this to interfaces: the system must communicate what happened, and motion is one of the strongest signals it can send. When motion is used decoratively, it occupies the same channel as meaningful feedback and degrades that channel. Emil Kowalski's essay "You Don't Need Animations" makes the modern case explicitly: each animation is a claim, and the claim must be true.

## Tactics

- For each animation in a component, state what the user learns from it. If the answer is empty, remove the animation.
- Reserve entrance animations for content that loads asynchronously or appears in response to a user action. Static content has nothing to announce.
- Use motion to signal state transitions: loading to success, selected to deselected, empty to populated. These have information the static frame cannot show.
- Apply Kowalski's test before shipping: "Does the user need this animation to understand what just happened?" If no, cut.

## Anti-patterns

- **Decorative page-load fade:** Page content fades in on every load regardless of whether the user navigated. The motion communicates nothing; the content was always going to be there.
- **Pulse on a static badge:** A notification dot that pulses when there are no new notifications. The signal says "something is happening." The data says nothing is. The animation is a lie.
- **Animated counter from zero:** A number that counts up from zero to its real value on render. The number was fetched; it is known. The animation performs computation it is not doing.

## Exemplars

- **Sonner toast lifecycle:** Loading, success, and error each have distinct motion. Remove any one of them and you lose the state signal, not just polish. The animation is the feedback.
- **NotBoring Habits:** The completion animation communicates "you did the thing" without performing celebration as a proxy for quality. Motion appetite is high; honesty is intact.

## Further study

- Emil Kowalski, "You Don't Need Animations" (emilkowal.ski) ... the primary case for motion as a citable claim
- Dieter Rams, principle 6 in *Less and More: The Design Ethos of Dieter Rams* (Klemp & Ueki-Polet, 2009)
