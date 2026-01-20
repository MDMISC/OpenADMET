# Methodology Report

## Data Source

For training all single-task models, only the dataset provided by the competition organizers was used.  
No preliminary data preprocessing was applied to the original dataset.

## Graph-Based Molecular Representation Learning

A pretrained **Chemeleon Bond Message Passing Neural Network (MPNN)**  
(https://github.com/JacksonBurns/chemeleon) was used as the base molecular representation model.

For each of the nine target properties, an independent MPNN was fine-tuned using the corresponding
target-specific training dataset. Each dataset was randomly split into training (90%) and validation
(10%) sets.

After training, the best-performing checkpoint for each target-specific MPNN was frozen and used
exclusively as a feature extractor. The resulting molecular embeddings encode both molecular
structural information and target-dependent patterns learned during supervised fine-tuning.

The same target-specific MPNN models were then used to extract embeddings for the shared external
test set. These embeddings served as numerical feature vectors for downstream regression modeling.

## TabM Regression Model

For downstream modeling, the **TabM** framework  
(https://github.com/yandex-research/tabm) was employed.

TabM is a deep probabilistic architecture for tabular data that relies on ensembling *k* independent
submodels and specialized numerical embeddings. The model integrates several key components:

- **Piecewise-linear embeddings** for numerical features, enabling adaptive partitioning of
  feature value ranges;
- A compact **TabM-mini** backbone consisting of multiple computational blocks with residual
  connections and dropout regularization;
- An ensemble of *k* submodels whose predictions are averaged, reducing variance and improving model
  robustness, particularly on small datasets.

Prior to training, input features were standardized using a **QuantileTransformer**, and the target
variable was standardized using statistics computed on the training set.

Model optimization was performed using the **AdamW** optimizer with gradient clipping and an
early stopping strategy based on validation performance. Final TabM predictions were obtained using
an ensemble strategy with five different random initializations.
