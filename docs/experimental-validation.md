# Experimental Validation

Bridging computational design to wet lab testing.

## Overview

After computational filtering, validate designs experimentally:

```
Computational QC → Expression → Binding Assay → Characterization
   (protein-qc)      (CFPS)       (ELISA)        (SPR/BLI)
```

## Expression testing

**Skill**: `cell-free-expression`

### System selection

| System | Yield | PTMs | Disulfides | Best For |
|--------|-------|------|------------|----------|
| E. coli extract | High | None | Limited | Quick screen |
| PURE | Medium | None | Yes | Toxic proteins |
| Wheat germ | Medium | Some | Yes | Complex proteins |

### Quick protocol

```
1. Order DNA (codon-optimized for E. coli CFPS)
2. Prepare reaction (25µL scale)
3. Express at 30°C for 4h
4. SDS-PAGE to verify expression
5. Western blot to confirm identity
```

### Troubleshooting expression

| Issue | Likely Cause | Fix |
|-------|--------------|-----|
| No expression | Rare codons | Check codon usage, optimize |
| Low yield | mRNA stability | Optimize 5' UTR |
| Aggregation | Hydrophobic | Lower temperature, add chaperones |
| Truncation | Rare codon cluster | Reoptimize codons |

## Binding assays

### ELISA (primary screen)
- **Throughput**: High (96-384 designs)
- **Sensitivity**: Moderate
- **Best for**: Initial triage

### SPR vs BLI

**Skill**: `binding-characterization`

| Aspect | SPR | BLI |
|--------|-----|-----|
| Sensitivity | Higher | Good |
| Throughput | Lower | Higher |
| Sample purity | Critical | Tolerant |
| Kinetic resolution | Better | Good |
| Best for | Lead characterization | Screening |

### When BLI works but SPR doesn't

| Cause | Solution |
|-------|----------|
| Hydrophobic CDRs | Add detergent |
| Aggregation | Optimize buffer, SEC purify |
| Instability | Test at lower temperature |

### When SPR works but BLI doesn't

| Cause | Solution |
|-------|----------|
| Small analyte | Use SPR |
| Weak affinity | Increase concentration |
| Low expression | Scale up production |

## Kinetic analysis

### Quality checklist

- [ ] Baseline stable
- [ ] Association curve fits 1:1 model
- [ ] Dissociation complete
- [ ] Residuals < 10% Rmax
- [ ] Chi² < 10% Rmax²

### Interpreting results

| KD Range | Interpretation |
|----------|----------------|
| < 1 nM | Excellent (therapeutic) |
| 1-10 nM | Very good |
| 10-100 nM | Good (starting point) |
| 100 nM - 1 µM | Moderate (needs optimization) |
| > 1 µM | Weak (consider redesign) |

## Optimization path

```
ipTM > 0.5, pLDDT > 0.85 (computational)
         ↓
    Express in CFPS
         ↓
    ELISA screen
         ↓
    BLI confirmation
         ↓
    SPR characterization
         ↓
    Affinity optimization
```

## Common artifacts

### Biphasic binding
- **Symptom**: Two-rate association/dissociation
- **Cause**: Heterogeneous ligand or avidity
- **Fix**: Reduce ligand density, fit heterogeneous model

### Negative dissociation
- **Symptom**: Signal increases during dissociation
- **Cause**: Mass transport or aggregation
- **Fix**: Reduce flow rate, dilute analyte

### Hook effect
- **Symptom**: Signal decreases at high concentration
- **Cause**: Surface saturation
- **Fix**: Reduce ligand density

## Integration with computational

| Experimental Result | Computational Next Step |
|---------------------|------------------------|
| No expression | Check sequence liabilities (rare codons) |
| Aggregation | Check hydrophobicity, redesign |
| Weak binding | Different hotspots, longer binder |
| Off-target binding | Negative design constraints |

## See also

- [Cell-Free Expression skill](../skills/cell-free-expression/SKILL.md)
- [Binding Characterization skill](../skills/binding-characterization/SKILL.md)
- [Standard Pipeline](standard-pipeline.md)
