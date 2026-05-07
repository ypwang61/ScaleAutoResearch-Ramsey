# R(4,15) Run Record (summarized)

Run tag: `apr22_codex_r4_s15_0` (initial), continued under `apr27_codex_r4_s15_99`.
Target: `R(4,15)`.
Known best lower bound at start: `159`.
Known bundled witness: `ramsey_resources/best_constructions/R(4,15)>=159.txt`.

`verify.py` is treated as a read-only oracle; every claimed best is required to pass it.

## Setup

- Initialized `results.tsv` with the required header.
- Created experiment directory scaffold under `experiments/r4_s15/apr22_codex_r4_s15_0/`.
- Re-verified the bundled 158-vertex witness with `verify.py`.

## Experiment Log

### Phase 1: Generic SA from random and seeded n=159

- `run_001_baseline`: unmodified `search.go`, `n=158`, short smoke test from scratch. Checkpoint verified with `verify.py`: `23001` total conflicts (`21973` `K4` + `1028` `I15`). Logged as `no_improvement`.
- `run_002/003/004_resume159*`: relaunched after fixing an initial path mistake so `seedfromsub` actually loads the verified 158-base.
- Early signal: seeded `n=159` search is dramatically better than random initialization. `run_002_resume159` reached internal best `426` conflicts very quickly, so extension from the verified 158 witness is the right immediate direction.
- `run_004_resume159_long`: verified at `397` total conflicts on `n=159`, split as `352` `K4` + `45` `I15`. Best frontier from the generic search family.
- `run_005_rowclone`: verified at `402` total conflicts, split as `401` `K4` + `1` `I15`. Worse total, but nearly solves the blue side and suggests the main barrier is now concentrated in `K4`.
- Long-running generic searches plateaued around `397` and `402`, so the next algorithmic step is to stop spending all budget on generic SA and instead use a dedicated single-vertex extension search for the `158 → 159` move.

### Phase 2: Dedicated single-vertex extension

- Implemented `experiments/r4_s15/apr22_codex_r4_s15_0/search_single_vertex.py`: a dedicated repair search that freezes the verified 158-vertex base graph and searches only the adjacency vector of the new vertex. It alternates between eliminating `I15` witnesses in the non-neighborhood and eliminating red triangles in the neighborhood.
- `run_011_single_vertex_rowclone` and `run_012_single_vertex_long` quickly outperformed the generic search family. Both verified at `272` total conflicts, all on the red side: `272` `K4` + `0` `I15`.
- After the dedicated search proved superior, the old generic long runs were stopped and the active budget was reallocated to specialized one-vertex repair branches.
- `run_013_single_vertex_rowclone_branch` improved the dedicated-search frontier to `262` total conflicts, with the blue side still fully clean: `262` `K4` + `0` `I15`.
- `run_012_single_vertex_long` improved further to `218` (`218 K4 + 0 I15`); `run_011_single_vertex_rowclone` improved to `213` (`213 K4 + 0 I15`).
- The blue-side constraint has been solved robustly by the dedicated extension search; weaker branches are recycled to start from the latest `213/218` parents instead of older `262/272` states.
- `run_011_single_vertex_rowclone` and `run_016_from_218` both reached verified `148` (all red), with last-row adjacency vectors differing in 48 positions even though both have red degree 45 — different basins.
- `run_017_mix148` verified at `152` (`152 K4 + 0 I15`): worse than the incumbent, but from a genuinely different mixed-parent basin. `run_018_flip148` improved the frontier to `137` (`137 K4 + 0 I15`).
- New `137` incumbent has new-vertex red degree `42`, vs `45` for the older `148` family and `44` for the mixed-parent `152` family. Lower-red-degree neighborhoods become the main direction.
- Made the blue-side penalty configurable in `search_single_vertex.py`. This lets the search keep promising states like `83 K4 + 1 blue` long enough to repair the last blue witness instead of immediately discarding them for much worse `blue_bad=0` states.
- `run_021_penalty_smoke` reached verified `136` (`136 K4 + 0 I15`).

