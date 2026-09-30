# CHEERS: PrimeKG Graph Composition + R-GCN for Drug–Drug Link Prediction

**Team CHEERS**
**Pusan National University — Computer Science Capstone Project**

> **Research Title:** *Effect of Biomedical Knowledge Graph Composition on R-GCN-Based Drug–Drug Interaction Prediction*

---

## Project Overview

CHEERS is a research-oriented prototype that investigates how the composition of a biomedical knowledge graph affects **drug–drug link prediction using a Relational Graph Convolutional Network (R-GCN)**.

The main research question is:

> **How does biomedical knowledge graph composition affect R-GCN-based drug–drug link prediction?**

The final controlled experiment compares four graph compositions derived from **PrimeKG** while keeping the following factors fixed:

* Drug–drug interaction split
* R-GCN architecture
* Decoder
* Training procedure
* Negative sampling strategy
* Filtered ranking evaluation

The project also provides a lightweight web application that uses an independently verified final model export for demonstration and exploration.

CHEERS consists of three main scopes:

1. **Final graduation research experiment**
2. **Earlier project development and research stages**
3. **Lightweight demonstration and web application**

The system is intended for **academic and research use only**. It is not a clinical decision-support system and must not be used for prescribing, discontinuing, or modifying medication.

---

# 1. Project Background

## 1.1. Global and Domestic Market Trends and Problems

Drug–drug interactions (DDIs) are an important concern in healthcare because patients may take multiple medications simultaneously. Identifying potentially relevant drug pairs requires integrating information from different biomedical domains, including drugs, genes, proteins, diseases, indications, contraindications, and other biomedical relationships.

Traditional DDI information is often presented as individual drug-pair records. However, biomedical knowledge is naturally heterogeneous and interconnected. A drug may be related to multiple genes, proteins, diseases, indications, and other drugs. Therefore, a knowledge graph can provide a broader representation of the relationships surrounding a drug pair.

CHEERS investigates whether incorporating such heterogeneous biomedical context into a graph neural network can improve drug–drug link prediction compared with using only direct drug–drug relationships.

The project uses **PrimeKG** as the primary biomedical knowledge graph source.

The target relation is PrimeKG's:

* Internal relation: `drug_drug`
* Display relation: **synergistic interaction**

The project does **not** attempt to predict every clinically relevant drug–drug interaction. Instead, it studies link prediction for the specific target relation represented in PrimeKG.

---

## 1.2. Necessity and Expected Benefits

The project is motivated by three main needs.

### 1. Biomedical knowledge integration

Biomedical relationships are heterogeneous and interconnected. Modeling only direct drug–drug edges may ignore useful contextual information.

### 2. Controlled evaluation of graph composition

Rather than simply adding as many relationships as possible, CHEERS explicitly compares different graph compositions:

* DDI-only graph
* DDI + Drug–Gene/Protein relationships
* DDI + Drug–Disease relationships
* Combined graph

This allows the effect of graph composition to be examined under a controlled experimental setting.

### 3. Research-oriented interactive exploration

The project also provides a lightweight application through which users can:

* Search drugs
* Generate ranked drug-pair predictions
* Explore graph context
* Inspect known-positive relationships
* Retrieve external FDA/PubMed evidence
* Examine experiment and verification information

The expected benefit is not clinical decision-making, but a reproducible environment for studying **biomedical knowledge graph composition and graph-based link prediction**.

---

# 2. Development Goals

## 2.1. Objectives and Detailed Goals

The primary objective is to evaluate the effect of biomedical graph composition on R-GCN-based drug–drug link prediction.

The final research pipeline is:

```text
PrimeKG
   ↓
Canonicalize drug–drug pairs
   ↓
Create a fixed DDI train/validation/test split
   ↓
Construct controlled graph variants G0–G3
   ↓
Train the same R-GCN architecture
   ↓
Evaluate using filtered link prediction
   ↓
Compare MRR and Hits@K across seeds
   ↓
Analyze relation-family contributions
```

The detailed goals are:

