Идея: генерация последовательности (антибактериального пептида) и предсказание, вероятности, что этот пептид может быть эффективен против определённой бактерии.

Note: Gram+/Gram- - две группы бактерий, классификация может быть бинарной для MVP и много классовой далее, то есть начать с бинарной.

2 части:
- генератор
- классификатор

---

Data:

1. Data sources and usage rights:
	Мы планируем использовать в нашем проекте 2 открытые базы данных, которые также были использованы авторами статьи Variational Autoencoder for Generation of Antimicrobial Peptides.
	1) [APD6 Antimicrobial Peptide Database](https://aps.unmc.edu/)
	2) [CAMPR4](https://camp.bicnirrh.res.in/)
2. Preprocessing steps:
	Первым делом нам необходимо собрать данные с web баз данных в .csv, после этого мы перейдём к EDA этапу.

---
Генератор:

1) baseline - [cVAE](https://pyro.ai/examples/cvae.html) c условием 
```
c = 0 # Gram-negative 
c = 1 # Gram-positive
```
2) Потом будем расширять наши условия до большего числа параметров, будем строить UMAP (Uniform Manifold Approximation and Projection) - good для наглядности результата генерации с условием.

Notes:
- Math view to model:
```
(x, c) → Encoder → [μ(x,c), σ(x,c)] → z ~ q_φ(z|x,c) = N(μ, σ²) → Decoder → p_θ(x̂|z,c)
```
- Метрики обучения:
	- ELBO (Evidence Lower Bound)
	- Reconstruction Loss — насколько хорошо восстанавливается вход
	- [KL Divergence](https://ru.ruwiki.ru/wiki/%D0%A0%D0%B0%D1%81%D1%81%D1%82%D0%BE%D1%8F%D0%BD%D0%B8%D0%B5_%D0%9A%D1%83%D0%BB%D1%8C%D0%B1%D0%B0%D0%BA%D0%B0_%E2%80%94_%D0%9B%D0%B5%D0%B9%D0%B1%D0%BB%D0%B5%D1%80%D0%B0) — насколько латентное пространство близко к prior
	- Total ELBO
	- Perplexity
- Метрики генерации:
	- Validity - доля сгенерированных последовательностей, которые являются корректными пептидами. (похоже на TP)
	- Uniqueness - lоля уникальных молекул среди валидных сгенерированных последовательностей.
	- Novelty - доля уникальных последовательностей, которые **не встречались** в обучающей выборке. (не факт, что они полезные, мы не можем этого проверить, ты понимаешь)

| Метрика                     | Целевое значение                   | Приоритет            | Категория   |
| --------------------------- | ---------------------------------- | -------------------- | ----------- |
| **ELBO**                    | максимизировать                    | Высокий              | Training    |
| **Reconstruction Loss**     | минимизировать                     | Высокий              | Training    |
| **KL Divergence**           | контролировать (не слишком низкий) | Высокий              | Training    |
| **Perplexity**              | < 15 для пептидов                  | Средний              | Training    |
| **Validity**                | > 95%                              | Высокий              | Quality     |
| **Uniqueness**              | > 70%                              | Высокий              | Quality     |
| **Novelty**                 | > 70%                              | Высокий              | Quality     |
| **Internal Diversity**      | максимизировать                    | Средний              | Diversity   |
| **Condition Accuracy**      | > 85%                              | Высокий              | Conditional |
| **Predicted Activity Rate** | > 70%                              | **Критически важно** | Biological  |
| **Property KL Divergence**  | < 0.5                              | Средний              | Biological  |

---

Классификация:

1) baseline - Gram+/Gram- классификация, с помощью таких моделей как: logistic regression, CatBoost, XGBoost, MLP (можно замутить малую CNN для feature extraction и выход давать в MLP)
2) Хочу попробовать собрать ансамбль (key word - diversity) из;
	1. Softmax, CatBoost, XGBoost, MLP
	2. MLP (метамодель) можно и log regression нужно попробовать

Notes:
- CatBoost чувствителен к категориальным признакам, вообще для бустинга нужна хорошая инженерия данных (PSAAC, AAC)
- брать random forest не хочется, потому как много памяти, больше сложность обучения
- **Важно**: Эти 4 модели должны быть **достаточно разными**:
	- Softmax: обучается на последовательностях (sequence-based learning)
	- XGBoost/CatBoost: обучаются на признаках (feature-based learning)
	- MLP: может работать с обоими типами входов


### Data

