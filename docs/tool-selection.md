# Tool Selection Guide

Master decision tree for choosing the right protein design tool.

## Quick decision tree

```
What are you designing?
│
├─ Miniprotein binder (60-100 AA)
│  ├─ Need diversity/exploration → rfdiffusion
│  ├─ Need high success rate → bindcraft
│  └─ Need all-atom precision → boltzgen
│
├─ Sequence design (have backbone)
│  ├─ Standard design → proteinmpnn
│  ├─ Ligand in binding site → ligandmpnn
│  └─ Need solubility → solublempnn
│
└─ Structure prediction (validation)
   ├─ Need open weights → boltz
   └─ Need highest accuracy → alphafold
```

## Tool comparison

### Design tools

| Tool | Method | Strengths | Weaknesses | Best For |
|------|--------|-----------|------------|----------|
| rfdiffusion | Diffusion | High diversity, fast | Needs ProteinMPNN | Exploration |
| bindcraft | End-to-end | High success rate | Less diverse | Production |
| boltzgen | All-atom diffusion | Side-chain aware | Slower | Precision |

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
| boltz | No | ~15-30s | Open weights |
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
