# 02 · Architecture

*Written against commit `2e94f1b` on 2026-07-27. Every `file:line` below is a line number **at that commit** — check that commit out to verify a claim, or re-run the skill to regenerate the trail against current source.*

## The shape

There is no runtime, so "architecture" here means **what gets loaded into an agent's context, and
when**. Everything else is documentation for humans, brand assets, or hosting plumbing.

The organising principle is **progressive disclosure**: a lean always-loaded file plus heavier files
pulled in only when a run needs them (`README.md:136-138`).

## Top-level entries

Every one is accounted for — none left unexplored.

| Entry | What it is | Runtime role |
| --- | --- | --- |
| `onboard-me/` | The one skill | **The entire product** |
| `README.md` | Repo front door — pitch, skills table, install, layout | None |
| `onboard-me/README.md` | Human usage docs for the skill | **None** — never loaded by the agent |
| `.kt/` | This trail; committed on purpose, and published via Pages | None (it is *output*) |
| `assets/` | 4 PNGs (wordmark + logo, light/dark each) | None |
| `LICENSE` | MIT | None |
| `.gitignore` | Two comment lines, zero patterns | None |
| `.nojekyll` | Empty file; makes Pages publish `.kt/` | None — but load-bearing for hosting |

[fact] `.gitignore` ignores nothing. Both lines are a comment explaining that `.kt/` is committed on
purpose.

[fact] `.nojekyll` has zero bytes of content. Its *existence* is the entire signal.

## Inside the skill

```
onboard-me/
├── SKILL.md          362 L  ── ALWAYS LOADED. The method.
├── README.md         439 L  ── humans only; no runtime edge points at it
└── references/              ── ON DEMAND
    ├── repo-playbooks.md         360 L
    ├── synthesis.md               76 L
    └── study-page-template.html  376 L
```

[fact] 362 lines always loaded vs 812 lines on demand — the heavy 69% of the skill stays out of
context until a run reaches the stage that needs it.

[fact] `SKILL.md` has 17 top-level sections, from "What this skill is for" (`:15`) to "Example of one
good turn" (`:341`).

## How the pieces talk

The dispatch mechanism is **a prose sentence naming a file path**. There is no import graph, no
config, no registry — an instruction like "read `references/repo-playbooks.md`" *is* the call.

```
                      SKILL.md
                (frontmatter = trigger)
                          |
        +-----------------+------------------+
        |                                    |
   :185 | at Orientation                :312 | at `stop`
        |                               :330 |
        v                                    v
 repo-playbooks.md                     synthesis.md
 (13 types + fallback)                       |
                                        :63  | "copy the template"
                                             v
                                   study-page-template.html
                                             |
                                             v
                                     .kt/onboarding.html
                                             |
                                             v
                                   GitHub Pages (published)
```

- [fact] `SKILL.md:185` → `references/repo-playbooks.md`, during Stage 1 Orientation.
- [fact] `SKILL.md:312` and `SKILL.md:330` → `references/synthesis.md`, at `stop` / end of speedrun.
- [fact] `synthesis.md:63` → `references/study-page-template.html`, the only second-hop edge.
- [fact] All edges are one-way and originate in `SKILL.md`. No reference file points back.
- [fact] `onboard-me/README.md` is referenced by nothing at runtime — documentation *about* the
  skill, never loaded *by* it.

## The one external URL

[fact] `study-page-template.html:251` now contains a single external reference:

```
'<a href="https://mermaid.live" target="_blank" rel="noopener">mermaid.live</a>, '+
```

[fact] This is a **hyperlink, not a fetch** — the page still requests nothing over the network, so
the self-contained/offline guarantee (`SKILL.md:207`, `:329`, `synthesis.md:59`, `:64-65`) is intact.

[fact] `SKILL.md`, `repo-playbooks.md`, and `synthesis.md` contain **zero** URLs (grep count 0 for
each). Everything the agent loads is fully self-contained.

## Two couplings that nothing enforces

These are the interesting part of this architecture, because no test, build, or schema protects them.

**1. The HTML template is hard-coded to `SKILL.md`'s evidence vocabulary.**

[fact] `study-page-template.html:218`:

```js
s=s.replace(/\[(fact|inference|unknown|human)\]/g,'<span class="tag tag-$1">[$1]</span>');
```

Those four words are defined at `SKILL.md:74-82`. [fact] Rename or add a tag there and the study page
silently stops highlighting it — no error, just unstyled text.

**2. Unfilled slots hide themselves.**

[fact] `study-page-template.html:314`:

```js
.filter(b=>b.md && !/^<!--[\s\S]*-->$/.test(b.md.trim()));
```

Any slot still holding its `<!-- PASTE … -->` placeholder is dropped at render time, so
`synthesis.md:68-69`'s "delete the slot" instruction is belt-and-braces rather than load-bearing.

[fact] There are 9 slots, at `study-page-template.html:171-205`, keyed by `data-id` — `08-onboarding`
in the `guide` group and `00`–`07` in the `trail` group.

## A third coupling, new today: hosting

[fact] `.nojekyll` → GitHub Pages → `.kt/onboarding.html` → the link in `onboard-me/README.md`'s PS.
Deleting the empty `.nojekyll` breaks that chain silently: Pages resumes skipping dot-directories,
`.kt/` disappears from the site, and the README link 404s. Nothing in the repo warns about this
except this trail.

## The asset path split

[fact] The two READMEs reference the same PNGs differently:

- `README.md:4-6`, `:161-163` — relative (`assets/wordmark-dark.png`)
- `onboard-me/README.md:4-6`, `:421-423` — absolute `raw.githubusercontent.com/...` URLs

[inference] Deliberate. The `onboard-me/` folder is designed to be copied *out* of the repo
(`README.md:78-81`), where a relative `../assets/` path would resolve to nothing. Based on the
install instructions; no comment explains it.

[fact] Cosmetic inconsistency, noted not fixed: the wordmark is `width="640"` in `README.md:6` but
`width="565"` in `onboard-me/README.md:6`. `da19904` enlarged only the root.

## Open unknowns

- [unknown] Nothing enforces the `name:`/directory match, the tag-vocabulary coupling, or the
  `.nojekyll` dependency. Whether the author has any checking process outside the repo is not
  visible from here.
