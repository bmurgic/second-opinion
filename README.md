# second-opinion

A Claude Code plugin with one subagent, `second-opinion`. It reviews a teammate's design or spec on the teammate's
git branch, writes the review as a separate file beside it, pushes it, and approves the design when it is ready.

It never edits the design, never force-pushes, and never posts to chat. It runs no tests, builds, or application
code. It only reads the code.

## Install

```sh
claude plugin marketplace add bmurgic/second-opinion
claude plugin install second-opinion@second-opinion
```

On Claude Code 2.1.275 or later, one command does both:

```text
/plugin install second-opinion --marketplace bmurgic/second-opinion
```

To use the agent without the plugin, copy [`agents/second-opinion.md`](agents/second-opinion.md) into
`~/.claude/agents/` or a repository's `.claude/agents/`. The file reads nothing else.

The agent runs on Fable at high effort. The `fable` alias means Fable 5.2 on Claude Code 2.1.321 and later, and
Fable 5.1 before that. Without Fable access, change the `model:` line in the agent file.

## Run a review

Ask Claude for a second opinion, and give the branch and the design path:

```text
Get a second opinion on the design.
Branch: feat/driver-documents
Design: docs/specs/driver-documents.md
```

| Input | Required | Default |
|---|---|---|
| `Branch:` | Yes | None. The teammate's branch that holds the design. |
| `Design:` | Yes | None. The design file's path on that branch. |
| `Target:` | No | The branch the repository's instructions name for pull requests, else the remote's default branch. |
| `Reviewer:` | No | `git config user.name`. The name on the approval line. |
| `Worktree:` | No | `../<repo>-review-<branch>` beside the repository. |

The agent pushes to the teammate's branch, so Claude dispatches it only when you ask. Run it from a clone of the
repository that has push access to that branch.

If `Branch:` or `Design:` is missing, or the design isn't on the branch, the agent stops and returns a message that
asks the teammate for it. It doesn't guess.

## What a round does

1. **Checks the branch.** It fetches the branch and confirms the design is on it. If the branch is behind the
   target, it lists the rule files that changed since, and the message tells the teammate to pull first.
2. **Opens a worktree.** It checks the branch out in a separate git worktree, so your checkout stays where it is.
   A later round reuses that worktree.
3. **Checks the rules.** It reads `CLAUDE.md`, `AGENTS.md`, `.claude/rules/`, ADRs, and any glossary, then checks
   the design against each rule.
4. **Deepens the design.** It picks the one to three decisions that are hardest to undo. For each one, it:
   - searches for existing code that could hold it, and lists reuse as an option.
   - names the outcome one level above the design's goal, and the simplest outcome that would make the decision
     unnecessary.
   - says what the best expert would do differently, and which failure the design invites, such as duplication or
     two sources of truth.
   - sets a target ten times harder, such as "the flag is never wrong" instead of "the flag clears".
   - borrows checks from two or three fields outside software that do the same job.
   - names what would settle each option outside its own reasoning, such as a query against real data or a test.

   It checks every claim against the code and cites `file:line`.
5. **Sorts the open points.** The teammate's design choices, such as which table or library to use, become
   neutral questions: options and facts, no recommendation. Questions about what the team allows, such as which
   login may touch the shared database, get a ruling from the agent, in your voice. A repository rule decides the
   ruling when one applies. Otherwise the agent picks the option that is easier to undo and safer for people,
   money, and shared data.
6. **Writes the review.** It writes `<design name>-review.md` beside the design. The first line names the design
   commit it reviewed.
   - **Part 1: Changes to make.** First the changes the rules require, each citing its rule. Then your rulings,
     each with its reason. Then the changes the agent recommends, each naming the question that would change it.
   - **Part 2: Questions.** Numbered and grouped, with no recommendations.
7. **Approves, if ready.** It approves only when every required change is done, no question is open, and the
   design breaks no rule. A first review rarely qualifies. Approval appends one line:
   `**Approved <date> (<Reviewer>, via second-opinion):** <what is approved>, at design commit <sha>.`
8. **Commits and pushes.** It commits only the review file, with `git add -f`, because doc folders are often
   git-ignored and a plain `git add` skips them silently. If the teammate pushed in the meantime, it rebases,
   reviews anything that changed in the design or the rules, and pushes again.

## Follow-up rounds

Run it again after the teammate updates the design. The agent finds the existing review and diffs the design
against the commit it last reviewed. It marks each earlier change and question as done, done differently, or still
open, and reviews the new material. Then it appends the next part, such as Part 3, to the same file. One review file
per design.

## What you get back

1. The branch and the pushed commit, or why nothing was pushed.
2. Whether it approved the design, and if not, what is missing.
3. Each ruling it made, with its reason and the rule it cites.
4. The questions waiting on the teammate.
5. A short chat message for you to send. It names the branch, the review file, and the part to read. On approval,
   it tells the teammate to start the feature branch from the docs branch, so the design and its review merge with
   the feature's pull request.

## License

MIT. See [LICENSE](LICENSE).
