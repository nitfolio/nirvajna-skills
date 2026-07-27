# 06 · Operations

## Build

[fact] **There is none.** No `package.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`, or `Makefile`
— each checked individually. Nothing compiles, bundles, minifies, or generates. The files in the
repo are the artifact.

[fact] No test suite. No linter. No formatter config. No pre-commit hooks in the tree.

## CI

[fact] **There is none.** No `.github/` directory at all, so no workflows, no PR checks, no branch
protection visible from the repo.

[fact] Consequence: every consistency rule in this repo is enforced by hand. Nothing checks that
`SKILL.md:2`'s `name` matches the directory, that the five playbook counters agree, that the study
page's tag regex still matches the tag vocabulary, or that the two READMEs tell the same story.

## Deploy / distribution

[fact] **`main` is production, and it deploys on push.** Three consumption paths, all reading `main`
directly:

| Path | Where documented | What it reads |
| --- | --- | --- |
| `npx skills@latest add nitfolio/nirvajna-skills` | `README.md:61` | The repo at **`main`** — the command carries no ref, tag, or version |
| `cp -r onboard-me ~/.claude/skills/` | `README.md:78-81` | Whatever the user has checked out |
| Zip / `.skill` upload to claude.ai or Cowork | `README.md:99-104`, `onboard-me/README.md:60-63` | A hand-made archive |

[fact] The 9 asset URLs in `onboard-me/README.md` are likewise pinned to `/main/`
(`raw.githubusercontent.com/nitfolio/nirvajna-skills/main/…`).

[fact] There are **0 tags and 0 releases**. So there is no version, no changelog, no pinning, and
nothing to roll back to. A bad push is live immediately for anyone installing, and the only remedy is
another push.

[fact] The `.skill` bundle is *not* a build output of this repo — no such file is tracked, and
`9a77940` deliberately removed the one that used to be. It is packaged ad hoc when someone needs it.

## GitHub Pages

[fact] The repo also publishes itself as a static site, enabled 2026-07-27 from `main` at the repo
root: `https://nitfolio.github.io/nirvajna-skills/`. Its only purpose is to serve the study page at
`https://nitfolio.github.io/nirvajna-skills/.kt/onboarding.html`, which `onboard-me/README.md` links
from its PS.

[fact] This requires the empty `.nojekyll` file at the repo root. Jekyll skips paths beginning with a
dot, so without it `.kt/` would never be published. Verified: with `.nojekyll` present, Pages serves
`.kt/onboarding.html` as `200 text/html` and `.kt/08-onboarding.md` as `200 text/markdown`.

[fact] The site root returns **404** — there is no `index.html`. Only the deep link is advertised.

[fact] Blast radius: deleting `.nojekyll` silently unpublishes `.kt/`, breaking the README link with
no error anywhere. It is a one-line file with no obvious purpose, which makes it exactly the kind of
thing a tidy-up deletes.

## Configuration

[fact] Almost none. The only tracked config is the empty `.nojekyll` marker described above — no
build config, no linter config, no CI config, no application config.

[fact] `.claude/settings.local.json` exists on this machine but is untracked, has one top-level key
(`permissions`), and is ignored by the *user's global* gitignore (`~/.config/git/ignore:1`) rather
than by this repo. It is a local Claude Code preference file, not project config.

[fact] `.gitignore` contains two comment lines and **zero patterns**. Everything in the working tree
is committed by default. This is deliberate: the comment says `.kt/` is kept on purpose as a worked
example of the skill's output.

[fact] No secrets exist in this repo to leak — no `.env`, no credentials, no tokens, no keys. The
only sensitive-ish surface is `.claude/settings.local.json`, which is untracked and was inspected for
shape only.

## The one runtime "environment"

[inference] The nearest thing to a deployment target is the user's skills directory. On this machine
`~/.claude/skills/onboard-me` is a **Windows junction** into this repo, so edits here are live in
every Claude Code session with no install step. Based on the junction observed during the `00b7dd3`
rename, when `git mv` broke it and it had to be recreated.

[fact] The `skills` CLI does the same thing on other platforms with symlinks — one canonical copy in
`.agents/skills/`, symlinked into `.claude/skills/` and other agent directories (`README.md:64-67`).

[fact] Operational consequence: **a rename or move in this repo silently breaks installed links**,
and the break is invisible from inside the repo. `--copy` (`README.md:67`) trades liveness for
immunity to this.

## Fragile spots

Named plainly, worst first.

1. **Doc drift between the two READMEs.** [fact] This has already happened twice and is the repo's
   demonstrated failure mode: `363b5ff` made `npx skills add` the primary install path in
   `README.md` only, leaving `onboard-me/README.md` telling people to `cp -r` and to download a
   non-existent `onboard-me.skill`. Both were corrected in `047333c` (2026-07-27). Nothing prevents a
   third instance.
2. **Five hand-synced playbook counters.** [fact] `README.md:48`, `README.md:118`,
   `onboard-me/README.md:14`, `:190-193`, `:359` — all currently agree; nothing keeps them that way.
3. **Tag vocabulary ↔ study-page regex.** [fact] `SKILL.md:74-82` vs
   `study-page-template.html:218`. Drift here fails silently and cosmetically.
4. **Installed junctions/symlinks break on rename.** See above; undocumented in the repo.
5. **`main` has no safety net.** [fact] No CI, no tags, no releases, no protected branch, and the
   install path reads `main` — so there is no gap between "committed" and "shipped".
6. **Bus factor 1.** [inference] 21 commits, one author (`nitfolio <ntv.nitin@gmail.com>`), no
   reviewers, no CI. Based on the full `git log` author breakdown.

## Conventions worth matching

[fact] Commit subjects are imperative, sentence-case, no prefix or scope — e.g. "Rename the
codebase-onboarding skill to onboard-me", "Document the npx skills installer as the primary install
path", "Enlarge the README wordmark to 640px". One exception: `6e83541` "wip: readme preview -
brand hero, badges, mermaid ladder".

[fact] Single branch `main`; every commit is on it. No merge commits, no PR history.

## Open unknowns

- [unknown] Whether branch protection or any GitHub-side setting exists — not visible from the repo,
  and the repo API fields checked did not cover it.
- [unknown] How the author packages a `.skill` bundle when one is needed. No script for it is tracked.
