# 03 · Domain glossary

The domain is *knowledge transfer as a procedure*. Terms are load-bearing — the skill's rules refer
to them constantly.

| Term | One-line meaning | Where it lives |
|---|---|---|
| **Skill** | Self-contained folder (`SKILL.md` + optional `references/`) that Claude Code loads when the situation matches | `README.md:27-29` |
| **KT** | Knowledge transfer — one guided onboarding session over an unfamiliar repo | `SKILL.md:1-15` |
| **Fog of war** | The governing metaphor: the unlit part of the map is as valuable as the lit part; hiding it is the failure mode | `SKILL.md:20-24` |
| **The core loop** | DISCOVER → EXPLAIN → ASSESS → PROPOSE → CONFIRM; exactly one per turn | `SKILL.md:47-56` |
| **Evidence tag** | `[fact]` / `[inference]` / `[unknown]` / `[human]` — mandatory label on every non-obvious claim | `SKILL.md:68-84` |
| **`[human]`** | A claim the engineer supplied — explicitly the *strongest* evidence class, outranking `[fact]` | `SKILL.md:78-81` |
| **The ladder** | The 7 ordered stages: Orientation → Architecture → Domain → Key flows → Dependencies → Operations → Safe contribution | `SKILL.md:150-180` |
| **"Done when"** | Per-stage checkable completion criterion; a stage may not be declared complete without it | `SKILL.md:154-158` |
| **Zone of proximal development** | Rule for picking the next step: just beyond current knowledge and reachable from it — not the most interesting thing found | `SKILL.md:58-63` |
| **Repo type / playbook** | One of 13 classifications (+ generic fallback) that reorders the ladder for the repo at hand | `references/repo-playbooks.md:32-360` |
| **The `.kt/` trail** | `00`–`07`: the raw, evidence-tagged working record. Deliberately left unpolished | `SKILL.md:190-215` |
| **`00-progress.md`** | The trail's source of truth; updated *every* turn; what makes a session resumable | `SKILL.md:~225` |
| **The deliverable** | `08-onboarding.md` — the curated reader-facing doc, written **only** at `stop` | `references/synthesis.md:1-43` |
| **Study page** | `.kt/onboarding.html` — self-contained offline page bundling `00`–`08`, built from the template | `references/synthesis.md:45-64` |
| **Coverage check** | Guard at `stop`: if too few stages met their criteria, refuse to silently polish — offer three options instead | `references/synthesis.md:6-14` |
| **Confidence filter** | Sorting rule at synthesis: promote `[fact]`/`[human]`, drop guesses, carry `[unknown]`/`[inference]` into an explicit section | `references/synthesis.md:16-26` |
| **Control** | A one-word command the human steers with (`start`, `continue`, `deeper`, `skip`, `jump to`, `why`, `summarize`, `pause`, `stop`, `speedrun`) | `SKILL.md:247-265` |
| **`speedrun`** | A *standing* `continue` — runs the whole ladder without per-turn gates, at unchanged rigor | `SKILL.md:271-302` |
| **`pause` vs `stop`** | Deliberately different promises: `pause` = bookmark, **no** deliverable; `stop` = finish and synthesize | `SKILL.md:304-318` |

## How the entities relate

**[fact]** A **session** has one **goal** (`SKILL.md:30`) and one **repo type** (`SKILL.md:182`).
The type selects a **playbook**, which reorders the **ladder**. Each **stage** runs the **core loop**
once, must satisfy its **"done when"**, and emits one **trail file**. Every claim in a trail file
carries an **evidence tag**. At `stop`, the trail passes the **coverage check** and **confidence
filter** to become the **deliverable**, which is then embedded in the **study page**.

**[inference]** The evidence tag is the atomic unit of the whole domain — coverage check, confidence
filter, fog map, and the `[human]` correction protocol are all downstream of it. Remove tagging and
nothing else in the design has anything to operate on.
