# When the repo fights back

Four situations where the normal exploration approach doesn't work cleanly. Adapt instead of guessing
— and say plainly, in the KT itself, that one of these applied.

## No usable git history

An exported tarball, an SVN mirror, a shallow clone. `git log`, `git blame`, and ownership signals are
unavailable or too thin to trust.

Say so plainly, then lean harder on structure, tests, and docs. Don't invent ownership or history you
can't see — an `[inference]` about "who owns this" with no git evidence behind it is exactly the kind
of confident-sounding guess this skill exists to prevent.

## Repo arrived as an upload or read-only mount

A zip uploaded to Claude.ai, a read-only directory, a mounted volume you can't write into.

Extract or copy it to a writable working directory before exploring, and put `.kt/onboard/` there
instead of the repo root — tell the human where it lives so they can carry it back into the real repo.
Uploaded archives usually lack `.git/`, so the no-git-history fallback above applies too.

## Very large repo / monorepo

Thousands of files, many services — too much to hold in one session's context.

Don't try to hold it all. After a quick top-level survey, add a scoping turn: "this is large; which
service or area should we KT first?" — and KT that slice. Note the rest as unexplored in
`00-progress.md` rather than silently ignoring it.

## Generated, vendored, or build output

`node_modules/`, `dist/`, `vendor/`, `*.pb.go`, and similar.

Skip it as source of truth. It's noise, not architecture — treating a generated file's shape as a
design decision produces confident nonsense.
