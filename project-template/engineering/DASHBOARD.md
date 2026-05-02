# Engineering DASHBOARD

## Purpose
- Quick entry-point for engineers and SREs to find the most-used technical docs and action items.

## Quick links (use as a living index)
- [TRD (trd.md)](trd.md) — architecture & system overview
    - [TRD-web](trd-web.md) / [TRD-iot](trd-iot.md) — platform-specific examples
- [RFCs summary (rfc.md)](rfc.md)
- [API Contract (designs/api-contract.md)](designs/api-contract.md) & [ERD (designs/entity-relationship-diagram.md)](designs/entity-relationship-diagram.md)
- [Bugs (bugs.md)](bugs.md)
- [Testing result dump (testing/)](testing/)
- [C4 diagrams (designs/c4-diagram.md)](designs/c4-diagram.md)

## Current action items (example checklist)
- [ ] Review RFCs in "Proposed" status
- [ ] Sync ERD with DB schema (owner: DB lead)
- [ ] Add e2e test results for sprint X

## Useful commands & runbooks
- Run local service: README in each repo → npm run dev / uvicorn
- Health check endpoints: /api/v1/health
- Where logs go: structured JSON to stdout; DataDog index: project-logs

## Who updates this
- Tech Lead / On-call SRE should update status items.
- Owners must link PR/issue when marking a checklist item done.

## How to maintain
- Keep checklist short (3–6 items). Move old items to archive with date.
- Update the "Current action items" weekly during sprint planning.
