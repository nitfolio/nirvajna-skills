# 05 · Dependencies & blast radius

*Written against commit `2e94f1b` on 2026-07-27. Every `file:line` below is a line number **at that commit** — check that commit out to verify a claim, or re-run the skill to regenerate the trail against current source.*

## External dependencies

There is no dependency manifest, so these were found by reading, not by resolving a lockfile.

| Dependency | Where it enters | What happens if it fails or changes |
| --- | --- | --- |
| **Claude Code** (the runtime) | The whole premise; format described at `README.md:27-29` | If the skill format changes — frontmatter keys, directory-name matching, `references/` loading — the skill stops triggering. Nothing in the repo would report it; it just goes quiet. |
| **`skills` CLI** (npm, `vercel-labs/skills`) | `README.md:61`, described at `:64-67` | [fact] Verified: npm `skills` **v1.5.20**, repo `github.com/vercel-labs/skills`. The documented primary install path. If its layout changes, the install prose goes stale — the skill itself is unaffected, since `cp -r` still works. |
| **GitHub Pages** *(new 2026-07-27)* | Hosts the study page; linked from `onboard-me/README.md`'s PS | [fact] Enabled from `main` at root, `build_type: legacy`, HTTPS enforced, status `built`. If a build fails or Pages is disabled, the README's study-page link 404s. |
| **`.nojekyll`** | Empty file at the repo root | [fact] Without it Jekyll skips dot-prefixed paths and `.kt/` is never published. Deleting it silently unpublishes the trail. See blast radius #2. |
| **GitHub raw + blob URLs** | 10 hardcoded `nitfolio` URLs, **all in `onboard-me/README.md`** | Renaming the repo or owner breaks every image and link in the skill's README wherever it renders outside GitHub. |
| **img.shields.io** | 4 badges in `README.md`, 4 in `onboard-me/README.md` | Badges break; nothing functional. One encodes a fact that can go stale (`Repo_playbooks-13`, `onboard-me/README.md:14`). |
| **oopsaididitagain.com** | `README.md:16`, `:166`; `onboard-me/README.md:21` | Dead link only. |
| **mermaid.live** *(new)* | `study-page-template.html:251` | [fact] A hyperlink in the mermaid fence note, not a fetch. If the site disappears the link dies; the page still renders offline exactly as before. |
| **Browser Clipboard API** | `study-page-template.html:348-355` | [fact] Already handled — falls back to a hidden `<textarea>` + `execCommand`. |

[fact] **`SKILL.md`, `repo-playbooks.md`, and `synthesis.md` contain zero URLs** (grep count 0 for
each). Everything the agent loads is self-contained — a run works with no network, and the skill
survives being copied anywhere.

[fact] The study page makes **no network requests**. Its single external reference is the
`mermaid.live` hyperlink, which fires only if a reader clicks it.

## Repo state (verified 2026-07-27, commit `2e94f1b`)

[fact] Public · 0 stars · 0 forks · 0 open issues · 0 releases · 0 tags · 30 commits · one author ·
`has_pages: true`.

[fact] Consequence: no `.skill` release asset exists and never has in this repo's lifetime
(`9a77940` removed the committed bundle). Any doc telling a user to download one points at nothing.

[fact] Consequence: no external adoption signal exists. The answer is "nobody yet", not "unmeasured".

## Blast radius — what breaks if I change this

Ordered by how much damage a careless edit does.

### 1. `SKILL.md:3-10` — the `description` — **highest**

[fact] It is the only trigger surface (`README.md:132-134`). Narrow it and the skill stops firing on
cases it used to catch; broaden it and it hijacks unrelated conversations. [fact] The failure is
**silent and invisible from the repo** — nothing tests it, and a skill that stopped triggering is
noticed only by its absence.

### 2. Deleting `.nojekyll` — **the newest and sneakiest**

[fact] It is an **empty file with no visible purpose**, which makes it exactly what a tidy-up
deletes. Removing it makes GitHub Pages resume skipping dot-prefixed paths, `.kt/` vanishes from the
published site, and the study-page link in `onboard-me/README.md`'s PS 404s.

[fact] Nothing warns about this — no comment in the file (it has none), no note in either README.
Only this trail records it.

### 3. Renaming the skill — **three places, one of them off-disk**

[fact] Demonstrated by `00b7dd3` (`codebase-onboarding` → `onboard-me`). Three things must move:

