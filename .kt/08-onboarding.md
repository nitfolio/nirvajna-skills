# Onboarding — `nirvajna-skills`

*Produced by a full-ladder KT session on 2026-07-27. All seven stages met their completion criteria;
the working trail with the underlying evidence is in `.kt/00-progress.md` through `07-safe-contribution.md`.*

---

## 1. What this system is

`nirvajna-skills` is a public repository of **Claude Code skills**. It ships exactly one: `onboard-me`,
a guided knowledge-transfer skill that walks an engineer through an unfamiliar codebase one stage at
a time, tagging every claim with its evidence.

The thing to internalise before reading anything: **there is no code here.** Thirteen tracked source
files — six Markdown/HTML documents, four PNGs, `LICENSE`, `.gitignore`, and an empty `.nojekyll`.
No build, no tests, no CI, no dependencies, no config. (The repo now also carries this KT trail in
`.kt/`, published as a static site via GitHub Pages.)
A skill is "a self-contained folder — a `SKILL.md` plus optional `references/`, `scripts/`, or
`assets/` — that Claude Code loads when the situation calls for it. No plugin, no config, no runtime"
(`README.md:27-29`). The repo is inert text; behaviour exists only when an agent loads it.

So the usual questions get unusual answers. *Architecture* means what gets loaded into context and
when. *Key flows* are paths through documents. *Operations* means that build, CI, and config are all
genuinely empty — and that `main` is production.

Single author, 21 commits, 2026-07-08 to 2026-07-27, public on GitHub with no stars, forks, issues,
tags, or releases.

---

## 2. Architecture map

The organising principle is **progressive disclosure**: 353 lines always loaded, 801 lines loaded
only when a run reaches the stage that needs them.

```
nirvajna-skills/
├── README.md          164 L   front door — pitch, install, layout      (no runtime role)
├── LICENSE                    MIT                                       (no runtime role)
├── .gitignore           2 L   two comments, ZERO patterns               (no runtime role)
├── .nojekyll            0 L   GitHub Pages marker — publishes .kt/      (no runtime role)
├── assets/                    4 PNGs, wordmark + logo, light/dark       (no runtime role)
└── onboard-me/                THE PRODUCT
    ├── SKILL.md       353 L   ── ALWAYS LOADED. The method.
    ├── README.md      426 L   ── humans only; nothing loads it at runtime
    └── references/            ── ON DEMAND
        ├── repo-playbooks.md         360 L
        ├── synthesis.md               65 L
        └── study-page-template.html  376 L
```

Modules "talk" through **a prose sentence naming a file path** — that is the entire dispatch
mechanism. There is no import graph, no config, no registry. Four edges, all originating in
`SKILL.md`, none pointing back:

```
                      SKILL.md
                (frontmatter = trigger)
                          |
        +-----------------+------------------+
        |                                    |
   :185 | at Orientation                :297 | at `stop`
        |                               :315 |
        v                                    v
 repo-playbooks.md                     synthesis.md
 (13 types + fallback)                       |
                                        :51  | "copy the template"
                                             v
                                   study-page-template.html
                                             |
                                             v
                                     .kt/onboarding.html
```

**Two couplings nothing enforces** — these are the interesting part of this architecture:

1. **The HTML template is hard-coded to the evidence vocabulary.**
   `study-page-template.html:218` regex-matches `[fact|inference|unknown|human]` to colour the tags.
   Those four words are defined at `SKILL.md:74-82`. Rename one and the study page silently stops
   highlighting it — no error, and no build step that would catch it.
2. **Unfilled slots hide themselves.** `study-page-template.html:314` drops any slot still containing
   its `<!-- PASTE … -->` comment, so a partial KT degrades gracefully.

One deliberate asymmetry: the root README references `assets/` by relative path, the skill README by
absolute `raw.githubusercontent.com` URL — because the `onboard-me/` folder is designed to be copied
*out* of the repo, where a relative path would resolve to nothing.