1. Construct a canonical drug–drug interaction dataset from PrimeKG.
2. Remove symmetric duplicate DDI pairs.
3. Create one fixed train/validation/test split shared by all graph variants.
4. Prevent validation/test DDI edges from being used in message-passing adjacency.
5. Construct four controlled graph compositions.
6. Train the same R-GCN architecture under the same optimization settings.
7. Evaluate using filtered ranking metrics.
8. Repeat experiments across multiple random seeds.
9. Perform relation-family ablation experiments.
10. Verify the final model and exported runtime independently.
11. Provide a lightweight web application for research demonstration.

---

## 2.2. Differentiation from Existing Services

CHEERS differs from a conventional DDI lookup service in several ways.

### Controlled graph-composition experiment

The main contribution is not simply building a DDI predictor. The project explicitly compares different biomedical graph compositions while keeping the prediction task and model architecture fixed.

### Heterogeneous biomedical context

The final graph incorporates multiple biomedical relation families, including:

* Drug–Drug
* Drug–Gene/Protein
* Drug–Disease
* Indication
* Contraindication
* Off-label use
* Target
* Enzyme
* Transporter
* Carrier

### Reproducible evaluation

All graph variants use the same DDI split and the same R-GCN architecture, allowing the graph composition itself to be examined as the main experimental variable.

### Lightweight inference

The final demonstration runtime exports the verified model representation into a NumPy-based runtime. The local application therefore does not require PyTorch, PyTorch Geometric, CUDA, or a GPU.

### External evidence separated from model prediction

FDA and PubMed evidence is retrieved independently from the model score. External evidence is not treated as model output and is not generated by the prediction model itself.

---

## 2.3. Social Value Implementation Plan

CHEERS is designed as a research and educational system that can help users understand how biomedical relationships can be represented and analyzed computationally.

The project emphasizes:

* Transparent model limitations
* Separation between prediction and external evidence
* Reproducible experimental procedures
* Responsible presentation of biomedical information
* Avoidance of unsupported clinical claims

The application explicitly communicates that:

* A model score is not a clinical probability.
* An unobserved pair is not necessarily a confirmed negative.
* Graph relationships are descriptive rather than causal.
* A predicted link is not equivalent to a clinically confirmed interaction.
* Clinical decisions require qualified healthcare professionals and authoritative medical information.

---

# 3. System Design

## 3.1. System Architecture

The overall CHEERS architecture can be summarized as follows:

```text
                         ┌─────────────────────┐
                         │       PrimeKG       │
                         └──────────┬──────────┘
                                    │
                                    ▼
                       ┌────────────────────────┐
                       │ DDI Canonicalization   │
                       │ and Fixed Data Split   │
                       └──────────┬─────────────┘
                                  │
                    ┌─────────────┼─────────────┐
                    │             │             │
                    ▼             ▼             ▼
                   G0            G1            G2
                    │             │             │
                    └─────────────┼─────────────┘
                                  ▼
                                 G3
                                  │
                                  ▼
                         ┌─────────────────┐
                         │      R-GCN      │
                         └────────┬────────┘
                                  │
                                  ▼
                    ┌──────────────────────────┐
                    │ Filtered Link Prediction │
                    └────────────┬─────────────┘
                                 │
                     ┌───────────┴───────────┐
                     ▼                       ▼
              Research Results        Runtime Export
                                             │
                                             ▼
                                    ┌─────────────────┐
                                    │ FastAPI Backend │
                                    └────────┬────────┘
                                             │
                                  ┌──────────┴──────────┐
                                  ▼                     ▼
                             Web Frontend        FDA / PubMed
```

---

## 3.2. Technologies Used

### Research and Machine Learning

* Python
* PyTorch
* PyTorch Geometric
* NumPy
* R-GCN / `RGCNConv`
* Jupyter Notebook

### Graph and Biomedical Data

* PrimeKG
* DrugBank identifiers
* Biomedical relation metadata

### Backend

* FastAPI
* Uvicorn
* Pydantic
* Python standard library
* NumPy

### Frontend

* HTML
* CSS
* Vanilla JavaScript

The lightweight local application does not require:

* React
* Node.js
* npm
* PyTorch
* PyTorch Geometric
* CUDA
* GPU

