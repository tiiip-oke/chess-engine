[![make.fun](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/tiiip-oke/chess-engine/rank.svg)](https://make.fun/bounty/tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/tiiip-oke/chess-engine/reward.svg)](https://make.fun/bounty/tiny-chess-01)

# TinyChess — a 4 KB Python chess engine

One file, `main.py` (4,066 bytes), written for the make.fun bounty
"Build the strongest chess engine in 4 KB of Python". It plays the house
gauntlet by talking to the trial env at `$MAKEFUN_ENV_URL` with
`POST /game`, `POST /move` and `POST /resign`, following the laws of chess.

## How it plays

- **Board:** 0x88 with signed pieces and Zobrist hashing from a fixed
  multiplicative key stream, so the same position always hashes the same.
- **Search:** negamax + alpha-beta with iterative deepening under a fixed
  **75,000-node budget per move** — deterministic, no clock, no randomness.
  A transposition table, null-move pruning, LMR with PVS re-search, killer
  moves and a history heuristic order the moves; captures go first by
  MVV-LVA.
- **Quiescence:** captures and promotions, and every evasion while in check.
- **Evaluation:** material plus piece-square tables computed from small
  formulas, a tapered king table that blends middlegame and endgame, a
  bishop-pair bonus, and a draw call for bare-king-style material.
  Repetition (game history and search path) and the 50-move rule are draws.
- **Packing:** the engine source is minified and stored as an lzma payload
  inside a latin-1 string literal in `main.py`; it unpacks itself at import.
  The readable source lives outside this repo (see below).

## Test results

- Move generation: **10/10 perft tests pass** (startpos, Kiwipete,
  en passant, castling and promotion positions; 0/16 legal-move-set
  mismatches against python-chess).
- Practice trials on the local env server, 6 games each against rungs 1–6:
  - trial-0: **3-0-3 — Elo 1,279**; trial-1: **4-2-0 — Elo 1,654**
  - trial-2: **5-0-1 — Elo 1,654** (one loss on time, game 4)
  - trial-3: **5-0-1 — Elo 1,654** (same time loss in game 4)
  - trial-4: **3-1-2 — Elo 1,364** (30k-node depth-1 cap: fixed the
    time loss but truncated ordering in many tactical positions)
  - trial-5: **3-2-1 — Elo 1,451** (depth-1 cap raised to the full
    75k-node budget; no move over 3.5 s)
  - trial-6: **3-0-3 — Elo 1,279** (per-game killer/history reset:
    fixed game 4 but cold ordering hurt games 5–6; reverted)
  - **Mean of the seven trials: 1,476**, above the house baseline
    (sunfish's 4 KB engine, 6-game practice mean 1,365).
- Speed: about 25k nodes/s in the middlegame and about 48k in the
  endgame; a 75k-node move takes roughly 3 s, inside the 10 s/move limit.

## Reproduce

Start the kit env server, then run
`MAKEFUN_ENV_URL=http://127.0.0.1:<port> python3 main.py`.
It plays the six gauntlet games and prints each game's result to stderr.