---

## 3. Domain glossary

Two vocabularies overlap. Confusing them is the main way a newcomer misreads the repo.

**Platform terms — Claude Code defines these, and they constrain what the repo may look like:**

| Term | Meaning | Home |
| --- | --- | --- |
| **Skill** | Self-contained folder Claude Code loads on demand | `README.md:27-29` |
| **`SKILL.md`** | The one required file; loaded whole when the skill triggers | `onboard-me/SKILL.md` |
| **`name`** | Skill identifier — must equal the containing directory name | `SKILL.md:2` |
| **`description` / trigger surface** | The *only* thing deciding whether a skill activates | `SKILL.md:3-10`, rule at `README.md:128-130` |
| **`references/`** | Files loaded lazily, only when a stage needs them | `onboard-me/references/` |
| **Progressive disclosure** | Keep `SKILL.md` lean; push heavy material into `references/` | `README.md:132-134` |

**Method terms — this skill invents these, and they constrain what a session does:**

| Term | Meaning | Home |
| --- | --- | --- |
| **KT** | Knowledge transfer: "I know nothing" → "I can safely make a change" | `SKILL.md:22-23` |
| **The core loop** | DISCOVER → EXPLAIN → ASSESS → PROPOSE → CONFIRM, once per turn | `SKILL.md:52-58` |
| **The ladder** | Seven stages, Orientation through Safe contribution | `SKILL.md:160-180` |
| **Completion criterion** | The checkable condition a stage must meet before being called done | `SKILL.md:155-158` |
| **Evidence tags** | `[fact]` (cited) · `[inference]` (reasoned, unconfirmed) · `[unknown]` (couldn't determine) · `[human]` (the engineer said so — strongest) | `SKILL.md:74-82` |
| **Fog of war** | The map of what's lit vs still dark; keeping it honest is "the skill's whole job" | `SKILL.md:25-28` |
| **The `.kt/` trail** | Findings written as stages complete, so work outlives the chat | `SKILL.md:190-208` |
| **Working trail (`00`–`07`)** | Raw, evidence-tagged, unpolished record | `SKILL.md:216-219` |
| **Deliverable (`08`)** | The clean reader-facing document, written only at `stop` | `SKILL.md:218-219` |
| **Repo-type playbook** | Per-repo-kind adaptation of the ladder; 13 types + generic fallback | `references/repo-playbooks.md` |
| **Coverage check** | The `stop` guard against polishing a barely-explored repo | `synthesis.md:6-14` |
| **Confidence filter** | Promote `[fact]`/`[human]`, drop guesses, carry the rest into an explicit section | `synthesis.md:16-26` |
| **Speedrun** | Standing `continue`: whole ladder, no gate, *stricter* read-only | `SKILL.md:277-308` |

Ten single words drive a session (`SKILL.md:258-268`): `start` · `continue` · `speedrun` · `deeper` ·
`skip` · `jump to <topic>` · `why` · `summarize` · `pause` · `stop`. `pause` and `stop` are different
promises — `pause` bookmarks and generates nothing; `stop` produces the deliverable.

The dependency runs one way: tags feed the trail, the trail feeds the filter, the filter feeds the
deliverable. Nothing downstream can promote a claim the tag didn't license. **That is the whole trust
mechanism.**

---

## 4. Key flows

### Flow A — install → trigger → first stage

| # | Waypoint | Where |
| --- | --- | --- |
| 1 | `npx skills@latest add nitfolio/nirvajna-skills` — one canonical copy in `.agents/skills/`, symlinked into `.claude/skills/` and other agent dirs | `README.md:61`, `:64-67` |
| 2 | The `description` matches what the user says — or `/onboard-me` bypasses matching | `SKILL.md:3-10`; `README.md:91` |
| 3 | **One** goal question, before any exploration; skipped if the opening message states the goal, never blocking if unanswered | `SKILL.md:30-45` |
| 4 | Classify the repo, then read **one** playbook | `SKILL.md:182-188`; `repo-playbooks.md:3-6` |
| 5 | Announce `.kt/` *before* creating it | `SKILL.md:250-251` |
| 6 | Core loop, one stage per turn, gated on the stage's completion criterion | `SKILL.md:52-58`, `:160-180` |

Exit: `pause` (bookmark, nothing generated) or `stop` (Flow B).

Manual install alternative: `cp -r onboard-me ~/.claude/skills/` — **the whole folder, not just
`SKILL.md`** (`README.md:74-81`), because `references/` must travel with it.

### Flow B — `stop` → two artifacts → rendered page

| # | Waypoint | Where |
| --- | --- | --- |
| 1 | Read `synthesis.md` before writing a word | `SKILL.md:319-324` |
| 2 | **Coverage check** — thin exploration must not become a polished document | `synthesis.md:6-14` |
| 3 | **Confidence filter** — promote `[fact]`/`[human]`, drop guesses, carry the rest | `synthesis.md:16-26` |
| 4 | Write `08-onboarding.md` to the six-part shape | `synthesis.md:33-41` |
| 5 | Fill the template: paste raw markdown into 9 slots, delete slots for missing files, set `<title>` and `.kt-repo-name`, **change nothing else** | `synthesis.md:52-59`; slots at `study-page-template.html:171-205`, placeholders at `:36` and `:147` |
| 6 | Browser render: collect slots `:311-314` → group and build cards `:321-338` → markdown parser `:208-307` → colour tags `:218` → copy buttons `:334`, `:344-345` → scroll-spy `:358` → live filter `:365` | `study-page-template.html` |
| 7 | Announce both outputs; the HTML opens straight in a browser | `synthesis.md:64-65` |

### The correction flow — short, and the most important

When the human corrects a claim, three things happen in the *same* turn (`SKILL.md:87-90`): retag it
as `[human]`, fix the affected `.kt/` file, and ask what else the correction invalidates. The stated
reason: "A mental model built on a stale assumption gets more wrong the longer it stands."

---

## 5. Your first change

**Add a repo-type playbook.** The repo asks for exactly this at `onboard-me/README.md:341-343`.

It is the safest real change because a new playbook section is inert until the Orientation classifier
selects it, and `SKILL.md:186-188` names the covered categories generically rather than by number —
so adding one cannot break an existing run. It also touches neither of the two silent couplings.

**Work in** `onboard-me/references/repo-playbooks.md`. Copy an existing playbook (e.g. `:130-152`)
and keep all five parts: `Recognize it` · `Ladder emphasis` · `Read first` · `The KT must answer` ·
`Traps`. Keep the generic fallback last — `:347` is numbered 14 because it is the "nothing fits" case.

**Then update six hand-synced places**, because nothing keeps them in agreement:

| File | Line |
| --- | --- |
| `onboard-me/references/repo-playbooks.md` | `13-28` (contents list) |
| `README.md` | `48` |
| `README.md` | `118` |
| `onboard-me/README.md` | `14` (badge URL) |
| `onboard-me/README.md` | `190-193` |
| `onboard-me/README.md` | `359` |

### How to verify

There is no test command — nothing in this repo builds, tests, or lints. Verification is a
consistency sweep plus one real run.

```bash
# 1 — structural checks, from the repo root
head -3 onboard-me/SKILL.md                                    # expect: name: onboard-me
grep -c '^## [0-9]' onboard-me/references/repo-playbooks.md    # expect: 14 (13 + fallback)
grep -rn 'repo-type playbooks\|repo types\|Repo_playbooks-' --include=*.md .
grep -c 'data-id=' onboard-me/references/study-page-template.html   # expect: 9

# the silent coupling — these two lists must name the same four tags
grep -n '^- \*\*\[' onboard-me/SKILL.md
grep -n 'fact|inference|unknown|human' onboard-me/references/study-page-template.html
```

```bash
# 2 — the real end-to-end test: run the skill
cp -r onboard-me <some-scratch-repo>/.claude/skills/
# then in Claude Code inside that repo:
/onboard-me
```

Confirm in order: it asks the goal question once, names the playbook it loaded, announces `.kt/`
before creating it, and on `stop` produces both `08-onboarding.md` and `onboarding.html`. A new
playbook is only exercised if the scratch repo actually *is* that type — pick the test repo to match,
or the change goes untested.

```bash
# 3 — check the study page renders (self-contained, no server)
start .kt/onboarding.html      # macOS: open   ·   Linux: xdg-open
```

**Commit convention:** imperative, sentence-case subject, no prefix — "Add a Rust workspace playbook".
Single branch `main`.

### What to avoid on a first change

- **Editing `SKILL.md:3-10`, the `description`.** Sole trigger surface; failure mode is a skill that
  silently stops firing.
- **Renaming anything.** Three things must move together — the directory, `SKILL.md:2`'s `name`, and
  every installed copy. Installed junctions/symlinks break invisibly and are undocumented in the repo.
- **Renaming a `.kt/` file or an evidence tag.** Both drift silently against the HTML template.

### Know before you push

**`main` is production and it deploys on push.** The install command carries no ref and every asset
URL is pinned to `/main/`. With 0 tags and 0 releases there is nothing to roll back to, and no CI
stands between a commit and every consumer.

Other easy starting points: the wordmark width mismatch (`README.md:6` says `640`,
`onboard-me/README.md:6` says `565`), prose in `references/`, or an entirely new skill folder
(`README.md:140-147`).

---

## 6. Assumptions & things to verify with a human

Everything above is drawn from `[fact]` claims with citations. These are not.

### Inferences — reasoned, not confirmed

- **The Library/SDK classification is soft.** It fits the shape (no server, explicit public surface,
  installed into other projects) but none of the signals (no registry metadata, `examples/`,
  CHANGELOG, or tests). The "library" is a prompt. Stages 4–6 leaned on the generic fallback's
  anchors instead.
- **The runtime is Claude Code itself.** Based on `README.md:27-29` plus nothing in the tree being
  executable. No file states it outright.
- **The relative-vs-absolute asset path split is deliberate**, because the skill folder is designed to
  be copied out of the repo. No comment says so.
- **Prose edits to `SKILL.md` fail soft** — the file is interpreted by a model, not parsed, so wording
  changes shift behaviour by degrees. The dangerous edits are structural. Based on there being no
  parser, schema, or test anywhere.
- **Bus factor 1** — 21 commits, one author, no CI, no reviewers.
- **`[human]` outranks `[fact]`** in the confidence filter. Implied by the tag definitions and the
  promotion rule; never stated explicitly.

### Unknowns — genuinely unresolved

- **Does Claude Code *enforce* `name:` == directory name, or merely convention it?** A platform
  behaviour; needs the Claude Code docs, not this repo.
- **Has `npx skills@latest add nitfolio/nirvajna-skills` ever run successfully against this repo?**
  The package is real (npm `skills` v1.5.20) and the command is well-formed, but nothing records a
  verified install.
- **What does "nirvajna" mean, and why was it chosen?** It appears only in the repo name and URLs —
  never in a document body. Ask the author.
- **Does branch protection or any GitHub-side setting exist?** Not visible from the repo.
- **How does the author package a `.skill` bundle?** No script is tracked, and no release asset has
  ever existed here.
- **PRs or issues first?** `README.md:140` welcomes both; there is no `CONTRIBUTING.md`, no PR
  template, and no PR history to infer a convention from.

### One caveat about this document

This KT ran as a `speedrun` — the full ladder with no per-turn human confirmation. Read-only was
enforced *more* strictly as a result: **no build, test, or script was executed at any point**, so
every command in section 5 is described rather than demonstrated. Nothing here has been checked by
running it.
