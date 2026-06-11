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