### Phase 3: Counted-swap and degree-aware moves

- After the `136` breakthrough, the main line split into two families:
  - **low-blue-penalty** branches that intentionally tolerate a tiny number of blue witnesses while reducing `K4` further before repairing blue to zero.
  - **swap-based** branches in `search_single_vertex_swap.py` that try to preserve the new vertex degree more closely and escape the `136/137` basin by exchanging in/out neighbors rather than mostly flipping single bits.
- The first swap implementation undercounted blue-side violations because it only sampled witnesses; `run_026_swap136` exposed this (`0 K4` but `1000` exact `I15` under `verify.py`). The searcher was upgraded to use capped counting of blue 14-cliques.
- With capped blue counts, `run_028_swap_count_smoke` verified at `118` (`116 K4 + 2 I15`).
- `run_030_swap_count_mix` improved through `96 → 82 → 75 → 73 → 61` exact conflicts (verified at each step). The new frontier is `61` (`52 K4 + 9 I15`).
- Updated `search_single_vertex_swap.py` with conflict-focused candidate generation: `--full-score` scores all possible swap-out/swap-in vertices instead of a random subsample, and `--blue-witness-boost` forces vertices from a current blue 14-clique witness into the swap-in candidate set.
- Added `--pick-pool` so plateau runs can randomly choose among the top candidate swaps instead of deterministically taking only the best move.
- The long-running counted-swap branches from the verified `61` parent kept improving. `run_035_swap_wide_count61` is verified at exact `46` (`35 K4 + 11 I15`). Other simultaneously checked candidates were exact `49/51/52`, so only the `46` row was added to `results.tsv`.
- Added optional pair-swap moves via `--pair-pool`. Added `--full-scan` and `--pick-pool` to `search_single_vertex.py`. In full-scan mode each step evaluates all 158 one-bit flips under the exact capped proxy, then optionally randomizes among the best few moves.
- `run_041_fullscan_count46` was the clear winner: exact `25` (`18 K4 + 7 I15`). Others at the same check were `41/44/46`, so only the `25` row was added.
- Verified-25 graph has new-vertex red degree `32`, a major structural shift from the earlier degree-`41` basin.
- Added `--pair-full-scan` for exact two-bit neighborhood search on a bounded candidate pool, biased toward vertices appearing in current witnesses, high triangle-load current neighbors, and low triangle-cost non-neighbors.
- Added `--beam-full-scan`: evaluate all single flips exactly, keep the best `beam_width` first moves, then evaluate all exact second flips from those beam states.
- A more fundamental reformulation followed: because every n=159 conflict must include the new vertex, the search over the final row is exactly a MaxSAT-style optimization over `158` Boolean variables. Red-side conflicts are 3-clauses, blue-side conflicts are 14-clauses.
- Implemented `search_single_vertex_maxsat.py` (lazy blue-clause discovery, weighted MaxSAT via `pysat` `RC2`). Smoke runs explored a qualitatively different region but did not yet beat verified `25`.
- Implemented `search_single_vertex_lns_maxsat.py`, a destroy-repair solver that freezes most variables and runs RC2 on a selected free subset only.
- Beam-based local search eventually escaped the verified `25` basin: `run_045_pairscan25` verified at exact `18` (`12 K4 + 6 I15`). Substantially better than the `25` incumbent.
- LNS-MaxSAT variants were not competitive on this frontier yet; budget was reallocated to deeper local search from the verified `18` parent.
- Hash comparison showed `run_045_pairscan25`, `run_056_beam18_diverse`, and `run_058_beam18_narrow` had all converged to the exact same incumbent — keeping all three alive was wasting budget rather than adding diversity.
- Updated the main local searcher with witness-focused move pools: candidate flips can be restricted to a dynamically refreshed pool built from current red-triangle support, current blue-clique support, top triangle-load current neighbors, top low-cost non-neighbors, and a small random tail. Focused branches did not produce a basin escape — every focused worker snapped back to the same exact-`18` graph.
- Repaired LNS-MaxSAT to recenter each round on accepted incumbents. Even after the fix, unconstrained or weakly constrained MaxSAT still over-optimized the red side at the expense of the blue side.

