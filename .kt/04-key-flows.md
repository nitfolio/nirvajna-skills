# 04 · Key flows

Nothing executes, so a "flow" here is a path through *documents* — but it is a real, traceable path
with waypoints, and the same discipline applies: every waypoint carries a `file:line`.

Two flows matter. Flow A is what a user does. Flow B is what the skill does at the end, and it is
where the only real machinery lives.

---

## Flow A — install → trigger → first stage

**Waypoint 1. Install.** [fact] `README.md:61`:

```bash
npx skills@latest add nitfolio/nirvajna-skills
```

[fact] `README.md:64-67` describes the result: one canonical copy in `.agents/skills/`, symlinked
into `.claude/skills/` and every other agent directory found, so Codex/Cursor/Gemini CLI read the
same source. `-g` installs for the user, `--copy` writes real files instead of symlinks.

[fact] Manual alternative at `README.md:78-81`: `cp -r onboard-me ~/.claude/skills/` (personal) or
into `<your-repo>/.claude/skills/` (project-level). The instruction stresses **the whole folder, not
just `SKILL.md`** (`README.md:74`) — `references/` must travel with it.

**Waypoint 2. Trigger.** [fact] `SKILL.md:3-10`. The `description` is matched against what the user
says. It is written as situations, not features — "where do I start", "walk me through this repo",
due diligence, reverse-engineering a legacy system, resuming an in-progress KT. [fact] Explicit
override at the end: "Prefer this over ad-hoc 'explain this code' whenever the goal is understanding
a whole system rather than one snippet."

[fact] An explicit invocation (`/onboard-me`, `README.md:91`) bypasses matching entirely.

**Waypoint 3. The goal question.** [fact] `SKILL.md:30-45`. Exactly one question, asked before any
exploration, with four options that reweight the whole ladder. [fact] `:42-45` gives two escape
hatches: if the opening message already states the goal, don't spend a turn asking; if the human
doesn't answer, assume "just understand it" and continue — never block.

**Waypoint 4. Classify, then load the playbook.** [fact] `SKILL.md:182-188` — classify the repo
during Orientation, then read `references/repo-playbooks.md` and follow **one** playbook.
[fact] `repo-playbooks.md:3-6` reinforces it: follow the dominant type, don't read all of them; if
the repo is a mix, name the pieces and make the rest `jump to` targets.

**Waypoint 5. Announce the trail, then write it.** [fact] `SKILL.md:244-245` — tell the human `.kt/`
is being created *before* creating it, and mention they may want to gitignore it.

**Waypoint 6. Loop.** [fact] `SKILL.md:52-58` — DISCOVER → EXPLAIN → ASSESS → PROPOSE → CONFIRM,
one stage per turn, then stop and wait. [fact] The stage may only be called done when its completion
criterion (`SKILL.md:160-180`) is met.

Exit: the human says `pause` (bookmark, no deliverable — `SKILL.md:308-311`) or `stop` (Flow B).

### The speedrun variant

[fact] `SKILL.md:271-302`. The gate at CONFIRM is removed and nothing else changes. Two clauses are
load-bearing:

- [fact] `:290-292` — read-only gets **stricter**, not looser: "Autonomy removes the human who would
  have approved running a build, test, or script, so in a speedrun you never run them."
- [fact] `:293-295` — genuine forks still stop the run (monorepo needing scoping, a destructive
  action, evidence contradicting the stated goal).

---

## Flow B — `stop` → two artifacts → rendered page

**Waypoint 1.** [fact] `SKILL.md:313-318` — before writing a word, read `references/synthesis.md`.

**Waypoint 2. Coverage check.** [fact] `synthesis.md:6-14`. If only a couple of stages met their
criteria, do **not** silently generate a polished document — it "manufactures exactly the artifact
this skill exists to prevent, and it will outlive the session and be believed by people who weren't
here." Offer three options: synthesize anyway, keep exploring, or pause.

**Waypoint 3. Confidence filter.** [fact] `synthesis.md:16-26`. Sort every claim:

- **Promote** `[fact]` and `[human]` into the body as settled statements.
- **Drop** dead ends and anything that was only ever a guess.
- **Carry** every surviving `[unknown]` and unpromoted `[inference]` into an explicit
  "Assumptions & things to verify with a human" section.

[fact] `:25-26`: "Never launder an inference into confident prose by dropping its tag."

**Waypoint 4. Write `08-onboarding.md`.** [fact] Six-part shape at `synthesis.md:33-40`: what this
system is · architecture map · domain glossary · one or two key flows · your first change ·
assumptions & things to verify. [fact] `:42-43` sets the acceptance test — readable start to finish
by someone who never ran the skill and has no access to the chat.

**Waypoint 5. Build `onboarding.html`.** [fact] `synthesis.md:51-58`, four steps:

1. Copy `references/study-page-template.html` to `.kt/onboarding.html`.
2. Paste each file's **raw markdown verbatim** into its slot — `08` fills the `guide` slot, `00`–`07`
   the `trail` slots. Blocks are inert so no escaping is needed, with one exception: a literal
   `</script>` in the markdown must be written `<\/script>`.
3. Delete the slot for any `.kt/` file that doesn't exist.
4. Set the `<title>` and the `.kt-repo-name` span to the repo name. **Change nothing else.**

[fact] The slots are at `study-page-template.html:171-205` — nine of them, each a
`<script type="text/markdown">` with `data-id`, `data-label`, and `data-group`.
[fact] The two `REPO_NAME` placeholders are at `:36` (title) and `:147` (sidebar span).

**Waypoint 6. Render — in the browser, no server.** [fact] The page's own pipeline:

| Step | Where |
| --- | --- |
| Collect every markdown slot, strip leading newline | `:311-314` |
| Drop slots still holding their `<!-- PASTE -->` comment | `:314` |
| Group into "Curated guide" (08) and "Working trail" (00–07) | `:321-324` |
| Build a nav link + a `<section>` card per file | `:325-338` |
| Render markdown with the inline regex parser | `:208-307` |
| Colour `[fact]` / `[inference]` / `[unknown]` / `[human]` | `:218` |
| Per-file "Copy markdown" button | `:334` |
| Per-code-block "Copy" button (hidden `<textarea>` holds the raw text) | `:255-257`, `:344-345` |
| Clipboard write, with a `<textarea>` + `execCommand` fallback | `:348-355` |
| Scroll-spy highlighting the active nav item | `:358` |
| Live filter over files and headings | `:365` |

**Waypoint 7.** [fact] `synthesis.md:63-64` — announce both outputs by name and mention the HTML
opens straight in a browser, no server needed.

---

## The correction flow (short, and the most important one)

[fact] `SKILL.md:87-90`. When the human corrects a claim, three things happen in the **same turn**:
retag it as `[human]`, fix the affected `.kt/` file, and ask whether the correction invalidates
anything else already said. [fact] The rationale is given: "A mental model built on a stale
assumption gets more wrong the longer it stands."

[inference] This is the only inbound edge into the trail from outside the repo, which is why
`[human]` outranks `[fact]` in the filter at `synthesis.md:20`. Based on the tag definitions at
`SKILL.md:80-82` and the promotion rule; no file states the ranking explicitly.

## Open unknowns

- [unknown] Whether the `npx skills@latest add` path has been run against this repo successfully —
  the command is documented, but nothing in the repo records a verified install.
