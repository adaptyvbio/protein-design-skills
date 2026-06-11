---
name: protein-qc
description: >
  Quality control metrics and filtering thresholds for protein design.
  Use this skill when: (1) Evaluating design quality for binding, expression, or structure,
  (2) Setting filtering thresholds for pLDDT, ipTM, PAE,
  (3) Checking sequence liabilities (cysteines, deamidation, polybasic clusters),
  (4) Creating multi-stage filtering pipelines,
  (5) Computing PyRosetta interface metrics (dG, SC, dSASA),
  (6) Checking biophysical properties (instability, GRAVY, pI),
  (7) Ranking designs with composite scoring.

  This skill provides research-backed thresholds from binder design
  competitions and published benchmarks.
license: MIT
category: evaluation
tags: [qc, filtering, metrics, thresholds]
---

# Protein Design Quality Control

## Critical Limitation

**Individual metrics have weak predictive power for binding**. Research shows:
- Individual metric ROC AUC: 0.64-0.66 (slightly better than random)
- Metrics are **pre-screening filters**, not affinity predictors
- **Composite scoring is essential** for meaningful ranking

These thresholds filter out poor designs but do NOT predict binding affinity.

## QC Organization

QC is organized by **purpose** and **level**:

| Purpose | What it assesses | Key metrics |
|---------|------------------|-------------|
| **Binding** | Interface quality, binding geometry | ipTM, PAE, SC, dG, dSASA |
| **Expression** | Manufacturability, solubility | Instability, GRAVY, pI, cysteines |
| **Structural** | Fold confidence, consistency | pLDDT, pTM, scRMSD |

Each category has two levels:
- **Metric-level**: Calculated values with thresholds (pLDDT > 0.85)
- **Design-level**: Pattern/motif detection (odd cysteines, NG sites)

---

## Quick Reference: All Thresholds

| Category | Metric | Standard | Stringent | Source |
|----------|--------|----------|-----------|--------|
| **Structural** | pLDDT | > 0.85 | > 0.90 | AF2/Chai/Boltz |
| | pTM | > 0.70 | > 0.80 | AF2/Chai/Boltz |
| | scRMSD | < 2.0 Å | < 1.5 Å | Design vs pred |
| **Binding** | ipSAE_min | > 0.61 | > 0.70 | AF3/Boltz (see ipsae) |
| | ipTM | > 0.50 | > 0.60 | AF2/Chai/Boltz |
| | PAE_interaction | < 12 Å | < 10 Å | AF2/Chai/Boltz |
| | Shape Comp (SC) | > 0.50 | > 0.62 | PyRosetta |
| | interface_dG | < -10 | < -15 | PyRosetta |
| | Interface BUNS | <= 4 | <= 2 | PyRosetta |
| **Expression** | Instability | < 40 | < 30 | BioPython |
| | GRAVY | < 0.4 | < 0.2 | BioPython |
| | ESM2 PLL | > 0.0 | > 0.2 | ESM2 |
| | Folding ΔG | < -2 kcal/mol | < -4 kcal/mol | SaProtΔG |

### Design-Level Checks (Expression)
| Pattern | Risk | Action |
|---------|------|--------|
| Odd cysteine count | Unpaired disulfides | Redesign |
| NG/NS/NT motifs | Deamidation | Flag/avoid |
| K/R >= 3 consecutive | Proteolysis | Flag |
| >= 6 hydrophobic run | Aggregation | Redesign |

See: references/binding-qc.md, references/expression-qc.md, references/structural-qc.md

---

## Interface metrics (PyRosetta)

Beyond shape complementarity and interface_dG, two interface metrics catch common
de novo failure modes:

- **Buried unsatisfied H-bonds (BUNS)**: buried polar atoms making no hydrogen bond.
  This is a dominant energetic failure mode and is orthogonal to dG and dSASA. Keep
  interface BUNS at or below 4 (standard) or 2 (stringent).
- **ContactMolecularSurface**: shape-complementarity-weighted contact area that, unlike
  dSASA, is not fooled by gappy or holey interfaces. Higher is better.

Both are in the Cao 2022, AlphaProteo, and BindCraft filter sets.

## Binder ranking (benchmark-backed)

A meta-analysis of 3,766 experimentally tested binders across 15 targets (Overath et
al., bioRxiv 2025, doi:10.1101/2025.08.14.670059) found that AF3 `ipSAE_min` is the
single best in-silico predictor of binding, and that a simple linear model of three
features generalizes best across targets. Complexity did not help: gradient-boosted
and many-feature models did not beat the linear one.

Recommended filtering strategies from that work:

1. Threshold on one of: `AF3 ipSAE_min > 0.61`, or `ipSAE_min × interface_ΔG/ΔSASA <
   -1.5`, or `LIS × shape_complementarity > 0.42`.
