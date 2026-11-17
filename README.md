# tiny-chess-01 entry

A compact chess engine in a single 4,095-byte `main.py` (MIT license).

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
- The whole tree (this README, LICENSE) is 6,536 bytes total, no file
  over 4,096. MIT licence.

## Development

Built from scratch (original code — no sunfish source copied) in
`engine-dev/engine.py`, then minified and LZMA-packed with
`makefun compress`. Full (unpacked) source: `source.py` alongside the
packed file in `engine-dev/golfed/` (8,586 bytes; it travels with the
submission via `makefun bundle` — it cannot live in this tree, the
per-file cap is 4,096).

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

Bench: ~26,800 nps, slowest move ~2.3 s at 50k nodes (~2.9 s at 65k)
— inside the 10 s/move limit on 1 CPU.
