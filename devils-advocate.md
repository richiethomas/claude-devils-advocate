---
description: Devil's advocate code review of current branch diff against master
argument-hint: [max-rounds (default 5)]
---

# Devil's Advocate Code Review

Simulate a code review conversation between two engineers:

- **Author**: The engineer who wrote the code. Defends decisions, explains
  tradeoffs, and proposes concrete fixes when conceding a point.
- **Reviewer**: A senior engineer doing a thorough review. Raises real,
  substantive concerns and provides specific code suggestions when pointing
  out problems.

## Scope

Review ONLY the diff between the current branch and `master`:

```
!`git diff master...HEAD`
```

If the diff is empty, say so and stop.

## Topic priority

Work through concerns in this order. Spend as many rounds as needed on each
topic before moving to the next. Skip topics where there's nothing to say.

1. **Correctness** - bugs, logic errors, unhandled edge cases, race conditions
2. **Error handling** - missing error paths, swallowed exceptions, unclear
   failure modes
3. **Performance** - N+1 queries, unnecessary allocations, algorithmic issues
4. **Security** - injection risks, auth gaps, data exposure
5. **Maintainability** - unclear abstractions, coupling, naming, readability
6. **Testing gaps** - missing test cases, untested branches, brittle assertions

Move on when a topic is genuinely resolved. Do not pad rounds with
manufactured concerns to stay on a topic longer.

Reserve at least 1 round for lower-priority topics if possible. If high-priority
topics consume almost all rounds, briefly flag any remaining lower-priority
concerns in the Summary rather than silently skipping them.

## Rules

- Run up to $ARGUMENTS rounds (default 5 if not specified).
- Label each round: "### Round N — [Topic]"
- The Reviewer MUST raise at least one substantive concern per round.
  Do not nitpick style when real issues remain.
- The Author MUST push back on at least one point per round rather than
  immediately conceding everything. Defend the decision or explain the
  tradeoff before agreeing to change anything.
- When either party suggests a change, include a concrete code snippet or
  diff showing the proposed fix, not just a description of what to change.
- **Early termination**: If all topics are resolved before reaching N rounds,
  stop and print "✅ Consensus reached after N rounds."

## Summary

After all rounds (or early termination), print:

- **Round breakdown**: table showing each topic, rounds used, and action items
  generated, e.g.:
  | Topic            | Rounds | Action Items |
  |------------------|--------|--------------|
  | Correctness      | 2      | 3            |
  | Error handling   | 1      | 2            |
  | Performance      | 0      | 0            |
  | Security         | 0      | 0            |
  | Maintainability  | 1      | 1            |
  | Testing gaps     | 1      | 2            |
  | **Total**        | **5**  | **8**        |

- **Agreed changes**: concrete list with code snippets
- **Open disagreements**: anything the Author pushed back on that wasn't
  resolved
- **Action items**: priority-ranked list the Author can work through.
  Print the total count at the top, e.g. "### Action Items (7)"
- **Deferred concerns**: any lower-priority topics that weren't reached
  during the rounds, briefly noted so nothing is silently dropped
- **Re-run recommendation**: Based on how far down the topic priority list
  the review reached, the number of action items, and whether any topics
  were deferred. If the review made it through all topics including
  lower-priority ones like testing gaps, a re-run is likely unnecessary.
  If most rounds were consumed by high-priority topics (correctness, error
  handling) and lower-priority topics were deferred or squeezed, recommend
  re-running after fixes.
  E.g. "Most rounds spent on correctness and error handling, maintainability
  and testing gaps deferred — recommend re-running after fixes."
  Or "All topics covered through testing gaps with only 2 minor items —
  likely clean, no re-run needed."
