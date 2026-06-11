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

## Which tool wins

No single tool is best for every target. Hit-rate is strongly target-dependent, so
choose by target type, what you want to control, and available compute.

The clearest signal comes from head-to-head competitions where many methods design
against the same target. On the Adaptyv Nipah de novo target, the public results show:

| Method | Tested | Binders | Hit-rate |
|--------|--------|---------|----------|
| Mosaic (gradient, multi-model) | 9 | 8 | 89% |
| ProteinMPNN hybrid | 28 | 7 | 25% |
| RFdiffusion | 60 | 13 | 22% |
| BindCraft | 98 | 7 | 7% |
| BoltzGen | 182 | 6 | 3% |

Mosaic had the highest hit-rate, but on a small, expert-tuned sample, and the ranking
shifts on other targets. Treat this as evidence that a well-designed gradient objective
can win on hard targets, not as a fixed leaderboard. BoltzGen remains a strong, cheap,
turnkey default; Mosaic is the high-ceiling option when you can invest tuning time.

```
De novo binder design?
│
├─ Highest hit-rate, willing to tune → Mosaic (gradient, multi-model objective)
├─ Cheap, turnkey, all-atom default → BoltzGen
├─ Ligand / small-molecule binding → BoltzGen (all-atom)
├─ Diversity / exploration → RFdiffusion + ProteinMPNN
├─ End-to-end with built-in validation → BindCraft
└─ Antibody / nanobody (VHH) → germinal skill (also mber, iggm in biomodals)
```

## Tool comparison

| Tool | Strengths | Weaknesses | Best for |
|------|-----------|------------|----------|
| Mosaic | Highest competition hit-rate, composable objective | Needs tuning, local JAX only | Hard targets, expert use |
| BoltzGen | All-atom, single-step, cheap, turnkey | Lower hit-rate on hard targets | Budget default, ligand binding |
| BindCraft | End-to-end, built-in AF2 validation | Less diverse | Production campaigns |
| RFdiffusion | High diversity | Requires ProteinMPNN; not in biomodals | Exploration, diversity |
| Germinal | Antibody and nanobody formats | Finicky | scFv / VHH design |

## Compute vs hit-rate tradeoff

- **Budget / quick exploration**: BoltzGen is cheap (about $8 for 50 designs) and needs
  no tuning. Good first pass and good for ligand binding.
- **Best hit-rate on a hard target**: Mosaic, given time to design and tune the
  objective. It runs locally on a JAX GPU rather than through biomodals.
- Whatever the generator, validate with `boltz` or `chai` and rank with `ipsae`.

Other biomodals-backed options: `modal_rso.py` (Rejection Sampling Optimization, an
AlphaFold-based gradient method) for minibinders, and `modal_mber.py` for VHH
nanobodies.

## Example pipeline: BoltzGen → Chai → QC

BoltzGen provides all-atom design with built-in side-chain packing. This is one cheap,
turnkey path; swap in Mosaic, RFdiffusion, or BindCraft depending on the target.

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
# Step 1: Backbone generation (RFdiffusion, run from the official repo)
python run_inference.py \
  inference.input_pdb=target.pdb \
  contigmap.contigs=[A1-150/0 70-100] \
  ppi.hotspot_res=[A45,A67,A89] \
  inference.num_designs=500

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
