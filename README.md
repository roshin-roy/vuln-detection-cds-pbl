# Vulnerability Detection — CDS Project

This repository contains the solutions for the **Cybersecurity Data Science (CDS)** project from the Institute of Software Security (E22), Hamburg University of Technology.

The project covers vulnerability-model evaluation, vulnerability-dataset construction, and machine-learning-based vulnerability prediction.

## Contributors

- Moniya Mohan | Matriculation Number: 675659
- Roshin Roy | Matriculation Number: 674412

## Repository Contents

```text
vuln-detection-cds-pbl/
├── Part_1/
│   └── CDS_project_part1.ipynb
├── Part_2/
│   ├── CDS_project_part2.ipynb
│   ├── vuln_dataset.jsonl
│   └── vuln_dataset.csv
├── Part_3/
│   ├── CDS_project_part3.ipynb
│   └── predictions.jsonl
└── README.md
```

Additional datasets and source files required to run the notebooks are not included in this repository. Update the `BASE_PATH` variable in the notebooks to point to the local directory containing the required data or cloned repositories.

## Part 1 — Evaluation of a Pretrained Model

Part 1 evaluates a provided machine-learning model for source-code vulnerability prediction.

The notebook includes:

- loading the provided HDF5 validation dataset;
- creating a PyTorch dataset and data loader;
- loading the pretrained model;
- generating vulnerability predictions;
- calculating the confusion-matrix values;
- calculating accuracy, precision, recall, and F1 score;
- discussing the effect of class imbalance on model evaluation.

The input vectors have 768 features, matching the input dimension of the provided prediction model.

## Part 2 — Vulnerability Dataset Construction

Part 2 creates a labeled Java vulnerability dataset using ProjectKB and vulnerability-fixing commits from open-source repositories.

The notebook includes:

- reading ProjectKB vulnerability statements;
- extracting repository URLs and fixing commit hashes;
- cloning and inspecting the corresponding Git repositories;
- locating methods modified by security fixes;
- extracting vulnerable methods from before the fix;
- extracting non-vulnerable methods from after the fix;
- sampling additional unchanged non-vulnerable methods;
- exporting the generated dataset in JSON Lines format.

The main output used in Part 3 is:

```text
vuln_dataset.jsonl
```

## Part 3 — Vulnerability Prediction Model

Part 3 trains and evaluates a model that predicts whether a Java function is vulnerable.

The notebook includes:

- loading the dataset generated in Part 2;
- checking class distribution and duplicate functions;
- splitting the data into training, test, and validation sets;
- generating function embeddings using `microsoft/codebert-base`;
- training multilayer perceptron classifiers with PyTorch;
- handling class imbalance using weighted cross-entropy loss;
- comparing different MLP configurations;
- evaluating the models using accuracy, precision, recall, and F1 score;
- training the selected model on the complete labeled dataset;
- generating predictions for the challenge dataset.

### Model architecture

The solution uses two stages:

1. **CodeBERT feature extraction**  
   `microsoft/codebert-base` converts each function into a 768-dimensional representation. The first-token representation is used as the fixed-size function embedding.

2. **Multilayer perceptron classification**  
   A configurable PyTorch MLP predicts one of two classes:
   - safe;
   - vulnerable.

CodeBERT is used as a frozen feature extractor. Only the MLP classifier is trained.

### Preprocessing

Functions are tokenized with:

- padding enabled;
- truncation enabled;
- maximum sequence length of 256 tokens;
- batches of 32 functions.

Embedding generation is divided into chunks of 2,048 samples. Each chunk is saved as an `.npy` file, allowing interrupted runs to resume without recomputing completed chunks.

Generated files follow this pattern:

```text
emb_chunk_0.npy
emb_chunk_2048.npy
emb_chunk_4096.npy
...
```

### Dataset split

A stratified and reproducible split is created using `random_state=42`:

- 70% training;
- 15% test;
- 15% validation.

Stratification preserves the vulnerable/non-vulnerable ratio across the three subsets.

The generated dataset used in the notebook contains:

| Class | Samples |
|---|---:|
| Vulnerable | 1,780 |
| Non-vulnerable | 16,328 |
| Total | 18,108 |

### Experiments

A smoke test is first performed with 1,000 training samples to verify the complete pipeline.

Two full model configurations are then compared:

| Model | Hidden layers | Dropout |
|---|---|---:|
| Model A | 256 | 0.3 |
| Model B | 512 → 256 → 128 | 0.4 |

The notebook reports the following test-set results:

| Model | Accuracy | Precision | Recall | F1 |
|---|---:|---:|---:|---:|
| Model A | 0.618 | 0.184 | 0.843 | 0.302 |
| Model B | 0.566 | 0.170 | 0.876 | 0.284 |

Model A obtains the higher vulnerable-class F1 score and is therefore selected for the final challenge model.

### Challenge prediction

The selected Model A configuration is retrained on all 18,108 labeled functions. The challenge functions are encoded using the same CodeBERT preprocessing pipeline and classified by the final MLP.

The final output is written to:

```text
predictions.jsonl
```

## Requirements

Python 3.10 or newer is recommended.

Install the required packages with:

```bash
pip install numpy pandas matplotlib scikit-learn torch transformers h5py pyyaml pydriller tqdm
```

Git must also be installed because Part 2 clones and examines external repositories.

A CUDA-enabled GPU is recommended for generating CodeBERT embeddings, but the code can also run on CPU.

## Running the Project

Run the notebooks in the following order:

1. `Part_1/CDS_project_part1.ipynb`
2. `Part_2/CDS_project_part2.ipynb`
3. `Part_3/CDS_project_part3.ipynb`

Before running the notebooks, update any local dataset, source-code repository, or `BASE_PATH` values used in the code.

## Output Files

Depending on the executed steps, the project may generate:

```text
vuln_dataset.jsonl
predictions.jsonl
emb_chunk_*.npy
```

## Possible Improvements

Future improvements could include:

- selecting the final model with the test split and reporting one final unbiased evaluation on the validation split;
- using early stopping based on holdout loss;
- comparing additional class weights or focal loss;
- fine-tuning CodeBERT end to end;
- increasing the token limit or using chunked representations for long functions;
- grouping samples by repository during splitting to reduce repository-specific leakage;
- performing stratified k-fold cross-validation;
- saving the trained model weights and preprocessing configuration.