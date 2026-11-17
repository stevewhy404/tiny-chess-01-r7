# Tiny Chess 01 — Round 7 Engine

A compact Python chess engine written for the make.fun tiny-chess-01
competition (round 7). One file, `main.py`, holds the whole engine; it is
an LZMA-packed build of the readable source in `engine.py` (same code,
kept in the repo for review), produced by `pack.py`. `main.py` is 3,958
bytes, under the 4,096-byte limit; the whole snapshot is under 8,192.

## How it plays

- **Board:** 0x88 mailbox, move encoded as from | to<<7 | promo<<14.
- **Search:** iterative deepening negamax alpha-beta to depth 12 with a
  fixed node budget of 40,000 nodes per move (deterministic: no wall
  clock, no unseeded randomness), so replay is byte-for-byte identical.
- **Pruning:** null-move pruning (depth >= 3, not in check) and late
  move reductions for quiet moves, with a null-window re-search when
  the reduced search beats alpha.
- **Transposition table:** keyed on the board tuple plus side, castling
  rights and en-passant square, with exact / lower / upper bounds and
  the stored best move used for ordering.
- **Ordering:** transposition-table move first, then MVV/LVA captures,
  then two killer moves per ply, then the history heuristic.
- **Quiescence:** captures and promotions, plus full evasions when in
  check; checkmate is scored by distance to mate, stalemate is 0.
- **Aspiration windows:** +-25 around the previous iteration's score
  from depth 4 on, with a full-window re-search on fail high/low.
- **Evaluation:** material plus arithmetic piece-square terms (pawn
  advancement and central files, knight/bishop/queen centrality, rook
  on the 7th rank, king centrality in the endgame), bishop-pair bonus,
  mobility, and a tempo bonus.
- **Opening book:** 15 replies, one for each opening id in the
  gauntlet (Italian, Berlin, Morphy, Najdorf, Classical, Semi-Slav,
  Slav, Philidor, Scandinavian, Catalan, Bird, Dutch, English,
  King's Indian Attack, Alapin), played on the 9th move when legal.

## Verification

- Move generation: perft from the start position gives 20 / 400 /
  8,902 / 197,281 / 4,865,609 at depths 1-5 (exact), and the standard
  en-passant/promotion and check-evasion test positions match
  python-chess exactly. Castling is generated only from the
  environment's legal-move list at the root (the search itself does
  not generate it); make/unmake handle castling moves.
- Speed: about 22,000-39,000 nodes/second on this VM, so the 40,000
  node budget is reached in roughly 1-2 seconds per move, well
  inside the 10-second move limit.
- Strength: self-play against a material-only variant of the same
  engine (same search, same node budget, evaluation reduced to
  material plus tempo) over 50 fast games from the 15 gauntlet
  openings, alternating colours, node budget 5,000: the full engine
  scored 14 wins / 31 draws / 5 losses (29.5/50, 59%), an estimated
  +63 Elo edge for the piece-square, mobility and bishop-pair terms.
  The review's +150-250 claim for those terms was optimistic; +63
  is the measured value at this budget.

## Files

- `main.py` — the packed engine (what the trial runs).
- `engine.py` — the readable source of the engine.
- `pack.py` — builds `main.py` from `engine.py`
  (`python3 pack.py`).
- `README.md`, `LICENSE`, `.makefun/` — repo metadata and anchor.
