# Synthesizing `02-handover.md`

Read this when the human says `stop`. It turns the capture trail and the register into the one
document a successor actually reads.

## 0. The inversion that governs everything here

A document that explains a system puts its unknowns near the end, because the point of it is what
*is* understood.

**This document is the other way round. What nobody knows goes first.** A successor inheriting a
system needs the landmines before the tour: the thing nobody could explain, the place where the last
expert and the code disagree, the deadline with no owner. Every other handover artifact in existence
buries these or omits them, which is exactly why they get discovered at 3am. Lead with them.

## 1. Check coverage — but expect it to be partial

Thin coverage here is **normal, not a failure**. This skill runs against a non-renewable
deadline, and reaching `stop` with open items is the expected ending. Do not refuse to
synthesize because the register isn't empty; state the coverage plainly at the top of the document and
carry on.

There is one floor. If **no human answers were captured at all** — a scan-only run, or the "already
left" mode — then what you have is a question list, not a handover, and it must say so in its title
and opening line. A register of unanswered questions formatted like a handover document will be
believed by someone who wasn't in the room.

Report coverage as three numbers, taken from the register: how many items were answered, how many were
deliberately deferred, and how many are unrecoverable. Those three numbers are the most honest summary
of the session that exists.

## 2. Synthesize through a confidence filter

Build the document *from* `00-risk-register.md` and `01-tribal-knowledge.md`, not from memory:

- **Promote** `[human]` and `[fact]` into the body as settled statements. `[human]` is the point of
  this skill — attribute it (*"per <name>"*) so a successor knows a person said it, not a scan.
- **Keep every hedge.** If they said "I *think* it was for the timezone bug", the document says that.
  Cleaning uncertainty into confidence is the one failure this whole method exists to prevent, and it
  is most tempting at synthesis time, when the prose wants to be tidy.
- **Carry** `[unknown]` into the opening section, not a footnote. Each with who was asked, so the next
  person knows the door was tried, not merely unopened.
- **Preserve** every `[conflict]` unresolved, both sides intact. Do not quietly pick the code or the
  human. If it stayed unresolved in the room, it stays unresolved on the page.
- **Drop** unconfirmed `[inference]` — your guesses about a system you spent an hour on are worth
  nothing next to the register. If an inference matters and was never put to the human, it belongs in
  "not covered", not in the body.

## 3. Suggested shape

```
# Handover — <system/area> · <departing name> · last day <date>
Coverage: N answered · N deferred · N unrecoverable · commit <hash>, <date>

1. Gone for good              ← [unknown]s. What nobody, including the author, could explain.
2. Where the author and the code disagree   ← [conflict]s, both sides, unresolved.
3. Live threads               ← in-flight work: state, deadline, and who owns it now (or "nobody").
4. Landmines                  ← each with: what it is, why it's there, what breaks if you change it.
5. Operating this             ← the runbook that was in their head. What pages, what it means, manual steps.
6. Why it's like this         ← decisions with rationale, and the "we already tried that" list.
7. People and access          ← named humans per external dependency; where keys live and who rotates them.
8. Not covered                ← deferred items and what was never scanned. Say it plainly.
```

Sections 1 and 2 go first even when they're short — especially when they're short, since a two-line
"gone for good" section is a two-line warning that would otherwise cost someone a week.

Section 6's failed-experiment list is the cheapest thing in the document to write and the most
expensive thing in it to rediscover. Don't let it get trimmed for length.

Keep the whole thing lean. The raw trail in `01-tribal-knowledge.md` is there for anyone who wants the full exchange.

## 4. Stamp the commit

Head the document with `git rev-parse --short HEAD` and the date. Every `file:line` is a line number
at that commit; say so. A citation that has drifted looks identical to one that was always wrong, and
the stamp is what tells them apart.

## 5. Offer the departing engineer the last word

Before the document is treated as final, offer it to them to read. This matters for two reasons, and
both are worth stating to the human.

It's accurate: they will catch the place where a hedge got hardened, or where you recorded the
intention rather than what shipped. And it's decent: the document quotes a named person, will outlive
their employment, and will be read by people who never met them. Someone who spent their last
afternoon helping their successor has earned a look at what it says.

If they decline or there's no time, note in the document that it wasn't reviewed by them.

## 6. What must not be in it

- **Secret values.** Locations, owners, and rotation paths only. If a credential was pasted into chat
  during the session, it does not enter the document — and say plainly that it should be rotated.
- **Anything about the departure itself.** Why they're leaving, their performance, their relationships,
  anything that reads as an exit interview. The scope was technical; the artifact stays technical.
- **Blame.** The register will surface undocumented, fragile, and strange code, much of it theirs.
  Describe the code and its risk, never the judgement. "This is uncommented and only they have touched
  it" is a fact about the system. "They left a mess" is an opinion about a person, it's unusable to a
  successor, and it makes every future run of this skill harder to get consent for.

## 7. Build the offline page (`handover.html`)

Once `02-handover.md` is written, always also produce `handover.html` in this session's own
`.kt/offboard/<slug>-<date>/` folder — a single self-contained page bundling the deliverable, the
register, and the capture, with a one-click copy button on every file. Build it from the template
rather than hand-rolling it:

1. Copy `references/handover-page-template.html` to `handover.html` inside this session's folder,
   alongside `00-risk-register.md`, `01-tribal-knowledge.md`, and `02-handover.md`.
2. It has three `<script type="text/markdown">` slots. Paste each file's **raw markdown verbatim** —
   `02-handover.md` fills the featured slot, `00-risk-register.md` and `01-tribal-knowledge.md` fill
   the trail slots. The blocks are inert, so nothing needs escaping (the one exception: literal
   `</script>` inside the markdown must be written `<\/script>`).
3. **Delete the slot for any file that doesn't exist** — a scan-only run has no capture trail, and an
   empty section must not render.
4. Set the page `<title>`, the `.kt-repo-name` span, and the `.kt-subtitle` span. Change nothing else —
   the CSS, renderer, nav, and copy buttons are complete in the template.

The page mirrors the three files exactly, so the secrets rule carries over untouched: those files are
already clean, so paste them as-is and never reintroduce a value while filling the template.

When you announce `stop` is done, name the outputs, give the three coverage numbers, and point at the
first section — the gone-for-good list is what the successor should read before anything else.
