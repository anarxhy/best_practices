# IT Development Best Practices — Comprehensive Guide (with Examples)

> # IT Development Best Practices — Comprehensive Guide (with Examples)

> A practical, example-driven handbook covering core dev principles, SDLC & methodologies, architecture patterns, coding standards, testing, CI/CD, security, DevOps/SRE, data, APIs, operations, and team practices.

---

## Table of Contents

1. Core Development Principles
2. SOLID Principles (with Code Examples)
3. Software Development Lifecycle (SDLC) & Methodologies
4. Architecture Patterns & Trade‑offs
5. Version Control & Branching
6. Coding Standards & Documentation
7. Testing Strategy & Quality
8. CI/CD, Release Strategies & Environments
9. Security by Design & Secure Coding
10. Observability, SRE & Reliability
11. Data & Database Practices
12. API Design (REST & GraphQL)
13. Performance & Scalability
14. Infrastructure as Code & Cloud‑Native
15. Collaboration, Governance & Compliance
16. Checklists & Templates

---

## 1) Core Development Principles

* **KISS**: Keep it simple; avoid unnecessary complexity.
* **DRY**: Don’t Repeat Yourself; extract shared logic.
* **YAGNI**: You aren’t gonna need it; defer speculative features.
* **SoC**: Separation of Concerns; layer responsibilities.
* **Fail Fast**: Surface errors early (assertions, validations, circuit breakers).
* **12‑Factor App** (for services): Config in env, stateless processes, disposability.
* **Code for Humans**: Readability first; optimize only when needed.
* **Small Batches**: Short PRs, small releases, quick feedback loops.

### Easy Mini‑Examples

**DRY — Extract Shared Logic**

```python
# BAD: duplicate logic
area_square = side * side
area_rect = width * width  # wrong reuse

# GOOD: one shared function
def area(width, height):
    return width * height

print(area(4,4)) # square
print(area(4,6)) # rectangle
```

**YAGNI — Avoid Over‑Engineering**

```js
// BAD: building a full plugin system for one feature
function exportData(data, format){ /* xml, csv, pdf, yaml... */ }

// GOOD: implement only what is needed now
function exportDataAsCsv(data){ /* just csv */ }
```

**SoC — Separation of Concerns**

```ts
// BAD: logic + DB + HTTP in one place
app.post('/user',(req,res)=>{ db.save(req.body); res.send('ok') })

// GOOD: separate layers
class UserService{ create(u){ return db.save(u) } }
app.post('/user',(req,res)=>{ userService.create(req.body); res.send('ok') })
```

**Fail Fast**

```java
// Validate upfront instead of failing later
if (input == null) throw new IllegalArgumentException("input required");
```

**12‑Factor App**

```bash
# Config in environment, not hard‑coded
export DB_URL=postgres://...
```

```js
// App reads from env
const db = connect(process.env.DB_URL)
```

**Code for Humans**

```python
# BAD: clever but unreadable
s=[i*i for i in d if not i%2]

# GOOD: clearer, even if longer
squares_of_even = []
for i in data:
    if i % 2 == 0:
        squares_of_even.append(i * i)
```

**Small Batches**

* BAD: PR with 5,000 lines, multiple features.
* GOOD: PR with 100 lines, one feature or bugfix.

---

## 2) SOLID Principles (with Code Examples)

**S — Single Responsibility**: One reason to change.

```ts
// Before: mixes parsing + persistence
class UserImporter{ import(csv:string){ /* parse CSV */ /* save DB */ }}

// After: separate concerns
class CsvParser { parse(csv:string){ /*...*/ } }
interface UserRepo { save(u:any):Promise<void> }
class DbUserRepo implements UserRepo { async save(u:any){ /*...*/ } }
class UserImporter { constructor(private p:CsvParser, private r:UserRepo){}
  async import(csv:string){ const users=this.p.parse(csv); for(const u of users) await this.r.save(u) }
}
```

