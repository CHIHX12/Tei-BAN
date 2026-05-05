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

  ---
  Full steps on a new computer (Linux / Mac)

  Step 1 — Install Miniconda (one time only, ~2 minutes)
  # Download the installer
  wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh

  # Run the installer (press Enter / yes throughout)
  bash Miniconda3-latest-Linux-x86_64.sh

  # Open a NEW terminal window (so the conda command takes effect)

  Step 2 — Navigate to the project folder
  cd /path/to/GCN-BILSTM-BAN

  Step 3 — One-click environment setup
  bash setup.sh

  ▎ Automatically: creates the drugban environment → installs PyTorch / DGL / RDKit and all other packages → verifies installation →
  ▎ prints Setup complete!
  ▎ First run takes 5–15 minutes (downloading packages)

  Step 4 — Activate the environment every time before use
  conda activate drugban

  ---
  Windows — same flow, different commands

  ┌──────────────────────┬─────────────────────────────────────────────────────────────────────────┐
  │         Step         │                                 Windows                                 │
  ├──────────────────────┼─────────────────────────────────────────────────────────────────────────┤
  │ Install Miniconda    │ Download the .exe installer, open "Anaconda Prompt" from the Start menu │
  ├──────────────────────┼─────────────────────────────────────────────────────────────────────────┤
  │ Navigate to folder   │ cd C:\Users\YourName\Desktop\GCN-BILSTM-BAN                             │
  ├──────────────────────┼─────────────────────────────────────────────────────────────────────────┤
  │ One-click setup      │ setup.bat (double-click it, or type it in Anaconda Prompt)              │
  ├──────────────────────┼─────────────────────────────────────────────────────────────────────────┤
  │ Activate environment │ conda activate drugban                                                  │
  └──────────────────────┴─────────────────────────────────────────────────────────────────────────┘

  ---
  So your understanding is correct, with one prerequisite

  Miniconda installed  ──►  cd into folder  ──►  bash setup.sh  ──►  Done

  Miniconda is free and only needs to be installed once.
  After that, bash setup.sh handles everything automatically.
