# Tool Selection Guide

Master decision tree for choosing the right protein design tool.

## Quick decision tree

```
What are you designing?
│
├─ Miniprotein binder (60-100 AA)
│  ├─ Highest hit-rate, willing to tune → mosaic
│  ├─ Cheap, turnkey, all-atom default → boltzgen
│  ├─ Diversity/exploration → rfdiffusion
│  └─ End-to-end with validation → bindcraft
│
├─ Antibody / nanobody → germinal
│
├─ Sequence design (have backbone)
│  ├─ Standard design → proteinmpnn
│  ├─ Ligand in binding site → ligandmpnn
│  └─ Need solubility → solublempnn
│
└─ Structure prediction (validation)
   ├─ Need open weights → boltz, chai, or protenix
   └─ Need highest accuracy → alphafold
```

Hit-rate is target-dependent; see the `binder-design` skill for head-to-head results.

## Tool comparison

### Design tools

| Tool | Method | Strengths | Weaknesses | Best For |
|------|--------|-----------|------------|----------|
| mosaic | Gradient, multi-model | Highest competition hit-rate | Needs tuning, local JAX | Hard targets |
| boltzgen | All-atom diffusion | Cheap, turnkey, side-chain aware | Lower hit-rate on hard targets | Budget default |
| rfdiffusion | Diffusion | High diversity | Needs ProteinMPNN; not in biomodals | Exploration |
| bindcraft | End-to-end | Built-in AF2 validation | Less diverse | Production |
| germinal | Hallucination + AbMPNN | Antibody / nanobody formats | Finicky | scFv / VHH |

### Sequence tools

| Tool | Use Case | Temperature |
|------|----------|-------------|
| proteinmpnn | General | 0.1 (production), 0.2 (exploration) |
| ligandmpnn | Ligand-bound | 0.1-0.2 |
| solublempnn | Expression | 0.1 |

### Validation tools

| Tool | MSA | Speed | Best For |
|------|-----|-------|----------|
| chai | No | ~20-40s | Speed + ligands |
| boltz | No | ~15-30s | Open weights, Boltz-2 affinity |
| protenix | Optional | ~60s+ | Open AF3 reproduction |
| alphafold | Yes | ~60s+ | High accuracy |

## Recommended combinations

### Standard campaign
1. rfdiffusion (100-500 backbones)
2. proteinmpnn (8 sequences each)
3. chai or boltz (validation)
4. protein-qc (filtering)

### High-success campaign
1. bindcraft (100 designs, end-to-end)
2. protein-qc (filtering)

## See also

- [Standard Pipeline](standard-pipeline.md) - Full workflow
- [Troubleshooting](troubleshooting.md) - When things go wrong