### Phase 4: SAT proof that the 158-base is not one-vertex-extendable

After all four beam runs plateaued at exact `18`, the natural question became whether the verified 18-conflict frontier is even reachable to 0 by flipping just the new vertex's row. Formulating the one-vertex extension as SAT makes this answerable directly:

- variables `x_1..x_158` represent the new vertex's red-edges to base;
- 14121 hard 3-clauses encode `K_4`-freeness with the new vertex (one per red triangle in the 158-vertex base);
- 13407 hard 14-clauses encode `K_15`-freeness with the new vertex (one per blue 14-clique in the 158-vertex base; full enumeration completed in ~50 s).

Both Glucose42 and Cadical195 returned **UNSAT** in <20 s. So the 158-vertex base graph **cannot** be extended to a 159-vertex Ramsey witness by adjusting only the new vertex's adjacencies. The local-search family was therefore optimizing toward an unreachable target.

This kills the prior "freeze 158 base, search new vertex's row" search family. The new strategy is to allow base edge flips and/or vertex swaps so that the resulting 158-base is *both* still a valid Ramsey witness *and* extendable.

- `search_flip_sat.py`: try flipping a small set of base edges, then attempt exact SAT extension. Smoke run on 200 random single-edge flips produced 182 invalid bases (introduced `K_4` or pushed `α` to 15) and 18 valid bases that were all UNSAT for one-vertex extension. So single-flip is *not* enough.
- All previous local-search runs were stopped because they were chasing a verified UNSAT problem.

### Phase 5: Full-graph SA on n=159 with incremental violation maintenance

- Implemented `sa_n159.py`: simulated annealing on the full 159-vertex adjacency matrix where the score is exactly `# K_4 + # K_15` (all 159 vertices, including the new vertex). The K4 delta after each edge flip is computed from bitmask-induced subgraphs, and the K15 delta is computed by enumerating K13 in the blue intersection of the flipped edge's endpoints. Both updates are incremental.
- Initial `k4=12, k15=6` (matching the verified-18 frontier). Single workers with high temperature explore widely (score grows to several hundred) but greedy descent rarely improves past 18.
- Adding ILS-style kicks (tuned `kick_after / kick_size / restart_prob`) restores best-of-search after each kick and perturbs the incumbent.
- Pool L (aggressive cooling) reached verified `9 K_4 + 7 I_15 = 16` conflicts.
- The 16-conflict state was extracted as `checkpoint_score16.graph` and used to seed the next pool.
- Cascade of SA pools each seeded from the previous best produced rapid descent: 18 → 16 → 14 → 12 → 11 → 10 → 9. Each step verified by `verify.py` and saved as `checkpoint_score{n}.graph`.
- The 9-conflict frontier (`k4=4 + k15=5`) is a strong local minimum:
  - All single-edge flips have non-negative delta on both K4 and K15.
  - All predicted 2-flips (pair sums of single deltas) have non-negative delta — verified by `fast_repair.py` over the 529 violation-incident edges. So no 1- or 2-flip improvement exists from this state.
  - SA pools spanning a wide range of kick sizes all bounce back to score 9 after each kick.
- Of the 9 violations: 4 K4s all touch the new vertex 158 (so are "x-fixable"), 2 K15s are entirely in base, and 3 K15s involve 158.
- Tried alternative starting families: circulant graphs on 159 vertices via SA over the 79-bit connection set (`circulant_search.py`). All workers reach `K4=0, K15=1` and get stuck. Random-init full-graph SA starts at ~10000 violations and is descending, but unlikely to reach the 9-basin without seeding.
- The 9-conflict frontier is the best search-frontier reached by these means, but is *not* a 159-Ramsey witness (still has 9 violations; would need 0).
- Tried big moves and crossover (`sa_n159_vertex_swap.py`, pure greedy with very large kicks, vertex-block crossover between many score-9/10 parents, huge ILS kicks) — all bounce back to 9.

