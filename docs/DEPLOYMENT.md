# EVERWORLD — Deployment & Automatic Updates

## 1. Runtime target

- Host: YunoHost on Raspberry Pi 5, 8 GB RAM
- Public entry: dedicated HTTPS subdomain, for example game.example.org
- Reverse proxy: YunoHost
- Application: Docker container
- Database: PostgreSQL container with persistent volume
- Application host binding: loopback only, for example 127.0.0.1:3087
- No public database port

The exact domain and filesystem path are deployment configuration, not hard-coded application values.

## 2. Browser-only delivery

Players open the game URL in a modern browser. Static frontend assets are served by the application/container. API and Socket.IO use the same HTTPS origin to simplify cookies, reverse proxy and browser security.

YunoHost must proxy normal HTTP requests and WebSocket upgrade requests. TLS is provided by YunoHost.

## 3. Container topology

Production Compose defines:
- app: published application image, bound to loopback only
- db: PostgreSQL, internal Docker network only
- persistent database volume
- health checks and restart policies

Do not run destructive database schema synchronization on every application start. Use versioned migrations and a controlled deployment step. Back up the database before a schema migration.

## 4. GitHub-driven update flow

1. Developer pushes a change to the protected main branch.
2. GitHub Actions runs tests and builds a multi-architecture container image, at minimum linux/arm64 for Raspberry Pi and optionally linux/amd64.
3. GitHub Actions publishes a versioned image and latest tag to GitHub Container Registry (GHCR).
4. After successful publication, the workflow connects to the server using a dedicated SSH deployment key.
5. The server runs the deployment script in the configured EVERWORLD directory.
6. The script pulls the new image, performs the approved migration step, restarts the application and checks health.
7. If health checks fail, deployment stops and the previous image tag remains available for rollback.

No GitHub personal access token or deployment private key is committed to the repository.

## 5. Required GitHub Actions secrets

Configure these repository secrets before enabling production deployment:

| Secret | Purpose |
|---|---|
| DEPLOY_HOST | Server hostname |
| DEPLOY_USER | Dedicated SSH deployment user |
| DEPLOY_SSH_KEY | Private key for GitHub Actions |
| DEPLOY_KNOWN_HOSTS | Pinned SSH host key entry |

GitHub's GITHUB_TOKEN is used by the workflow to publish packages with packages: write.

The server must be able to pull the GHCR package. If the package is private, authenticate Docker on the server with a narrowly scoped read-only package token. Store that credential on the server, not in the repository.

## 6. Server deployment script contract

A server-side deployment script should:
- Change to the fixed production directory.
- Acquire a deployment lock to prevent overlapping releases.
- Record the currently running image tag.
- Pull the requested immutable image tag.
- Run a pre-deployment database backup.
- Run versioned migrations in a controlled manner.
- Restart the app service.
- Wait for the health endpoint.
- Record the deployed version.
- On failure, restore the previous application image where schema compatibility permits and report the error.

Database rollback is not automatically safe after destructive migrations. Prefer backward-compatible expand/contract migrations.

## 7. Rollback

Each release uses an immutable image tag (commit SHA or release version). Keep the last known-good image available. Roll back the application image if health checks fail, but do not blindly restore an old database schema. Database restore is a deliberate recovery operation from a verified backup.

## 8. Backup policy

Back up PostgreSQL to storage outside the database container and, ideally, outside the Raspberry Pi. Define retention and encryption. Test restore regularly. A backup that has never been restored is not a verified backup.

## 9. Operational safeguards

- Deploy only from the protected main branch or approved release tags.
- Run tests before publishing.
- Use a dedicated deploy account with minimal SSH permissions.
- Restrict application port to loopback.
- Do not expose PostgreSQL.
- Keep secrets out of Git and logs.
- Require health checks before declaring a deployment successful.
- Record deployment commit/image version.
- Schedule maintenance windows for breaking migrations.

## 10. Status

This document defines the intended CI/CD design. The workflow and server deployment script must be configured with the actual host, domain, SSH account and package access before automatic production deployment can be enabled.
