# Changelog

## [2.1.0] - 2026-06-11

### Added
- `mosaic` skill: multi-objective, gradient-based binder design (Escalante Bio)
- `esm` skill now covers ESM C and ESMFold2, including binder design by inverting
  ESMFold2; ESM2 kept as legacy
- protein-qc: SaProtΔG stability prediction, benchmark-backed ipSAE_min ranking with
  thresholds, interface BUNS and ContactMolecularSurface, sequence-liability scan

### Changed
- Boltz skill documents Boltz-2 as the default, including the affinity-prediction module
- Binder design no longer frames BoltzGen as the single recommended pipeline; tools are
  chosen by target type and compute
- Fixed dangling links: skills now point to `getting-started.md` for setup
- Removed the dangling Germinal skill reference (antibody/nanobody tools noted as external)

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
