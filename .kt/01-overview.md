# 01 · Overview

*Written against commit `2e94f1b` on 2026-07-27. Every `file:line` below is a line number **at that commit** — check that commit out to verify a claim, or re-run the skill to regenerate the trail against current source.*

## What this system is

[fact] A public repository of **Claude Code skills**. `README.md:11`: "Skills that make Claude Code
do the tedious part properly." It ships exactly one skill, `onboard-me` (badge at `README.md:14`,
confirmed against `git ls-files`).

[fact] `README.md:27-29` defines the unit: a skill is "a self-contained folder — a `SKILL.md` plus
optional `references/`, `scripts/`, or `assets/` — that Claude Code loads when the situation calls
for it. No plugin, no config, no runtime."

[fact] As of 2026-07-27 the repo is **also a published website** — GitHub Pages, status `built`,
serving `main` from the repo root at `https://nitfolio.github.io/nirvajna-skills/`, HTTPS enforced.
Its only purpose is to host the study page this trail produces.

## Stack

[fact] **There is no executable code.** 13 tracked source files — 6 Markdown/HTML documents, 4 PNGs,
`LICENSE`, `.gitignore`, and an empty `.nojekyll`. With the committed `.kt/` trail the total is 23.

[fact] Absent, each checked individually: `package.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`,
`Makefile`, `.github/`. No build, no test suite, no CI, no lockfile, no dependency manifest.

[inference] The "runtime" is Claude Code itself — the repo is inert text, and behaviour exists only
once an agent loads `SKILL.md` into context. Based on `README.md:27-29` plus nothing in the tree
being executable. No file states it outright.

## Entry points

[fact] `onboard-me/SKILL.md:1-11` — the YAML frontmatter, and the only true entry point:

- `name: onboard-me` (`:2`) must match the containing directory name.
- `description:` (`:3-10`) is the **trigger surface**. `README.md:132-134` is explicit: "The
  description is the only thing that makes a skill trigger", which is why it is written as a list of
  situations rather than a feature summary.

[human] Secondary, human-facing, and deliberately split by scope: `README.md` is the **front door for
a collection** — it describes how skills work here and is written generically (`your-skill-name`)
because more skills are planned. Per-skill documentation lives in that skill's own README. Neither is
loaded by the agent at runtime.

[inference] The single-skill framing in this trail is therefore time-limited. A later session will
find several skills and should classify the repo accordingly. Based on the author's stated plan; the
repo currently contains one skill.

[fact] A third, new entry point exists for readers rather than agents: the published study page at
`https://nitfolio.github.io/nirvajna-skills/.kt/onboarding.html`, linked from the PS at the end of
`onboard-me/README.md`.

## Repo map

```
nirvajna-skills/
├── README.md                 168 L   repo front door: pitch, skills table, install, layout
├── LICENSE                          MIT (README.md:13)
├── .gitignore                  2 L   two comment lines, ZERO patterns
├── .nojekyll                   0 L   empty; makes GitHub Pages publish .kt/
├── assets/                           4 PNGs: wordmark + logo, each light/dark
├── .kt/                             this trail — committed on purpose, and published
└── onboard-me/                       the one skill
    ├── SKILL.md              362 L   always loaded — the method
    ├── README.md             439 L   human docs — not loaded by the agent
    └── references/                   loaded on demand only
        ├── repo-playbooks.md        360 L
        ├── study-page-template.html 376 L
        └── synthesis.md              76 L
```

[fact] Line counts measured with `wc -l` on 2026-07-27 at commit `2e94f1b`. Always-loaded weight is
362 lines; on-demand weight is 812 lines — a ~1:2.2 split. This is the progressive-disclosure design
described at `README.md:136-138`.

[human] `.kt/` is committed here as a **showcase, not a recommendation**. The author keeps it in the
repo so anyone evaluating the skill can see the exact shape of its output before installing it. The
skill's own guidance is neutral — commit it or gitignore it, either works. Don't read this repo's
choice as the default.

[fact] `.gitignore` reflects that: no ignore patterns at all, both lines comments explaining that
`.kt/` is kept on purpose.

[fact] `.nojekyll` is empty (0 lines). Its existence is the whole content: it tells GitHub Pages to
skip Jekyll, which otherwise refuses to publish any path beginning with a dot — including `.kt/`.

[fact] `.claude/settings.local.json` exists but is untracked, and is ignored by the *user's global*
gitignore (`~/.config/git/ignore:1`), not by this repo. Top-level key: `permissions`. Not project
config.

## History

[fact] 30 commits, 2026-07-08 → 2026-07-27, one author (`nitfolio`). Remote
`https://github.com/nitfolio/nirvajna-skills.git`. Single branch `main`. Public: 0 stars, 0 forks,
0 open issues, 0 releases, 0 tags.

[fact] Two renames dominate the early history and will confuse anyone reading old commits:

- the skill: `codebase-kt` → `codebase-onboarding` (`d52f691`) → `onboard-me` (`00b7dd3`)
- the repo: `claude-skills-repo` → `nirvajna-skills` (`c09418a`)

[fact] `9a77940` "Remove codebase-kt.skill" — a packaged `.skill` bundle was once committed and was
deliberately removed. None has been tracked since.

[fact] The most recent ten commits are all from 2026-07-27 and concern this trail, the study page's
hosting, and the diagram rule. The repo's own tooling changed under the previous trail, which is why
this one was regenerated.

## Open unknowns

- [unknown] Whether the Pages site is discoverable at all. The site root returns 404 and the study
  page URL appears only inside the skill README's PS — nothing links it from the repo front door.
