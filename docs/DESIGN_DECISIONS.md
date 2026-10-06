# Design decisions

Five decisions shape the system. Each names the problem, the choice and what it costs.

## 1. Preserve raw evidence; derive everything else

**Problem.** Forum posts get edited, replay pages disappear, and parsers have bugs. If only parsed data
were stored, a parser fix could not be applied to past data, and a disputed fact could not be traced
back to its source.

**Choice.** Every downloaded replay file and thread page is stored untouched, with a content hash
recorded in the database. Raw strings are kept beside their normalized forms (for example, the
scheduled name exactly as written beside the normalized name used for matching). Parsing can be re-run
from the raw files, and each replay is stored once, so re-processing updates it rather than
duplicating it.

**Tradeoff.** More disk and more bookkeeping, and the raw evidence is third-party content that this
showcase does not distribute. In return, corrections such as the bracket-seed fix
([case study 1](DEBUGGING_CASE_STUDIES.md)) could be applied to existing data from stored pages,
without downloading anything again.

## 2. Separate players from accounts

**Problem.** One person plays under several Showdown usernames, and different people's usernames can
look alike. Merging accounts by name would put strangers' games into someone's report.

**Choice.** Four separate identities: a canonical player, a Smogon forum account, a Showdown username,
and a tournament-specific alternate account. A replay always records the literal account that played.
A player reaches an account only through an alias with a status, a scope (global or one tournament)
and recorded evidence.

**Tradeoff.** More structure and more unresolved cases. A player's report can miss games played on an
alternate account nobody has proven yet. Showing an unproven game was judged worse than leaving it out.

## 3. Conservative identity inference

**Problem.** It is tempting to infer an alias from a single game, from player order (who was player 1),
or from similar names. Each of those is wrong often enough to matter.

**Choice.** Inference refuses player order, name similarity and partial series as evidence. A
best-of-N series must be complete before it can tell which account belongs to which player.
Contradictions block automatic confirmation, and a rejected pairing is never re-confirmed
automatically. In reports, an account claimed by two players is attributed to nobody, and an
attribution whose support is gone is withheld rather than deleted. Decisions made by a person are not
overridden by automation; anything that would affect one is reported for the owner to review.

**Tradeoff.** Lower coverage: many plausible links stay as review candidates (136,742 in the demo
data). The reports give up recall for precision on purpose.

## 4. Resumable, bounded processing

**Problem.** Acquisition and linking jobs touch hundreds of thousands of rows on a workstation, and
they get interrupted: by rate limits, crashes, and in one recorded case a power loss.

**Choice.** Work is split into units, and each unit's outcome is recorded. A failure on one item is
recorded and the batch continues; a rerun skips what is already recorded. Each run is bounded by a time
or request budget and resumes where it stopped. Where a unit writes, its data and its record are
written in one transaction. That is what let an interrupted validation run be reconciled with nothing
to undo: 425 placed threads had been committed, and all 425 were recorded.

**Tradeoff.** Every stage needs explicit state and idempotence checks, and "resume" has its own failure
modes. A rerun is accepted only if it changes no content, and that is checked, not assumed.

## 5. Snapshot-based deployment

**Problem.** The public demo should not put the working database at risk, and it has to run on a free
host. The data is about 3.4 GB (a dump of roughly 0.4 GB), and the site runs continuously.

**Choice.** One small free VM runs nginx (TLS), the application on loopback only, and a local
PostgreSQL. The demo serves a separate database restored from one consistent snapshot, always into a
new database rather than over an existing one. Before the copy is served, workstation-only history is
cleared from it and the copy is labelled. A check run with the service's own settings then confirms
which database will be served, before the service starts. The update channel stays off. Code releases
are archives of one commit: hash-checked, staged, and activated only after a readiness check.
Releasing code and refreshing the snapshot are separate operations.

**Tradeoff.** The demo is frozen at its snapshot, so newer data reaches it only through a deliberate
snapshot refresh; a code release does not change it. One host means no high availability. In the
deployed configuration the demo has no connection to the working database. That is a configuration
property that is checked before start, not a guarantee against every possible failure. A code rollback
switches back to the previous release, and the previous database is kept for the same purpose.
