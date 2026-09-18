# Architecture Decision Records (ADRs)

## Organization

### Structure

- **global/**: Decisions that apply across all Riplomacy services
  - **infrastructure/**: AWS, networking, deployment decisions
  - **development/**: Coding practices, tooling, testing approaches
- **services/**: Service-specific architectural decisions
  - Individual directories for each service (e.g., `user-data-service/`)
  - Services can create subdirectories as needed
- **lessons-learned/**: Postmortem-style records of real incidents (outages, near-misses) —
  what happened and why, distinct from ADRs' forward-looking decision records. See
  "Lessons Learned" below for its own naming convention and template.

### Naming Convention

Format: `STATUS-PREFIX-###-descriptive-title.md`

- **STATUS**: Status indicator
  - `A-`: Accepted
  - `P-`: Proposed
  - `S-`: Superseded
  - `R-`: Rejected
- **PREFIX**: Category-specific prefix
  - `META-###`: ADR process and organizational decisions
  - `DEV-###`: global/development decisions
  - `INFRA-###`: global/infrastructure decisions
  - Service initials (e.g., `UDS-###`): service-specific decisions
- **###**: Sequential number within each category (001, 002, 003...)
- **descriptive-title**: Kebab-case description of the decision

Examples:

- `A-INFRA-001-c4-model-for-architecture-documentation.md`
- `P-INFRA-002-public-api-gateway-with-iam-restrictions.md`
- `A-DEV-001-rust-coding-standards.md`
- `S-UDS-001-old-dynamodb-table-design.md`

### Referencing ADRs

Use the category and number for reference: "See INFRA-001" or "DEV-002"

## Template

Use `template.md` for creating new ADRs.

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
