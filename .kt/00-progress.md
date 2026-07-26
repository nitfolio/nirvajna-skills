# KT progress

Goal: just understand it (default — not stated by the human; assumed at session start) · Repo type:
**content / prompt-artifact repo**, run via playbook 14 *generic fallback* — classification is
**soft**. Mode: `speedrun` (full ladder, no per-turn gates). Date: 2026-07-26.

Note: this is a **self-referential KT** — the repo under exploration contains the skill running the
exploration. The installed copy and the repo copy are byte-identical (`diff -r`, no differences).

## Stages

- [x] Orientation — `01-overview.md`
- [x] Architecture — `02-architecture.md`
- [x] Domain — `03-domain-glossary.md`
- [x] Key flows — `04-key-flows.md`
- [x] Dependencies & blast radius — `05-dependencies.md`
- [x] Operations — `06-operations.md`
- [x] Safe contribution — `07-safe-contribution.md`
- [x] Synthesis — `08-onboarding.md` + `onboarding.html`

Coverage note: all 12 tracked files were read or surveyed; nothing in the repo is unexplored. The
repo is 1,683 lines of text, so coverage here is genuinely complete rather than sampled.

## Open questions

- [unknown] Does the Claude.ai / Cowork upload path (read-only mount, no `.git/`, relocated `.kt/` —
  `SKILL.md:118-121`) actually work? The logic is written; no evidence in the repo that it was
  exercised. → Find out by uploading the zipped folder plus a zipped repo and running a session.
- [unknown] Is the missing mermaid ladder in `README.md` intentional-in-progress or a dropped edit?
  HEAD `6e83541` is a `wip:` commit that names it. → Ask the author.
- [unknown] Is `codebase-onboarding/README.md:62` ("Upload `codebase-onboarding.skill`") stale, or
  is a `.skill` bundle published somewhere outside the repo (a release asset)? → Check GitHub
  releases / ask the author.
- [unknown] Whether more skills are planned. The repo is named `nirvajna-skills` (plural) and the
  README has a table and contribution rules built for many, but ships one. → Ask the author.
- [inference, unverified] `study-page-template.html`'s renderer has never been run in this session —
  it was read, not executed. Its correctness on real trail markdown is inferred from the code, not
  demonstrated. (The `onboarding.html` produced by this run is the first chance to check it.)

## Next step

Ladder complete and both deliverables written. Nothing is pending. A useful follow-up would be the
Tier-1 fixes in `07-safe-contribution.md` (stale install line, mermaid gap) — but that is a change
task, not a KT step.
