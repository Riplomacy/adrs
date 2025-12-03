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

Format: `PREFIX-###-descriptive-title.md`

- **PREFIX**: Category-specific prefix
  - `DEV-###`: global/development decisions
  - `INFRA-###`: global/infrastructure decisions
  - `ADR-###` or service initials (e.g., `UDS-###`): service-specific decisions
- **###**: Sequential number within each category (001, 002, 003...)
- **descriptive-title**: Kebab-case description of the decision

Examples:

- `INFRA-001-c4-model-for-architecture-documentation.md`
- `INFRA-002-public-api-gateway-with-iam-restrictions.md`
- `DEV-001-rust-coding-standards.md`
- `UDS-001-dynamodb-table-design.md`

### Referencing ADRs

Use the full identifier for reference: "See INFRA-001" or "DEV-002"

## Template

Use `template.md` for creating new ADRs.
