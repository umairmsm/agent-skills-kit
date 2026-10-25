---
name: docker-stack-dev
description: Local Docker Compose development stack — Postgres, Redis, MinIO and the backend service, driven through a Makefile. Use when starting or stopping the stack, diagnosing a container that will not come up, resolving host-vs-container connection errors, or resetting local data volumes.
---

# Docker Compose Dev Stack

A four-service local environment: Postgres (pgvector), Redis, MinIO (+ a one-shot
`minio-init` that creates buckets), and the application backend.

## Daily commands

```bash
make up          # start everything detached
make ps          # container status
make logs        # follow all logs
make down        # stop, volumes retained
make build       # rebuild the backend image after dependency changes
make dev         # run backend on the HOST in watch mode, infra in containers
make migrate     # apply pending migrations (host-side)
make db-check    # verify DB connectivity + extensions
```

`make dev` is the normal inner loop: infra in containers, the app on the host with
watch mode. `make up` runs the backend containerized too — use it to reproduce
something environment-specific, not for everyday editing.

## The host-vs-container hostname trap

This is the single most common failure, and its error message is misleading.

Inside the compose network, services resolve by **service name**: `postgres`, `redis`,
`minio`. From the host, they resolve as `localhost` on the published port.

So `DATABASE_URL` differs depending on where the process runs:

| Process runs on | Host |
|---|---|
| host (`make dev`, `make migrate`) | `localhost` |
| container (`make up`) | `postgres` |

`ECONNREFUSED 127.0.0.1:5432` from inside a container means the container is trying
`localhost` and should be using the service name. `getaddrinfo ENOTFOUND postgres` on
the host means the reverse. Read which side threw before changing anything.

## Startup ordering

`depends_on` waits for the container to *start*, not for the service to be *ready*.
Postgres accepts connections several seconds after the container is up. A backend that
connects on boot will crash-loop against a healthy-looking stack.

Fix it with a healthcheck plus `condition: service_healthy`:

```yaml
postgres:
  healthcheck:
    test: ["CMD-SHELL", "pg_isready -U $$POSTGRES_USER"]
    interval: 5s
    timeout: 5s
    retries: 10
backend:
  depends_on:
    postgres:
      condition: service_healthy
```

Retry-on-boot in the application is the belt to that braces — do both.

## Diagnosing a container that will not come up

```bash
make ps                              # look at STATUS and exit codes
docker compose logs <service>        # the actual error, almost always here
docker compose logs --tail=50 backend
docker compose config                # renders the file with env substituted
docker compose exec postgres psql -U <user> -d <db> -c '\dx'
```

`docker compose config` is the fastest way to catch an unset variable — an empty `${VAR}`
renders as blank rather than erroring, producing confusing downstream failures.

Exit code 137 is OOM — raise Docker's memory limit or the container's. Exit code 1 with
no logs usually means the entrypoint failed before the app started.

## Volumes and resetting

Named volumes (`postgres_data`, `redis_data`, `minio_data`) survive `make down`. That is
what you want most of the time.

To wipe local data and start clean:

```bash
docker compose down -v      # DESTROYS all volume data — local only, never on a server
make up
make migrate
```

Confirm before running this. It is unrecoverable, and it looks identical to `make down`
in a terminal scrollback.

## MinIO

The console is on its published port; the API is separate. `minio-init` runs once to
create buckets and exits — a `minio-init` container in `Exited (0)` state in `make ps` is
success, not a failure. Buckets are not recreated on restart unless the volume was wiped.

## Environment

The repo-root `.env` feeds both compose and host-side scripts. Keep `.env.example`
current: every key the stack needs, with a safe placeholder value. A missing key in
`.env` fails at container start with an unhelpful message, and `.env.example` is the only
record of what was required.

Never commit `.env`.
