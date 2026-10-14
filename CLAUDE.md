# CLAUDE.md — ADRs Repo

This repo is the Architecture Decision Record store for the Riplomacy workspace, covering `riplomacy_bot` and its sibling repositories. `README.md` holds the rules (what qualifies, numbering, lifecycle), `template.md` the format, and ADR-0034 the reasoning behind both.

## Key Behaviors

- **Only architecturally significant decisions.** Apply the README's "What qualifies" test before proposing a record. Features, business rules, parameter values, bug fixes and process choices are not ADRs. When in doubt, ask Signal rather than writing one.
- **Check before architectural work.** Before making or implementing a structural decision in any Riplomacy repo, check `decisions/` for a record that already covers it. Old identifiers like `INFRA-008` appear on each converted record's `Converted from` line.
- **Draft as Proposed, never self-accept.** Each new record goes on its own branch with `Status: Proposed`, in a pull request. Only Signal's explicit approval makes a record Accepted or Rejected; update the Status line on the branch, then merge. Rejected records are merged too.
- **Accepted and Rejected records are immutable.** A changed decision gets a new record that supersedes the old one. On the old record, only the Status line changes. Files are never renamed or deleted.
- **Why, not how.** No code, configuration, file paths or step-by-step instructions in a record.
- Commit messages follow the same conventions as `riplomacy_bot`. This repo has its own git remote (`git@github.com:Riplomacy/adrs.git`): pushing it is a separate action from pushing `riplomacy_bot`, so confirm before doing so.
