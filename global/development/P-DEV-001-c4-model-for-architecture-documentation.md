# DEV-001: Use C4 Model for Architecture Documentation

**Date:** 2025-12-03\
**Status:** Proposed\
**Deciders:** Jean-Sébastien Dominique

## Context

The Riplomacy Bot project needs consistent architecture documentation as it grows in complexity. Multiple microservices, AWS integrations, and data flows require clear visualization for development and maintenance. Current documentation lacks standardized diagramming approach.

## Decision

Adopt the C4 Model (Context, Containers, Components, Code) for all architecture documentation, focusing on the first three levels. Use Structurizr DSL for diagram-as-code implementation.

## Consequences

### Positive

- Standardized architecture visualization across all services
- Clear separation of concerns at different abstraction levels
- Version-controlled diagrams alongside code
- Improved onboarding for new developers

### Negative

- Learning curve for C4 Model and Structurizr DSL
- Additional maintenance overhead for keeping diagrams current
- Tool dependency for diagram generation

### Neutral

- Migration effort from existing documentation formats
- Need to establish modeling conventions and guidelines

## Alternatives Considered

- **Traditional UML:** Rejected due to complexity and poor developer adoption
- **Informal diagrams:** Rejected due to inconsistency and maintenance issues
- **AWS Architecture Icons only:** Rejected due to lack of business context

## Implementation Notes

- Create modeling guidelines document with naming conventions
- Use AWS service-specific technology tags at component level
- Focus on business domains at system level
- Organize containers by deployment and runtime patterns
- Define relationships with specific action verbs and data flow descriptions
