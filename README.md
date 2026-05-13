# ScaleAutoResearch for Ramsey: New Lower Bounds for R(3,17) After 32 Years

[Yiping Wang](https://ypwang61.github.io/)

## Outline

- **New lower bound after 32 years: `R(3, 17) >= 93`.** This improves the previous lower bound `R(3, 17) >= 92` of Wang-Wang-Yan (1994), the first improvement on this bound in 32 years. Google DeepMind's AlphaEvolve (2026) matched the previous bound but did not beat it.
- **Another new lower bound: `R(4, 15) >= 160`.** This further improves the `R(4, 15) >= 159` bound from AlphaEvolve.
- **Method:** Claude Code / Codex + simple autoresearch + scaling by width and depth.
- **Verification:** witnesses are plain adjacency-matrix text files, checked by the AlphaEvolve verifier notebook.

## Problem

For Ramsey number `R(r, s)`, an `n`-vertex graph with no `r`-clique and no `s`-independent set proves:

```text
R(r, s) >= n + 1
```

This repository contains the verified graph witnesses and summarized experiment records for the two bounds above.


## Method

The core method is deliberately simple: run many autonomous research agents, each following an autoresearch-style setup, and scale both width and depth.

Each agent session uses Claude Code or Codex and follows the same experiment scaffold: an immutable verifier, initial program, reference materials, a `program.md`-style instruction file that asks the agent to iterate indefinitely, a machine-readable `results.tsv`, and a human-readable `record.md` / summarized record for experiment progress.

Most attempts on open problems still fail, even with the strongest models. The main source of diversity is therefore many independent autoresearch agents running in parallel. During the search, stronger agents share progress with others based on measurable frontier quality, such as conflict counts. Some later runs directly inherit from or fork the experimental state of stronger agents. This sharing and inheritance are managed at the experiment level and surfaced through GitHub branches.

## Verification

`verify_bounds.ipynb` is taken from AlphaEvolve's [Ramsey improved-bounds repository](https://github.com/google-research/google-research/tree/master/ramsey_number_bounds/improved_bounds).

## Experiment records

Summarized records describing search progression, failed approaches, key algorithmic ideas, and verification checkpoints:

- `r3_17_record_summarized.md`
- `r4_15_record_summarized.md`

## Acknowledgements

This setup is inspired by:

- autoresearch: [karpathy/autoresearch](https://github.com/karpathy/autoresearch).
- Google DeepMind's [AlphaEvolve](https://arxiv.org/abs/2603.09172).
- Survey: Radziszowski, [Small Ramsey Numbers](https://www.cs.rit.edu/~spr/ElJC/ejcram18.pdf).
- [ThetaEvolve](https://github.com/ypwang61/ThetaEvolve) for scaling test-time compute on open problems.

Yiping Wang is supported in part by the [Amazon AI PhD Fellowship](https://www.amazon.science/news/amazon-launches-68-million-ai-phd-fellowship-program).

## License

[Apache License 2.0](LICENSE) 

## Citation

If you use these witnesses or experiment records, please cite this repository:

```bibtex
@misc{wang2026scaleautoresearchramsey,
  author       = {Yiping Wang},
  title        = {ScaleAutoResearch-Ramsey},
  howpublished = {https://github.com/ypwang61/ScaleAutoResearch-Ramsey},
  note         = {Verified graph witnesses and experiment records for new lower bounds on classical Ramsey numbers},
  year         = {2026}
}
```