Data sources License/usage rights:
We plan to use two open databases in our project, which were also used by the authors of the article "Variational Autoencoder for Generation of Antimicrobial Peptides".
1. [APD6 (Antimicrobial Peptide Database)](https://aps.unmc.edu/)
2. [CAMPR4 AMP database](https://camp.bicnirrh.res.in/)

Preprocessing steps: 
- Export to a unified CSV, clean/normalize sequences and labels, define MVP condition c∈{0,1}c∈{0,1} (Gram-/Gram+), and create leakage-safe train/val/test splits; run EDA and build tokenized inputs + engineered features.

### Baseline Plan

First implementation will include follow parts:
1. Data preprocessing pipeline.
2. Binary labeling: "Gram+" and "Gram-".
3. Generator model - generates the peptide sequence.
4. Simple classifier model - suggests against which type of microorganisms the generated sequence may be effective.

Baseline approach: 
- Conditional VAE (cVAE) conditioned on Gram label, trained by maximizing ELBO.
- Baseline classifier via logistic regression/boosting on basic composition features, with an MLP as a neural baseline.

### Evaluation Plan

Metrics:

In our project, we will have two ML/DL tasks: generation and classification. We will use different metrics for model training and inference evaluation, depending on the task, model architecture, and current stage. Now we will define metrics for the baseline solution.

Generation:

For the сVAE generative model, we will use ELBO (Evidence Lower Bound) as the main training loss. ELBO is well-suited to our case: it improves reconstruction quality and regularizes the latent space (via the KL divergence to the prior). As generation inference metrics, we plan to use validity, uniqueness, and novelty.

Classifier:

For the classifier model, we will use precision, recall, F1, F2(when recall is prioritized), ROC AUC, and PR AUC (especially informative under class imbalance, plus ROC AUC for threshold-independent ranking assessment).


### XAI Method: t-SNE Visualization

**t-SNE (t-distributed Stochastic Neighbor Embedding) description:**

t-SNE is a nonlinear dimensionality reduction technique that projects high-dimensional data into 2D or 3D space while preserving local structure. The algorithm works in two stages:
1. In the original high-dimensional space, it converts Euclidean distances between points into conditional probabilities using a Gaussian distribution.
2. In the low-dimensional space, it models similarity using a Student's t-distribution and minimizes the Kullback-Leibler (KL) divergence between the two probability distributions.

t-SNE is particularly effective at creating compact, well-separated clusters of similar points, making it well for visualizing learned representations.

**t-SNE in our project:**

In our project, t-SNE serves as an explainability tool for interpreting the latent space of our cVAE:
- **Latent space interpretation**: Visualizing latent vectors z reveals how the model organizes knowledge about peptides—which regions correspond to Gram+ vs. Gram− activity.
- **Feature-activity relationships**: By coloring points by physicochemical properties (charge, hydrophobicity, Boman index), we can understand which features drive antimicrobial activity.
- **Generation validation**: Comparing projections of real vs. generated peptides shows whether the model captures the training data distribution.
- **Transparency for biologists**: Visual clusters are more interpretable than raw 128-dimensional latent vectors."

**Mathematical basis:**

Conditional probabilities in high-dimensional space (Gaussian kernel):

$$
p_{j|i} = \frac{\exp(-\|x_i - x_j\|^2 / 2\sigma_i^2)}{\sum_{k \neq i} \exp(-\|x_i - x_k\|^2 / 2\sigma_i^2)}
$$

Symmetrized joint probabilities:

$$
p_{ij} = \frac{p_{j|i} + p_{i|j}}{2N}
$$

Probabilities in low-dimensional space (Student's t-distribution):

$$
q_{ij} = \frac{(1 + \|y_i - y_j\|^2)^{-1}}{\sum_{k \neq l} (1 + \|y_k - y_l\|^2)^{-1}}
$$

Objective function (KL divergence):

$$
C = \sum_i \sum_j p_{ij} \log \frac{p_{ij}}{q_{ij}}
$$
---
## 1. Objectives and scope

### Project objective

The goal of this project is to build a **small conditional VAE (cVAE)** that can generate **novel antimicrobial peptide (AMP) sequences** and can be **steered by activity labels**, with the primary controllability target being:

- **anti-Gram-positive (Gram+) vs anti-Gram-negative (Gram−)** activity.

At this stage we treat all results as **in‑silico** (no wet‑lab validation). The baseline focuses on (1) a reproducible dataset with consistent activity tags, and (2) a compact generative model that can be trained and sampled on modest hardware.

  

### Stage 2 baseline deliverable

  

This Stage 2 deliverable baseline includes:

  

- **A dataset preprocessing pipeline**  

  Implemented in `Data/c_vector.py` (branch `Data`). The script loads multiple processed CSVs, derives/standardizes binary activity flags from the `Activity` text field using simple regex rules, and merges everything into a single master dataset.

  **Condition vector (7 binary tags):**

  - `is_antibacterial`

  - `is_anti_gram_positive`

  - `is_anti_gram_negative`

  - `is_antifungal`

  - `is_antiviral`

  - `is_antiparasitic`

  - `is_anticancer`

  **Key preprocessing decisions:**

  - Gram labels are extracted via substring matching (`"gram+"`, `"gram-"`, case‑insensitive).

  - `is_antibacterial` is derived as `Gram+ OR Gram−`.

  - Deduplication is done **by sequence** via `groupby("Sequence")` with **`max` aggregation** for the binary flags (so that existing `1`s are not lost when duplicates are collapsed).

  - Export artifacts:

    - `Data/processed/master_dataset_before_cleaning.csv`

    - `Data/processed/master_dataset.csv`

  

- **Training and inference scripts**  

  Provided in `Data/notebooks/baseline-cvae-and-tsne.ipynb` (branch `Data`). The notebook:

  1) loads `Data/processed/data.csv`,  

  2) builds a **character‑level vocabulary** (with `<PAD>/<SOS>/<EOS>/<UNK>` tokens),  

  3) tokenizes sequences to a fixed `max_len = max(Length) + 2`,  

  4) trains a compact **GRU-based cVAE** in PyTorch, and  

  5) performs conditional sampling and latent-space visualization.

  **Baseline cVAE architecture (high level):**

  - **Encoder:** Embedding → GRU → concatenate GRU hidden state with condition vector → two linear heads for `μ` and `log σ²`.

  - **Decoder:** initial hidden state computed from `[z, cond]` → GRU autoregressive decoding → linear projection to vocabulary logits.

  **Default hyperparameters (CPU‑friendly):**

  - `embed_dim=64`, `hidden_dim=128`, `latent_dim=32`, `dropout=0.2`

  - `batch_size=256`, `lr=1e-3`

  - early stopping with `patience=5` (max `num_epochs=150`)

  **Saved artifacts (from the notebook):**

  - `best_cvae.pt` (best checkpoint by validation loss)

  - `vocab.pkl` (vocabulary + `max_len` for decoding)

  - `parametric_tsne.pt` (optional 2D projection model)

  

