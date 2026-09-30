# CHEERS

### Effect of Biomedical Knowledge Graph Composition on R-GCN-Based Drug–Drug Interaction Prediction

**Team CHEERS**
**Pusan National University — Department of Computer Science**

---

## 1. Project Background

### 1.1. Domestic and International Market Status and Problems

Drug–Drug Interaction (DDI) is an important problem in biomedical and clinical informatics because drugs may interact with one another through shared biological targets, enzymes, transporters, diseases, and other biomedical relationships.

Biomedical Knowledge Graphs (KGs) provide a way to represent these heterogeneous relationships as a graph. By combining multiple biomedical relation types, a knowledge graph can provide contextual information beyond direct drug–drug relationships.

However, adding more biomedical information does not necessarily guarantee better DDI prediction performance. Additional relations may provide useful contextual information, but they may also introduce noise or unnecessary graph complexity.

CHEERS therefore focuses on the following research question:

> **How does biomedical knowledge graph composition affect R-GCN-based drug–drug link prediction?**

Rather than simply building a larger graph, CHEERS evaluates how different combinations of biomedical relations affect an R-GCN model under a controlled experimental setting.

The project uses **PrimeKG** as the main biomedical knowledge graph and focuses on its `drug_drug` relation as the target prediction task.

In the application, this target relation is displayed as **“synergistic interaction.”**

> **Scope:** CHEERS does not attempt to predict every clinically meaningful drug–drug interaction. The target relation is the `drug_drug` relation available in PrimeKG.

---

### 1.2. Necessity and Expected Effects

A major challenge in biomedical graph learning is determining which types of contextual information are actually useful for a prediction task.

Simply adding more relations can increase graph complexity, computational cost, and noise. Therefore, it is important to experimentally compare different graph compositions while keeping the prediction task and model architecture fixed.

CHEERS addresses this problem through a controlled comparison of four graph variants:

* **G0:** DDI only
* **G1:** DDI + Drug-Gene/Protein
* **G2:** DDI + Drug-Disease
* **G3:** DDI + Drug-Gene/Protein + Drug-Disease

All graph variants use the same DDI split, R-GCN architecture, decoder, training procedure, and evaluation protocol.

The expected contributions are:

1. Quantifying the effect of biomedical graph composition on DDI link prediction.
2. Identifying relation families that provide useful contextual information.
3. Providing reproducible experimental artifacts and verification results.
4. Demonstrating how research-oriented DDI prediction can be connected to a lightweight web application.
5. Separating model-generated ranking results from independently retrieved FDA/PubMed evidence.

---

# 2. Development Goals

## 2.1. Goals and Detailed Contents

The main goal of CHEERS is to investigate the effect of biomedical knowledge graph composition on **R-GCN-based drug–drug link prediction**.

### Research Question

> **How does biomedical knowledge graph composition affect R-GCN-based drug–drug link prediction?**

### Overall Research Pipeline

```text
PrimeKG
   ↓
Canonical DDI Pair Construction
   ↓
Fixed Train / Validation / Test Split
   ↓
G0 / G1 / G2 / G3 Graph Construction
   ↓
Same R-GCN Architecture
   ↓
Filtered Link Prediction
   ↓
MRR / Hits@1 / Hits@5 / Hits@10
   ↓
Graph Composition Comparison
```

### Project Evolution

The project initially considered a broader clinical inference pipeline:

```text
Symptoms
   ↓
Disease
   ↓
Treatment Drug
   ↓
Drug–Drug Interaction Warning
```

Early development considered multiple biomedical knowledge graph and embedding approaches, including:

* PrimeKG
* DrugBank
* DDInter
* TransE
* ComplEx
* RotatE
* PyKEEN
* FastAPI
* React
* Cytoscape

The research scope was subsequently narrowed to:

> **PrimeKG + R-GCN + Drug–Drug Interaction Prediction**

This narrower scope allows the effect of graph composition to be evaluated under controlled experimental conditions.

---

### PrimeKG DDI Canonicalization

PrimeKG contains directed DDI rows that represent the same undirected drug pair in both directions.

The canonicalization process produced:

