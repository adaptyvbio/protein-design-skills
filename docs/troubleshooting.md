# Troubleshooting

Centralized guide for common issues across all tools.

## Quick diagnosis

```
Campaign failing?
│
├─ No output files?
│  └─ Check: CUDA, paths, permissions, disk space
│
├─ Backbones look bad?
│  ├─ Random coil → Reduce noise_scale to 0.5-0.8
│  ├─ No secondary structure → Check model weights
│  └─ Wrong size → Check contig syntax
│
├─ Low sequence diversity?
│  └─ Increase temperature to 0.2-0.3
│
├─ Low validation scores?
│  ├─ Low pLDDT → Backbone problem, regenerate
│  ├─ Low ipTM → Hotspot problem, try different residues
│  └─ High PAE → Interface not forming
│
└─ Nothing passes QC?
   ├─ Check pLDDT scale (0-1 vs 0-100)
   ├─ Relax thresholds temporarily
   └─ Try different design approach
```

## Error reference

### RFdiffusion

| Error | Cause | Fix |
|-------|-------|-----|
| `CUDA out of memory` | GPU VRAM exceeded | Use A100 or reduce batch |
| `KeyError: 'A'` | Chain not in PDB | Check chain IDs |
| `ValueError: invalid contig` | Syntax error | Remove spaces, quotes, commas |
| `FileNotFoundError: ckpt` | Missing weights | Download from IPD |

### ProteinMPNN

| Error | Cause | Fix |
|-------|-------|-----|
| `CUDA out of memory` | Long protein | Reduce batch_size |
| `KeyError: 'A'` | Wrong chain | Check PDB chain IDs |
| `JSONDecodeError` | Bad JSONL | Validate JSON syntax |

### Chai/Boltz

| Error | Cause | Fix |
|-------|-------|-----|
| `CUDA out of memory` | Complex too large | Use A100-80GB |
| `KeyError: 'iptm'` | Single chain | Ensure 2+ chains in FASTA |
| `invalid SMILES` | Bad ligand | Validate with RDKit |

### ColabFold

| Error | Cause | Fix |
|-------|-------|-----|
| `ConnectionError: MSA` | Server overloaded | Retry or use local MMseqs2 |
| `sequence length` | > 2500 AA | Split into domains |

## Common mistakes

### Contig syntax
```bash
# WRONG
contigmap.contigs=[A1-150 70-100]     # Missing /0
contigmap.contigs="A1-150/0 70-100"   # Quotes break parsing
contigmap.contigs=[A1-150/0, 70-100]  # Comma breaks parsing

# CORRECT
contigmap.contigs=[A1-150/0 70-100]
```

### Hotspot syntax
```bash
# WRONG
ppi.hotspot_res=[45,67,89]           # Missing chain letter
ppi.hotspot_res=[A45, A67, A89]      # Spaces break parsing

# CORRECT
ppi.hotspot_res=[A45,A67,A89]
```

### Temperature format
```bash
# WRONG
--sampling_temp 0.1    # Float without quotes

# CORRECT
--sampling_temp "0.1"  # String with quotes
```

### pLDDT scale
```python
# ColabFold outputs 0-100 scale
plddt_normalized = plddt / 100  # Convert to 0-1

# Chai/Boltz output 0-1 scale directly
```

## Recovery strategies

### Low success rate

1. **Try different hotspots** - Surface-exposed, conserved
2. **Increase design count** - 500-1000 backbones
3. **Try different tool** - BindCraft for higher success
4. **Relax thresholds** - Temporary for debugging

### All designs fail validation

1. **Check target preparation** - Correct chain, no waters
2. **Check hotspot selection** - Not buried
3. **Visual inspection** - Are backbones reasonable?
4. **Try MSA mode** - ColabFold with full MSA

### Expression failures

1. **Check sequence** - Rare codons, hydrophobicity
2. **Try CFPS** - Faster iteration than cells
3. **Add tags** - MBP, SUMO for solubility
4. **Lower temperature** - 25°C or 16°C

## Tool-specific tips

### RFdiffusion
- Use A10G for standard runs, A100 for large campaigns
- 100-500 designs is typical
- Check secondary structure visually

### ProteinMPNN
- Temperature 0.1 for production, 0.2-0.3 for diversity
- 8-16 sequences per backbone
- Omit cysteines with `--omit_AAs "C"`

### Chai/Boltz
- No MSA required (faster)
- Good for ligand complexes
- CIF output - convert to PDB if needed

### ColabFold
- Use `--model-type alphafold2_multimer_v3` for complexes
- MSA improves accuracy but slower
- `--num-models 1` reduces memory

## When to restart

Consider restarting campaign if:
- 0% pass rate after 500+ designs
- All backbones are random coil
- Hotspots are buried
- Target has major issues (missing regions, wrong chain)

## See also

- [Tool Selection](tool-selection.md) - Try different tool
- [Standard Pipeline](standard-pipeline.md) - Check workflow
- Individual skill troubleshooting sections
