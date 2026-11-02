# iam

Generated from the bus-2.0 Go boilerplate with
`db=postgres kafka=true redis=true`.

## Run locally

```bash
make up                                   # postgres, mongo, kafka, redis (repo root)
cp services/iam/.env.example services/iam/.env
make db svc=iam                     # create the database
make migrate-diff svc=iam name=init # first time only: generate SQL from the ent schema
make migrate-up svc=iam
make run svc=iam mode=topics        # create Kafka topics
make run svc=iam mode=all           # API + outbox relay + consumer
```

API on `http://localhost:8100`, health/metrics on `http://localhost:8101`.

```bash
CID=0190a8f0-0000-7000-8000-000000000001
curl -s -X POST localhost:8100/v1/items \
  -H "X-Company-ID: $CID" -H "Content-Type: application/json" \
  -H "Idempotency-Key: $(uuidgen)" \
  -d '{"name":"Dhaka → Chattogram"}'
curl -s localhost:8100/v1/items -H "X-Company-ID: $CID"
```

## Run modes

| Mode | What it runs |
|---|---|
| `api` | public HTTP API on `HTTP_ADDR` |
| `relay` | outbox → Kafka publisher (leader-elected) |
| `consumer` | Kafka consumer group `iam.item-events` |
| `topics` | create Kafka topics (local only) |
| `all` | everything in one process (local only) |

## Layout

```
cmd/iam/          main: config, run modes
internal/config/        env configuration
internal/bootstrap/     dependency wiring per run mode
internal/api/           HTTP handlers, permissions
internal/domain/        business types
internal/service/       business logic (+ unit tests with fakes)
internal/store/         persistence (postgres)
internal/events/        owned topics, event types, payloads
internal/consumer/      Kafka handlers
ent/schema/             ent schema (run `make gen` after changes)
migrations/             Atlas SQL migrations
```

The `Item` entity is a sample. Replace it with your own entities and keep
the patterns: company scoping, outbox events, inbox claims, status
instead of delete, permission check per record.