| Item                    |     Count |
| ----------------------- | --------: |
| Directed DDI rows       | 2,672,628 |
| Unique undirected pairs | 1,336,314 |
| Reverse duplicates      | 1,336,314 |
| Self-loops              |         0 |

The final experiment uses canonical unique DDI pairs.

---

### Fixed DDI Split

The same DDI split is used for every graph variant.

| Split      |         Pairs |
| ---------- | ------------: |
| Train      |     1,069,080 |
| Validation |       133,620 |
| Test       |       133,614 |
| **Total**  | **1,336,314** |

The split is:

* fixed across G0–G3
* free of train/validation/test overlap
* free of symmetric duplicates
* transductive

Validation and test DDI edges are excluded from the message-passing adjacency to prevent target-edge leakage.

---

### Graph Composition

The experiment compares four graph variants.

| Variant | Composition                            | Directed Edges | Active Relations |
| ------- | -------------------------------------- | -------------: | ---------------: |
| **G0**  | DDI only                               |      2,138,160 |                1 |
| **G1**  | DDI + Drug-Gene/Protein                |      2,189,466 |                9 |
| **G2**  | DDI + Drug-Disease                     |      2,223,422 |                7 |
| **G3**  | DDI + Drug-Gene/Protein + Drug-Disease |      2,274,728 |               15 |

Additional relation counts:

**G1 Drug-Gene/Protein forward relations**

| Relation    |      Edges |
| ----------- | ---------: |
| Target      |     16,380 |
| Enzyme      |      5,317 |
| Transporter |      3,092 |
| Carrier     |        864 |
| **Total**   | **25,653** |

**G2 Drug-Disease forward relations**

| Relation         |      Edges |
| ---------------- | ---------: |
| Indication       |      9,388 |
| Contraindication |     30,675 |
| Off-label use    |      2,568 |
| **Total**        | **42,631** |

**G3 additional support edges**

> **68,284 forward support edges**

---

### Global Relation Mapping

| ID | Relation             |
| -: | -------------------- |
|  0 | drug_drug            |
|  1 | target               |
|  2 | rev_target           |
|  3 | enzyme               |
|  4 | rev_enzyme           |
|  5 | transporter          |
|  6 | rev_transporter      |
|  7 | carrier              |
|  8 | rev_carrier          |
|  9 | indication           |
| 10 | rev_indication       |
| 11 | contraindication     |
| 12 | rev_contraindication |
| 13 | off-label use        |
| 14 | rev_off-label use    |

---

### Graph Nodes

The final graph contains:

* **13,094 shared graph nodes**
* **4,278 candidate drug nodes**
* **15 relation types**
* **4,278 × 4,278 known-positive mask**
* **2,672,628 symmetric known-positive entries**

---

### R-GCN Model

The final model uses a two-layer Relational Graph Convolutional Network.

| Configuration            | Value                            |
| ------------------------ | -------------------------------- |
| GNN                      | 2-layer RGCNConv                 |
| Embedding dimension      | 128                              |
| Hidden dimension         | 128                              |
| Dropout                  | 0.2                              |
| Learning rate            | 0.001                            |
| Weight decay             | 1e-5                             |
| Maximum epochs           | 500                              |
| Early stopping patience  | 10                               |
| Positive samples / epoch | 100,000                          |
| Negative sampling ratio  | 1:1                              |
| Parameters               | 2,200,704                        |
| Decoder                  | Symmetric DistMult-style decoder |

---

### Negative Sampling

Negative samples are randomly sampled from **unobserved drug pairs**.

Therefore:

> An unobserved pair is not treated as a confirmed negative interaction.

The sampled negatives represent candidate unknown pairs rather than clinically verified non-interactions.

---

### Training and Model Selection

All graph variants use the same:

* fixed DDI split
* R-GCN architecture
* optimization configuration
* training protocol
* evaluation procedure

Validation BCE is used for checkpoint selection.

The test set is not used for model selection.

For the verified demonstration runtime, the G3 seed-44 checkpoint is:

```text
checkpoints/rgcn_multiseed/G3_seed44_best.pt
```

