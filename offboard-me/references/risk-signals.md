# Risk signals: what to scan for before you ask anything

Come here from the Triage stage. The job is to arrive at the interview already knowing what to ask
about, so the departing engineer's time is spent only on what the repo genuinely cannot answer.

Work through this in three passes, in order: **identify the person**, **neutralize the traps**, then
**collect the signals**. Skipping the middle pass is the most common way this scan produces a register
full of confident noise.

Shell pipelines below assume a POSIX shell (bash, zsh, Git Bash). On PowerShell the `git` half is
identical; adapt the piping.

---

## Pass 1 — Identify the departing author

You cannot compute sole ownership until you know which identities are theirs. People commit under
several: work email, personal email, a laptop that had `user.name` set to a nickname, a GitHub
`noreply` address.

```bash
git shortlog -sne --all                 # every author, with counts and emails
git log --all --format='%an <%ae>' | sort -u | grep -i "<name-fragment>"
cat .mailmap 2>/dev/null                # existing identity mapping, if the repo has one
```

Confirm the full set with the human in the Triage turn — "I'm treating these three identities as
yours, is that right?" — before ranking anything. A missed alias silently understates their ownership
and can hide their most exclusive work.

Note bot and service accounts (`dependabot`, `github-actions`, release bots) so you can exclude them.

---

## Pass 2 — Neutralize the traps

Every one of these makes someone look like the author of code they never wrote. Handle them before
computing anything.

**Mass mechanical commits.** A repo-wide reformat, a lint autofix, a license-header sweep, or a
find-and-replace rename reassigns blame for thousands of lines to whoever ran it. Look for commits
with an enormous file count and a trivial message, then ignore them:

```bash
git log --all --format='%h %an %s' --shortstat | grep -B1 "files changed" | head -40
git blame -w --ignore-rev <mechanical-sha> <file>    # -w also ignores whitespace-only changes
cat .git-blame-ignore-revs 2>/dev/null                # repo may already list them
```

**File moves and renames.** Plain `git log <file>` stops dead at the rename, making long-lived code
look new and its real author look uninvolved.

```bash
git log --follow -- <file>       # follow the file across renames
git blame -C -C -- <file>        # detect lines moved or copied from other files
```

**Squash merges.** If the repo squash-merges pull requests, every line in a PR is attributed to
whoever pressed merge, not the person who wrote it. Check whether merge commits share one author while
`%an` on the underlying work differs — if the repo squashes, say so plainly and treat authorship
counts as *low* confidence for the whole scan. This one is worth telling the human about, because it
changes how much any ownership claim is worth.

**Generated, vendored, and build output.** `node_modules/`, `vendor/`, `dist/`, `*.pb.go`, lockfiles,
migrations that a framework wrote. High line counts, zero tribal knowledge. Exclude them:

```bash
git log --format='%an' -- . ':(exclude)vendor' ':(exclude)dist' ':(exclude)*.lock'
```

**Shallow or absent history.** A shallow clone, an exported tarball, or a fresh repo imported from
elsewhere makes all git-derived signals meaningless. Check `git rev-parse --is-shallow-repository` and
the age of the first commit. If history is missing, say so in the first turn and fall back to the
code-shape signals (marked workarounds, magic constants, operational docs) which need no history at
all — then lean much harder on the human's own sense of what's unusual.

---

## Pass 3 — Collect the signals

Each signal below carries a **confidence** level. Respect it: a high-confidence signal can be asserted
as `[fact]`, a low-confidence one is `[inference]` at best and must be offered as "this looked odd —
is it?" rather than "this is a workaround."

### 1. Sole authorship · high confidence

Files or directories where the departing person is effectively the only author.

```bash
# per-file author breakdown
git log --format='%an' --follow -- <file> | sort | uniq -c | sort -rn

# candidate files: those they've touched, ranked by their share
git log --author="<email>" --name-only --format= --all | sort | uniq -c | sort -rn | head -50
```

