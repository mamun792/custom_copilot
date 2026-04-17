# সংক্ষিপ্ত নির্দেশনা (বাংলা)
- যেকোনো প্রজেক্টে এই নির্দেশনা সবসময় অনুসরণ করো (সব কোড জেনারেশন/অটোমেশন সহ)।
- Clean Architecture: Controller → Service → Repository → DB, এবং Validation → DTO → Response Resource বাধ্যতামূলক।
- সব ইনপুট validate/sanitize করো; secrets শুধুই .env-এ রাখো; refresh token সবসময় httpOnly cookie।
- API অবশ্যই /api/v1/ versioned; standard response format, rate limit, এবং idempotency key (payment/critical write)।
- Third‑party কলের জন্য circuit breaker + retry + fallback বাধ্যতামূলক।
- বিস্তারিত নিয়মাবলি দেখো: .github/dev-reference-EN.md

# Copilot Instructions — Full-Stack Dev Rules

## Developer Profile
Full-stack engineer: Java (Spring Boot) | PHP (Laravel 11) | C# (ASP.NET Core) | JS (Next.js 15, Vue.js 3)
Reference: See `.github/dev-reference-EN.md` for complete patterns, examples, and system design.

---

## Architecture (Always Apply)

**Layer order — never skip:**
```
Request → Controller → Service → Repository → Database
                            ↓
                    Validation → DTO → Response Resource
```

**Always use:**
- Repository Pattern — DB abstraction, interface in Domain, implementation in Infrastructure
- Service Layer — all business logic here, never in Controller or Repository
- DTO — typed input (Request DTO) and output (Response Resource), never raw arrays or `$request->all()`
- Domain Events — for side effects (email, inventory, audit)

**Use when needed:**
- CQRS — when read/write logic diverges significantly, or read replica needed
- DDD — complex domain with 3+ bounded contexts and rich business rules
- Saga Pattern — distributed transactions across services
- Circuit Breaker — every third-party API call (Stripe, Courier, bKash)
- Outbox Pattern — when DB write must guarantee event publication

---

## Folder Structure

**PHP / Laravel:**
```
app/
├── Domain/          # Entities, Value Objects, Repository Interfaces, Domain Events
├── Application/     # Commands, Queries, Handlers, DTOs
├── Infrastructure/  # Eloquent Repositories, External API clients, Queue Jobs
├── Http/
│   ├── Controllers/Api/V1/
│   ├── Middleware/
│   ├── Requests/    # Form Request validation (Request DTOs)
│   └── Resources/   # API Resources (Response DTOs)
└── Providers/       # DI bindings
tests/
├── Unit/            # Service layer logic
├── Feature/         # API endpoint tests
└── Load/            # k6 scripts
```

**Java / Spring Boot:**
```
src/main/java/com/company/
├── domain/          # Entities, Value Objects, Repository interfaces
├── application/     # Commands, Queries, Handlers, Services
├── infrastructure/  # JPA Repos, External clients, Config
└── presentation/    # Controllers, DTOs, Exception handlers
```

**C# / ASP.NET Core:**
```
CompanyApp.Domain/         # Entities, Interfaces, Value Objects
CompanyApp.Application/    # Commands, Queries, DTOs, Handlers
CompanyApp.Infrastructure/ # EF Core, External services
CompanyApp.API/            # Controllers, Middleware, Program.cs
```

---

## Security (Non-Negotiable)

**Authentication:**
- JWT: access token 15min + refresh token 7 days
- Refresh token: httpOnly Secure cookie ONLY (never localStorage)
- RBAC: check permission on every route AND every API endpoint
- Failed login: rate limit + lockout after 5 attempts

**Input / Output:**
- Validate ALL input — whitelist approach, never blacklist
- Never use raw string concat in SQL — ORM or prepared statements always
- Sanitize output — prevent XSS
- File uploads: whitelist extension + MIME type + max size check
- Validate URLs before fetching — block private IP ranges (SSRF prevention)

**Secrets:**
- `.env` only — never hardcode in source code
- Never log: passwords, tokens, card numbers, PII, JWT tokens
- Production: use Vault or AWS Secrets Manager, not `.env`

**Webhooks:**
- Verify HMAC signature before processing
- Check idempotency (event already processed?)
- Return 200 immediately, process async via queue

**HTTP Security Headers (always add):**
```
Content-Security-Policy
X-Frame-Options: DENY
X-Content-Type-Options: nosniff
Strict-Transport-Security
Referrer-Policy: strict-origin
```

---

## API Design Rules

**Response format — always this exact structure:**
```json
{
  "success": true,
  "data": {},
  "message": "Order placed successfully",
  "errors": {},
  "code": "ORDER_PLACED",
  "timestamp": "2025-04-14T10:00:00Z"
}
```

**Routing:**
- Versioned: `/api/v1/` — always
- RESTful: GET (read) / POST (create) / PUT (replace) / PATCH (partial) / DELETE
- Rate limit on every endpoint
- Idempotency-Key header: required for payment + critical write operations
- Pagination: `?page=1&limit=20&sort=created_at&order=desc`

**Status codes:**
- 201 Created — POST success
- 204 No Content — DELETE success
- 400 Bad Request — invalid input
- 401 Unauthorized — no/invalid token
- 403 Forbidden — valid token, no permission
- 409 Conflict — duplicate resource
- 422 Unprocessable — validation error
- 429 Too Many Requests — rate limit hit
- 500 Server Error — never expose internal details

---

## Pattern Selection Guide

