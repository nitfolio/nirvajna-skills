# Building the behavior index

Read this before Pass 1. The index is the spine of the whole run: it is the list of things that
might be behavior, it is the coverage denominator you report at the end, and it is what makes the
run resumable and honest about what it missed.

**Enumerate first, read second.** A system too large to hold in context can still be *listed* in
context. Listing is cheap, deep reading is expensive, and doing them in the wrong order is how a run
spends its budget on the first module it opened and never learns the other eleven exist.

## What an entry looks like

One line per candidate behavior in `01-behavior-index.md`:

```
| ID | Surface | Candidate behavior | Where | State | Obligation |
|----|---------|--------------------|-------|-------|------------|
| C-014 | HTTP | Cancel an order | api/orders.py:214 | specified | contract |
| C-015 | HTTP | Export orders as CSV | api/export.py:40 | open | — |
| C-016 | job | Nightly reconciliation | jobs/recon.py:1 | specified | contract |
| C-017 | HTTP | Debug echo endpoint | api/debug.py:9 | excluded | incidental |
```

Five states, and every entry ends the run in one of them: **open** (not yet read), **specified**
(written into the trail), **excluded** (out of scope or `[incidental]` — with the reason recorded),
**unknown** (read, and its behavior couldn't be determined), **deferred** (real, in scope, not
reached before the budget ran out).

*Open* is the only state that must be empty at `stop`. "We ran out of budget" is an acceptable
ending; an entry that quietly stayed unread is not, because the document will look complete.

IDs are permanent. Never renumber — an ID that appears in a delivered document and later means
something else is worse than no ID.

## Enumerate every surface, not every file

The index is organized by **surface** — the ways behavior can be triggered — because that's how a
reimplementer will approach it. Work this list; most systems have four or five of these.

- **HTTP / RPC** — every route, method, and handler. Include admin routes, health checks, webhook
  receivers, and anything mounted by a framework convention rather than declared.
- **UI** — every screen, and on each screen every control that does something: forms and their
  validation, buttons, filters, bulk actions, keyboard shortcuts, drag targets, empty states, error
  states, permission-dependent visibility.
- **CLI** — every command, subcommand, flag, and the interactions between them. Exit codes. What is
  read from stdin, written to stdout vs stderr, and what changes when not a TTY.
- **Public API of a library** — every exported symbol, its signature contract, what it throws.
- **Events in** — queue consumers, webhook handlers, file watchers, pub/sub subscriptions.
- **Events out** — published messages, webhooks sent, emails, notifications, file exports, anything
  written to a location another system reads.
- **Scheduled** — cron entries, timers, periodic tasks, retention sweeps, anything with a cadence.
- **Startup / shutdown** — migrations run on boot, seed behavior, warm-up, graceful-drain behavior.
- **Config-triggered** — flags and settings whose values switch behavior. Each distinct behavior is
  its own entry.
