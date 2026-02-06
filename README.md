# devils-advocate

A [Claude Code slash command](https://docs.anthropic.com/en/docs/claude-code/slash-commands) that simulates a multi-round code review between an Author and Reviewer before opening a PR. Reviews the diff against `master`, works through topics by priority (correctness → error handling → performance → security → maintainability → testing gaps), and produces a summary with concrete action items.

## Why

Getting useful self-review out of an LLM takes a specific kind of prompting. Without structure, you tend to get shallow style nitpicks, premature agreement, and abstract suggestions like "consider adding error handling" with no actual code. This command addresses those problems:

- **Prioritized topics.** Correctness and error handling get reviewed before naming and style, so the important stuff doesn't get crowded out by easy wins.
- **Forced disagreement.** The Reviewer must raise a substantive concern every round. The Author must push back before conceding. This counteracts the LLM's tendency to converge too quickly.
- **Concrete output.** Both parties must include code snippets or diffs when suggesting changes, not just descriptions.
- **Early termination.** If consensus is reached before the round limit, the review stops rather than manufacturing filler.
- **Summary table.** Shows rounds and action items per topic at a glance, so you can tell whether another pass is warranted.

## Install

Copy `devils-advocate.md` to one of these locations:

```bash
# Global (available in all projects)
cp devils-advocate.md ~/.claude/commands/devils-advocate.md

# Project-specific
cp devils-advocate.md .claude/commands/devils-advocate.md
```

## Usage

```
/devils-advocate 6
```

The argument sets the maximum number of rounds (defaults to 5 if omitted). The command automatically diffs your current branch against `master`.

## What you get

Each round is labeled with the round number and topic (e.g. "Round 3 -- Performance") and contains a back-and-forth between the Author and Reviewer with concrete code suggestions.

After all rounds, a summary prints:

- **Round breakdown** -- table of topics, rounds spent, and action items per topic
- **Agreed changes** -- concrete list with code snippets
- **Open disagreements** -- anything unresolved
- **Action items** -- priority-ranked, with a total count
- **Deferred concerns** -- lower-priority topics that weren't reached
- **Re-run recommendation** -- whether to run again after addressing the action items, based on how far down the topic list the review got

## Design decisions

**Why prioritized topics instead of freeform review?** Without priority ordering, LLM reviewers front-load easy observations (naming, formatting) and miss deeper issues. Forcing correctness-first ensures bugs surface before bikeshedding.

**Why force disagreement?** Research on multi-agent LLM debate ([Du et al., 2023](https://arxiv.org/abs/2305.14325); [Kim et al., 2024](https://aclanthology.org/2024.findings-acl.112/)) shows that adversarial structure improves output quality, but a single model playing both roles tends to converge quickly. The forced pushback rules slow that convergence down.

**Why early termination?** If the diff is small or clean, forcing 5+ rounds of debate produces manufactured concerns. Early termination with an explicit consensus message avoids that.

**Why a re-run recommendation?** If most rounds were spent on correctness and error handling, lower-priority topics may have been skipped entirely. The recommendation tells you whether there's likely undiscovered stuff worth a second pass after fixing the current action items.

## Customization

Some things you might want to change:

- **Target branch.** The command diffs against `master`. If your repo uses `main`, edit the `git diff` line in the command file.
- **Topic list.** Add, remove, or reorder topics to match what matters for your codebase. For example, you might add "Accessibility" or "API contract" for a frontend or API-heavy project.
- **Round default.** 5 rounds works well for medium PRs. For large diffs you might want 8-10; for small changes, 3 is usually enough.

## Related reading

- [Improving Factuality and Reasoning in Language Models through Multiagent Debate](https://arxiv.org/abs/2305.14325) (Du et al., ICML 2024)
- [DEBATE: Devil's Advocate-Based Assessment and Text Evaluation](https://aclanthology.org/2024.findings-acl.112/) (Kim et al., ACL Findings 2024)
- [Enhancing AI-Assisted Group Decision Making through LLM-Powered Devil's Advocate](https://dl.acm.org/doi/fullHtml/10.1145/3640543.3645199) (Chiang et al., IUI 2024)
- [Claude Code slash commands documentation](https://docs.anthropic.com/en/docs/claude-code/slash-commands)

## License

MIT
