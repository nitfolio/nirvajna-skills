# 05 · Dependencies & blast radius

## External dependencies

**[fact]** **Zero package dependencies.** No manifest, no lockfile, no vendored code, no CDN
reference. `study-page-template.html` is fully self-contained (see `.kt/04-key-flows.md`, Flow C).

The real dependencies are runtime and social, not packaged:

| Dependency | Nature | Failure behaviour |
|---|---|---|
| **Claude Code (or Claude.ai / Cowork)** | The host that loads and executes `SKILL.md`. The only "runtime". | **[fact]** If frontmatter parsing or skill discovery changes upstream, the skill silently never triggers — no error surfaces. `README.md:110-112` |
| **Skill-discovery convention** (`~/.claude/skills/<name>/`) | Install contract, `README.md:60-66` | **[inference]** Wrong path = skill absent with no diagnostic. The README's remedy is the manual check at `onboard-me/README.md:65-67`: ask *"what skills do you have available?"* |
| **The agent's own tools** (`ls`, `git`, grep, file read) | Every DISCOVER step assumes them, `SKILL.md:92-112` | **[fact]** `SKILL.md:113-128` covers degraded cases explicitly: no git history, read-only/uploaded repo, huge monorepo, vendored dirs. |
| **The human's replies** | The controls, `SKILL.md:247-265` | **[fact]** `speedrun` removes this dependency for the duration of a run — hence the *stricter* read-only rule at `SKILL.md:~292`. |
| **`assets/*.png` via raw.githubusercontent.com** | `onboard-me/README.md` bottom uses absolute raw URLs | **[inference]** Deliberate — that README renders correctly when the folder is copied out of the repo. The root `README.md:139-141` uses relative paths instead. |
| **[reference]** `mattpocock/skills` | Cited influence, `onboard-me/README.md:376-379` | Credit only; nothing is imported. |

## Blast radius — "if I change X, what breaks?"

- **`SKILL.md` frontmatter `description`** — **highest blast radius in the repo.** **[fact]** It is
  the *only* thing that makes the skill trigger (`README.md:110-112`). Weaken it and the skill
  becomes dead code that nobody notices, because nothing errors.
- **`SKILL.md` body** — always in context, so every added line costs budget for the whole run.
  **[inference]** Growth here is the main pressure the `references/` split exists to relieve
  (`README.md:114-116`); moving material *out* is safe, adding material *in* is the expensive
  direction.
- **A "Done when" criterion** — **[fact]** `onboard-me/README.md:330-331`: "If you add
  [a stage], give it a completion criterion — a stage without one gets declared done early."
  Deleting one silently shortens every future run.
- **`references/repo-playbooks.md`** — lowest blast radius. Playbooks are independent; adding #15
  affects only repos of that type. **[fact]** The one coupling is the Contents list at
  `repo-playbooks.md:13-31`, which must be updated alongside (`onboard-me/README.md:326-328`).
- **`references/synthesis.md`** — affects only the final turn, but that's the turn that produces the
  artifact that outlives the session. A weakened coverage check (`synthesis.md:6-14`) reintroduces
  precisely the failure the whole skill exists to prevent.
- **`references/study-page-template.html`** — **[fact]** the slot IDs
  (`data-id`/`data-label`/`data-group`, lines 170-204) are a contract with `synthesis.md:52-53` and
  with the `.kt/` filenames in `SKILL.md:194-205`. Renaming a `.kt/` file means editing **three**
  files, and nothing checks the invariant.
- **`.gitignore`** — in most repos this should ignore `.kt/`, or every KT run starts committing its
  working trail into the user's repo. **This** repo deliberately does the opposite and tracks `.kt/`
  as a worked example.

## Known drift (this repo, today)

- **[fact]** `onboard-me/README.md:62` still instructs users to "Upload
  `onboard-me.skill`". No such file exists — the packaged bundle was deleted in `9a77940`,
  and commit `ce43ad1` fixed the *root* README's wording but not this one. Stale instruction.
- **[fact]** HEAD (`6e83541`) is titled "wip: readme preview — brand hero, badges, **mermaid
  ladder**", but `README.md` contains no mermaid block (`grep -n mermaid README.md` → no match).
  **[inference]** The ladder diagram was dropped or not yet added; the commit is marked `wip:`, so
  this is likely in-flight rather than a bug.
- **[fact]** Counts are duplicated in three places and currently agree: `README.md:14` badge
  (`Skills-1`), `README.md:100` and `onboard-me/README.md:344` ("13 repo-type playbooks +
  generic fallback"), matching the 14 `##` headings in `repo-playbooks.md`. **[inference]** Nothing
  enforces this; adding a playbook or a skill means hand-editing every copy.