**O — Open/Closed**: Open for extension, closed for modification.

```ts
interface DiscountPolicy { apply(amount:number):number }
class NoDiscount implements DiscountPolicy { apply(a){return a} }
class SeasonalDiscount implements DiscountPolicy { constructor(private pct:number){} apply(a){return a*(1-this.pct)} }
class Checkout { constructor(private policy:DiscountPolicy){} total(a:number){return this.policy.apply(a)} }
// Add a new policy without touching Checkout
```

**L — Liskov Substitution**: Subtypes must be substitutable.

```ts
abstract class Bird{ abstract fly():string }
class Sparrow extends Bird{ fly(){ return 'flap' } }
class Ostrich extends Bird{ /* violates LSP if forced to fly */ } // Prefer different hierarchy
```

**I — Interface Segregation**: Many small interfaces over one large.

```ts
interface Printer{ print(doc:string):void }
interface Scanner{ scan():string }
// Client needs only printing
class ReportService{ constructor(private p:Printer){} run(){ this.p.print('monthly') } }
```

**D — Dependency Inversion**: Depend on abstractions, not concretions.

```ts
interface Logger{ info(msg:string):void }
class ConsoleLogger implements Logger{ info(m){ console.log(m) } }
class OrderService{ constructor(private log:Logger){} create(){ this.log.info('created') } }
// Swap ConsoleLogger for JSONLogger in tests without changing OrderService
```

---

# Software Development Lifecycle (SDLC) & Methodologies

## SDLC Phases & Typical Artifacts

1. **Discovery & Requirements** → Vision, personas, high-level scope, user stories with acceptance criteria.
2. **Design & Architecture** → ADRs, system context, ERD/API specs, threat model.
3. **Implementation** → Code, tests, doc, feature flags.
4. **Verification** → Automated tests, UAT sign-off.
5. **Release** → Release notes, migration plan, runbooks.
6. **Operations & Monitoring** → Dashboards, alerts, incident docs.
7. **Retrospective** → Post-release review, improvements.

---

## Methodologies Explained

### Waterfall

* **Brief**: Sequential phases; requirements frozen early. Good for regulated/contract-driven projects.
* **Example**: Airplane navigation software needing regulatory sign-off.
* **Drawing**:

```
[Requirements] → [Design] → [Implementation] → [Verification] → [Maintenance]
```

### Scrum

* **Brief**: Time-boxed sprints (2–4 weeks), roles (PO, Scrum Master, Devs), ceremonies. Good when scope evolves.
* **Example**: E-commerce platform rolling out weekly features.
* **Drawing**:

```
[Product Backlog]
      ↓
[ Sprint Planning ] → [ Sprint Execution ] → [ Review ] → [ Retrospective ]
      ↑———————————————————————————————————————————————↑
```

### Kanban

* **Brief**: Continuous flow, visualize tasks, limit WIP. Ideal for operations/support.
* **Example**: IT support desk managing incident tickets.
* **Drawing**:

```
[To Do] → [In Progress] → [Review] → [Done]
   (WIP limits applied in In Progress)
```

### Extreme Programming (XP)

* **Brief**: Strong engineering practices: TDD, pair programming, CI/CD, collective ownership.
* **Example**: Fintech API startup delivering weekly high-quality releases.
* **Drawing**:

```
[Plan] → [Pair Code + TDD] → [Integrate] → [Release] → [Feedback]
```

### Dual-Track Agile

* **Brief**: Discovery (research/prototyping) runs in parallel with Delivery (build/ship).
* **Example**: Mobile app team prototyping AI recommendations while shipping bug fixes.
* **Drawing**:

```
Discovery: [Hypothesize] → [Prototype] → [Validate]
Delivery:  [Plan] → [Build] → [Release]
   (Both tracks run in parallel)
```

---

## Example — User Story & Acceptance Criteria

