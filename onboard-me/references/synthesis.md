# Synthesizing `08-onboarding.md`

Read this when the human says `stop`. It turns the raw `.kt/` trail into the one document a newcomer
will actually read.

## 1. Check coverage before writing anything

If exploration is thin — only a couple of the ladder stages met their completion criteria — do **not**
silently generate a polished document. A confident onboarding doc built on a barely-explored repo
manufactures exactly the artifact this skill exists to prevent, and it will outlive the session and be
believed by people who weren't here.

Say what's covered and what isn't, then offer three options: synthesize now anyway, keep exploring, or
just pause. Proceed to synthesis only on a clear yes, or when coverage is genuinely sufficient.

## 2. Synthesize through a confidence filter

Build the document *from* the `.kt/` trail, not from memory, and sort every claim:

- **Promote** `[fact]` and `[human]` claims into the body as settled statements.
- **Drop** dead-end explorations and anything that was only ever a guess.
- **Carry** every remaining `[unknown]` and unpromoted `[inference]` into an explicit
  **"Assumptions & things to verify with a human"** section.

Never launder an inference into confident prose by dropping its tag. The fog map is the most honest
thing you produced — it belongs in the deliverable, not on the cutting-room floor.

## 3. Keep it lean

A tight, true onboarding doc beats an exhaustive one nobody trusts. Resist turning this into a
handbook; the working trail (`00`–`07`) is already there for anyone who wants the full reasoning.

## Suggested shape

1. **What this system is** — one paragraph a newcomer could repeat back in a standup.
2. **Architecture map** — at the level of modules/services. Draw it in text by default: the study
   page ships no diagram renderer, so a mermaid fence shows up there as source.
3. **Domain glossary** — each term with a one-line meaning and where it lives.
4. **One or two key flows** — traced end to end with real `file:line` waypoints.
5. **Your first change** — a low-risk area, plus the exact commands to run and verify it.
6. **Assumptions & things to verify** — the surviving unknowns and inferences, named plainly.

It should be readable start to finish by someone who never ran the skill and has no access to the
chat that produced it.

## 4. Build the offline study page (`onboarding.html`)

Once `08-onboarding.md` is written, always also produce `.kt/onboarding.html` — a single, self-contained
page that bundles the whole trail (`00`–`08`) into one place a newcomer can study, with a one-click copy
button on every file. Build it from the template, don't hand-roll it:

1. Copy `references/study-page-template.html` to `.kt/onboarding.html`.
2. It has one `<script type="text/markdown">` slot per KT file. Into each slot, paste that file's **raw
   markdown verbatim** — `08-onboarding.md` fills the featured "guide" slot; `00`–`07` fill the "trail"
   slots. The blocks are inert, so no escaping is needed (the only exception: if the markdown literally
   contains `</script>`, write it `<\/script>`).
3. **Delete the slot for any `.kt/` file that doesn't exist** — a partial KT (or a paused-then-stopped
   one) should never render empty sections.
4. Set the page `<title>` and the `.kt-repo-name` span to the repo's name. Change nothing else — the
   CSS, the markdown renderer, the nav, and the copy buttons are complete in the template.

The page mirrors the `.kt/` files exactly, which means the secrets rule carries over untouched: those
files are already redacted, so paste them as-is and never reintroduce a secret value while filling the
template. When you announce `stop` is done, name both outputs — `08-onboarding.md` and `onboarding.html`
— and mention the HTML opens straight in a browser, no server needed.
