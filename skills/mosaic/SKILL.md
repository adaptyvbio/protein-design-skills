---
name: mosaic
description: >
  Multi-objective, gradient-based protein binder design with Mosaic. Use this
  skill when: (1) Composing several structure or sequence models into one design
  objective, (2) Optimizing binders against a custom loss rather than a fixed
  pipeline, (3) Wanting gradient descent over sequence space in the style of
  ColabDesign, RSO, or BindCraft but with interchangeable predictors, (4) Letting
  the optimizer choose the epitope instead of fixing hotspots.

  For an end-to-end binder pipeline with default filters, use bindcraft.
  For all-atom diffusion design, use boltzgen.
  For backbone-only generation, use rfdiffusion.
license: MIT
category: design-tools
tags: [design, gradient-optimization, multi-objective, jax, binder]
---

# Mosaic Multi-Objective Design

[Mosaic](https://github.com/escalante-bio/mosaic) (Escalante Bio) is a JAX framework
for "functional, multi-objective protein design using continuous relaxation." It
optimizes a soft sequence by gradient descent over a continuous relaxation of
sequence space, in the lineage of ColabDesign, RSO, and BindCraft, with one key
difference: it composes multiple learned objectives from different models in a single
differentiable loss.

## When Mosaic fits

Mosaic is a framework for custom objectives, not a one-click method. The README is
explicit: it "may require substantial hand-holding (tuning learning rates, etc),
often produces proteins that fail simple in-silico tests, [and] should be combined
with standard filtering methods." Reach for it when a fixed pipeline cannot express
the objective you need. For a turnkey binder run, use `bindcraft` instead.

## Prerequisites

| Requirement | Minimum | Recommended |
|-------------|---------|-------------|
| Python | 3.11+ | 3.11 |
| Framework | JAX with CUDA or TPU | JAX CUDA 12 |
| GPU VRAM | 24GB | 48GB+ (depends on predictors used) |

JIT compilation makes the first call to any loss slow; later calls are fast.

## Install

```bash
git clone https://github.com/escalante-bio/mosaic && cd mosaic
uv sync --group jax-cuda
uv run marimo edit examples/example_notebook.py
```

## Core idea

A design objective is built from `LossTerm` objects that you add and scale with plain
Python arithmetic, then hand to an optimizer.

```python
import mosaic.losses.structure_prediction as sp

# Compose a loss from interface, confidence, and inverse-folding terms
design_loss = (
    sp.BinderTargetContact()
    + sp.WithinBinderContact()
    + 0.05 * sp.TargetBinderPAE()
    + 0.05 * sp.BinderTargetPAE()
    + 0.025 * sp.IPTMLoss()
    + 0.1 * sp.PLDDTLoss()
)
```

Loss terms can wrap one model used several ways (for example a structure predictor
scoring both the binder-target complex and the binder as a monomer). Composing
different architectures also lowers the chance of finding adversarial sequences that
fool a single predictor.

### What you can compose

| Category | Options |
|----------|---------|
| Structure predictors | AF2, Boltz-1, Boltz-2, Protenix, OpenFold3, ESMFold2 |
| Generative / design | BoltzGen, Proteina-Complexa |
| Inverse folding | ProteinMPNN, SolubleMPNN, AbMPNN |
| Language models | ESM-2, ESM-C, AbLang, trigram |
| Property heads | Stability (megascale-trained) |

### Optimizers

| Optimizer | Use |
|-----------|-----|
| `simplex_APGM` | Default; proximal gradient / mirror descent on the probability simplex |
| `batched_simplex_APGM` | The same, vmapped over many designs |
| `gradient_MCMC` | Discrete moves for fine-tuning a sequence |

A reasonable `simplex_APGM` step size is about `0.1 * sqrt(binder_length)`.

## Worked example: ranking with ipSAE

The published [Nipah competition recipe](https://blog.escalante.bio/180-lines-of-code-to-win-the-in-silico-portion-of-the-adaptyv-nipah-binding-competition/)
optimizes a design loss on Boltz-2, then ranks candidates with a separate
multi-sample loss built from ipTM and ipSAE.

```python
ranking_loss = folder.build_multisample_loss(
    loss=1.00 * sp.IPTMLoss()
    + 0.5 * sp.TargetBinderIPSAE()
    + 0.5 * sp.BinderTargetIPSAE(),
    num_samples=6,
    recycling_steps=3,
)
```

Two practices from that work are worth carrying over:

- **Let the optimizer choose the epitope.** Asking for a binder, without fixing
  hotspots, can find a better interface than a manually chosen one.
- **Match filter stringency to assay throughput.** With high-throughput testing,
  filter lightly to keep diversity rather than applying heavy consensus filters that
  can reject good binders.

## Decision tree

```
Should I use Mosaic?
│
├─ Need a custom objective across multiple models? → Mosaic
├─ Want one-click binders with default filters?    → bindcraft
├─ Want all-atom diffusion design?                  → boltzgen
└─ Want backbone-only diversity?                    → rfdiffusion + proteinmpnn
```

## Troubleshooting

| Issue | Cause | Fix |
|-------|-------|-----|
| Designs fail simple in-silico checks | Under-constrained objective | Add inverse-folding and confidence terms; filter with `protein-qc` |
| Optimization unstable | Step size too large | Lower the `simplex_APGM` step size |
| First call very slow | JIT compilation | Expected; reuse the compiled loss across designs |
| OOM with large predictors | Several models in one loss | Use smaller predictors or a larger GPU |

---

**Next**: Validate designs with `boltz` or `chai`, rank with `ipsae`, then filter
with `protein-qc`.
