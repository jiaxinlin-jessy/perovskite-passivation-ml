# Dataset Documentation

## File: `ML_dataset.xlsx`

This file contains the core training dataset used for machine learning-based screening of HTL/perovskite interface passivation agents.

---

## Overview

| Property | Value |
|----------|-------|
| Number of molecules | 12 (selected from 80-molecule initial library via Kennard-Stone sampling) |
| Benchmark molecule | ID 0 (standard reference passivation agent used in the lab) |
| Feature columns | 17 molecular descriptors |
| Target columns | 4 device performance metrics |
| File format | `.xlsx` (Excel) |

---

## Column Definitions

### Identifier Columns

| Column | Description | Example |
|--------|-------------|---------|
| `Name` | Molecule ID (0–11, 14) corresponding to Figure 3A in the paper. ID 0 is the benchmark molecule. | `0`, `1`, `2` |
| `SMILES` | Canonical SMILES string of the molecule | `C1=CC(=CC=C1N)C(F)(F)F` |

---

### Molecular Descriptor Columns (Features for ML)

These 17 descriptors were computed using RDKit and encode the structural, geometric, and electronic properties of each molecule.

#### Size & Weight

| Column | Full Name | Unit | Description |
|--------|-----------|------|-------------|
| `MW` | Molecular Weight | g/mol | Total molecular mass |
| `NumHeavy` | Number of Heavy Atoms | count | Total non-hydrogen atoms |

#### Topology & Flexibility

| Column | Full Name | Unit | Description |
|--------|-----------|------|-------------|
| `NumRB` | Number of Rotatable Bonds | count | Measure of molecular flexibility |
| `FlexibilityIndex` | Flexibility Index | ratio | NumRB / NumHeavy, normalized flexibility |
| `ConjugationLength` | Conjugation Length | count | Length of the longest conjugated path |

#### Polarity & Solubility

| Column | Full Name | Unit | Description |
|--------|-----------|------|-------------|
| `TPSA` | Topological Polar Surface Area | Å² | Sum of surface contributions of polar atoms; correlated with membrane permeability and wettability |
| `MolLogP` | Wildman-Crippen LogP | dimensionless | Calculated octanol-water partition coefficient; measure of hydrophobicity/lipophilicity. **Most important feature (importance = 0.670)** |

#### Hydrogen Bonding

| Column | Full Name | Unit | Description |
|--------|-----------|------|-------------|
| `HBA` | Hydrogen Bond Acceptors | count | Number of H-bond acceptor atoms (N, O) |
| `HBD` | Hydrogen Bond Donors | count | Number of H-bond donor groups (N-H, O-H) |

#### Hybridization State

| Column | Full Name | Unit | Description |
|--------|-----------|------|-------------|
| `Num_sp` | sp Carbon Count | count | Number of sp-hybridized carbon atoms (e.g., alkynes) |
| `Num_sp2` | sp² Carbon Count | count | Number of sp²-hybridized carbon atoms (aromatic + alkene) |
| `Num_sp3` | sp³ Carbon Count | count | Number of sp³-hybridized carbon atoms (saturated) |
| `F_sp3` | Fraction of sp³ Carbons | ratio | Num_sp3 / total carbons; measure of 3D character |

#### Heteroatoms & Aromaticity

| Column | Full Name | Unit | Description |
|--------|-----------|------|-------------|
| `NumHetero` | Number of Heteroatoms | count | Non-carbon, non-hydrogen atoms (N, O, F, Cl, etc.) |
| `NumAR` | Number of Aromatic Rings | count | Count of aromatic ring systems |
| `NumSC` | Number of Stereocenters | count | Number of chiral centers |

#### Categorical Descriptors

| Column | Full Name | Description |
|--------|-----------|-------------|
| `AromaticSpecies` | Aromatic Ring Types | List of aromatic moieties present (e.g., `['benzene']`) |
| `HasAmines` | Has Amine Group | Boolean (TRUE/FALSE) — whether the molecule contains a primary or secondary amine functional group |

---

### Target / Performance Columns

These columns contain experimentally measured device performance data from fabricated perovskite solar cells (ITO/SnO₂/perovskite/passivation layer/Spiro-OMeTAD/Ag architecture).

| Column | Full Name | Unit | Description |
|--------|-----------|------|-------------|
| `Efficiency (%)_mean` | Mean Power Conversion Efficiency | % | Average PCE across multiple devices. **Primary ML target variable** |
| `Jsc (mA/cm²)_mean` | Mean Short-Circuit Current Density | mA/cm² | Average current density at zero voltage |
| `Voc (V)_mean` | Mean Open-Circuit Voltage | V | Average voltage at zero current |
| `Fill Factor (%)_mean` | Mean Fill Factor | % | Ratio of maximum power to (Jsc × Voc); strongly correlated with PCE (r = 0.98) |

> **Note on ML target construction:** The ML classification task uses `Efficiency (%)_mean` as the target, binarized at the median value (above median = class 1, below = class 0).

---

## Performance Correlations

Based on the Gradient Boosting model analysis:

| Metric pair | Pearson r |
|-------------|-----------|
| Efficiency vs FF | **0.98** (very strong) |
| Efficiency vs Voc | **0.83** (strong) |
| Efficiency vs Jsc | 0.44 (moderate) |
| Voc vs FF | 0.74 (strong) |

**Implication:** Passivation agents that improve Voc and FF have a stronger positive impact on overall PCE than those that primarily boost Jsc.

---

## File: `mol2mol.smi`

Contains the SMILES strings of the 5 seed molecules used as input to the REINVENT4 reinforcement learning model. These were selected as the top-performing molecules from the initial 12-molecule ML screening.

```
C1=CC(=C(C=C1N)F)C(F)(F)F         # ID 2
C1=C(C=C(C(=C1Cl)N)Cl)C(F)(F)F   # ID 3
CNC1=CC=C(C=C1)C(F)(F)F           # ID 5
C1=C(C=C(C=C1C(F)(F)F)F)CN([H])[H]  # ID 7
CC(C)(C)C1=CC=C(C=C1)CN           # ID 11
```

These 5 molecules were chosen based on their combined ML model score and experimental PCE performance.

---

## Notes & Caveats

- **Small sample size:** Only 12 molecules in the training set. Cross-validation results (especially Random Forest std = ±0.40) show high variance due to ~2–3 test samples per fold. Treat model outputs as trend indicators, not absolute predictions.
- **Binary classification:** The ML task is formulated as binary classification (high/low PCE relative to median), not regression.
- **Benchmark molecule (ID 0):** Uses a SMILES with `(C)(` notation indicating tert-butyl substitution; serves as the internal reference for all PL and device comparisons.
- **Missing IDs:** Molecules are labeled 0–11 and 14 (ID 14 = 4-tert-butylbenzylamine, the tBBA reference). Not all integer IDs are present.
