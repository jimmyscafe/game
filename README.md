# Spray the Fly

A single-file 3D browser game. Two flies and a cockroach are loose in a small lounge room (couch, TV on a stand, a plant in the corner). You walk around in first person with a can of Mortein and spray them. Get all three in 10 sprays or fewer to win a ticket to the Vertacy Conference, Port Stephens. Winners can put their name on an online leaderboard.

## Play

Open `index.html` in any modern browser, or visit the GitHub Pages link for this repository. You need a keyboard and a mouse or trackpad, and an internet connection (the 3D engine, three.js, loads from a CDN).

- Click the game to lock the mouse, then move the mouse or trackpad to look around. The white dot in the middle is where the spray goes.
- WASD (or arrow keys) to walk, Shift to run, Esc to pause.
- Click or press Space to spray. The spray reaches about two metres in a narrow cone.
- The bugs flee when you get close: flies climb away, the cockroach scurries across the floor. Corner them.
- You have 10 sprays for all three bugs. Get them all and you win; run out and you miss out.
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
- `SPRAY_RANGE` and `SPRAY_HALF_ANGLE`: how far the spray reaches (2.2 m) and how wide the cone is (9°). Bigger is easier.
- In `updateBug`, `base` and `dartSpeed` set how fast flies and the cockroach move, and `fleeR` how close you can get before they run.
- `ROOM` sets the room size; the furniture positions follow from it.
- `resetBugs` decides how many flies and cockroaches spawn.