- **A quantitative metric and qualitative samples**  

  The notebook includes a minimal generation evaluation routine for a chosen condition `cond`:

  

  **Quantitative metrics (computed over N generated samples):**

  - **Validity:** fraction of generated sequences consisting only of valid vocabulary symbols (excluding special tokens).

  - **Uniqueness:** fraction of unique sequences among valid generations.

  - **Novelty:** fraction of unique sequences not seen in the training set.

  

  **Qualitative samples:**

  - printed examples of generated sequences under selected conditions (e.g., Gram+ and Gram−),

  - a **t‑SNE / parametric t‑SNE** plot showing real training latents vs generated latents in a shared 2D space.

  

- **One sanity check**  

  We include one lightweight sanity check to verify the end‑to‑end pipeline before deeper experimentation:

  

  **Overfit / reconstruction sanity check (recommended):**

  - train the cVAE on a small subset (e.g., 256 sequences) for a few epochs,

  - confirm that the reconstruction loss decreases sharply,

  - decode a few training sequences with greedy argmax and check that reconstructions are close to the originals.

  

  This check ensures the tokenization, conditioning wiring, and decoder loop are implemented correctly.

  

- **CPU-friendly configuration options**  

  The baseline is designed to run on CPU (GPU is optional). Practical knobs for CPU‑only runs:

  

  - keep the default small model (`hidden_dim=128`, `latent_dim=32`), or reduce further (`hidden_dim=64`, `latent_dim=16`),

  - reduce `num_epochs` and/or rely on early stopping (already enabled),

  - lower `batch_size` if RAM is limited (e.g., 32–128),

  - skip or reduce the parametric t‑SNE training (`pt_epochs`) if runtime matters,

  - run with `device = "cpu"` explicitly (the notebook selects CPU automatically if CUDA is unavailable).

  

### Out of scope for this baseline

  

- wet‑lab validation or claims of real antimicrobial efficacy,

- strong activity predictors (MIC regression / calibrated classifiers),

- extensive hyperparameter sweeps or large‑scale generation campaigns.