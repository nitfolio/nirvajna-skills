# 05 · Dependencies & blast radius

## External dependencies

There is no dependency manifest, so these were found by reading, not by resolving a lockfile.

| Dependency | Where it enters | What happens if it fails or changes |
| --- | --- | --- |
| **Claude Code** (the runtime) | The whole premise; format described at `README.md:27-29` | If the skill format changes — frontmatter keys, directory-name matching, `references/` loading — the skill stops triggering. Nothing in the repo would report the breakage; it just goes quiet. |
| **`skills` CLI** (npm, `vercel-labs/skills`) | `README.md:61`, described at `:64-67` | [fact] Verified live: version **1.5.20**, repo `github.com/vercel-labs/skills`. It is the *documented primary install path*. If it changes its layout (`.agents/skills/` + symlinks), the install prose goes stale — the skill itself is unaffected, since `cp -r` still works. |
| **GitHub raw + blob URLs** | 9 hardcoded `nitfolio` URLs, **all in `onboard-me/README.md`** | Renaming the repo or the owner breaks every image and link in the skill's README wherever it renders outside GitHub. See blast radius below. |
| **img.shields.io** | 4 badges in `README.md`, 4 in `onboard-me/README.md` | Badges break; nothing functional. One badge encodes a fact that can go stale (`Repo_playbooks-13`, `onboard-me/README.md:14`). |
| **oopsaididitagain.com** | `README.md:16`, `:162`; `onboard-me/README.md:21` | Dead link only. |
| **Browser Clipboard API** | `study-page-template.html:348-355` | [fact] Already handled — falls back to a hidden `<textarea>` + `execCommand`. The study page has no other browser requirement and no network requirement at all. |

[fact] **`SKILL.md`, `repo-playbooks.md`, and `synthesis.md` contain zero URLs** (grep count: 0 for
each). Everything the agent actually loads is self-contained. This is a real property, not an
accident of scale — it means a run works with no network and the skill survives being copied
anywhere.

## Repo state (verified 2026-07-27)

[fact] Public · 0 stars · 0 forks · 0 open issues · 0 releases · 0 tags · last push
`2026-07-27T06:41:09Z`.

[fact] Consequence: there is **no `.skill` release asset**, and never has been in this repo's
lifetime. `9a77940` removed the previously-committed bundle. Any doc telling a user to "download
`onboard-me.skill`" would be pointing at nothing — which is exactly the drift corrected in `047333c`
earlier today.

[fact] Consequence: no external adoption signal exists. This resolves the Stage-1 unknown — the
answer is "nobody yet", not "unmeasured".

## Blast radius — what breaks if I change this

Ordered by how much damage a careless edit does.

### 1. `SKILL.md:3-10` — the `description` — **highest**

[fact] It is the only trigger surface (`README.md:128-130`). Narrow it and the skill stops firing on
cases it used to catch; broaden it and it hijacks unrelated conversations. [fact] The failure is
**silent and invisible from the repo** — nothing tests it, and you only notice a skill that stopped
triggering by its absence.

### 2. Renaming the skill — **three places, one of them off-disk**

[fact] Demonstrated by commit `00b7dd3` (`codebase-onboarding` → `onboard-me`). Three things must
move together:

1. The directory `onboard-me/`.
2. `SKILL.md:2`'s `name:` — the platform requires it to match the directory.
3. **Every already-installed copy.** [fact] On this machine `~/.claude/skills/onboard-me` is a
   Windows *junction* pointing back into this repo, not a copy. A `git mv` breaks it, leaving a
   dangling link that resolves to nothing.

[fact] Point 3 is invisible from inside the repo and is not documented anywhere in it. On Unix the
`skills` CLI creates symlinks (`README.md:64-67`), so the same hazard applies there. Anyone renaming
a skill should expect to repair installs afterward.

Also in scope for a rename: the 9 hardcoded `github.com/nitfolio/nirvajna-skills` URLs in
`onboard-me/README.md`, plus every `onboard-me` mention across both READMEs.

### 3. The evidence-tag vocabulary — **silent cosmetic break**

[fact] `SKILL.md:74-82` defines `[fact]`, `[inference]`, `[unknown]`, `[human]`.
[fact] `study-page-template.html:218` hard-codes exactly those four in a regex. Rename one, add a
fifth, and every existing study page renders it as plain text with no error. Nothing links the two
files; nothing checks them.

### 4. The `.kt/` file names or numbering — **breaks the study page**

[fact] `study-page-template.html:171-205` has exactly 9 slots keyed by `data-id`
(`08-onboarding`, `00-progress` … `07-safe-contribution`). [fact] `SKILL.md:196-208` and
`onboard-me/README.md:225-237` both document the same list. Renaming a `.kt/` file means editing
three files, one of which is HTML.

[fact] Partial mitigation: `:314` drops any slot that is still an unfilled comment, so a *missing*
file degrades gracefully. A *renamed* one does not — its content simply never appears.

### 5. Adding or removing a repo-type playbook — **5 stale counters**

[fact] The number `13` is stated in five places, kept in sync by hand alone:

- `README.md:48` — "Adapts to 13 repo types"
- `README.md:118` — layout tree comment
- `onboard-me/README.md:14` — the `Repo_playbooks-13` badge
- `onboard-me/README.md:190-193` — the prose list of covered types
- `onboard-me/README.md:359` — files tree comment

Plus the contents list at `repo-playbooks.md:13-28`, which is numbered 1–14 (13 types + the generic
fallback). [fact] All six currently agree — checked this session.

### 6. Adding a ladder stage — **needs a completion criterion**

[fact] `onboard-me/README.md:345-346` states the trap outright: "If you add one, give it a completion
criterion — a stage without one gets declared done early." Downstream, a new stage likely wants a new
`.kt/` file, which drags in item 4 above.

### 7. Prose in `SKILL.md` generally — **cheap**

[inference] Because the file is loaded whole and interpreted by a model rather than parsed, most
edits fail soft: wording changes shift behaviour by degrees rather than breaking anything. Based on
there being no parser, no schema, and no test anywhere in the repo. The dangerous edits are the
structural ones above, not the wording.

## What has no blast radius at all

[fact] `assets/`, `LICENSE`, `.gitignore`, and both READMEs are inert at runtime — nothing the agent
loads points at them. Editing them cannot change how a KT session behaves.

## Open unknowns

- [unknown] Whether Claude Code enforces the `name:`-equals-directory rule, or merely conventions it.
  Not determinable from this repo; it is a platform behaviour.
- [unknown] Whether `npx skills@latest add nitfolio/nirvajna-skills` has ever been run successfully
  against this repo. The package exists and the command is well-formed, but nothing records a
  verified install.
