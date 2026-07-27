# 07 · Safe contribution

## The house rules

[fact] `README.md:140-147` states them for anyone adding a skill:

- Keep it **self-contained** in its own folder.
- Write the `SKILL.md` description as **trigger conditions**, not a summary of features.
- Push reference material that only some runs need into `references/`, so `SKILL.md` stays lean.
- Give any multi-stage process a checkable **"done when"** condition per stage — "vague criteria are
  the single most common reason a skill stops early."
- Add a `README.md` for humans, and a row to the skills table.

[fact] `onboard-me/README.md:339-348` adds four ways to extend the existing skill: add a repo type,
change the ladder, change the `.kt/` layout, tune the house style.

## Recommended first change: add a repo-type playbook

[fact] The repo invites exactly this at `onboard-me/README.md:341-343`.

**Why it's the safest real change:**

[fact] A new playbook section is inert until the Orientation classifier picks it. `SKILL.md:186-188`
names the covered categories generically ("backend services, monorepos, frontends…") and does not
enumerate them by number, so adding a 14th type cannot break an existing run.

[inference] It also can't reach the two silent couplings from `02-architecture.md` — it touches
neither the evidence-tag vocabulary nor the `.kt/` filenames. Based on those couplings living in
`SKILL.md:74-82` and `study-page-template.html:171-205`, neither of which a playbook edits.

**Where to work:** `onboard-me/references/repo-playbooks.md`.

**Keep the shape.** [fact] Every existing playbook has the same five parts — copy one (e.g.
`:130-152`) and fill them in:

1. `**Recognize it:**` — the signals that classify a repo as this type
2. `**Ladder emphasis:**` — where depth goes, and what "key flows" means for this kind of system
3. `**Read first:**` — the files worth opening before anything else
4. `**The KT must answer:**` — the questions a newcomer to this system always ends up asking
5. `**Traps:**` — the ways a KT of this type goes wrong

[fact] `repo-playbooks.md:8-11` explains why parts 2 and 4 exist: a playbook adapts the ladder but
never replaces the core loop, the evidence tags, or the artifacts; and the must-answer list is the
exit criterion for a good KT of that type, with unanswered items going to `00-progress.md` as
`[unknown]`.

**Then update the counters.** [fact] Five hand-synced places plus the contents list:

| File | Line | What to change |
| --- | --- | --- |
| `onboard-me/references/repo-playbooks.md` | `13-28` | Add the new number to the contents list |
| `README.md` | `48` | "Adapts to 13 repo types" |
| `README.md` | `118` | Layout tree comment |
| `onboard-me/README.md` | `14` | The `Repo_playbooks-13` badge URL |
| `onboard-me/README.md` | `190-193` | The prose list of covered types |
| `onboard-me/README.md` | `359` | Files tree comment |

[fact] Keep the generic fallback last — `repo-playbooks.md:347` is numbered 14 today precisely
because it is the "nothing fits" case.

## How to verify it

There is no test command. [fact] Nothing in the repo builds, tests, or lints — so verification is a
consistency sweep plus one real run. **I did not run any of these** (`SKILL.md:290-292`: a speedrun
never executes commands); they are for you.

### 1. Structural checks (seconds, from the repo root)

```bash
# frontmatter name must equal the directory name
head -3 onboard-me/SKILL.md          # expect: name: onboard-me

# playbook count: numbered sections vs every claimed counter
grep -c '^## [0-9]' onboard-me/references/repo-playbooks.md
grep -rn 'repo-type playbooks\|repo types\|Repo_playbooks-' --include=*.md .

# the silent coupling: tag vocabulary must match the study-page regex
grep -n '^- \*\*\[' onboard-me/SKILL.md
grep -n 'fact|inference|unknown|human' onboard-me/references/study-page-template.html

# the .kt/ filenames documented in three places must agree
grep -c 'data-id=' onboard-me/references/study-page-template.html   # expect: 9
```

[fact] Expected today: 14 numbered sections (13 types + fallback), all counters reading 13, the tag
list and the regex both naming the same four tags, and 9 slots.

### 2. The real end-to-end verification — run the skill

[fact] The only way to check a prompt change actually works is to run it. `README.md:88-92`:

```bash
# install the local copy into a scratch project
cp -r onboard-me <some-scratch-repo>/.claude/skills/

# then, in Claude Code inside that repo:
/onboard-me
```

Then confirm, in order: it asks the goal question once (`SKILL.md:30-45`), it classifies the repo and
says which playbook it loaded (`SKILL.md:182-188`), it announces `.kt/` before creating it
(`SKILL.md:244-245`), and — for a full check — `stop` produces both `08-onboarding.md` and
`onboarding.html`.

[fact] A new playbook is only exercised if the scratch repo actually *is* that type. Pick the test
repo to match what you added, or the change goes untested.

### 3. Check the study page renders

[fact] `.kt/onboarding.html` is self-contained and needs no server (`synthesis.md:63-64`) — open it
directly:

```bash
start .kt/onboarding.html      # Windows;  macOS: open,  Linux: xdg-open
```

Confirm the sidebar lists every file, the `[fact]`/`[inference]`/`[unknown]`/`[human]` tags are
colour-coded, and the "Copy markdown" buttons work.

### 4. Committing

[fact] Match the existing convention: imperative, sentence-case subject, no prefix
("Add a Rust workspace playbook"). Single branch `main`; no PR history exists, but note from
`06-operations.md` that **`main` is what the installer reads** — there is no staging.

## Other low-risk starting areas

- [fact] **Wordmark width mismatch** — `README.md:6` says `width="640"`, `onboard-me/README.md:6`
  says `565`. `da19904` enlarged only the root. Cosmetic, zero runtime impact, one-character-ish fix.
- [fact] **Prose in `references/`** — `synthesis.md` and `repo-playbooks.md` are loaded on demand, so
  edits there affect only the stage that reads them.
- [fact] **A whole new skill** — the process is documented at `README.md:140-147`. Higher effort, but
  it cannot affect `onboard-me` at all, since skills are independent folders.

## What to avoid on a first change

- [fact] **Editing `SKILL.md:3-10`, the `description`.** It is the sole trigger surface
  (`README.md:128-130`) and its failure mode is a skill that silently stops firing.
- [fact] **Renaming anything.** Three places must move together, one of them off-disk — see
  `05-dependencies.md`. Installed junctions/symlinks break invisibly.
- [fact] **Renaming a `.kt/` file or an evidence tag.** Both drift silently against
  `study-page-template.html`, with no error to catch it.

## Open unknowns

- [unknown] Whether the author wants PRs or prefers issues first. `README.md:140` says "Issues and
  pull requests are welcome" but there is no `CONTRIBUTING.md`, no PR template, and no PR history to
  read a convention from.
