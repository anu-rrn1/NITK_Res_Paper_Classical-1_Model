# Classical-1 Model for SST-2 GLUE

## Aim

The aim of this work is to evaluate a **classical model** for sentiment classification on the **SST-2 GLUE dataset** and compare its performance with the hybrid quantum-classical model. The classical model uses the same input representation, trainable projection, and final classifier as the quantum model, while replacing the quantum circuit with a small classical neural network.

## Model

The model takes **768-dimensional BERT embeddings** and uses the same trainable projection network to reduce them to **8 values**. Instead of passing these 8 values through an 8-qubit quantum circuit, the Classical-1 model processes them using a **classical equivalent block with an 8 → 5 → 8 architecture**. The resulting 8-dimensional representation is then passed to the same small classical classifier for the final sentiment prediction.

The Classical-1 model contains **103 trainable parameters in the classical middle block**, compared with **96 trainable parameters in the quantum circuit**. This provides a closely capacity-matched classical comparison. The middle block architecture is `8 → 5 → 8`, with LayerNorm and GELU activations.

## Workflow

```text
SST-2 GLUE Dataset
      ↓
BERT Embeddings
      ↓
Train / Validation / Test Split
      ↓
Standardization
      ↓
Trainable Projection
768 → 128 → 64 → 8
      ↓
Classical Equivalent Block
8 → 5 → 8
      ↓
Classical Classifier
8 → 16 → 1
      ↓
Sentiment Prediction
      ↓
SST-2 GLUE Predictions
      ↓
SST-2.tsv
```

The trainable projection and final classifier are kept architecturally identical to the hybrid quantum-classical model. Only the middle computational block is replaced with the classical equivalent block, allowing the two models to be compared under the same input processing and classification setup.

## Architecture

### Trainable Projection

The BERT embeddings are reduced from 768 dimensions to 8 dimensions using the trainable projection network:

The resulting 8-dimensional representation is then scaled and passed to the classical middle block.

### Classical Equivalent Block

The 8-dimensional representation is processed by:

```text
8
 ↓
Linear
8 → 5
 ↓
LayerNorm
 ↓
GELU
 ↓
Linear
5 → 8
 ↓
GELU
 ↓
8
```

This block contains **103 trainable parameters** and serves as the classical counterpart to the quantum computational block.


```

The final output is a single logit for binary sentiment classification.

## Complete Architecture

```text
BERT Embedding
768
 ↓
Trainable Projection
768 → 128 → 64 → 8
 ↓
ClassicalEquivalentBlock
8 → 5 → 8
 ↓
Classifier
8 → 16 → 1
 ↓
Sentiment Prediction
```

## Training

The model is trained using:

* **Loss:** Binary Cross-Entropy with Logits Loss (`BCEWithLogitsLoss`)
* **Optimizer:** Adam
* **Learning rate:** `1e-3`
* **Batch size:** `128`
* **Epochs:** `10`

The training and validation procedure follows the same setup as the hybrid quantum-classical model. The best checkpoint is selected based on validation loss.

## Comparison with the Hybrid Quantum-Classical Model

The Classical-1 model is designed to keep the surrounding architecture unchanged:

| Component               | Hybrid Quantum-Classical        | Classical-1                     |
| ----------------------- | ------------------------------- | ------------------------------- |
| Input                   | 768-dimensional BERT embeddings | 768-dimensional BERT embeddings |
| Trainable Projection    | 768 → 128 → 64 → 8              | 768 → 128 → 64 → 8              |
| Middle Block            | 8-qubit quantum circuit         | 8 → 5 → 8 classical block       |
| Middle-Block Parameters | 96 quantum parameters           | 103 classical parameters        |
| Classifier              | 8 → 16 → 1                      | 8 → 16 → 1                      |
| Task                    | SST-2 sentiment classification  | SST-2 sentiment classification  |

Thus, the main architectural difference is the computational block applied to the shared 8-dimensional representation.

## Prediction and Submission

After training, the best Classical-1 model is used to generate predictions on the SST-2 test set. The predictions are paired with the original test IDs and saved in the GLUE submission format:

```text
index    prediction
0        0
1        1
2        1
...
```

The submission file is saved as:

```text
SST-2.tsv
```

The test IDs are preserved from the original SST-2 test split, ensuring that predictions remain aligned with the corresponding test examples.

## Output

The model produces:

```text
SST-2.tsv
```

containing the predicted sentiment labels for all SST-2 GLUE test instances.
