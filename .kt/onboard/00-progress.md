# KT progress

*Written against commit `2e94f1b` on 2026-07-27. Every `file:line` below is a line number **at that commit** — check that commit out to verify a claim, or re-run the skill to regenerate the trail against current source.*

Goal: regenerate the trail from scratch against the repo as it stands after the 2026-07-27 changes
(GitHub Pages, `.nojekyll`, the diagram rule) · Repo type: **Library / SDK** (playbook 5), soft match
Mode: `speedrun` (full ladder, no per-turn gate) · Session date: 2026-07-27 · Repo at `2e94f1b`
Status: **complete** — all 7 stages met their completion criteria; `08` + study page produced.

## Stages

- [x] 1. Orientation — purpose, stack, entry points, repo type, all cited → `01-overview.md`
- [x] 2. Architecture — all 8 top-level entries explained, 4 load edges traced → `02-architecture.md`
- [x] 3. Domain — every term has a meaning, a home file, and its relationships → `03-domain-glossary.md`
- [x] 4. Key flows — 2 flows traced end to end, Flow B now 7 waypoints → `04-key-flows.md`
- [x] 5. Dependencies & blast radius — 9 externals + 9 ranked blast radii → `05-dependencies.md`
- [x] 6. Operations — build/CI/config described (near-empty), 8 fragile spots → `06-operations.md`
- [x] 7. Safe contribution — one starting area + 5 verification steps → `07-safe-contribution.md`

Coverage: **7/7**. The coverage check at `references/synthesis.md:6-14` passes on merit, not by
waiver.

## Classification note

Library/SDK fits the *shape* — no server, no long-running entry point, an explicit public surface,
consumed by installing into other projects. It fits none of the *signals*: no registry metadata, no
`examples/`, no CHANGELOG, no tests. The "library" is a prompt, not executable code. Classification
is **soft**; the generic fallback's four anchors (`references/repo-playbooks.md:353-356`) carried the
stages where there was no code path to trace.

## What changed since the previous trail (same day)

The repo moved underneath the first run, which is why this one exists:

- `.nojekyll` added; **GitHub Pages enabled** from `main` at root — the repo is now also a website.
- `SKILL.md` gained a **diagram rule** (347 → 353 lines): text by default, mermaid only for GitHub.
- `synthesis.md` 64 → 65; `study-page-template.html` 368 → 376 (mermaid fence note + its CSS).
- `onboard-me/README.md` gained a PS linking the published study page.
- Tracked source files 12 → 13; commits 25 → 30.
- The template now carries **one external URL** (`mermaid.live`, a hyperlink, not a fetch).

## Resolved during the session

- [resolved] External adoption — public, **0 stars, 0 forks, 0 issues, 0 releases, 0 tags**
  (GitHub API, 2026-07-27). "Nobody yet", not "unmeasured".
- [resolved] Pages state — `built`, `main` at root, HTTPS enforced; serves `.kt/onboarding.html` as
  `200 text/html` and `.kt/08-onboarding.md` as `200 text/markdown`.
- [resolved] The offline guarantee survives the mermaid link — the page still makes zero network
  requests; the link fires only on a click.

## Open questions

- [unknown] Whether the Pages site is discoverable. The root 404s and the study-page URL appears only
  in the skill README's PS — nothing links it from the repo front door. Ask the author.
- [unknown] Whether Claude Code *enforces* `name:` == directory name, or merely conventions it.
  A platform behaviour; needs the Claude Code docs.
- [unknown] Whether `npx skills@latest add nitfolio/nirvajna-skills` has ever run successfully
  against this repo. Would need a scratch install.
- [unknown] What "nirvajna" means — appears only in the repo name and URLs, never in a document body.
- [unknown] Whether branch protection or other GitHub-side settings exist.
- [unknown] How the author packages a `.skill` bundle — no script is tracked.
- [unknown] Whether the author prefers PRs or issues first — no `CONTRIBUTING.md`, no PR history.

## Next step

None — the ladder is finished. A future session can `start` to extend any stage, or
`jump to <topic>`. The citation-drift question the trail raised was resolved during the session:
every `.kt/` file now carries the commit it was written against, so a drifted citation is checkable
rather than wrong. A checker script was built, evaluated, and deliberately discarded — see
`05-dependencies.md`.
