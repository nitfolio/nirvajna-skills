# 03 · Domain glossary

Two vocabularies overlap here. **Platform terms** come from Claude Code and constrain what the repo
may look like. **Method terms** are invented by this skill and constrain what a KT session does.
Getting them confused is the main way a newcomer misreads the repo.

## Platform terms (Claude Code defines these)

| Term | One-line meaning | Lives in |
| --- | --- | --- |
| **Skill** | A self-contained folder Claude Code loads on demand — `SKILL.md` plus optional `references/`, `scripts/`, `assets/`. No plugin, no config, no runtime. | `README.md:27-29` |
| **`SKILL.md`** | The one required file. Its body is the instructions; it is loaded in full whenever the skill triggers. | `onboard-me/SKILL.md` |
| **Frontmatter** | The YAML block at the top of `SKILL.md`, carrying `name` and `description`. | `SKILL.md:1-11` |
| **`name`** | The skill's identifier. Must equal the containing directory name. | `SKILL.md:2` |
| **`description` / trigger surface** | The *only* thing that decides whether a skill activates — so it is written as a list of situations, not a feature summary. | `SKILL.md:3-10`; rule stated at `README.md:128-130` |
| **`references/`** | Files loaded lazily, only when a run reaches the stage that needs them. | `onboard-me/references/` |
| **Progressive disclosure** | The design rule: keep `SKILL.md` lean enough to stay in context all session, push heavy material into `references/`. | `README.md:132-134` |

## Method terms (this skill defines these)

| Term | One-line meaning | Lives in |
| --- | --- | --- |
| **KT** | Knowledge transfer — a guided session that moves someone from "I know nothing" to "I can safely make a change". | `SKILL.md:13`, `:22-23` |
| **The core loop** | The fixed five-beat shape of every turn: DISCOVER → EXPLAIN → ASSESS → PROPOSE → CONFIRM. | `SKILL.md:52-58` |
| **The ladder** | The seven-stage default exploration order, from Orientation to Safe contribution. | `SKILL.md:160-180` |
| **Stage** | One rung of the ladder. Exactly one is covered per turn (except in a speedrun). | `SKILL.md:49-50` |
| **Completion criterion** | The checkable condition that must hold before a stage may be called done — the guard against declaring "Architecture complete" after a folder listing. | `SKILL.md:155-158` |
| **Evidence tag** | The label on every non-obvious claim: `[fact]`, `[inference]`, `[unknown]`, `[human]`. | `SKILL.md:74-82` |
| **`[fact]`** | Directly supported by something read, *with a citation*. No citation → not a fact. | `SKILL.md:74-75` |
| **`[inference]`** | A reasonable deduction, not confirmed; must state what it rests on. | `SKILL.md:76-77` |
| **`[unknown]`** | Couldn't determine it. Treated as valuable output, not failure. | `SKILL.md:78-79` |
| **`[human]`** | The engineer said so — the strongest evidence in the room, treated as settled. | `SKILL.md:80-82` |
| **Fog of war** | The metaphor for the map: what's lit vs what's still dark. Keeping it honest is "the skill's whole job". | `SKILL.md:25-28` |
| **Zone of proximal development** | How the next step is chosen — just beyond current knowledge and reachable from it, not the most interesting thing found. | `SKILL.md:60-63` |
| **The `.kt/` trail** | The directory of findings written as stages complete, so the work outlives the chat. | `SKILL.md:190-208` |
| **Working trail (`00`–`07`)** | Raw, evidence-tagged, unpolished record of the learning. | `SKILL.md:210-213` |
| **Deliverable (`08-onboarding.md`)** | The clean reader-facing document, written only at `stop`. | `SKILL.md:212-213` |
| **`00-progress.md`** | The resumability file — kept current every turn so a fresh session can pick up exactly where the last stopped. | `SKILL.md:220-237` |
| **Repo-type playbook** | A per-repo-kind adaptation of the ladder: recognition signals, ladder emphasis, files to read first, must-answer questions, traps. 13 types + a generic fallback. | `references/repo-playbooks.md` |
| **Coverage check** | The `stop`-time guard that refuses to polish a barely-explored repo into a confident document. | `references/synthesis.md:6-14` |
| **Confidence filter** | The `stop`-time sort: promote `[fact]`/`[human]`, drop guesses, carry surviving `[unknown]`/`[inference]` into an explicit "Assumptions & things to verify" section. | `references/synthesis.md:16-26` |
| **Study page** | `onboarding.html` — one self-contained offline page bundling `00`–`08`, with per-file copy buttons. | `references/synthesis.md:45-64` |
| **Speedrun** | A standing `continue`: the whole ladder end to end with no per-turn gate, same rigor, *stricter* read-only. | `SKILL.md:271-302` |

## The controls

[fact] Ten single words drive a session, listed once at `SKILL.md:252-262` and mirrored for humans at
`onboard-me/README.md:115-126`:

`start` · `continue`/`yes` · `speedrun` · `deeper` · `skip` · `jump to <topic>` · `why` ·
`summarize` · `pause` · `stop`

[fact] `pause` and `stop` are deliberately different promises — `pause` bookmarks and generates
nothing; `stop` produces the deliverable (`SKILL.md:304-318`).

## How the terms relate

```
Skill (platform)
  └── SKILL.md ── description ──> triggers the session
                    │
                    └── the method:
                          core loop  ──runs once per──>  stage
                                                          │
                                          stage ──gated by──> completion criterion
                                          stage ──adapted by──> repo-type playbook
                                          stage ──emits──>  evidence-tagged claims
                                                                  │
                                                      claims ──written to──> .kt/ working trail (00–07)
                                                                                    │
                                                        at `stop`: coverage check ──┤
                                                                confidence filter ──┤
                                                                                    v
                                                              08-onboarding.md + onboarding.html
```

[fact] The dependency runs one way: tags feed the trail, the trail feeds the filter, the filter feeds
the deliverable. Nothing downstream can promote a claim the tag didn't license — that is the whole
trust mechanism.

## Naming

[fact] The repo is named `nirvajna-skills` (`c09418a` renamed it from `claude-skills-repo`). No file
in the repo defines, explains, or otherwise uses the word "nirvajna" — a full-text search finds it
only in the repo/remote name and URLs.

[fact] The brand is "Oops!... AI Did It Again", carried by the wordmark alt text (`README.md:6`) and
the site link `oopsaididitagain.com` (`README.md:16`).

## Open unknowns

- [unknown] What "nirvajna" means or why it was chosen — not answerable from the repo.
