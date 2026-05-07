# ScaleAutoResearch Ramsey

[Yiping Wang](https://ypwang61.github.io/)

## Introduction

Verified graph witnesses and experiment records for new lower bounds on classical Ramsey numbers.

For Ramsey number `R(r, s)`, an `n`-vertex graph with no `r`-clique and no `s`-independent set proves:

```text
R(r, s) >= n + 1
```

The two new lower bounds in this repository are:

- **`R(3, 17) >= 93`** — improves the previous lower bound `R(3, 17) >= 92` of Wang-Wang-Yan (1994). **This is the first improvement on this lower bound in 32 years.** Google DeepMind's AlphaEvolve (2026)  matched the previous bound but did not beat it.
- **`R(4, 15) >= 160`** — improves the bound `R(4, 15) >= 159` from AlphaEvolve.

Both witnesses were found via Claude Code or Codex, scaled with the autoresearch pattern, and independently verified.

The witnesses are stored as plain adjacency-matrix text files:

- `R(3, 17) >= 93.txt`
- `R(4, 15) >= 160.txt`

## Verification

`verify_bounds.ipynb` is taken from AlphaEvolve ([Ramsey improved-bounds repository](https://github.com/google-research/google-research/tree/master/ramsey_number_bounds/improved_bounds)). 

## Experiment records

Summarized experiment records describing search progression, failed approaches, key algorithmic ideas, and verification checkpoints:

- `r3_17_record_summarized.md`
- `r4_15_record_summarized.md`

## Acknowledgements

This setup is inspired by:

- autoresearch: [karpathy/autoresearch](https://github.com/karpathy/autoresearch).
- [ThetaEvolve](https://github.com/ypwang61/ThetaEvolve) and [SimpleTES](https://github.com/wq-will/SimpleTES) for scaling batch sampling of LLM/agents in evolving problems.
- Google DeepMind's [AlphaEvolve](https://arxiv.org/abs/2603.09172).
- Survey: Radziszowski, [Small Ramsey Numbers](https://www.cs.rit.edu/~spr/ElJC/ejcram18.pdf).


Yiping Wang is supported in part by the [Amazon AI PhD Fellowship](https://www.amazon.science/news/amazon-launches-68-million-ai-phd-fellowship-program).

## License

[Apache License 2.0](LICENSE) 

## Citation

If you use these witnesses or experiment records, please cite this repository:

```bibtex
@software{scaleautoresearch_ramsey_2026,
  title  = {ScaleAutoResearch-Ramsey},
  author = {Wang, Yiping},
  year   = {2026},
  url    = {https://github.com/ypwang61/ScaleAutoResearch-Ramsey},
  note   = {Verified graph witnesses for new lower bounds on classical Ramsey numbers}
}
```