* *Story*: As a shopper, I want to save my cart so I can purchase later.
* *Acceptance Criteria*: Given items in cart, When I click “Save Cart”, Then a named draft appears under My Carts and persists for 30 days.


---
# Architecture Patterns & Trade-offs

## Layered (N‑tier)

* **Details**: Presentation, business logic, and data access layers. Common in traditional web apps.
* **Pros**: Simple, widely understood, good for small to medium projects.
* **Cons**: Risk of tight coupling between layers; business logic may become thin.
* **Diagram**:

```
[ UI Layer ]
     ↓
[ Business Layer ]
     ↓
[ Data Layer ]
```

---

## Hexagonal / Clean Architecture

* **Details**: Domain-centric. Uses ports and adapters to isolate business rules from infrastructure.
* **Pros**: Testable, infrastructure can be swapped, domain remains clean.
* **Cons**: Steeper learning curve; extra abstraction.
* **Diagram**:

```
     [ UI / API ]     [ DB Adapter ]
            ↘         ↙
      [ Application Services ]
                  ↓
               [ Domain ]
```

---

## Monolith

* **Details**: All components in one deployable unit.
* **Pros**: Simple to deploy and monitor, fewer moving parts.
* **Cons**: Harder to scale development teams, long builds, larger blast radius for failures.
* **Diagram**:

```
[ Monolith Application ] → [ Single Database ]
```

---

## Microservices

* **Details**: Independent, small services, each with its own database.
* **Pros**: Independent deployability, team autonomy, scalability.
* **Cons**: Distributed system complexity, network overhead, data consistency challenges.
* **Diagram**:

```
[ Service A ] ↔ [ DB A ]
[ Service B ] ↔ [ DB B ]
[ API Gateway / Service Mesh ]
```

---

## Event‑Driven (Pub/Sub, CQRS)

* **Details**: Services communicate asynchronously via events.
* **Pros**: Decoupled, scalable, resilient to failure.
* **Cons**: Eventual consistency, harder debugging/tracing.
* **Diagram**:

```
[ Producer Service ] → [ Event Bus ] → [ Consumer Services ]
```

---

## Serverless

* **Details**: Functions running on demand in a managed environment.
* **Pros**: Scale-to-zero, pay only for usage, fast iteration.
* **Cons**: Cold starts, vendor lock-in, limited runtime execution.
* **Diagram**:

```
[ Client ] → [ Function as a Service ] → [ Managed Database / External Service ]
```

---

## Choosing Guide

* Start with a **modular monolith**; extract services only when necessary.
* Use **async events** to integrate across teams.
* Keep **sync APIs** small, stable, and versioned.

---

## Example — Clean Architecture (Pseudo‑Diagram)

```
[ Controllers ] → [ Application Services ] → [ Domain ] ← [ Ports ] ← [ Adapters: DB/HTTP/Queue ]
```


---

## 5) Version Control & Branching

* **Trunk‑Based Development** with short‑lived branches; integrate daily.
* Use **feature flags** for incomplete work.
* **Commit hygiene**: imperative present tense; small, logical commits.

**Example — PR Checklist**

* [ ] Linked issue & scope clear
* [ ] Tests added/updated
* [ ] Doc/ADR updated
* [ ] Security considerations addressed
* [ ] Rollback plan noted

---

## 6) Coding Standards & Documentation

* **Style**: Linters/formatters (ESLint, Prettier, Pylint, Black).
* **Idiomatic Code**: Follow language conventions.
* **Docs**: README, contribution guide, code comments where intent isn’t obvious.
* **ADRs**: Record key decisions with context/alternatives.

**ADR Template (snippet)**

```
# ADR 0001: Choose Postgres for OLTP
Context: need relational integrity + ACID.
Decision: Use Postgres 16 managed service.
Alternatives: MySQL, Aurora, CockroachDB.
Consequences: native JSONB, logical replication, managed backups.
```

