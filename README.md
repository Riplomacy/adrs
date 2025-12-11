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
