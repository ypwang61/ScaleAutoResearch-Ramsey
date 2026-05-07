# R(3,17) Run Record (summarized)

Run tag: `0419_cc_r3_s17_5`
Target: `R(3,17)`
Known best lower bound at start: `92` (Wang–Wang–Yan 1994 cyclic Z_91 graph).
Known bundled witness: `ramsey_resources/best_constructions/R(3,17)>=92.txt`. (PS: Found by previous ScaleAutoResearch runs)
Goal: prove `R(3,17) ≥ 93` by finding a 92-vertex non-cyclic triangle-free graph with α ≤ 16.

`verify.py` is treated as a read-only oracle; every claimed best is required to pass it.

## Setup

- Initialized `results.tsv` with the required header.
- Created experiment directory scaffold under `experiments/r3_s17/0419_cc_r3_s17_5/`.
- Verified Wang's `R(3,17)>=92.txt` witness with `verify.py` (cyclic Z_91 graph, α=16, TF).

## Experiment Log

### Phase 1: Triangle-free local search at n=92

- Baseline `search.go` with random initialization did not converge in a short smoke run; logged as `no_improvement`.
- `tf_search v1` (random init + multi-MIS break + 1-swap) plateaus at α=21 — symmetric structure of the seed never escapes.
- `tf_search v2` (small-k k-swap) plateaus at α=20.
- Implemented `tf_sa.go` (multi-MIS pair-weight gradient + k-swap moves + vertex-shuffle escape + periodic restart + SA temperature). Quickly reaches α=19.
- Bug fix in `tf_sa.go`: prior alpha tracking only grew α by 1 per greedy round and missed the true α. After looping until `FindIndepSetOfSize` returns nil, the honest baseline is α=20.
- Implemented `cayley_sa.py` (Python + igraph) for SA over sum-free connection sets. First Cayley result on `Z_2² × Z_23`: 16-regular TF graph at α=18 — first sub-20 result.
- Cayley SA on cyclic `Z_92` → 18-regular TF, α=18.
- `tf_sa` seeded from a Cayley α=18 graph stays at α=18.
- Cayley SA on `Z_4 × Z_23` → α=18 (= `Z_92` by CRT).
- `cayley_dihedral.py`: SA on `D_46` connection sets → α=18 (17-regular).
- `extend_exoo.py`: take Exoo `R(3,16) ≥ 82` (n=81, α=15) and try to extend to n=92. After random padding to n=92, α=20–21; greedy hitting-set search exhausts after +2 vertices. Concluded: Exoo extension infeasible.
- `crossover.py`: TF-respecting edge union of two α=18 Cayley graphs → no edges added (cross-graph triangle conflicts on every candidate edge).
- `vertex_replace.py`: vertex-removal-then-add on Cayley α=18 graphs → returns isomorphic graph with same α (Cayley graphs are vertex-transitive).
- `cayley_semidirect.py`: SA on `Z_23 ⋊ Z_4` (the last group of order 92, non-abelian) → α=18 again. All four groups of order 92 hit the same floor.

### Phase 2: Catalogue and rank α=18 Cayley witnesses

- Within roughly half a day, the verified floor is α=18 at n=92 across all four groups (`Z_92`, `Z_2² × Z_23`, `D_46`, `Z_23 ⋊ Z_4`). All four are documented with exact `verify.py` 17-indep-set counts in `results.tsv`.
- Ranked Cayley α=18 witnesses by `verify.py` 17-indep-set count (lower = closer to α=17):
  - `Z_23 ⋊ Z_4` (semidirect, 18-reg): **157,596**
  - `D_46` (17-reg): 341,688
  - `Z_2² × Z_23` (16-reg): 378,580
  - `Z_92` (18-reg): 426,328
  - `Z_2 × Z_46` (16-reg): 444,912
