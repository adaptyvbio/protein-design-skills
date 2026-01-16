# Compute Setup

## Where to run?

| Provider | Best For | Setup Time | Cost |
|----------|----------|------------|------|
| **Modal** (Recommended) | Quick runs, exploration, no GPU ownership | 5 min | Pay-per-use |
| **Local** | Large campaigns, full control, development | 30+ min | GPU ownership |

## First-time setup

### Option 1: Modal (recommended)

Modal provides serverless GPU access - no hardware required.

**Step 1: Create Account**
1. Go to https://modal.com
2. Sign up (free tier available with $30/month credits)
3. Verify your email

**Step 2: Install CLI**
```bash
pip install modal
```

**Step 3: Authenticate**
```bash
modal setup
```
This opens a browser. Click "Authorize".

**Step 4: Clone Biomodals**
```bash
git clone https://github.com/hgbrian/biomodals
cd biomodals
```

**Step 5: Verify**
```bash
uvx modal run modal_boltzgen.py --help
```

### Option 2: Local installation

For local GPU execution, each tool requires specific setup. See individual skill documentation.

**General Requirements:**
- NVIDIA GPU with CUDA 11.7+
- Python 3.10+
- 16-48GB GPU VRAM (tool-dependent)

## GPU selection guide

| GPU | VRAM | Best For | Modal Cost/hr |
|-----|------|----------|---------------|
| T4 | 16GB | ProteinMPNN, ESM | ~$0.50 |
| A10G | 24GB | RFdiffusion, Chai | ~$1.10 |
| L40S | 48GB | BoltzGen, BindCraft | ~$1.80 |
| A100 | 40-80GB | Large complexes, Chai batch | ~$3.50 |

Set GPU in Modal:
```bash
GPU=A10G uvx modal run modal_rfdiffusion.py ...
GPU=L40S uvx modal run modal_boltzgen.py ...
```

## Tool-specific requirements

| Tool | Minimum GPU | Recommended | biomodals Script |
|------|-------------|-------------|------------------|
| BoltzGen | L40S (48GB) | A100 | modal_boltzgen.py |
| BindCraft | L40S (48GB) | L40S | modal_bindcraft.py |
| RFdiffusion | A10G (24GB) | A10G | modal_rfdiffusion.py |
| ProteinMPNN | T4 (16GB) | T4 | modal_ligandmpnn.py |
| LigandMPNN | T4 (16GB) | T4 | modal_ligandmpnn.py |
| Chai | A10G (24GB) | A100 | modal_chai1.py |
| Boltz | L40S (48GB) | L40S | modal_boltz.py |
| AlphaFold | A100 (40GB) | A100 | modal_alphafold.py |
| ESM | A10G (24GB) | A10G | modal_esm2_predict_masked.py |

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
First run downloads model weights. Subsequent runs are faster.

---

## See also

- [Skills](skills.md) - All 21 skills
- [Getting started](getting-started.md) - Setup guide
- [Standard pipeline](standard-pipeline.md) - Full workflow
