# Repo-type playbooks

Come here from the Orientation stage of SKILL.md. Classify the repo using the signals below, then
follow **one** playbook — the dominant type — rather than reading all of them. If the repo is a mix
(a backend service plus its Terraform, a library that ships a CLI), name the pieces to the human and
start with the dominant one; the rest become later `jump to` targets.

A playbook adapts the ladder — it never replaces the core loop, the evidence tags, or the artifacts.
**Ladder emphasis** says where the depth should go and what "key flows" means for this kind of system.
**The KT must answer** lists the questions a newcomer to *this kind* of system always ends up asking;
treat them as the exit criteria for a good KT. Unanswered ones go into `00-progress.md` as `[unknown]`.

## Contents

1. Backend service / API
2. Microservices / multi-service repo
3. Frontend / web app
4. Service-oriented monolith
5. Library / SDK
6. CLI tool
7. Data / ETL pipeline
8. ML system
9. Mobile app
10. Infrastructure-as-code
11. Embedded / firmware
12. Smart contracts
13. Notebook-heavy data science
14. Generic fallback (nothing fits)

---

## 1. Backend service / API

**Recognize it:** one deployable with a server entry point (`cmd/server`, `app.py`, `main.ts`);
route/controller registration; a `Dockerfile`; DB migrations; framework markers (FastAPI, Express,
Spring, Gin, Rails API, Phoenix).

**Ladder emphasis:** Key flows are the heart — one request traced route → middleware → handler →
domain logic → data store teaches more than any diagram. Dependencies next (DB, cache, queues,
third-party APIs). Architecture can stay brief when the framework is conventional.