### Phase 6: AlphaEvolve Algorithm 6 (harmonic tunneling) breaks the 9-plateau

After studying AlphaEvolve's Algorithm 6 for R(4,15), implemented its key components in `algo6_r415.py`:

- **Spectral / Paley initialization**: generalized Paley/QR graphs on primes `p ≈ n` (e.g. `p = 157`).
- **Tabu-Enhanced SA** with adaptive cooling that slows down as `K_4 → 0`.
- **Look-Ahead & Harmonic Scoring**: edges scored by `ΔK_4 - λ₁·Risk(I_neighbor) - λ₂·M(e)` using a harmonic-memory map of edge/orbit success counts.
- **Harmonic Tunneling**: when stuck with `K_4 > 0`, identify the cyclic distance `d*` most frequent in the current `K_4`s and **flip ALL edges** with `|u−v| (mod n) = d*`. This is a structural mutation that is invisible to ordinary local search.
- **Constraint Toggle**: prioritise `K_4 = 0` first; once satisfied, target `I_15` violations directly.

From `checkpoint_score9.graph`, Pool JJ (`run_112_algo6_from9`) found a verified `K_4 = 4 + I_15 = 4 = 8` graph (`checkpoint_score8.graph`). This was the first improvement past `9` after dozens of generic SA / ILS / vertex-swap pools failed.

Subsequent pools all started from the new score-8 incumbent with varied algo6 parameters (a range of `init_temp` and `tunnel frequency` settings); all workers stayed at `score = 8`.

### Phase 7: SAT-CEGAR exploration around score-8

Implemented `sat_cegar_n159.py` and `sat_cegar_restricted.py`: counter-example guided abstraction refinement with a Hamming-distance budget around the verified-8 seed. Each iteration:

1. Solve SAT for a graph within Hamming `B` of seed that satisfies all current violation clauses.
2. Verify the returned graph; if 0 K4 / K15, FOUND.
3. Otherwise, add new violation clauses and re-solve.

Findings:

- **Unrestricted CEGAR** (all 12,561 edges flippable, varied Hamming budgets): each iteration the SAT solver finds Hamming-near solutions that satisfy cumulative constraints but introduce *hundreds* of new K4/K15. Did not converge in reasonable time.
- **Restricted CEGAR** with a 1-hop edge restriction (only edges incident to the 81 violation-related vertices ≈ 1500 edges, modest Hamming budget): returned **UNSAT** in 29 iterations / a couple of seconds — proving that no flip combination of that size, restricted to violation-incident edges, can simultaneously break all 8 current violations + the lazily-discovered ones without introducing more.
- Larger budgets / wider hops are intractable (cardinality encoding scales with `budget × #edges`).

Combined with the earlier proof that the 158-vertex base is **not one-vertex-extendable**, and with AlphaEvolve themselves only improving `R(4,15)` from `158` to `159` (a 158-vertex witness, not a 159-vertex one), the cumulative evidence at this point suggested `R(4,15) = 159` might be tight.

---

## apr27 continuation

Continued from artifacts already present in this repo, under run tag `apr27_codex_r4_s15_99`.

- Re-verified the bundled 158-vertex witness with `verify.py`.
- Re-verified the current 159-vertex near-frontier seed from `run_045_pairscan25/best_verify.txt`: exact `18` conflicts (`12` `K4` + `6` `I15`). Trusted as a near-miss, not logged as an improvement.

### Phase 8a: Exact ball enumeration

- Added `search_release_edges.py`, a bounded full-graph edge-release searcher to test the hypothesis that freezing the 158-vertex base is now too restrictive. Smoke tests showed naive objective is too weak: low-temperature witness destruction quickly creates many unseen `K4`/`I15` and did not improve on exact `18`. Secondary line.
- Main wave: exact row-ball enumeration to avoid proxy mismatch. Active layout:
  - `run_012_ball19_r5_anchor18`: radius-5 exact ball on the 19-variable core around the exact-18 parent.
  - `run_013_ball24_r5_anchor18`: radius-5 exact ball on the 19-core plus five hot vertices.
  - `run_014_ball24_r5_anchor19`: same 24-variable radius-5 ball around the exact-19 alternative plateau point.
  - `run_015_ball28_r4_anchor18`: wider 28-variable radius-4 exact ball.
