---
name: protenix
description: >
  Structure prediction with Protenix, an open AlphaFold3 reproduction. Use this
  skill when: (1) Predicting complex structures with an AF3-class model,
  (2) Wanting an open alternative to AF3 alongside Boltz and Chai,
  (3) Validating designed binder-target complexes.

  For QC thresholds, use protein-qc. For ipSAE ranking, use ipsae.
license: MIT
category: design-tools
tags: [structure-prediction, validation, alphafold3, open-source]
biomodals_script: modal_protenix.py
---

# Protenix Structure Prediction

[Protenix](https://github.com/bytedance/Protenix) is ByteDance's open PyTorch
reproduction of AlphaFold3 (Apache 2.0). It is an AF3-class complex predictor, useful
next to `boltz` and `chai` for cross-checking designed complexes. Runnable through
biomodals.

**Use Protenix-v2 for antibody-antigen complexes.** The v2 model (464M params, April
2026) adds 9 to 13 percentage points of antibody-antigen accuracy over v1 at the
DockQ > 0.23 threshold and is more sample-efficient (v2 at 5 seeds exceeds v1 at 1000).
Select it with `--model-name protenix-v2`. For general complexes, the v1 base model is
fine.

## Prerequisites

| Requirement | Value |
|-------------|-------|
| Runner | Modal (biomodals) |
| GPU | L40S (default; `GPU` env var) |
| Setup | See [Getting started](../../docs/getting-started.md) |

## How to run

```bash
git clone https://github.com/hgbrian/biomodals && cd biomodals

printf '>protein|A\nMAWTPLLLLLLSHCTGSLSQ...\n' > target.faa

uv run --with modal modal run modal_protenix.py \
  --input-faa target.faa \
  --seeds 42 \
  --no-use-msa
```

## Key parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `--input-faa` | one required | FASTA input (or `--input-json`) |
| `--seeds` | `42` | Comma-separated seeds |
| `--use-msa` / `--no-use-msa` | MSA on | Pass `--no-use-msa` for single-sequence |
| `--model-name` | v1 base | Set `protenix-v2` for antibody-antigen complexes |
| `--use-mini` | off | Switch to the smaller `protenix_mini` model |
| `--out-dir` | `./out/protenix` | Output directory |

## When to use Protenix vs Boltz vs Chai

| Need | Tool |
|------|------|
| Affinity head (small molecules) | boltz (Boltz-2) |
| Fastest, ligand support | chai |
| Open AF3 reproduction | protenix (v1 base) |
| Antibody-antigen complexes | protenix-v2 |

Ranking a shortlist across more than one predictor is more reliable than trusting a
single model.

## Troubleshooting

| Issue | Cause | Fix |
|-------|-------|-----|
| Missing input error | No `--input-faa`/`--input-json` | Provide one |
| Slow run | MSA enabled | Add `--no-use-msa` |
| OOM | Large complex | Use `--use-mini` or a larger GPU |

---

**Next**: Rank with `ipsae`, filter with `protein-qc`.