### External Evidence

* openFDA
* PubMed

### Development and Version Control

* Git
* GitHub

---

# 4. Development Results

## 4.1. Overall System Flow

The complete development and inference process is organized into the following stages:

```text
PrimeKG
  ↓
Drug–Drug Relation Extraction
  ↓
Canonical DDI Pair Construction
  ↓
Fixed Train / Validation / Test Split
  ↓
Graph Variant Construction
  ├── G0: DDI only
  ├── G1: DDI + Drug-Gene/Protein
  ├── G2: DDI + Drug-Disease
  └── G3: Combined
  ↓
R-GCN Training
  ↓
Multi-seed Evaluation
  ↓
Relation-level Ablation
  ↓
Final Verification
  ↓
Lightweight Runtime Export
  ↓
FastAPI + Web Application
```

---

## 4.2. Functional Description and Major Feature Specifications

### 4.2.1. PrimeKG DDI Canonicalization

The original PrimeKG DDI relation contains:

* **2,672,628 directed rows**
* **1,336,314 unique undirected drug pairs**
* **1,336,314 reverse duplicates**
* **0 self-loops**

The project canonicalizes the DDI relation into unique undirected drug pairs before constructing the fixed evaluation split.

---

### 4.2.2. Fixed DDI Split

The final DDI split contains:

| Split      | Number of pairs |
| ---------- | --------------: |
| Train      |       1,069,080 |
| Validation |         133,620 |
| Test       |         133,614 |
| **Total**  |   **1,336,314** |

The same split is used for G0, G1, G2, and G3.

The following controls are applied:

* No train/validation/test overlap
* Symmetric duplicates removed
* Validation and test DDI pairs excluded from message-passing adjacency
* Same split used across all graph variants
* Transductive evaluation setting

---

### 4.2.3. Graph Composition

Four controlled graph variants are constructed.

| Graph  | Composition             | Directed Edges | Active Relations |
| ------ | ----------------------- | -------------: | ---------------: |
| **G0** | DDI only                |      2,138,160 |                1 |
| **G1** | DDI + Drug–Gene/Protein |      2,189,466 |                9 |
| **G2** | DDI + Drug–Disease      |      2,223,422 |                7 |
| **G3** | Combined graph          |      2,274,728 |               15 |

#### G1: Drug–Gene/Protein Context

Forward relation counts:

| Relation    |      Count |
| ----------- | ---------: |
| Target      |     16,380 |
| Enzyme      |      5,317 |
| Transporter |      3,092 |
| Carrier     |        864 |
| **Total**   | **25,653** |

#### G2: Drug–Disease Context

Forward relation counts:

| Relation         |      Count |
| ---------------- | ---------: |
| Indication       |      9,388 |
| Contraindication |     30,675 |
| Off-label use    |      2,568 |
| **Total**        | **42,631** |

#### G3: Combined Context

The combined graph contains **68,284 forward support edges** from the Drug–Gene/Protein and Drug–Disease relation families.

---

### 4.2.4. Global Relation Mapping

The model uses a shared global relation mapping:

| ID | Relation               |
| -: | ---------------------- |
|  0 | `drug_drug`            |
|  1 | `target`               |
|  2 | `rev_target`           |
|  3 | `enzyme`               |
|  4 | `rev_enzyme`           |
|  5 | `transporter`          |
|  6 | `rev_transporter`      |
|  7 | `carrier`              |
|  8 | `rev_carrier`          |
|  9 | `indication`           |
| 10 | `rev_indication`       |
| 11 | `contraindication`     |
| 12 | `rev_contraindication` |
| 13 | `off-label use`        |
| 14 | `rev_off-label use`    |

---

### 4.2.5. Graph Statistics

The shared graph contains:

* **13,094 nodes**
* **4,278 candidate drug nodes**
* **15 relation types**
* **4,278 × 4,278 known-positive mask**
* **2,672,628 symmetric known-positive entries**

The known-positive mask is used during filtered evaluation and inference to distinguish already observed DDI pairs from remaining candidates.

---

### 4.2.6. R-GCN Model

The final model uses a two-layer Relational Graph Convolutional Network.

