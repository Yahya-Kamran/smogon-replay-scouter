# Reviewer guide

A three-minute tour of the live demo at **https://srs-myk.duckdns.org**. Credentials are provided on
request; reviewer accounts can scout and download reports, but cannot manage users or see owner tools.

## The tour

1. **Sign in**, then open **About** in the top navigation. It explains the terms and the pipeline in
   plain language and lists three verified examples.
2. **Pick an example** and press **Open**. The Player Scout page shows the player, the format, the
   record and the win rate.

   | Player | Format | Tournament games | Record |
   |---|---|---|---|
   | NoName6293 | `gen6ou` | 84 | 54-30 |
   | SoulWind | `gen5ou` | 101 | 65-36 |
   | ABR | `gen2ou` | 26 | 13-13 |

   SoulWind shows alternate accounts: several Showdown usernames grouped under one player by
   confirmed aliases.
3. **Inspect the teams.** Scroll through *Pokémon usage*, *Leads*, *Common pairs*, *Cores*,
   *Recurring teams* and *Revealed sets*. Moves and items that a game never revealed are not shown.
4. **Open a replay.** In *Replay history*, **open ↗** plays the original game on Pokémon Showdown.
5. **Generate the report.** Press **Generate Excel**, then download it from the confirmation. Nothing
   is generated until you press the button.
6. **Navigate the workbook.** Its seven sheets are Scout Overview, Replay Summary, Team Details,
   Player Usage Stats, Opponent Usage Stats, Usage Trends, and Provenance & Scope. The last one
   states exactly which games were included and why.
7. **Optional: Batch Scout** produces reports for several players at once, as a ZIP of independent
   workbooks.

## Things worth noticing

- **Partial coverage is intentional.** A player with few games may have games the evidence could not
  prove. The site never fills the gap by guessing (see [Project status](PROJECT_STATUS.md)).
- **One exact format per report.** `gen6ou` and `gen7ou` are never combined.
- **Provenance.** Every game in a report can be traced back to where it was found.

## What the demo is

The demo serves a fixed copy of the data restored from one snapshot. It does not collect new games.
The figures in this guide were measured against that snapshot.