---

## 7) Testing Strategy & Quality

* **Testing Pyramid**: Unit (fast, many) → Integration → E2E (few, critical paths).
* **TDD/BDD**: Given/When/Then for business logic.
* **Non‑Functional**: Performance, security, accessibility (a11y), i18n, chaos testing.
* **Test Data**: Factories/builders; avoid shared mutable fixtures.

**Example — Minimal Testing Targets**

* Unit ≥ 70% critical modules; Integration for all data boundaries; E2E for checkout, signup, auth.

**Example — Jest Unit Test**

```ts
import { totalWithTax } from './pricing'

test('applies tax to sum', ()=>{
  expect(totalWithTax([{price:100},{price:50}])).toBe(180)
})
```

---

## 8) CI/CD, Release Strategies & Environments

* **CI**: Lint → build → unit tests → security scan → artifact.
* **CD**: Deploy via pipelines; immutable artifacts; infra as code.
* **Environments**: Dev → Staging → Prod; keep **parity**; seed prod‑like data safely.
* **Release Strategies**: Blue/Green, Canary, Feature Flags, Shadow.

**Example — GitHub Actions CI (snippet)**

```yaml
name: ci
on: [push, pull_request]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20 }
      - run: npm ci
      - run: npm run lint && npm test -- --ci
      - run: npm run build
```

**Example — Canary Steps**

1. Deploy v2 to 5% traffic; watch latency, error rate, saturation.
2. Increase to 25% → 50% → 100% if SLOs hold.
3. Auto‑rollback if error budget burn > threshold.

---

## 9) Security by Design & Secure Coding

* **Threat Modeling**: STRIDE per component; update with each change.
* **OWASP Top 10**: Validate inputs, sanitize outputs, parameterized queries.
* **Secrets**: Never in code; use vault/KMS; rotate regularly.
* **AuthN/Z**: Centralized identity, least privilege, MFA for ops.
* **Supply Chain**: Pin deps, use SBOMs, sign artifacts.

**Example — Input Validation (Express)**

```ts
import express from 'express'
import { z } from 'zod'
const app = express(); app.use(express.json())
const schema = z.object({ email: z.string().email(), qty: z.number().int().min(1) })
app.post('/order', (req,res)=>{
  const parsed = schema.safeParse(req.body)
  if(!parsed.success) return res.status(400).json({error:'invalid input'})
  // ...safe to use parsed.data
  res.status(201).end()
})
```

---

## 10) Observability, SRE & Reliability

* **Golden Signals**: Latency, Traffic, Errors, Saturation.
* **SLIs/SLOs**: Define measurable targets; use error budgets.
* **Tracing**: Propagate correlation IDs; sample wisely.
* **Runbooks**: Clear steps to diagnose/mitigate common incidents.

**Example — SLO Definition**

* API success rate ≥ 99.9% monthly; p95 latency ≤ 300ms for /checkout.
* Alert if burn rate > 2× over 1h; page if > 6× over 30m.

---

## 11) Data & Database Practices

* **Migrations**: Versioned, reversible (Flyway/Liquibase); run in CI before deploy.
* **Backups & DR**: RPO/RTO objectives; test restores quarterly.
* **Schema Design**: Explicit constraints, indexes, naming conventions.
* **Data Lifecycle**: Retention, archival, GDPR/PII handling, DPO reviews.

**Example — Safe Backfill**

1. Add nullable column → backfill in batches with throttling → switch reads → make NOT NULL.

---

## 12) API Design (REST & GraphQL)

* **REST**: Nouns, plural resources; use standard codes; HATEOAS optional.
* **Versioning**: /v1, or media type; never breaking changes without a new version.
* **Pagination & Filtering**: limit/offset or cursor; document defaults and caps.
* **Idempotency**: Keys for POST where appropriate.
* **GraphQL**: Schema first; limit query complexity/depth.