The best epoch for G3 seed-44 was **499**.

---

### Filtered Ranking Evaluation

The test set contains:

* 133,614 held-out DDI pairs
* 2 directions per pair
* 267,228 ranking queries
* 4,278 candidate drugs per query

The evaluation metrics are:

* Mean Reciprocal Rank (MRR)
* Hits@1
* Hits@5
* Hits@10

Known positive DDI pairs are filtered from the candidate set during ranking evaluation.

---

## 2.2. Differentiation from Existing Services

CHEERS differs from conventional drug interaction lookup approaches in several aspects.

### 1. Controlled Knowledge Graph Composition Experiment

Rather than only retrieving known DDI information, CHEERS investigates how different biomedical graph compositions affect an R-GCN model.

### 2. Same Model and Evaluation Conditions

The G0–G3 comparison keeps the following fixed:

* DDI split
* R-GCN architecture
* decoder
* training procedure
* filtered ranking protocol

Therefore, graph composition is the primary experimental variable.

### 3. Relation-Level Ablation

The project also examines individual relation families through A1–A7 ablation experiments.

### 4. Research-Oriented Prediction Interface

The web application allows users to explore model-generated rankings while clearly distinguishing those rankings from external biomedical evidence.

### 5. Lightweight Inference

The final demonstration can run using NumPy-based inference without requiring a GPU or the original PyTorch/PyG training environment.

---

## 2.3. Social Value and Sustainability Plan

CHEERS is designed as a **research and educational demonstration**, rather than a clinical decision-support system.

The project aims to contribute to:

* reproducible biomedical AI research
* transparent knowledge graph experimentation
* educational use of graph-based biomedical prediction
* responsible presentation of AI-generated biomedical information
* separation of model predictions and external medical evidence

The system does not provide:

* prescribing recommendations
* medication discontinuation recommendations
* dosage recommendations
* definitive safe/dangerous judgments
* clinical risk assessments

Clinical interpretation requires qualified medical professionals and appropriate external evidence.

---

# 3. System Design

## 3.1. System Architecture

### Overall Research Architecture

```text
                         ┌─────────────────────┐
                         │       PrimeKG       │
                         └──────────┬──────────┘
                                    │
                                    ▼
                       ┌─────────────────────────┐
                       │ Canonical DDI Processing│
                       └────────────┬────────────┘
                                    │
             ┌──────────────────────┼──────────────────────┐
             │                      │                      │
             │                      │                      │
             ▼                      ▼                      ▼
          ┌──────┐               ┌──────┐               ┌──────┐
          │  G0  │               │  G1  │               │  G2  │
          │ DDI  │               │ DDI  │               │ DDI  │
          │ only │               │ + GP │               │ + DD │
          └───┬──┘               └───┬──┘               └───┬──┘
              │                      │                      │
              │                      │                      │
              │                  ┌───┴───┐                  │
              │                  │  G3   │                  │
              │                  │ DDI   │                  │
              │                  │ + GP  │                  │
              │                  │ + DD  │                  │
              │                  └───┬───┘                  │
              │                      │                      │
              └──────────────────────┼──────────────────────┘
                                     │
                                     ▼
                              ┌─────────────┐
                              │   Same      │
                              │   R-GCN     │
                              │  2 Layers   │
                              └──────┬──────┘
                                     │
                                     ▼
                            ┌──────────────────┐
                            │  Link Prediction │
                            └────────┬─────────┘
                                     │
                                     ▼
                         ┌────────────────────────┐
                         │   Filtered Ranking     │
                         │   MRR / Hits@K         │
                         └────────────────────────┘
```

> **GP:** Drug-Gene/Protein
> **DD:** Drug-Disease

The four graph variants are evaluated under the same R-GCN architecture, training configuration, and filtered ranking protocol. Graph composition is therefore the primary experimental variable.

---

### Application Architecture

