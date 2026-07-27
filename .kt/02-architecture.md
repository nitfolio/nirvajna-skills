# 02 · Architecture

## The shape

There is no runtime, so "architecture" here means **what gets loaded into an agent's context, and
when**. Everything else is documentation for humans or brand assets.

The organising principle is **progressive disclosure**: a lean always-loaded file plus heavier files
pulled in only when a run actually needs them (`README.md:132-134`).

## Top-level modules

Every top-level entry is accounted for — none left unexplored.

| Module | What it is | Runtime role |
| --- | --- | --- |
| `onboard-me/` | The one skill | **The entire product** |
| `README.md` | Repo front door — pitch, skills table, install, layout | None |
| `onboard-me/README.md` | Human usage docs for the skill | **None** — never loaded by the agent |
| `assets/` | 4 PNGs (wordmark + logo, light/dark each) | None |
| `LICENSE` | MIT | None |
| `.gitignore` | Two comment lines, zero patterns | None |

[fact] `.gitignore` ignores nothing. Both lines are a comment explaining that `.kt/` is committed on
purpose. Verified: the file is 2 lines and neither is a pattern.

## Inside the skill

```
onboard-me/
├── SKILL.md          347 L  ── ALWAYS LOADED. The method.
├── README.md         411 L  ── humans only; no runtime edge points at it
└── references/              ── ON DEMAND
    ├── repo-playbooks.md         360 L
    ├── synthesis.md               64 L
    └── study-page-template.html  368 L
```

[fact] 347 lines always loaded vs 792 lines on demand — the heavy 70% of the skill stays out of
context until a run reaches the stage that needs it.

## How the pieces talk

The dispatch mechanism is **a prose sentence naming a file path**. There is no import graph, no
config, no registry — an instruction like "read `references/repo-playbooks.md`" is the call.

Every edge, enumerated:

```
                      SKILL.md
                (frontmatter = trigger)
                          |
        +-----------------+------------------+
        |                                    |
   :185 | at Orientation                :297 | at `stop`
        |                               :315 |
        v                                    v
 repo-playbooks.md                     synthesis.md
 (13 types + fallback)                       |
                                        :51  | "copy the template"
                                             v
                                   study-page-template.html
                                             |
                                             v
                                     .kt/onboarding.html
```

- [fact] `SKILL.md:185` → `references/repo-playbooks.md`, during Stage 1 Orientation.
- [fact] `SKILL.md:297` and `SKILL.md:315` → `references/synthesis.md`, at `stop`/end of speedrun.
- [fact] `synthesis.md:51` → `references/study-page-template.html`, the only second-hop edge.
- [fact] All edges are one-way and originate in `SKILL.md`. No reference file points back.

## Two couplings that are invisible from either side

These are the interesting bits of this architecture, because nothing enforces them.

**1. The HTML template is hard-coded to `SKILL.md`'s evidence vocabulary.**

[fact] `study-page-template.html:215`:

```js
s=s.replace(/\[(fact|inference|unknown|human)\]/g,'<span class="tag tag-$1">[$1]</span>');
```

Those four words are defined in `SKILL.md:74-82`. [fact] Rename or add a tag there and the study page
silently stops highlighting it — no error, just unstyled text. There is no test and no build step
that would catch it.

**2. Unfilled slots hide themselves.**

[fact] `study-page-template.html:305`:

```js
.filter(b=>b.md && !/^<!--[\s\S]*-->$/.test(b.md.trim()));
```

Any slot still containing its `<!-- PASTE … -->` placeholder is dropped at render time. So
`synthesis.md:56-57`'s instruction to delete slots for missing files is belt-and-braces, not
load-bearing — a partial KT degrades gracefully either way.

## The asset path split

[fact] The two READMEs reference the same PNGs differently:

- `README.md:4-6`, `:157-159` — relative (`assets/wordmark-dark.png`)
- `onboard-me/README.md:4-6`, `:404-406` — absolute (`https://raw.githubusercontent.com/nitfolio/nirvajna-skills/main/assets/…`)

[inference] Deliberate, not drift. The `onboard-me/` folder is *designed to be copied out of the
repo* (`README.md:78-81`), where a relative `../assets/` path would resolve to nothing. Based on the
install instructions, not on a comment — no comment explains it.

[fact] Cosmetic inconsistency, noted not fixed: the wordmark is `width="640"` in `README.md:6` but
`width="565"` in `onboard-me/README.md:6`. Commit `da19904` ("Enlarge the README wordmark to 640px")
only touched the root.

## Open unknowns

- [unknown] Nothing enforces the `name:`/directory-name match or the tag-vocabulary coupling. Whether
  the author has a checking process outside the repo is not visible from here.