| Parameter                |         Value |
| ------------------------ | ------------: |
| GNN architecture         | 2-layer R-GCN |
| Layer                    |    `RGCNConv` |
| Embedding dimension      |           128 |
| Hidden dimension         |           128 |
| Dropout                  |           0.2 |
| Learning rate            |         0.001 |
| Weight decay             |          1e-5 |
| Maximum epochs           |           500 |
| Early stopping patience  |            10 |
| Positive samples / epoch |       100,000 |
| Negative sampling ratio  |           1:1 |
| Parameters               |     2,200,704 |

The decoder follows a symmetric DistMult-style formulation.

---

### 4.2.7. Negative Sampling

Negative samples are sampled from unobserved drug pairs.

Importantly:

> **An unobserved pair is not treated as a confirmed negative interaction.**

The negative samples represent currently unobserved candidate pairs for the link-prediction task.

---

### 4.2.8. Training and Model Selection

All graph variants use:

* The same DDI split
* The same R-GCN architecture
* The same optimization settings
* The same negative sampling strategy
* The same evaluation protocol

Validation BCE is used for checkpoint selection.

The test set is not used for model selection.

The final G3 seed-44 checkpoint is:

```text
checkpoints/rgcn_multiseed/G3_seed44_best.pt
```

The best epoch for this checkpoint is **epoch 499**.

---

### 4.2.9. Filtered Link Prediction Evaluation

The held-out test set contains:

* 133,614 DDI pairs
* 2 directions per pair
* 267,228 ranking queries
* 4,278 candidate drugs per query

The evaluation metrics are:

* Mean Reciprocal Rank (MRR)
* Hits@1
* Hits@5
* Hits@10

Filtered ranking removes known-positive drug pairs from candidate rankings when appropriate, preventing already observed interactions from artificially affecting the ranking of held-out pairs.

The latest multi-seed graph-composition results are maintained in:

```text
results/live_5seed/final_experiment_summary.json
```

and the corresponding result artifacts under:

```text
results/
```

The latest five-seed results should be treated as the authoritative experimental summary for the final version of this repository.

---

### 4.2.10. Relation-level Ablation

A separate relation-family ablation experiment evaluates the contribution of individual biomedical relation families.

The relation-level experiment includes:

* A1: Target
* A2: Enzyme
* A3: Transporter
* A4: Carrier
* A5: Indication
* A6: Contraindication
* A7: Off-label use

The baseline and ablation results are summarized using MRR differences.

The current project includes the corresponding analysis artifacts:

```text
figures/relation_ablation_delta_mrr_3seed.png
```

and result files under:

```text
results/
```

The ablation experiments indicate that the contribution of heterogeneous context depends on the relation family. These results are interpreted as associations observed under the experimental setting rather than causal effects.

---

### 4.2.11. Final Verification

The final project includes multiple independent verification procedures.

#### 1. Target-edge leakage check

Validation and test DDI target edges are checked to ensure that they do not leak into message-passing adjacency.

Result:

```text
Validation leakage: 0
Test leakage: 0
```

#### 2. Test-pair ranking sanity check

A subset of 1,000 test pairs is evaluated in both directions:

* 2,000 ranking queries
* MRR: 0.502226
* Hits@1: 0.4525
* Hits@5: 0.5465
* Hits@10: 0.5870
* Median rank: 2

#### 3. Positive vs. unobserved score sanity check

For the verification sample:

| Statistic    |   Positive | Unobserved |
| ------------ | ---------: | ---------: |
| Mean score   | 161.370407 |  -2.773926 |
| Median score |   6.497307 |  -1.558111 |

Additional results:

* Pairwise win rate: **97.54%**
* ROC-AUC: **0.9737**

These statistics are verification results for the model output and should not be interpreted as clinical performance measures.

#### 4. Metadata resolution

All:

* 13,094 graph nodes
* 4,278 candidate drugs

resolve to available metadata.

#### 5. Checkpoint reproducibility

The final G3 seed-44 checkpoint reproduces:

* MRR: 0.540359
* Hits@1: 0.490656
* Hits@5: 0.588273
* Hits@10: 0.626229