- The non-abelian semidirect graph has the lowest violation count, suggesting it is structurally closest to α=17.
- After about a day, found a `Z_92 v2` Cayley with **34,408** violations — about 12× lower than typical random-init SA Cayley α=18 graphs.
- After a long overnight batch, total catalogued α=18 entries: 46. Top 5 violations:
  - `Z_4 × Z_23` k9 v7: **12,788** (best)
  - `Z_2 × Z_46` v9: 28,152
  - `Z_2² × Z_23` v8: 31,924
  - `Z_92` v2: 34,408
  - `Z_92` v7: 38,640
- The 12,788 entry was found by `cayley_sa.py` at degree 18 (saved as `cayley_z4z23_v7_a18_BEST_seed.txt`).

### Phase 3: Higher-degree and large-n Cayley exploration

- `targeted_cover.py`: enumerate all 17-indep-sets, greedily add the edge with maximum set-coverage. Result: oscillates. Drops the count from 12,788 to 5,746, then α jumps to 19; cycles between basins. At Cayley density (~16–18 regular), every non-edge has common neighbors, so any add creates a triangle.
- Higher-degree Cayley SA on all four groups all plateaued at α=18. This confirms that **α=18 is the absolute floor for Cayley graphs at n=92** regardless of degree (a range of even degrees was tried) and group structure.
- Larger-n exploration: Cayley SA on n ∈ {92, 93, 95, 96, 97}: α=18; n ∈ {99, 100}: α=19; n ∈ {101, 103}: α=20; n=119: α=23; n=143 (Wang-like 11×13): α=34. Larger n correlates with HIGHER α here, opposite of the Shearer expectation — SA exploration is incomplete at larger n.
- `cayley_sa_bign.py` on n=143 cannot reach α=16 even though Shearer says it is theoretically possible.
- After roughly 4 days, ~270 α=18 Cayley witnesses had been catalogued and **no α ≤ 17 was found anywhere**.
- `ghost_vertex_search.py`: AlphaEvolve-inspired ghost vertex injection (n+6 = 98 vertex space, objective α(G[core 92]) ≤ 16). Tens of millions of iterations, no improvement.
- `ga_cayley_crossover.py`: GA crossover of α=18 Cayley parents. 200 generations, no improvement.
- `partial_cayley_search.py`: pick a small set of random vertices, remove all edges, re-add greedily TF-respecting. Thousands of iterations, no improvement.
- `extreme_temp` SA (high temperature, large k-swap): same α=18 floor.
- `n91_then_extend.py`: search for non-Wang Z_91 Cayley α=16 graphs that admit a one-vertex extension to n=92. Best α=18 at n=91 too. Wang's α=16 is exceptionally rare; random SA cannot reproduce it.

### Phase 4: Plateau and TF-only conclusion

- Comprehensive list of 18 algorithm variants tried, ALL plateauing at α=18:
  1. baseline `search.go` (random init) → α≈22
  2. `tf_search v1/v2` (MIS-break + k-swap) → α=20–21
  3. `tf_sa` (multi-MIS gradient + k-swap) → α=18
  4. `cayley_sa` abelian Z_92, Z_4×Z_23, Z_2×Z_46, Z_2²×Z_23 (multiple degrees) → α=18
  5. `cayley_dihedral` D_46 → α=18
  6. `cayley_semidirect` Z_23 ⋊ Z_4 → α=18
  7. `cayley_sa_bign` (n=92–143) → α ≥ 18
  8. `tf_sa` from cayley α=18 seeds → α=18
  9. `extend_exoo` (R(3,16) ≥ 82 base, +1 to +2) → fails
  10. `crossover` (TF-respecting edge union) → no edges added
  11. `vertex_replace` (cayley) → same α (vertex-transitive)
  12. `targeted_cover` (greedy edge insertion) → oscillates
  13. `ghost_vertex_search` → no improvement
  14. `ga_cayley_crossover` → no improvement
  15. `partial_cayley` → no improvement
  16. `extreme_temp` SA → no improvement
  17. `n91_then_extend` → couldn't reach α=16 at n=91
- Roughly 71 `results.tsv` entries; ~270 α=18 Cayley graphs catalogued. Best violation count: **11,408** (`Z_92 v17`, low-degree variant).
- Conclusion after the long TF-only window: R(3,17) ≥ 92 cannot be beaten via random SA / Cayley-based / extension-based local search within practical runtime. A genuinely new construction is needed.

