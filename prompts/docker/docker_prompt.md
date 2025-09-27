# (Draft)

# Production‑ready Docker Compose checklist (go‑siem)

## Compose file hygiene
- Remove obsolete `version:` and any non‑YAML lines.
- Validate YAML; keep `deployments/docker-compose.yml` as the single source.
- Use deterministic relative paths; document the working directory.

## Build contexts and Dockerfiles
- Set `build.context` to project root (`../`) so `go.mod`/`go.sum` are included.
- Point `dockerfile` to `cmd/producer/Dockerfile` and `cmd/ingestor/Dockerfile`.
- Add a strict `.dockerignore` (e.g., `.git/`, `deployments/`, `build/`, test artifacts).

## Images and builds
- Use multi‑stage builds; ship minimal runtime (e.g., distroless/static).
- Run as non‑root `USER`; avoid shell in final image.
- Pin base images and dependencies.

## Configuration and env
- Keep runtime config in `deployments/.env`; never bake secrets into images.
- Prefer `secrets:` with `_FILE` envs (e.g., `POSTGRES_PASSWORD_FILE`).
- When running from repo root, use `--env-file 'deployments/.env' -f 'deployments/docker-compose.yml'`.

## Services and dependencies
- Add healthchecks for Postgres, Kafka, and apps.
- Use `depends_on` with `condition: service_healthy` (or `service_completed_successfully` for one‑shots).
- Ensure migration paths and working dirs are correct.

## Migrations
- Run a dedicated `migrator` as one‑off (`restart: "no"`) behind DB health.
- Make migrations idempotent; store migration artifacts in image or a read‑only mount.

## Networking and exposure
- Use a dedicated user‑defined network.
- Publish only necessary ports; keep Postgres/Kafka internal.
- Configure Kafka advertised listeners for the Compose network.

## Data persistence
- Use named volumes for Postgres, Kafka, Grafana data.
- Avoid bind mounts in production; document backup/restore of volumes.

## Security hardening
- Run as non‑root; set `read_only: true` where possible.
- Mount writable dirs as `tmpfs` if needed.
- `cap_drop: ["ALL"]`, add only required caps; `security_opt: ["no-new-privileges:true"]`.
- Set `pids_limit`, `ulimits` (e.g., `nofile`), and `init: true`.

## Resource management
- Set CPU/memory limits and reservations.
- Configure `restart: unless-stopped`, `stop_grace_period`, and `stop_signal`.

## Logging and observability
- Configure log driver with rotation (`json-file` with `max-size`/`max-file` or `local`).
- Expose metrics endpoints; keep Grafana/Prometheus under a separate `profile`.

## Image hygiene
- Pin image tags (or digests) for Postgres, Kafka, and base images; avoid `latest`.
- Use `pull_policy: always` (or `--pull always`) for security updates.

## Execution discipline
- Do not bind‑mount source in production; ship images only.
- Prefer CI to build and push images; reference them via `image:` in Compose.

## Operational extras
- Use `x-` extension anchors to DRY shared settings (healthcheck, security, logging).
- Provide app readiness checks to avoid accepting traffic too early.
- Document bootstrap, upgrades, backups, and secret rotation.

## Scope note
- Compose suits single‑host production; use an orchestrator for HA/rollouts/auto‑healing.