#### 6. Graph edge-count verification

The expected graph edge counts are checked against the generated tensors.

#### 7. Standalone inference verification

A standalone inference test was performed for:

```text
Colchicine — DB01394
```

The candidate set contains 4,278 drugs.

After filtering:

* Known-positive pairs: 1,488
* Remaining candidate pairs: 2,789

The verified top-ranked candidates include:

| Rank | Drug                             | DrugBank ID | Raw Score |
| ---: | -------------------------------- | ----------- | --------: |
|    1 | Probenecid                       | DB01032     |   40.8524 |
|    2 | Hydrocortisone                   | DB00741     |    7.9139 |
|    3 | Ondansetron                      | DB00904     |    5.7451 |
|    4 | Sulfinpyrazone                   | DB01138     |    5.6925 |
|    5 | Melengestrol acetate             | DB14659     |    5.5811 |
|    6 | Prednisone acetate               | DB14646     |    5.2154 |
|    7 | Coumarin                         | DB04665     |    5.1917 |
|    8 | Dicoumarol                       | DB00266     |    5.1416 |
|    9 | Methylprednisolone hemisuccinate | DB14644     |    5.0514 |
|   10 | Oxycodone                        | DB00497     |    4.9924 |

These values are raw ranking scores and are **not probabilities, calibrated confidence values, or clinical risk scores**.

---

### 4.2.12. Lightweight Inference Runtime

The lightweight runtime is located at:

```text
final_release/lightweight_runtime/
```

It contains:

```text
ddi_runtime_embeddings.npz
drug_metadata.csv
known_positive_mask_packed.npz
```

The runtime uses a NumPy scoring operation conceptually equivalent to:

```text
query_embedding @ (candidate_embeddings * ddi_relation).T
```

The exported runtime has been independently verified against the full model for the Colchicine example.

---

### 4.2.13. G3 Graph Context Runtime

The G3 context runtime is located at:

```text
final_release/g3_context_runtime/
```

It preserves the heterogeneous support relations used by the final graph.

Total forward support rows:

```text
68,284
```

Breakdown:

```text
Drug–Gene/Protein: 25,653
Drug–Disease:      42,631
```

For the Colchicine–Probenecid pair:

* Colchicine node: 39
* Probenecid node: 69
* Shared entities: 33
* Shared gene/protein entities: 3
* Shared disease entities: 30

Shared gene/protein relationships include:

* ALB — carrier/carrier
* CYP2C8 — enzyme/enzyme
* CYP3A4 — enzyme/enzyme

These shared entities are presented as graph context only. They are **not causal explanations** for the predicted score.

---

### 4.2.14. Independent FDA and PubMed Evidence

CHEERS separates model prediction from external evidence retrieval.

The application can retrieve evidence from:

* openFDA drug labeling
* PubMed

The evidence retrieval process includes:

1. Retrieve relevant FDA labeling information.
2. Search for explicit cross-drug mentions.
3. Retrieve PubMed records using a conservative query containing both drugs.
4. Return up to five relevant records where available.

No LLM-generated summary is used as the source of the retrieved evidence.

The evidence endpoint is:

```text
/api/evidence/pair?drug_a_id=DB01394&drug_b_id=DB01032
```

External evidence should be interpreted independently from the model score.

---

### 4.2.15. Web Application

The CHEERS web application consists of:

```text
FastAPI Backend
      ↓
NumPy Inference Runtime
      ↓
Graph Context Index
      ↓
FDA / PubMed Retrieval
      ↓
HTML / CSS / Vanilla JavaScript Frontend
```

Major functions include:

* Partial drug search
* Exact drug-name search
* DrugBank ID search
* Autocomplete
* Top-K drug-pair prediction
* Raw-score ranking
* Known-positive filtering
* Self-pair filtering
* Graph-composition experiment information
* Model verification information
* Drug-pair graph context
* External FDA/PubMed evidence
* Medicine/Disease exploration
* Graph Explorer
* Subgraph / Pair Context Explorer
* My Health
* DDI Predictor

The application is designed as a research demonstration rather than a clinical decision-support tool.

---

