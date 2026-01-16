# Contributing to Protein Designer Skills

Thank you for your interest in contributing to Protein Designer Skills! This document provides guidelines for contributing new skills or improving existing ones.

## Skill Structure

Every skill must follow the [Anthropic Skills Specification](https://github.com/anthropics/skills):

```
skill-name/
├── SKILL.md              # Required: Frontmatter + instructions
├── references/           # Optional: Detailed documentation
├── scripts/              # Optional: Executable helpers
└── assets/               # Optional: Templates, examples
```

## Creating a New Skill

### Quick Start (5 minutes)

1. **Clone and setup**
   ```bash
   git clone https://github.com/adaptyvbio/protein-design-skills
   cd protein-design-skills
   ```

2. **Create skill directory**
   ```bash
   mkdir -p skills/your-skill-name
   ```

3. **Create SKILL.md with required metadata**

### Required Metadata (YAML Frontmatter)

```yaml
---
name: skill-name
description: >
  Description with numbered use cases (1), (2), (3)...
  Include when to use this skill AND when to use alternatives.
license: MIT
category: design-tools | evaluation | utilities | experimental | orchestration
tags: [relevant, tags, here]
proteinbase_slug: method-name  # Optional, if on proteinbase.com
biomodals_script: modal_*.py   # Optional, for Modal execution
---
```

### Category Reference

| Category | Purpose | Examples |
|----------|---------|----------|
| `design-tools` | Core design methods | boltzgen, bindcraft, rfdiffusion, chai |
| `evaluation` | Metrics and filtering | protein-qc, ipsae |
| `utilities` | Database access, conversions | pdb, uniprot, foldseek |
| `experimental` | Experimental validation | cell-free-expression, binding-characterization |
| `orchestration` | End-to-end workflows | binder-design, campaign-manager |

### Tag Taxonomy

| Tag | Description |
|-----|-------------|
| `structure-design` | Generates backbone/structure |
| `sequence-design` | Generates sequences |
| `structure-prediction` | Predicts structure |
| `inverse-folding` | Backbone → sequence |
| `diffusion` | Uses diffusion models |
| `all-atom` | All-atom generation |
| `docking` | Molecular docking |
| `pipeline` | Multi-model workflow |

## SKILL.md Template

```yaml
---
name: tool-name
description: >
  Brief description of what the tool does. Use this skill when:
  (1) First use case,
  (2) Second use case,
  (3) Third use case.

  For related functionality, use other-skill.
  For QC thresholds, use protein-qc.
license: MIT
category: design-tools
tags: [relevant, tags]
biomodals_script: modal_tool.py  # Optional
---

# Tool Name

## Prerequisites

| Requirement | Minimum | Recommended |
|-------------|---------|-------------|
| Python | 3.10+ | 3.11 |
| CUDA | 12.0+ | 12.1+ |
| GPU VRAM | 24GB | 48GB |

## How to Run

> **First time?** See [Compute Setup](../../docs/compute-setup.md) to choose where to run.

### Option 1: Modal (Recommended)
\`\`\`bash
git clone https://github.com/hgbrian/biomodals && cd biomodals
uvx modal run modal_tool.py --input input.pdb
\`\`\`

**GPU**: A10G (24GB) | **Timeout**: 600s default

### Option 2: Local Installation
\`\`\`bash
git clone https://github.com/org/tool.git
pip install -e .
python tool.py --input input.pdb
\`\`\`

## Key Parameters

| Parameter | Default | Range | Description |
|-----------|---------|-------|-------------|
| `--param1` | 10 | 1-100 | Description |

## Output Format

\`\`\`
output/
├── result.pdb
└── metrics.json
\`\`\`

## Sample Output

### Successful Run
\`\`\`
$ uvx modal run modal_tool.py --input input.pdb
[INFO] Loading model...
[INFO] Processing...
[INFO] Saved to output/
\`\`\`

**What good output looks like:**
- Metric1: > threshold
- Metric2: < threshold

## Decision Tree

\`\`\`
Should I use this tool?
│
├─ Use case 1? → This tool ✓
├─ Use case 2? → Alternative tool
└─ Use case 3? → This tool ✓
\`\`\`

## Typical Performance

| Campaign Size | Time | Cost (Modal) | Notes |
|---------------|------|--------------|-------|
| 100 | 30 min | ~$5 | Standard |

---

## Verify

\`\`\`bash
ls output/*.pdb | wc -l  # Should match expected
\`\`\`

---

## Troubleshooting

**Common issue**: Solution here

### Error Interpretation

| Error | Cause | Fix |
|-------|-------|-----|
| `Error message` | Cause | Solution |

---

**Next**: Use output with `next-skill` for next step.
```

## Content Guidelines

1. **Keep it under 500 lines** - Use `references/` for detailed docs
2. **Imperative form** - "Run the command" not "You can run"
3. **Include execution patterns** - Both Modal and local options
4. **Add parameter tables** - With defaults, ranges, descriptions
5. **Include troubleshooting** - Common issues and solutions
6. **Decision trees** - When to use this vs alternatives

## Creating Skills with Claude

The easiest way to create a new skill is to use Claude Code's built-in skill creator:

```
/skill skill-creator
```

This will guide you through creating a well-structured skill. See [Anthropic's skill-creator](https://github.com/anthropics/skills/tree/main/skills/skill-creator) for details.

## Suggested Sections (Tool Skills)

These sections help users understand and use your skill effectively:

1. **Prerequisites** - Python, CUDA, GPU VRAM requirements
2. **How to Run** - Modal (recommended) and/or local options
3. **Key Parameters** - Table with defaults and ranges
4. **Output Format** - Directory structure, file formats
5. **Sample Output** - What success looks like
6. **Decision Tree** - When to use this vs alternatives
7. **Typical Performance** - Time and cost estimates
8. **Troubleshooting** - Common errors and fixes

## Quality Checklist

Before submitting:

- [ ] YAML frontmatter complete (name, description, license, category, tags)
- [ ] Execution instructions provided (one or both):
  - [ ] `biomodals_script` for Modal execution, OR
  - [ ] Local installation instructions with dependencies
- [ ] Parameter table with defaults
- [ ] Output format documented
- [ ] Sample output included
- [ ] Decision tree for tool selection
- [ ] Related skills cross-referenced
- [ ] Under 500 lines (use references/ for details)
- [ ] Added to marketplace.json

## Pull Request Process

1. Fork the repository
2. Create a feature branch: `git checkout -b add-skill-name`
3. Add your skill following the guidelines
4. Test with Claude Code to ensure skill triggers correctly
5. Add skill to `.claude-plugin/marketplace.json`
6. Submit PR with description of the skill

## Testing Your Skill

1. Add your local clone to Claude Code:
   ```
   /plugin marketplace add /path/to/your/local/clone
   /plugin install adaptyv@protein-design-skills
   ```
2. Start a new Claude Code session
3. Test that skill triggers on relevant queries (e.g., "How do I design a binder?" should trigger binder-design)

## Questions?

Open an issue on GitHub for questions or discussions about new skills.
