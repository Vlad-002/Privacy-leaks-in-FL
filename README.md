# Detecting Data Privacy Leaks in Federated Data Analysis with Large Language Models

Artifacts for the BSc thesis *Using Large Language Models to Detect Data Privacy Leaks in
Federated Data Analysis*, Department of Advanced Computing Sciences, Maastricht University.

**Author:** Vlad Stefan
**Year:** 2026

This repository contains the labeled corpus of federated medical workflows, the two prompts
used to build and audit it, and the spreadsheet recording every label, audit result and
evaluation reported in the paper.

---

## What this is

Federated learning lets institutions train a shared model without exchanging raw records,
but the updates exchanged during training still leak information about the underlying data.
A data owner asked to run an unfamiliar analysis script has to decide whether executing it
is safe — normally a decision that requires reading the code.

This corpus supports evaluating whether a large language model can make that judgment and
communicate it. It contains 61 federated medical workflows: 20 benign baselines and 41
variants, each carrying one camouflaged privacy attack. Every variant is hosted by a
baseline from the same corpus, so a baseline and any of its variants differ *only* in the
presence of the attack.

| Class | Count | What the inserted mechanism captures |
|---|---:|---|
| Baseline | 20 | Nothing — no attack inserted |
| Feature inference | 15 | A quantity computed from a small batch in one forward or backward pass |
| Membership inference | 13 | Per-record measures of how well the model fits an identified record |
| Property inference | 13 | A statistic aggregated over a client's records |
| **Total** | **61** | |

---

## Layout

```
.
├── README.md
├── LICENSE
├── FL Dataset.xlsx        # labels, audit results, evaluation
├── prompts/
│   ├── attack_insertion_prompt.txt
│   └── detection_prompt.txt
└── FL workflows dataset/                   # the 61 federated workflows, as .py
```

### Naming convention

Workflows are numbered by host. A file with no suffix is a baseline; a suffixed file is an
attack variant inserted into an unmodified copy of that same baseline. `kaggle_script_5.py` and
`script_5_1.py` are therefore a matched pair differing only by the attack.

| Suffix | Class |
|---|---|
| *(none)* | Baseline |
| `_1` | Feature inference |
| `_2` | Membership inference |
| `_3` | Property inference |

Not every host carries all three classes, so variant numbers are not contiguous. Hosts 3
and 4 are absent from the corpus entirely: both were withdrawn when their source datasets
ceased to be publicly available on Kaggle.

Workflows were developed and executed as Colab notebooks and are stored here as `.py` for
readability and diffing. The `Colab link` column of the spreadsheet points to the
executable copy of each.

---

## The spreadsheet

`FL_Dataset_final.xlsx` is the label and evaluation record for all 61 workflows.

| Column | Meaning |
|---|---|
| Original Kaggle URL | Source notebook the host was derived from |
| Colab link | Executable copy of the federated workflow |
| Base task / domain | Clinical task (dementia, pneumonia, breast cancer, brain tumor, …) |
| Dataset name & type | Source dataset and modality |
| Original framework | Framework of the source notebook |
| Adjusted framework | Framework after federated adaptation |
| Attack label | Ground truth: baseline, membership, feature, or property |
| Attack explanation | What the inserted mechanism captures and how it escapes |
| LLM predicted label | The auditor's classification |
| LLM explanation | Summary of the auditor's reasoning |
| Assigned reviewer | Questionnaire placement, where applicable |
| Evaluation of detections | Per-criterion content scoring of the audit report |

### Content scoring codes

The evaluation column holds an ordered list of 0/1 values scoring the audit report against
the code it describes.

**Attack variants** carry four values, in order:

1. **Captured quantity** — the report names the quantity the mechanism captures
2. **Disclosure route** — it names the route by which that quantity leaves the client
3. **Granularity** — the disclosure it attributes to the mechanism is one the mechanism supports
4. **False claim** — `1` means a false claim **was** made

For the first three, `1` means the criterion was satisfied. The fourth is inverted: a
report satisfying all four criteria therefore reads `1,1,1,0`.

