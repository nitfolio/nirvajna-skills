# Synthesizing `rebuild.md`

Read this when the run reaches `stop`. It turns the trail into the one document a reimplementer
will build from — and the document is the only thing that crosses the gap, so this step is not a
formatting pass.

## 1. Coverage check, before writing anything

Take the numbers from `01-behavior-index.md`: **specified · excluded · deferred · unknown**, out of
the total. Those four numbers are the most honest summary of the run and they go in the document's
header, not in a footnote.

Two conditions stop you:

- **Any entry still `open`** — not read, not excluded, not deferred. Resolve them or move them to
  `deferred` with a reason. An unresolved `open` entry means the document claims coverage it doesn't
  have.
- **Specified coverage under roughly half** of the in-scope index — do not silently produce a
  polished document. Say what's covered and what isn't and offer three options: synthesize now with
  the gap stated prominently, keep working the index, or pause. Proceed only on a clear yes.

Thin coverage is survivable if it's *named*. A document that looks complete and isn't will be
believed by someone who wasn't here, and they'll discover the gap by shipping.

## 2. Filter twice: confidence, then obligation

Build from the trail, not from memory.

**Confidence filter.**
- **Promote** `[fact]` and `[human]` into the body as settled rules.
- **Promote a `[inference]` only when it describes behavior you traced**, never when it describes
  intent you reconstructed. Behavior inferred from an unambiguous code path is fine; a purpose
  inferred from a variable name is not.
- **Carry** every `[unknown]` and every unpromoted `[inference]` into *Coverage & open questions*,
  each naming what would settle it.
- **Preserve** every `[conflict]` unresolved, both sides intact, in *Suspect behaviors*. A
  documented disagreement between a system and its own tests is worth more than a confident guess.
- **Drop** dead-end explorations and anything about how the code is organized.

**Obligation filter.** Every statement in the body carries `[contract]`, `[incidental]`,
`[suspect]`, or `[undefined]` — explicitly, at capability level or on the individual rule. Unmarked
means contract, which is a safe default and a wasteful one, so mark the exceptions deliberately.
Telling the reimplementer what they're *free* to change is half the document's value and the half
every other spec omits.

## 3. Strip the implementation on the way out

The trail is full of citations, file paths, class names, framework names. **None of them survive
into `rebuild.md`.** Before finalizing, sweep for and remove:

- `file:line` references, function, class, module and table names
- framework, library, vendor and language names — except where the vendor *is* the contract (a
  payment provider whose behavior the rebuild must match, a database whose schema is declared
  `exact`)
- directory structure, layering vocabulary, and anything describing how code is organized
- "the service", "the handler", "the model" as nouns — name the *behavior*, not the component

The test for a surviving proper noun: would the reimplementer's system be observably different if
they used something else? If no, it doesn't belong.

Where a name has to stay because it's a boundary — a database table another team reads, a wire
field, a URL path — keep it and say why it's fixed. A reader must be able to tell a constraint from
a leak.

## 4. The section spine

Fixed. Every section renders even when it's empty, because "none found" is information a
reimplementer can act on and an absent section is one they can't distinguish from an oversight. The
interior of section 5 adapts to the system type; the spine does not.

```
# <System> — rebuild specification
Coverage: N specified · N excluded · N deferred · N unknown (of N)
Source snapshot: commit <hash>, <date>. Describes behavior as of that snapshot.

1.  Purpose                     ← what this software is for, in a paragraph someone can repeat back.
2.  Boundaries & compatibility  ← the observers and their fidelity levels. Read this before anything.
3.  Actors & permissions        ← every actor; the full actor × capability matrix, denials included.
4.  Domain model & invariants   ← entities, identity, lifecycles with legal transitions, derived values.
5.  Capabilities                ← the bulk. One entry per capability, uniform shape, stable IDs.
6.  State & persistence         ← durability, consistency, observable atomicity, retention, deletion.
7.  Integrations                ← each external dependency: contract, failure behavior, retries.
8.  Background & scheduled      ← triggers, cadence, guarantees, idempotency, catch-up behavior.
9.  Configuration surface       ← every behavior-changing key, default, precedence, absence behavior.
10. Errors & edge cases         ← user-visible errors, codes, retryability; cross-cutting edge rules.
11. Non-functional contract     ← limits, caps, timeouts, concurrency, locale, retention, compliance.
12. Suspect behaviors           ← [suspect] and [conflict]. What happens, why it looks wrong, who sees it.
13. Coverage & open questions   ← excluded, deferred, unknown. Questions for a human. Named plainly.
Appendix A. Acceptance checklist
```

Section 2 goes near the top because it governs how every later section is read. Sections 12 and 13
go at the end but must not be trimmed for length: they are the two sections that tell a reader how
far to trust the other eleven.

## 5. The capability entry

Uniform shape, same field order every time. A reader hunting one rule in a long document needs
predictability more than they need prose.

