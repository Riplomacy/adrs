# INFRA-005: Use DynamoDB with Provisioned Capacity

**Date:** 2025-12-03\
**Status:** Accepted\
**Deciders:** Jean-Sébastien Dominique

## Context

Database selection for the Riplomacy Bot project requires balancing cost, performance, and operational complexity. The application workload doesn't require complex relational queries.

## Decision

Use DynamoDB with provisioned capacity mode to stay within AWS free tier limits. Configure tables to share the free tier allocation of 25 total read capacity units and 25 total write capacity units across all tables and global secondary indexes, which accommodates current production workloads at zero cost.

## Consequences

### Positive

- Zero database costs by staying within free tier limits
- Familiar technology with existing DynamoDB experience
- Serverless operation with no infrastructure management
- Scales automatically within provisioned limits

### Negative

- Capacity planning required to stay within free tier limits across all tables
- Potential throttling if workload exceeds provisioned capacity
- Less flexible than on-demand pricing for variable workloads

## Alternatives Considered

- **RDS:** Rejected due to higher costs and unnecessary relational complexity
- **DynamoDB on-demand:** Rejected due to costs exceeding free tier provisioned capacity

## Implementation Notes

- Configure all DynamoDB tables with provisioned capacity mode
- Distribute 25 RCU/25 WCU free tier limits across all tables and GSIs
- Monitor capacity utilization to ensure workloads fit within total limits
- Design data access patterns to optimize capacity usage across all tables
