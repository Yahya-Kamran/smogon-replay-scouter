# Debugging case studies

Three real defects, in the order they were found. Each records the symptom, the root cause, the fix,
the check that keeps it fixed, and the lesson. Player names in the examples are illustrative.

| Case | Status |
|---|---|
| 1. Bracket seeds in player names | **Live**: applied to the working data before the demo snapshot |
| 2. Order-dependent tournament placement | **Validated, not applied** to the working data or the demo |
| 3. A measurement that rolled back its own setup | **Validated, not applied**; one script fix is still pending |

The code and tests behind these cases are private; this page describes them.

---

## 1. Bracket seeds contaminating player names

**Symptom.** In playoff schedules, some scheduled players could never be matched to the account in
their replays. A schedule line like `(2) PlayerOne vs PlayerTwo (15)` was stored as `2playerone` and
`playertwo15`, while the replay accounts were `playerone` and `playertwo`.

**Root cause.** Brackets write seeds beside names, and normalizing the whole side kept the seed. A
blunt fix would have been worse: many real usernames end in digits, and `PlayerA (2) vs PlayerB (1)`
can be a score rather than a seeding.

**Fix.** A seed is separated only when three conditions hold within one opening post:
- **syntax**: a delimited 1–3 digit token at the edge of the name, never a bare run of digits;
- **consistency**: one seed per name across the post, seeds on both sides, never zero;
- **proof of seeding in the post itself**: either a bracket-round shape, where each line's two seeds
  sum to the same total (1 v 8, 2 v 7, …), or explicit seeding wording with dense numbers.

The name as written is kept beside the corrected one. **682 stored schedule entries were corrected.**
**977** entries that look seeded, in posts that do not prove a seeding, were deliberately left as they
are.

**Regression check.** Tests check that usernames ending in digits are never treated as seeds, that a
score-shaped post is left alone, that a proven bracket is separated, and that separating a seed never
moves a line, a format or an occurrence.

**Lesson.** A pattern that "looks like" noise is often real data. Decide per source, from evidence the
source itself contains, and keep the raw value so the decision can be revisited.

## 2. Order-dependent tournament placement

**Symptom.** A bulk placement of threads into tournaments was rehearsed on an isolated copy. One event
(four round threads of one open tournament) came out differently depending on processing order. In 8
of 9 orders, three rounds were placed and Round 3 was held; with Round 3 processed first, the reverse
happened.

**Root cause.** The data already held **two** tournament records for that one event, named for two
different prize amounts. Each thread followed its own stored record, so one event pointed at two
tournaments, and whichever thread ran first claimed the edition.

**Fix.** Before anything is placed, an event whose threads would point at more than one existing
tournament has its mappings withdrawn, and all its threads are held with that reason. The decision now
depends on the set of threads, not their order. Merging the two stored records was deliberately not
done; it is a separate decision for a person to make.

**Regression check.** A permutation test applies the rule to every ordering of an event's threads. On
the isolated copy, all nine orders were re-run inside one rolled-back transaction, and after the fix
every order gives the same result.

**Lesson.** "Process each item independently" is only order-independent if no item's decision reads
state that another item writes. Test that by reordering, not by inspection.

**Status.** Validated end to end on an isolated copy. Not applied to the working data or the demo.

## 3. A measurement that rolled back its own setup

**Symptom.** A validation run of the placement change stopped at its reporting step. It had produced
**2,921** new report scopes where the expectation said **2,938**: 17 short, with 3 fewer "changed" and
3 more "unchanged".

**Root cause.** The expectation was wrong, not the run. It came from a measurement that undid one
withdrawn event's changes inside a transaction and then measured the result. A shared helper called
in between ended by rolling back the transaction, which silently restored the undone changes. The side
and game counts had been measured before that call, so they were right. The report-scope
classification ran after it, read the uncorrected state, and concluded the correction had no effect on
those scopes.

**Fix.** The validation copy was checked by identity, not by count:
- the 17 scopes the expectation called "new" do not exist in the copy, or in the source data;
- the 3 "changed" scopes match the source data's digest exactly;
- every other format's scope count equals the earlier rehearsal's.

The expectation was corrected to the earlier rehearsal's figures minus exactly those 20 scopes. The
run's recheck mechanism re-ran only the checks against the recorded result, without repeating any
write. The run then completed: +34,335 tournament sides, +19,397 games and 89 new links, and a rerun
changed nothing.

**Regression check.** Tests derive the reporting expectation from the earlier rehearsal minus those 20
scopes. A stopped step's recorded writes are counted exactly, so a recheck never repeats them. The
measurement script itself still needs a fix (re-apply the undo after the helper returns); that is
recorded and not yet done.

**Lesson.** A helper that ends a transaction is a hidden side effect. When a measurement disagrees with
a result, compare the actual identities on both sides before changing either the code or the
expectation.

**Status.** Validated on an isolated copy. The placement change it measures is not applied to the
working data or the demo.