```
### C-014 · Cancel an order  ·  [contract]

**Actor:** who may trigger it.
**Trigger:** what starts it.
**Preconditions:** what must be true. What happens when each isn't — with the error.
**Rules:** the logic, as executable statements. Numbers, thresholds, formulas, rounding.
**Effects:** every state change and side effect, including asynchronous ones.
**Outputs:** what the actor gets back, and what anything else observes.
**Edge cases:** concurrency, repetition, partial failure, boundary values, empty and maximum inputs.
**Undefined:** what the reimplementer may choose freely. [undefined]
```

Rules that matter:

- **Every rule executable.** "Validates the input" is not a rule. "Rejects a quantity below 1 or
  above the remaining stock, with E-12, without reserving anything" is.
- **Concrete values, always.** 30 minutes, 10%, 100 items, half-up to two decimals. A category is
  not a specification.
- **Failure paths get equal weight to the happy path.** They're the reason the document exists.
- **Omit any field that genuinely doesn't apply** — but never omit *Edge cases* by default; if there
  are none, write "none identified", because that's a claim and its absence isn't.
- **Cross-reference by ID** (`see C-021`), never by section number or page.

## 6. The self-audit — the acceptance test's stand-in

The real acceptance test can't be run in this session: nobody is going to rebuild the system while
you wait. So run its proxy, and run it *before* declaring the document finished.

**Pass A — reconstruction questions.** Reread the document as if the source didn't exist. At every
capability, write down every question you would have to answer to actually write code — a value you'd
have to invent, an ordering you'd have to guess, an error you'd have to design. Each question is
either a hole to fill or a deliberate `[undefined]` to mark. This pass finds more real gaps than any
other, because it swaps you from the author's chair into the reader's.

**Pass B — branch spot-check.** Pick a dozen behavior-bearing branches from the source at random —
conditionals in validation, error paths, permission checks, retry logic. For each, confirm it is
either stated in the document or explicitly excluded. A miss rate above roughly one in six means the
deep pass was too shallow; go back to Stage 2 rather than shipping.

**Pass C — the leak sweep.** Section 3 above, mechanically. `rg -n '\.(py|ts|go|java|rb):[0-9]'` over
the document catches the obvious ones; the subtle leaks are framework nouns and layering vocabulary.

**Pass D — the numbers.** Every threshold, timeout, limit and percentage in the document traces to a
trail citation. A confidently stated number nobody verified is the most damaging single failure mode
here, because it will be implemented exactly.

Report the audit in one line when announcing `stop`: questions raised and resolved, spot-check miss
rate, leaks removed.

## 7. Appendix A: the acceptance checklist

Close the document with a flat, checkable list — one line per `[contract]` behavior, phrased as a
test a reimplementation either passes or fails, each carrying its capability ID.

```
- [ ] C-014 · Cancelling a `paid` order within 30 minutes refunds 100% of the captured amount.
- [ ] C-014 · Cancelling a `paid` order after 30 minutes deducts a 10% restocking fee, itemized.
- [ ] C-014 · Cancelling a `shipped` order fails with E-31 and changes no state.
- [ ] C-014 · Two concurrent cancels produce exactly one refund.
```

This is what makes the contract enforceable rather than aspirational: it converts the document into
an acceptance suite someone can work through, and it gives the reimplementer a way to report
coverage back in the document's own vocabulary. Derive it from the *Rules* and *Edge cases* fields —
if a rule can't be phrased as a checkable line, it wasn't finished in section 5.

Don't include `[incidental]` or `[undefined]` items. Do include `[suspect]` ones, flagged, so nobody
"fixes" one by accident.

## 8. Stamp the snapshot

Head the document with the commit (`git rev-parse --short HEAD`) and date, and say that it describes
behavior as of that snapshot. Unlike the trail, this isn't citation support — there are no citations
— it's provenance: it tells a reader whether the world has moved since, and gives them something to
diff against when they need to regenerate. No git: date only, and say so.

## 9. What must not be in it

- **Secret values.** Names, purposes, and absence-behavior only. If a credential appeared in chat
  during the run, it doesn't enter the document, and say plainly that it should be rotated.
- **Real data.** Fixtures and seed files carry real names, emails, and payloads. Invent every example.
- **Judgement about the people who wrote it.** Describe the behavior and its risk. "This appears
  unintended and any external caller can observe it" is a fact about the system; "this is a mess" is
  an opinion, is unusable to a reimplementer, and ages badly in a document that will outlive
  everyone's employment.
- **Architecture.** No module maps, no dependency graphs, no layering diagrams. If it's interesting
  but doesn't survive reimplementation, it belongs in an `onboard-me` document, not this one.

## 10. Announcing the finish

Name the output path, give the four coverage numbers, give the audit line, and point at the two
sections that decide how far the document can be trusted: *Suspect behaviors* and *Coverage & open
questions*. Then say what the document is not — a description of how the current system is built —
so nobody goes looking in it for that and concludes it's incomplete.
