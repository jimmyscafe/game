# Spray the Fly

A single-file browser game: a fly buzzes around the screen, you chase it with a can of Mortein using your mouse or trackpad, and you click to spray. Get the fly in 10 sprays or fewer to win a ticket to the Vertacy Conference, Port Stephens. Winners can put their name on an online leaderboard.

## Play

Open `index.html` in any modern browser, or visit the GitHub Pages link for this repository.

- Move the mouse or trackpad to chase the fly with the can.
- Click, tap, or press Space to spray. The spray hits only the small ring at the nozzle.
- You have 10 sprays. Hit the fly and you win; miss all 10 and you miss out.
- The leaderboard ranks winners by fewest sprays, then fastest time.

## Leaderboard

Scores live in a Supabase project that belongs only to this game. The game talks to it with the public (anon) key, which is safe to expose: row-level security lets players read scores and add new ones, and nothing else.

The `scores` table has `name` and `score` columns. `score` is a single integer that packs both numbers so that lower is better:

```
score = sprays * 100000 + milliseconds   (time capped at 99.999s)
```

For example `200750` means 2 sprays in 0.75 seconds.

To remove a score, delete its row in the Supabase dashboard under **Table Editor → scores**.

### Set up the table from scratch

Run this in the Supabase **SQL Editor**:

```sql
create table scores (
  id bigint generated always as identity primary key,
  name text not null check (char_length(name) between 1 and 20),
  score integer not null check (score >= 0 and score < 10000000),
  created_at timestamptz not null default now()
);

alter table scores enable row level security;
create policy "read scores" on scores for select using (true);
create policy "add scores" on scores for insert with check (true);
create index scores_top on scores (score desc);
grant select, insert on scores to anon, authenticated;
```

Then set `SUPABASE_URL` and `SUPABASE_KEY` near the top of the script in `index.html`.

## Changing the game

Everything is in `index.html`. Useful knobs near the top of the script:

- `MAX_SPRAYS`: how many sprays a player gets (default 10).
- `HIT_RADIUS`: how close the nozzle has to be to the fly (default 34 px). Bigger is easier.
- In `updateFly`, the base speed (`420`) and dart speed (`1100`) control how hard the fly is to catch.
