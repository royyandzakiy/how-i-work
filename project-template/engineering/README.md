# Engineering HOW-TO

## What this folder is for
- Central place for technical artifacts: TRDs, RFCs, designs, testing notes, bug lists, API contracts, diagrams, and ADRs.

## Structure (what you'll find)
- rfcs.md — long-form list and templates for RFCs
- trd*.md — Technical requirement docs (general, web, IoT)
- bugs.md — canonical project bug/triage log (not a substitute for issue tracker; summary & links)
- testing/ — test plans, E2E notes, test results
- designs/ — architecture diagrams, C4, ERD, API contract

## How & when to fill these
- Project start: create/update TRD and Architecture section; link PRD.
- Before large changes: write an RFC; discuss and mark status in rfcs.md.
- When a defect is discovered: add a concise entry to bugs.md with reproduction, severity, owner, and issue link.
- After tests/runbooks: commit artifacts to testing/ with date and author.

## How to use (for engs/prod)
- Developers: read TRD before implementing features; reference APIs and ERDs when altering schemas.
- Reviewers/Tech leads: require a linked RFC for design changes >1 sprint worth of work.
- Operations: use testing and monitoring notes to understand runbooks and health checks.

## Maintenance
- Add version and last-updated header in each doc.
- Link any external internal-wiki or repo references.
- Keep rfcs.md status (Proposed, Accepted, Rejected) and reference ADRs.
- Clean old drafts under designs/archive annually; keep ERD/C4 up-to-date when DB or major flows change.

## Templates & quick rules
- RFCs: Problem, Proposal, Alternatives, Rollout Plan, Rollback Plan, Owners.
- TRD sections: Architecture, Tech Stack, Repos, Data Flow, NFRs, Monitoring, Deployment.
- Bugs: Title | Severity | Repro steps | Owner | Issue link | Status

## Contact
- Add the Tech Lead or Engineering Manager as default owner for missing fields.
