# tiny-chess-engine

[![make.fun · tiny-chess-01](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/diki965/tiny-chess-engine/rank.svg)](https://make.fun/bounty/tiny-chess-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/diki965/tiny-chess-engine/reward.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)


A compact chess engine in a single 4,095-byte `main.py` (MIT license).

## Run it

```sh
python3 main.py
```


## Design

- **Move generation:** 0x88 board, fully legal moves incl. castling,
  en passant and promotions.
- **Search:** alpha-beta with a fixed 65,000-node budget per move
  (deterministic — no clock), iterative deepening, quiescence over
  captures, promotions and check evasions, transposition table
  **persistent across moves within a game**, killer moves + history
  heuristic, null-move pruning (R=2) and LMR.
- **Evaluation:** tapered piece-square tables with material
  100/320/330/500/900, bishop-pair bonus, pawn structure, rook
  open/semi-open files and 7th-rank bonuses, king shelter and an
  endgame king table.
- **Protocol:** speaks the make.fun env over HTTP (`MAKEFUN_ENV_URL`),
  returning one UCI move per turn; deterministic: the same input
  replays byte-for-byte.

## Size rule

- `main.py` is the whole engine: 4,095 bytes as committed (cap 4,096),
  Python 3.13, standard library only. It reads, imports or execs no other
  file in the repository. Compression is allowed and counted — this file
  unpacks and execs an LZMA payload.
- The whole tree is 8,108 bytes total (README, LICENSE, main.py), no file
  over 4,096. MIT licence.

## Development

Built from scratch (original code — no sunfish source copied) in
`engine-dev/engine.py`, then minified and LZMA-packed with
`makefun compress`. Full (unpacked) source: `source.py` (8,586 bytes),
kept uncommitted in this folder (it cannot be committed — the
per-file cap is 4,096) so `makefun bundle` can send it beside the
sessions; a copy also lives in `engine-dev/golfed/`.

## Practice results (kit gauntlet, 6 games per trial)

| version | trial 0 | trial 1 | trial 2 | mean Elo |
|---------|---------|---------|---------|----------|
| v1 (50k nodes) | 1654 | 1364 | 1451 | 1490 |
| v2 (65k nodes, restored eval) | 1364 | 1364 | 1545 | 1424 |
| **v3 (persistent TT + full eval)** | **1545** | **1654** | **1545** | **1581** |

v3 record: **13 wins, 2 draws, 3 losses** across the three trials
(9-0-0 vs the depth-2/3/4 house engines; losses only vs the
nodes-100k/500k rungs). Trial-0 replayed 3x with byte-identical
`state.json`. Sunfish 4k baseline practice mean: **~1365** (6-game).
v3 clears it by ~+216 Elo.

