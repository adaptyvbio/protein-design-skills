---
name: esm
description: >
  ESM protein language models for embeddings, sequence scoring, structure
  prediction, and binder design. Use this skill when: (1) Computing
  pseudo-log-likelihood (PLL) or mutation-effect scores, (2) Getting protein
  embeddings for clustering or filtering, (3) Predicting complex structures with
  ESMFold2, (4) Designing binders by inverting ESMFold2, (5) Filtering designs by
  sequence plausibility.

  For diffusion-based structure prediction, use boltz or chai.
  For QC thresholds, use protein-qc.
  For gradient-based multi-objective design, use mosaic.
license: MIT
category: design-tools
tags: [sequence-design, embeddings, scoring, structure-prediction, binder]
proteinbase_slug: esm2-optimization
proteinbase_url: https://proteinbase.com/design-methods/esm2-optimization
biomodals_script: modal_esm2_predict_masked.py
---

# ESM Protein Language Models

The ESM line is maintained at [github.com/Biohub/esm](https://github.com/Biohub/esm)
(Chan Zuckerberg Biohub, MIT license; the older `evolutionaryscale/esm` URL
redirects here). The current generation ships three artifacts: **ESM C** (language
model), **ESMFold2** (structure prediction), and **ESM Atlas** (a map of predicted
structures). Weights are on [huggingface.co/biohub](https://huggingface.co/biohub);
the hosted API is at `biohub.ai`.

This skill covers ESM C, ESMFold2, and legacy ESM2. ESM3 is not covered because its
open weights are non-commercial.

## Which model to use

| Task | Model |
|------|-------|
| Embeddings, PLL, mutation scoring | ESM C (ESMC-6B), or ESM2 for a lighter run |
| Complex structure prediction | ESMFold2 |
| High-throughput single-sequence folding | ESMFold2 fast mode |
| Binder design | ESMFold2 inversion (see below), or the `mosaic` / `bindcraft` skills |
| Variant effect / zero-shot scoring | ESM C or ESM2 |

## Prerequisites

| Requirement | Minimum | Recommended |
|-------------|---------|-------------|
| Python | 3.10+ | 3.11 |
| PyTorch | 2.0+ | Latest |
| CUDA | 12.0+ | 12.1+ |
| GPU VRAM | 24GB (ESM2 / small ESMC) | 80GB (ESMC-6B, ESMFold2) |

## ESM C: embeddings and scoring

ESM C is the successor to ESM2. It improves long-range structural understanding as
model scale grows and is the default choice for embeddings, pseudo-log-likelihood,
and mutation-effect scoring.

### Python (Hugging Face)
```python
from transformers import AutoModelForMaskedLM, AutoTokenizer
import torch

model_id = "biohub/ESMC-6B"
tok = AutoTokenizer.from_pretrained(model_id)
model = AutoModelForMaskedLM.from_pretrained(
    model_id, output_hidden_states=True, torch_dtype=torch.bfloat16
).eval().cuda()

batch = tok(["MKTAYIAKQRQISFVK..."], return_tensors="pt").to("cuda")
with torch.no_grad():
    out = model(**batch)

logits = out.logits                      # for PLL / mutation scoring
embeddings = out.hidden_states[-1]       # per-residue representations
```

Install the package with `pip install esm@git+https://github.com/Biohub/esm.git@main`.

### Hosted API
```python
from esm.sdk import esmc_client
from esm.sdk.api import ESMProtein, LogitsConfig

model = esmc_client(model="esmc-600m-2024-12", url="https://biohub.ai", token="<API token>")
tensor = model.encode(ESMProtein(sequence="MKTAYIAKQRQISFVK..."))
out = model.logits(tensor, LogitsConfig(sequence=True, return_embeddings=True))
```

ESMC-6B has open weights; `esmc-600m` is the smaller API model. For mutation
scoring and fine-tuning, see the `esmc_mutation_scoring` and `esmc_finetune`
notebooks under [cookbook/tutorials](https://github.com/Biohub/esm/tree/main/cookbook/tutorials).

## ESMFold2: complex structure prediction

ESMFold2 is built on ESMC-6B with a diffusion structure head. Unlike the original
ESMFold, it predicts complexes (protein, DNA, ligand, and modified residues), takes
an optional MSA, and has a single-sequence fast mode for high-throughput screening.
It is validated for protein-protein interaction design and leads DockQ pass-rate on
Foldbench protein-protein and antibody-antigen complexes.

### Modal (biomodals)
```bash
printf '>protein|A\nMKTAYIAKQRQISFVK...\n' > target.faa
uv run --with modal modal run modal_esmfold2.py --input-faa target.faa
```

The FASTA header tags `protein|`, `dna|`, `rna|`, and `ligand|` (SMILES) let you fold
complexes. GPU defaults to A100-40GB (set with `MODAL_GPU`).

### Python (local weights)
```python
from transformers.models.esmfold2.modeling_esmfold2 import ESMFold2Model
from esm.models.esmfold2 import ProteinInput, StructurePredictionInput, ESMFold2InputBuilder

model = ESMFold2Model.from_pretrained("biohub/ESMFold2").cuda().eval()
spi = StructurePredictionInput(sequences=[ProteinInput(id="A", sequence="BINDER_SEQ")])
result = ESMFold2InputBuilder().fold(model, spi, num_loops=20, num_sampling_steps=100)
# result.plddt, result.ptm, result.iptm, result.complex.to_mmcif()
```

For single-sequence high-throughput folding, the fast variant is the SDK model string
`esmfold2-fast-2026-05` (HF repo `biohub/ESMFold2-Fast`). ESMFold2 is one option for
complex validation alongside `boltz` and `chai`; ranking a shortlist across more than
one predictor is more reliable than trusting a single model.

## Binder design by inverting ESMFold2

The [binder_design cookbook](https://github.com/Biohub/esm/blob/main/cookbook/tutorials/binder_design.ipynb)
runs gradient optimization through ESMFold2 (a BindCraft-style loop) with an ESMC
language-model term for sequence plausibility. The published protocol is validated in
the lab to nanomolar affinity across five targets and supports both minibinders and
antibody-derived scFvs with framework scaffolds.

biomodals wraps this as `modal_esmfold2_binder_design.py`:

```bash
uv run --with modal modal run modal_esmfold2_binder_design.py \
  --target-name pd-l1 --binder-name minibinder
```

- Targets: presets `cd45, ctla4, egfr, pd-l1, pdgfr`, or pass `--target-sequence`.
- Binders: presets `minibinder` and antibody frameworks (for example
  `trastuzumab_framework_vhvl`), or pass `--binder-sequence` with `#` for designable
  positions. Use `--is-antibody` for scFv designs.
- Rank candidates by ipTM, filter minibinders to pI below 6, then validate the top
  shortlist with `boltz` or `chai` and rank with `ipsae`.

For a framework that composes ESMFold2 with other predictors in one objective, use the
`mosaic` skill.

## ESM2 (legacy)

ESM2 still works well for quick embeddings and PLL when ESMC-6B is too large for the
available GPU.

```python
import torch, esm
model, alphabet = esm.pretrained.esm2_t33_650M_UR50D()
bc = alphabet.get_batch_converter()
model = model.eval().cuda()
_, _, toks = bc([("seq1", "MKTAYIAKQRQISFVK...")])
with torch.no_grad():
    rep = model(toks.cuda(), repr_layers=[33])["representations"][33]
```

| Model | Parameters | Use |
|-------|------------|-----|
| esm2_t12_35M | 35M | Fast screening |
| esm2_t33_650M | 650M | Standard embeddings/PLL |
| esm2_t36_3B | 3B | Highest-quality ESM2 |

## PLL interpretation

PLL (pseudo-log-likelihood) scores how natural a sequence looks to the model. Higher
is more natural. Designed sequences often score lower than natural ones, so treat PLL
as a soft filter, not a hard cutoff.

| Normalized PLL | Interpretation |
|----------------|----------------|
| > 0.2 | Very natural |
| 0.0 to 0.2 | Natural-like |
| -0.5 to 0.0 | Acceptable |
| < -0.5 | May be unnatural |

## Troubleshooting

| Issue | Cause | Fix |
|-------|-------|-----|
| CUDA out of memory | ESMC-6B / ESMFold2 too large | Use ESMC-600m API, ESM2, or an 80GB GPU |
| Wrong layer for embeddings | Layer index mismatch | Use the last hidden state (layer 33 for ESM2-650M) |
| Invalid amino acid | Non-standard residue | Check for non-canonical characters |
| Slow ESMFold2 on many designs | Full MSA mode | Use `esmfold2-fast-2026-05` single-sequence mode |

---

**Next**: Validate structures with `boltz` or `chai`, rank with `ipsae`, then filter
with `protein-qc`.
