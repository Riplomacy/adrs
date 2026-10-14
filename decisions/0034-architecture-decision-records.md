# 0034: Architecture Decision Records for Architecturally Significant Decisions Only

**Status:** Accepted 2026-10-14\
**Date:** 2026-10-14\
**Deciders:** Jean-Sébastien Dominique\
**Scope:** Workspace\
**Supersedes:** ADR-0001

## Decision

We will record only architecturally significant decisions, one decision per record, as short records in a single numbered sequence that are immutable once accepted. The rules live in this repository's `README.md` and `template.md`.

## Context

The decision log has drifted away from architecture. Feature behavior, business rules, single parameter values, bug write-ups and process choices were all recorded as ADRs. The log had no definition of what qualifies, so anything that felt like a decision ended up in it.

The template invited the same drift: its Implementation Notes section kept collecting code, configuration and runbook detail.

Status was encoded in the filename, so every status change meant a rename that broke links. Numbering was per category, so references needed the prefix to be unambiguous.

The workspace has a single maintainer. Pull request review doesn't fit, because GitHub doesn't let an author approve their own pull request.

## Options Considered

- **Keep the ADR-0001 conventions and add a significance test:** fixes the scope drift, but keeps the renames and a template that invites detail.
- **Pull request per decision, merge to accept:** a good review loop for teams, but a solo maintainer can't approve their own pull request.
- **Move the records next to the code in `riplomacy_bot`:** keeps decisions close to what they govern, but they span several repositories, and moving them is churn without new capability.
- **A heavier template such as full MADR:** more structure than one-page records need.

## Consequences

- Fewer records, each worth reading. The log becomes a reliable map of why the system has its current shape.
- Deciding significance is a judgment call; the README's examples and exclusions guide it.
- Existing records are converted to the new format and numbering, keeping their meaning. That is a one-time exception to immutability; references to the old identifiers across the workspace have to be updated.
- Accepted records that don't meet the bar are retired by marking them Deprecated, not by deleting them. Proposed records that don't meet it are dropped, since nothing was ever decided.
- A record's status is no longer visible from its filename; it has to be read from the file.
- Past structural decisions that were never recorded can be added retroactively.

## Confidence

High. Revisit if a second maintainer joins, which would make pull request review viable, or if nearly every decision ends up concerning a single repository.

## References

- Michael Nygard, [Documenting Architecture Decisions](https://www.cognitect.com/blog/2011/11/15/documenting-architecture-decisions)
- Martin Fowler, [Architecture Decision Record](https://martinfowler.com/bliki/ArchitectureDecisionRecord.html)
- [adr.github.io](https://adr.github.io/)
- Microsoft, [Maintain an architecture decision record](https://learn.microsoft.com/en-us/azure/well-architected/architect-role/architecture-decision-record)
- AWS, [ADR process](https://docs.aws.amazon.com/prescriptive-guidance/latest/architectural-decision-records/adr-process.html)
- Red Hat, [Why you should be using architecture decision records](https://www.redhat.com/en/blog/architecture-decision-records)
