# 04 · Key flows

## Flow A — a normal interactive session, end to end

No hand-waving; every waypoint is a real location.

1. **Trigger.** Claude Code matches the user's situation against the frontmatter `description`
   (`SKILL.md:2-10`). **[fact]** `README.md:110-112` states the mechanism: "The description is the
   only thing that makes a skill trigger", which is why it's written as a list of situations
   ("where do I start", "walk me through this repo", "a `.kt/` directory exists") rather than as a
   feature summary.
2. **Load.** The `SKILL.md` body (lines 12-347) enters context. `references/` do **not**.
3. **Goal question.** `SKILL.md:30-45` — ask *why* they're here, once, offering four framings
   (bug fix / ownership / due diligence / just understand). If the opening message already states
   it, infer and skip the turn. Never block on it.
4. **Announce `.kt/`.** `SKILL.md:~237` — tell the human the directory is being created and that
   they may want to gitignore it.
5. **Stage 1, Orientation.** Runs DISCOVER→EXPLAIN→ASSESS→PROPOSE→CONFIRM (`SKILL.md:47-56`) using
   the tool list at `SKILL.md:92-112` (`ls`, manifests, entry points, `git log`, grep, tests).
   Completion criterion at `SKILL.md:160-162`: purpose + stack + entry points + repo type, **all
   cited**.
6. **Branch on repo type.** `SKILL.md:182-188` → read `references/repo-playbooks.md`, follow one of
   the 14 playbooks (`repo-playbooks.md:32-360`). This is the first lazy load.
7. **Write the trail.** Stage output → `.kt/01-overview.md`; `.kt/00-progress.md` rewritten
   (`SKILL.md:~228-243` gives its exact shape: Goal · Repo type / Stages checklist / Open questions
   / Next step).
8. **Stop and wait.** Turn ends with the fog assessment, **one** proposal, and 2-3 relevant controls
   — not all ten (`SKILL.md:267-269`).
9. **Loop stages 2-7**, one per `continue`, each writing its numbered file.
10. **Exit.** `pause` → bookmark only (`SKILL.md:306-311`). `stop` → Flow B.

## Flow B — `stop` / end of speedrun → the two deliverables

1. `SKILL.md:313-318` → read `references/synthesis.md` before writing anything.
2. **Coverage check** (`synthesis.md:6-14`). If only a couple of stages met their criteria: do
   **not** generate a polished doc. Say what's covered, offer three options (synthesize anyway /
   keep exploring / pause). Proceed only on a clear yes.
3. **Confidence filter** (`synthesis.md:16-26`). Promote `[fact]`+`[human]` into the body; drop
   dead ends and pure guesses; carry every surviving `[unknown]` and unpromoted `[inference]` into
   an explicit **"Assumptions & things to verify with a human"** section. `synthesis.md:25` forbids
   the failure mode by name: "Never launder an inference into confident prose by dropping its tag."
4. **Write `08-onboarding.md`** in the 6-part shape at `synthesis.md:33-41`.
5. **Build the study page** (`synthesis.md:45-64`):
   - copy `references/study-page-template.html` → `.kt/onboarding.html`;
   - paste each trail file's **raw markdown verbatim** into its matching
     `<script type="text/markdown">` slot — `08` into `data-group="guide"`
     (`study-page-template.html:170`), `00`–`07` into the eight `data-group="trail"` slots
     (`study-page-template.html:174-204`);
   - **delete the slot for any file that doesn't exist** (`synthesis.md:57-58`) so a partial KT
     never renders empty sections;
   - set `<title>` (`study-page-template.html:36`) and `.kt-repo-name`
     (`study-page-template.html:146`); change nothing else.
6. **Announce both outputs by name**, noting the HTML opens in a browser with no server
   (`synthesis.md:62-64`).

## Flow C — how the study page renders (the repo's only runtime code)

**[fact]** `study-page-template.html:303` collects every
`document.querySelectorAll('script[type="text/markdown"]')` block, and `:322` builds one `<section
class="file">` per block, adding the `feature` class when `data-group="guide"`.
**[fact]** Markdown is rendered by a **hand-rolled renderer** — `esc()` at `:208`, `inline()` at
`:210`, `renderMarkdown()` at `:222`. There is no `marked`, no CDN, no external asset of any kind.
**[fact]** A filter input (`:148`) drives the nav (`:149`), with scroll-spy at `:352` toggling the
`active` class; `copy()` at `:340` has a `<textarea>` fallback (`:345`) for browsers without the
async clipboard API.
**[inference]** The `script[type="text/markdown"]` choice is what makes step 5 safe: the browser
never executes or parses those blocks, so arbitrary markdown pastes in unescaped. `synthesis.md:54-55`
names the single exception — a literal `</script>` must be written `<\/script>`.
