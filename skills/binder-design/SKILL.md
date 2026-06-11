---
name: binder-design
description: >
  Guidance for choosing the right protein binder design tool.
  Use this skill when: (1) Deciding between BoltzGen, BindCraft, or RFdiffusion,
  (2) Planning a binder design campaign,
  (3) Understanding trade-offs between different approaches,
  (4) Selecting tools for specific target types.

  For specific tool parameters, use the individual tool skills
  (boltzgen, bindcraft, rfdiffusion, etc.).
license: MIT
category: orchestration
tags: [guidance, tool-selection, workflow]
---

# Binder Design Tool Selection

## Decision tree

No single tool is best for every target. Pick by target type, what you want to
control, and available compute.

```
De novo binder design?
│
├─ All-atom design or ligand/small-molecule binding → BoltzGen
│   All-atom output (no separate ProteinMPNN step)
│   Single-step design (backbone + sequence + side chains)
│   Higher GPU requirement
│
├─ Diversity / exploration → RFdiffusion + ProteinMPNN
│   Maximum backbone diversity, two-step
│
├─ End-to-end with built-in validation → BindCraft
│   AF2 validation in the loop
│
├─ Custom multi-model objective → Mosaic
│   Gradient optimization, compose several predictors
│
└─ Antibody / nanobody (VHH)
    External tools, not yet packaged as skills:
    Germinal and RFantibody
```

## Tool comparison

| Tool | Strengths | Weaknesses | Best For |
|------|-----------|------------|----------|
| BoltzGen | All-atom, single-step, ligand-aware | Higher GPU requirement | All-atom and ligand binding |
| BindCraft | End-to-end, built-in AF2 validation | Less diverse | Production campaigns |
| RFdiffusion | High diversity, fast | Requires ProteinMPNN | Exploration, diversity |
| Mosaic | Compose any predictors, custom loss | Needs tuning, no default filters | Custom objectives |

## Example pipeline: BoltzGen → Chai → QC

BoltzGen provides all-atom design with built-in side-chain packing. This is one
solid path; swap in RFdiffusion, BindCraft, or Mosaic depending on the target.

```
Target → BoltzGen → Validate → Filter
 (pdb)  (all-atom)   (chai)     (qc)
```

### 1. Target preparation
```bash
# Fetch structure from PDB
# Use pdb skill for guidance
```
- Trim to binding region + 10A buffer
- Remove waters and ligands
- Renumber chains if needed

### 2. Hotspot selection
- Choose 3-6 exposed residues
- Prefer charged/aromatic residues
- Cluster spatially (within 10-15A)

### 3. Design with BoltzGen

First, create a YAML config file (e.g., `binder.yaml`):
```yaml
entities:
  - protein:
      id: B
      sequence: 70..100

  - file:
      path: target.cif
      include:
        - chain:
            id: A
      binding_types:
        - chain:
            id: A
            binding: 45,67,89
```

Then run:
```bash
modal run modal_boltzgen.py \
  --input-yaml binder.yaml \
  --protocol protein-anything \
  --num-designs 50
```

**Why BoltzGen?**
- All-atom output (no separate ProteinMPNN step needed)
- Better for ligand/small molecule binding
- Single-step design (backbone + sequence + side chains)

### 4. Alternative: RFdiffusion Pipeline
For maximum diversity or when backbone-only is preferred:
```bash
# Step 1: Backbone generation
modal run modal_rfdiffusion.py \
  --pdb target.pdb \
  --contigs "A1-150/0 70-100" \
  --hotspot "A45,A67,A89" \
  --num-designs 500

# Step 2: Sequence design
modal run modal_ligandmpnn.py \
  --pdb-path backbone.pdb \
  --num-seq-per-target 16 \
  --sampling-temp 0.1
```

### 5. Validation
```bash
modal run modal_chai1.py \
  --input-faa sequences.fasta \
  --out-dir predictions/
```

### 6. Filtering
Apply standard thresholds:
- pLDDT > 0.80
- ipTM > 0.50
- PAE_interface < 10
- scRMSD < 2.0 A

See protein-qc skill for details.

## Number of designs

| Stage | Count | Purpose |
|-------|-------|---------|
| Backbone generation | 500-1000 | Diversity |
| Sequences per backbone | 8-16 | Sequence space |
| AF2 predictions | All | Validation |
| After filtering | 50-200 | Candidates |
| Experimental testing | 10-50 | Final selection |

## Common mistakes

### Wrong hotspots
- Using buried residues
- Too many hotspots (over-constrain)
- Wrong chain/residue numbers

### Insufficient diversity
- Too few designs generated
- Low temperature in ProteinMPNN
- Not exploring multiple backbones

### Poor target preparation
- Including full protein instead of binding region
- Missing important structural features
- Wrong protonation states

## Timeline guide

| Step | Compute Time |
|------|--------------|
| RFdiffusion (500 designs) | 2-4 hours |
| ProteinMPNN (8000 sequences) | 1-2 hours |
| AF2 prediction (8000 sequences) | 12-24 hours |
| Filtering and analysis | 1-2 hours |

Total: 1-2 days of compute
