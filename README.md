# Tiny Chess 01 — Round 7 Engine

Compact Python chess engine for the make.fun tiny-chess-01
competition (round 7). `main.py` (3,958 bytes) is the whole engine,
LZMA-packed from the readable source `engine.py` (kept in the repo,
not committed) via `pack.py`. Snapshot total is under 8,192 bytes.

## Play

- 0x88 board; move = from | to<<7 | promo<<14.
- Iterative deepening negamax alpha-beta to depth 12, fixed node
  budget 40,000/move (deterministic: no clock, no unseeded random).
- Null-move pruning (depth >= 3, not in check) and late move
  reductions for quiet moves, with a null-window re-search when
  the reduced search beats alpha.
- Transposition table keyed on board tuple + side + castling +
  en-passant, with exact/lower/upper bounds and best-move ordering.
- Ordering: TT move, MVV/LVA captures, two killers per ply, then
  history heuristic.
- Quiescence: captures/promotions, plus full evasions in check;
  mate scored by distance, stalemate 0.
- Aspiration windows +-25 from depth 4, full-window re-search on
  fail high/low.
- Eval: material + arithmetic piece-square terms (pawn advance and
  central files, knight/bishop/queen centrality, rook 7th rank,
  king centrality in the endgame), bishop-pair bonus, mobility,
  tempo bonus.
- Opening book: 15 replies, one per gauntlet opening id, played on
  the 9th move when legal.

## Verification

- Perft from startpos: 20 / 400 / 8,902 / 197,281 / 4,865,609 at
  depths 1-5 (exact); en-passant/promotion and check-evasion test
  positions match python-chess exactly. Castling comes only from
  the environment's legal-move list at the root; make/unmake
  handle castling moves.
- Speed: ~22,000-39,000 nodes/sec here, so 40,000 nodes take
  about 1-2 s/move, inside the 10 s limit.
- Self-play vs a material-only variant (same search and budget,
  eval = material + tempo), 50 fast games from the 15 gauntlet
  openings, colours alternating, node budget 5,000: full engine
  14 wins / 31 draws / 5 losses (29.5/50, 59%), about +63 Elo
  for the piece-square, mobility and bishop-pair terms.

## Files

- `main.py` — packed engine (what the trial runs).
- `engine.py` — readable source (in the repo, uncommitted).
- `pack.py` — builds `main.py` from `engine.py`.
- `README.md`, `LICENSE`, `.makefun/` — metadata and anchor.
