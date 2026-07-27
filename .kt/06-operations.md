# 06 · Operations

## Build

[fact] **There is none.** No `package.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`, or `Makefile`
— each checked individually. Nothing compiles, bundles, or generates. The files in the repo are the
artifact.

[fact] No test suite. No linter. No formatter config. No pre-commit hooks in the tree.

## CI

[fact] **There is none.** No `.github/` directory at all — no workflows, no PR checks.

[fact] The one automated process is **GitHub Pages**, which builds on push. It is hosting, not CI:
it never validates anything and cannot fail a bad change.

[fact] Consequence: every consistency rule here is enforced by hand. Nothing checks that
`SKILL.md:2`'s `name` matches the directory, that the five playbook counters agree, that the study
page's tag regex still matches the tag vocabulary, that `.kt/` citations still resolve, or that the
two READMEs tell the same story.

## Deploy / distribution

[fact] **`main` is production, and it deploys on push — now along two paths at once.**

| Path | Where documented | What it reads |
| --- | --- | --- |
| `npx skills@latest add nitfolio/nirvajna-skills` | `README.md:61` | The repo at **`main`** — no ref, tag, or version |
| `cp -r your-skill-name ~/.claude/skills/` | `README.md:78-81` | Whatever the user has checked out |
| Zip / `.skill` upload to claude.ai or Cowork | `README.md:99-103`, `onboard-me/README.md:74-78` | A hand-made archive |
| **GitHub Pages** *(new)* | `onboard-me/README.md` PS | **`main`** at repo root, rebuilt on every push |

[fact] The 10 asset and link URLs in `onboard-me/README.md` are likewise pinned to `/main/`.

[fact] There are **0 tags and 0 releases**. No version, no changelog, no pinning, nothing to roll
back to. A bad push is live immediately for installers *and* for the website; the only remedy is
another push.

[fact] The `.skill` bundle is *not* a build output — no such file is tracked, and `9a77940`
deliberately removed the one that used to be. It is packaged ad hoc when someone needs it.

## GitHub Pages

[fact] Enabled 2026-07-27 from `main` at the repo root. `build_type: legacy`, HTTPS enforced, status
`built`. Site: `https://nitfolio.github.io/nirvajna-skills/`.

[fact] Its only purpose is to serve the study page at
`https://nitfolio.github.io/nirvajna-skills/.kt/onboarding.html`, linked from the PS at the end of
`onboard-me/README.md`.

[fact] It requires the empty `.nojekyll` at the repo root — Jekyll skips paths beginning with a dot,
so without it `.kt/` would never be published. Verified: with `.nojekyll` present, Pages serves
`.kt/onboarding.html` as `200 text/html` and `.kt/08-onboarding.md` as `200 text/markdown`.

[fact] The site root returns **404** — there is no `index.html`. Only the deep link is advertised.

[fact] Builds are not instant. Observed on 2026-07-27: roughly a couple of minutes from push to the
new content being served, during which the old page is still live.

## Configuration

[fact] Almost none. The only tracked config is the empty `.nojekyll` marker — no build config, no
linter config, no CI config, no application config.

[fact] `.claude/settings.local.json` exists on this machine but is untracked, has one top-level key
(`permissions`), and is ignored by the *user's global* gitignore (`~/.config/git/ignore:1`) rather
than by this repo. A local Claude Code preference file, not project config.

[fact] `.gitignore` contains two comment lines and **zero patterns**. Everything in the working tree
is committed by default — deliberately, so `.kt/` ships as a worked example.

[fact] No secrets exist in this repo — no `.env`, no credentials, no tokens, no keys. The only
sensitive-ish surface is the untracked `.claude/settings.local.json`, inspected for shape only.

## The one runtime "environment"

[inference] The nearest thing to a deployment target is the user's skills directory. On this machine
`~/.claude/skills/onboard-me` is a **Windows junction** into this repo, so edits here are live in
every session with no install step. Based on the junction observed during the `00b7dd3` rename, when
`git mv` broke it and it had to be recreated.

[fact] The `skills` CLI does the same with symlinks elsewhere (`README.md:64-67`). Operational
consequence: **a rename or move silently breaks installed links**, invisibly from inside the repo.
`--copy` trades liveness for immunity.

## Fragile spots

Named plainly, worst first.

1. **Citation drift in `.kt/`.** [fact] The trail cites `file:line` throughout, and editing any cited
   file silently invalidates them. Observed **three times on 2026-07-27 alone** — `SKILL.md` +6 from
   `:210`, `synthesis.md` +1 from `:37`, the template +8 — each needing a ~20-citation sweep. [fact]
   One citation was wrong from the moment it was written (`template:215`; the regex was at `:217`).
   Nothing checks this.
2. **Deleting `.nojekyll`.** [fact] An empty file with no visible purpose; removing it unpublishes
   `.kt/` and 404s the README's study-page link, with no error anywhere.
3. **Doc drift between the two READMEs.** [fact] Already happened twice: `363b5ff` changed the
   install path in `README.md` only, leaving `onboard-me/README.md` recommending `cp -r` and a
   non-existent `onboard-me.skill`. Both corrected in `047333c`.
4. **Five hand-synced playbook counters.** [fact] `README.md:48`, `:118`,
   `onboard-me/README.md:14`, `:207-210`, `:361` — all currently agree; nothing keeps them that way.
5. **Tag vocabulary ↔ study-page regex.** [fact] `SKILL.md:74-82` vs
   `study-page-template.html:218`. Drift fails silently and cosmetically.
6. **Installed junctions/symlinks break on rename.** Undocumented in the repo.
7. **`main` has no safety net.** [fact] No CI, no tags, no releases, no protected branch, and both
   the installer and the website read `main` — there is no gap between "committed" and "shipped".
8. **Bus factor 1.** [inference] 30 commits, one author, no CI, no reviewers.

## Conventions worth matching

[fact] Commit subjects are imperative, sentence-case, no prefix or scope — "Rename the
codebase-onboarding skill to onboard-me", "Add .nojekyll so GitHub Pages publishes the dot-prefixed
.kt/ directory". One exception across all 30: `6e83541` "wip: readme preview - brand hero, badges,
mermaid ladder".

[fact] Single branch `main`; every commit is on it. No merge commits, no PR history.

## Open unknowns

- [unknown] Whether branch protection or other GitHub-side settings exist — not visible from the
  repo, and not covered by the API fields checked.
- [unknown] How the author packages a `.skill` bundle when one is needed. No script is tracked.