**Example — REST Contract**

```
GET /v1/orders?customerId=123&cursor=abc → 200 { items: [...], nextCursor: "..." }
POST /v1/orders (Idempotency-Key: uuid) → 201 Location: /v1/orders/456
```

---

## 13) Performance & Scalability

* **Measure**: Profilers, APM, flame graphs; set budgets (TTI, p95 latency).
* **Caching**: Client/CDN/app tier; choose TTL & invalidation rules.
* **Concurrency**: Use queues, backpressure, circuit breakers.
* **Capacity Planning**: Load testing before big launches.

**Example — Cache Aside Pattern**

1. Read: check cache → miss → read DB → populate cache.
2. Write: update DB → invalidate cache key.

---

## 14) Infrastructure as Code & Cloud‑Native

* **IaC**: Terraform/CloudFormation; code reviews & plans.
* **Containers**: Minimal base images; non‑root user; health/liveness probes.
* **Kubernetes**: Resource requests/limits; readiness gates; PodDisruptionBudgets.
* **GitOps**: Desired state in Git; controllers (ArgoCD/Flux) sync clusters.

**Example — Minimal Dockerfile**

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
USER node
EXPOSE 3000
CMD ["node","dist/server.js"]
```

---

## 15) Collaboration, Governance & Compliance

* **Definition of Ready/Done**: Clear entry/exit criteria.
* **Code Review Culture**: Kind, specific, actionable feedback.
* **Compliance**: Map controls (e.g., ISO 27001, SOC 2); automate evidence where possible.
* **Change Management**: RFCs for risky changes; CAB only when necessary.

**Definition of Done (sample)**

* Coded + reviewed + green CI
* Tests updated; coverage not reduced
* Docs/ADRs updated; feature flagged if partial
* Observability in place; runbook entries

---

## 16) Checklists & Templates

**Architecture Decision Checklist**

* [ ] Context & constraints captured
* [ ] Alternatives compared (with trade‑offs)
* [ ] Security, privacy, compliance assessed
* [ ] Cost & ops impact estimated
* [ ] Rollback/exit strategy defined

**Pre‑Merge Checklist**

* [ ] Build/lint/test pass
* [ ] Backward compatible (DB/API)
* [ ] Feature toggle for risky paths
* [ ] Secrets/config via env/secret store
* [ ] Telemetry added (logs/metrics/traces)

**Release & Rollback Plan Template**

```
Version: 2.3.0
Change summary: ...
Migration steps: ...
Canary scope: 10% region‑A
Success metrics: error rate < 0.2%, p95 < 300ms
Rollback: revert tag v2.2.5, run down-migration 034, invalidate cache keys order:*
Owner/Pager: team‑checkout#oncall
```

**Incident Postmortem (blameless) Outline**

* What happened, when (UTC + local)
* Impact (users, revenue, SLIs)
* Timeline of events
* Root cause(s) & contributing factors
* What went well/poorly
* Action items (owners, due dates)

---

## Appendix: Further Mini‑Examples

**Feature Flags (TypeScript)**

```ts
if (flags.isEnabled('newPricing')) { return calcV2() } else { return calcV1() }
```

**Retry with Backoff (pseudo)**

```
for attempt in 1..5: try op(); catch e: sleep(2**attempt + jitter)
```

**Circuit Breaker (concept)**

* Open after N failures; half‑open to probe; close on success.

**SQL Migration (Flyway style)**

```sql
-- V034__add_order_status.sql
ALTER TABLE orders ADD COLUMN status TEXT NOT NULL DEFAULT 'PENDING';
CREATE INDEX IF NOT EXISTS idx_orders_status ON orders(status);
```

---

### How to Use This Guide

* Adopt sections as policy; copy checklists into your PR template and runbooks.
* Start with modular monolith + trunk‑based + CI → evolve as scale demands.
* Review this document quarterly; keep ADRs and SLOs current.
