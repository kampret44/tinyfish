# tinyfish

A 4KB Python chess engine for make.fun tiny-chess-01. One file
(`main.py`, 4,052 bytes), Python stdlib only — no model calls, no
clock, no randomness in move choice. Same position, same move,
every time.

## How it searches

0x88 board, negamax with principal-variation search and quiescence
search. A transposition table, two killer moves per ply, late-move
reductions, null-move pruning, check extensions, and mate-distance
pruning. Evaluation is incremental, from original piece values and
piece-square tables (MIT) — no sunfish code or tables. Every move
uses a fixed node budget, so search depth adapts to the position
instead of the clock.

## Practice trials

Trial-0 is 16 games against the house gauntlet (8 engines, once per
colour). Node-budget ladder:

- 50,000 nodes: Elo 1,087, W3 D2 L11, 18.3 min
- 100,000 nodes: Elo 1,290, W4 D5 L7, 26.6 min
- 200,000 nodes: Elo 1,396, W4 D8 L4, 47.7 min

Trial-1 at 200,000 nodes: Elo 1,362, W5 D5 L6, 49.8 min.

The house baseline to beat is mean Elo 1,360 over practice trials
0–2 (1,326 / 1,464 / 1,290). The prize needs a mean above that
over 6 final trials on fresh openings.

## Why 150,000 nodes

200k beats the baseline (1,396 on trial-0) but took 47.7 min — too
close to the 60-min wall cap, and the grading machine is 1 CPU
against 1.5 shared CPUs here, so a real 200k trial could forfeit.
100k is safe on time (26.6 min) but scores 1,290, below the
baseline. 150,000 splits the difference: expected to keep most of
the 200k strength with time to spare (expected ~40 min a trial).
Per-move time is safe too: the slowest move at 200k nodes took
~3.8 s, well under the 10 s per-move limit.

## Running it

Set `MAKEFUN_ENV_URL` to the game server URL, then run `main.py`.
It polls `POST /game` for its turn, searches the FEN, and plays its
move with `POST /move`. A trial is 16 games against 8 house engines;
the engine exits when the server reports the trial done.
