# KT progress

Goal: regenerate the `.kt/` trail from scratch as the worked example of this skill's own output ·
Repo type: **Library / SDK** (playbook 5) — soft match, see note below
Mode: `speedrun` (full ladder, no per-turn gate) · Session date: 2026-07-27
Status: **complete** — all 7 stages met their completion criteria; `08` + study page produced.

## Stages

- [x] 1. Orientation — purpose, stack, entry points, repo type, all cited → `01-overview.md`
- [x] 2. Architecture — all 5 top-level modules explained, all 4 load edges traced → `02-architecture.md`
- [x] 3. Domain — every term has a meaning, a home file, and its relationships → `03-domain-glossary.md`
- [x] 4. Key flows — 2 flows traced end to end with `file:line` waypoints → `04-key-flows.md`
- [x] 5. Dependencies & blast radius — 6 externals + 7 ranked blast radii → `05-dependencies.md`
- [x] 6. Operations — build/deploy/config described (all three are empty), 6 fragile spots named → `06-operations.md`
- [x] 7. Safe contribution — one specific starting area + exact verify commands → `07-safe-contribution.md`

Coverage: **7/7**. The coverage check at `references/synthesis.md:6-14` passes on merit, not by
waiver.

## Classification note

The Library/SDK playbook fits the *shape* — no server, no long-running entry point, an explicit
public surface, consumed by installing it into other projects. It does not fit the *signals*: no
registry metadata, no `examples/`, no CHANGELOG, no tests. The "library" here is a prompt, not
executable code. Treat the classification as **soft**; the generic fallback's four anchors
(`references/repo-playbooks.md:353-356`) carried Stages 4–6 where there was no code path to trace.

## What made this repo unusual

Zero lines of executable code. "Architecture" meant *what loads into context and when*; "key flows"
meant paths through documents; "operations" meant that build, CI, and config are all genuinely
empty and `main` ships on push.

## Resolved during the session

- [resolved] External adoption — public repo, **0 stars, 0 forks, 0 open issues, 0 releases, 0 tags**
  (GitHub API, 2026-07-27). The answer is "nobody yet", not "unmeasured".
- [resolved] The `skills` CLI is real — npm `skills` **v1.5.20**, `github.com/vercel-labs/skills`.
- [resolved] There is no `.skill` release asset and never has been in this repo (0 releases; `9a77940`
  removed the committed bundle). Doc corrected in `047333c`.

## Open questions

- [unknown] Whether Claude Code *enforces* `name:` == directory name, or merely conventions it —
  a platform behaviour, not answerable from this repo. Would need the Claude Code docs.
- [unknown] Whether `npx skills@latest add nitfolio/nirvajna-skills` has ever run successfully
  against this repo. Would need a scratch install.
- [unknown] What "nirvajna" means or why it was chosen — appears only in the repo name and URLs,
  never in a document body. Ask the author.
- [unknown] Whether branch protection or other GitHub-side settings exist. Would need repo settings
  access.
- [unknown] How the author packages a `.skill` bundle when one is needed — no script is tracked.
- [unknown] Whether the author prefers PRs or issues first. `README.md:140` welcomes both; there is no
  `CONTRIBUTING.md`, no PR template, and no PR history to infer from.

## Next step

None — the ladder is finished. `08-onboarding.md` and `onboarding.html` are written. A future session
can `start` to extend any stage, or `jump to <topic>` for a specific area.