- After a couple of hours, all ball runs remained flat at exact `18` — supports a basin diagnosis rather than "needs more of the same".
- Hygiene fix: the supervisor briefly misparsed `INVALID:` as containing `VALID:` and appended a false `new_best` row. After the row was removed, the supervisor was tightened to require a line beginning with `VALID:`.

### Phase 8b: Full-graph exact-delta — the breakthrough family

Implemented `search_fullgraph_delta.py`, which no longer freezes the 158-vertex base or relies on broad CEGAR. It evaluates exact single-edge deltas in the full 159-vertex graph: removing an edge destroys `K4`s through that edge and may create `I15`s; adding a nonedge destroys `I15`s through that pair and may create `K4`s. The expensive part is exact 13-clique counting in common blue neighborhoods, but it is targeted only at edges appearing in current conflicts.

- `run_027_fullgraph_delta_smoke` immediately showed this is the first genuinely promising post-18 algorithm: exact conflicts improved by verified steps `18 → 16 → 14 → 12`. `verify.py` confirmed exact `12` (`7 K4 + 5 I15`).
- Continuation runs from the verified exact-12 graph reached exact `10` (multiple branches, with different K4/I15 splits).
- Continuation reached exact `8` (`5 K4 + 3 I15` on `run_033`, and `3 K4 + 5 I15` on `run_031`). New near-frontier plateau.
- Added a tabu window to prevent sideways plateau traversal from immediately undoing the same edge flips. `run_035` escaped exact `8` to verified exact `7` (`3 K4 + 4 I15`).
- `run_043_fullgraph_delta_tabu8_wide` reached verified exact `6` (`3 K4 + 3 I15`) — a new in-repo near-miss.
- The exact-6 state exposes a new basin barrier: best available single-edge move has positive delta (`+1`).

### Phase 8c: Compound moves and structural diagnostics

- `search_fullgraph_pair.py`: keep the full-graph formulation but evaluate two-edge joint flips using verifier-consistent local exact delta. First smoke from exact `6` did not improve.
- Wider two-edge enumeration also failed (best `8/8/9` across pool sizes 56/72/96).
- Generalized to compound moves with configurable `--move-size` and added `--score-mode exact`. Triple smoke tests still did not improve.
- Four-edge compound wave (`run_055/056/057`) also failed: best exact `11/22/11`. Rules out the simple "sample top-k edge pool and flip four edges" version.
- `search_fullgraph_hitset.py`: structural repair constrained to hit every current violated witness at least once. Long bounded hitset runs ended at `22/25/30` exact — forcing every current witness to be hit in one compound move creates too many new conflicts.
- SAT-based local repair diagnostics: `search_fullgraph_sat_repair.py` enumerates all possible conflicts under a bounded active edge set; full witness-vertex model is too expensive (possible `I15` enumeration explodes). Cheaper incremental version `search_fullgraph_cegar_repair.py` proved: with only current conflict edges, flips `1..10` are UNSAT; with the complete edge set on the 44 witness vertices, flips `1..12` are UNSAT. So exact-6 is **not** a small local exact-repair miss inside the current support.

### Phase 8d: Bounded uphill — the actual escape

The successful reflection was that exact-6 is separated by a low uphill barrier, not by a wide simultaneous repair. Extended `search_fullgraph_delta.py` with bounded uphill moves (`--max-uphill`, `--temperature`, `--patience`) while keeping exact verifier-consistent edge deltas and tabu.

