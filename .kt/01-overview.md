# 01 · Overview

## What this system is

[fact] A public repository of **Claude Code skills**. `README.md:11` states the pitch: "Skills that
make Claude Code do the tedious part properly." It currently ships exactly one skill, `onboard-me`
(badge at `README.md:14`, confirmed against `git ls-files`).

[fact] `README.md:27-29` defines the unit of work: a skill is "a self-contained folder — a `SKILL.md`
plus optional `references/`, `scripts/`, or `assets/` — that Claude Code loads when the situation
calls for it. No plugin, no config, no runtime."

## Stack

[fact] **There is no code.** 12 tracked source files: 6 Markdown/HTML documents, 4 PNGs, `LICENSE`,
and `.gitignore`. (22 tracked in total once the `.kt/` trail this session produced was committed.)

[fact] Absent, each checked individually: `package.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`,
`Makefile`, `.github/workflows`, `.github/`. There is no build, no test suite, no CI, no lockfile,
and no dependency manifest of any kind.

[inference] The "runtime" is Claude Code itself. The repo is inert text; behaviour only exists when
an agent loads `SKILL.md` into its context. Based on `README.md:27-29` and the fact that nothing in
the tree is executable.

## Entry points

[fact] `onboard-me/SKILL.md:1-11` — the YAML frontmatter, and the only true entry point:

- `name: onboard-me` (`:2`) must match the containing directory name.
- `description:` (`:3-10`) is the **trigger surface**. `README.md:128-130` is explicit: "The
  description is the only thing that makes a skill trigger", which is why it is written as a list of
  situations rather than a feature summary.

[fact] Secondary, human-facing entry points: `README.md` (repo front door) and
`onboard-me/README.md` (skill usage docs). Neither is loaded by the agent at runtime.

## Repo map

```
nirvajna-skills/
├── README.md                 164 L   repo front door: pitch, skills table, install, layout
├── LICENSE                          MIT (README.md:13)
├── .gitignore                  2 L   two comment lines, ZERO patterns
├── assets/                           4 PNGs: wordmark + logo, each light/dark
└── onboard-me/                       the one skill
    ├── SKILL.md              347 L   always loaded — the method
    ├── README.md             426 L   human docs — not loaded by the agent
    └── references/                   loaded on demand only
        ├── repo-playbooks.md        360 L
        ├── study-page-template.html 368 L
        └── synthesis.md              64 L
```

[fact] Line counts measured with `wc -l` on 2026-07-27. Always-loaded weight is 347 lines; on-demand
weight is 792 lines — a ~1:2.3 split. This is the progressive-disclosure design `README.md:132-134`
describes.

[fact] `.gitignore` contains no ignore patterns at all — both its lines are comments explaining that
`.kt/` is committed on purpose as a worked example of the skill's output.

[fact] `.claude/settings.local.json` exists but is untracked, and it is ignored by the *user's global*
gitignore (`~/.config/git/ignore:1`), not by this repo. Top-level key: `permissions`. Not part of the
project.

## History

[fact] 21 commits, 2026-07-08 → 2026-07-27, one author (`nitfolio <ntv.nitin@gmail.com>`).
Remote: `https://github.com/nitfolio/nirvajna-skills.git`. Single branch `main`.

[fact] The history is dominated by two renames, which is worth knowing before reading old commits:

- the skill: `codebase-kt` → `codebase-onboarding` (`d52f691`) → `onboard-me` (`00b7dd3`)
- the repo: `claude-skills-repo` → `nirvajna-skills` (`c09418a`)

[fact] `9a77940` "Remove codebase-kt.skill" and `ce43ad1` "Fix Claude.ai install instructions after
removing codebase-kt.skill" — a packaged `.skill` bundle used to be committed and was deliberately
removed.

## Open unknowns

- [unknown] Whether anyone outside the author uses this — no stars/forks/issues data gathered.
