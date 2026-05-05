    USER INPUT
    ──────────
    Drug SMILES string          Protein amino-acid sequence
    "CC(=O)Oc1ccccc1C(=O)O"    "MENFQKVEKIGEGTYGVV..."
          │                              │
          ▼                              ▼
    ┌─────────────┐              ┌──────────────────┐
    │  RDKit      │              │  Input cleaner   │
    │  SMILES →   │              │  strip digits,   │
    │  Mol graph  │              │  spaces, FASTA   │
    └──────┬──────┘              └────────┬─────────┘
           │                             │
           ▼                             ▼
    ┌─────────────┐              ┌──────────────────┐
    │  DGL graph  │              │  Token encoding  │
    │  node=atom  │              │  (integer AA     │
    │  edge=bond  │              │   indices)       │
    └──────┬──────┘              └────────┬─────────┘
           │                             │
           ▼                             ▼
    ┌────────────────┐        ┌─────────────────────┐
    │  GCN Encoder   │        │  BiLSTM Encoder      │
    │  (3 layers)    │        │  (2-layer bidirec-   │
    │  drug graph →  │        │   tional LSTM)       │
    │  drug vector   │        │  seq → context vecs  │
    └───────┬────────┘        └──────────┬──────────┘
            │                            │
            └──────────┬─────────────────┘
                       ▼
             ┌──────────────────┐
             │  BAN (Bilinear   │
             │  Attention       │
             │  Network)        │
             │                  │
             │  drug vec  ──┐   │
             │  prot vecs ──┼─► │  attention map
             │              │   │  (which residues
             │              └─► │   the drug "looks at")
             └────────┬─────────┘
                      │
                      ▼
             ┌──────────────────┐
             │  MLP Classifier  │
             │  → sigmoid →     │
             │  binding_prob    │
             └────────┬─────────┘
                      │
          ┌───────────┴───────────┐
          ▼                       ▼
    prob ≥ threshold          prob < threshold
    (0.1511 for BiLSTM)
          │                       │
      ►  BIND                 ►  NO BIND
      ►  confidence level     ►  confidence level

    ─────────────────────────────────────────────────────────────

    ENTRY POINTS

    ┌─────────────────────────────────────────────────────────┐
    │  predict_simple.py   — interactive, 1 drug + 1 protein  │
    │  predict_batch.py    — file/folder input, auto-mode     │
    │    ├─ 1:N  (1 drug   vs many proteins)                  │
    │    ├─ N:1  (many drugs vs 1 protein)                    │
    │    └─ N:M  (all combinations)                           │
    │  screening/screen.py — HTS with heatmap + binding sites │
    │  predict.py          — legacy CSV / screening flags     │
    └─────────────────────────────────────────────────────────┘

    SETUP

    ┌──────────────────────────────────────────────────────────┐
    │  setup.sh  (Linux/Mac)                                   │
    │  setup.bat (Windows)                                     │
    │    └─► conda env create -f environment.yml               │
    │        (Python 3.10, PyTorch 2.2.1, DGL 2.1+cu121)      │
    └──────────────────────────────────────────────────────────┘

    MODEL FILES

    result/DrugBAN_BiLSTM/best_model_epoch_94.pth   ← default
    result/DrugBAN/best_model_epoch_90.pth           ← CNN baseline

─────────────────────────────────────────────────────────────
  Demo files are ready — examples/ folder

  examples/
  ├── drugs.txt          ← 5 drugs (caffeine, aspirin, ibuprofen, metformin, tamoxifen)
  ├── proteins.fasta     ← 4 proteins (CDK2, ESR1, DHFR, A2AR)
  ├── one_drug.txt       ← single drug (caffeine)
  └── one_protein.txt    ← single protein (ESR1 estrogen receptor)

  ---
  Ready-to-run commands — copy and paste as-is

  Demo 1 — 1 drug vs 4 proteins (Mode 1:N)
  python predict_batch.py --ligands examples/one_drug.txt --receptors examples/proteins.fasta --output
  examples/demo1_1drug_vs_4proteins.csv

  Demo 2 — 5 drugs vs 1 protein (Mode N:1 — virtual screening)
  python predict_batch.py --ligands examples/drugs.txt --receptors examples/one_protein.txt --output examples/demo2_5drugs_vs_ESR1.csv

  Demo 3 — 5 drugs vs 4 proteins (Mode N:M — all 20 combinations)
  python predict_batch.py --ligands examples/drugs.txt --receptors examples/proteins.fasta --output
  examples/demo3_5drugs_vs_4proteins.csv

  All three commands have been tested and confirmed working.
  Results are saved as CSV files inside the examples/ folder — open directly in Excel.