- **Data-triggered** — behavior driven by the content of records rather than by code paths (a
  status field that changes what's allowed, a rules table, a template system).

## Commands that find them fast

Adapt to the stack; the point is enumeration, not elegance.

```bash
# shape and size — decide scope before anything else
find . -type f -not -path '*/.git/*' | wc -l
tokei . 2>/dev/null || cloc . 2>/dev/null || true
ls -d */ && cat package.json go.mod pyproject.toml Cargo.toml pom.xml composer.json 2>/dev/null

# HTTP routes
rg -n "@(app|router|blueprint)\.(get|post|put|patch|delete)" --type py
rg -n "(app|router)\.(get|post|put|patch|delete)\(" --type js --type ts
rg -n "@(Get|Post|Put|Patch|Delete|RequestMapping|RestController)" --type java --type ts
rg -n "^\s*(get|post|put|patch|delete|resources|namespace)\s" config/routes.rb
rg -n "http\.HandleFunc|mux\.(Handle|HandleFunc)|r\.(GET|POST|PUT|DELETE)" --type go
rg -n "path\(|re_path\(|url\(" --type py            # Django urls
find . -name "*.openapi.*" -o -name "openapi*.y*ml" -o -name "swagger*" -o -name "*.proto"

# CLI surface
rg -n "add_argument|argparse|click\.(command|option)|cobra\.Command|yargs|commander|clap::"

# jobs, schedules, consumers
rg -n "@(shared_task|task|scheduled|Scheduled|Cron)|celery|sidekiq|cron|schedule\.|@every"
find . -name "crontab*" -o -name "*.cron" -o -name "Procfile" -o -name "*.timer"
rg -n "consume|subscribe|@KafkaListener|SQS|on_message|addEventListener"

# data shape and rules
find . -path '*migration*' -o -path '*migrate*' | head -50
rg -n "CREATE TABLE|ALTER TABLE" --type sql | head -50
rg -n "class .*\((Base|Model|db\.Model)\)|@Entity|type .* struct|schema\.define"

# permissions, validation, errors, flags
rg -n "permission|can_|authorize|@PreAuthorize|policy|role|is_admin|require_"
rg -n "validate|validator|constraint|@NotNull|zod|joi|pydantic"
rg -n "raise |throw new |errors\.New|panic\(" | head -100
rg -n "feature_flag|isEnabled|LaunchDarkly|unleash|FLAG_|toggle"

# the config surface, complete
rg -no "(os\.environ|process\.env|getenv|ENV\[)[\.\[]?['\"]?[A-Z_]+" | sort -u
cat .env.example .env.sample config/*.example 2>/dev/null

# tests — the intent record, and a free index of edge cases
find . -path '*test*' -name "*.*" | head -80
rg -n "^\s*(def test_|it\(|test\(|func Test|describe\()" | head -200
```

**Test names are the single highest-yield source in this pass.** A suite with 400 test names is 400
statements about intended behavior, already written in behavioral language, and the ones describing
failure modes are the edge cases you would otherwise have to find by reading every branch. Harvest
them into the index directly.

Also read, in this order, before deep reading anything: `README`, `CHANGELOG`, `docs/`, any `.env`
example, and the last 100 commit subjects (`git log --oneline -100`). Changelogs are behavioral
history — deprecations, defaults that changed, bugs that were fixed on purpose.

## False positives that inflate an index

Prune these before Pass 2, or the coverage number becomes a lie and the deep pass wastes its budget:

- **Generated, vendored, and build output** — `node_modules/`, `vendor/`, `dist/`, `*.pb.go`,
  generated clients, lockfiles. Never source of truth.
- **Framework scaffolding never customized** — a default health check, an untouched generator
  output, boilerplate error pages. Check whether it was actually modified before indexing it.
- **Dead surfaces** — routes with no reachable link, commands nobody calls, flags whose branch has
  been off for three years. Index them, but mark `excluded / [incidental]` with a note. Distinguish
  "dead" from "rarely used"; git history and any usage telemetry in the repo help.
- **Debug and development-only surfaces** — index them and exclude them explicitly, because a
  reimplementer needs to know they weren't overlooked.
- **Internal helpers that aren't behavior** — one entry per user-triggerable behavior, not per
  function. If three routes are three views of one rule, that's one capability with three triggers.
- **Deprecated-but-live** — anything marked deprecated that still responds is still contract.
  Include it, marked as deprecated, with whatever sunset the code implies.

## Sizing the index against the budget

Before Pass 2, do the arithmetic out loud:

- Count the open entries. Estimate 15–40 lines of finished document per capability entry, more for
  anything with a real edge-case surface.
- If the estimate exceeds roughly 3,000 lines, **scope down rather than compress**. Say the number,
  propose the narrower target, and let the human choose. A complete document for one bounded context
  is worth more than a thin one for six.
- If the human wants the whole thing anyway, work the index in **rank order** — by how much
  behavior an entry carries and how badly a rebuild would suffer without it — so the budget runs out
  on the least important entries rather than on whatever was alphabetically last. Everything unreached
  ends as `deferred`, named in the document.

Rank roughly by: money and data-destroying operations first, then core user workflows, then
permissions and auth, then integrations and jobs, then reporting and admin, then everything else.

## What the index is not

It isn't an architecture map. Resist recording module structure, dependency graphs, or call
hierarchies — that's `onboard-me`'s job and it doesn't survive reimplementation. If the run is
turning into a tour of the codebase, the index has drifted from surfaces to files, and the document
will be a port guide.
