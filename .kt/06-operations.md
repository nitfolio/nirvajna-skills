# 06 · Operations

## Build

**[fact]** There is none. No build script, no bundler, no generated output, no `dist/`. The source
files *are* the artifact. `README.md:29`: "No plugin, no config, no runtime."

## Deploy

**[fact]** "Deploy" = copy a folder (`README.md:58-66`, `onboard-me/README.md:45-58`):

```bash
cp -r onboard-me ~/.claude/skills/          # personal, all projects
cp -r onboard-me <your-repo>/.claude/skills/ # project-level, checked in
```

Then restart Claude Code (`onboard-me/README.md:56-57`).

**[fact]** The whole folder must be copied, not just `SKILL.md` — `README.md:58` and `README.md:84-86`
both warn that a skill missing `references/` "will be missing capability."

**[fact]** For Claude.ai / Cowork: zip the folder and upload under Settings → Capabilities → Skills
(`README.md:81-86`). See the stale `.skill` instruction noted in `.kt/05-dependencies.md`.

## Configuration

**[fact]** No config files. The only tunable surface is the prose itself —
`onboard-me/README.md:324-335` lists the four supported customizations: add a playbook,
change the ladder, change the `.kt/` layout, tune the house style.

**[fact]** `.gitignore` is the sole repo-level setting, and it is a comment only: `.kt/` is
**deliberately tracked** here as a worked example of the skill's output. In a normal consumer repo
`.kt/` would be ignored instead.

## Verification / test story

**[fact]** **There is no automated test or CI of any kind** — no `.github/`, no linter, no
link-checker, no markdown validation. This is the repo's most notable operational gap.

**[fact]** The only prescribed verification is manual and in-band
(`onboard-me/README.md:65-67`): ask *"what skills do you have available?"* and confirm
`onboard-me` is listed.

**[inference]** The de facto test suite is *running the skill on a real repo and reading the
output* — which is exactly what this session is. Nothing catches a broken cross-reference, a stale
line-number citation, or a drifted count until a run trips over it.

## Fragile spots

1. **Silent trigger failure.** **[fact]** Only the `description` triggers the skill
   (`README.md:110-112`) and nothing reports a non-trigger. A weakened description degrades the
   product invisibly.
2. **Un-enforced cross-file contracts.** **[fact]** `.kt/` filenames appear in `SKILL.md:194-205`,
   `synthesis.md:52-53`, and as `data-id` slots in `study-page-template.html:170-204`. Three copies,
   zero checks.
3. **Hardcoded counts drift.** See `.kt/05-dependencies.md` — badge and two prose counts, all
   hand-maintained.
4. **Context budget.** **[inference]** `SKILL.md` at 347 lines is always loaded. Every future rule
   added to it competes with the exploration itself for context; the `references/` escape hatch has
   to be used deliberately or the design erodes.
5. **[unknown]** Whether the skill behaves correctly in the Claude.ai/Cowork upload path (read-only
   mount, no `.git/`, `.kt/` relocation — `SKILL.md:118-121`). The logic is written; there is no
   evidence in the repo that it was exercised.
