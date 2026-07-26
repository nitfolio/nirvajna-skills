# claude-skills-repo

A collection of [Claude Code](https://docs.claude.com/en/docs/claude-code) skills.

Each skill is a self-contained folder with a `SKILL.md` (and optional `references/`, `scripts/`, or
`assets/`). Drop a skill into your Claude Code skills directory and it becomes available in your
sessions.

## Skills

| Skill | What it does |
| --- | --- |
| [`codebase-onboarding`](codebase-onboarding) | Guided, evidence-based knowledge transfer for an unfamiliar codebase. Explores the repo one stage per turn, tags every claim with its evidence, keeps an honest map of what's still unknown, and proposes the next step — you steer with one-word replies. Adapts to 13 repo types, leaves a resumable `.kt/` trail, and finishes with a curated onboarding document plus a self-contained study page. Say `speedrun` to run the whole ladder without stopping. |

## Installing a skill

Copy the skill's folder into one of these locations:

```bash
# Personal — available in all your projects
~/.claude/skills/<skill-name>/

# Project-level — checked into a repo, shared with everyone who clones it
<your-repo>/.claude/skills/<skill-name>/
```

For example, for `codebase-onboarding`:

```bash
cp -r codebase-onboarding ~/.claude/skills/
```

Copy the whole folder, not just `SKILL.md` — skills that use `references/` load those files at
runtime and will be missing capability without them.

Then start Claude Code. Skills trigger on their own when relevant, or you can invoke them
explicitly. See each skill's own `README.md` for usage details.

### Using these on claude.ai or Cowork

Zip the skill folder (or package it as a `.skill` file) and upload it under
Settings → Capabilities → Skills.

## Repository layout

```
claude-skills-repo/
├── README.md
├── LICENSE
└── codebase-onboarding/
    ├── SKILL.md            ← the skill: name, description, and instructions
    ├── README.md           ← human-facing usage docs
    └── references/         ← loaded on demand, only when a run needs them
        ├── repo-playbooks.md
        ├── synthesis.md
        └── study-page-template.html
```

`SKILL.md` frontmatter carries the `name` and `description`. The description is what makes the skill
trigger, so it's written as a list of the situations that should reach for it — keep it specific
when you edit one.

## Contributing

Issues and pull requests are welcome. If you add a skill:

- Keep it self-contained in its own folder.
- Write the `SKILL.md` description as trigger conditions, not a summary of features.
- Push reference material that only some runs need into `references/`, so `SKILL.md` stays lean.
- Give any multi-stage process a checkable "done when" condition per stage — vague criteria are the
  most common reason a skill stops early.
- Add a `README.md` for humans, and a row to the table above.

## License

[MIT](LICENSE) — free to use, modify, and share.
