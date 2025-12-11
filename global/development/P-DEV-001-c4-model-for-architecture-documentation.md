# DEV-001: Use C4 Model with D2 for Architecture Documentation

**Date:** 2025-12-03\
**Status:** Proposed\
**Deciders:** Jean-Sébastien Dominique

## Context

The Riplomacy Bot project needs consistent architecture documentation as it grows in complexity. Multiple microservices, AWS integrations, and data flows require clear visualization for development and maintenance. Current documentation lacks standardized diagramming approach.

Initial evaluation considered Structurizr DSL for C4 implementation, but encountered significant limitations including rigid definition ordering requirements that constrain natural model development and iteration.

## Decision

Adopt the C4 Model (Context, Containers, Components, Code) for all architecture documentation, focusing on the first three levels. Use D2 with layered architecture for diagram-as-code implementation.

## Consequences

### Positive

- Standardized architecture visualization across all services
- Clear separation of concerns at different abstraction levels
- Version-controlled diagrams alongside code
- Improved onboarding for new developers
- Flexible model development without rigid definition ordering constraints
- Multiple focused views from single model using layer-based organization
- Natural iterative modeling process that supports refactoring

### Negative

- Learning curve for C4 Model and D2 syntax
- Additional maintenance overhead for keeping diagrams current
- Manual view management without automatic view generation
- Tool dependency for diagram generation

### Neutral

- Migration effort from existing documentation formats
- Need to establish modeling conventions and guidelines
- Layer organization approach replaces predefined view types

## Alternatives Considered

- **Structurizr DSL:** Rejected due to rigid definition ordering requirements that constrain model development workflow
- **Traditional UML:** Rejected due to complexity and poor developer adoption
- **Informal diagrams:** Rejected due to inconsistency and maintenance issues
- **AWS Architecture Icons only:** Rejected due to lack of business context

## Implementation Notes

- Create modeling guidelines document with naming conventions
- Use AWS service-specific technology tags at component level
- Focus on business domains at system level
- Organize containers by deployment and runtime patterns
- Define relationships with specific action verbs and data flow descriptions
