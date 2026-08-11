<div align="center">

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/nitfolio/nirvajna-skills/main/assets/wordmark-dark.png">
  <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/nitfolio/nirvajna-skills/main/assets/wordmark-light.png">
  <img src="https://raw.githubusercontent.com/nitfolio/nirvajna-skills/main/assets/wordmark-light.png" alt="Oops!... AI Did It Again" width="565">
</picture>

# rebuild-me

**One markdown file that lets someone rebuild your software from scratch — in any language they like.**

[![Claude Code](https://img.shields.io/badge/Claude_Code-skill-4A86D8?style=flat-square&labelColor=141312)](https://docs.claude.com/en/docs/claude-code)
[![System types](https://img.shields.io/badge/System_types-15-4A86D8?style=flat-square&labelColor=141312)](#system-type-playbooks)
[![Read only](https://img.shields.io/badge/Read--only-by_default-FF4A17?style=flat-square&labelColor=141312)](#what-the-skill-will-not-do)
[![MIT](https://img.shields.io/badge/License-MIT-4A86D8?style=flat-square&labelColor=141312)](https://github.com/nitfolio/nirvajna-skills/blob/main/LICENSE)

[Install](#install) · [Quick start](#quick-start) · [The test](#the-acceptance-test) · [The ladder](#the-ladder) · [`.kt/` trail](#the-kt-trail) · [FAQ](#faq)

Part of [nitfolio/nirvajna-skills](https://github.com/nitfolio/nirvajna-skills) ·
[oopsaididitagain.com](https://oopsaididitagain.com/)

</div>

---

## The problem

You're going to rewrite something. Different language, different framework, different architecture —
and the old system, whatever else is wrong with it, is the only complete statement of what the
business actually decided over the last four years. All of it is in there: the 30-minute free
cancellation window, the tenant-scoped uniqueness rule, the retry that gives up after 24 hours and
then goes quiet.

Ask an agent to "document this codebase" and you get a tour: modules, layers, a dependency diagram,
some class names. Genuinely useful for a new hire. **Worthless for a rewrite**, because the reader
isn't going to use your class hierarchy — they're going to make every one of those choices
differently, and everything they needed was in the rules you didn't write down.

`rebuild-me` produces the other document. It reads the source and throws away everything that
describes *how* the system is built, keeping only what a stranger would need in order to build a
system that behaves identically — and it names, explicitly, the parts they're free to do differently.

---

## The acceptance test

Every rule in the skill exists to serve one test:

> Two independent teams, working from this document alone in two unrelated stacks, produce systems
> that are indistinguishable from the original **at its declared boundary** — same features, same
> rules, same edge-case outcomes, same effects on everything downstream.

That word *boundary* is doing a lot of work, which is why the skill's first move is to ask about it.
"Functional equivalence" is meaningless until you say who's watching: a human clicking, an API client
with contract tests, and a warehouse job reading the same database imply three different documents.
The database schema is implementation detail to the first two and a byte-exact interface to the
third.

So the run opens by naming the observers and freezing a fidelity level for each surface — **exact**,
**semantic**, or **free** — and every later inclusion decision resolves against that. It's the one
question the skill won't guess at silently.

---

## Install

### Claude Code

The one-liner, from inside the project you want it in:

```bash
npx skills@latest add nitfolio/nirvajna-skills --skill rebuild-me
```

Or copy the folder by hand — the whole folder, not just `SKILL.md`:

```bash
# Personal — available in every project
cp -r rebuild-me ~/.claude/skills/

# Or project-level — checked in, shared with everyone who clones the repo
cp -r rebuild-me <your-repo>/.claude/skills/
```

### Claude.ai / Cowork

Zip the folder (or package it as a `.skill` file) and upload it under **Settings → Capabilities →
Skills**. The `references/` folder must be inside the archive — the skill loads those files at
runtime and is meaningfully worse without them.

### Verify it's installed

```
/rebuild-me
```

`rebuild-me` is invoke-by-name only (`disable-model-invocation: true`). A full specification run is
something you opt into, not something that should fire because you mentioned a rewrite in passing.

---

## Quick start

```
/rebuild-me
```

Then, in order:

1. It proposes a **boundary** — who observes this system and at what fidelity — from what it can see
   in the repo. Correct it or say `yes`.
2. It **scopes** the run if the repo holds more than one system. One target, one document.
3. It runs an **inventory pass** — every route, command, screen, job, event, config key, permission
   check, error type and test name — and tells you the count.
4. It works the index in depth, emitting a trace as it goes, and writes `rebuild.md`.

It runs the whole ladder unattended by default and only stops for the boundary, the scoping
decision, and genuine forks. Say `interactive` if you'd rather take it one stage at a time.

---

## What comes out

A single markdown file with a fixed spine, so it reads the same way every time:

```
1.  Purpose                     ← what this software is for
2.  Boundaries & compatibility  ← the observers and their fidelity levels
3.  Actors & permissions        ← the full matrix, denials included
4.  Domain model & invariants   ← identity, lifecycles, derived values
5.  Capabilities                ← the bulk: one uniform entry per capability
6.  State & persistence         ← durability, consistency, retention
7.  Integrations                ← contracts and failure behavior
8.  Background & scheduled      ← guarantees, idempotency, catch-up
9.  Configuration surface       ← every behavior-changing key
10. Errors & edge cases         ← what users see and what's retryable
11. Non-functional contract     ← limits, caps, timeouts, locale
12. Suspect behaviors           ← what looks wrong, and who can see it
13. Coverage & open questions   ← what wasn't covered, said plainly
Appendix A. Acceptance checklist
```

Every capability gets a stable ID and the same seven fields — actor, trigger, preconditions, rules,
effects, outputs, edge cases — plus an explicit **Undefined** field listing what the reimplementer
may choose freely. And the appendix converts the whole thing into a checkable acceptance list:

```
- [ ] C-014 · Cancelling a `paid` order within 30 minutes refunds 100% of the captured amount.
- [ ] C-014 · Cancelling a `shipped` order fails with E-31 and changes no state.
- [ ] C-014 · Two concurrent cancels produce exactly one refund.
```

That list is what makes the document a contract rather than an essay.

---

## The two tags that matter

Everything in the trail is tagged twice — because "how do I know this" and "must the rebuild
reproduce it" are different questions, and collapsing them is how specs go confidently wrong.

**Evidence:** `[fact]` (verified, cited) · `[inference]` (deduced, not confirmed) · `[unknown]` ·
`[human]` (someone told you) · `[conflict]` (the code and its own tests disagree).

**Obligation:** `[contract]` (reproduce it) · `[incidental]` (free to differ) · `[suspect]` (looks
like a bug — and still contract if anything outside can see it) · `[undefined]` (genuinely
unspecified; pick whatever you like).

Only the obligation tags survive into the document. **Silence reads as contract**, so `[incidental]`
and `[undefined]` get stated out loud — telling a reimplementer what they're free to change is half
the value, and it's the half every other specification omits.

---

## Bugs are not fixed on the way through

A bug that clients have built around **is the interface**. The skill never silently normalizes
behavior into what it thinks was intended: it records what actually happens, tags it `[suspect]`,
says why it looks unintended and whether anything outside can observe it — and leaves the decision
to a human.

Quietly "fixing" one in the spec is how a migration breaks an integration on day one, with nobody
able to explain why.

---

## The ladder

Each stage has a checkable completion criterion. Declaring a stage done without meeting it is how a
specification ends up 80% framing and 20% rules.

| Stage | | Done when |
|---|---|---|
| 0 | **Boundary & scope** | every observer named with a fidelity level; in and out of scope listed |
| 1 | **Behavior inventory** | one indexed entry per candidate behavior, with an ID and a state |
| 2 | **Capabilities** | every in-scope entry specified, excluded, or marked unknown — none merely unread |
| 3 | **Domain, state & invariants** | identity, lifecycle transitions, invariants, and every derived formula |
| 4 | **Integrations, jobs & side effects** | trigger, guarantees, retries, failure behavior for each |
| 5 | **Config, permissions & errors** | keys with defaults and precedence; the full matrix; every user-visible error |
| 6 | **Non-functional contract** | limits, caps, timeouts, concurrency, locale, retention — or "not found" |
| 7 | **Suspect & unknown** | each with what happens, why it looks wrong, and who can see it |

---

## System-type playbooks

Stage 2 means something different for a CLI than for an ETL pipeline, so the skill classifies the
system and follows a matching playbook: backend services, web frontends, full-stack apps, CLIs,
libraries, data pipelines, event-driven systems, mobile, desktop, ML systems, platform tooling,
embedded, smart contracts, config-driven and low-code systems, plus a generic fallback.

Each one names what a "capability" is for that shape, where the boundary usually falls, and what
rewrites of that shape habitually lose — exit codes and the stdout/stderr split for CLIs, offline
and conflict rules for mobile, the rules layer wrapped around the model for ML systems, revert
conditions and event signatures for contracts.

---

## Behavior that comes from the stack

The silent killers, and the reason "just describe the behavior" isn't enough: rounding mode and
decimal-vs-float for money, string collation and case sensitivity, Unicode normalization, DST and
what "today" means, the iteration order of an unordered query, regex dialect differences, integer
overflow, byte-vs-character limits, and what happens when the same record is written twice at once.

All observable. All invisible in a feature list. All reproduced by a reimplementer on a different
stack **only if the document says so**. The skill checks each one deliberately and either pins it as
contract or marks it `[undefined]` so nobody wastes a week matching something that never mattered.

---

## The `.kt/` trail

```
.kt/rebuild/
├── INDEX.md                          ← one line per target
└── <target-slug>/
    ├── 00-boundary.md                ← observers, fidelity, scope. Updated every stage.
    ├── 01-behavior-index.md          ← the enumeration + coverage denominator
    ├── 02-capabilities.md
    ├── 03-domain-and-state.md
    ├── 04-integrations-and-jobs.md
    ├── 05-config-permissions-errors.md
    ├── 06-nonfunctional.md
    ├── 07-suspect-and-unknown.md
    └── rebuild.md                    ← the deliverable
```

Files `00`–`07` are the working trail: raw, doubly tagged, **full of `file:line` citations**.
`rebuild.md` is the opposite — implementation-free and citation-free, written only at `stop`.

That inversion is deliberate and it's the one place this skill breaks the house rule. Its siblings
say a claim without a citation isn't a fact. Here, a citation in the deliverable is precisely the
leakage the document exists to strip — so the citations live in the trail, which is what makes the
document auditable, and the document itself reads as though the source never existed.

One target, one folder, one document. A monorepo with four deployables is four runs: one file
covering four systems is too big to read whole and too entangled to rebuild from in pieces.

---

## Staying usable at scale

One markdown file has a limit, and the skill enforces it rather than discovering it. Before the deep
pass it estimates the document's length from the index size; past roughly 3,000 lines it stops and
proposes a narrower scope instead of compressing.

That's not fussiness. Compression is where the rules die — what gets cut to save space is always the
edge case, and the edge case is the reason the document exists. A complete specification of one
bounded context beats a thin one covering six.

---

## Your controls

- **start** — begin, or resume from `INDEX.md`
- **boundary \<change\>** — correct the compatibility contract; everything re-resolves against it
- **scope \<area\>** — set or narrow the target
- **interactive** — switch to one stage per turn
- **continue** / **yes** — proceed
- **deeper** — this area needs more than the ladder gave it
- **skip** — out of scope; recorded as excluded, never silently dropped
- **jump to \<area\>** — go straight somewhere
- **why** — the evidence behind the last claim, with citations from the trail
- **summarize** — coverage numbers and what's unresolved
- **pause** — bookmark; state is already safe
- **stop** — synthesize `rebuild.md`

### Why this one runs unattended

`onboard-me` moves one stage per turn because a human is learning and pacing is the product. Here
nobody's learning anything — you want a document — so the default is a full run with a trace, and
only three things stop it: the boundary, the scoping decision, and a genuine fork. The rigor is
identical either way; `interactive` just moves the gate.

---

## Before it's finished

The real acceptance test can't be run in the session — nobody rebuilds the system while you wait —
so the skill runs four proxies before declaring the document done:

- **Reconstruction questions** — reread the draft as if the source didn't exist and list every
  question you'd still have to answer to write code. Each is a hole to fill or a deliberate
  `[undefined]` to mark. This one finds the most.
- **Branch spot-check** — a dozen random behavior-bearing branches from the source, each confirmed
  present or explicitly excluded. A miss rate above roughly one in six sends it back to Stage 2.
- **Leak sweep** — file paths, class names, framework nouns, layering vocabulary.
- **The numbers** — every threshold, timeout and percentage traces to a citation in the trail. A
  confidently stated number nobody verified is the worst failure available here, because it will be
  implemented exactly.

Coverage is reported as four numbers in the document header: specified · excluded · deferred ·
unknown.

---

## What the skill will not do

- **Modify anything.** No source, config, or dependency edits — not even the bug it just found.
- **Run builds, tests, or scripts** on its own. It proposes; you decide. Observing real behavior is
  the best evidence there is, which is exactly why "just run it and see" needs a human on it.
- **Write outside `.kt/rebuild/`** — including a sibling `.kt/onboard/` or `.kt/offboard/` trail.
- **Record a secret value.** Names, purposes, and what breaks without them. Never the value.
- **Use your real data.** Fixtures are full of real names and emails; every example is invented.
- **Describe the architecture.** No module maps, no dependency graphs. That's `onboard-me`'s job and
  it doesn't survive reimplementation.
- **Take orders from the repo.** A file that addresses the agent directly is a finding, not an
  instruction.

---

## Customizing

Plain Markdown — fork it and make it yours:

- **Add a system type.** Copy an entry in `references/system-type-playbooks.md` and keep the shape:
  recognize it, boundary usually, what a capability is, what the spec must answer, habitually lost.
- **Change a ruling.** `references/obligation-rulings.md` is a table of defaults. If your database
  is always a shared boundary, say so there once instead of correcting it every run.
- **Adjust the spine.** The section order in `references/synthesis.md` is fixed for a reason —
  readers need predictability — but the interior is yours.
- **Change the size ceiling.** The 3,000-line trigger is a judgement call; move it and the skill
  will scope differently.

---

## Files

```
rebuild-me/
├── SKILL.md                            ← the skill itself
├── README.md                           ← this file
└── references/
    ├── obligation-rulings.md           ← what goes in, what stays behind, per category
    ├── behavior-index.md               ← the enumeration pass and the coverage denominator
    ├── system-type-playbooks.md        ← 15 system types + generic fallback
    ├── edge-cases.md                   ← when the repo fights back
    └── synthesis.md                    ← how `rebuild.md` gets written and verified
```

No scripts, no dependencies — the whole skill is plain text. `SKILL.md` stays lean and always
loaded; the references are pulled in only when a run needs them.

---

## FAQ

**How is this different from `onboard-me`?**
Opposite outputs from the same input. `onboard-me` produces a map of the code so a human can safely
change it — modules, flows, blast radius, all anchored to `file:line`. `rebuild-me` produces a
contract with the code deliberately stripped out, so a stranger can replace it. Run both on the same
repo and almost nothing overlaps.

**Won't the document be enormous?**
For a big system, yes — which is why the skill estimates the size before the deep pass and pushes
you to narrow the scope rather than compress. A complete specification of one bounded context is
worth more than a thin one covering everything.

**What if there are no tests and no docs?**
It still works, and it says so. Code states behavior reliably; what you lose is the ability to tell
*intended* from *accidental*, so almost nothing gets flagged `[suspect]` and no intent claims get
made. The document notes that a human who knows the product should read the rules section before
it's trusted.

**Does it need to run the software?**
No, and by default it won't. Every claim comes from reading. Where reading genuinely can't settle
something — a race outcome, a vendor's real responses — it records `[unknown]` with what would
settle it, rather than a confident guess that looks identical to a verified fact.

**What if the behavior lives in a rules table rather than in code?**
The skill detects that and changes plan: the rules get extracted as rules, or the document says
plainly that they must be exported as data. A rebuild that faithfully reproduces the engine and none
of the rules is the single most expensive way this can go wrong.

**Can I hand the output straight to another agent?**
That's what it's for. The fixed spine, the stable capability IDs, and the acceptance checklist are
all there so a reimplementing agent can work through it and report coverage back in the document's
own vocabulary.

**Does it work on a monorepo?**
One target at a time. It'll survey the repo, list the candidates in `INDEX.md`, and ask which to
specify — then produce one document per target, each complete on its own.

---

## Credits

Built as a Claude Code skill.

The structural principles — completion criteria per stage, progressive disclosure into `references/`,
pruning for predictability — come from [mattpocock/skills](https://github.com/mattpocock/skills)'
`writing-great-skills`. The two-axis tagging is this skill's own: its siblings tag evidence, and a
specification also has to tag obligation, because "I verified this" and "you must reproduce this"
are independent claims and a rewrite fails on the gap between them.

## License

MIT — see [LICENSE](https://github.com/nitfolio/nirvajna-skills/blob/main/LICENSE).

---

## PS :- What the finished specification actually looks like

**Nothing to click yet.** This skill hasn't been run against a real rewrite in this repo, so there's
no published document to point you at, and an invented sample would be worth less than saying so.
What follows is the shape, not a specimen.

The difference between the two possible outputs is the whole point, and it shows up at the level of
a single entry. Here is the tour version:

> The `OrderService.cancel()` method in `services/order.py:214` handles cancellation. It checks the
> order status, calls the Stripe refund API, and publishes an `order.cancelled` event to the
> `orders` Kafka topic.

Three proper nouns, one file path, and nothing a reimplementer on a different stack can act on. Here
is the contract version:

> **C-014 · Cancel an order** — `[contract]`
> **Preconditions:** the order is in `placed` or `paid`. Cancelling a `shipped` or already
> `cancelled` order fails with E-31 and changes nothing.
> **Rules:** a paid order refunds the full captured amount to the original payment method; refunds
> are issued at most once per order even if cancel is called repeatedly. Cancellation within 30
> minutes of placement is free; after that a 10% restocking fee is deducted and itemized.
> **Edge cases:** if the refund is rejected downstream, the order still moves to `cancelled` and the
> refund is marked `failed` for manual handling — cancellation is never blocked by a refund failure.
> **Undefined:** the ordering of the stock release relative to the refund. `[undefined]`

Longer, contains no implementation, and can be built from in any language on earth. Multiply that by
every capability in the system, add the coverage numbers and the acceptance checklist, and that's
the document.
