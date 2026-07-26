# 07 · Safe contribution

## The house rules (non-negotiable, and short)

**[fact]** `README.md:120-129` states them:

1. Keep each skill **self-contained** in its own folder.
2. Write the `SKILL.md` description as **trigger conditions**, not a feature summary.
3. Push material only *some* runs need into `references/`, so `SKILL.md` stays lean.
4. Give any multi-stage process a checkable **"done when"** per stage.
5. Add a `README.md` for humans, and a row to the README table.

## Good first changes, lowest risk first

**Tier 1 — no behavioural risk at all**

- **Fix the stale install line.** `onboard-me/README.md:62` tells users to upload
  `onboard-me.skill`, which no longer exists (deleted in `9a77940`). The root README's
  correct wording is at `README.md:81-83` — mirror it. Human-docs only; the agent never reads this
  file.
- **Resolve the `wip:` mermaid gap.** HEAD claims a "mermaid ladder" in the README that isn't there.
  Either add the diagram or amend the intent.

**Tier 2 — additive, isolated**

- **Add a 15th repo-type playbook.** `onboard-me/README.md:326-328` gives the recipe: copy
  an existing playbook, keep the five-part shape (recognition signals · ladder emphasis · files to
  read first · must-answer questions · traps), then **add it to the Contents list at
  `repo-playbooks.md:13-31`** and update the two "13 repo-type playbooks" strings
  (`README.md:100`, `onboard-me/README.md:344`). Blast radius: repos of that type only.

**Tier 3 — touches always-loaded context; review carefully**

- **Editing `SKILL.md`.** Every line is loaded for every run. If you add a ladder stage, you must
  give it a completion criterion (`onboard-me/README.md:330-331`), add its `.kt/` file to
  the layout at `SKILL.md:194-205`, add a slot to `study-page-template.html:170-204`, and mention it
  in `synthesis.md`. Four files.
- **Editing the frontmatter `description`.** Highest blast radius in the repo — see
  `.kt/05-dependencies.md`.

## How to run and verify a change

There is no test command to run. **[fact]** Verification is manual and behavioural:

```bash
# 1. Reinstall the edited folder
cp -r onboard-me ~/.claude/skills/

# 2. Confirm the copy actually landed (should print nothing)
diff -r onboard-me ~/.claude/skills/onboard-me

# 3. Restart Claude Code, then ask:
#      "what skills do you have available?"        → onboard-me must be listed
#      "walk me through this repo"                 → must trigger WITHOUT being named
#
# 4. Exercise the path you changed:
#      /onboard-me            → one stage, then it must STOP and wait
#      /onboard-me speedrun   → full ladder, then 08-onboarding.md + onboarding.html
#
# 5. Inspect the output, then reset:
open .kt/onboarding.html   # (macOS) — start .kt\onboarding.html on Windows
rm -rf .kt                 # NOTE: .kt/ is TRACKED in this repo - use git restore, not rm
```

**[fact]** Step 2's `diff -r` is a real check — it was run during this session and confirmed the
repo and installed copies were byte-identical.

**[inference]** The highest-value check is step 3's *unnamed* trigger. A change that only ever gets
tested via `/onboard-me` will never catch a broken `description`, which is the one failure
mode that fails silently.

## What not to do

**[fact]** `SKILL.md:130-148` binds the skill itself to read-only behaviour when exploring someone
else's repo: no source/config edits, no state-mutating commands, and **no running builds, tests, or
scripts** without the human's decision. When contributing *to* this repo, that rule is the product —
weakening it is the change most likely to make the skill harmful in the field.
