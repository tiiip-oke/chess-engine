# TinyChess — a 4 KB Python chess engine

A chess engine in a single `main.py` (4,090 bytes) for the make.fun
"Build the strongest chess engine in 4 KB of Python" bounty.
It talks to the house env at `$MAKEFUN_ENV_URL` (`POST /game`, `POST /move`,
`POST /resign`) and plays by the rules of chess.

## Approach

- **Board:** 0x88, signed pieces, Zobrist hashing (fixed multiplicative hash).
- **Search:** negamax with alpha-beta, iterative deepening to a **fixed
  75,000-node budget per move** (deterministic — no clock, no randomness),
  transposition table, null-move pruning, LMR + PVS, killer moves and a
  move-based history heuristic for ordering, MVV-LVA captures first.
- **Quiescence:** captures/promotions, all evasions when in check.
- **Eval:** material + formula piece-square tables with a tapered king
  endgame table, bishop-pair bonus, insufficient-material draw detection.
  Repetition (game history + search path) and the 50-move rule are draws.
- **Packing:** `main.py` is a minified engine packed as an lzma payload in
  a latin-1 string literal, unpacked at import. (The readable source and
  the build script are kept outside this submission repo.)

## Local test results

- Move generation: **10/10 perft tests pass** (startpos, Kiwipete,
  en-passant, castling, promotions; 0/16 legal-move-set mismatches
  vs python-chess).
- Practice trials (local env server, 6 games each, rungs 1–6):
  - trial-0: **3 won, 0 drawn, 3 lost — Elo 1,279**
    (W depth-2, W depth-3, W depth-4, L nodes-100k, L nodes-250k, L nodes-500k)
  - trial-1: **4 won, 2 drawn, 0 lost — Elo 1,654** (undefeated)
    (W depth-2, W depth-3, D depth-4, W nodes-100k, D nodes-250k, W nodes-500k)
  - **Mean of the two trials: 1,466**, above the house baseline
    (sunfish's 4 KB engine, 6-game practice mean 1,365) and well above
    the 1,154 practice floor.
- Speed: ~25k nodes/s middlegame, ~48k endgame; a 75k-node move takes
  ~3 s, inside the 10 s/move limit and the trial wall clock.

## Reproduce

Run the kit env server, then `MAKEFUN_ENV_URL=http://127.0.0.1:<port>
python3 main.py`. It plays the gauntlet and prints each game's result.