```text
┌─────────────────────────────────────────────────────┐
│                    Web Frontend                     │
│              HTML / CSS / Vanilla JS                │
└───────────────────────┬─────────────────────────────┘
                        │ HTTP
                        ▼
┌─────────────────────────────────────────────────────┐
│                  FastAPI Backend                    │
├─────────────────────────────────────────────────────┤
│ Drug Search                                         │
│ DDI Prediction                                      │
│ Pair Context                                        │
│ Experiment Information                              │
│ Verification Information                            │
│ FDA / PubMed Evidence Retrieval                     │
└───────────────┬──────────────────┬──────────────────┘
                │                  │
                ▼                  ▼
       ┌────────────────┐   ┌──────────────────┐
       │ NumPy Runtime  │   │ External Sources │
       │ Final G3 Model │   │ openFDA / PubMed │
       └────────────────┘   └──────────────────┘
```

The web application uses a lightweight NumPy-based runtime for model inference. External biomedical evidence is retrieved independently from openFDA and PubMed.

---

## 3.2. Technologies Used

### Machine Learning and Training

* Python
* PyTorch
* PyTorch Geometric
* R-GCN

### Lightweight Inference

* NumPy

### Backend

* FastAPI
* Python standard library
* NumPy

### Frontend

* HTML
* CSS
* Vanilla JavaScript

### Biomedical Data and External Evidence

* PrimeKG
* openFDA
* PubMed

### Development Environment

The original model training environment was:

```text
Environment: /workspace/primekg_ddi_rgcn
Conda environment: primekg-rgcn

Python: 3.10.20
PyTorch: 2.2.1+cu121
PyTorch Geometric: 2.5.3
NumPy: 1.26.4
GPU: 4 × RTX 2080 Ti
```

The final physical training run used an isolated GPU.

> **Note:** The original PyTorch/PyG training environment is not required for the lightweight demonstration runtime. The final demonstration uses pre-exported model artifacts and NumPy-based inference.

---

# 4. Development Results

## 4.1. Overall System Flow

### Research Flow

```text
                         PrimeKG
                            │
                            ▼
                 Canonical DDI Processing
                            │
              ┌─────────────┼─────────────┐
              │             │             │
              ▼             ▼             ▼
             G0            G1            G2
           DDI only      DDI + GP      DDI + DD
              │             │             │
              │             │             │
              └─────────────┼─────────────┘
                            │
                            ▼
                           G3
                       DDI + GP + DD
                            │
                            ▼
                    Same R-GCN Model
                            │
                            ▼
                    DDI Link Prediction
                            │
                            ▼
                     Filtered Ranking
                            │
                            ▼
                     MRR / Hits@K
```

The four graph variants, G0–G3, are evaluated under the same R-GCN architecture, training configuration, and filtered ranking protocol. Graph composition is therefore the primary experimental variable.

*GP: Drug-Gene/Protein relations*
*DD: Drug-Disease relations*

### Application Flow

```text
User
 │
 ▼
Drug Search
 │
 ▼
Select Drug
 │
 ▼
DDI Prediction
 │
 ▼
Known DDI Filtering
 │
 ▼
Top-K Ranking
 │
 ├───────────────┐
 ▼               ▼
Pair Context   External Evidence
 │               │
 ▼               ├── openFDA
Graph Context    └── PubMed
```

---

## 4.2. Feature Description and Main Functional Specifications

### 4.2.1. Drug Search

Users can search for drugs using:

* partial drug names
* exact drug names
* DrugBank IDs
* autocomplete suggestions

**Input**

```text
Drug name or DrugBank ID
```

**Output**

```text
Matched drug
DrugBank ID
Available prediction information
```

---

### 4.2.2. DDI Predictor

The DDI Predictor uses the final G3/seed-44 model export to rank candidate drugs.

**Input**

```text
Query drug
Top-K value
```

**Output**

```text
Candidate drug
DrugBank ID
Raw model score
Ranking
```

The raw score is a ranking value.

It is **not**:

* a probability
* calibrated confidence
* clinical risk
* interaction severity
* a safe/dangerous classification

---

### 4.2.3. Known-Positive Filtering

Known DDI pairs are filtered from candidate results to avoid presenting already-known positive pairs as newly predicted candidates.

For the final standalone inference verification:

```text
Candidate drug nodes: 4,278
Known positive pairs filtered: 1,488
Remaining ranked candidates: 2,789
Query drug itself: excluded
```

