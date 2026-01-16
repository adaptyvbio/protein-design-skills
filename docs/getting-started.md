# Getting Started

Step-by-step setup guide for Protein Design Skills.

## Prerequisites

- Claude Code installed (`claude --version`)
- Python 3.10+ (`python --version`)

## Step 1: Install skills

In Claude Code, run:

```
/plugin marketplace add adaptyvbio/protein-design-skills
/plugin install design-tools@protein-design-skills
```

**Available plugins:** `design-tools`, `evaluation`, `utilities`, `experimental`, `orchestration`

## Step 2: Set up Modal

Modal provides cloud GPUs so you don't need your own hardware.

### 2.1 Create a Modal account
1. Go to https://modal.com
2. Click "Sign Up" (free tier available with $30/month credits)
3. Verify your email

Verify: You can log in to modal.com

### 2.2 Install Modal CLI
```bash
pip install modal
```

Verify: Run `modal --version` - should show version number

### 2.3 Authenticate
```bash
modal setup
```
This opens a browser. Click "Authorize".

Verify: Run `modal token show` - should show your token

## Step 3: Get biomodals

Biomodals contains the GPU wrappers for all design tools.

```bash
git clone https://github.com/hgbrian/biomodals
cd biomodals
```

Verify: You should see files like `modal_boltzgen.py`, `modal_rfdiffusion.py`

## Step 4: Test your setup

Let's verify everything works:

```bash
cd biomodals
uvx modal run modal_boltzgen.py --help
```

Expected output:
```
Usage: modal_boltzgen.py [OPTIONS]
Options:
  --target TEXT          Target PDB file
  --num-designs INTEGER  Number of designs to generate
  ...
```

## Step 5: Run your first design

### Download a test target
```bash
curl -o target.pdb https://files.rcsb.org/download/1ALU.pdb
```

### Run BoltzGen (Recommended Pipeline)
```bash
uvx modal run modal_boltzgen.py --target target.pdb --num-designs 5
```

Expected output:
```
[INFO] Loading BoltzGen model...
[INFO] Generating 5 designs...
[INFO] Design 1/5 completed
...
[INFO] Saved to output/
```

## Quick reference: common commands

### Design
```bash
# BoltzGen (recommended - all-atom)
uvx modal run modal_boltzgen.py --target target.pdb --num-designs 50

# RFdiffusion (backbone generation)
uvx modal run modal_rfdiffusion.py --input-pdb target.pdb --contigs "A1-150/0 70-100" --num-designs 100

# BindCraft (end-to-end)
uvx modal run modal_bindcraft.py --target-pdb target.pdb --num-designs 50
```

### Sequence design
```bash
# LigandMPNN/ProteinMPNN
uvx modal run modal_ligandmpnn.py --pdb-path backbone.pdb --num-seq-per-target 8
```

### Validation
```bash
# Chai (fast, no MSA)
uvx modal run modal_chai1.py --fasta-path designs.fasta --output-dir predictions/

# Boltz (open-source)
uvx modal run modal_boltz.py --fasta-path designs.fasta --output-dir predictions/
```

## GPU selection

Set GPU with environment variable:
```bash
GPU=A10G uvx modal run modal_rfdiffusion.py ...
GPU=L40S uvx modal run modal_boltzgen.py ...
GPU=A100 uvx modal run modal_chai1.py ...
```

| GPU | VRAM | Best For |
|-----|------|----------|
| T4 | 16GB | ProteinMPNN, ESM |
| A10G | 24GB | RFdiffusion, Chai |
| L40S | 48GB | BoltzGen, BindCraft |
| A100 | 40-80GB | Large complexes |

## Next steps

You're ready! Try these:

1. **Design a binder**: Ask Claude "Design a binder for PDB 1ALU using BoltzGen"
2. **Validate designs**: Ask Claude "Validate my designs with Chai"
3. **Check quality**: Ask Claude "Run QC on my designs"

## Troubleshooting

### "modal: command not found"
```bash
pip install modal
# Restart your terminal
```

### "CUDA out of memory"
- Reduce `--num-designs` to 1-5
- Use a larger GPU: `GPU=A100 uvx modal run ...`

### "Permission denied" on Modal
Run `modal setup` again to re-authenticate.

### Slow first run
First run downloads model weights (~5-10 min). Subsequent runs are faster.

---

**Need help?** Open an issue: https://github.com/adaptyvbio/protein-design-skills/issues

## See also

- [Skills](skills.md) - All 21 skills
- [Standard pipeline](standard-pipeline.md) - Full workflow details
- [Compute setup](compute-setup.md) - Modal vs local setup
