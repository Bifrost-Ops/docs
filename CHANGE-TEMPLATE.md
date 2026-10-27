# Change request template

## Purpose
Describe the intended outcome and the reason for the change.

## Target and ownership
- Environment:
- Repository / subsystem:
- Target APIC or cluster (non-secret identifier):
- Current deployed commit:
- Proposed commit:

## Prerequisites and dependencies
Document versions, backups, state lock, management access and related services.

## Impact and risk
Describe affected endpoints, routing, policies, workloads and possible outages.

## Plan review
- [ ] Syntax/schema validation passed
- [ ] Terraform plan or GitOps diff attached (sanitized)
- [ ] All deletes/replacements individually reviewed
- [ ] Secret exposure checked
- [ ] Change approved by responsible operator

## Deployment steps
Number each action and include the expected verification result immediately afterward.

## Rollback trigger and steps
Specify a measurable stop condition, the known-good commit, new plan approval and any data restore dependencies.

## Verification and evidence
Record pre/post health checks, observed results, timestamps and deviations.

## Rebuild impact
State whether the change modifies clean-room bootstrap, backup, restore or teardown instructions.
