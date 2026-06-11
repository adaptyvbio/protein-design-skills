# Getting Started

Step-by-step setup guide for Protein Design Skills.

## Prerequisites

- Claude Code installed (`claude --version`)
- Python 3.10+ (`python --version`)

## Step 1: Install skills

In Claude Code, run:

```
/plugin marketplace add adaptyvbio/protein-design-skills
/plugin install adaptyv@protein-design-skills
```

This installs all 24 skills.

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

Verify: You should see files like `modal_boltzgen.py`, `modal_chai1.py`, `modal_esmfold2.py`

## Step 4: Test your setup

Let's verify everything works:

```bash
cd biomodals
uvx modal run modal_boltzgen.py --help
```

Expected output (key options):
```
modal_boltzgen.py
  --input-yaml TEXT     Design specification YAML (required)
  --protocol TEXT       protein-anything (default), peptide-anything, ...
  --num-designs INTEGER Number of designs to generate
```

## Step 5: Run your first design

### Download a test target
```bash
curl -o target.pdb https://files.rcsb.org/download/1ALU.pdb
```

### Run BoltzGen
BoltzGen takes a YAML design spec (see the `boltzgen` skill for the format):
```bash
uvx modal run modal_boltzgen.py --input-yaml binder.yaml --protocol protein-anything --num-designs 5
```

## Quick reference: common commands

### Design
```bash
# BoltzGen (all-atom, single-step); needs a YAML design spec
uvx modal run modal_boltzgen.py --input-yaml binder.yaml --protocol protein-anything --num-designs 50

# RFdiffusion (backbone generation; official repo, not biomodals)
python run_inference.py inference.input_pdb=target.pdb 'contigmap.contigs=[A1-150/0 70-100]' inference.num_designs=100

# BindCraft (end-to-end)
uvx modal run modal_bindcraft.py --input-pdb target.pdb --lengths "70,100" --number-of-final-designs 50
```

### Sequence design
```bash
# LigandMPNN (also runs ProteinMPNN weights); extra args go in --params-str
uvx modal run modal_ligandmpnn.py --input-pdb backbone.pdb --params-str "--number_of_batches 8 --temperature 0.1"
```

### Validation
```bash
# Chai (fast, no MSA)
uvx modal run modal_chai1.py --input-faa designs.fasta --out-dir predictions/

# Boltz (open-source)
uvx modal run modal_boltz.py --input-faa designs.fasta --out-dir predictions/
```

## GPU selection

Set GPU with environment variable:
```bash
GPU=L40S uvx modal run modal_boltzgen.py ...
GPU=A100 uvx modal run modal_chai1.py ...
MODAL_GPU=A100 uvx modal run modal_esmfold2.py ...
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

- [Skills](skills.md) - All 24 skills
- [Standard pipeline](standard-pipeline.md) - Full workflow details
- [Compute setup](compute-setup.md) - Modal vs local setup
