# Obligation rulings: what goes in, what stays behind

Come here whenever an item sits between behavior and implementation — which is most of the
interesting content. Every ruling below is a **default**, and `00-boundary.md` outranks all of them:
if a surface was declared `exact`, its representation is contract no matter what this file says.

The test is always the same one:

> Would a reimplementer, free to choose language, framework, architecture and layout, produce a
> boundary-visible difference if nobody told them this?

## The quick table

| Category | In the document | Left behind |
|---|---|---|
| Domain data | concepts, identity rules, invariants, lifecycles, derived-value formulas, required vs optional, value ranges | table layout, column types, indexes, ORM shape |
| Database schema | **only if a consumer you can't redeploy reads it** — then verbatim | otherwise everything |
| API surface | paths, methods, status codes, request/response shapes, error bodies, pagination, versioning, idempotency keys | routing library, serializer, controller layout |
| Auth | who may do what, session and token lifetimes, expiry and refresh behavior, lockout, revocation, what a failure looks like to the caller | JWT vs session vs opaque token, hashing library, middleware order |
| Permissions | the complete actor × capability matrix, including the deny cases and what a denial looks like | how checks are wired |
| Integrations | what is sent and received, when, failure and retry semantics, idempotency, sandbox vs live differences, what happens when it's down | SDK choice, client wrapper, connection pooling |
| Persistence | durability and consistency guarantees, what survives a restart or crash, transaction boundaries *as observable atomicity*, retention | storage engine, migrations, caching layer |
| Background work | trigger, cadence, delivery guarantees, ordering, idempotency, observable effects, what happens on failure and on catch-up | Celery vs cron vs Lambda, queue technology |
| Config | every key that changes behavior, its default, its precedence, and what changes when it's absent | file format, loading mechanism, secret store |
| Errors | user-visible messages and codes, which are retryable, what state is left behind | exception hierarchy |
| Migrations | data-repair rules and still-enforced compatibility windows | the migration history itself |
| Caching | only where it's observable — staleness windows, invalidation guarantees, what a user can see go stale | the cache itself |
| Logging & metrics | only what something outside depends on: an alert that parses a log line, a metric on a dashboard, an audit trail with a retention requirement | everything else |
| Feature flags | the behavior of each state, and which state is default | the flag system |

The rest of this file is the reasoning for the cases that actually go wrong.

## Data models

The trap is thinking "the schema is implementation" and losing the invariants with it. What a
reimplementer needs is everything that makes a record *valid* and *identifiable*, none of which is
recoverable from a description of features:

- **Identity** — what makes two records the same record. Natural keys, uniqueness constraints, case
  sensitivity, scoping (unique per tenant, not globally). Get this wrong and the rebuild silently
  permits duplicates.
- **Invariants** — what must always be true. "An order always has at least one line item"; "a
  balance never goes negative"; "exactly one address per user is primary". State them as rules with
  what happens when they'd be violated.
- **Lifecycle** — the states and the *legal transitions between them*. A state machine, drawn in
  text. Illegal transitions are behavior: say what happens when one is attempted.
- **Derived values** — every computed field, with its formula, its rounding, and when it's
  recomputed. Totals, scores, statuses derived from other fields. These are pure behavior and are
  lost more often than anything else on this list.
- **Optionality and defaults** — required vs optional, and what a missing value means at read time.
- **Relationships and cascades** — what happens to the children when the parent is deleted. Cascade
  behavior is user-visible and is almost never written down.

**When the schema itself is the contract:** a reporting tool, an analytics pipeline, another team's
service, or a BI dashboard reading the same tables makes the physical schema an interface. Then
table and column names, types, and nullability go into the document verbatim, in their own section,
marked `[contract]` — and say *why*, so a future reader knows the constraint isn't arbitrary.

## API contracts

If there is any client you cannot redeploy in lockstep, the API is `exact` and gets specified to the
byte: paths, methods, status codes per outcome, request and response field names and types, error
body shape, pagination scheme and its limits, sort defaults, filtering semantics, versioning scheme,
content types, and idempotency-key behavior.

Specify **what makes a request invalid and what the caller sees**, not just the happy path. Also:
whether unknown fields are rejected or ignored, whether field order matters anywhere, and what a
partial failure returns. These are the details contract tests catch and prose descriptions miss.

If the only client is the system's own frontend and both get rebuilt together, drop to `semantic`
and specify the *capability*, not the endpoint.

## Auth and permissions

Split cleanly: **mechanism is implementation, policy is contract.**

Out: JWT vs session cookie vs opaque token, hashing algorithm, where middleware sits. In: who may
do what; how long a session lives and what happens at expiry; whether sessions renew on activity;
what invalidates a session everywhere (password change, role change, explicit logout); lockout
thresholds and durations; password or credential rules that a user experiences; what a denied
request looks like — status, message, and crucially whether it distinguishes "not allowed" from
"doesn't exist", because that distinction is a security decision someone made deliberately.

**The permission matrix is always in, in full, including the denials.** It is the single most
commonly under-specified area in a rewrite, and the failure mode is a security hole rather than a
missing feature. Include the ones that look obvious.

Mechanism becomes contract the moment a token crosses the boundary — a mobile app holding a
long-lived refresh token, a partner validating your JWT signature. Then the token format is `exact`.