2. Pre-filter on `shape_complementarity > 0.62` and `RMSD_binder < 3.73` (input vs
   re-predicted), then take the top-K by `ipSAE_min`.

`ipSAE_min` is the minimum of the two asymmetric ipSAE directions (binder→target and
target→binder), not the average or max. Use the `ipsae` skill to compute it. Note the
RMSD_binder filter can be over-restrictive on some targets, so prefer it as a soft
pre-filter rather than a hard cutoff.

## Stability prediction (small domains)

For small domains (roughly 60 to 100 residues, the minibinder range), absolute folding
stability can be predicted directly. SaProtΔG (Cho et al., bioRxiv 2026,
doi:10.64898/2026.05.19.726285) predicts absolute folding ΔG at about 0.8 kcal/mol RMSE
and improves discrimination of stable versus unstable designed proteins. Use the SaProt
variant rather than the ESM3 variant for commercial work, since the ESM3 weights are
non-commercial. Filter for more negative (more stable) ΔG.

## Sequence-liability scan

Implement liability checks directly as motif rules rather than taking an
antibody-specific dependency. Severity rises with solvent exposure (gate by SASA when a
structure is available).

```python
import re

LIABILITIES = {
    "deamidation":   (r"N[GSNTH]",   "NG/NS high, NN/NT moderate"),
    "isomerization": (r"D[GSTDH]",   "Asp isomerization"),
    "N-glycosylation": (r"N[^P][ST]", "NxS/T sequon"),
    "polybasic":     (r"[KR]{3,}",   "proteolysis / charge patch"),
    "hydrophobic_run": (r"[AILMFWVY]{6,}", "aggregation"),
}

def scan_liabilities(seq):
    hits = {}
    for name, (pattern, note) in LIABILITIES.items():
        positions = [m.start() for m in re.finditer(pattern, seq)]
        if positions:
            hits[name] = (positions, note)
    # Unpaired cysteine check
    if seq.count("C") % 2 == 1:
        hits["unpaired_cysteine"] = ([seq.index("C")], "odd cysteine count")
    return hits
```

Met and Trp oxidation are also liabilities but should be flagged only when the residue
is solvent-exposed.

---

## Sequential Filtering Pipeline

```python
import pandas as pd

designs = pd.read_csv('designs.csv')

# Stage 1: Structural confidence
designs = designs[designs['pLDDT'] > 0.85]

# Stage 2: Self-consistency
designs = designs[designs['scRMSD'] < 2.0]

# Stage 3: Binding quality
designs = designs[(designs['ipTM'] > 0.5) & (designs['PAE_interaction'] < 10)]

# Stage 4: Sequence plausibility
designs = designs[designs['esm2_pll_normalized'] > 0.0]

# Stage 5: Expression checks (design-level)
designs = designs[designs['cysteine_count'] % 2 == 0]  # Even cysteines
designs = designs[designs['instability_index'] < 40]
```

---

## Composite Scoring (Required for Ranking)

Individual metrics alone are too weak. Use composite scoring:

```python
def composite_score(row):
    return (
        0.30 * row['pLDDT'] +
        0.20 * row['ipTM'] +
        0.20 * (1 - row['PAE_interaction'] / 20) +
        0.15 * row['shape_complementarity'] +
        0.15 * row['esm2_pll_normalized']
    )

designs['score'] = designs.apply(composite_score, axis=1)
top_designs = designs.nlargest(100, 'score')
```

For advanced composite scoring, see references/composite-scoring.md.

---

## Tool-Specific Filtering

### BindCraft Filter Levels
| Level | Use Case | Stringency |
|-------|----------|------------|
| Default | Standard design | Most stringent |
| Relaxed | Need more designs | Higher failure rate |
| Peptide | Designs < 30 AA | ~5-10x lower success |

### BoltzGen Filtering
```bash
boltzgen run ... \
  --budget 60 \
  --alpha 0.01 \
  --filter_biased true \
  --refolding_rmsd_threshold 2.0 \
  --additional_filters 'ALA_fraction<0.3'
```

- `alpha=0.0`: Quality-only ranking
- `alpha=0.01`: Default (slight diversity)
- `alpha=1.0`: Diversity-only

---

## Design-Level Severity Scoring

For pattern-based checks, use severity scoring:

| Severity Level | Score | Action |
|----------------|-------|--------|
| LOW | 0-15 | Proceed |
| MODERATE | 16-35 | Review flagged issues |
| HIGH | 36-60 | Redesign recommended |
| CRITICAL | 61+ | Redesign required |

---

## Experimental Correlation

| Metric | AUC | Use |
|--------|-----|-----|
| ipTM | ~0.64 | Pre-screening |
| PAE | ~0.65 | Pre-screening |
| ESM2 PLL | ~0.72 | Best single metric |
| Composite | ~0.75+ | **Always use** |

