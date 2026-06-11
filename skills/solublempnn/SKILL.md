---
name: solublempnn
description: >
  Solubility-optimized protein sequence design using SolubleMPNN.
  Use this skill when: (1) Designing for E. coli expression,
  (2) Optimizing solubility of designed proteins,
  (3) Reducing aggregation propensity,
  (4) Need high-yield expression,
  (5) Avoiding inclusion body formation.

  For standard design, use proteinmpnn.
  For ligand-aware design, use ligandmpnn.
license: MIT
category: design-tools
tags: [sequence-design, inverse-folding, solubility]
biomodals_script: modal_ligandmpnn.py
---

# SolubleMPNN Solubility-Optimized Design

## Prerequisites

| Requirement | Minimum | Recommended |
|-------------|---------|-------------|
| Python | 3.8+ | 3.10 |
| CUDA | 11.0+ | 11.7+ |
| GPU VRAM | 8GB | 16GB (T4) |
| RAM | 8GB | 16GB |

## How to run

> **First time?** See [Getting started](../../docs/getting-started.md) to set up Modal and biomodals.

### Option 1: Modal (recommended)
SolubleMPNN is the soluble model type within the LigandMPNN wrapper:
```bash
cd biomodals
modal run modal_ligandmpnn.py \
  --input-pdb backbone.pdb \
  --params-str "--model_type soluble_mpnn --number_of_batches 16 --temperature 0.1"
```

**GPU**: A10G default | **Timeout**: 900s default

### Option 2: Local installation
```bash
git clone https://github.com/dauparas/ProteinMPNN.git
cd ProteinMPNN

# The soluble weights are selected with --use_soluble_model, not a model name
python protein_mpnn_run.py \
  --pdb_path backbone.pdb \
  --out_folder output/ \
  --num_seq_per_target 16 \
  --sampling_temp "0.1" \
  --use_soluble_model
```

## Key parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `--pdb_path` | required | Input structure |
| `--use_soluble_model` | off | Use the solubility-trained weights |
| `--num_seq_per_target` | 1 | Sequences per structure |
| `--sampling_temp` | "0.1" | Temperature (string) |
| `--model_name` | v_48_020 | Noise level (0.20 A); orthogonal to solubility |

## Model weights

`--model_name` sets the training-noise level (v_48_002 = 0.02 A, v_48_010 = 0.10 A,
v_48_020 = 0.20 A), not a solubility tier. Solubility is a separate weight set chosen
with `--use_soluble_model`, available for v_48_010 and v_48_020. Higher noise gives
more sequence diversity.

## Output format

```
output/
├── seqs/backbone.fa
└── backbone_pdb/backbone_0001.pdb
```

## Sample output

### Successful run
```
$ python protein_mpnn_run.py --pdb_path backbone.pdb --use_soluble_model --num_seq_per_target 8
Loading soluble model weights (v_48_020)...
Designing sequences for backbone.pdb
Generated 8 sequences in 2.1 seconds

output/seqs/backbone.fa:
>backbone_0001, score=1.31, global_score=1.24, seq_recovery=0.78
MKTAYIAKQRQISFVKSHFSRQLE...
>backbone_0002, score=1.28, global_score=1.21, seq_recovery=0.81
MKTAYIAKQRQISFVKSQFSRQLD...
```

**What good output looks like:**
- Score: 1.0-2.0 (lower = more confident)
- Reduced hydrophobic patches compared to standard MPNN
- Improved charge distribution

## Decision tree

```
Should I use SolubleMPNN?
│
├─ What expression system?
│  ├─ E. coli → SolubleMPNN ✓
│  ├─ Mammalian → ProteinMPNN (PTMs matter more)
│  └─ Yeast → Either
│
├─ History of expression problems?
│  ├─ Yes, aggregation → SolubleMPNN ✓
│  ├─ Yes, low yield → SolubleMPNN ✓
│  └─ No → ProteinMPNN is fine
│
├─ What's in the binding site?
│  ├─ Small molecule / ligand → Use LigandMPNN
│  └─ Nothing / protein only → SolubleMPNN ✓
│
└─ Optimizing for expression?
   └─ Add --use_soluble_model to ProteinMPNN
```

## Typical performance

| Campaign Size | Time (T4) | Cost (Modal) | Notes |
|---------------|-----------|--------------|-------|
| 100 backbones × 8 seq | 15-20 min | ~$2 | Standard |
| 500 backbones × 8 seq | 1-1.5h | ~$8 | Large campaign |

**Expected improvement**: +15-30% solubility score vs standard ProteinMPNN.

---

## Verify

```bash
grep -c "^>" output/seqs/*.fa  # Should match backbone_count × num_seq_per_target
```

---

## Troubleshooting

**Still insoluble**: Confirm `--use_soluble_model` is set; redesign more positions or add explicit hydrophobic-residue bias
**Low diversity**: Increase temperature to 0.2
**Poor folding**: Use standard ProteinMPNN and optimize later

### Error interpretation

| Error | Cause | Fix |
|-------|-------|-----|
| `RuntimeError: CUDA out of memory` | Long protein or large batch | Reduce batch_size |
| `FileNotFoundError: v_48_020` | Missing model weights | Download soluble weights |

---

**Next**: Structure prediction for validation → `protein-qc` for filtering.
