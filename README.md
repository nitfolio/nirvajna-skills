<div align="center">

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="assets/wordmark-dark.png">
  <source media="(prefers-color-scheme: light)" srcset="assets/wordmark-light.png">
  <img src="assets/wordmark-light.png" alt="Oops!... AI Did It Again" width="640">
</picture>

# nirvajna-skills

**Skills that make [Claude Code](https://docs.claude.com/en/docs/claude-code) do the tedious part properly.**

[![MIT License](https://img.shields.io/badge/License-MIT-4A86D8?style=flat-square&labelColor=141312)](LICENSE)
[![Skills](https://img.shields.io/badge/Skills-2-FF4A17?style=flat-square&labelColor=141312)](#-skills)
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

### [`onboard-me`](onboard-me)

`knowledge transfer`

</td>
<td valign="top">

Guided, evidence-based knowledge transfer for an unfamiliar codebase. Explores the repo **one stage
per turn**, tags every claim with its evidence, keeps an honest map of what's still unknown, and
proposes the next step — you steer with one-word replies.

Adapts to 13 repo types, leaves a resumable trail in `.kt/onboard/`, and finishes with a curated
onboarding document plus a self-contained HTML study page. Say `speedrun` to run the whole ladder
without stopping.

</td>
</tr>
<tr>
<td width="220" valign="top">

### [`offboard-me`](offboard-me)

`knowledge capture`

</td>
<td valign="top">

The inverse: capture what leaves with a departing engineer. Scans the repo to work out what **only
they** can answer — sole authorship, uncommented magic values, marked workarounds, unmerged branches —
then interviews them one area per turn against whatever time is left.

Never asks a bare question: it offers its own reading first, so their job is to confirm or correct
rather than recall. Leaves a resumable trail in `.kt/offboard/`, and ends with a handover document
that **leads with what nobody could explain** instead of burying it.

</td>
</tr>
</table>

## ✦ Install

**1. Add the skills.** The one-liner, from inside the project you want them in:

```bash
npx skills@latest add nitfolio/nirvajna-skills
```

That shows you what's in the repo and lets you pick. For one skill without the prompt, name it:

```bash
npx skills@latest add nitfolio/nirvajna-skills --skill onboard-me
npx skills@latest add nitfolio/nirvajna-skills --skill offboard-me
```

Either uses the [`skills` CLI](https://github.com/vercel-labs/skills). It keeps one canonical copy in
`.agents/skills/` and symlinks it into `.claude/skills/` — plus every other agent directory it finds,
so Codex, Cursor, Gemini CLI and the rest pick it up from the same source. Add `-g` to install for
your user instead of the project, or `--copy` if you'd rather have real files than symlinks.

<details>
<summary><b>Or copy the folder by hand</b></summary>

<br>

The whole folder, not just `SKILL.md`:

```bash
# Personal — available in all your projects
cp -r your-skill-name ~/.claude/skills/

# Project-level — checked into a repo, shared with everyone who clones it
cp -r your-skill-name <your-repo>/.claude/skills/
```

</details>

**2. Start Claude Code.** That's the whole install.

**3. Use it.** Invoke a skill by name:

```text
/your-skill-name
```

`onboard-me` and `offboard-me` are both invoke-by-name only (`disable-model-invocation: true`) — a
long KT session or a departure capture is something you opt into, not something that should fire from
an offhand sentence. A skill without that flag can also trigger on its own when the situation matches
its description.

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
nirvajna-skills/
├── README.md
├── LICENSE
├── assets/                     ← brand marks used by this README
├── onboard-me/
│   ├── SKILL.md                ← the skill: name, description, and instructions
│   ├── README.md               ← human-facing usage docs
│   └── references/             ← loaded on demand, only when a run needs them
│       ├── repo-playbooks.md         ← 13 repo-type playbooks + generic fallback
│       ├── repo-edge-cases.md        ← no git history, uploaded/read-only repos, monorepos, generated code
│       ├── synthesis.md              ← how the final onboarding doc gets built
│       └── study-page-template.html  ← the shell for the `onboarding.html` study page
├── offboard-me/
│   ├── SKILL.md
│   ├── README.md
│   └── references/
│       ├── risk-signals.md             ← 11 scan signals, the traps, and how to rank
│       ├── synthesis.md                ← how the handover document gets built
│       └── handover-page-template.html ← the shell for the `handover.html` page
├── your-skill-3/               ← each new skill is a sibling folder, same shape
└── your-skill-n/
```

Each skill writes its findings to its own folder under `.kt/` in the repo you point it at, so two
sessions never collide and neither skill needs the other to have run:

```text
.kt/
├── onboard/    ← 00-progress.md … 08-onboarding.md + onboarding.html
└── offboard/   ← INDEX.md + one <name>-<date>/ subfolder per capture, so a second departure
                  next year doesn't overwrite the first (00-risk-register.md, 01-tribal-knowledge.md,
                  02-handover.md + handover.html, per subfolder)
```

<details>
<summary><b>Why the split matters</b></summary>

<br>

`SKILL.md` frontmatter carries the `name` and `description`, which together are what makes a skill
trigger on its own — so the description is written as a list of the situations that should reach for
it, not as a feature summary. Keep it specific when you edit one. Set `disable-model-invocation: true`
on a skill that should only ever start by explicit name (a long, stateful session isn't something to
trip into from an offhand sentence); the description still documents when to reach for it, it just no
longer triggers that on its own.

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
