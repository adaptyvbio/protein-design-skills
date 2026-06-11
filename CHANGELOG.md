# Changelog

## [2.1.0] - 2026-06-11

### Added
- `mosaic` skill: multi-objective, gradient-based binder design (Escalante Bio)
- `germinal` skill: de novo antibody and nanobody (VHH) design via biomodals
- `protenix` skill: open AlphaFold3-class structure prediction via biomodals
- `esm` skill now covers ESM C and ESMFold2, including binder design by inverting
  ESMFold2; ESM2 kept as legacy
- protein-qc: SaProtΔG stability prediction, benchmark-backed ipSAE_min ranking with
  thresholds, interface BUNS and ContactMolecularSurface, sequence-liability scan

### Changed
- Binder design is framed by target type and hit-rate rather than a single recommended
  pipeline; adds a proteinbase Nipah head-to-head hit-rate table, features Mosaic as the
  top competition performer, and keeps BoltzGen as the cheap turnkey default with a
  compute-versus-hit-rate note
- Boltz skill documents Boltz-2 as the default, including the affinity-prediction module
- Fixed dangling links: skills now point to `getting-started.md` for setup

### Fixed
- Corrected biomodals references: ProteinMPNN runs via `modal_ligandmpnn.py`, ESM2 via
  `modal_esm2_predict_masked.py`, folding via `modal_esmfold2.py`, and ColabFold steps
  use `modal_alphafold.py`
- RFdiffusion is documented as running from the official RosettaCommons repo, since it
  is not packaged in biomodals
- Corrected the Mosaic `build_multisample_loss` usage and the ESMFold2 / ESM C API calls
- Fact-check pass across all skills: corrected biomodals CLI flags (`--input-pdb`,
  `--input-fasta`, `--input-faa`/`--out-dir`, LigandMPNN `--params-str`), BindCraft and
  BoltzGen run commands, RFdiffusion install/weights/hydra quoting, SolubleMPNN
  `--use_soluble_model`, Foldseek `--max-seqs` default and database sizes, ipSAE
  attribution, SaProtΔG domain range, the inverted BLI koff red flag, and references to
  non-existent `colabfold`/`colabdesign` skills
- Binder-tool default is framed by cost and effort to a binder, since hit-rate is
  target-dependent for every method

### Added (follow-up)
- Protenix-v2 noted for antibody-antigen complexes (`--model-name protenix-v2`)
- Measured compute cost per accepted design (averaged across 7 internal benchmark
  targets) added to binder-design and the relevant tool skills

### Fixed (follow-up)
- Marketplace now ships a single `adaptyv` plugin (all 24 skills). The previous
  category and single-skill plugins all loaded every skill anyway, since a plugin's
  skills are discovered by scanning its source directory and cannot be subset per entry
  in a flat repo. Docs updated to match.
- Corrected stale README and asset SVGs: skill count 21 to 24, design-tools count to 13,
  the install command (`adaptyv@`, not `all@`)

## [2.0.0] - 2026-01-16

### Added
- 20 skills for computational protein design
- Design tools: BoltzGen, BindCraft, RFdiffusion, ProteinMPNN, LigandMPNN, SolubleMPNN
- Validation tools: Chai, Boltz, AlphaFold, ESM
- Evaluation: protein-qc, ipSAE
- Utilities: PDB, UniProt, Foldseek
- Orchestration: binder-design, protein-design-workflow, campaign-manager
- Experimental: cell-free-expression, binding-characterization
- Plugin system with 6 install options (all, design-tools, evaluation, utilities, experimental, orchestration)
- Comprehensive documentation