---

### 4.2.4. Graph Context Explorer

The application provides graph context associated with predicted drug pairs.

For the G3 context runtime:

```text
Total forward support rows: 68,284
Drug-Gene/Protein: 25,653
Drug-Disease: 42,631
```

The graph context is descriptive and should not be interpreted as a causal explanation.

---

### 4.2.5. Pair Context

For the verified example:

```text
Query drug: Colchicine
DrugBank ID: DB01394

Candidate drug: Probenecid
DrugBank ID: DB01032
```

The G3 context runtime reports:

```text
Colchicine support entities: 39
Probenecid support entities: 69
Shared entities: 33
Shared gene/protein entities: 3
Shared disease entities: 30
```

Examples of shared gene/protein entities include:

```text
ALB
CYP2C8
CYP3A4
```

These relationships are presented as graph context rather than causal explanations.

---

### 4.2.6. External Evidence

CHEERS separately retrieves external evidence from:

* **openFDA**
* **PubMed**

The evidence retrieval process does not use an LLM to generate medical conclusions.

For selected drug pairs, the system retrieves explicit cross-drug mentions from relevant FDA label sections and searches PubMed using both drug names.

**API endpoint**

```text
/api/evidence/pair?drug_a_id=DB01394&drug_b_id=DB01032
```

Model predictions and external evidence are presented as separate information sources.

---

### 4.2.7. Experiment Information

The web application provides information about:

* G0–G3 graph composition
* model configuration
* evaluation methodology
* verification results
* research limitations

---

### 4.2.8. API Endpoints

| Endpoint             | Description              |
| -------------------- | ------------------------ |
| `/`                  | Web application          |
| `/api`               | API information          |
| `/api/health`        | Health check             |
| `/api/model`         | Model information        |
| `/api/experiment`    | Experiment information   |
| `/api/verification`  | Verification information |
| `/api/drugs/search`  | Drug search              |
| `/api/predict`       | DDI prediction           |
| `/api/context/pair`  | Pair graph context       |
| `/api/evidence/pair` | FDA/PubMed evidence      |
| `/docs`              | FastAPI documentation    |

---

### 4.2.9. Final Verification Results

The final project includes several verification procedures to check data integrity, model behavior, runtime reproducibility, and application consistency.

#### 1. Target-Edge Leakage Check

```text
Validation target-edge leakage: 0
Test target-edge leakage: 0
```

#### 2. Test Ranking Verification

A 1,000-pair test subset was evaluated in both directions.

```text
Queries: 2,000

MRR:     0.502226
Hits@1:  0.4525
Hits@5:  0.5465
Hits@10: 0.5870

Median rank: 2
```

#### 3. Positive vs. Unobserved-Pair Sanity Check

```text
Positive mean score:       161.370407
Positive median score:       6.497307

Unobserved mean score:      -2.773926
Unobserved median score:    -1.558111

Pairwise win rate:           97.54%
ROC-AUC:                     0.9737
```

These values are used as a model-behavior sanity check against unobserved pairs and should not be interpreted as clinical performance.

#### 4. Metadata Resolution

All:

```text
13,094 graph nodes
4,278 candidate drugs
```

resolve to available metadata.

#### 5. Checkpoint Reproducibility

The G3 seed-44 checkpoint reproduces:

```text
MRR:     0.540359
Hits@1:  0.490656
Hits@5:  0.588273
Hits@10: 0.626229
```

#### 6. Graph Edge-Count Verification

The final graph variants match the expected edge counts specified in the experiment configuration.

#### 7. Standalone Inference Verification

A standalone lightweight inference test was performed using:

```text
Query drug: Colchicine
DrugBank ID: DB01394
```

After filtering known-positive pairs and excluding the query drug itself, the final G3/seed-44 runtime produced the following Top-10 ranking:

| Rank | Drug                             | DrugBank ID |   Score |
| ---: | -------------------------------- | ----------- | ------: |
|    1 | Probenecid                       | DB01032     | 40.8524 |
|    2 | Hydrocortisone                   | DB00741     |  7.9139 |
|    3 | Ondansetron                      | DB00904     |  5.7451 |
|    4 | Sulfinpyrazone                   | DB01138     |  5.6925 |
|    5 | Melengestrol acetate             | DB14659     |  5.5811 |
|    6 | Prednisone acetate               | DB14646     |  5.2154 |
|    7 | Coumarin                         | DB04665     |  5.1917 |
|    8 | Dicoumarol                       | DB00266     |  5.1416 |
|    9 | Methylprednisolone hemisuccinate | DB14644     |  5.0514 |
|   10 | Oxycodone                        | DB00497     |  4.9924 |

---

### 4.2.10. Final Graph Composition Results

The main graph-composition experiment compares four graph variants:

* **G0:** DDI only
* **G1:** DDI + Drug-Gene/Protein relations
* **G2:** DDI + Drug-Disease relations
* **G3:** DDI + Drug-Gene/Protein + Drug-Disease relations

Each graph variant was evaluated using five random seeds (42, 43, 44, 45, and 46). All variants used the same DDI split, R-GCN architecture, training configuration, and full filtered-ranking evaluation protocol.

The final five-seed experiment evaluated 267,228 ranking queries using MRR, Hits@1, Hits@5, and Hits@10.

The complete experiment summary is available at:

```text
results/live_5seed/final_experiment_summary.json
```

| Graph | Composition                            |                 MRR |              Hits@1 |              Hits@5 |             Hits@10 |
| ----- | -------------------------------------- | ------------------: | ------------------: | ------------------: | ------------------: |
| G0    | DDI only                               | 0.527284 ± 0.006373 | 0.480464 ± 0.005801 | 0.572694 ± 0.007746 | 0.609447 ± 0.009457 |
| G1    | DDI + Drug-Gene/Protein                | 0.530969 ± 0.007414 | 0.484194 ± 0.007010 | 0.575418 ± 0.007907 | 0.612683 ± 0.007986 |
| G2    | DDI + Drug-Disease                     | 0.526776 ± 0.007482 | 0.480197 ± 0.006691 | 0.571716 ± 0.008899 | 0.608608 ± 0.009758 |
| G3    | DDI + Drug-Gene/Protein + Drug-Disease | 0.534209 ± 0.006288 | 0.486290 ± 0.005899 | 0.580468 ± 0.007000 | 0.618074 ± 0.007312 |

### Primary Result

G3 achieved the highest mean performance among the four graph variants:

```text
G3 mean MRR:                  0.534209
G3 MRR standard deviation:    0.006288
Improvement over G0:          +0.006924 MRR
Relative MRR improvement:     +1.31%
Seeds outperforming G0:       5/5
```

G3 outperformed the DDI-only G0 baseline in all five random seeds across MRR, Hits@1, Hits@5, and Hits@10.

These results provide robustness evidence for the effect of graph composition under the specified experimental setting. Statistical significance is not claimed.

The seed-44 checkpoint used for the verified lightweight application runtime is a specific model artifact and should not be treated as the sole estimate of the graph-composition effect. The five-seed results above are the primary results for the G0–G3 graph-composition experiment.

---

## 4.3. Directory Structure

```text
CHEERS/
├── api/
├── checkpoints/
├── data/
├── docs/
├── figures/
├── final_release/
├── frontend/
├── notebooks/
├── results/
├── scripts/
├── src/
├── web/
├── .github/
├── .gitignore
├── PORTABLE_APP_MANIFEST.json
├── README.md
└── THIRD_PARTY_NOTICES.md
```

### Important Directories

```text
checkpoints/
```

Contains trained model checkpoints.

```text
data/
```

Contains processed graph and model-related data.

```text
figures/
```

Contains research visualization files.

```text
final_release/
```

Contains lightweight runtime artifacts, context runtime files, verification materials, and release manifests.

```text
results/
```

Contains experiment results, including the current five-seed graph-composition summary.

```text
scripts/
```

Contains utility, experiment, and verification scripts.

```text
src/
```

Contains research and model source code.

```text
web/
```

Contains web application-related components.

---

# 5. Installation and Execution