- `run_066_delta_uphill_smoke` immediately broke the basin. It accepted a `+1` move from exact `6` to exact `7`, then descended through exact `5` and exact `4` to a verified exact `3` graph (`2 K4 + 1 I15`). This is now the dominant line.
- Continuation waves from exact `3` and exact `6`: early logs show exact-3 itself also has `best_delta=+1`; several runs cross to exact `4/5` and return to exact `3`, but no exact `2` yet.
- A hotter exact-3 wave (`run_075` through `run_082`) with larger top pools, longer tabu windows, and a higher `--max-uphill` budget produced the next breakthrough: `run_079_delta_uphill_exact3_hot` reached verified exact `2` (`1 K4 + 1 I15`). Trajectory: `3 → 6 → 4 → 3 → 2`.
- Exact-2 diagnostics: the `1 K4 + 1 I15` graph has conflicts `(0,19,66,158)` and one 15-independent set on `(13,27,30,41,44,58,75,87,99,111,123,129,140,157,158)`. CEGAR over the complete witness-vertex edge set proved no valid repair through 16 flips in that support.
- A second exact-2 basin with `2 K4 + 0 I15` was found at round 27 of `run_079`.
- Added `--random-candidates` so uphill walks can sample non-conflict edges as preparatory moves. A smoke test with random full-graph candidates found exact-2 sideways moves immediately.
- `run_087_delta_uphill_exact2_hot` reached verified exact `1` (`0 K4 + 1 I15`). The local best delta at that state is `+6`.
- CEGAR on the exact-1 graph over the final I15's 15-vertex support proved UNSAT for `1..20` flips, so the final obstruction is not an internal repair of that independent set.
- Random-candidate exact-1 runs initially kept taking delta-0 sideways moves; new runs were launched with `--allow-sideways 0` to force crossing the `+6` final barrier before descending on the K4 side.
- Added anchored restart support (`--restart-above`, `--restart-stale`) so failed barrier attempts reset to the current best graph with fresh tabu and uphill budget.
- Structural analysis of the final I15 found a special edge: adding red edge `(125,150)` destroys the lone I15 and creates exactly one K4, yielding a different exact-1 basin (`1 K4 + 0 I15`, K4 `(8,125,150,152)`). Confirmed by `verify.py`.
- Both exact-1 anchors were used as parents; `run_125_delta_exact1_plateau_wide` explored zero-delta exact-1 transitions directly with a very wide random candidate pool.

### Phase 9: NEW SOTA R(4,15) ≥ 160

**Status correction and verified breakthrough**: an older uphill branch, `run_070_delta_uphill_exact3`, had in fact reached total `0` at round `67`. Re-running the read-only `verify.py` on `experiments/r4_s15/apr27_codex_r4_s15_99/run_070_delta_uphill_exact3/best_verify.txt` returned `VALID` with exact `0 K4 + 0 I15` on `159` vertices.

The graph was copied to `experiments/r4_s15/apr27_codex_r4_s15_99/sota_159_verify.txt`, and `results.tsv` records this as a `new_best`, **proving R(4,15) ≥ 160**.

This changes the live search target. The exact-1 plateau / BFS work is now mostly diagnostic for understanding why the final barrier was hard.

### Phase 10: n=160 extension

