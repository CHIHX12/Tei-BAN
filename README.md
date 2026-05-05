  ╔══════════════════════════════════════════════════════════════════════╗
  ║              Tei-BAN  —  System Architecture                 ║
  ╚══════════════════════════════════════════════════════════════════════╝

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