1. The directory `onboard-me/`.
2. `SKILL.md:2`'s `name:` — the platform requires it to match the directory.
3. **Every already-installed copy.** [fact] On this machine `~/.claude/skills/onboard-me` is a
   Windows *junction* into this repo, not a copy; a `git mv` leaves it dangling.

[fact] Point 3 is invisible from inside the repo and undocumented in it. The `skills` CLI uses
symlinks elsewhere (`README.md:64-67`), so the hazard is not Windows-specific. Also in scope: the 10
hardcoded GitHub URLs in `onboard-me/README.md`.

### 4. The evidence-tag vocabulary — **silent cosmetic break**

[fact] `SKILL.md:74-82` defines the four tags; `study-page-template.html:218` hard-codes exactly
those four in a regex. Rename one or add a fifth and every study page renders it as plain text, with
no error. Nothing links the two files.

### 5. The `.kt/` file names or numbering — **breaks the study page**

[fact] `study-page-template.html:171-205` has 9 slots keyed by `data-id`. [fact] `SKILL.md:196-208`
and `onboard-me/README.md:227-239` document the same list. Renaming a `.kt/` file means editing three
files, one of them HTML.

[fact] Partial mitigation: `:314` drops any slot that is still an unfilled comment, so a *missing*
file degrades gracefully. A *renamed* one does not — its content simply never appears.

### 6. Editing any file the trail cites — **the repo's most active failure mode**

[fact] The `.kt/` trail cites `file:line` throughout. Editing `SKILL.md`, `synthesis.md`, or the
template shifts those lines and silently invalidates the citations. [fact] On 2026-07-27 this
happened **four separate times in one day** — the Diagrams paragraph moved `SKILL.md` by 6 lines
from `:210`, the synthesis shape list by 1 from `:37`, the template by 8, and a mid-session README
edit shifted both READMEs — each requiring a sweep of ~20 citations.

[fact] Worse, a systematic audit of all 97 citations found roughly **15 that were wrong before any
of today's edits**. They were inherited from an earlier run and never re-verified after the install
section was rewritten in `047333c`. Examples: `onboard-me/README.md:190-193` pointed at mermaid
`classDef` lines instead of the "Covered:" list (correct: `207-210`); `README.md:128-130` pointed at
a `<summary>` tag instead of the description rule (correct: `132-134`);
`study-page-template.html:215` pointed one line off the tag regex.

[fact] Every one of these is *in range* — the cited line exists. Only reading the target reveals the
error, which is why an existence check is not enough.

[human] Resolved by the author: **stamp each `.kt/` file with the commit it was written against.**
A citation is then a line number at a known snapshot — a reader checks out that commit to verify, or
regenerates the trail. Drift stops being wrongness and becomes a versioning question.

[fact] A `scripts/check-citations.py` was written, tested, and then removed on 2026-07-27. It only
hard-failed citations pointing *past the end* of a file; all ~15 real errors were in range and passed
it silently. What caught them was reading the target lines, which `grep` and `sed` do without adding
a Python dependency, executable code, or an exception to the read-only boundary — all of which cut
against the skill's "no plugin, no config, no runtime" claim (`README.md:27-29`).

### 7. Adding or removing a repo-type playbook — **5 stale counters**

[fact] The number `13` is stated in five places, hand-synced: `README.md:48`, `README.md:118`,
`onboard-me/README.md:14` (badge), `:207-210`, `:361`. Plus the contents list at
`repo-playbooks.md:13-28`, numbered 1–14 (13 types + generic fallback). [fact] All six currently
agree — checked this session.

### 8. Adding a ladder stage — **needs a completion criterion**

[fact] `onboard-me/README.md:346-347`: "If you add one, give it a completion criterion — a stage
without one gets declared done early." A new stage likely wants a new `.kt/` file, dragging in #5.

### 9. Prose in `SKILL.md` generally — **cheap**

[inference] The file is loaded whole and interpreted by a model rather than parsed, so most edits
fail soft — wording shifts behaviour by degrees. Based on there being no parser, schema, or test
anywhere. The dangerous edits are the structural ones above.

## What has no blast radius at all

[fact] `assets/`, `LICENSE`, and `.gitignore` are inert — nothing the agent loads points at them.
Editing them cannot change how a KT session behaves.

## Open unknowns

- [unknown] Whether Claude Code *enforces* `name:` == directory name, or merely conventions it. A
  platform behaviour, not answerable from this repo.
- [unknown] Whether `npx skills@latest add nitfolio/nirvajna-skills` has ever run successfully
  against this repo.
- [unknown] Whether anything other than the skill README's PS links the published Pages site.