### 4.2.16. Ask CHEERS

The application also provides an **Ask CHEERS** interface for grounded interaction with the project information.

Where applicable, explanations are grounded in available project/model context and external evidence rather than treating the raw model score as a medical conclusion.

The system is designed to distinguish:

```text
Model Prediction
      ≠
Graph Context
      ≠
External Evidence
      ≠
Clinical Judgment
```

---

### 4.2.17. API Specification

The main API endpoints are:

| Endpoint             | Description                           |
| -------------------- | ------------------------------------- |
| `/`                  | Application entry point               |
| `/api`               | API information                       |
| `/api/health`        | Health check                          |
| `/api/model`         | Model/runtime information             |
| `/api/experiment`    | Experiment information                |
| `/api/verification`  | Verification information              |
| `/api/drugs/search`  | Drug search                           |
| `/api/predict`       | Drug-pair prediction                  |
| `/api/context/pair`  | Graph context for a drug pair         |
| `/api/evidence/pair` | FDA/PubMed evidence                   |
| `/docs`              | FastAPI interactive API documentation |

---

## 4.3. Directory Structure

The official repository is organized as follows:

```text
capstone-2026-team-05/
│
├── api/
│   └── main.py
│
├── checkpoints/
│   └── rgcn_multiseed/
│
├── data/
│   └── processed/
│       ├── mappings/
│       └── rgcn_tensors/
│
├── docs/
│   ├── 01.보고서/
│   ├── 02.포스터/
│   └── 03.발표자료/
│
├── figures/
│
├── final_release/
│   ├── lightweight_runtime/
│   ├── g3_context_runtime/
│   ├── app_requirements.txt
│   ├── PORTABLE_APP_MANIFEST_V2.json
│   └── PORTABLE_APP_MANIFEST_V3.json
│
├── frontend/
│
├── notebooks/
│
├── results/
│   ├── live_5seed/
│   └── ...
│
├── scripts/
│
├── src/
│
├── web/
│
├── .github/
│   └── workflows/
│
├── .gitignore
├── PORTABLE_APP_MANIFEST.json
├── README.md
└── THIRD_PARTY_NOTICES.md
```

---

## 4.4. Industry Mentoring Feedback and Reflected Improvements

> **To be completed with the final mentoring record.**

The final version of this section should describe:

* Date and topic of each mentoring session
* Feedback provided by the industry mentor
* Technical or design issues identified
* Changes made based on the feedback
* Remaining limitations

Suggested format:

| Mentoring Topic     | Feedback          | Reflected Improvement |
| ------------------- | ----------------- | --------------------- |
| Model / Research    | [To be completed] | [To be completed]     |
| System Architecture | [To be completed] | [To be completed]     |
| UI / UX             | [To be completed] | [To be completed]     |
| Deployment          | [To be completed] | [To be completed]     |

---

# 5. Installation and Execution

## 5.1. Installation Procedure and Execution

CHEERS provides a lightweight local application that can be executed without a GPU.

### Requirements

Recommended environment:

* Python 3.12.6
* NumPy 1.26.4
* FastAPI 0.141.1
* Uvicorn 0.52.1
* Pydantic 2.13.4

The required packages are listed in:

```text
final_release/app_requirements.txt
```

### Step 1. Clone the repository

```bash
git clone https://github.com/pnucse-capstone2026/capstone-2026-team-05.git
cd capstone-2026-team-05
```

### Step 2. Create a virtual environment

#### macOS / Linux

```bash
python3 -m venv .venv
source .venv/bin/activate
```

#### Windows

```powershell
python -m venv .venv
.venv\Scripts\activate
```

### Step 3. Install dependencies

```bash
pip install -r final_release/app_requirements.txt
```

### Step 4. Run verification scripts

The repository contains verification scripts for checking the exported runtime and project artifacts.

Refer to:

```text
scripts/
final_release/
```

for the available verification commands.

### Step 5. Start the FastAPI server

```bash
python -m uvicorn api.main:app --host 127.0.0.1 --port 8000
```

### Step 6. Open the application

Web application:

```text
http://127.0.0.1:8000
```

FastAPI documentation:

```text
http://127.0.0.1:8000/docs
```

---

### Full Research Training Environment

The full R-GCN training environment was originally configured on a university GPU environment:

```text
/workspace/primekg_ddi_rgcn
```

Environment:

* Conda environment: `primekg-rgcn`
* Python 3.10.20
* PyTorch 2.2.1 + CUDA 12.1
* PyTorch Geometric 2.5.3
* NumPy 1.26.4
* 4 × RTX 2080 Ti

The final training environment was configured with an isolated physical GPU for the project.

The full preprocessing and retraining pipeline is not intended to be fully self-contained in the portable repository.

---

## 5.2. Troubleshooting

### Port 8000 is already in use

Run the application on another port:

```bash
python -m uvicorn api.main:app --host 127.0.0.1 --port 8001
```

Then open:

```text
http://127.0.0.1:8001
```

### Python package installation fails

Check the Python version:

```bash
python --version
```

Then reinstall the requirements:

```bash
pip install -r final_release/app_requirements.txt
```

### The application cannot find runtime files

Make sure the command is executed from the repository root:

```text
capstone-2026-team-05/
```

and that the following directory exists:

```text
final_release/lightweight_runtime/
```

### GPU or CUDA errors

The lightweight application does not require CUDA or a GPU.

For the portable application, use:

```text
final_release/app_requirements.txt
```

rather than installing the full PyTorch/PyTorch Geometric training environment.

### External evidence is unavailable

FDA and PubMed retrieval requires external network access. If the external services are temporarily unavailable, model prediction and local graph-context functions may still be available independently.

---

# 6. Introduction Materials and Demonstration Video

## 6.1. Project Introduction Materials

The official project materials are stored in:

```text
docs/
├── 01.보고서/
│   ├── 01.착수보고서.pdf
│   ├── 02.중간보고서.pdf
│   └── 03.최종보고서.pdf
│
├── 02.포스터/
│   └── 포스터파일.pdf
│
└── 03.발표자료/
    ├── 발표자료.pdf
    └── 발표자료.pptx
```

Additional project figures and experiment artifacts are available under:

```text
figures/
results/
```

---

## 6.2. Demonstration Video

**Demo Video:** [To be added]

**Live Demo:** [To be added, if available]

The demonstration video should show:

1. Opening the CHEERS application
2. Searching for a drug
3. Running a DDI prediction
4. Viewing ranked candidates
5. Inspecting graph context
6. Viewing external evidence
7. Exploring experiment and verification information

---

# 7. Team

## 7.1. Team Member Introduction and Role Distribution

**Team CHEERS**
Pusan National University — Computer Science

| Team Member | Role   | Main Responsibilities |
| ----------- | ------ | --------------------- |
| [Member 1]  | [Role] | [Responsibilities]    |
| [Member 2]  | [Role] | [Responsibilities]    |
| [Member 3]  | [Role] | [Responsibilities]    |
| [Member 4]  | [Role] | [Responsibilities]    |
| [Member 5]  | [Role] | [Responsibilities]    |

The final team information should match the official PNU capstone registration record.

---

## 7.2. Team Member Reflections

> **To be completed by each team member.**

Each reflection may include:

* Personal contribution
* Technical challenges
* Research experience
* Collaboration experience
* What was learned through the project
* Future improvements

Suggested format:

### Member 1

> [Reflection]

### Member 2

> [Reflection]

### Member 3

> [Reflection]

### Member 4

> [Reflection]

### Member 5

> [Reflection]

---

# 8. References and Sources

## 8.1. Main Biomedical Data Source

**PrimeKG**

PrimeKG is used as the primary biomedical knowledge graph for the project.

The project specifically uses the `drug_drug` relation represented in PrimeKG as a synergistic interaction relationship.

---

## 8.2. Drug Identifiers

DrugBank identifiers are used for drug-level identification and metadata matching where available.

---

## 8.3. External Evidence Sources

### openFDA

FDA labeling information is retrieved independently from the model prediction pipeline.

### PubMed

PubMed is used to retrieve literature records relevant to selected drug pairs.

External evidence is presented as supporting information and is not used to convert the model score into a clinical conclusion.