## Integrations and external services

Specify the **conversation**, not the client. For each integration: what triggers a call, what is
sent (semantically), what comes back, what the system does with each response class, timeout and
retry policy with backoff, whether the call is idempotent and how that's achieved, what happens when
the service is down or slow, whether failures are user-visible or absorbed, and what the sandbox
does differently from live.

The dangerous omission is always **failure behavior**. "Sends a receipt email" is the easy half.
"If the mail provider returns a 5xx the order still completes and the email is retried for 24 hours,
then dropped silently" is the half that determines whether the rebuild behaves the same on a bad
day — which is the only day anyone notices.

## Background work, jobs, and side effects

For each one: what triggers it, how often, whether it can run concurrently with itself, its delivery
guarantee (at-least-once, at-most-once, exactly-once-as-observed), whether ordering matters, whether
it's idempotent, what it does on failure, and what happens on catch-up after downtime — a job that
processes "yesterday" behaves very differently when it has been down for a week, and that behavior
is nearly always undocumented.

Also specify what is **observably asynchronous**. If a user sees a result immediately but the email
arrives later, that gap is part of the contract; a reimplementer who makes it synchronous has
changed the system's behavior under load even though every feature still works.

## Persistence and consistency

What a reimplementer needs is guarantees, not storage:

- What survives a crash mid-operation, and what a partially-completed operation looks like from
  outside. Describe atomicity as *observable* atomicity: "either the order and all its line items
  exist, or neither does".
- Whether a write is immediately readable by the same actor, and by others.
- What is retained, for how long, and what deletion actually means — soft delete that stays visible
  to admins is behavior, not implementation.
- Ordering guarantees on anything a user can observe as a sequence.

## Configuration

Every key that changes behavior goes in, with default, allowed values, precedence (env over file
over built-in, or whatever it is), and — most importantly — **what changes when it's absent or
wrong**. A system that silently falls back to a default is behaving differently from one that
refuses to start, and both are common.

Feature flags get their own treatment: for each flag, the behavior under each state and which state
is default. A flag whose "off" branch is dead code is `[incidental]`; say so rather than making the
reimplementer build both.

Never record a secret's value. Record that it exists, what it configures, and what fails without it.

## Migrations

Out by default: a migration history is the story of how the old implementation got here, and the
reimplementer starts from a clean schema.

Three exceptions go in:

1. **Data-repair logic** — a migration that backfilled or corrected values encodes a rule about what
   valid data means. That rule is contract.
2. **Live compatibility windows** — if the system still accepts an old format, still writes both
   old and new, or still reads a deprecated column, that dual behavior is current behavior.
3. **Data that must be carried over** — if the rebuild inherits the existing data, the document must
   say what shape that data is in, including its historical inconsistencies.

## Behavior that comes from the stack

These are the silent killers: real, observable, and invisible to anyone reading a feature list. A
reimplementer on a different stack reproduces them **only if told**. Check each one against the
system; where it's load-bearing, state it explicitly as `[contract]`; where it genuinely doesn't
matter, mark it `[undefined]` so nobody wastes a week matching it.

- **Numeric behavior** — float vs decimal for money, rounding mode (half-up, half-even), where
  rounding happens in a chain of operations, precision of stored values, integer overflow, division
  semantics. Any system handling money must have this stated outright.
- **String comparison and sorting** — case sensitivity in identifiers and lookups, collation and
  locale-dependent sort order, Unicode normalization (`é` as one codepoint or two), whitespace
  trimming, what "empty" means.
- **Dates and times** — the storage timezone, what "today" means for a user in another zone, DST
  handling at boundaries, week and month boundaries, whether timestamps are inclusive or exclusive
  in ranges, and the format of anything a user or client sees.
- **Ordering of unordered things** — a query with no `ORDER BY` returns rows in an order that is
  stable in practice and undefined in theory. If users depend on it, it's contract and must be
  written as an explicit sort; if not, mark it `[undefined]`.
- **Regex dialects** — greediness, anchoring, Unicode classes, and lookbehind differ between
  languages. Any user-visible pattern (validation, search, routing) needs its intent stated in
  words, not just the pattern.
- **Encoding and size limits** — character encoding of inputs and outputs, byte vs character
  limits on fields, what happens to input that exceeds them (truncate, reject, error).
- **Concurrency** — what happens when the same entity is modified twice at once. Last-write-wins,
  optimistic locking with a visible conflict error, or a lock the user can feel as a delay: these
  are three different products.
- **Null and empty semantics** — whether null, empty string, and absent are distinguished, and what
  each means in a filter.

## Two rules that override everything above

**Anything a consumer already depends on is contract, however wrong it looks.** A bug that clients
have built around is the interface. Record it as `[contract] [suspect]`, describe it exactly, and
say plainly that it appears unintended — then let the human decide whether the rebuild is allowed
to fix it. Deciding that yourself, silently, breaks their integration on migration day.

**Silence reads as contract.** Anything you leave unmarked will be reproduced. That's why
`[incidental]` and `[undefined]` have to be stated out loud: telling the reimplementer what they're
*free* to change is as valuable as telling them what they must match, and it is the part every
other specification omits.
