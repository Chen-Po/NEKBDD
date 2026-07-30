# Reproducibility — self-contained notebooks

Two notebooks reproduce the method end-to-end. They define every core function inline.

- **`NP_ENS_KBDD_Simulation.ipynb`** — the *P* = 10 simulation. Labelling is **exhaustive over all
  *P*! permutations**, so it returns the global optimum and no stochastic search is involved.
- **`NP_ENS_KBDD_Application.ipynb`** — the breast JAK-STAT application (*P* = 32). Labelling uses
  the convergence-based permutation search described below.


## Pipeline

1. 52 KEGG reference networks → mean sparsity + power-law α.
2. `M = 100` degree sequences → `net_gen` (Erdős–Gallai enumeration) → potential networks.
3. Network properties → **Deviation** ranking.
4. Expression data → |partial correlation| confidence matrix `C`.
5. Top 1% non-isomorphic structures.
6. Label each structure (permutation search maximising `S = Σ A∘C`).
7. Ensemble edge proportion → sparsity-matched threshold `t*` → final network.

## The permutation search (application notebook)

Starting from swap size 2, the swap size is **increased by one after `UPDATE_COUNT = 2000`
consecutive non-improving proposals**, and the search **stops after `PATIENCE_T = 6000` consecutive
non-improving proposals**. `PERMUTE_TIMES = 100000` is only a safety cap and is not normally reached.
Convergence traces for the four highest-ranked structures are plotted in the last cell.


```
data/
├── simulation/                    # 100 ground-truth networks + 100 expression matrices
│   ├── true_network{0..99}.csv
│   └── label_gene_exp{0..99}.csv
└── application/
    ├── JAKSTATdata.csv            # 32 JAK-STAT genes x 35 samples
    └── KEGG signaling pathway/    # 52 reference adjacency matrices (pandas binary
                                   # DataFrame format; loaded by the notebook)
```

**`JAKSTATdata.csv` provenance.** It is the 32 JAK-STAT-pathway gene subset of GEO series
[**GSE69240**](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE69240) — *"A Molecular Portrait
Of High-Grade Ductal Carcinoma In Situ (DCIS) [RNA-seq]"*, Illumina HiSeq 2000 (GPL11154), public
since 25 July 2015, PubMed [26249178](https://pubmed.ncbi.nlm.nih.gov/26249178/). Columns `N1`–`N10`
are the 10 normal breast organoids and `T1`–`T25` the 25 pure HG-DCIS samples; the application uses
the 25 tumour samples. Values are identical to the corresponding rows of the GEO series matrix.

## Requirements

See `requirements.txt` (`pip install -r requirements.txt`). Pinned to the environment the notebooks
were last run with: Python 3.9.7, `powerlaw==1.5`, `numpy==1.20.3`, `pandas==1.4.4`,
`networkx==3.2.1`, `scipy==1.7.3`, `pingouin==0.5.4`, `matplotlib==3.4.3`, `joblib==1.5.1`.


## Reproducibility notes

- **Seeding.** Labelling is seeded per structure (`random.seed(SEED + k)`, `SEED = 20211208`). The
  published run seeded numpy (so the candidate pool is reproducible) but not Python's `random`; in
  practice the labelling was still stable, but seeding it makes the result exactly reproducible.
- **Runtime** (Apple M1 Pro, 8 cores, idle machine): simulation, one construction on a single core
  10.1 s and all 100 replications 149 s; breast application about 11 min on 8 cores.