| Scenario | Pattern / Tool |
|---|---|
| Multiple payment methods (bKash, Stripe, Nagad) | Strategy Pattern |
| Add behavior without modifying class | Decorator Pattern |
| Complex object creation | Factory Pattern |
| Session, cache, rate limit counters | Redis |
| Email, PDF, reports, heavy processing | Queue Job |
| Payment processing | Queue + Idempotency + Pessimistic Lock |
| Third-party API (Stripe, Courier, bKash) | Circuit Breaker + Retry + Fallback |
| DB write must trigger event reliably | Outbox Pattern |
| Read-heavy data | Cache-Aside (must have invalidation plan) |
| Audit trail, notifications, side effects | Domain Events + Listeners |
| Multi-tenant query isolation | Global Scope (auto inject tenant_id) |
| Distributed transaction across services | Saga Pattern (choreography preferred) |
| Real-time updates | WebSocket (bidirectional) or SSE (server→client) |

---

## Database Rules

**Queries:**
- Never `SELECT *` — always name the columns needed
- Always eager load relations — never lazy load in loops (N+1)
- Use `EXPLAIN` on any query touching > 10k rows
- Pagination on large datasets: keyset pagination (not OFFSET)
- Batch inserts/updates — never loop individual queries

**Schema:**
- Soft delete: `deleted_at` timestamp — never hard delete
- Always: `created_at`, `updated_at` timestamps
- Migrations: always write both `up()` and `down()`
- Foreign keys: always index them
- UUID for distributed systems, auto-increment for simple apps

**Locking:**
- Payment / critical state: `lockForUpdate()` (pessimistic)
- Inventory / balance: `version` column + optimistic lock (return 409 on conflict)
- Always lock in consistent order to prevent deadlock

**Replica:**
- Payment reads: always from PRIMARY after write
- Analytics / reports: replica acceptable
- Configure `sticky: true` in Laravel for same-request read-after-write

---

## Caching Rules

- Cache-Aside pattern: check cache → miss → query DB → store in cache
- Always define TTL and invalidation strategy before caching anything
- Cache keys: namespaced (`product:{tenantId}:{productId}`)
- Never cache: payment data, real-time critical data, POST responses
- Cache stampede prevention: use mutex lock on cache miss
- Redis eviction policy: `allkeys-lru` for cache, `noeviction` for queues

---

## Testing Rules

**What to test:**
- Unit tests: service layer business logic, domain methods
- Integration tests: API endpoints (full request → response)
- Contract tests: API contracts between services
- Load tests: critical endpoints before production

**Rules:**
- Minimum 70% coverage
- Freeze time: `Carbon::setTestNow()` — never use real `now()`
- Fake external: `Queue::fake()`, `Event::fake()`, `Http::fake()`
- Seed data: deterministic only — no `rand()`, no `faker->unique()`
- Never hit real external APIs, payment gateways, or send real emails in tests
- Multi-tenant tests: create isolated test tenant, clean up in `tearDown()`

---

## Code Quality

- Max function length: 20-30 lines — extract if longer
- Max file length: 300 lines — split into smaller classes
- Names: descriptive (`processRefund` not `do`, `getActiveOrders` not `data`)
- One reason to change per class (Single Responsibility)
- No dead code in production
- No debug statements in commits: `console.log`, `dd()`, `var_dump()`, `print_r()`
- SOLID principles in every class

---

## Git Rules

**Branches:**
```
feature/order-refund-flow
bugfix/cart-negative-quantity
hotfix/payment-timeout
release/v2.1.0
chore/update-dependencies
```

**Commits (Conventional Commits):**
```
feat(payment): add bKash refund endpoint
fix(cart): prevent negative quantity on update
perf(order): add composite index on tenant_id + status
test(auth): add JWT refresh token rotation tests
chore(deps): upgrade laravel to 11.x
refactor(order): extract pricing logic to domain service
docs(api): update OpenAPI spec for order endpoints
```

**Rules:**
- Never push directly to `main` or `develop`
- All changes via PR — no exceptions
- Squash WIP commits before PR
- Rebase feature branch on develop (not merge) to stay updated

---

## LLM Integration (Anthropic Claude)

**Latest models (April 2025):**
```javascript
// Daily coding, features, reviews
model: "claude-sonnet-4-20250514"

// Complex architecture, large codebase analysis
model: "claude-opus-4-20250514"
```

**Standard API call:**
```javascript
const response = await fetch("https://api.anthropic.com/v1/messages", {
  method: "POST",
  headers: {
    "x-api-key": process.env.ANTHROPIC_API_KEY,
    "anthropic-version": "2023-06-01",
    "Content-Type": "application/json"
  },
  body: JSON.stringify({
    model: "claude-sonnet-4-20250514",
    max_tokens: 4096,
    system: "You are a [role]. [constraints]. Output format: [format].",
    messages: [{ role: "user", content: sanitizedUserInput }]
  })
});
```

**Rules:**
- Sanitize user input before sending — detect prompt injection
- Never send: passwords, card numbers, PII, internal secrets
- Validate LLM output structure before using in business logic
- Rate limit LLM calls per user (cost + abuse prevention)
- Cache responses for identical deterministic prompts (TTL: 1hr)
- Always set `max_tokens` — never leave unbounded
- Graceful fallback if LLM API fails

---

## Always Remember

1. Security first — then feature
2. Cache only when you have a clear invalidation plan
3. Index only after checking with EXPLAIN
4. Idempotency is mandatory for payments and critical writes
5. Circuit Breaker on every third-party service call
6. Design for failure — what happens when X fails?
7. Stateless services — session state in Redis, not in-memory
8. Small PRs — easier to review, faster to merge
9. Test behavior, not implementation
10. Write the test you wish you had when the bug hits production