**Key insight**: Metrics work as **filters** (eliminating failures) not **predictors** (ranking successes).

---

## Campaign Health Assessment

Quick assessment of your design campaign:

| Pass Rate | Status | Interpretation |
|-----------|--------|----------------|
| > 15% | Excellent | Above average, proceed |
| 10-15% | Good | Normal, proceed |
| 5-10% | Marginal | Below average, review issues |
| < 5% | Poor | Significant problems, diagnose |

---

## Failure Recovery Trees

### Too Few Pass pLDDT Filter (< 5% with pLDDT > 0.85)

```
Low pLDDT across campaign
├── Check scRMSD distribution
│   ├── High scRMSD (>2.5Å): Backbone issue
│   │   └── Fix: Regenerate backbones with lower noise_scale (0.5-0.8)
│   └── Low scRMSD but low pLDDT: Disordered regions
│       └── Fix: Check design length, simplify topology
├── Try more sequences per backbone
│   └── modal run modal_proteinmpnn.py --num-seq-per-target 32 --sampling-temp 0.1
├── Use SolubleMPNN instead of ProteinMPNN
│   └── Better for expression-optimized sequences
└── Consider different design tool
    └── BindCraft (integrated design) may work better
```

### Too Few Pass ipTM Filter (< 5% with ipTM > 0.5)

```
Low ipTM across campaign
├── Review hotspot selection
│   ├── Are hotspots surface-exposed? (SASA > 20Å²)
│   ├── Are hotspots conserved? (check MSA)
│   └── Try 3-6 different hotspot combinations
├── Increase binder length (more contact area)
│   └── Try 80-100 AA instead of 60-80 AA
├── Check interface geometry
│   ├── Is target flat? → Try helical binders
│   └── Is target concave? → Try smaller binders
└── Try all-atom design tool
    └── BoltzGen (all-atom, better packing)
```

### High scRMSD (> 50% with scRMSD > 2.0Å)

```
Sequences don't specify intended structure
├── ProteinMPNN issue
│   ├── Lower temperature: --sampling-temp 0.1
│   ├── Increase sequences: --num-seq-per-target 32
│   └── Check fixed_positions aren't over-constraining
├── Backbone geometry issue
│   ├── Backbones may be unusual/strained
│   ├── Regenerate with lower noise_scale (0.5-0.8)
│   └── Reduce diffuser.T to 30-40
└── Try different sequence design
    └── ColabDesign (AF2 gradient-based) may work better
```

### Everything Passes But No Experimental Hits

```
In silico metrics don't predict affinity
├── Generate MORE designs (10x current)
│   └── Computational metrics have high false positive rate
├── Increase diversity
│   ├── Higher ProteinMPNN temperature (0.2-0.3)
│   ├── Different backbone topologies
│   └── Different hotspot combinations
├── Try different design approach
│   ├── BindCraft (different algorithm)
│   ├── ColabDesign (AF2 hallucination)
│   └── BoltzGen (all-atom diffusion)
└── Check if target is druggable
    └── Some targets are inherently difficult
```

### Too Many Designs Pass (> 50%)

```
Suspiciously high pass rate
├── Check if thresholds are too lenient
│   └── Use stringent thresholds: pLDDT > 0.90, ipTM > 0.60
├── Verify prediction quality
│   ├── Are predictions actually running? Check output files
│   └── Are complexes being predicted, not just monomers?
├── Check for data issues
│   ├── Same sequence being predicted multiple times?
│   └── Wrong FASTA format (missing chain separator)?
└── Apply diversity filter
    └── Cluster at 70% identity, take top per cluster
```

---

## Diagnostic Commands

### Quick Campaign Assessment

```python
import pandas as pd

df = pd.read_csv('designs.csv')

# Pass rates at each stage
print(f"Total designs: {len(df)}")
print(f"pLDDT > 0.85: {(df['pLDDT'] > 0.85).mean():.1%}")
print(f"ipTM > 0.50: {(df['ipTM'] > 0.50).mean():.1%}")
print(f"scRMSD < 2.0: {(df['scRMSD'] < 2.0).mean():.1%}")
print(f"All filters: {((df['pLDDT'] > 0.85) & (df['ipTM'] > 0.5) & (df['scRMSD'] < 2.0)).mean():.1%}")

# Identify top issue
if (df['pLDDT'] > 0.85).mean() < 0.1:
    print("ISSUE: Low pLDDT - check backbone or sequence quality")
elif (df['ipTM'] > 0.50).mean() < 0.1:
    print("ISSUE: Low ipTM - check hotspots or interface geometry")
elif (df['scRMSD'] < 2.0).mean() < 0.5:
    print("ISSUE: High scRMSD - sequences don't specify backbone")
```

---