A file where they hold nearly all commits *and* nobody else has touched it in a year is the strongest
single signal in this list.

### 2. Recency-weighted exclusivity · high confidence

Historical authorship matters less than who has touched it *lately*. Code only they have modified in
the last 6–12 months is knowledge that has not yet been shared with anyone, regardless of who wrote it
originally.

```bash
git log --since="12 months ago" --format='%an' --follow -- <file> | sort -u
```

One name in that output, and it's theirs → high rank.

### 3. Marked workarounds · high confidence, cheapest win

The comments where someone already told you there's a story:

```bash
grep -rniE "TODO|FIXME|HACK|XXX|WORKAROUND|GOTCHA|do not (touch|change|remove)|don't (touch|change)|temporary|for now|leave this|magic|why|weird|careful" \
  --include="*.*" . | grep -v node_modules
```

Cross-reference each hit against authorship — a `HACK` comment written by the person leaving is a
question that answers itself once asked. Prioritize ones with no explanation attached, and ones whose
"temporary" is measured in years (`git log -S` the comment to date it).

### 4. Undocumented magic values · medium-high confidence

Constants, thresholds, timeouts, retry counts, buffer sizes, feature-flag names, and cron expressions
with no comment, no test explaining them, and no derivation. Find the constant, then date it:

```bash
git log -S "TIMEOUT = 300" --format='%h %an %ad %s' -- <file>   # the commit that introduced it
```

**The commit it arrived in is the question.** A magic number introduced alongside "handle Stripe 5xx
on capture" tells you what to offer as your reading — which is what turns recall into recognition.
Rank higher when the value is unusual (90 seconds, not 60; 4 retries, not 3) and when nothing else in
the repo uses the same shape.

### 5. Reverts and failed experiments · high confidence, badly underused

The cheapest knowledge to capture and the most expensive to rediscover. A successor who doesn't know
what was already tried will spend a quarter re-running it.

```bash
git log --all --format='%h %an %ad %s' | grep -iE "revert|roll ?back|back out|undo|didn't work|abandon"
git log --all --format='%h %an %ad %s' --merges | grep -iE "revert"
```

Also look for directories that appear and disappear, and dependencies added then removed
(`git log -p -- package.json | grep -E '^[-+].*"'`). Each one is a "we tried X" worth one question.

### 6. Deliberately retained dead code · medium confidence

Code that appears unreachable but has survived several of the author's own later commits. Surviving
their own cleanup passes usually means *kept on purpose*, not forgotten — and the purpose is exactly
the kind of thing that lives only in a head.

Find unreferenced symbols (grep the definition, count call sites), then check whether the author
touched the file after the code became dead. High value when confirmed, so ask — but offer it as a
question, since "unreachable" is easy to get wrong in languages with reflection, DI, dynamic
dispatch, or plugin loading.

### 7. Unmarked workarounds · low confidence — ask, never assert

Code that is unusually complex relative to its neighbourhood, on a path they solely own, with thin
test coverage. Signals: a function far longer than its siblings, deeply nested conditionals guarding
specific values, a comment that explains *what* rather than *why*, a `sleep` in production code, a
retry wrapped around something that shouldn't need one, an unusually specific error string.

This signal has a real false-positive rate. Surface at most a handful, always as "this looked unusual
— is there a story here?", and drop them quickly if the answer is no.

### 8. In-flight work · high confidence, decays fastest

Everything that dies on their last day. Scan this early — it feeds Stage 2, the most time-critical
stage in the ladder.

```bash
git branch -a --sort=-committerdate --format='%(refname:short) %(committerdate:relative) %(authorname)'
git log --all --author="<email>" --since="3 months ago" --format='%h %ad %s' --oneline
git branch -a --no-merged main
git log --all --author="<email>" --format='%h %s' | grep -iE "wip|draft|part 1|first pass|temp"
```

