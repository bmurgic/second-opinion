# second-opinion

A Claude Code plugin with one subagent, `second-opinion`. It reviews a teammate's design or spec on the teammate's
git branch. It writes the review as a separate file beside the design, pushes it to that branch, and approves the
design when it is ready. It never edits the design and never posts to chat. It returns a message draft for you to
send.

## Install

```sh
claude plugin marketplace add bmurgic/second-opinion
claude plugin install second-opinion@second-opinion
```

On Claude Code 2.1.275 or later, one command does both:

```text
/plugin install second-opinion --marketplace bmurgic/second-opinion
```

To use the agent without the plugin, copy [`agents/second-opinion.md`](agents/second-opinion.md) into `~/.claude/agents/` or into a
repository's `.claude/agents/`. The file reads nothing else.

The agent runs on Fable at high effort. The `fable` alias means Fable 5.2 on Claude Code 2.1.321 and later, and
Fable 5.1 before that. If your account has no Fable access, change the `model:` line in the agent file.

## Run a review

Ask Claude for a second opinion and name the branch and the design file:

```text
Get a second opinion on the design.
Branch: feat/driver-documents
Design: docs/specs/driver-documents.md
```

Three inputs are optional:

- `Target:` the branch that pull requests merge into. Without it, the agent reads the repository's instructions,
  then falls back to the remote's default branch.
- `Reviewer:` the name on the approval line. Without it, the agent uses `git config user.name`.
- `Worktree:` where the agent checks out the teammate's branch. Without it, the agent creates a folder beside the
  repository.

The agent pushes to the teammate's branch, so Claude dispatches it only when you ask for it.

## What one round does

1. Fetches the branch and checks that the design is on it. If the branch is behind the target branch, the agent
   lists the rule files that changed since.
2. Checks the design out in a separate git worktree, so your own checkout stays put.
3. Checks the design against the repository's rules, such as `CLAUDE.md`, `AGENTS.md`, and `.claude/rules/`.
4. Reworks the one to three hardest-to-undo decisions. It searches for existing code that could be reused, sets a
   target ten times harder, and borrows checks from fields outside software.
5. Writes `<design name>-review.md` beside the design. Part 1 lists the changes to make. Part 2 lists neutral
   questions that only the teammate can answer.
6. Rules on the points that need the reviewer's position. It cites a repository rule when one decides the point.
   When none does, it picks the option that is easier to undo.
7. Commits the review with `git add -f`, because doc folders are often git-ignored, and pushes it. It never
   force-pushes.
8. On a later round, it marks each earlier point done or open and appends the next part to the same file.

It approves the design only when every required change is done, no question is open, and the design breaks no rule.
