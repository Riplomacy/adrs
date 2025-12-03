# Architecture Decision Records (ADRs)

## Organization

### Structure

- **global/**: Decisions that apply across all Riplomacy services
  - **infrastructure/**: AWS, networking, deployment decisions
  - **development/**: Coding practices, tooling, testing approaches
- **services/**: Service-specific architectural decisions
  - Individual directories for each service (e.g., `user-data-service/`)
  - Services can create subdirectories as needed

### Naming Convention

Format: `YYMM###-descriptive-title.md`

- **YYMM**: Year and month (e.g., `2512` for December 2025)
- **###**: Sequential number across all ADRs within that month (001, 002, 003...)
- **descriptive-title**: Kebab-case description of the decision

Examples:

- `2512001-c4-model-for-architecture-documentation.md`
- `2512002-public-api-gateway-with-iam-restrictions.md`

### Referencing ADRs

Use the numeric identifier for quick reference: "See ADR 2512001" or just "2512001"

## Template

Use `template.md` for creating new ADRs.