Unmerged branches with recent commits, WIP commits, and anything whose message implies a sequel
("part 1", "first pass", "will follow up") are all live threads. Check the hosting platform for open
draft PRs and assigned issues if that's reachable; if not, ask them directly — it's one question with
a high yield.

### 9. Operational knowledge · medium confidence

The runbook that never got written:

```bash
grep -rniE "manually|by hand|ssh into|run this|remember to|make sure to|before deploying|after deploying|note:|warning:" \
  --include="*.md" --include="*.txt" .
ls -la scripts/ bin/ ops/ tools/ 2>/dev/null      # one-off scripts are runbooks in disguise
grep -rniE "cron|schedule|0 \* \* \*" --include="*.y*ml" --include="*.tf" .
```

Also: alert and monitor definitions (what does this alert *mean*, and what do you do about it?),
anything in CI marked `continue-on-error` or `allow_failure`, and deploy steps that require a human.

### 10. External relationships and access · medium confidence

Third-party clients and SDK imports, webhook endpoints, callback URLs, config keys naming vendors,
OAuth app registrations, service accounts, DNS or certificate config. Each names an external party
where a human relationship may exist and is about to lapse.

Record **locations only, never values.** The question is "who owns this account and who can rotate
this key", not "what is the key".

### 11. Inherited `[unknown]`s · highest priority when present

If a `.kt/onboard/` trail exists from a prior knowledge-transfer session, every `[unknown]` in it is a
question that a careful reader already failed to answer from the code. That's a pre-filtered,
high-value question set — put it straight into the register near the top.

```bash
grep -rn "\[unknown\]" .kt/onboard/ 2>/dev/null
```

Read only. Those files belong to another session; your answers go in `.kt/offboard/`.

---

## Ranking the register

Order by **exclusivity × cost**, not by how interesting the code is.

**Exclusivity** — how likely it is that this knowledge exists nowhere else:
- *High*: they are the only recent author, nothing is documented, no test explains it.
- *Medium*: shared authorship but they're dominant; a stale doc exists.
- *Low*: several active authors, or it's well covered by tests and comments.

**Cost if lost** — what it does to whoever inherits it:
- *High*: touches money, auth, data integrity, or production stability; or it's on a deadline; or
  getting it wrong is silent rather than loud.
- *Medium*: costs real debugging time.
- *Low*: annoying to rediscover, cheap once you do.

Anything **high × high** goes to the top regardless of category. Below that, prefer the ladder's order,
because in-flight work expires and old landmines don't.

Two adjustments worth applying:
- **Promote silence.** A failure that is loud (crashes, pages) will teach the successor eventually. A
  failure that is silent — wrong numbers, dropped records, a retry that quietly gives up — never will.
  Rank silent risks above loud ones of the same size.
- **Demote what a test already documents.** If a test asserts the behaviour and explains it by name,
  the knowledge is captured. Spend the human's time elsewhere.

## Mapping signals to ladder stages

| Signal | Stage |
|---|---|
| 8 — In-flight work | 2 · In-flight and imminent |
| 3, 4, 6, 7 — Workarounds, magic values, dead code | 3 · Landmines |
| 9 — Operational knowledge | 4 · Operational reality |
| 5 — Reverts and failed experiments | 5 · Decisions and dead ends |
| 10 — External relationships and access | 6 · Relationships and access |
| 1, 2, 11 — Sole authorship, exclusivity, inherited unknowns | 7 · Sole-ownership sweep |

## Sizing the scan

Don't inventory everything. A register of 200 items is not more thorough than one of 25 — it is less
useful, because the ranking is what makes it actionable and 200 items cannot be ranked meaningfully.

Aim for **15–30 items**, weighted toward the top of the ranking, and say plainly in the Triage turn
what you didn't scan. On a large repo, scope to the areas they actually owned rather than the whole
tree; on a monorepo, ask which services are theirs before scanning at all.
