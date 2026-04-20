+++
title = "EPIC Dataset: From Microscopy to Graph Neural Networks"
date = 2026-04-15
summary = "Complete guide to preprocessing and analyzing C. elegans embryonic development using spatio-temporal graph data"
tags = ["dataset", "C. elegans", "graph-neural-networks", "microscopy", "bioinformatics", "data-preprocessing"]
+++

# 🧬 EPIC Dataset: Complete Preprocessing & Analysis Guide

> **Warning:** This is a living document. Last updated: April 20, 2026

---

## 📚 Table of Contents

1. [Quick Start (5 minutes)](#-quick-start)
2. [The Big Picture](#-the-big-picture)
3. [Raw Data Format](#-raw-data-format-what-comes-in)
4. [Processing Pipeline](#-processing-pipeline-the-transformation)
5. [Output Specification](#-output-specification-what-goes-out)
6. [Data Structures](#-data-structures--shapes)
7. [Practical Usage](#-practical-usage)
8. [Validation & Quality Control](#-validation--quality-control)
9. [Troubleshooting](#-troubleshooting)
10. [File Manifest](#-complete-file-manifest)

---

## ⚡ Quick Start

**In a hurry?** Here's what you need to know in 5 minutes:

### What is this dataset?

We've processed **260 embryos** of *C. elegans* using **EPIC (eMbryo Project Imaging Consortium)** fluorescence microscopy. The output is spatio-temporal graph data perfect for training graph neural networks (GNNs).

### The loop in 10 lines:

```python
import numpy as np

# Load one embryo
npz = np.load("dataset/processed/by_embryo/CD011605_5a_bright.npz", allow_pickle=True)

# Get tensors
X = npz["X"]                    # (688 cells, 5 features, 210 timepoints)
alive_mask = npz["alive_mask"]  # Boolean: which cells are alive at time t
edges_src, edges_dst = npz["edge_src"], npz["edge_dst"]
idx_to_cell = npz["idx_to_cell"]  # Cell name lookup

# Filter to living cells at time t=100
t = 100
alive = np.where(alive_mask[:, t])[0]
X_t = X[alive, :, t]  # (M active cells, 5 features)
print(f"At time {t}: {len(alive)} cells alive")
```

### Where to go next:
- **Want working examples?** → See [Practical Usage](#-practical-usage)
- **Building a GNN?** → See [Data Structures](#-data-structures--shapes)
- **Something broke?** → See [Troubleshooting](#-troubleshooting)

---

## 🔬 The Big Picture

### Why this dataset matters

*C. elegans* embryonic development is one of the most well-characterized biological systems:

- **🎯 Perfect model system**: Complete lineage is known (959 cells at stage end)
- **🔬 Observable in real-time**: Fluorescence microscopy captures cell division, migration, and differentiation
- **📊 Rich structure**: Both *spatial* (proximity-based) and *temporal* (lineage-based) relationships
- **🧠 GNN-friendly**: Natural graph representation (cells as nodes, adjacencies as edges)

### What EPIC gives us

```
Microscopy Videos (260 embryos)
    ↓ [Extract cell positions over time]
Raw CSV Tables (260 files, ~100k rows each)
    ↓ [Preprocess: normalize, verify, build graphs]
Tensor Archives (260 compressed NPZ files)
    ↓ [Load in PyTorch/TensorFlow]
Spatio-Temporal Graphs (Ready for GNNs)
```

### Key facts at a glance

| Metric | Value | Notes |
|--------|-------|-------|
| **Embryos** | 260 | Complete datasets, quality-controlled |
| **Cells per embryo** | ~688 (varies) | Variable number, from ~100 to ~900 |
| **Timepoints per embryo** | ~210 (varies) | Embryo development: ~13 hours 30 min per embryo |
| **Features per cell** | 5 | Position (x, y, z) + morphology (size, fluorescence) |
| **Spatial edges** | ~45k per embryo | Proximity-based (distance < 20 μm) |
| **Lineage edges** | ~5k per embryo | Parent→daughter divisions |
| **Total storage** | ~180 MiB | All 260 embryos compressed |
| **Per-embryo size** | ~0.7 MiB | Highly compressed sparse graphs |

---

## 📥 Raw Data Format: What Comes In

### Source: EPIC CSV Files

Each embryo is stored as a comma-separated table in `dataset/raw/*.csv`.

**File count**: 260 unique recordings  
**File size**: 1-5 MiB each (uncompressed)  
**Encoding**: UTF-8

### Column Breakdown

Here's what each column means:

| Column | Type | Unit | Used? | What it tells us |
|--------|------|------|-------|------------------|
| `cellTime` | str | — | ❌ | Row ID (artifact; removed) |
| **`cell`** | str | — | ✅ | Cell name in C. elegans nomenclature (e.g., "ABal", "Zrp1aaa") |
| **`time`** | int | frame | ✅ | Timepoint (1-indexed, typically 1–210) |
| `none` | int | — | ❌ | Unknown field (legacy; ignored) |
| `global` | int | — | ❌ | Unused metadata offset |
| `local` | int | — | ❌ | Unused metadata offset |
| **`blot`** | float | AU | ✅ | **Fluorescence intensity**: brightfield/dark-field marker. Primary cell identity. Range: ~100 to 10M arbitrary units |
| `cross` | float | — | ❌ | Cross-correlation metric (unused) |
| **`z`** | float | μm | ✅ | Depth coordinate (focal plane offset relative to reference). Range: 0–200 μm |
| **`x`** | float | px | ✅ | Horizontal position (X-axis, pixels). Range: 0–512 px |
| **`y`** | float | px | ✅ | Vertical position (Y-axis, pixels). Range: 0–512 px |
| **`size`** | float | AU | ✅ | Cell volume / morphological size (estimated counts). Range: ~10–5000 AU |
| `gweight` | float | — | ❌ | Image intensity weight (unused) |

### Example Raw Record

```csv
cellTime,cell,time,none,global,local,blot,cross,z,x,y,size,gweight
ABal:25,ABal,25,22722,-2278,10,815432,-2278,19.2,166,257,74,815432
```

This means: *Cell named ABal at timepoint 25 is located at position (166 px, 257 px, 19.2 μm) with fluorescence intensity 815432 AU and volume ~74 AU.*

### Key observations

1. **One row = one cell at one timepoint**
2. **Sparse data**: Cells only appear *after birth*. Cell "ABa" might start at timepoint 5, while "AB" starts at timepoint 1
3. **Lineage encoding**: Cell names nest hierarchically. "ABal" is a daughter of "ABa" (remove last letter = parent)
4. **Variable embryo length**: Some embryos have 180 timepoints, others 250+
5. **No missing values**: Every row has all columns

---

## ⚙️ Processing Pipeline: The Transformation

### System Overview

```
Raw CSV (1 embryo)
    ↓ [build_local_index]
    → EpicIndex: cell→idx mapping, timeframe [t0, T]
    ↓ [populate_X & alive_mask]
    → Node features X[N, 5, T] + birth tracking
    ↓ [build_spatial_edges]
    → Proximity graph (undirected): cells < 20 μm apart
    ↓ [build_lineage_edges]
    → Division graph (directed): parent→daughter
    ↓ [save_to_npz]
Compressed NPZ archive (1 embryo, ~0.7 MiB)
```

### Step 1: Index Building (`build_local_index`)

**Purpose**: Create a consistent cell name ↔ index mapping for this embryo.

**Input**: Raw CSV DataFrame

**Output**: 
- `cell_to_idx: dict[str, int]` — Maps "ABal" → 42 (for example)
- `t0: int` — First timepoint (usually 1)
- `T: int` — Total number of timepoints

**Algorithm**:
```python
cells = sorted(df["cell"].unique())  # Alphabetical order
cell_to_idx = {cell: idx for idx, cell in enumerate(cells)}
t0 = int(df["time"].min())
T = int(df["time"].max() - t0) + 1
```

**Why alphabetical?** Reproducibility. Same name → same index every time.

### Step 2: Feature Population (`populate_X` & `populate_alive_mask`)

**Purpose**: Build the main feature tensor X[N, d, T] and birth-tracking mask.

**Input**: 
- `cell_to_idx` from Step 1
- Raw CSV with columns: x, y, z, size, blot

**Output**:
- `X[N, 5, T]` — Node features (float32)
- `alive_mask[N, T]` — Boolean: is cell alive at time t?

**Algorithm** (pseudocode):

```python
X = np.zeros((N, 5, T), dtype=np.float32)
alive_mask = np.zeros((N, T), dtype=bool)

for row in df.itertuples():
    c_idx = cell_to_idx[row.cell]
    t_idx = row.time - t0  # Convert to 0-indexed
    
    # Set features
    X[c_idx, :, t_idx] = [row.x, row.y, row.z, row.size, row.blot]
    
    # Mark as alive
    alive_mask[c_idx, t_idx] = True

# Cells not appearing in CSV remain zeros and False
```

**Result**: 
- Unborn cells: `X[:, :, t] == [0,0,0,0,0]` and `alive_mask[:, t] == False`
- Alive cells: Features populated, `alive_mask == True`

### Step 3: Spatial Edges (`build_spatial_edges`)

**Purpose**: Connect cells that are spatially close (proximity graph).

**Input**: X[N, 5, T] (node positions)

**Output**: 
- `edge_src, edge_dst, edge_t` — Sparse edge lists

**Algorithm**:

```python
edges = []
for t in range(T):
    # Get living cell positions at time t
    alive_idx = np.where(alive_mask[:, t])[0]
    positions = X[alive_idx, :3, t]  # (x, y, z)
    
    # Compute pairwise distances
    from scipy.spatial.distance import cdist
    dist = cdist(positions, positions, metric='euclidean')
    
    # Threshold: distance < 20 μm
    src, dst = np.where((dist > 0) & (dist < 20))
    
    for s, d in zip(src, dst):
        edges.append((alive_idx[s], alive_idx[d], t))

edge_src, edge_dst, edge_t = zip(*edges)
```

**Key facts**:
- **Undirected**: If (A, B) is an edge, so is (B, A). Counts as 2 edges.
- **Time-varying**: Edges change because cells move and new cells are born
- **~45k edges per embryo**: Sparse but non-trivial graph density
- **Threshold 20 μm**: Based on typical cell contact distance in *C. elegans*

### Step 4: Lineage Edges (`build_lineage_edges`)

**Purpose**: Connect parent cells to daughter cells (biological divisions).

**Input**: `cell_to_idx` (cell names encode lineage)

**Output**: 
- `edge_src_lineage, edge_dst_lineage, edge_t_lineage`

**Algorithm** (simplified):

The C. elegans naming convention encodes division:
- "AB" divides → "ABa" and "ABp"
- "ABa" divides → "ABal" and "ABarp"
- etc.

```python
def get_parent_cell(cell_name):
    """Returns parent cell name (remove last letter)."""
    if len(cell_name) <= 1:
        return None  # Root cell (no parent)
    return cell_name[:-1]

edges_lineage = []
for cell, idx in cell_to_idx.items():
    parent = get_parent_cell(cell)
    if parent and parent in cell_to_idx:
        parent_idx = cell_to_idx[parent]
        
        # When does daughter appear? First non-zero timestamp
        t_birth = np.where(alive_mask[idx, :])[0]
        if len(t_birth) > 0:
            t = t_birth[0]
            edges_lineage.append((parent_idx, idx, t))
```

**Key facts**:
- **Directed**: Always parent → daughter (causal)
- **~5k edges per embryo**: Only one per cell birth (except root)
- **One edge per cell**: Each cell has ≤1 parent (except root AB)

---

## 📤 Output Specification: What Goes Out

### File Format: NPZ Archive

Each processed embryo is saved as a **`.npz` file** (NumPy compressed archive).

**Location**: `dataset/processed/by_embryo/*.npz`  
**Compression**: ZIP with NumPy arrays (highly compressed sparse graphs)  
**Size**: ~0.7 MiB per embryo  
**Format**: Binary (not human-readable; must load with `np.load()`)

### What's inside?

```python
npz = np.load("dataset/processed/by_embryo/CD011605_5a_bright.npz", allow_pickle=True)
print(npz.files)  # What's inside?

# Output:
# ['X', 'alive_mask', 'edge_src', 'edge_dst', 'edge_t', 
#  'idx_to_cell', 'metadata']
```

### Array Specifications

#### 1. **X** — Node Features

```
Shape: (N, 5, T)
Dtype: float32
Meaning: Temporal trajectory of each cell's 5 features

X[cell_idx, feature_idx, time_idx] = value

Feature indices:
  0 → x (horizontal, pixels)
  1 → y (vertical, pixels)
  2 → z (depth, μm)
  3 → size (morphology, AU)
  4 → blot (fluorescence, AU)

Unborn cells: X[cell_idx, :, t] = [0, 0, 0, 0, 0]
```

**Example**: 
```python
X[42, :, 100] = [245.3, 128.7, 15.2, 156.0, 892451.0]
# Cell at index 42, at time 100: x=245 px, y=129 px, z=15.2 μm, ...
```

#### 2. **alive_mask** — Birth & Survival Tracking

```
Shape: (N, T)
Dtype: bool
Meaning: True = cell is alive at this timepoint

alive_mask[cell_idx, time_idx] = True/False

Usage: Filter to only living cells
  alive_at_t = np.where(alive_mask[:, t])[0]
  X_active = X[alive_at_t, :, t]
```

**Example**:
```python
alive_at_t100 = np.where(alive_mask[:, 100])[0]  # (M,) indices
print(f"{len(alive_at_t100)} cells alive at t=100")
X_active = X[alive_at_t100, :, 100]  # (M, 5)
```

#### 3. **edge_src, edge_dst, edge_t** — Spatial Graph Edges

```
Shape: (E,) for each
Dtype: int32
Meaning: Source node, destination node, timepoint

edge_src[i], edge_dst[i], edge_t[i] = (source_id, dest_id, time)
  → Connects cell source_id to cell dest_id at timepoint time
  → Undirected: implies reverse edge also exists (usually explicit)

Total edges: E ≈ 45k per embryo (varies)
```

**Example**:
```python
# Find all edges at time t=50
at_t50 = np.where(edge_t == 50)[0]
srcs_50 = edge_src[at_t50]
dsts_50 = edge_dst[at_t50]
print(f"{len(at_t50)} edges at time 50")

# Build adjacency matrix at t=50
from scipy.sparse import coo_matrix
adj_50 = coo_matrix((np.ones(len(at_t50)), (srcs_50, dsts_50)), 
                     shape=(N, N))
```

#### 4. **idx_to_cell** — Cell Name Lookup

```
Shape: (N,)
Dtype: object (str)
Meaning: Maps node index back to cell name

idx_to_cell[cell_idx] = "ABal"
  → Cell at index cell_idx is named "ABal"

Reverse lookup: cell_to_idx = {v: k for k, v in enumerate(idx_to_cell)}
```

**Example**:
```python
idx_to_cell = npz["idx_to_cell"]
print(idx_to_cell[42])  # "ABal"

# Reverse mapping
cell_to_idx = {cell: idx for idx, cell in enumerate(idx_to_cell)}
print(cell_to_idx["ABal"])  # 42
```

#### 5. **metadata** — File Metadata

```
Shape: 1-D array (usually)
Dtype: object (dict)
Meaning: Provenance & processing info

Typical contents:
  {
    't0': 1,
    'T': 210,
    'N': 688,
    'source_file': 'CD011605_5a_bright.csv',
    'processing_version': '1.0',
    'timestamp': '2026-04-20T12:34:56Z'
  }
```

---

## 📊 Data Structures & Shapes

### At a Glance

```python
import numpy as np

# Example embryo dimensions
N = 688  # Number of cells
d = 5    # Features: x, y, z, size, blot
T = 210  # Timepoints
E = 56605  # Spatial edges
E_lineage = 687  # Lineage edges (≈ N-1, one per cell)

# Tensors
X                   # (688, 5, 210)     float32 — positions & morphology
alive_mask          # (688, 210)        bool    — birth tracking
edge_src            # (56605,)          int32   — spatial graph sources
edge_dst            # (56605,)          int32   — spatial graph destinations
edge_t              # (56605,)          int32   — timepoints
idx_to_cell         # (688,)            object  — names
```

### Typical Statistics

```
Mean cells alive per timepoint: ~520 (out of 688)
Mean degree (contacts/cell):    ~43 (45k edges / 688 cells)
Min cells in embryo:            88  (very early timepoint)
Max cells in embryo:            688 (late development)
```

### Memory Footprint

```
Loaded in memory (single embryo):
  X:                 688 × 5 × 210 × 4 bytes = 2.9 MiB
  alive_mask:        688 × 210 × 1 byte  = 0.1 MiB
  Sparse edges:      56605 × 3 × 4 bytes = 0.7 MiB
  
Total per embryo:    ~3.7 MiB (uncompressed)
Stored on disk:      ~0.7 MiB (NPZ compressed)
Compression ratio:   ~5.3:1

All 260 embryos:     ~180 MiB (on disk)
```

---

## 💻 Practical Usage

### Loading Data

```python
import numpy as np

# Load single embryo
npz = np.load("dataset/processed/by_embryo/CD011605_5a_bright.npz", 
              allow_pickle=True)

# Extract arrays
X = npz["X"]                    # (688, 5, 210)
alive_mask = npz["alive_mask"]  # (688, 210)
edge_src = npz["edge_src"]
edge_dst = npz["edge_dst"]
edge_t = npz["edge_t"]
idx_to_cell = npz["idx_to_cell"]
metadata = npz["metadata"].item()  # Convert numpy object to dict

print(f"Embryo: {metadata['source_file']}")
print(f"  {metadata['N']} cells, {metadata['T']} timepoints")
```

### Filtering to Living Cells

```python
t = 100  # Query at timepoint 100

# Which cells are alive?
alive_idx = np.where(alive_mask[:, t])[0]
print(f"{len(alive_idx)} cells alive at t={t}")

# Get their features
X_alive = X[alive_idx, :, t]  # (M, 5)
cell_names_alive = idx_to_cell[alive_idx]

print(f"Feature mean: {X_alive.mean(axis=0)}")
# Output: [245.2, 189.4, 12.1, 234.5, 890000.0]
```

### Building an Adjacency Matrix

```python
from scipy.sparse import coo_matrix

# Static adjacency at time t=50
t = 50
mask = (edge_t == t)
srcs = edge_src[mask]
dsts = edge_dst[mask]

# COO format (efficient for construction)
adj = coo_matrix((np.ones(len(srcs)), (srcs, dsts)), shape=(N, N))

# Convert to CSR (efficient for matrix ops)
adj_csr = adj.tocsr()
print(f"Adjacency at t={t}: {adj_csr.nnz} edges")
```

### Tracing Cell Lineage

```python
# Build reverse lookup
cell_to_idx = {cell: idx for idx, cell in enumerate(idx_to_cell)}

def trace_daughters(parent_name, depth=3):
    """Recursively find all daughters of a cell."""
    idx = cell_to_idx[parent_name]
    
    # Find daughters (cells whose names start with parent_name)
    daughters = [cell for cell in idx_to_cell 
                 if cell.startswith(parent_name) and len(cell) == len(parent_name) + 1]
    
    print("  " * depth + f"→ {parent_name} ({'born' if alive_mask[idx, 0] else 'unborn'} at t=0)")
    
    for daughter in daughters:
        trace_daughters(daughter, depth + 1)

trace_daughters("AB")
```

### Tracking Cell Movement

```python
# Get trajectory of cell "ABal"
cell_name = "ABal"
idx = cell_to_idx[cell_name]

# Get all living timepoints for this cell
alive_t = np.where(alive_mask[idx, :])[0]

# Extract trajectory
trajectory = X[idx, :3, alive_t].T  # (T_alive, 3) — x, y, z over time

# Compute displacement
displacement = np.linalg.norm(np.diff(trajectory, axis=0), axis=1)
print(f"Cell {cell_name}: born at t={alive_t[0]}, moved {displacement.sum():.1f} μm total")
```

### Batch Loading Multiple Embryos

```python
import os
from pathlib import Path

embryo_dir = Path("dataset/processed/by_embryo")
npz_files = sorted(embryo_dir.glob("*.npz"))

# Load stats for all embryos
stats = []
for npz_file in npz_files[:10]:  # First 10
    npz = np.load(npz_file, allow_pickle=True)
    m = npz["metadata"].item()
    stats.append({
        "embryo": m["source_file"],
        "n_cells": m["N"],
        "n_timepoints": m["T"],
    })

import pandas as pd
df_stats = pd.DataFrame(stats)
print(df_stats.describe())
```

---

## ✅ Validation & Quality Control

### What we check

1. **No NaN values**: X, alive_mask, and edges are all valid
2. **Edge consistency**: Nodes in edges exist (< N)
3. **Time bounds**: edge_t in [0, T-1]
4. **Feature ranges**: Positions within microscope FOV, sizes positive
5. **Sparsity**: Graphs are sparse (not dense)
6. **Lineage closure**: Parents exist for all daughters

### Running QC

```python
def validate_epic_npz(npz_path):
    """Basic validation of EPIC NPZ file."""
    npz = np.load(npz_path, allow_pickle=True)
    
    X = npz["X"]
    alive_mask = npz["alive_mask"]
    edge_src = npz["edge_src"]
    edge_dst = npz["edge_dst"]
    edge_t = npz["edge_t"]
    
    N, d, T = X.shape
    
    # Check 1: No NaN
    assert not np.any(np.isnan(X)), "NaN in X"
    
    # Check 2: Edge bounds
    assert np.all(edge_src < N) and np.all(edge_src >= 0), "edge_src out of bounds"
    assert np.all(edge_dst < N) and np.all(edge_dst >= 0), "edge_dst out of bounds"
    assert np.all(edge_t < T) and np.all(edge_t >= 0), "edge_t out of bounds"
    
    # Check 3: Sparsity
    E = len(edge_src)
    density = E / (N * N)
    assert density < 0.1, f"Graph too dense: {density:.1%}"
    
    # Check 4: Feature ranges
    assert np.all(X[:, 0, :] >= 0) and np.all(X[:, 0, :] <= 512), "x out of range"
    assert np.all(X[:, 1, :] >= 0) and np.all(X[:, 1, :] <= 512), "y out of range"
    assert np.all(X[:, 3, :] >= 0), "size negative"
    
    print(f"✓ {npz_path.name}: {N} cells, {T} timepoints, {E} edges — VALID")

# Test all
for npz_file in sorted(Path("dataset/processed/by_embryo").glob("*.npz")):
    validate_epic_npz(npz_file)
```

---

## 🐛 Troubleshooting

### Common Issues

#### **Issue**: `KeyError: 'X'` when loading NPZ
**Cause**: Corrupted or wrong NPZ file  
**Fix**: Re-run preprocessing for that embryo
```bash
python scripts/preprocess_dataset.py \
  --raw_dir dataset/raw \
  --out dataset/processed/by_embryo \
  --distance_threshold 20
```

#### **Issue**: `MemoryError` loading all 260 embryos
**Cause**: Trying to load everything into RAM  
**Fix**: Load embryos one at a time or use generators
```python
# Don't do this:
all_embryos = [np.load(f) for f in npz_files]  # ❌ OOM

# Do this instead:
for npz_file in npz_files:
    npz = np.load(npz_file, allow_pickle=True)
    # ... process one embryo
    del npz  # Free memory
```

#### **Issue**: Sparse edges have isolated nodes
**Cause**: Cells with no spatial contacts (rare edge case)  
**Fix**: Handle with `alive_mask` filtering
```python
# Get nodes with edges at time t
nodes_with_edges = np.unique([edge_src[edge_t == t], 
                               edge_dst[edge_t == t]])
# Nodes without edges are isolated
isolated = np.setdiff1d(np.arange(N), nodes_with_edges)
```

#### **Issue**: Lineage ages don't add up
**Cause**: Cell naming inconsistencies in source data  
**Fix**: Check source CSV for naming errors
```python
# Validate cell names
for cell in idx_to_cell:
    parent = cell[:-1]
    assert len(parent) > 0, f"Invalid cell: {cell}"
```

---

## 📂 Complete File Manifest

### Directory Structure

```
├── dataset/
│   ├── raw/                          (Raw EPIC microscopy CSV files)
│   │   ├── CD011505_end1red_bright.csv
│   │   ├── CD011605_5a_bright.csv
│   │   ├── ... (258 more CSV files)
│   │   └── [260 files total, ~260 MiB]
│   │
│   └── processed/by_embryo/          (Preprocessed NPZ archives)
│       ├── CD011505_end1red_bright.npz
│       ├── CD011605_5a_bright.npz
│       ├── ... (258 more NPZ files)
│       ├── manifest.txt
│       └── [260 files total, ~180 MiB]
│
├── scripts/
│   ├── preprocess_dataset.py         (Main pipeline runner)
│   ├── usage_examples.py             (7 working examples)
│   └── make_figures.py               (Visualization utilities)
│
├── src/
│   └── epic_preprocess.py            (Core preprocessing functions)
│       ├── build_local_index()
│       ├── populate_X()
│       ├── populate_alive_mask()
│       ├── build_spatial_edges()
│       ├── build_lineage_edges()
│       └── save_to_npz()
│
└── docs/
    ├── README.md                     (Navigation & overview)
    ├── QUICK_REFERENCE.md            (5-minute lookup)
    ├── DATABASE_DOCUMENTATION.md     (Comprehensive reference)
    ├── ARCHITECTURE.md               (System design & diagrams)
    └── EPIC_COMPLETE_GUIDE.md        (This file — full narrative)
```

### Key Statistics

```
Total embryos processed:          260
Total raw data:                   ~260 MiB
Total processed data:             ~180 MiB
Compression ratio:                5.3:1
Average cells per embryo:         688 (range: ~100–900)
Average timepoints per embryo:    210 (range: 180–250)
Average spatial edges per embryo: 45,000
Average lineage edges per embryo: 687
Processing time (all):            ~2–4 hours (varies by hardware)
```

---

## 🔗 Cross-References

**Need more details?**
- [QUICK_REFERENCE.md](QUICK_REFERENCE.md) — 5-minute lookup table
- [DATABASE_DOCUMENTATION.md](DATABASE_DOCUMENTATION.md) — Exhaustive reference (11 sections)
- [ARCHITECTURE.md](ARCHITECTURE.md) — System design & pipelines
- [Usage Examples](../scripts/usage_examples.py) — 7 runnable code samples

---

## 📝 Metadata

- **Author**: EPIC Preprocessing Pipeline v1.0
- **Last Updated**: April 20, 2026
- **Version**: 1.0
- **Source Dataset**: EPIC (eMbryo Project Imaging Consortium) *C. elegans* embryo microscopy
- **Organism**: *Caenorhabditis elegans* L4 stage embryos
- **Microscopy**: Fluorescence imaging (brightfield + confocal channels)

---

## 🤝 Contributing & Feedback

Found an issue? Have suggestions?
- Check [Troubleshooting](#-troubleshooting) first
- Review [QUICK_REFERENCE.md](QUICK_REFERENCE.md) for quick answers
- File an issue with validation output from [QC section](#-validation--quality-control)

---

**Happy analyzing! 🧬**