## 5.1. Installation and Execution Procedure

CHEERS provides two execution environments:

1. **Full Research / Training Environment** for model training and experimental reproduction
2. **Lightweight Web Application** for the final demonstration and inference

### A. Full Research / Training Environment

The original research and training environment used:

```text
Python 3.10.20
PyTorch 2.2.1+cu121
PyTorch Geometric 2.5.3
NumPy 1.26.4
```

The original research environment was configured at:

```text
/workspace/primekg_ddi_rgcn
```

with the Conda environment:

```text
primekg-rgcn
```

The original training pipeline used NVIDIA GPUs.

The full preprocessing and retraining pipeline is **not completely self-contained in the portable repository**. Therefore, the original research environment and data preparation process are not required for running the final lightweight application.

The original research notebooks included:

```text
00_environment_check
01_inspect_primekg
02_build_graph_variants
03_prepare_rgcn_data
04_train_rgcn
05_repeat_seeds
06_finalize_project
```

These notebooks document the research and experimental workflow but are not required for the lightweight application.

---

### B. Lightweight Web Application

The final CHEERS web application uses:

```text
FastAPI
NumPy
Python standard library
HTML
CSS
Vanilla JavaScript
```

The lightweight runtime does **not** require:

* PyTorch
* PyTorch Geometric
* CUDA
* GPU
* React
* Node.js
* npm
* External CDN

### Lightweight Runtime Files

The main model runtime artifacts are located in:

```text
final_release/lightweight_runtime/
├── ddi_runtime_embeddings.npz
├── drug_metadata.csv
├── known_positive_mask_packed.npz
└── manifest
```

The runtime performs lightweight NumPy-based scoring using the exported model artifacts:

```text
query_embedding @ (candidate_embeddings * ddi_relation).T
```

The lightweight runtime was verified against the G3/seed-44 model inference output.

### G3 Context Runtime

The graph-context runtime is located at:

```text
final_release/g3_context_runtime/
```

It contains relation-preserving support information used by the application:

```text
68,284 total forward support rows
25,653 Drug-Gene/Protein rows
42,631 Drug-Disease rows
```

### Running the Web Application

From the project root, install the lightweight application dependencies according to the project's dependency configuration and start the FastAPI application using the provided application entry point.

The application exposes the following main endpoints:

```text
/
 /api
 /api/health
 /api/model
 /api/experiment
 /api/verification
 /api/drugs/search
 /api/predict
 /api/context/pair
 /api/evidence/pair
 /docs
```

The `/docs` endpoint provides the automatically generated FastAPI API documentation after the server is running.

> **Note:** The exact launch command and host/port configuration should follow the current application entry point and dependency configuration included in the repository.

---

## 5.2. Troubleshooting

### Problem 1. Missing Python Dependencies

For the full research environment, verify the Python version:

```bash
python --version
```

Expected:

```text
Python 3.10.20
```

For model training, also verify that the required PyTorch and PyTorch Geometric versions are installed.

For the lightweight application, install only the dependencies required by the application environment.

---

### Problem 2. GPU or CUDA Errors

The full research and training environment requires a compatible NVIDIA GPU, CUDA configuration, and PyTorch installation.

The lightweight runtime does **not** require CUDA or a GPU.

For final demonstration and application testing, use the lightweight runtime whenever possible.

---

### Problem 3. Missing Model Artifacts

Verify that the lightweight runtime files are present:

```text
final_release/lightweight_runtime/
```

For the original research checkpoints, check:

```text
checkpoints/
```

The lightweight application uses exported runtime artifacts and does not require loading the original PyTorch training checkpoint for normal inference.

---

### Problem 4. Port Already in Use

If the configured application port is already occupied, stop the process currently using the port or change the application port according to the FastAPI launch configuration.

---

### Problem 5. External Evidence Is Unavailable

The FDA/PubMed evidence feature requires network access to the corresponding external services.

If external evidence retrieval is unavailable, the DDI prediction and graph-context components remain independent of the external evidence layer. The model inference itself does not depend on successful FDA/PubMed retrieval.

---

### Problem 6. Application Starts but Prediction Fails

