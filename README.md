# SAMT Desktop: Axillary Non-Sentinel Lymph Node Metastasis Prediction for Sentinel-Lymph-Node-Positive Breast Cancer (V1.0)

> **Reviewer note on setup time.** With the hardware in Section 1, the full
> environment can be prepared within a single working day. Downloading the
> packages, extracting or mounting them, and completing a first prediction on
> the two bundled studies takes well under an hour. The rest is buffer for the
> first-launch research-consent step and, only if you delete the supplied
> products, N4 bias-field correction, which depends on image size and CPU
> speed. The bundled studies already ship precomputed `_DCE_N4` volumes, so a
> first pass does not wait for that stage.

The application estimates the probability that a patient with a positive
sentinel lymph node (SLN) also has axillary non-sentinel lymph node (NSLN)
metastasis. It wraps SAMT, a multimodal attention-transformer model that fuses
DCE-MRI and mammography MLO with adaptive spatial decay priors and 32 clinical
plus 23 pathology features encoded from the study workbook.

> **Current ensemble definition.** Each prediction uses five checkpoints, one
> per fold, and every one of them is the AUC-selected state (`best_AUC_val`).
> `AUC` is the only ensemble metric; there is no ACC prediction path and no
> metric selector. The lossless store still carries all ten training states so
> it stays traceable to the experiment, but the accuracy-selected files
> (`best_ACC_val`) are never loaded for prediction. The original `--scan_all`
> behavior averaged those ten mixed states and is not the desktop rule.

---

## Contents

| Item | Description |
|---|---|
| `SAMT-Desktop-macOS-arm64.dmg` | macOS Apple Silicon application image |
| `SAMT-Desktop-Windows-x64.zip` | Portable Windows x64 application |
| `sample-data/` | Two de-identified example studies and one clinical workbook |
| `README-ASSETS.txt` | Short extraction notes shipped with the binaries |
| `SHA256SUMS.txt` | Download verification hashes |
| `README.md` | This guide |