- Started the first n=160 extension wave from `sota_159_verify.txt`. Added `make_extend_seed.py` to convert the verified 159 graph into the internal row-search format and create randomized 160-vertex parent rows. Launched `run_128_extend160_row`, `run_129_extend160_sparse`, `run_130_extend160_dense`. Early `verify.py` checks gave exact conflicts `1098`, `1334`, `1559`, all K4-side and no I15.
- Generic row beam was too slow per step on n=160. Added `search_extend160_delta_row.py`, a specialized row repairer for the "valid base plus one new vertex" case. Since the 159-base has no internal conflicts, it can score K4 conflicts as red triangles inside the new vertex's neighbor set and protect against I15 by checking for blue 14-cliques in the non-neighbor set. `run_131_extend160_delta_row` improved to verified exact `65` (`65 K4 + 0 I15`) on n=160.
- `run_131` then plateaued for a long time. Structural lesson: all remaining conflicts still pass through the new vertex, but deleting more new-row edges is constrained by imminent blue 14-cliques. The next step is to leave the row-only manifold.
- Launched many full-graph exact-delta/uphill walks from the verified exact-65 n=160 graph. Initial best single-edge delta is `−5`, so exact-65 is not a full-graph basin. Several branches reached internal exact `55` quickly; `verify.py` confirmed `run_136_n160_fullgraph_delta65/best_verify.txt` at exact `55` (`53 K4 + 2 I15`).
- The same wave continued: verified `52` (`48 K4 + 4 I15` on `run_154`).
- `run_136` reached verified exact `49` (`44 K4 + 5 I15`). Best descent: `65 → 60 → 55 → 52 → 50 → 49`.
- Wave reached verified exact `48` (`41 K4 + 7 I15` on `run_140`). Progress slower now and the I15 side is rising.
- Continued improvement: verified exact `46` (`40 K4 + 6 I15` on `run_148`).
- `run_144` improved further to verified exact `44` (`41 K4 + 3 I15`) — a better I15 balance.
- `run_144` continued to verified exact `43` (`39 K4 + 4 I15`).
- `run_144` improved again to verified exact `42` (`37 K4 + 5 I15`).
- After more runtime, the best n=160 near-miss is `run_148_n160_fullgraph_delta65/best_verify.txt`: verified exact `33` (`31 K4 + 2 I15`).
- The first full-graph wave was very productive (`65 → 33`), but many branches later drifted into `90+` conflict states while retaining their saved best. The next scheduling change relaunched from the exact-33 graph with tighter anchored restarts, lower drift ceiling, and shorter stale windows.
- Anchored exact-33 relaunch: early logs show exact-33 has no immediate negative single-edge delta — best moves are mostly zero-delta and sometimes `+1`. Suggests a plateau/basin rather than insufficient runtime.
- Two verified exact-32 candidates: `run_164` has `30 K4 + 2 I15`; `run_177` has `32 K4 + 0 I15`. The more balanced `run_164` candidate logged in `results.tsv`.
- `run_164` continued improving: verified exact `31` on n=160 (`30 K4 + 1 I15`). Current best n=160 near-miss.
- After several more hours, exact `30` confirmed: `run_164` (`28 K4 + 2 I15`), and `run_169` / `run_173` each at `29 K4 + 1 I15`.
- Launched a focused exact-30 continuation wave (`run_196` through `run_203`) after retiring stale/high-drift branches; wider sideways budgets and a hotter range of uphill settings, alternating across all three verified exact-30 parents.

## Current frontier

- 🏆 **R(4,15) ≥ 160 NEW SOTA** (`experiments/r4_s15/apr27_codex_r4_s15_99/sota_159_verify.txt`) — verified by `verify.py`: 0 K4 + 0 I15 at n=159.
- 🎯 **n=160 best so far: 30 conflicts** (`run_164/169/173_*` near-miss) — pursuing R(4,15) ≥ 161.
- Active strategy: full-graph exact-delta with bounded uphill moves and anchored restarts, alternating across multiple verified exact-30 parents.

## Algorithm summary

The two algorithmic moves that drove the SOTA:

1. **Full-graph exact-delta search** (`search_fullgraph_delta.py`): no base freezing. For every candidate edge, compute the exact `ΔK4 + ΔI15` using bitmask induced subgraphs and exact 13-clique counting in common blue neighborhoods. Refresh tabu and candidate pools each round.
2. **Bounded uphill walks**: allow temporary `+k` increases (small `k`, e.g. `1..3`) to cross narrow plateaus and ridges. Combined with anchored restarts when the walker drifts too far above the best-so-far.

Diagnostics that mattered:

- The early SAT proof that the bundled 158-vertex base is **not one-vertex-extendable**. This stopped a long line of one-vertex-only local search that was provably aiming at an unreachable target.
- Restricted CEGAR proofs that exact-6 / exact-2 / exact-1 are not small local exact-repair misses inside their own conflict supports.
- Hash-comparing converged incumbents to decide when post-`18` beam workers were producing identical graphs and were therefore wasting effort.
- Always re-checking each claimed best directly with the read-only `verify.py` before logging to `results.tsv`.
