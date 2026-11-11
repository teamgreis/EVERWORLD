# EVERWORLD — Administration, Accounts & Security

## 1. Account creation policy

Public registration is disabled. Every account is created by the owner or an authorized administrator in the admin dashboard. The owner can disable this policy only through a future explicit product decision.

Account creation supports a unique username, initial password or secure one-time setup flow, character display name, permitted role assignment, active/disabled status, creation timestamp and creator audit record.

Passwords must never be displayed again after creation. Prefer a one-time password setup link or forced password change over sending reusable passwords through chat.

## 2. Roles

| Role | Scope |
|---|---|
| OWNER | Full control; cannot be removed by ordinary admin actions |
| ADMIN | Account and game administration within delegated permissions |
| MODERATOR | Reports, chat moderation and player support |
| PLAYER | Normal gameplay |

Authorization is deny-by-default and checked on the server for every operation. Hiding a UI control is not authorization.

## 3. Owner capabilities

- Create, disable and reactivate accounts
- Assign/revoke delegated admin and moderator roles
- Initiate password reset
- Inspect account and character state for support
- Apply documented support corrections
- Grant or remove items/currency only through audited operations
- Review reports and moderation history
- Configure world events and maintenance mode
- View operational status and deployment version
- Export required operational data subject to privacy policy

## 4. Admin dashboard

The dashboard provides a searchable user list, account status and role, character summary and level, account creation and reset workflow, disable/reactivate controls, audit history, player report queue and safe support tools with confirmation dialogs.

High-impact actions require confirmation and a reason. The UI must make the target account unmistakable.

## 5. Audit logging

Audit events are append-only from the application perspective.

Record actor account, action name, target account/entity, timestamp, reason, relevant before/after summary where appropriate and request/correlation ID.

Never log password values, authentication tokens, session cookies or secrets.

Critical audit events include account creation/disable/reactivation, role changes, password reset initiation, currency/item grants or removal, character corrections, moderation actions, world configuration changes and maintenance mode changes.

## 6. Authentication

- Argon2id password hashing.
- Secure, HttpOnly, SameSite cookies.
- HTTPS only.
- CSRF protection for cookie-authenticated write operations.
- Session expiration and revocation.
- Login rate limiting and generic authentication errors.
- Password reset uses expiring, single-use tokens.
- Owner account recovery procedure is documented and tested.

## 7. Authorization and game integrity

The client is untrusted. The server validates item ownership and inventory capacity, currency balances and transaction counterparties, crafting inputs and outputs, quest requirements and rewards, movement speed/collision/zone membership, combat turns/damage/rewards, property access and object placement, and offline action completion/claim uniqueness.

Use database transactions for all operations involving multiple valuable records.

## 8. Player safety and moderation

Before broad public access:
- Mute and block controls
- Report player/message flow
- Moderator review queue
- Temporary chat restrictions
- Account suspension
- Rate limits and spam controls
- Safe-area PvP restrictions
- Anti-harassment controls for repeated attacks and unwanted interactions

## 9. Privacy and data lifecycle

Before inviting external players, publish a clear privacy notice and define data collected, purpose and retention, account deletion/deactivation behavior, chat retention, admin access rules, backup retention and a contact for privacy requests.

Because younger players may use the game, privacy, moderation and applicable child-protection requirements must be reviewed before public launch. Avoid collecting unnecessary personal data.

## 10. Operational security

- Keep database private on the Docker network.
- Bind the app host port to loopback only.
- Use YunoHost TLS.
- Store secrets in server environment files and GitHub Actions secrets.
- Restrict SSH deployment access to a dedicated deployment key/account.
- Back up PostgreSQL off-host.
- Test restoration, not just backup creation.
- Pin and review production dependencies.
- Apply security updates.
- Keep staging separate from production when available.