**Baselines** carry three values, in order:

1. **Transmitted payload** — the report correctly identifies what the workflow does transmit
2. **Checks named** — it names each of the three attack patterns it ruled out
3. **False claim** — `1` means a false claim **was** made

A fully correct baseline report therefore reads `1,1,0`.

---

## How the corpus was built

1. **Source selection.** Public Kaggle notebooks meeting four criteria: the task is in the
   medical domain, the notebook has at least fifty community votes, the underlying data are
   static and either tabular or image-based, and the task is supervised classification.
2. **Federated adaptation.** Each script converted to a synchronous Federated Averaging
   simulation across four or five stratified clients under a fixed global random seed.
   Pretrained backbones and tree-based estimators replaced by compact CNN or MLP models, so
   that every host has a differentiable, gradient-based training surface and the model
   family is not a confounding variable. Exploratory and plotting code removed.
3. **Attack insertion.** Attack logic inserted into an unmodified copy of the baseline using
   `prompts/attack_insertion_prompt.txt`, which specifies the capture site, the scope of the
   source data, and the requirement that the captured quantity reach the aggregation payload
   rather than a discarded local variable. Each mechanism is camouflaged: no revealing
   identifiers or comments, the captured quantity labeled as routine diagnostic telemetry,
   and no side effects on training.
4. **Verification and labeling.** Every variant executed to confirm it runs without error
   and preserves its baseline's training behaviour, then labeled by hand under the same rule
   the auditor is later given.
5. **Audit.** Each workflow submitted to the auditor under `prompts/detection_prompt.txt`
   in a fresh temporary session, with no information about the corpus, the script's
   provenance, or the base rate of attacks within it.

### Models

| Role | Model | Configuration |
|---|---|---|
| Attack insertion | Gemini 3.1 Pro | Extended thinking, temporary chat |
| Detection | Claude Sonnet 5 | Maximum effort, incognito chat |

Different model families were used deliberately, so that the auditor is not recognizing the
stylistic signature of a generator from its own family. Both are specific released versions,
and the results reported in the paper describe those versions rather than the systems in
general.

---

## Reproducing an audit

1. Open a workflow from `workflows/`.
2. Start a fresh session with the auditor model, with no prior context — the decision
   procedure assumes the auditor knows nothing about the corpus or the base rate of attacks
   within it.
3. Paste `prompts/detection_prompt.txt`, then the script **as a line-numbered listing**,
   then a column-level statistical summary of its dataset.
4. The response ends with a machine-readable line encoding the predicted label, for
   comparison against the `Attack label` column of the spreadsheet.

Scripts are stored here unnumbered; numbering was applied at submission time so that the
line citations in the technical audit are locatable rather than inferred, since language
models count lines unreliably.

Model outputs are not deterministic. Each workflow in the reported results was audited once,
so individual runs may differ.

---

## Data and ethics

No patient data is collected, stored, or redistributed in this repository. Every source
dataset is publicly available and is **referenced, not mirrored** — the workflows point to
the original Kaggle datasets, which remain under their own terms of use.

The reviewer study reported in the paper involved voluntary participants who were informed
of its purpose and are identified only by professional role. No questionnaire responses are
included here.

---

## License

Original contributions in this repository — the federated adaptations, the inserted attack
mechanisms, the two prompts, and the evaluation spreadsheet — are released under the MIT
License.

The workflows are derived from third-party Kaggle notebooks, which remain subject to their
original terms. The `Original Kaggle URL` column of the spreadsheet records the provenance
of every host; attribution should follow that source when reusing an individual workflow.

---

## Citation

```bibtex
@misc{stefan2026flprivacy,
  author       = {Stefan, Vlad},
  title        = {Detecting Data Privacy Leaks in Federated Data Analysis
                  with Large Language Models: corpus, prompts and evaluation},
  year         = {2026},
  publisher    = {Zenodo},
  doi          = {10.5281/zenodo.XXXXXXX},
  url          = {https://doi.org/10.5281/zenodo.XXXXXXX}
}
```
