# AI for Software Engineers

*Frontend Masters Course*

## The Core ML Loop

Machine learning is fundamentally about building a **model** from historic labeled data, then using that model to predict on new, unlabeled data.

The model is only as good as the data it was trained on, and that data must be a **representative sample** of the broader population.

### Key Components

**Training** — Feeding labeled data to an algorithm so it learns the mapping from inputs (features) to outputs (labels)

**Inference** — Using a trained model to make predictions on new inputs

**Overfitting** — The model learned the training data too well (including noise) and fails to generalize
- Addressed by validation, regularization, or more data

**Underfitting** — The model is too simple to capture the patterns in the data

---

## Data Exploration Process

1. **Select a representative sample** of historic labeled data
2. **Choose the features** (input variables) that matter
3. **Find the decision boundaries** that correctly classify as many training examples as possible — the "best fit"
4. **Validate on a held-out portion** of the data to check for overfitting
5. **Deploy:** Run inference on new, unlabeled data

---

## Model Types

| Model | Best For |
|-------|----------|
| **Decision Tree** | Interpretable rules, non-linear boundaries |
| **Random Forest** | More robust version of decision trees (ensemble) |
| **Linear / Logistic Regression** | Linear relationships, binary classification |
| **Support Vector Machine (SVM)** | High-dimensional data, clear margin separation |
| **Neural Network (NN)** | Complex patterns, unstructured data |
| **CNN (Convolutional NN)** | Image recognition, spatial data |
| **Transformer** | NLP, code generation, modern LLMs |

---

## Pre-Processing

### Dimensionality Reduction
Eliminate features that carry little signal.
- Fewer dimensions = faster training
- Less overfitting
- Example: Greyscale instead of full RGB if color isn't predictive

### Normalization / Scaling
Bring feature values onto a comparable scale (e.g., 0–1) so features with large absolute values don't dominate the model.

### Data Augmentation
Artificially expand the training set by generating variations:
- Rotate, flip, crop images
- Paraphrase text
Helps with small datasets.

### Handling Missing Values
Three approaches:
- **Impute** — Fill with mean/median/mode
- **Drop rows** — Remove rows with missing values
- **Treat as category** — "Missing" becomes its own category

---

## Key Vocabulary

**Features** — The input variables fed to the model
- Feature selection and engineering are often where the most value is created

**Labels** — The target variable (what you're predicting)
- In supervised learning, training data is labeled

**Supervised vs Unsupervised**
- **Supervised** — Has labeled data (classification, regression)
- **Unsupervised** — Finds structure in unlabeled data (clustering, dimensionality reduction)

**Bias vs Variance**
- **Bias** — Systematic error from a model that's too simple
- **Variance** — Sensitivity to noise from a model too complex
- **Goal:** Appropriate balance between the two

**Drift** — When real-world data distribution shifts away from the training distribution
- Model performance degrades
- Requires monitoring and retraining

**MNIST** — "Hello world" dataset for image classification
- 70,000 handwritten digits
- 28x28 pixels
- [Wikipedia](https://en.wikipedia.org/wiki/MNIST_database)

---

## LLMs and Transformers

Large Language Models (like GPT, Claude, Gemini) are transformer-based neural networks trained on massive text corpora.

### What's Relevant for Software Engineers

**LLMs predict the most likely next token** given context. Remarkably capable at:
- Code generation
- Code explanation
- Technical reasoning

### Prompt Engineering

How you frame input significantly affects output quality. Best practices:
- Be specific
- Provide context
- Give examples

### RAG (Retrieval-Augmented Generation)

Augment LLM responses by retrieving relevant documents from a database and including them in the prompt context.

Addresses the limitation of static training data.

### Fine-Tuning

Further training a pre-trained model on a domain-specific dataset to specialize its behavior.

### Hallucination

LLMs can generate confident-sounding but **incorrect information**.

**Always verify factual claims**, especially for code or technical specifics.

---

## Practical Applications for Engineers

- **Code generation** — Auto-complete, test generation
- **Code explanation** — Understanding unfamiliar code
- **Bug detection** — Finding potential issues
- **Documentation generation** — Auto-doc tools
- **Code review** — Automated style and logic checks
- **Refactoring suggestions** — Improving code structure

---

*Course: Frontend Masters*
