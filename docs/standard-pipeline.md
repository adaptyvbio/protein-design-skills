# Standard pipeline

End-to-end protein binder design workflow.

## Recommended: BoltzGen pipeline

```
Target → BoltzGen → Validate → Filter → Experiment
 (pdb)  (all-atom)   (chai)     (qc)    (cfps/spr)
```

BoltzGen provides all-atom design with built-in side-chain packing - no separate sequence design step needed.

### Why BoltzGen?
- **All-atom output**: Backbone + sequence + side chains in one step
- **Better for ligand binding**: All-atom diffusion handles small molecules
- **Single step**: No need for separate ProteinMPNN

---

## Phase 1: Target preparation

**Skill**: `pdb`

```bash
# Fetch structure
curl -o target.pdb "https://files.rcsb.org/download/1ALU.pdb"

# Extract chain (using Python)
python -c "
from Bio.PDB import PDBParser, PDBIO, Select
class ChainSelect(Select):
    def accept_chain(self, chain):
        return chain.id == 'A'
parser = PDBParser()
structure = parser.get_structure('target', 'target.pdb')
io = PDBIO()
io.set_structure(structure)
io.save('target_chainA.pdb', ChainSelect())
"
```

**Checklist**:
- [ ] Correct chain extracted
- [ ] Waters/ligands removed (unless needed for context)
- [ ] Residue numbering verified
- [ ] 3-6 hotspots identified (surface-exposed)

## Phase 2: Design with BoltzGen (recommended)

**Skill**: `boltzgen`

```bash
uvx modal run modal_boltzgen.py \
  --target target_chainA.pdb \
  --hotspot-res "A42,A58,A62" \
  --num-designs 50
```

**Expected**: 50 all-atom designs, ~30-60 min on L40S

**Output**: CIF files with complete structures (backbone + sequence + side chains)

### Alternative: RFdiffusion pipeline

For maximum diversity or backbone-only control:

**Step 1: Backbone Generation** (`rfdiffusion`)
```bash
uvx modal run modal_rfdiffusion.py \
  --input-pdb target_chainA.pdb \
  --contigs "A1-150/0 70-90" \
  --hotspot-res "A42,A58,A62" \
  --num-designs 100
```

**Step 2: Sequence Design** (`ligandmpnn`)
```bash
uvx modal run modal_ligandmpnn.py \
  --pdb-path output/output_0.pdb \
  --num-seq-per-target 8 \
  --sampling-temp 0.1
```

**Expected**: 100 backbones × 8 sequences = 800 sequences

## Phase 3: Structure validation

**Skill**: `chai` or `boltz`

```bash
# Validate with Chai
uvx modal run modal_chai1.py \
  --fasta-path all_designs.fasta \
  --output-dir predictions/
```

**Expected**: Predictions with pLDDT, pTM, ipTM scores

## Phase 4: Filtering

**Skill**: `protein-qc`

```python
import json
import glob

passing = []
for f in glob.glob("predictions/*/scores.json"):
    scores = json.load(open(f))
    if scores.get("iptm", 0) > 0.5 and scores.get("plddt", 0) > 0.85:
        passing.append(f)

print(f"Passing: {len(passing)}")  # Expect 10-15% pass rate
```

### QC thresholds

| Metric | Standard | Stringent |
|--------|----------|-----------|
| pLDDT | > 0.85 | > 0.90 |
| ipTM | > 0.50 | > 0.70 |
| PAE interface | < 10 Å | < 5 Å |
| scRMSD | < 2.0 Å | < 1.5 Å |

## Timeline comparison

### BoltzGen pipeline (recommended)
| Phase | Tool | Time | Output |
|-------|------|------|--------|
| Target prep | pdb | 10 min | 1 PDB |
| Design | boltzgen | 30-60 min | 50 designs |
| Validation | chai | 30-60 min | 50 predictions |
| Filtering | protein-qc | 10 min | 5-10 candidates |
| **Total** | | **~2 hours** | |

### RFdiffusion pipeline (alternative)
| Phase | Tool | Time | Output |
|-------|------|------|--------|
| Target prep | pdb | 10 min | 1 PDB |
| Backbone | rfdiffusion | 30 min | 100 backbones |
| Sequence | ligandmpnn | 20 min | 800 sequences |
| Validation | chai | 4-8 hr | 800 predictions |
| Filtering | protein-qc | 10 min | 80-120 candidates |
| **Total** | | **5-9 hours** | |

## Common issues

| Issue | Cause | Fix |
|-------|-------|-----|
| Low ipTM | Poor hotspot selection | Try different hotspots |
| All random coil | High noise | Reduce noise_scale to 0.5-0.8 |
| Low diversity | Low temperature | Increase to 0.2-0.3 |
| No designs pass | Wrong thresholds | Check pLDDT scale (0-1 vs 0-100) |

## Next steps

After filtering:
1. Rank by composite score (ipTM + pLDDT) or ipSAE
2. Visual inspection in PyMOL
3. Select top 10-50 for experimental testing
4. Use `cell-free-expression` for CFPS optimization
5. Use `binding-characterization` for SPR/BLI

## See also

- [Skills](skills.md) - All 21 skills
- [Getting started](getting-started.md) - Setup guide
- [Compute setup](compute-setup.md) - Modal vs local setup