---

## 8.4. Technical References

The project uses and builds upon established methods and software including:

* Relational Graph Convolutional Networks
* PyTorch
* PyTorch Geometric
* FastAPI
* NumPy
* PrimeKG
* openFDA
* PubMed

Specific bibliographic references and licenses are documented in:

```text
THIRD_PARTY_NOTICES.md
```

---

# Research Scope and Reproducibility

## Experimental Scope

The final experiment investigates:

> **The effect of biomedical knowledge graph composition on R-GCN-based drug–drug link prediction.**

The controlled comparison is:

```text
G0: DDI
G1: DDI + Drug-Gene/Protein
G2: DDI + Drug-Disease
G3: DDI + Drug-Gene/Protein + Drug-Disease
```

All variants use the same DDI split, R-GCN architecture, optimization settings, and filtered evaluation procedure.

---

## Reproducibility Levels

### Level 1 — Verified Demonstration

The lightweight runtime and web application are included and independently verified.

### Level 2 — Full Model Retraining

The complete original preprocessing and training environment is not fully self-contained in the portable repository.

The repository therefore prioritizes reproducible **final inference and verification** rather than claiming complete one-command retraining from raw PrimeKG.

---

# Limitations

The current project has the following limitations:

1. The model is based primarily on PrimeKG.
2. The target relation is PrimeKG's `drug_drug` / synergistic interaction relation.
3. Only one principal GNN architecture, R-GCN, is used.
4. The current graph-composition experiment uses multiple seeds for robustness.
5. No formal statistical significance testing is performed.
6. The evaluation is transductive.
7. Unobserved drug pairs are not confirmed negative interactions.
8. Relation-level ablation does not establish causal importance.
9. Knowledge graph incompleteness and source bias may affect results.
10. Raw model scores are not calibrated probabilities.
11. Graph context does not establish causality.
12. A predicted link does not constitute a clinically confirmed drug interaction.
13. Full preprocessing and retraining from raw data are not included as a self-contained pipeline.
14. External evidence retrieval depends on the availability of the corresponding services.

---

# Future Work

Potential future work includes:

* More extensive relation-level ablation
* Additional random seeds and bootstrap analysis
* Statistical significance testing
* Comparison with stronger GNN and knowledge-graph baselines
* Inductive and cold-start evaluation
* External DDI validation datasets
* Calibrated classification
* Integration of additional biomedical knowledge sources
* Broader DailyMed/openFDA evidence retrieval
* Improved synonym-aware drug matching
* Systematic literature review
* Pair-level explanatory paths
* Improved semantic representation of graph relations
* More comprehensive clinical validation

---

# Safety and Responsible Use

CHEERS is a **research and educational prototype**.

The model output must not be used to:

* Prescribe medication
* Stop medication
* Change medication dosage
* Determine whether a medication is safe or dangerous
* Replace professional medical advice

A raw model score represents a ranking signal within the project's experimental setting. It is not:

* A probability
* A calibrated confidence value
* A clinical risk score
* A severity score
* A confirmation of a real-world drug interaction

Likewise, graph relationships should not be interpreted as causal explanations.

Clinical interpretation requires qualified healthcare professionals and authoritative medical information.

---

# Project Acknowledgment

**Team CHEERS**
**Pusan National University**

This repository contains the final academic project artifacts, research results, lightweight inference runtime, and web demonstration developed for the PNU Computer Science Capstone Project.

For third-party software, datasets, and licenses, see:

```text
THIRD_PARTY_NOTICES.md
```

---

## Quick Start

```bash
git clone https://github.com/pnucse-capstone2026/capstone-2026-team-05.git
cd capstone-2026-team-05

python3 -m venv .venv
source .venv/bin/activate

pip install -r final_release/app_requirements.txt

python -m uvicorn api.main:app --host 127.0.0.1 --port 8000
```

Then open:

```text
http://127.0.0.1:8000
```

API documentation:

```text
http://127.0.0.1:8000/docs
```

---

> **CHEERS is a research prototype for studying biomedical knowledge graph composition and drug–drug link prediction. It is not a clinical decision-support system.**