**Release files:** download from the
[Releases](https://github.com/1009476063/samt-nsln-review/releases) page. All
assets are listed with exact sizes and SHA-256 values below. The current
reviewer distribution is `SAMT Desktop V1.0 · Reviewer Distribution
(build 1.8.0)`.

| Platform | File name | Size (bytes) | SHA-256 |
|---|---|---|---|
| macOS arm64 | `SAMT-Desktop-macOS-arm64.dmg` | 1,899,865,704 | `a7125598905311810c29ce9625f7072d14d995da5ea9487a52bf6f1ff8f7388f` |
| Windows x64 | `SAMT-Desktop-Windows-x64.zip` | 2,096,857,716 | `821459c19c93e327f2e0f61c90e9efd86d4e26ae587d8ed43d2f817abed45fc1` |

The same values are in `SHA256SUMS.txt`, which also covers
`README-ASSETS.txt`. Compare the full 64-character digest, not a prefix.

A current package is self-contained. Python, PyTorch, the runtime and the
lossless checkpoint store do not need to be installed separately. The store
keeps all ten checkpoint states from the training runs; five of them, the
AUC-selected state of each fold, are the only weights inference reads. The
ACC-selected states are shipped as provenance, not as a second prediction
mode.

This guide describes the v1.8.0 build in the reviewer release and the numbers
below come from it. Two earlier packages are still in circulation and do not
reproduce this table: build 1.7.1 averaged all ten AUC/ACC checkpoints in a
single run, and the first 1.8.0 build still exposed ACC as a selectable
metric. Version strings do not separate the two 1.8.0 builds, so compare the
download against the SHA-256 values below instead of reading the version
label. The reviewer release assets were replaced on 2026-11-25; a file with
the older digest belongs to the superseded build.

---

## 1. System Requirements

| Item | Requirement |
|---|---|
| macOS | macOS 11 or later, Apple Silicon (arm64). Intel Macs are not supported |
| Windows | Windows 10 or 11, 64-bit x86-64. Windows on ARM runs the x64 build through emulation |
| Memory | 8 GB minimum, 16 GB recommended |
| Free disk space | macOS: 4 GB (image plus installed app); Windows: 5 GB (archive plus extracted folder) |
| Network | Only for downloading. No account, licence server or telemetry is used |
| Display | 1280 x 800 or larger |

The Windows build needs no separate runtime. If a machine reports a missing
`VCRUNTIME140.dll`, install the Microsoft Visual C++ 2015-2022 Redistributable
(x64).

CPU is the numerical reference for the published probabilities. A CUDA or MPS
device, when selected under Runtime settings, may differ in the last bits; the
five-fold arithmetic mean and the class rule are unchanged.

---

## 2. Download and Verify

1. Open the [Releases](https://github.com/1009476063/samt-nsln-review/releases)
   page of this repository.
2. Download the asset for your platform and `SHA256SUMS.txt`.

macOS:

```bash
shasum -a 256 SAMT-Desktop-macOS-arm64.dmg
```

Windows PowerShell:

```powershell
Get-FileHash -Algorithm SHA256 ".\SAMT-Desktop-Windows-x64.zip"
```

Compare the output with the matching line in `SHA256SUMS.txt`.

### 2.1 macOS

1. Open the `.dmg` and drag `SAMT-Desktop.app` to **Applications**.
2. The build is not signed with a developer certificate and is not notarized.
   On first launch, use Finder's context menu **Open**, or allow it under
   **System Settings > Privacy & Security > Open Anyway**.
3. If macOS reports that the app is damaged because it was quarantined, run:

```bash
xattr -dr com.apple.quarantine /Applications/SAMT-Desktop.app
```

Before running that command, check the download itself. A truncated or
error-page download also opens as "damaged". The current image is 1,899,865,704
bytes and begins with the bytes `78 01 ed d0 b1 09`; a file of a few hundred
bytes, or one that `file` reports as XML or HTML, is an interrupted download.
Delete it, verify the SHA-256, and download again.

### 2.2 Windows

1. Extract the **entire** archive to a local folder, for example
   `D:\SAMT-Desktop`.
2. Run `SAMT-Desktop\SAMT-Desktop.exe`.
3. If SmartScreen appears, select **More info**, then **Run anyway**. The
   executable is not code-signed, so this prompt is expected.

Keep the whole folder, including `_internal`. Copying only the `.exe` will not
run. Do not launch from inside the ZIP and avoid network shares or synced
folders as the extraction target.

---

## 3. First Launch

The first start takes 10 to 30 seconds while the runtime and the selected
checkpoints load. The status bar reports `Models ready` when inference is
available.

The application opens behind a **Research Use Consent** gate. The statement
declares research-only use, absence of regulatory review, academic and
educational scope, and that no physician-patient relationship is created.
Declining or closing the window exits the application. Acceptance is stored
locally, and the remember option keeps it across restarts. The same statement
is available later under **Method and scope**.

The interface language defaults to **English**. The header selector, or
**Runtime settings > Interface language**, switches the entire interface to
Simplified Chinese immediately, with no restart. The language choice is
remembered; the theme selector next to it is independent.

Case paths and clinical fields start empty. Nothing is bundled into the form,
and no patient data leaves the machine.

---

## 4. Input Data

A prediction consumes four model-ready images per case, plus the clinical and
pathology row of that patient.

### 4.1 Study tree or flat collection

`StudyRoot`, `<Label>` and `<CaseID>` are placeholders; the folder names
`MRMG`, `MG`, `CP.xlsx` and the imaging suffixes follow the experiment
contract. `<Label>` is the on-disk class folder (`Positive` / `Negative` in
the bundled data) and is kept only as provenance for the result table.

Layout A is the experiment tree, with cases grouped under a label folder:

```text
StudyRoot/
├── CP.xlsx
└── MRMG/
    └── <Label>/
        └── <CaseID>/
            ├── <CaseID>_DCE.nii[.gz]
            ├── <CaseID>_DCE_GTV.nii[.gz]
            ├── <CaseID>_DCE_GTVnd1.nii[.gz]
            └── MG/
                ├── *<CaseID>_*MLO.nii[.gz]
                ├── *<CaseID>_*MLO_GTV.nii[.gz]
                └── *<CaseID>_*MLO_GTVnd1.nii[.gz]
```

Layout B is a flat collection: one workbook with the case folders beside it,
and no `MRMG/<Label>/` level. This is the usual shape of a shared case
collection exported without the training hierarchy.

```text
StudyRoot/
├── CP.xlsx
├── <CaseID>/
│   ├── <CaseID>_DCE.nii[.gz]
│   ├── <CaseID>_DCE_GTV.nii[.gz]
│   ├── <CaseID>_DCE_GTVnd1.nii[.gz]
│   └── MG/
│       ├── *<CaseID>_*MLO.nii[.gz]
│       ├── *<CaseID>_*MLO_GTV.nii[.gz]
│       └── *<CaseID>_*MLO_GTVnd1.nii[.gz]
└── <CaseID>/ …
```

In a flat collection the case ID comes from the folder name, and each case
folder must hold at least one `<CaseID>_DCE*.nii[.gz]` file for it to be
recognised. Folders named `MG` and `__MACOSX` are skipped.

The four products actually read by the model are:

```text
<CaseID>/
├── <CaseID>_DCE_N4.nii[.gz]
├── <CaseID>_DCE_decay1.nii[.gz]
└── MG/
    ├── *<CaseID>_*MLO.nii[.gz]
    └── *<CaseID>_*MLO_decay1.nii[.gz]
```

- Raw DCE alone is not enough: a missing `_DCE_N4` and both `decay1` priors
  are generated from the raw DCE, the MLO image and the paired `GTV` /
  `GTVnd1` contours.
- Existing products are used unchanged; nothing is overwritten.
- A single case may sit anywhere as a standalone `<CaseID>/` folder. Batch
  mode accepts either layout: the selected folder, `MRMG`, a label folder, a
  case folder or its `MG` folder are all accepted, and the resolver walks up
  to the nearest root that is valid in one of the two layouts.
- Batch mode picks the layout from the root, not from the selection alone. A
  root with `MRMG/` and `CP.xlsx` is a study tree; a root with `CP.xlsx`, no
  `MRMG/`, and `<CaseID>/` folders that hold DCE files is a flat collection.
  Selecting one case inside a flat collection narrows the batch to that case.
- A folder without a workbook is never promoted to a collection. Ambiguous or
  conflicting candidates are reported, never guessed.
- NIfTI `.nii` and `.nii.gz` are accepted. DICOM directories are not read.

### 4.2 Clinical workbook

One row per case, matched by the `ID` column. The required experiment fields
are defined in the desktop source's `experiment_contract.py`; category values
and units keep their original meaning.

| Accepted variation | Behavior |
|---|---|
| Reordered rows or columns; header whitespace, BOM, letter case, full-width characters | Normalized to the exact experiment names |
| `case_id`, `Case ID`, `patient_id`, `Patient ID` and the two localized case/patient ID names | Explicit aliases of `ID` |
| Existing `Sheet1` with the required fields | Preferred |
| No usable `Sheet1`, exactly one usable data sheet | Selected; notes-only sheets are allowed |
| Several candidate sheets, duplicate column mappings, conflicting rows for one ID | Blocked with an explanation |
| Unknown names, translated categories, different units, merged or multi-level headers | Not inferred; prepare the workbook to the contract |

Field coverage means non-empty cells, not verified medical semantics. CSV and
legacy binary `.xls` are not accepted for clinical import.

### 4.3 Bundled sample studies

`sample-data/` contains the two de-identified studies used for the
reproducibility checks, plus their workbook.

| Study | Folder | Laterality | cT | Lesion site | DCE volume | Disk label | NSLN reference |
|---|---|---|---|---|---|---|---|
| case1 | `sample-data/MRMG/Negative/case1` | Left | T1 | UOQ | 352 x 352 x 144 | Negative | Negative |
| case2 | `sample-data/MRMG/Positive/case2` | Right | T2 | UIQ | 320 x 320 x 144 | Positive | Positive |

Each study carries the raw DCE, a precomputed `_DCE_N4`, the DCE and MLO
`decay1` priors, `GTV` and `GTVnd1` contours for both modalities, and its row
in `CP.xlsx`. The workbook carries `ID`, the 9 clinical fields and the 10
pathology fields defined by the experiment contract, with the original
category values and units.

Both studies were resolved with the application's own loader:

| Study | Model inputs resolved | Problems |
|---|---|---|
| case1 | DCE_N4, DCE_decay1, MLO, MLO_decay1 | none |
| case2 | DCE_N4, DCE_decay1, MLO, MLO_decay1 | none |

---

## 5. Workflow

### 5.1 Single case

1. Open **Single case**.
2. Select **Browse…** and choose the case folder, for example a local copy of
   `sample-data/MRMG/Negative/case1`.
3. Confirm that all four image badges are present and that the clinical form
   is filled from the matched workbook row. Use **Import clinical table…** to
   select another `.xlsx` / `.xlsm` explicitly; the review dialog lists the
   experiment field, source column, parsed value and any warning before
   anything is written into the form.
4. Select **View images** for the axial, coronal and sagittal review, with
   contours and decay masks as display-only overlays.
5. Select **Run ensemble prediction**. Missing offline products are prepared
   in scratch storage first.
6. Read the result on **Prediction results**.

### 5.2 Batch

1. Open **Batch prediction** and select the root that contains `CP.xlsx`,
   either with `MRMG/<Label>/<CaseID>/` or with case folders directly beside
   the workbook; `sample-data/` is a study tree.
2. Select **Scan cases**. The readiness column reports every required input;
   only complete cases can be scored.
3. Select **Run batch prediction**. Progress is reported per case and per
   fold checkpoint.
4. Review the table, inspect a selected case's details, and export the CSV.

### 5.3 Preprocessing

**Preprocessing** is the explicit offline stage. Scan the study root, review
the queue and generate only the missing products:

| Stage | Product | Rule |
|---|---|---|
| N4 | `<CaseID>_DCE_N4.nii.gz` | Existing file is used exactly as supplied; otherwise N4 bias-field correction runs on the raw DCE |
| DCE decay | `<CaseID>_DCE_decay1.nii.gz` | Adaptive Gaussian decay over `GTV ∪ GTVnd1` |
| MLO decay | `MG/*<CaseID>_*MLO_decay1.nii.gz` | Adaptive linear decay over the same core region |
| Clinical | `CP_prepared.xlsx` | Fills only an empty lowest-layer ALN diameter when derivable from `GTVnd1`; the source workbook is preserved |

Both decay masks are computed from the union of `GTV` and `GTVnd1`; the core
region is 1 and values decay outward with a scale adapted to the equivalent
radius `r = sqrt(area / pi)` of the mask. The derived table becomes stale when
its source workbook or the associated contours change; the application reports
this instead of silently reusing it.

### 5.4 Reading the result

The results page shows six-decimal probabilities, the class, the five-fold
vote split, the margin from `0.5`, the usable checkpoint count and the source
of each row. Every row was produced by the five AUC-selected fold
checkpoints, so there is no metric column to compare.

| Displayed probability of NSLN-positive | Ensemble class |
|---|---|
| `P(positive) > P(negative)` | Positive |
| `P(positive) <= P(negative)` | Negative |

Each of the five selected checkpoints, one per fold, returns a two-class
softmax; the two displayed probabilities are the arithmetic means across those
five checkpoints, and the higher mean determines the class. The
positive/negative vote count describes disagreement between folds and is not a
majority-vote rule. Values stay at six decimals so a margin below `0.01`
remains visible.

---

## 6. Walkthrough with the bundled studies

Copy `sample-data/` to a local disk first, for example `~/SAMT-Data` or
`D:\SAMT-Data`. This avoids file locking on network shares and keeps the
repository copy pristine.

1. Launch the application and accept the research-use statement.
2. Open **Batch prediction**, select the copied `sample-data` root and select
   **Scan cases**. Two runnable cases are reported.
3. Select **Run batch prediction**. Because both studies ship `_DCE_N4`,
   inference starts without the N4 stage.
4. Open **Prediction results** and compare with the reference values below.
5. Export the CSV to keep the ensemble and per-selected-checkpoint
   probabilities.

Reference values for a current CPU build on the supplied products. Each case
uses the AUC-selected checkpoint of each fold, one per fold, in fold order 1
to 5:

| Study | `P(NSLN negative)` | `P(NSLN positive)` | Ensemble class |
|---|---:|---:|---|
| case1 | 0.5527292549610138 | 0.44727074503898623 | Negative |
| case2 | 0.2268205642700195 | 0.7731794357299805 | Positive |

Case1 is a near-threshold Negative result: `0.447271` sits about `0.0527`
below the `0.5` rule. That is expected. The earlier ten-checkpoint ensemble,
which averaged five AUC-selected with five ACC-selected states, reached
`0.508991` and classed case1 Positive. Dropping the ACC states is the whole
change; preprocessing, the network, the softmax and the `0.5` rule are
untouched. Case2 is Positive either way. The single-case page reaches the
same values.

Build 1.8.0 no longer offers an ACC run, so there is no second reference row
to compare against. If **Runtime settings** still shows a metric dropdown,
the installed package is the superseded 2026-11-24 build.

If you delete `_DCE_N4`, `_DCE_decay1` and the MLO `decay1` files from a copy
of the studies and run **Preprocessing** instead, the inputs are regenerated
before the same five-fold ensemble is applied. Regenerating N4 changes the
input tensor, so those results are not a model-drift comparison and are not
interchangeable with the table above.

---

## 7. Image Viewer

The viewer shows DCE (raw or N4), MLO, both `decay1` priors, the `GTV` and
`GTVnd1` masks and any other identifiable NIfTI in the case. Physical
metadata drives the axial, coronal and sagittal planes and the anatomical
markers; overlays align to the base physical grid. Display transforms never
alter model inputs.

| Action | Control |
|---|---|
| Browse slices | Mouse wheel, slice slider, or previous/next buttons |
| Zoom | Vertical left-button drag, or `+` / `-` |
| Pan | Right- or middle-button drag |
| Reset | Fit to window |
| Switch plane / source | Plane tabs and the image-source selector |

Contours and decay masks are overlays only. Editing labels is not part of this
distribution; delineate in your own tooling (for example 3D Slicer or
ITK-SNAP) and place the resulting NIfTI masks in the case folder.

---

## 8. Settings and Data Locations

| Section | Setting | Purpose |
|---|---|---|
| Interface | Interface language | English or Simplified Chinese, applied immediately |
| Model and compute | Checkpoint directory | Re-detect or point at another lossless store, or at fold checkpoint files; only `best_AUC_val` states are selected |
| Model and compute | Ensemble metric | Read-only `AUC`. There is no dropdown and no reload step; an `ACC` value saved by an older build has no effect and is dropped when settings are applied |
| Model and compute | Compute backend | CPU (numerical reference), CUDA or MPS where available |
| Model and compute | Low memory mode | Load the five AUC-selected checkpoints sequentially; same ensemble rule |
| Workflow | Open results after prediction | Navigation only |
| Method and scope | Research use statement | Full scope declaration and version |

Settings and the rotating log are stored per user:

| Platform | Location |
|---|---|
| macOS | `~/Library/Application Support/SAMT/` |
| Windows | `%APPDATA%\SAMT\` |

`settings.json` holds preferences and the consent record; `samt.log` is the
rotating application log. A legacy `ensemble_metric` key is no longer read,
and applying settings removes it, so an old `ACC` selection cannot change
which weights load. Report the version, OS and architecture, the steps that
failed and a redacted log. Do not upload patient images or full clinical
tables.

---

## 9. Method and Fidelity

The desktop shell wraps the experiment inference path without changing
preprocessing or model structure; the ensemble is fixed to the AUC-selected
weights:

1. DCE-MRI and MLO are concatenated with their `decay1` spatial priors, then
   z-score normalized and resized to `144 x 320 x 320` and `640 x 512`.
2. `CP.xlsx` is encoded into 32 clinical and 23 pathology features with the
   original category order, missing-value handling and normalization constants.
3. Two ResNet18 backbones and two MLPs map the four inputs to 256-dimensional
   tokens; a CLS Transformer fuses them into two-class logits.
4. Five checkpoints, one per fold, are the `best_AUC_val` states. Each one
   produces a softmax, and the per-class arithmetic mean over those five gives
   the reported result.

The original `infer.py --scan_all` recursively collected every `.pth` file and
therefore averaged five AUC-selected with five ACC-selected checkpoints. The
desktop build keeps a single path: it requires exactly one AUC checkpoint for
each of the five folds, rejects a missing or duplicate fold, and never treats
an `best_ACC_val` file as a prediction candidate. Selecting the accuracy
states is not a supported mode, so no run mixes the two sets.

There is no quantization, pruning, conversion, sample-specific threshold or
majority vote. The lossless store is built from all ten training states and
keeps each unique tensor once, so the package stays byte-traceable to the
experiment; the checkpoint builder accepts five or ten inputs, and inference
materialises only the five AUC ones.

The desktop source is not part of this repository. This distribution exists so
that a reviewer can reproduce the reported predictions on the two supplied
studies and inspect every intermediate product.

---

## 10. Intended Use and Limitations

This tool is intended for academic and research use only. It is not a medical
device, has not been cleared or approved by any regulatory authority, and must
not be used as a substitute for clinical judgement. Predictions are model
estimates from a breast cancer cohort and may not apply to an individual
patient. All output should be interpreted by a licensed clinician.

Two bundled cases demonstrate that the software runs, that the packaged
weights match the experiment and that the input contract is met. They are not
an accuracy evaluation, a calibration study or evidence of clinical utility.

The studies under `sample-data/` are de-identified research examples shared for
software verification and method reproduction. They carry a study identifier,
image volumes, contours and the clinical/pathology panel only. They contain no
name, hospital number, medical record number, date of birth or acquisition
date, and they must not be used for clinical decisions or re-identification.

Native Windows acceptance on physical hardware, GPU parity and long-queue
stress testing were not part of this packaging run and remain separate work.
The CPU reference probabilities above were reproduced on macOS arm64.

---

## 11. Troubleshooting

| Symptom | Cause and resolution |
|---|---|
| macOS refuses to open the app | Unsigned, unnotarized build. Use **Open** from the Finder context menu, or **Open Anyway** in Privacy & Security |
| macOS reports the app is damaged | Quarantine attribute. Run the `xattr` command in Section 2.1 |
| `Windows protected your PC` | Unsigned executable. **More info**, then **Run anyway** |
| The application closes immediately | The archive was not fully extracted, or `_internal` is missing. Re-extract the complete package |
| `VCRUNTIME140.dll was not found` | Install the Microsoft Visual C++ 2015-2022 Redistributable (x64) |
| A case is not listed in batch mode | The root has no `CP.xlsx`, or a flat case folder holds no `<CaseID>_DCE*.nii[.gz]` file. In a study tree the folder must also sit at `MRMG/<Label>/<CaseID>` |
| The batch list is empty after selecting one case | In a flat collection the selected case folder needs a DCE file named after that folder, and the root workbook must sit in the parent directory |
| Settings still show a metric selector | That is the superseded 2026-11-24 build, which also reports version 1.8.0. Compare the file hash with Section 1 rather than the version label |
| A case is reported incomplete | One of the four model-ready inputs is missing, empty, or has mismatched array dimensions. Run **Preprocessing** and read the reported reason |
| Clinical row not matched | The `ID` cell does not match the folder name, or several workbook rows claim it. Make the identifier unique |
| Workbook rejected | Unknown field names, several candidate sheets or duplicate mappings. See Section 4.2; nothing is guessed |
| Derived table reported stale | The source workbook or contours changed after `CP_prepared.xlsx` was written. Regenerate it in **Preprocessing** |
| N4 correction takes several minutes | Expected on a first pass for a large volume. The result is reused afterwards |

---

## 12. Distribution

This repository distributes compiled binaries and two research examples for
peer review. Source code is not included, and no licence to modify,
redistribute, sublicense or incorporate the software into another product is
granted. All rights remain with the authors.

For questions about the manuscript, the model, or the sample studies, contact
the corresponding author through the journal's editorial system.
