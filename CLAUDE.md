# AGENTS.md — ADRs Repo

This repo is the Architecture Decision Record store for the Riplomacy workspace (see `../AGENTS.md`), covering `riplomacy_bot` and its sibling services. See `README.md` for directory structure and the `STATUS-PREFIX-###-title.md` naming convention, and `template.md` for the ADR format itself.

## Key Behaviors

- **Check before architectural work.** Before making or implementing a non-trivial architectural decision in any Riplomacy repo, check whether an ADR here already covers the area (`global/infrastructure`, `global/development`, or the relevant `services/<name>` directory).
- **Write one after.** After landing an architectural decision (not an implementation detail), add an ADR here recording it — context, decision, consequences, alternatives considered.
- **Existing ADR content is immutable.** Once written, an ADR's Context/Decision/Consequences/Alternatives sections stay as originally written, even after the decision changes — it's a record of what was decided and why *at the time*. A changed decision means writing a **new** ADR that supersedes or augments the old one (reference it by category+number, e.g. "Supersedes INFRA-002"), never rewriting the old one's body to match the new reality.
- **The one exception:** the **Status line** of a superseded/rejected ADR may be updated to point at what replaced it (e.g. `Superseded by INFRA-008`), and the file renamed from its `A-`/`P-` prefix to `S-`/`R-` per the naming convention — that's lifecycle tracking, not a content rewrite.
- Commit messages here follow the same conventions as `riplomacy_bot`. This repo has its own git remote (`git@github.com:Riplomacy/adrs.git`) — pushing it is a separate action from pushing `riplomacy_bot`, confirm before doing so.
