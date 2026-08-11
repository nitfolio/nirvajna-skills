# System-type playbooks

Come here from Stage 1, once the index tells you what kind of system this is. Follow **one**
playbook — the dominant type. If the system is genuinely a mix, name the pieces to the human and
scope to one; the rest become separate targets with their own folders and their own documents.

A playbook adapts the ladder. It never replaces the boundary declaration, the inclusion test, the
two tag axes, or the artifacts.

- **Boundary, usually** — where the compatibility line normally falls for this shape. A starting
  proposal for Stage 0, not a substitute for asking.
- **What a capability is here** — the unit of specification, so entries come out uniform.
- **The spec must answer** — the exit criteria. Unanswered ones become `[unknown]` in the document.
- **Habitually lost** — what rewrites of this shape actually drop. Check each one deliberately.

## Contents

1. Backend service / API · 2. Web frontend / SPA · 3. Full-stack web app · 4. CLI tool ·
5. Library / SDK · 6. Data / ETL pipeline · 7. Event-driven / worker system · 8. Mobile app ·
9. Desktop app · 10. ML system · 11. Infrastructure / platform tooling · 12. Embedded / firmware ·
13. Smart contracts · 14. Spreadsheet, low-code, or config-driven system · 15. Generic fallback

---

## 1. Backend service / API

**Recognize it:** one deployable, a server entry point, route registration, a datastore, a Dockerfile.

**Boundary, usually:** the API is `exact` if any client ships separately; the database is `exact`
only if something else reads it; internals `free`.

**A capability is:** one operation a caller can invoke, across all its outcomes — not one endpoint.
Three routes onto the same rule are one capability with three triggers.

**The spec must answer:** every operation with its full outcome set including failures · the
complete permission matrix · what is transactional as observed from outside · every background
effect a synchronous call kicks off · what each external dependency's outage does to each operation ·
pagination, filtering, and sorting semantics with their limits.

**Habitually lost:** middleware behavior read as boilerplate — rate limits, tenancy scoping, request
size caps, CORS rules, auth edge cases all live there and all are contract. Also: the second entry
point (a queue consumer or cron beside the HTTP surface) that nobody mentions.

## 2. Web frontend / SPA

**Recognize it:** a build tool, a component tree, a router, no server-side business logic.

**Boundary, usually:** the user's experience is `semantic`; the API it consumes is `exact` if the
backend isn't being rebuilt with it; URLs are `exact` if anything is bookmarked or linked.

**A capability is:** one thing a user can accomplish on a screen, with its states — empty, loading,
partial, error, permission-denied, success.

**The spec must answer:** every screen and what it shows for each actor · every form's validation
rules, client-side and where they differ from the server · what happens on network failure mid-action ·
optimistic updates and what a user sees when one is rolled back · URL and deep-link structure ·
what persists across reload and what doesn't · keyboard and accessibility commitments.

