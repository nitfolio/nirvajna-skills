# When the repo fights back

Situations where the normal approach doesn't work cleanly. Adapt rather than guess — and say plainly
in `rebuild.md` that one of these applied, because each one changes how much the document can be
trusted.

## The repo is huge, or a monorepo

Don't try to specify it all. After the top-level survey, stop and scope: one target, one folder, one
document. Name the other candidates in `INDEX.md` as unstarted so nobody mistakes a partial run for
a complete one.

Watch for the shared-library trap: behavior that lives in a package used by six services belongs in
each document that depends on it, stated as behavior — not referenced as "see the shared module",
which is a pointer into an implementation the reader doesn't have.

## No usable git history

An exported tarball, a shallow clone, an SVN mirror. You lose commit messages, changelog-by-history,
and the ability to tell deliberate from accidental.

Lean harder on tests, docs, and config. Be more conservative about `[suspect]`: without history you
usually can't tell a bug from a decision, so record the behavior and say the evidence for intent
wasn't available. Stamp the trail date-only.

## The repo arrived read-only or as an upload

Copy it to a writable working directory first, and put `.kt/rebuild/` there. Tell the human where it
lives so they can carry it back. Uploaded archives usually lack `.git/`, so the previous case applies
too — and often lack `node_modules`-style context that reveals versions, so pin claims to what the
manifests actually say.

## Generated, vendored, or build output

`node_modules/`, `vendor/`, `dist/`, `*.pb.go`, generated clients, compiled assets. Skip as source of
truth. One exception worth checking: a generated file whose *schema source* is missing from the repo
is sometimes the only remaining statement of a contract — in that case read the generated artifact,
and record that it's the only evidence available.

## Behavior lives in data, not code

Rules tables, workflow definitions, templates, feature flags whose values live in a service you
can't see, content that drives logic. The code is an engine and specifying it teaches nothing.

Say so early and change the plan: the rules must be extracted as rules, or exported as data
alongside the document. If the rule data isn't in the repo at all, that is a `[unknown]` that
outweighs everything else in the document — put it in the coverage section, not a footnote, because
a rebuild without it isn't possible at any level of effort.

## No tests and no docs

Common, and it changes what the document can claim. You can still specify behavior — code states
behavior reliably — but you cannot distinguish *intended* from *accidental*, so nearly everything
that would have been `[suspect]` stays unmarked, and intent claims shouldn't be made at all.

Compensate with commit messages, issue links, changelogs, and error-message wording (which often
states intent more plainly than any comment). Then flag it in the document's coverage section: a
spec derived from code alone is a description of what happens, and a human who knows the product
should read the rules section before it's trusted.

## The system can't be run

No credentials, no environment, a dead service. This is the normal case, and it's why the boundaries
rule says propose rather than run. It means every behavioral claim is derived from reading, never
from observation, and the document should say so once, plainly.

Where reading genuinely can't settle something — a library's runtime behavior, an external service's
actual responses, a race outcome — record `[unknown]` with what would settle it, rather than
producing a confident guess that reads identically to a verified fact.

## It's a rewrite of a rewrite

The repo contains dead code from a previous migration, two implementations of the same feature, or a
compatibility layer for a system that no longer exists. Establish which path is live before
specifying anything — git history and route registration usually settle it — and record the dead
one as excluded rather than ignoring it, because a reader who later finds it needs to know you saw it.

## A vendor or platform does the work

Payments, auth, search, notifications: the behavior lives in a third-party product configured
through a console you can't see. The repo shows only the integration.

Specify the observable behavior and the integration contract, then name the configuration you
couldn't read as a `[unknown]` with a pointer to where it lives. A rebuild will need that console
exported; saying so early is worth more than a document that quietly assumes defaults.