Verify that the lightweight runtime artifacts are available and that the application can access:

```text
final_release/lightweight_runtime/ddi_runtime_embeddings.npz
final_release/lightweight_runtime/drug_metadata.csv
final_release/lightweight_runtime/known_positive_mask_packed.npz
```

Also check the application health and model endpoints:

```text
/api/health
/api/model
```

If the problem persists, check the server logs for missing files, invalid paths, or dependency errors.

---

# 6. Introduction Materials and Demonstration Video

## 6.1. Project Introduction Materials

- [Project Presentation (PDF)](docs/03.발표자료/project_presentation.pdf)
- [Project Presentation (PPTX)](docs/03.발표자료/project_presentation.pptx)
---

## 6.2. Demonstration Video

> **[YouTube Demo Video](https://www.youtube.com/watch?v=Z7AGAvI0R7Q)**

---

## 7. Team Contributions

The CHEERS project was collaboratively developed by three team members. While all members participated in project discussions, application testing, result interpretation, and preparation of the final graduation project materials, each member took primary responsibility for different parts of the research and system development.

### 7.1. Team Members and Roles

#### Byambasuren Tuvshinjargal

* Led data preprocessing, entity normalization, triple validation, and heterogeneous knowledge graph construction.
* Prepared DDI pairs and related biomedical relationship data for graph-based experiments.
* Implemented, trained, and evaluated the exploratory TransE model, including filtered link-prediction evaluation and metric analysis.
* Led the final R-GCN G0–G3 graph-composition training pipeline and multi-seed experiments.
* Analyzed and interpreted ranking, classification, cold-start, external-evaluation, and qualitative experimental results.
* Developed the backend/API, model-runtime integration, and frontend components of the CHEERS web application.
* Conducted application debugging, integration testing, UI/UX refinement, runtime verification, and model verification.
* Contributed to research-direction refinement, methodology design, related-work analysis, experimental documentation, figures, presentation and poster preparation, and final report writing.

#### Galbadrakh Buyandelger

* Processed, normalized, validated, and integrated disease–drug `treated_by` triples into the shared biomedical knowledge graph.
* Implemented, trained, optimized, and evaluated the exploratory ComplEx model.
* Conducted filtered link-prediction evaluation and metric analysis using MRR and Hits@K.
* Compared the exploratory TransE, ComplEx, and RotatE model results.
* Participated in final R-GCN experimentation, dataset verification, experimental evaluation, and result analysis.
* Supported application testing, experiment verification, and integration of research results into the final CHEERS system.
* Contributed to experimental and system documentation, result interpretation, presentation-material preparation, and final report writing.

#### Bavuujav Delgerbayar

* Constructed and validated `interacts_with` triples, including duplicate and reverse-pair handling.
* Implemented, trained, and evaluated the exploratory RotatE model and contributed to DDI safety-checking preparation.
* Conducted relation-level ablation, multi-seed performance analysis, five-seed verification, and paired statistical analysis.
* Performed DDI-edge cold-start evaluation and generalization analysis.
* Developed the G3 Subgraph Explorer and graph/context visualization features.
* Integrated local biomedical metadata, entity information, and relationship-detail features into the CHEERS web application.
* Contributed to web-application integration, debugging, quality assurance, reproducibility verification, result interpretation, and final report preparation.

---

# 8. References and Sources

## Biomedical Knowledge Graph

* PrimeKG — Precision Medicine Knowledge Graph

## Drug–Drug Interaction Data

* PrimeKG `drug_drug` relation

## External Evidence

* U.S. Food and Drug Administration — openFDA
* National Library of Medicine — PubMed

## Graph Neural Network

* Relational Graph Convolutional Network (R-GCN)
* Schlichtkrull, M., Kipf, T. N., Bloem, P., van den Berg, R., Titov, I., & Welling, M. (2018). *Modeling Relational Data with Graph Convolutional Networks.*

## Software and Frameworks

* PyTorch
* PyTorch Geometric
* NumPy
* FastAPI

## Project Resources

The repository includes:

```text
THIRD_PARTY_NOTICES.md
```

for third-party software and resource notices.
