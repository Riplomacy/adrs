# Architecture Decision Records (ADRs)

The decision log for the Riplomacy workspace: why the system has the shape it has. Each record captures one architecturally significant decision, its context, the options that lost, and its consequences. See ADR-0034 for why the log works this way.

## What qualifies

Record a decision when it affects at least one of:

- **Structure:** how the system is divided into components, services, stacks and stores
- **Quality attributes:** security, availability, fault tolerance, cost, performance
- **Dependencies:** what depends on what, including external systems
- **Interfaces:** APIs, published contracts, entry points
- **Construction techniques:** languages, frameworks, libraries, build and test tooling

A decision that would be hard to reverse almost always qualifies.

These are **not** ADRs:

- Features and their behavior
- Business rules and domain policies
- Single parameter values, limits and timeouts
- Bug fixes and their explanations: that's what commit messages are for
- Changelog entries
- Work-management process
- Incidents: those go in `lessons-learned/`

When unsure, ask: would someone changing this later need to know *why* it was done this way, and would getting it wrong reshape the system? If not, it belongs in a commit message.

## Writing a record

- Copy `template.md` to `decisions/NNNN-kebab-case-title.md`, using the next number in the sequence.
- **One decision per record.** A decision rolled out in phases gets one record per phase.
- **Lead with the decision.** Context and options follow.
- **Why, not how.** No code, configuration, file paths or step-by-step instructions. Link to supporting material instead of embedding it; the record must still stand on its own.
- **About one page.** Pithy, assertive, factual.
- **Don't hide consequences.** List the costs alongside the benefits.

## Numbering and references

- Records live in `decisions/`, numbered in one sequence across the whole workspace: `0001`, `0002`, ...
- Numbers are never reused.
- Refer to a record as `ADR-0001`.
- Status lives in the file, never in the filename, so files are never renamed.

## Lifecycle

| Status | Meaning |
|---|---|
| Proposed | Under discussion. The content may still change. |
| Accepted | Decided. The content is frozen. |
| Rejected | Decided against. The content is frozen and states why, so the question isn't reopened. |
| Superseded | Replaced by a later record, named on the Status line. |
| Deprecated | No longer in effect, with no replacement. The reason is given on the Status line. |

- A record is committed to `main` as Proposed.
- It becomes Accepted or Rejected only on the decider's explicit approval. On approval, the Status line and Date are updated, and the content is frozen.
- To change a decision, write a new record that supersedes the old one. On the old record, only the Status line changes, to point at its replacement.
- Records are never deleted.

## Using the log

- Check for an existing record before architectural work, and write one when a qualifying decision is made.
- In code review, a change that contradicts an accepted record is either reworked, or a superseding record is written first.
- Each workspace repository's README links here.
- Past decisions that were never recorded may be added retroactively, dated when they were made.

## Converted records

ADR-0001 to ADR-0033 were written under an earlier convention (`PREFIX-###` identifiers such as `INFRA-008`) and converted to this format on 2026-10-14. Each one names its old identifier on a `Converted from` line. They predate the Confidence section, so they don't have one. Records that didn't meet the "What qualifies" test were marked Deprecated. Proposed records that didn't qualify were dropped; they remain in git history.

## Lessons Learned

Unlike ADRs (which record a decision and its rationale, and whose content stays immutable
once written), entries in `lessons-learned/` record what actually happened during a real
incident and why — no Decision/Alternatives framing, no immutability requirement if new
facts come to light.

### Naming Convention

Format: `LESSON-###-descriptive-title.md`, sequential across the whole directory (no
per-category numbering, no status prefix — a lesson doesn't get superseded, it just is).

### Template

Use `lessons-learned/template.md`: What Happened / Root Cause / Resolution / Takeaway.
The Takeaway section is the one worth linking to from elsewhere — it's the actionable
rule, not the incident narrative.

### Referencing

Use the lesson number: "See LESSON-001".
