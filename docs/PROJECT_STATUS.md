# Project status

As of 2026-10-06. Figures describe the demo snapshot `db-snapshot-20261006-085714`, measured
read-only. The implementation, tests and data are private.

## Live

- **Demo website**, https://srs-myk.duckdns.org: private and login-only. It runs a code release built
  from one commit (`c1c1d5c`) and serves a separate, restored copy of the data. The update channel and
  acquisition are switched off in the deployed configuration.
- **Release checks** (reported 2026-10-06): all 22 post-deployment smoke checks passed, and the reviewer
  account's sign-in, the About page, the three example players, report generation and download, and the
  access restrictions were checked on the live site.
- **Working features:**
  - Player Scout (one player, one exact format, the Tournament pool);
  - per-player tournament and edition selection;
  - Excel reports (seven sheets);
  - Batch Scout;
  - an About / Try-the-demo page;
  - owner-managed accounts and invitations;
  - role-based permissions.
- **Code and data move separately.** Deploying a new code release does not change the data the demo
  shows; refreshing the demo's data is a separate snapshot operation.

## Milestones

| Milestone | State | Measured result |
|---|---|---|
| Replay archive and battle parser | Implemented | 223,717 stored replays; raw evidence preserved privately |
| Tournament sources and schedules | Implemented for the approved sources | 458 tournaments, 302,398 scheduled matches |
| Replay ↔ match linking | Implemented, partial coverage | 87,444 stored links; 136,742 candidates kept unresolved |
| Identity (accounts → players) | Implemented, partial coverage | 5,393 players; 7,704 confirmed aliases |
| Tournament reporting and Excel | Implemented | 61,084 replays reportable for a player |
| Website and deployment | Live (above) | — |
| Website-driven incremental updates | Built and exercised on the workstation; off on the demo | — |

Stored links are not automatically reportable. They still pass withholding (466 are withheld) and
tournament-resolution checks first; see [Architecture](ARCHITECTURE.md).

## Validated but not applied

- **Placing threads whose edition or round cannot be established.** Many tournament threads cannot be
  tied to a specific edition or round from the evidence. A validated change places them under an
  explicit "not established" marker instead of leaving them unplaced. On an isolated copy it would add
  34,335 tournament sides, 19,397 games and 89 links, and a rerun changes nothing. It is not applied to
  the working data or the demo. See [case studies 2 and 3](DEBUGGING_CASE_STUDIES.md).
- **A measurement-script fix** (case study 3) has been identified but not made.

## Remaining limitations

- **Coverage is partial**, for two kinds of reason:
  - **Deliberate holds.** 136,742 replay–match candidates are not proven. 466 links are withheld
    because the evidence behind them lost its support. There are 382 open review items for
    contradictory results, and 2,716 for tournament metadata that does not resolve.
  - **Acquisition and parser limits.** There are 3,841 open review items for thread layouts no reader
    understands yet, and 50,147 unresolved fetches the source answered with "not found".
- **One pool.** Only the Tournament pool is implemented; ladder and qualifier pools are not.
- **Owner review queues** (identity conflicts, edition questions) remain open; they are left for a
  person to decide.
- **Frozen demo data.** The demo shows its snapshot until a deliberate snapshot refresh.
- **One host** with no high availability; backups are scripted and run by hand.
- **Tests (reported).** The private suite has about 10,000 tests and is not fully green; its known
  failures include checks that read live private data. On 2026-10-06 the subset that runs without
  private inputs measured 2,542 passed, 1,876 skipped (the database tests) and 0 failed. That result
  is reported here; the tests are not part of this repository.
