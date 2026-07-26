# 01 · Overview

## What this system is

**[fact]** `nirvajna-skills` is a **distribution repo for Claude Code skills** — a curated
collection of prompt-engineering artifacts, not a program. `README.md:11` states the thesis:
"Skills that make Claude Code do the tedious part properly."

**[fact]** It contains **zero executable source code**. All 12 tracked files (`git ls-files`) are
markdown, one HTML template, four PNGs, a LICENSE, and a `.gitignore`. Total 1,683 lines of text
across the 5 content files.

**[fact]** There is exactly **one skill** in the repo today: `codebase-onboarding/`. The README
badge at `README.md:14` hardcodes `Skills-1`.

## Stack

**[fact]** No stack in the usual sense — no package manifest (`package.json`, `pyproject.toml`,
`go.mod`, …), no build step, no dependency lockfile, no tests, no CI (`.github/` does not exist),
no Dockerfile. `README.md:29` says this explicitly: "No plugin, no config, no runtime."

**[fact]** The only code in the repo is inside `codebase-onboarding/references/study-page-template.html`
— vanilla browser JavaScript, no external libraries (`study-page-template.html:206-352`).

## Entry points

Because the "runtime" is Claude Code itself, entry points are load points, not `main()`s:

- **[fact]** `codebase-onboarding/SKILL.md:1-11` — YAML frontmatter (`name`, `description`). This is
  the true entry point: Claude Code reads the `description` to decide whether to load the skill.
- **[fact]** `codebase-onboarding/SKILL.md:12+` — the skill body, loaded into context on trigger.
- **[fact]** `codebase-onboarding/references/*` — loaded lazily, only when the skill body says to
  read them.
- **[fact]** `README.md:60-66` — the human entry point: `cp -r codebase-onboarding ~/.claude/skills/`.

## Repo type

**[inference]** Closest match is **playbook 14, generic fallback** (`references/repo-playbooks.md:347`)
— a content/prompt repo fits none of the 13 named types. Classification is therefore **soft**; the
ladder was adapted, with "state" read as *the artifacts a run writes* and "boundaries" read as *the
host agent runtime*.

**[fact]** This is a **self-referential KT**: the skill being explored is the same skill running the
exploration. The installed copy at `~/.claude/skills/codebase-onboarding` is **byte-identical** to
the repo copy (verified with `diff -r`, no differences).

## History & ownership

**[fact]** 14 commits, **a single author** (`nitfolio`) — `git log --format='%an' | sort | uniq -c`.
**[fact]** The project was originally named `codebase-kt` and renamed in `d52f691`
("Rename codebase-kt to codebase-onboarding, add references and READMEs"). Older commits reference
`codebase-kt/` paths and a packaged `codebase-kt.skill` bundle that was later removed (`9a77940`).
**[fact]** HEAD is `6e83541`, a `wip:` commit — the working tree is clean.
