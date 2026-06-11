---
name: germinal
description: >
  De novo antibody and nanobody (VHH) design with Germinal. Use this skill when:
  (1) Designing epitope-targeted nanobodies or scFvs, (2) Needing CDR design on a
  fixed framework, (3) Working on antibody-format binders rather than miniproteins.

  For miniprotein binders, use binder-design (BoltzGen, BindCraft, RFdiffusion, Mosaic).
  For structure validation, use boltz or chai.
license: MIT
category: design-tools
tags: [antibody, nanobody, vhh, scfv, binder]
biomodals_script: modal_germinal.py
---

# Germinal Antibody and Nanobody Design

[Germinal](https://github.com/SantiagoMille/germinal) is an open pipeline for
epitope-targeted de novo antibody and nanobody design. It hallucinates CDRs on a
fixed framework, designs sequences with AbMPNN, and cofolds with a structure
predictor (it downloads AlphaFold-Multimer params). Runnable through biomodals.

The biomodals author notes Germinal is finicky and suggests BoltzGen for general
binder design; treat Germinal as the antibody-format option, not a default.

## Prerequisites

| Requirement | Value |
|-------------|-------|
| Runner | Modal (biomodals) |
| GPU | H100 (default; `GPU` env var) |
| Setup | See [Getting started](../../docs/getting-started.md) |

## How to run

```bash
git clone https://github.com/hgbrian/biomodals && cd biomodals

uv run --with modal --with PyYAML modal run modal_germinal.py \
  --target-yaml target_example.yaml \
  --max-trajectories 1 \
  --max-passing-designs 1
```

## Key parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `--target-yaml` | required | Target config (target_name, target_pdb_path, target_chain, binder_chain, target_hotspots, length) |
| `--run-type` | `vhh` | `vhh` (nanobody) or `scfv` |
| `--max-trajectories` | 100 | Trajectories to run |
| `--max-passing-designs` | 10 | Stop after this many passing designs |
| `--out-dir` | `./out/germinal` | Output directory |

## Target YAML

```yaml
target_name: PDL1
target_pdb_path: target.pdb
target_chain: A
binder_chain: B
target_hotspots: "45,67,89"
length: 120
```

## Decision tree

```
Antibody-format binder?
│
├─ Nanobody / VHH → germinal (run-type vhh) or mber
├─ scFv → germinal (run-type scfv)
└─ Miniprotein (not antibody) → binder-design (boltzgen, bindcraft, mosaic)
```

For VHH nanobodies, biomodals also has `modal_mber.py` (mBER) and `modal_iggm.py`
(IgGM) as alternatives.

## Troubleshooting

| Issue | Cause | Fix |
|-------|-------|-----|
| Pipeline fails early | Missing PyYAML | Add `--with PyYAML` to the invocation |
| No passing designs | Hard epitope or low budget | Raise `--max-trajectories` |
| OOM | Large target | Use the default H100 or trim the target |

---

**Next**: Validate with `boltz` or `chai`, rank with `ipsae`, filter with `protein-qc`.
