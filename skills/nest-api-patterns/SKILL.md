---
name: nest-api-patterns
description: NestJS service architecture — module layout, config and validation, TypeORM wiring, BullMQ queues, throttling, and MinIO object storage. Use when adding or refactoring a module, controller, service, queue processor, or DTO in a NestJS backend, or when wiring a new external dependency into the DI container.
---

# NestJS API Patterns

Conventions for a NestJS backend running TypeORM against Postgres, BullMQ on Redis,
and MinIO for objects.

## Module layout

One directory per bounded context. A module owns its entities, DTOs, service, and
controller; it does not reach into another module's files.

```
src/
  <context>/
    <context>.module.ts      # imports, providers, exports
    <context>.controller.ts  # HTTP surface only — no business logic
    <context>.service.ts     # business logic, injectable
    dto/                     # class-validator DTOs
    entities/                # TypeORM entities
  database/
    data-source.ts           # TypeORM CLI datasource (migrations read this)
    migrations/
  common/                    # guards, interceptors, filters, shared pipes
```

Cross-context access goes through the other module's **exported service**, injected via
DI. If two modules need each other, the shared piece belongs in a third module.

## Configuration

Use `@nestjs/config` with a typed accessor. Never read `process.env` inside a service —
it defeats testing and hides required keys.

```ts
// good
constructor(private readonly config: ConfigService) {}
const url = this.config.getOrThrow<string>('DATABASE_URL');

// bad
const url = process.env.DATABASE_URL;
```

`getOrThrow` over `get` for anything the service cannot start without — fail at boot,
not on the first request.

## Validation

Every DTO carries `class-validator` decorators, and the app enables a global pipe:

```ts
app.useGlobalPipes(new ValidationPipe({
  whitelist: true,            // strip unknown properties
  forbidNonWhitelisted: true, // 400 on unknown properties
  transform: true,            // apply class-transformer
}));
```

`whitelist` without `forbidNonWhitelisted` silently drops typo'd fields — set both.

## TypeORM

- Entities are declarative; queries live in services or custom repositories.
- **Never enable `synchronize: true`.** Schema changes go through migrations only.
- Generate against the datasource, then read the generated SQL before applying:

```bash
npm run migration:generate -- src/database/migrations/AddThing
# read the file, then:
npm run migration:run
npm run migration:revert   # if it went wrong
```

- Prefer the query builder over raw SQL; when raw is unavoidable, use parameter
  binding (`:param`), never string interpolation.

## Queues (BullMQ)

Long or failure-prone work — crawling, extraction, embedding, generation — goes on a
queue, never in the request path.

- Producers inject `@InjectQueue('<name>')` and `add()` a job.
- Processors are `@Processor('<name>')` classes in the owning module.
- Jobs must be **idempotent**: they will be retried. Key on a stable job id so a
  duplicate enqueue collapses instead of doubling work.
- Set explicit `attempts` and `backoff`; the default of one attempt loses work silently.

## Throttling

`@nestjs/throttler` is applied globally. Any endpoint that is expensive or hits a paid
upstream gets a tighter per-route `@Throttle()`. Public write endpoints always get one.

## Object storage (MinIO)

The MinIO client is a provider, not a per-call `new Client()`. Bucket names come from
config. Treat the API as S3-compatible so the same code works against S3 in production.
Stream large objects; never buffer a whole file to memory.

## Health and startup

Expose a health endpoint that checks the dependencies the service actually needs
(database, Redis, object store). A service that reports healthy while its database is
unreachable is worse than one that reports unhealthy.

## What to check before saying a change works

```bash
npm run build          # tsc must pass
npm run start:dev      # boots without unresolved DI
npm run db:check       # DB reachable, extensions present
```

A NestJS DI failure surfaces at boot with `Nest can't resolve dependencies of X` — read
the bracketed list, the `?` marks the parameter it could not satisfy. Usually the
provider is missing from the module's `providers`, or its module is missing from
`imports`, or it is not in the other module's `exports`.