**Habitually lost:** client-side validation rules (assumed to be duplicated on the server —
frequently they aren't); loading and empty states; the exact URL structure; local storage that
survives reload; debounce and autosave timing that users have come to rely on.

## 3. Full-stack web app

**Recognize it:** server-rendered templates or a co-deployed frontend and backend; sessions;
one deployable.

**Boundary, usually:** everything internal is `free`, which is the freest case there is — the whole
contract is the user's experience plus persisted data.

**A capability is:** one complete user journey, end to end, including the redirects.

**The spec must answer:** every page and its access rules · every form and every validation
message · the full navigation and redirect graph, especially after login and after failure ·
what a session is worth and when it dies · flash messages and where state is carried between
requests · every email or notification the app sends and what triggers it.

**Habitually lost:** the redirect chain; error and success message wording (which users and support
docs depend on); email templates and their triggers; "remember me" semantics.

## 4. CLI tool

**Recognize it:** an argument parser, subcommands, exit codes, stdout/stderr discipline.

**Boundary, usually:** everything a script could observe is `exact` — flags, exit codes, stdout
format, file formats. This is the shape where the boundary is widest and least negotiable, because
the consumers are other people's scripts.

**A capability is:** one command, with its full flag matrix and every exit path.

**The spec must answer:** every command and flag, including short forms and deprecated aliases ·
every exit code and what produces it · exactly what goes to stdout vs stderr · behavior when piped
vs on a TTY (colour, progress, prompts) · config file locations and precedence against flags and
env vars · what happens on interruption mid-operation · idempotency of repeated runs.

**Habitually lost:** exit codes; the stdout/stderr split; non-TTY behavior; flag precedence; the
undocumented alias someone's CI depends on.

## 5. Library / SDK

**Recognize it:** no entry point, a published package, an exported public surface, versioning.

**Boundary, usually:** the entire public API is `exact` in semantics if not in syntax — every
caller is a consumer you cannot redeploy.

**A capability is:** one exported symbol's contract: inputs, outputs, errors, and side effects.

**The spec must answer:** every public symbol with its full contract · what it throws and when ·
thread-safety and reentrancy · resource ownership — who closes what · default values and how they're
overridden · backward-compatibility promises the version scheme implies · what is public by
declaration versus public by accident but depended upon.

**Habitually lost:** error types and the conditions that raise them; thread-safety guarantees;
the semver contract; the "internal" symbol everyone imports anyway.

## 6. Data / ETL pipeline

**Recognize it:** scheduled batch jobs, source and sink connectors, transformation steps, a warehouse.

**Boundary, usually:** the output schema and the semantics of every field are `exact` — dashboards
and models downstream break silently otherwise. The transformation is `free`.

**A capability is:** one transformation rule, or one output field's derivation.

**The spec must answer:** every output field's exact derivation including filters and joins ·
grain and uniqueness of each output · how late, duplicate, and malformed input is handled ·
backfill and reprocessing semantics, and whether a rerun is idempotent · watermarks, cutoffs, and
what "yesterday" means in which timezone · null and missing-value handling per field · row counts
or freshness checks that gate publication.

**Habitually lost:** the exact filter predicates ("excluding test accounts", "excluding refunded
orders") that make every number match; timezone cutoffs; how duplicates are resolved; the manual
correction someone applies each quarter.

## 7. Event-driven / worker system

**Recognize it:** queues, topics, consumers, retry configuration, dead-letter handling.

**Boundary, usually:** message formats are `exact` when producers or consumers outside this system
exist; effects are `semantic`.

**A capability is:** one message type's handling, from receipt to effect.

**The spec must answer:** every message type and its schema · delivery guarantee per consumer ·
ordering requirements and whether they're real · idempotency: what happens on redelivery, and how
duplicates are detected · retry policy, backoff, and what reaches the dead-letter queue · what a
poison message does · consumer concurrency and whether it's safe to raise · replay semantics.

**Habitually lost:** idempotency keys and dedupe windows; ordering assumptions never written down;
what the DLQ actually means operationally; catch-up behavior after downtime.

## 8. Mobile app

**Recognize it:** a platform project, lifecycle callbacks, an app-store artifact.

**Boundary, usually:** the backend API is `exact` because old app versions live in users' pockets;
local storage is `exact` if upgrades must preserve it.

**A capability is:** one user task, including its offline and interrupted states.

**The spec must answer:** offline behavior for every action — queued, blocked, or degraded · sync
and conflict resolution · what survives force-quit, backgrounding, and OS-initiated kills · push
notification triggers and what each does when tapped · permission prompts, when they appear, and
behavior when denied · deep links · minimum supported versions and what happens below them.

**Habitually lost:** offline queueing and conflict rules; background-refresh behavior; the
notification-tap routing table; permission-denied fallbacks.

## 9. Desktop app

**Recognize it:** a windowing framework, a local filesystem dependency, an installer or updater.

**Boundary, usually:** the file formats it reads and writes are `exact`; local state and settings
are `exact` across upgrades.

**A capability is:** one user action, including its undo behavior.

**The spec must answer:** every file format read and written, including versions and backward
compatibility · undo/redo scope and what isn't undoable · autosave and crash-recovery behavior ·
where settings live and what survives an upgrade or reinstall · multi-window and multi-instance
behavior · update mechanism and what a partial update leaves behind · OS integrations (tray,
file associations, shortcuts).

**Habitually lost:** the undo model; crash recovery; file-format backward compatibility; settings
migration.

## 10. ML system

**Recognize it:** training scripts, model artifacts, feature pipelines, an inference surface.

**Boundary, usually:** the inference contract is `semantic`; feature definitions are `exact` because
they must match between training and serving.

**A capability is:** one prediction surface, plus every deterministic rule wrapped around it.

**The spec must answer:** the exact inference input contract and its preprocessing · every
deterministic rule around the model — thresholds, overrides, blocklists, fallbacks, business rules
applied to the output · what happens when the model is unavailable or a confidence floor isn't met ·
feature definitions precisely enough to reproduce training/serving parity · retraining cadence and
what triggers it · acceptance metrics · fairness or compliance constraints applied to outputs.

**Habitually lost:** the rules layer around the model, which is usually where most of the actual
business logic lives; the fallback path; the exact preprocessing. Say plainly that model weights are
an artifact to be carried over or retrained, not something a document can specify.

## 11. Infrastructure / platform tooling

**Recognize it:** IaC modules, operators, deployment tooling, internal developer platforms.

**Boundary, usually:** the interface other teams consume — module inputs, CRDs, pipeline
contracts — is `exact`; the provisioned resources are `semantic`.

**A capability is:** one guarantee the platform makes to its users.

**The spec must answer:** every input and its validation · what is guaranteed to exist after a
successful apply · idempotency and what a re-run changes · what a failed or partial apply leaves
behind · rollback semantics · policy and guardrail rules that reject a configuration · defaults, and
what changes if a consumer doesn't override them.

**Habitually lost:** guardrail rules; partial-failure states; the defaults everyone silently depends
on.

## 12. Embedded / firmware

**Recognize it:** hardware register access, interrupt handlers, no OS or an RTOS, timing constraints.

**Boundary, usually:** every wire protocol and pin behavior is `exact`; timing is `exact` and is
first-class behavior rather than a non-functional footnote.

**A capability is:** one device behavior, with its timing envelope.

**The spec must answer:** every protocol and message format · real-time deadlines and what a miss
does · state machine including power states and transitions · what happens on brownout, watchdog
reset, and unexpected power loss · calibration and persisted configuration · error and fault
signalling · bootloader and update behavior including a failed update.

**Habitually lost:** timing constraints; recovery behavior; what persists across reset.

## 13. Smart contracts

**Recognize it:** on-chain languages, deployment addresses, an ABI.

**Boundary, usually:** everything is `exact`. The ABI, event signatures, and storage layout are all
consumed externally and immutably.

**A capability is:** one externally callable function with its full precondition and revert set.

**The spec must answer:** every external function, its access control, preconditions and every
revert condition · every emitted event and its exact signature · economic rules stated as formulas,
including rounding · upgrade and pause mechanics and who holds them · invariants that must hold
across every state transition · reentrancy expectations · gas-driven behavior a caller can observe.

**Habitually lost:** revert conditions; event signatures downstream indexers depend on; rounding
direction in economic formulas; the admin capability nobody documented.

## 14. Spreadsheet, low-code, or config-driven system

**Recognize it:** the behavior lives in data — rule tables, workflow definitions, templates,
formulas — rather than in code. Often the code is a thin engine.

**Boundary, usually:** the rules are the entire contract; the engine is `free`.

**A capability is:** one rule, expressed independently of the engine that runs it.

**The spec must answer:** every rule extracted and written as a rule, not as a reference to a
row or cell · evaluation order and precedence between rules · what happens when two rules conflict
or none match · the engine's own semantics that the rule authors relied on (recalculation order,
type coercion, error propagation) · who edits the rules and how often, because that decides whether
the rebuild needs an editing surface as well.

**Habitually lost:** essentially everything — this is the shape most likely to be "rebuilt" as a
faithful engine with none of the rules. Extract the rules exhaustively or say explicitly that they
must be exported as data.

## 15. Generic fallback

Nothing fits, or it's a mix nobody named. Do this:

Take the boundary declaration seriously and derive the rest from it — list every way the outside
world touches this system, and specify each one exhaustively as a capability with trigger,
preconditions, rules, effects, outputs, and edge cases. Then sweep for the four things every system
type above shares and every rewrite loses: **what happens on failure**, **what happens on repeat**,
**who is allowed to do it**, and **what is left behind afterwards**.

If you can't classify it, say so in the document rather than forcing it into a shape — and name the
surfaces you enumerated, so a reader can see the perimeter you actually walked.
