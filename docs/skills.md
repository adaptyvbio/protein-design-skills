# Skills

All 21 skills available in Protein Design Skills.

## Install

```
/plugin marketplace add adaptyvbio/protein-design-skills
/plugin install adaptyv@protein-design-skills
```

Or install specific categories:

| Command | Skills | Description |
|---------|--------|-------------|
| `/plugin install adaptyv@protein-design-skills` | 21 | Everything |
| `/plugin install design-tools@protein-design-skills` | 10 | BoltzGen, BindCraft, RFdiffusion, ProteinMPNN, Chai, etc. |
| `/plugin install evaluation@protein-design-skills` | 2 | protein-qc, ipsae |
| `/plugin install utilities@protein-design-skills` | 4 | pdb, uniprot, foldseek, setup |
| `/plugin install experimental@protein-design-skills` | 2 | cell-free-expression, binding-characterization |
| `/plugin install orchestration@protein-design-skills` | 3 | binder-design, protein-design-workflow, campaign-manager |

---

## Design tools (10)

| Skill | Purpose |
|-------|---------|
| `boltzgen` | All-atom diffusion (recommended) |
| `bindcraft` | End-to-end binder design |
| `rfdiffusion` | Backbone generation |
| `proteinmpnn` | Inverse folding |
| `ligandmpnn` | Ligand-aware sequence design |
| `solublempnn` | Solubility-optimized design |
| `chai` | Structure prediction (fast) |
| `boltz` | Structure prediction (open-source) |
| `alphafold` | Structure prediction (reference) |
| `esm` | Sequence embeddings/scoring |

## Evaluation (2)

| Skill | Purpose |
|-------|---------|
| `protein-qc` | QC metrics and filtering |
| `ipsae` | ipSAE ranking for binders |

## Utilities (4)

| Skill | Purpose |
|-------|---------|
| `pdb` | Structure retrieval |
| `uniprot` | Sequence lookup |
| `foldseek` | Structure similarity search |
| `setup` | First-time environment setup |

## Experimental (2)

| Skill | Purpose |
|-------|---------|
| `cell-free-expression` | CFPS optimization |
| `binding-characterization` | SPR/BLI kinetics |

## Orchestration (3)

| Skill | Purpose |
|-------|---------|
| `binder-design` | Tool selection guidance |
| `protein-design-workflow` | Pipeline guidance |
| `campaign-manager` | Campaign planning |

---

## Skill details

For detailed usage of each skill, see the individual skill documentation:

- **Design tools**: [BoltzGen](../skills/boltzgen/SKILL.md), [BindCraft](../skills/bindcraft/SKILL.md), [RFdiffusion](../skills/rfdiffusion/SKILL.md), [ProteinMPNN](../skills/proteinmpnn/SKILL.md), [LigandMPNN](../skills/ligandmpnn/SKILL.md), [SolubleMPNN](../skills/solublempnn/SKILL.md), [Chai](../skills/chai/SKILL.md), [Boltz](../skills/boltz/SKILL.md), [AlphaFold](../skills/alphafold/SKILL.md), [ESM](../skills/esm/SKILL.md)

- **Evaluation**: [Protein QC](../skills/protein-qc/SKILL.md), [ipSAE](../skills/ipsae/SKILL.md)

- **Utilities**: [PDB](../skills/pdb/SKILL.md), [UniProt](../skills/uniprot/SKILL.md), [FoldSeek](../skills/foldseek/SKILL.md), [Setup](../skills/setup/SKILL.md)

- **Experimental**: [Cell-Free Expression](../skills/cell-free-expression/SKILL.md), [Binding Characterization](../skills/binding-characterization/SKILL.md)

- **Orchestration**: [Binder Design](../skills/binder-design/SKILL.md), [Protein Design Workflow](../skills/protein-design-workflow/SKILL.md), [Campaign Manager](../skills/campaign-manager/SKILL.md)

---

## See also

- [Getting started](getting-started.md)
- [Standard pipeline](standard-pipeline.md)
- [Compute setup](compute-setup.md)