**Read first:** the route registration file (the system's table of contents), the middleware chain
(auth, tenancy, rate limits live there), the DB schema or migrations directory, config loading,
external service clients, and one background job or consumer if any exists.

**The KT must answer:**
- Which 3–5 endpoints matter most (by traffic or business weight), and what does each touch?
- Where does state live — which store is the source of truth for what?
- What external services does it call, and what happens when each one is down?
- How is a request authenticated and authorized, and where could that be bypassed?
- Is there a second entry point (queue consumer, cron, admin surface) hiding beside the HTTP one?

**Traps:** don't inventory every endpoint — trace one read path and one write path deeply instead.
Background workers are usually the least documented and most fragile part. Middleware order is
behavior; read it as code, not boilerplate.

## 2. Microservices / multi-service repo

**Recognize it:** several services each with its own manifest and Dockerfile; a `docker-compose.yml`
or k8s manifests naming many services; a shared `proto/` or OpenAPI contracts directory; an API
gateway or service-mesh config.

**Ladder emphasis:** Architecture dominates — the service map, ownership, and contracts are worth
more than any single service's internals. Insert an explicit scoping turn after Orientation: "here's
the map; which service do we KT first?" Then run the backend-service playbook on that slice.

**Read first:** compose/k8s manifests (the *real* service registry — trust them over folder names),
the shared contracts (protos/OpenAPI), the gateway's routing table, broker topic and queue configs,
any service-ownership or dependency docs.

**The KT must answer:**
- Which services are core and which peripheral, and who owns each?
- What's synchronous (RPC) vs asynchronous (events, queues), and where are those contracts defined?
- For each core entity, which service is its source of truth?
- To ship a typical feature, which services have to change together? (The practical coupling question.)

**Traps:** repo layout ≠ runtime topology — deployment manifests are the truth. Don't KT every
service: map, scope, go deep on one. Shared internal libraries between services are hidden coupling
worth naming in `05-dependencies.md`.

## 3. Frontend / web app

**Recognize it:** `package.json` with React/Vue/Svelte/Angular; `src/components` or framework `app/`
routes; a bundler or meta-framework config (`vite.config`, `next.config`, `nuxt.config`); `public/`
assets.

**Ladder emphasis:** Architecture = routing + state management + data-fetching layer; that trio is
the skeleton. Domain lives in the state shapes and the API client. Key flow: one user interaction
traced event → state change → re-render → network call. Operations = build, env config, and where it
runs (SSR, static, CDN).

**Read first:** the router (what screens exist), the store setup (Redux/Zustand/Pinia/context), the
API layer (fetch wrappers, generated clients, query hooks), one representative feature folder, how
the design system is consumed.

**The KT must answer:**
- SPA, SSR, or hybrid — and what does that mean for where code executes?
- Where does server state live vs UI state, and what's the caching/invalidation story?
- How do loading and error states reach the user?
- What's the component organization convention — where does a new feature go?

**Traps:** the component tree is not the architecture; the state and data-fetch layers are.
Generated code (route types, API clients) is the product of a contract, not something to read
line-by-line. Storybook stories and component tests reveal intended behavior fastest.

## 4. Service-oriented monolith

**Recognize it:** one deployable containing many internal modules (`modules/`, `apps/`, `packages/`
under a single runtime); a DI container or app registry wiring them together; Django/Rails/Spring/
Laravel at scale; usually one shared database.

**Ladder emphasis:** Domain and Dependencies dominate. The interesting questions are internal
boundaries: which modules exist, who may import whom, and whether that's enforced or aspirational.
Blast radius is the critical stage — in a monolith, "what breaks if I change X" is the whole game.

**Read first:** the module registry or DI wiring (installed apps, container config), the shared
kernel (`core/`, `common/` — often the load-bearing wall), the DB schema with an eye for tables
touched by multiple modules, any boundary-enforcement tooling (import linters, module rules).

**The KT must answer:**
- What are the real module boundaries, and are they enforced by tooling or only by convention?
- Which database tables are shared across modules? (That's the true coupling map.)
- Where do cross-module calls happen — interfaces, events, or direct imports?
- Which module is riskiest to touch, and which is safest for a first change?

**Traps:** directory names advertise an architecture the imports may not respect — spot-check actual
import edges for a couple of modules. The shared DB is usually the hidden coupling. God-modules
(`utils`, `helpers`) deserve an explicit blast-radius warning.

## 5. Library / SDK

**Recognize it:** no server or long-running entry point; an explicit public export surface
(`index.ts`, `__init__.py`, `lib.rs`); registry publishing metadata (npm/PyPI/crates fields);
an `examples/` directory; a semver CHANGELOG.

**Ladder emphasis:** The public API surface *is* the architecture — map it first. Key flows become
the two or three canonical usage patterns, best learned from examples and tests. Operations shrinks
to the release process (versioning, publishing, deprecation). Safe contribution weighs heavily:
backward compatibility is the prime directive.

**Read first:** the public export surface, `examples/`, the tests (they are the spec of intended
use), CHANGELOG and deprecation patterns, the docs build, the CI release workflow.

**The KT must answer:**
- What exactly is public vs internal — and is anything internal leaking into the public surface?
- What compatibility promise does the versioning make, and what's the deprecation path?
- Who consumes this, and which usage patterns do the examples bless?
- How does a change get released, and what would count as a breaking change here?

**Traps:** read tests and examples first, not last — they beat source for intent. Consumers may
depend on accidentally-public internals. A breaking change here has a blast radius invisible from
inside the repo; say so explicitly.

## 6. CLI tool

**Recognize it:** an argument-parsing framework (Cobra, Click, argparse, clap, commander); `bin`
entries in the manifest; subcommand directories; man pages or `--help` text in the source.

**Ladder emphasis:** Key flows = the two or three most-used subcommands traced end to end
(parse → validate → do the thing → exit code). Domain = the config format, flags, and any state that
persists between runs. Operations = distribution (how users install and update it) more than deployment.

**Read first:** the command registration tree (the CLI's sitemap), config loading and its precedence
(flags vs env vars vs dotfiles), error handling and exit codes, anything that writes to disk or
network, shell-completion definitions.

**The KT must answer:**
- What are the primary subcommands, and what does each actually mutate — files, network, remote state?
- What configuration precedence applies: flag over env over file over default?
- What persists between runs (dotfiles, caches, stored credentials), and where does it live?
- What's the stdout contract — human-readable text, or parseable output that scripts depend on?

**Traps:** hidden state in `~/.config`, env vars, and keychains drives "works on my machine"
mysteries. Exit codes and output formats are API — changing them breaks users' scripts. Side effects
define blast radius more than code structure does.

## 7. Data / ETL pipeline

**Recognize it:** DAG or model definitions (Airflow `dags/`, dbt `models/`, Dagster/Prefect flows);
a SQL-heavy codebase; warehouse and connection configs; scheduling and backfill scripts.

**Ladder emphasis:** Key flows = **data lineage**, not code paths: trace one critical dataset from
source → transforms → sink. Domain = the datasets and tables themselves and what each column means.
Operations = scheduling, retries, idempotency, and backfills — where pipelines actually hurt.
Architecture is often just "the orchestrator plus the warehouse."

**Read first:** the DAG or model dependency graph, source definitions and sink configs, schema
files, data-quality tests (dbt tests, Great Expectations), the backfill story, alerting config.

**The KT must answer:**
- Which datasets are business-critical, what freshness do they promise, and who consumes them?
- Is each step idempotent — can it rerun safely? What actually happens on partial failure?
- How does a backfill work here, and which one is most expensive to get wrong?
- When data is late or wrong, where do you look first?

**Traps:** the code is thin — the *data* is the system; trace tables, not functions. Repo schemas
can lag the warehouse; claims about live data from code alone are `[inference]`. Transforms with
silent NULL handling and no tests are the classic fragility — flag them.

## 8. ML system

**Recognize it:** training scripts plus a config system (Hydra, YAML sweeps); an `experiments/`
directory or notebooks; feature-engineering code; model registry or artifact-store references; an
inference server living beside training code; an eval harness.

**Ladder emphasis:** Treat it as **two systems sharing a repo** — the training pipeline and the
inference path. KT them separately and map the seam: feature contracts and model artifacts. Key
flows: (a) how a model gets trained and blessed, (b) how a request gets features and a prediction.
Operations = model deploy, rollback, and drift monitoring.

**Read first:** the training entry point and its config tree, feature definitions (offline and
online versions), the eval code (it is the real definition of "good"), the serving code, experiment
tracking config (W&B, MLflow), data versioning.

**The KT must answer:**
- What's the path from "training run" to "model in production," and what approves it?
- What's the feature contract between offline training and online serving — and where could they skew?
- How is model quality measured, and what triggers a rollback?
- How reproducible is a training run — seeds, data snapshots, environment?

**Traps:** training/serving skew hides in duplicated feature code — diff the two implementations.
Notebooks may be stale ancestors of the real pipeline. Read the eval harness deeply: a system
optimizing the wrong metric looks healthy right up until it isn't.

## 9. Mobile app

**Recognize it:** `ios/` and `android/` directories; `.xcodeproj` or `Package.swift`; Gradle files;
Flutter or React Native manifests; app-store metadata (fastlane); signing configuration.

**Ladder emphasis:** Architecture = navigation structure + state + the platform boundary (if
cross-platform). Key flow: cold launch → first meaningful screen (auth, data fetch, cache).
Operations is uniquely heavy — build variants, signing, release trains, staged rollouts — because
you can't hotfix a binary users already installed.

**Read first:** navigation/routing setup, the app's composition root (DI, App/Application classes),
the API layer, local persistence (CoreData, Room, SQLite) and its migration story, platform-channel
or native-module code, CI signing and release lanes, feature-flag wiring.

**The KT must answer:**
- What OS versions and devices are supported, and what does that constrain?
- How does a release actually ship — trains, store review time, staged rollout, kill switches?
- What works offline, and how do sync and conflict resolution behave?
- Which features hide behind flags or remote config, and who flips them?

**Traps:** release mechanics are a first-class subsystem, not an afterthought — a bad build can be
unrecoverable for days. Local DB migrations outlive every refactor; treat them with extreme caution.
Cross-platform repos hide the scary parts in the native folders.

## 10. Infrastructure-as-code

**Recognize it:** `.tf` files; Pulumi, CDK, or CloudFormation; Helm charts and Kustomize overlays;
Ansible playbooks; a state-backend config; per-environment variable files.

**Ladder emphasis:** Domain = the environments and the real cloud resources they own. **Blast radius
is the whole KT** — an apply can take down production. Key flow: change → plan → review → apply, per
environment. Safe contribution = how to produce and read a plan without applying anything.

**Read first:** the state backend and workspace/environment layout, the module structure and what
each environment overrides, the CI plan/apply pipeline and its approval gates, IAM and access
definitions, secrets handling (tfvars? vault? SOPS?).

**The KT must answer:**
- What environments exist, how do they differ, and which definitions are shared between them?
- Where is state stored, who can apply, and what approval stands between a merge and production?
- Which resources are irreplaceable (stateful, data-bearing) vs safely recreatable?
- Is there drift — does the code still match reality, and how would we know?

**Traps:** **never run apply or anything mutating during KT — plan is the ceiling, and say so out
loud.** Code silently diverges from live infrastructure; a claim about "what's deployed" made from
code alone is `[inference]`, not `[fact]`. Watch for secrets committed in variable files.

## 11. Embedded / firmware

**Recognize it:** C/C++ with linker scripts and memory maps; HAL or BSP directories; RTOS config
(FreeRTOS, Zephyr) or a superloop `main`; cross-compilation toolchain files; register and peripheral
definitions.

**Ladder emphasis:** Orientation must pin the hardware target first — nothing else makes sense
without it. Architecture = the concurrency model (ISRs, tasks, priorities) and the memory map. Key
flows: boot → init → steady state, plus one interrupt-driven path. Operations = how a unit gets
flashed and debugged.

**Read first:** `main` and task/ISR creation, the interrupt vector table, the linker script (the
memory budget in writing), the HAL boundary (vendor code vs this team's code), the build system and
toolchain config, watchdog and fault handlers.

**The KT must answer:**
- What hardware does this run on, and what are the flash and RAM budgets?
- What's the concurrency model — which ISRs exist, what may they touch, how do they hand off to tasks?
- How does a developer flash and debug a physical unit?
- What happens on a fault — watchdog, brownout, hard-fault handler?

**Traps:** timing and memory constraints are invisible in source — a correct-looking change can miss
a deadline or overflow a stack. Vendor HAL code is noise; map the boundary and stay on this team's
side of it. Without the datasheet or schematic, hardware behavior is `[unknown]` — say so rather
than inferring from register names.

## 12. Smart contracts

**Recognize it:** `.sol` or `.vy` files; Foundry, Hardhat, or Truffle config; deployment scripts per
network; OpenZeppelin imports; an `audits/` directory; upgradeable-proxy patterns.

**Ladder emphasis:** Security *is* the KT. Dependencies = inheritance chains, external calls, and
trust assumptions (oracles, admin keys). Operations = the deployment/upgrade mechanism and key
custody. "Safe contribution" changes meaning entirely: deployed code may be immutable, and a small
mistake can be an unrecoverable loss of funds.

**Read first:** the contract inheritance graph, every external call and its trust assumption, access
control (owner, roles, timelocks), the upgrade mechanism (proxy? which pattern?), invariant and fuzz
tests, deployment scripts and network configs, past audit reports and whether findings were fixed.

**The KT must answer:**
- What is immutable vs upgradeable, and exactly which keys can upgrade or pause?
- What are the trust assumptions on external contracts, oracles, and privileged roles?
- What invariants must always hold, and are they encoded as tests?
- Does the deployed bytecode match this repo, and which audits cover *this* version?

**Traps:** the chain is the truth — repo HEAD may not be what's deployed; verify before claiming
`[fact]`. Audits age: a clean audit of v1 says little about v3. Never frame any change here as "low
risk" — the safe-first-change recommendation should be tests-only.

## 13. Notebook-heavy data science

**Recognize it:** many `.ipynb` files, often exploratively named (`final_v2_REAL.ipynb`); `data/`
directories; loosely pinned requirements; sometimes a small `src/` that notebooks import; outputs
committed alongside code.

**Ladder emphasis:** Orientation's first job is triage: separate **load-bearing** notebooks (things
downstream depends on) from exploration scratch. Key flows = which notebook produces which artifact,
in what order. Operations = reproducibility: the environment, data provenance, and whether the
critical path runs top-to-bottom today.

**Read first:** any README or run-order doc, the `src/` modules notebooks import (the stable core,
if one exists), each notebook's first cells (imports and data loads reveal dependencies), data-file
provenance, the environment spec.

**The KT must answer:**
- Which notebooks are load-bearing, which are scratch, and which are dead?
- What's the execution order, and can the critical path actually run end to end today?
- Where does the data come from, and is it versioned or a one-time snapshot?
- Is there a productionization path, or is this repo the terminal artifact?

**Traps:** committed cell outputs *lie* — they show what happened once, not what the code does now.
Treat outputs of unrunnable notebooks as `[unknown]`, not evidence. Hidden execution-order
dependencies are the classic failure; duplicated logic across notebook copies drifts silently.

## 14. Generic fallback (nothing fits)

When no playbook matches cleanly — a game, an editor plugin, a browser extension, a research
artifact — don't force a bad match. Run the standard ladder, but spend Orientation finding the four
things every system has:

- **Entry points** — where execution starts: binaries, exported handlers, lifecycle hooks.
- **State** — where data lives and what owns it.
- **Boundaries** — every place the system touches the outside world: network, disk, OS, host app.
- **The verification story** — tests, or whatever stands in for them.

Name the mismatch honestly to the human ("this is a Godot game; I'll adapt the ladder as we go"),
let those four anchors drive Key flows and Dependencies, and note in `00-progress.md` that the
generic path was used — so the next session knows the classification is soft.
