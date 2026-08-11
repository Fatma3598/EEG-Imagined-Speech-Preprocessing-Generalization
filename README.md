# Comparative Analysis of EEG Signal Preprocessing Impact on Generalization of a Transformer-Based EEG Imagined Speech Recognition

This repository contains the code and computational artifacts associated with a published study on the
effect of EEG signal preprocessing (frequency-band selection and ICA) on the generalization of a
transformer-based imagined-speech decoder, evaluated on the Kumar dataset under a 30-class setting.

## Research / Publication

This repository accompanies the published paper (see **Citation** below). It provides two notebooks that
serve distinct roles:

- **`01_publication_state_FROZEN.ipynb`** — the immutable **publication-state artifact**. It preserves the
  code and the saved cell outputs associated with the published experiments and remains the record of the
  published computational state. It is not modified.
- **`02_reproduction_parameterized.ipynb`** — a cleaner, **configuration-driven reproduction notebook**.
  It exposes the experimental choices (validation protocol, frequency filter, ICA, normalization) explicitly
  and routes the preprocessing, windowing, and validation pipeline automatically from a single configuration
  block. Its scientific core functions are copied verbatim from the frozen notebook.

The parameterized notebook is intended for **reproduction and exploration**; the frozen notebook remains the
**record of the published results**. The two notions kept separate throughout this README are: *published
results* (from the frozen artifact), *deterministic parity* (the parts of the pipeline verifiable against the
frozen artifact without training), and *reproduction/exploration* (re-running or varying configurations).

## Repository structure

```
repository/
├── README.md
├── notebooks/
│   ├── 01_publication_state_FROZEN.ipynb
│   └── 02_reproduction_parameterized.ipynb
└── results/
    └── table1_frozen.csv
```

`results/table1_frozen.csv` records the twelve published results together with their reported uncertainty
conventions, for reference and sanity comparison.

## Reproduction workflow

1. Open `02_reproduction_parameterized.ipynb`.
2. When exploring a configuration, modify **only** the configuration variables — `VALIDATION_PROTOCOL`,
   `FILTER_TYPE`, `USE_ICA`, and `USE_NORMALIZATION`.
3. Run the pipeline. Preprocessing, filtering, windowing, and validation-protocol/grouping selection are
   handled automatically; no downstream cell editing is required. A status banner reports whether the current
   configuration is publication-state compatible and warns on deviations.
4. Use the deterministic parity-validation section to inspect what can be verified against the frozen artifact
   (no model training is required for this section).
5. Treat `01_publication_state_FROZEN.ipynb` as the publication-state reference.

`USE_NORMALIZATION = None` resolves to the frozen protocol-specific default:

- Random Split → normalization **ON**
- GroupKFold → normalization **OFF**
- LOSO → normalization **OFF**

Setting `USE_NORMALIZATION` to `True`/`False` overrides this default and produces a non-publication-state
variant (flagged by the status banner).

## Deterministic parity (what is and is not verified)

The parameterized notebook includes a deterministic parity validator (no training) that checks, per
configuration, the pipeline properties reconstructable from the frozen artifact: filtered data, window
ordering, labels, participant IDs, sample/window IDs, grouping semantics, train/test indices where the
historical ordering is recoverable, and normalization behavior.

- **Random Split** — partition sizes reproduce (`test_size=0.1`, `random_state=i+1`). PASS.
- **LOSO** — the held-out-subject sequence reproduces the frozen output exactly
  (`[22,21,20,...,1,0]`), grouping = `participants_windows`. PASS.
- **GroupKFold** — grouping semantics (window-level, one unique group per window, 2400 groups/fold) are
  confirmed, but the exact frozen fold-0 index sequence (`[3,9,31,39,...]`) is **not** reproducible from the
  current frozen `samples_windows = arange` code, which yields `[9,19,29,...]`. Flagged **UNRESOLVED**.
  The grouping *type* (window-level) is unchanged, but because the exact fold membership is not
  reconstructable, the effect on the historical GroupKFold accuracy cannot be established from the artifact;
  the published GroupKFold numbers should be read from the frozen notebook as the record.

Overall deterministic parity status: **8/12 configurations PASS** (Random ×4, LOSO ×4); **4/12 (GroupKFold)
UNRESOLVED**. Array-level checks (filtered data, window ordering, labels, participant IDs) require the dataset.

## Frozen invariants (preserved for reproduction)

Participant exclusion `[2, 15, 18, 23, 24]`; window 32 / stride 32; `sfreq = 128`; Picard ICA
(`random_state=42`, `artifact_threshold=2.5`, `max_iter=500`, `n_components=0.9999`); NetTraST with the full
`args` configuration; batch 256; epochs 500; early-stopping patience 30; Adam default optimizer; `n_seeds=5`;
`n_splits=10`; GroupKFold `groups=samples_windows`; LOSO `groups=participants_windows` with 20%/80%
calibration; no global torch seed.

## Important reproducibility note

- Model training is **stochastic**: no global torch seed was introduced, so retrained accuracies vary.
  The frozen saved results are therefore the publication record.
- Deterministic parity checks concern only the **deterministic** parts of the pipeline.
- GroupKFold exact historical fold membership remains **UNRESOLVED**: it cannot be reconstructed from the
  frozen artifact.
- Array-level parity checks require access to the dataset.
- Reported uncertainty in the paper is not from a single formula (Random Split: 1.96·SE; GroupKFold and most
  LOSO rows: t-based CI; LOSO band-reject+ICA: raw SD). The parameterized `summarize_results` prints all
  three separately and changes no published value.
- LOSO band-reject−ICA prints 39.08% in the notebook versus 39.01% reported in the paper.

## Citation

If you use this code or results in your research, please cite:

> F. Elwasify, E. Shaaban, and R. M. Abdelmoneem, "Comparative Analysis of EEG Signal Preprocessing Impact
> on Generalization of a Transformer-Based EEG Imagined Speech Recognition," in *2026 IEEE International
> Conference on Smart Sustainable Systems for Computer and Engineering Applications (3SCEA)*, 2026,
> pp. 345–351.

```bibtex
@inproceedings{Elwasify2026EEGPreprocessing,
  author    = {Elwasify, Fatma and Shaaban, Eman and Abdelmoneem, Randa M.},
  title     = {Comparative Analysis of {EEG} Signal Preprocessing Impact on Generalization of a Transformer-Based {EEG} Imagined Speech Recognition},
  booktitle = {2026 IEEE International Conference on Smart Sustainable Systems for Computer and Engineering Applications (3SCEA)},
  year      = {2026},
  pages     = {345--351},
  publisher = {IEEE},
  isbn      = {979-8-3315-5668-6},
  doi       = {https://doi.org/10.1109/3SCEA68071.2026.11603016}
}
```

> **Note on bibliographic fields:** authors, title, venue, year, pages (345–351), and ISBN
> (979-8-3315-5668-6) are taken directly from the paper. The paper's own DOI is not printed in the
> camera-ready PDF; the `doi` field above is marked **TODO: confirm** and should be filled from the final
> IEEE Xplore record. Do not cite a DOI until verified.