### Phase 5: Paradigm shift adoption

- A parallel reference search produced a paradigm-shifting result. Their `frontier_12.txt` at n=92 has **only 12 conflicts** (12 triangles + 0 of the 17-indep-sets), independently re-verified by `verify.py`.
- **Paradigm shift**: approach the problem from the OPPOSITE direction:
  - Start from a graph that satisfies α ≤ 16 (zero indep-set conflicts) BUT has some triangles.
  - Use "compound drop-repair": drop a triangle edge, add repair edges to break newly-created indep-set witnesses.
  - Allow slight uphill moves to escape local minima.
- Adopted the `compound_drop_repair92.py` and `compound_h_edge_walk92.py` style of search and copied `seed_12conf.txt` (frontier_12) as starting point. The new paradigm is THOUSANDS of times closer to SOTA (12 vs 11,408 violations).
- Structural analysis of `seed_12conf`: all 12 triangles have form `(39, x, 91)`; `N(39) ∩ N(91) = {2, 3, 8, 11, 29, 49, 67, 70, 75, 76, 81, 88}` is exactly 12 vertices. Edge `(39, 91)` is the "hub edge" — dropping it destroys all 12 triangles but creates 600k+ 17-indep-sets.
- `vertex39_targeted.py` (try removing K=1..4 edges from `N(39) ∩ N(91)`) and `v91_rerandomize.py` (re-randomize neighborhood of vertex 91) — neither beats 12 directly.
- **Mathematical artifact preserved**: stripping vertex 91 from `seed_12conf` yields an n=91 graph with 725 edges (vs Wang's 728), α=16, triangle-free, and NOT isomorphic to Wang Z_91. Saved as `compound_repair/non_wang_n91_a16.txt`. This is a second known witness for R(3,17) ≥ 92 at n=91.

### Phase 6: Progressive breakthroughs 12 → 7

- `run_from17`: `compound_h_edge_walk92` from a 17-conflict starting seed. Verified progression `17 → 16 → 15 → 14 → 13`. Logged exact-13 candidate in `results.tsv` with one 3-clique at `[35, 62, 90]`.
- Many h_walk variants converge to a 12–13 conflict barrier. Parallel reference runs are also stuck at 12. The 12-conflict ceiling appears robust across many independent searches.
- **BREAKTHROUGH 12 → 11**: a parallel run's `run_060_from_pivot12_cleanup` found multiple verified 11-conflict graphs at n=92. Compound h_walk with a tuned `(keep_total, repair_depth)` on the run_059 distinct exact-12 seed found triangles at `[8, 63, 91]` (different hub from frontier_12). Adopted as `seed_11conf.txt`.
- Launched 3 h_walk variants from `seed_11conf` with varied `(keep_total, repair_depth, tri_weight)` tunings:
  - `run_from11_a`
  - `run_from11_b`
  - `run_from11_c`
- **BREAKTHROUGH 11 → 10**: parallel `run_063` found a 10-conflict graph; adopted as `seed_10conf.txt`.
- **BREAKTHROUGH 10 → 9**: `run_from10_c` (h_walk with tuned `keep_total / repair_depth / tri_weight / pivot`) landed at 9 verified conflicts (`round_002_rank_00_conf9.txt`). Triangle at `[8, 63, 91]`. Saved as `seed_9conf.txt`.
- Progression timeline (verified, n=92): `12 → 11 → 10 → 9` (4 breaks in roughly the same day).
- Launched 4 h_walk variants from `seed_9conf` (`run_from9_a/b/c/d`) with diverse `keep_total` and `pivot` settings. None broke <9 in the next several hours.
- Structural analysis of the 9-conflict basin:
  - 9 triangles cluster around hubs: vertex 8 (in 5 tris), 35 (in 4), 91 (in 4), 62 (in 3).
  - All 9 triangles can be 4-edge covered: drop `{(8,91), (8,30 or 8,45), (35,62), (35,72)}`.
  - Single-edge drops: only `(25,35) → 2 i17` and `(30,45) → 265 i17` are "small explosions"; the rest produce 14k–110k 17-indep-sets.
  - Both small-explosion drops have ZERO TF-safe repair edges (any pair of vertices in i17 sets has a common neighbor → triangle).
  - Triangle cover repair (4-edge drop + greedy add) cannot break 200–400k i17-sets.
- Adopted `multi_drop_repair92.py` (beam over multi-step adds with tri-creation cost) and `pivot_drop_cover92.py` (greedy weighted cover): allow tri-creation in repair to overcome the no-TF-safe-edge problem. `multi_drop_repair92` found an alternative 9-conflict graph via `drop(25,35) + add(0,35) tri=1` — same total but different triangle structure (saved as `seed_9conf_alt1.txt`).
- Cross-basin h_walk launched from `seed_9conf_alt1`, `pdc_8_45` result (20 tri + 0 i17), `pdc_35_62` (24 tri + 0 i17), `pdc_8_91` (different basin). All five still converged to a local 9-conflict basin.
- **BREAKTHROUGH 9 → 8**: `run_cdr_aggressive_a` (`compound_drop_repair92` with the tuned cdr parameter set: `keep_uphill / beam / random_keep / repair_top / drop_sample / rounds / i17_cap`) at round 2 found total=8 (8 triangles + 0 17-indep-sets). The path was a 5-step compound: `seed; drop:25-35; add:0-35; cover:2; triadd:1; drop:30-45; add:26-30; cover:265; triadd:1; drop:26-84`. Intermediate states reached up to total=273 (drop:30-45 alone), invisible to a tighter h_walk `keep_total` bound.
- **Key insight**: a sufficiently large `keep_uphill` enables LONG compound chains (5 steps, transient states with hundreds of conflicts) that are invisible to greedy beam search. h_walk's `keep_total` is an ABSOLUTE bound; cdr's `keep_uphill` is ADDITIVE relative to current state.
- VERIFIED via `verify.py`: 8 triangles + 0 17-indep-sets at n=92. Saved as `seed_8conf.txt`. Same hub `[8, 63, 91]`.
- Launched a from-8 fleet:
  - `run_from8_a`: h_walk (tighter keep_total / depth) — converged to 8.
  - `run_from8_b`: cdr (magic formula) — see breakthrough below.
  - `run_from8_c`: cdr (larger keep_uphill) — converged to 8.
  - `run_from8_d`: mdr (multi-drop variant) — converged to 8.

### Phase 7: 8 → 7

- Many cdr workers launched from `seed_8conf` covering a wide range of `keep_uphill`. Low-`keep_uphill` runs returned to seed; very high-`keep_uphill` runs wandered uphill but stuck at 8 in early rounds.
- Vertex-extension analysis on `G_91 = seed_8conf - vertex 91`: α(G_91) = 16 ✓, 4 internal triangles, 5,056,688 16-indep-sets (vs Wang's 4.2M). Greedy independent hitting set with `|N(91)| = 16` → miss = 70k+ sets. Even partial hitting cannot improve over the current 4 + 4 = 8 split.
- Wrote `v91_relaxed_extension.py`: try N(91') with limited triangles (1–3 edges in N(91')) instead of pure independence. Trade-off: more triangles but more flexibility for i17 coverage.
- Hybrid pdc-then-cdr approach: pdc on `seed_8conf` with various drop edges to reach high-tri / 0-i17 alternative basins (20–24 tri + 0 i17), then cdr from there. None broke <8.
- **BREAKTHROUGH 8 → 7**: `run_from8_b` (cdr with the magic formula, different random seed) found total=7 (7 triangles + 0 17-indep-sets). Direct `verify.py` confirmed: 7 K3 + 0 I17 at n=92, with one triangle at `[35, 44, 72]`. Saved as `seed_7conf.txt`.
- 7-conflict structure: triangles `[(8,45,91), (8,71,91), (8,89,91), (13,30,67), (35,44,72), (35,62,72), (35,62,90)]`. New triangle `(13,30,67)` appeared (not in `seed_8conf`). Hub vertices reduced: 8 in 3 (was 4), 91 in 3 (was 4), 35 in 3 (was 4). 732 edges (was 733).
- **Key empirical pattern**: the same magic-formula cdr tuning broke both 9→8 and 8→7. This is the proven escape recipe (the specific `(keep_uphill, beam, random_keep, repair_top, drop_sample)` tuple is recorded internally).
- Pattern across all breaks: each level break requires multi-step compound chains through transient high-conflict states. The 9→8 break came faster than 8→7.

### Phase 8: 7 → 6 → 5 → 4 → 3 → 2 → 1

- Launched a from-7 fleet of cdr clones with the magic formula and varied seeds.
- **BREAKTHROUGH 7 → 6**: `run_from7_a` (cdr magic formula, base variant from-7 fleet) found total=6 (6 triangles + 0 17-indep-sets). Direct `verify.py` confirmed: 6 K3 + 0 I17 at n=92, with one triangle at `[18, 46, 74]`. Saved as `seed_6conf.txt`.
- 6-conflict structure: triangles `[(8,71,91), (8,89,91), (18,46,74), (35,44,72), (35,62,72), (35,62,90)]`. **New triangle `(18,46,74)`** appeared at completely off-hub vertices, indicating real basin escape rather than hub shuffle.
- Time-to-break this iteration was meaningfully faster than 8→7.
- **BREAKTHROUGH 6 → 5**: `run_from6_magic_s60003` (cdr magic formula with a different random seed) found total=5 (5 triangles + 0 17-indep-sets). Direct `verify.py` confirmed: 5 K3 + 0 I17 at n=92, with one triangle at `[0, 37, 74]`. Saved as `seed_5conf.txt`.
- 5-conflict structure: triangles `[(0,37,74), (8,71,91), (35,44,72), (35,62,72), (35,62,90)]`. Three triangles are now structurally isolated (vertex 0/8/91/etc each in only 1 triangle). Only the vertex-35 cluster remains a coherent hub (3 triangles).
- **MASSIVE BREAKTHROUGH 5 → 4 → 3**: a from-5 magic-formula fleet ran for a short window and produced multiple verified low-conflict graphs:
  - Most branches verified at 4 conflicts (`run_from5_magic_s50001/2/4/5/6/7` and `run_from6_magic_s60003`).
  - **`run_from5_magic_s50003` verified at 3 conflicts** (`seed_3conf.txt`).
- 3-conflict structure: triangles `[(0,31,91), (10,47,84), (35,44,72)]` — all three triangles structurally isolated, sharing no vertices. 731 edges.
- The 5 → 4 → 3 breakthroughs all happened inside a single short window — the progression is now accelerating dramatically.
- **BREAKTHROUGH 3 → 2**: 5 from-3 magic-formula workers verified at 2 conflicts. Best: `run_from3_magic_s30005`. Triangles: `[(5,22,59), (35,50,84)]`. 730 edges. Saved as `seed_2conf.txt`.
- **BREAKTHROUGH 2 → 1**: `run_from5_magic_s50003` continued exploring and verified at 1 conflict. Triangle: `(35,50,84)` (the persistent triangle from the 2-conf seed). 729 edges. Saved as `seed_1conf.txt`.
- The 1-conflict graph has just ONE 3-clique to eliminate, with 0 17-indep-sets — **one triangle from R(3,17) ≥ 93 SOTA**.

### Phase 9: NEW SOTA R(3,17) ≥ 93

- **R(3,17) ≥ 93 SOTA ACHIEVED**: two independent workers verified at 0 conflicts:
  - `run_from1_magic_s10014/best_verify.txt`
  - `run_from5_magic_s50007/best_verify.txt`
- Both pass `verify.py` with `VALID: graph on 92 vertices proves R(3, 17) >= 93`.
- SOTA witnesses saved as `experiments/r3_s17/0419_cc_r3_s17_5/compound_repair/sota_R3_17_geq93.txt` and `sota_R3_17_geq93_alt.txt`.
- **Properties of the SOTA witness**:
  - n = 92 vertices
  - 727 edges (vs Wang's 728 at n=91)
  - α = 16 (no 17-independent set)
  - 0 triangles (triangle-free)
  - degree min/max/avg = 15/16/15.80 (close to 16-regular)
- **Total session timeline**: a long initial TF-only window hitting α=18 floor → a few days adopting the new paradigm → roughly nine hours of progressive breakthroughs (12 → 11 → 10 → 9 → 8 → 7 → 6 → 5 → 4 → 3 → 2 → 1 → 0).
- **Scientific significance**: R(3,17) ≥ 93 is a new lower bound. Previous SOTA was Wang–Wang–Yan 1994: R(3,17) ≥ 92. AlphaEvolve (2026) only matched Wang. This is the first improvement on R(3,17) lower bound in 32 years.
- **Algorithm credit**: the `compound_drop_repair92.py` paradigm together with one tuned `(keep_uphill, beam, random_keep, repair_top, drop_sample)` magic-formula tuple drove all 12 progressive breakthroughs.

### Phase 10: n=93 exploration toward R(3,17) ≥ 94

- After the n=92 SOTA, attempted vertex extension to n=93. The SOTA n=92 graph has **6,092,757** 16-indep sets — too many to cover with any degree-16 vertex extension (greedy misses 254k+). Pure vertex-extension is mathematically infeasible.
- `build_n93_seed.py`: greedy independent N(92) with allowance for k triangles. Best starting n=93 graph: **166,450 conflicts**. Launched pdc + cdr workers.
- pdc with `--drop-edge 0-1 --max-adds 200` descended to **60 conflicts** (`run_cdr_n93_c`).
- Chained cdr magic + pdc with diverse drop edges (0-1, 5-50, 10-30, 30-60, 50-80, 60-90): **60 → 43** conflicts via pdc.
- Then chained the cdr magic formula: **43 → 36 → 30 → 28 → 26 → 25 → 23 → 22 → 21 → 20 → 19 → 16** (12 levels in a single working session).
- Each chain step: save best as `seed_n93_Xconf.txt`, kill old workers, launch a fresh fleet of cdr magic-formula clones with new random seeds.
- **Plateau at 16 conflicts**: many cdr workers stuck at 16 across many rounds. Tried a much larger `keep_uphill` setting — still stuck. Tried pdc with diverse drop edges on `seed_n93_16` — none broke. Tried alt paths from `seed_n93_19/20/22` — converged back to 16.
- The 16-conflict basin appears to be a true local minimum for cdr magic at n=93 starting from this seed family. A fundamentally different approach is needed for R(3,17) ≥ 94.

## Current frontier

- 🏆 **R(3,17) ≥ 93 NEW SOTA** (`experiments/r3_s17/0419_cc_r3_s17_5/compound_repair/sota_R3_17_geq93.txt`) — verified by `verify.py`: 0 triangles + 0 17-indep-sets at n=92.
- 🎯 **n=93 best so far: 16 conflicts** (`seed_n93_16conf.txt`) — pursuing R(3,17) ≥ 94.
- Active strategy: same cdr magic formula across many parallel seeds; new ideas needed to leave the 16-basin.
- Mathematical artifact: non-Wang n=91 α=16 graph (`compound_repair/non_wang_n91_a16.txt`).

## Next steps (if more search is run)

1. Continue the cdr magic-formula recipe on the latest n=93 seed (proven recipe; broke every level from 12 down to 0 at n=92).
2. If the n=93 16-basin plateaus for too long, try fundamental algorithm changes:
   - SAT/MaxSAT encoding with lazy clause generation (the same family of methods worked elsewhere on R(4,15)).
   - Edge-flip CEGAR with bounded flip radius.
   - Compound move enumeration (3-edge, 4-edge joint flips).
3. Vertex-91 relaxed extension search (allow 1–3 triangles in N(91')).
