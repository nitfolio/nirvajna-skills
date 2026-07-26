<div align="center">

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="assets/wordmark-dark.png">
  <source media="(prefers-color-scheme: light)" srcset="assets/wordmark-light.png">
  <img src="assets/wordmark-light.png" alt="Oops!... AI Did It Again" width="520">
</picture>

# claude-skills-repo

**Skills that make [Claude Code](https://docs.claude.com/en/docs/claude-code) do the tedious part properly.**

[![MIT License](https://img.shields.io/badge/License-MIT-4A86D8?style=flat-square&labelColor=141312)](LICENSE)
[![Skills](https://img.shields.io/badge/Skills-1-FF4A17?style=flat-square&labelColor=141312)](#-skills)
[![Claude Code](https://img.shields.io/badge/Claude_Code-ready-4A86D8?style=flat-square&labelColor=141312)](https://docs.claude.com/en/docs/claude-code)
[![Website](https://img.shields.io/badge/oopsaididitagain.com-FF4A17?style=flat-square&labelColor=141312)](https://oopsaididitagain.com/)

[Skills](#-skills) · [Install](#-install) · [Layout](#-repository-layout) · [Contributing](#-contributing) · [Website](https://oopsaididitagain.com/)

</div>

---

Dropping a new engineer — or a new agent — into an unfamiliar repo is expensive. They read the wrong
files first, mistake a guess for a fact, and produce a summary nobody can check.

A **skill** fixes that shape of problem once. It's a self-contained folder — a `SKILL.md` plus
optional `references/`, `scripts/`, or `assets/` — that Claude Code loads when the situation calls
for it. No plugin, no config, no runtime.

## ✦ Skills

<table>
<tr>
<td width="220" valign="top">

### [`codebase-onboarding`](codebase-onboarding)

`knowledge transfer`

</td>
<td valign="top">

Guided, evidence-based knowledge transfer for an unfamiliar codebase. Explores the repo **one stage
per turn**, tags every claim with its evidence, keeps an honest map of what's still unknown, and
proposes the next step — you steer with one-word replies.

Adapts to 13 repo types, leaves a resumable `.kt/` trail, and finishes with a curated onboarding
document plus a self-contained HTML study page. Say `speedrun` to run the whole ladder without
stopping.

</td>
</tr>
</table>

## ✦ Install

**1. Copy the folder** into your skills directory — the whole folder, not just `SKILL.md`.

```bash
# Personal — available in all your projects
cp -r codebase-onboarding ~/.claude/skills/

# Project-level — checked into a repo, shared with everyone who clones it
cp -r codebase-onboarding <your-repo>/.claude/skills/
```

**2. Start Claude Code.** That's the whole install.

**3. Use it.** Skills trigger on their own when relevant, or invoke one explicitly:

```text
/codebase-onboarding
```

<details>
<summary><b>Using these on claude.ai or Cowork</b></summary>

<br>

Zip the skill folder (or package it as a `.skill` file) and upload it under
**Settings → Capabilities → Skills**.

The `references/` folder must be inside the archive — skills load those files at runtime and will be
missing capability without them.

</details>

## ✦ Repository layout

```text
claude-skills-repo/
├── README.md
├── LICENSE
├── assets/                     ← brand marks used by this README
└── codebase-onboarding/
    ├── SKILL.md                ← the skill: name, description, and instructions
    ├── README.md               ← human-facing usage docs
    └── references/             ← loaded on demand, only when a run needs them
        ├── repo-playbooks.md         ← 13 repo-type playbooks + generic fallback
        ├── synthesis.md              ← how the final onboarding doc gets built
        └── study-page-template.html  ← the shell for the `onboarding.html` study page
```

<details>
<summary><b>Why the split matters</b></summary>

<br>

`SKILL.md` frontmatter carries the `name` and `description`. **The description is the only thing
that makes a skill trigger**, so it's written as a list of the situations that should reach for it —
not as a feature summary. Keep it specific when you edit one.

Everything in `references/` is loaded *on demand*. That keeps `SKILL.md` lean enough to stay in
context for the whole run, while the heavy material is one read away when a run actually needs it.
This is progressive disclosure, and it's the main reason a large skill stays reliable.

</details>

## ✦ Contributing

Issues and pull requests are welcome. If you add a skill:

- Keep it **self-contained** in its own folder.
- Write the `SKILL.md` description as **trigger conditions**, not a summary of features.
- Push reference material that only some runs need into `references/`, so `SKILL.md` stays lean.
- Give any multi-stage process a checkable **"done when"** condition per stage — vague criteria are
  the single most common reason a skill stops early.
- Add a `README.md` for humans, and a row to the table above.

## ✦ License

[MIT](LICENSE) — free to use, modify, and share.

<div align="center">
<br>

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="assets/logo-dark.png">
  <source media="(prefers-color-scheme: light)" srcset="assets/logo-light.png">
  <img src="assets/logo-light.png" alt="" width="40">
</picture>

**[oopsaididitagain.com](https://oopsaididitagain.com/)**

</div>
