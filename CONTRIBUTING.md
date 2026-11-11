# Contributing to EVERWORLD

## Project principles

- Browser-only player experience.
- TypeScript for client and server code.
- Server-authoritative game state.
- PostgreSQL is the persistent source of truth.
- Keep features modular and testable.
- Never trust client-supplied currency, item ownership, damage, XP or timestamps.
- Public registration remains disabled.
- Do not commit secrets, credentials, private player data or production database dumps.

## Branching and releases

- main is the deployable branch.
- Feature work should use short-lived feature branches.
- Pull requests should include a summary, tests and documentation updates.
- Production images should be tagged with the commit SHA and release version.
- Breaking database changes require a migration plan and rollback/recovery notes.

## Definition of done

A feature is complete only when:
1. It works in the browser.
2. Server-side validation and authorization exist.
3. Persistent state is handled correctly.
4. Critical logic has automated tests.
5. Failure and reconnect behavior are considered.
6. Documentation is updated.
7. It has been verified in the target deployment environment where practical.

## Security

Report security issues privately to the repository owner rather than publishing exploit details in an issue. Never include tokens or personal data in logs, screenshots or test fixtures.
